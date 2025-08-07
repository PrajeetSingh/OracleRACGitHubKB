# Startup and Shutdown RAC Instances

## Start & Stop RAC Databases

```sh
# Display the database registered in the repository
srvctl config database

# Display configuration details of database
srvctl config database -d RAC
```

```sh
# Start/Stop/Check database (All Instances, Services and Listeners)
srvctl status database -d RAC
srvctl start database -d RAC -o nomount / mount / open
srvctl stop database -d RAC -o immediate
srvctl stop database -d RAC -eval
```

```sh
# Start/Stop/Check RAC Instances
srvctl status instance -d RAC -i instancename
srvctl start instance -d RAC -i instancename
srvctl stop instance -d RAC -i instancename
srvctl stop instance -d RAC -i instance1,instance2

srvctl add instance -d RAC -i instancename -n racnode1
srvctl modify instance -d RAC -i instancename -n racnode1 -K
srvctl remove instance -d RAC -i instancename
```

To manage PDBs, we need to add them as Oracle Service, though from version 12c to 19.8, we could add them using `srvctl add pluggabledatabase` too.

We do not directly start or stop a PDB with srvctl. Instead we create a dedicated service for the PDB. When we start the service, Clusterware automatically opens the PDB on the required instances. When we stop the service, the PDB is closed.

```sh
# Add a Service for the PDB
srvctl add service -db RAC -service mypdb_svc -pdb mypdb

# Check your existing service
srvctl config service -d RAC -s mypdb_svc

# Start and Stop the PDB via its Service
srvctl start service -d RAC -s mypdb_svc
srvctl stop service -d RAC -s mypdb_svc
srvctl status service -d RAC -s mypdb_svc
```

OR

```sql
-- Open a PDB in the current instance
alter pluggable database pdb2  open;

-- Open a PDB in some instances
alter pluggable database pdb2 open Instances = ('RACInstance1');

-- Open a PDB in all instances
alter pluggable database database all open instances = all;

alter pluggable database pdb2 save state;
-- OR
alter pluggable database all save state;
```

Close a PDB in the current instance only

```sql
alter pluggable database pdb2 close;
```

Close a PDB in some or all instances of the CDB

```sql
alter pluggable database pdb2 close instances = ('racInstance1');
-- OR
alter pluggable database pdb2 close instances = all;
```

```sh
# Start/Stop/Check ASM
srvctl status asm 
srvctl status asm -a
srvctl status asm -n racnode1 -a

srvctl start asm 
srvctl start asm -n racnode1
srvctl stop asm
srvctl stop asm -n racnode1 -f

srvctl config asm
srvctl add asm -n racnode3 -G +DATA
srvctl remove asm -n racnode3
srvctl modify asm -n racnode3 -p '+DATA'
```

```sh
# Start/Stop/Check Listener
srvctl start listener -n racnode1
srvctl stop listener -n racnode1
srvctl status listener -n racnode1

srvctl config listener
srvctl add listener -n racnode1 -l Listener1 -p 1522
srvctl modify listener -l Listener1 -p 1523
srvctl remove listener -l Listener1
```

```sh
# Check Nodeapps, like VIP, Network, and ONS
srvctl status nodeapps
srvctl status nodeapps -n racnode1 
srvctl config nodeaspps
srvctl config nodeapps -n racnode2
```

```sh
# Check SCAN listener
srvctl status scan
srvctl config scan
srvctl stop scan
srvctl start scan

srvctl status scan_listener
srvctl config scan_listener
srvctl stop scan_listener
srvctl start scan_listener

# Add/Remove SCAN VIP & Listener

srvctl add scan -n <scan_name>
srvctl remove scan

srvctl add scan_listener
srvctl remove scan_listener

# Change SCAN Listener Port
srvctl modify scan_listener -p <port_number>
```

```sh
# Check Oracle Service
srvctl config service -db RAC

srvctl add service -d RAC -s service_name -preferred racnode1,racnode2
srvctl remove service -d RAC -s service_name
srvctl start service -d RAC -s service_name
srvctl stop service -d RAC -s service_name
srvctl relocate service -d RAC -s service_name -i instance
```

```sh
# Check Node Applications (VIP, Network, GSD, ONS)
srvctl config nodeapps
srvctl status nodeapps
srvctl status nodeapps -n racnode1

srvctl stop nodeapps
srvctl stop nodeapps -n racnode1

srvctl start nodeapps
srvctl start nodeapps -n racnode1
```

## Oracle ASM (ASMCLI / ASMCMD)

### List and manage Disk Groups

```sh
asmcmd lsdg
asmcmd mkdg -G DATA -c HIGH -A NORMAL +DATA2
asmcmd dropdg +OLD_DG
```

### View and Rebalance disks

```sh
asmcmd lsdsk -k
asmcmd rebal -G DATA -p 8
asmcmd lsop
```

```sh
asmcmd spget
asmcmd spbackup +data/.../registry.###.######## /backup/spfile.ora
```

```sh
asmcmd showclustermode
asmcmd showversion
```

<!-- https://valehagayev.wordpress.com/2016/04/16/useful-oracle-rac-commands/ -->