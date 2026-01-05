# Oracle 19c Non‑ASM → 19c RAC ASM Migration (End‑to‑End Runbook)

Validated, complete, and production‑ready

## PHASE 0 — Prerequisites & Assumptions

Source DB: 19c Single Instance, Non‑ASM
Target: 19c RAC (2 nodes), ASM (+DATA, +FRA)
Backup-based restore (RMAN Level 0 + archivelogs + PFILE + Control File)
DB name: CDBAPP1
RAC instances:
Node 1 → cdbapp11
Node 2 → cdbapp12
SCAN name: rac-scan.localdomain

## PHASE 1 — Restore Database Into ASM (Node 1)

### 1.0 Preparation

#### Verify both nodes are active in the cluster

`olsnodes -n`

#### Verify the high-availability stack is healthy on both nodes

```bash
crsctl check cluster -all

vi .bash_profile
export ORACLE_SID=cdbapp1
source .bash_profile
mkdir -p /u01/app/oracle/admin/cdbapp1/adump
mkdir /u01/app/oracle/backup
cd /u01/app/oracle/backup
rsync -avhzP oracle@192.168.56.110:/u01/app/oracle/backup ./
```

#### Create directory for Redo logs. It is temporarily required for redo logs, later can be removed as new Redo Logs will be in ASM.

`mkdir -p /u01/app/oracle/oradata/CDBAPP1/`

Generate a Clean PFILE:

`CREATE PFILE='/tmp/source_init.ora' FROM SPFILE;`

Sanitize the PFILE: Edit source_init.ora, ensuring these specific values:

*.control_files='+DATA','+FRA'
*.db_create_file_dest='+DATA'
*.db_recovery_file_dest='+FRA'
*.local_listener=''
*.db_create_online_log_dest_1='+DATA' (This is critical for the Redo Log fix).

#### 1.1 Start instance in NOMOUNT using temporary PFILE

`STARTUP NOMOUNT PFILE='source_init.ora';`

### 1.2 Restore controlfile into ASM

`RESTORE CONTROLFILE FROM '/u01/app/oracle/backup/CONTROL.bkp';`
`ALTER DATABASE MOUNT;`

### 1.3 Catalog backups

`CATALOG START WITH '/u01/app/oracle/backup';`

### 1.4 Restore datafiles into ASM

```bash
RUN {
  SET NEWNAME FOR DATABASE TO '+DATA';
  RESTORE DATABASE;
  SWITCH DATAFILE ALL;
}
```

### 1.5 Recover and open RESETLOGS

`RECOVER DATABASE;`
`ALTER DATABASE OPEN RESETLOGS;`

## PHASE 2 — Fix Redo Logs (Mandatory Before RAC Conversion)

### 2.1 Add new ASM redo groups

Thread 1 groups (4, 5, 6) must be the same size as Thread 2 groups (7, 8, 9).

```SQL
ALTER DATABASE ADD LOGFILE GROUP 4 ('+DATA') SIZE 200M;
ALTER DATABASE ADD LOGFILE GROUP 5 ('+DATA') SIZE 200M;
ALTER DATABASE ADD LOGFILE GROUP 6 ('+DATA') SIZE 200M;
```

### 2.2 Switch logs until old groups become INACTIVE

`ALTER SYSTEM SWITCH LOGFILE;`
`ALTER SYSTEM CHECKPOINT;`

Repeat until filesystem groups show INACTIVE.

### 2.3 Drop old filesystem redo logs

```sql
ALTER DATABASE DROP LOGFILE GROUP 1;
ALTER DATABASE DROP LOGFILE GROUP 2;
ALTER DATABASE DROP LOGFILE GROUP 3;
-- etc.
```

## PHASE 3 — Fix TEMPFILE (Often Forgotten)

### 3.1 Check TEMPFILE location

`select name, bytes/1024/1024 size_mb from v$tempfile;`

### 3.2 If TEMPFILE is on filesystem, move it

```sql
ALTER TABLESPACE TEMP ADD TEMPFILE '+DATA' SIZE 2G AUTOEXTEND ON;
ALTER DATABASE TEMPFILE '/old/path/temp01.dbf' DROP;
```

## PHASE 4 — RAC-Specific Storage Setup

### 4.1 Create UNDO tablespace for Node 2

`CREATE UNDO TABLESPACE undotbs2 DATAFILE '+DATA' SIZE 200M AUTOEXTEND ON;`

### 4.2 Create redo thread for Node 2

Thread 2 groups (7, 8, 9) must be the same size as Thread 1 groups (4, 5, 6).

```sql
ALTER DATABASE ADD LOGFILE THREAD 2 GROUP 7 ('+DATA') SIZE 200M;
ALTER DATABASE ADD LOGFILE THREAD 2 GROUP 8 ('+DATA') SIZE 200M;
ALTER DATABASE ADD LOGFILE THREAD 2 GROUP 9 ('+DATA') SIZE 200M;

ALTER DATABASE ENABLE PUBLIC THREAD 2;
```

## PHASE 5 — RAC Parameter Configuration

### 5.1 Build final SPFILE in ASM

`CREATE SPFILE='+DATA/CDBAPP1/spfilecdbapp1.ora' FROM MEMORY;`

```bash
vi /u01/app/oracle/product/19.0.0/dbhome_1/dbs/initcdbapp1.ora
SPFILE='+DATA/CDBAPP1/spfilecdbapp1.ora'
```

```sql
shutdown immediate;
startup;
```

## 5.2 Set RAC parameters

```sql
ALTER SYSTEM SET cluster_database=TRUE SCOPE=SPFILE;

ALTER SYSTEM SET instance_number=1 SCOPE=SPFILE SID='cdbapp11';
ALTER SYSTEM SET instance_number=2 SCOPE=SPFILE SID='cdbapp12';

SELECT tablespace_name, status FROM dba_tablespaces WHERE contents = 'UNDO';
CREATE UNDO TABLESPACE UNDOTBS2 DATAFILE '+DATA' SIZE 200M AUTOEXTEND ON;
SELECT tablespace_name, status FROM dba_tablespaces WHERE contents = 'UNDO';

ALTER SYSTEM SET undo_tablespace='UNDOTBS1' SCOPE=SPFILE SID='cdbapp11';
ALTER SYSTEM SET undo_tablespace='UNDOTBS2' SCOPE=SPFILE SID='cdbapp12';

ALTER SYSTEM SET remote_listener='rac-scan.localdomain:1521' SCOPE=BOTH SID='*';
```

### 5.3 Set local listener per node (critical for PMON registration)

```sql
ALTER SYSTEM SET local_listener='(ADDRESS=(PROTOCOL=TCP)(HOST=rac1-vip)(PORT=1521))' SCOPE=SPFILE SID='cdbapp11';
ALTER SYSTEM SET local_listener='(ADDRESS=(PROTOCOL=TCP)(HOST=rac2-vip)(PORT=1521))' SCOPE=SPFILE SID='cdbapp12';
```

## PHASE 6 — Clusterware Registration

### 6.1 Register database

`srvctl add database -db cdbapp1 -oraclehome $ORACLE_HOME -dbtype RAC -spfile +DATA/CDBAPP1/spfilecdbapp1.ora -diskgroup "DATA,FRA"`

### 6.2 Create password file in ASM

`orapwd file='+DATA/CDBAPP1/PASSWORD/orapwcdbapp1' dbuniquename='cdbapp1' password='Welcome_2026_#' force=y`

### 6.3 Update srvctl with password file

`srvctl modify database -db cdbapp1 -pwfile +DATA/CDBAPP1/PASSWORD/orapwcdbapp1`

### 6.4 Register instances

Start both Nodes now and verify rac1/rac2 names

```bash
olsnodes -n
crsctl check cluster -all
crsctl stat res -t

srvctl add instance -db cdbapp1 -instance cdbapp11 -node rac1
srvctl add instance -db cdbapp1 -instance cdbapp12 -node rac2
```

### 6.5 Create application service

```bash
srvctl add service -db cdbapp1 -service app1_srvc -preferred "cdbapp11,cdbapp12" -pdb APP1
srvctl start service -db cdbapp1 -service app1_srvc
srvctl status service -db cdbapp1 -service app1_srvc
srvctl config service -db cdbapp1
```

`select inst_id, service_id, name from gv$active_services;`

## PHASE 7 — Node-Level OS Setup

### 7.1 Node 1

```bash
echo "SPFILE='+DATA/CDBAPP1/spfilecdbapp1.ora'" > $ORACLE_HOME/dbs/initcdbapp11.ora

vi .bash_profile
export ORACLE_SID=cdbapp11
source .bash_profile
```

### 7.2 Node 2

```bash
mkdir -p $ORACLE_BASE/admin/cdbapp1/adump
mkdir -p $ORACLE_HOME/dbs
echo "SPFILE='+DATA/CDBAPP1/spfilecdbapp1.ora'" > $ORACLE_HOME/dbs/initcdbapp12.ora
vi .bash_profile
export ORACLE_SID=cdbapp12
source .bash_profile
```

## PHASE 8 — Final Startup & Validation

### 8.1 Restart via GI

`shutdown immediate`

```bash
rm -f $ORACLE_HOME/dbs/lkCDBAPP1*
srvctl stop database -db cdbapp1 -force
srvctl start database -db cdbapp1
srvctl start service -db cdbapp1 -service app1
```

### 8.2 Validate instances

```sql
SELECT inst_id, instance_name, status FROM gv$instance;
SELECT thread#, instance, status FROM gv$thread;
```

### 8.3 PDB State Management

```sql
ALTER PLUGGABLE DATABASE ALL OPEN;
ALTER PLUGGABLE DATABASE ALL SAVE STATE;
```

### 8.4 Validate listener registration (as GRID user)

```bash
lsnrctl status
lsnrctl status LISTENER_SCAN1
srvctl status listener
srvctl status scan_listener

netstat -tulpn | grep 1521
```

### PHASE 9 — Post‑RESETLOGS Backup (Mandatory)

BACKUP DATABASE PLUS ARCHIVELOG

#### Post-Migration Archive Log Cleanup.

Since the RESTORE and RESETLOGS process generates significant metadata, your FRA might fill up quickly.

Run this via RMAN after your backup:

```bash
rman target /
DELETE ARCHIVELOG ALL COMPLETED BEFORE 'SYSDATE-1';
```

## PHASE 10 — TNS entry

app1=
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = rac-scan)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = app1)
    )
  )
