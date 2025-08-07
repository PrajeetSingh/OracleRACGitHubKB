# Oracle RAC and Multi-tenant Architecture

* In a MultiTenant architecture, there are:
  * Several instances per CDB
  * One UNDO tablespace per CDB instance
  * At least two groups of REDO log files per CDB instance

* Each container is exposed as a Service
  * The root
  * Each PDB

* Management of Services
  * Default database services
  * Policy managed databases
    * Singletons
    * Uniform across all servers in a server pool

* Checking Status of CDB instances

```sh
srvctl status database -d RACDB

lsnrctl status
```

* Check the UNDO tablespaces

```sql
select tablespace_name, con_id from cdb_tablespaces where contents = 'UNDO';
```

> **Remember**: In Multi-tenant architecture, Oracle opens REDO Log files and datafiles of Root container, not PDB containers.
> By default datafiles of PDB are closed in open state.

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

alter pluggable database pdb2 discard state;
-- OR
alter pluggable database all discard state;
```

## Closing a PDB in a RAC CDB

* Shutdown the instances of a RAC CDB:
  * All PDBs are closed
  * CDB closed/CDB dismounted/Instance shut down

```sh
srvctl stop database -d RACDB
```

* Close a PDB

  * In the current instance only

```sql
alter pluggable database pdb2 close;
```

* In some or all instances of the CDB

```sql
alter pluggable database pdb2 close instances = ('racInstance1');
-- OR
alter pluggable database pdb2 close instances = all;
```

## Managing Services

There are two categories of services to manage:

* Default PDB services; one service per PDB
  * Created at PDB creation
  * Not managed by the clusterware
  * Started at PDB opening

* Dynamic PDB services
  * Useful to Start and Stop PDBs across the CDB instances
  * Manually assigned to PDBs with `srvctl`
  * Manually started with `srvctl`
  * Automatically restored to its original state by clusterware
  * PDB automatically opened when the service is started

## Adding a PDB service to a RAC CDB

* A PDB can be assigned several services
* Each PDB service can be exposed on some or all of the RAC instances

```sh
srvctl add service -d RACDB -pdb pdb1 -service pdb1srvc

srvctl start service -d RACDB -s pdb1srvc
```

## Dropping a PDB from a RAC CDB

1. Remove a Dynamic PDB service

```sh
srvctl remove service -db RACDB -service mysrvc
```

2. Close the PDB in all instances

  ```sql
  alter pluggable database PDB1 close instances = all;
  ```

3. Drop the PDB

  ```sql
  drop pluggable database pdb1 [ including datafiles ];
```
