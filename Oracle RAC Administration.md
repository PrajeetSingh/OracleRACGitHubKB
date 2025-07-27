# Oracle RAC Administration

## Policy Managed Databases

**Server Pool**: It is a group of servers in a single cluster. Instances are not hard linked to a Node like Administrator managed databases. When we restart the Cluster, instance1 may startup on node2 and instance2 on node1.

Same way, when we add a new Node to the `server pool`, instance is automatically added to the cluster on new Node, which is not in the case of Administrator managed Clusters.
Like, we'll have to manually add new instance if we want to add new node to the cluster.

There should be different set of REDO Groups assigned to different Nodes. Thread 1 is assigned to Instance 1 and Thread 2 is assigned to Instance 2. If we add new node, then we need to manually add new REDO Log groups for this Instance but if Policy managed database, all such things are automated. Same is the case of dropping the node, things will be automatically taken care by server pool in case of policy managed databases but need to be done manually by DBA for Adminsitrator managed databases.

Policy managed database is Dynamic database management, whereas Administrator managed databases are Static database management.

So, in case of Policy managed databases, instances are associated to Server Pool, not to nodes, unlike Administrator managed databases.

## Associate an UNDO Tablespace with a particular RAC instance

```sql
alter system set undo_tablespace=UNDOTBS1 sid='RAC01';
alter system set undo_tablespace=UNDOTBS2 sid='RAC02';

-- Here, if you run above commands in CDB, UNDO tablespaces will be assigned to CDB but if you run inside PDB, these will be assigned to PDBs.
```

## Managing PDBs and Services

In Oracle RAC, we can manage RAC PDBs by managing services. Assign one database service to each PDB to coordinate start, stop and placement of PDBs across instances.

### To add a service for a PDB

```sh
srvctl add service -db RAC -pdb mypdb -service mypdbsrv
```

### To start and stop the PDB

```sh
# Check your existing service
srvctl config service -d RAC -s mypdb_svc

# Start and Stop the PDB via its Service
srvctl start service -d RAC -s mypdb_svc
srvctl stop service -d RAC -s mypdb_svc
srvctl status service -d RAC -s mypdb_svc
```

```sql
col inst_name format a20
select * from v$active_instances;
```

By default, `Management policy` is `AUTOMATIC`. Means, when server is restarted, Oracle instance will start automatically, or when instance crashes, it'll attempt to restart automatically.

We can change this policy to `MANUAL` too.

```sh
srvctl config database -d RAC -a
srvctl modify database -d RAC -policy MANUAL;
```

> Remember: Pfiles have pointers to SPFILES.

## SPFILE parameter values and RAC

We can change parameter settings using `ALTER SYSTEM` command, but now in RAC, we need to use `sid='<sid|*>'` along with it.

```sql
-- To set a parameter value. For PDB, ISPDB_MODIFIABLE value for the parameter must be TRUE in GV$SYSTEM_PARAMETER view.
alter system set <parameter_name>=<value> scope=MEMORY|SPFILE|BOTH sid='<sid|*>';

-- To reset a parameter value to default
alter system reset <parameter_name>  scope=MEMORY|SPFILE|BOTH sid='<sid|*>';

col name format a50
col value format a100
select name, value From gv$system_parameter where name = <parameter_name>;
```

## Terminating Sessions on a Specific Instance

```sql
select sid, serial#, inst_id from gv$session where username = 'myuser';

alter system kill session 'SID,Serial#,@InstID';

alter system kill session '101,1003,@1';
```

## How SQL * Plus commands affect instances

| SQL*Plus Command           | Associated Instance                                                                 |
|----------------------------|-------------------------------------------------------------------------------------|
| ARCHIVE LOG                | Generally affects the current instance                                              |
| CONNECT                    | Affects the default instance if no instance is specified in the CONNECT command     |
| RECOVER                    | Does not affect any particular instance, but rather the database                    |
| SHOW PARAMETER / SHOW SGA  | Displays parameters and SGA info for the current instance                           |
| STARTUP / SHUTDOWN         | Operate on the current instance                                                     |
| SHOW INSTANCE              | Shows details about the current instance                                            |
