# Oracle RAC Administration

## Policy Managed Databases

**Server Pool**: It is a group of servers in a single cluster. Instances are not hard linked to a Node like Administrator managed databases. When we restart the Cluster, instance1 may startup on node2 and instance2 on node1.

Same way, when we add a new Node to the `server pool`, instance is automatically added to the cluster on new Node, which is not in the case of Administrator managed Clusters.
Like, we'll have to manually add new instance if we want to add new node to the cluster.

There should be different set of REDO Groups assigned to different Nodes. Thread 1 is assigned to Instance 1 and Thread 2 is assigned to Instance 2. If we add new node, then we need to manually add new REDO Log groups for this Instance but if Policy managed database, all such things are automated. Same is the case of dropping the node, things will be automatically taken care by server pool in case of policy managed databases but need to be done manually by DBA for Adminsitrator managed databaes.

Policy managed database is Dynamic database management, whereas Administrator managed databases are Static database management.

So, in case of Policy managed databaes, instances are associated to Server Pool, not to nodes, unlike Administrator managed databases.

## Associate an UNDO Tablespace with a particular RAC instance

```sql
alter system set undo_tablespace=UNDOTBS1 sid='RAC01';
alter system set undo_tablespace=UNDOTBS2 sid='RAC02';

-- Here, if you run above commands in CDB, UNDO tablespaces will be assigned to CDB but if you run inside PDB, these will be assigned to PDBs.
```

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
