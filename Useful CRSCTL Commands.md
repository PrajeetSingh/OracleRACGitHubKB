# Useful commands for Oracle Grid Infrastructure and RAC

## Start, Stop or Check Cluster

```sh
# Start/Stop/Check cluster (CRS) on the local node
crsctl start cluster
crsctl stop cluster
crsctl check cluster
```

```sh
# Start/Stop/Check cluster on all nodes in the cluster.
crsctl start cluster -all
crsctl stop cluster -all
crsctl check cluster -all
```

```sh
# Start/Stop/Check cluster on a remote node
crsctl start cluster -n rac1
crsctl stop cluster -n rac1
crsctl check cluster -n rac1
```

```sh
# Start/Stop/Check lower and upper stack (CRS + OHAS) on the local node
crsctl start crs
crsctl stop crs
crsctl check crs
```

```sh
# Start/Stop/Check HAS on current node
crsctl start has
crsctl stop has
crsctl check has
```

```sh
# Start/Stop/Check cluster services
crsctl start resource -all
crsctl stop resource -all

crsctl stat res -t
# or
crsctl status resource -t
```

```sh
# Check status of clusterware nodes / services
crsctl status server -f
```

```sh
# Enable or disable Auto Start of Cluster
crsctl enable has
crsctl disable has
```

```sh
# List all nodes in the cluster and their corresponding unique numeric IDs.
olsnodes -n

# List private interconnect address for the local node
olsnodes -l -p

# List VIP address with node name
olsnodes -i
olsnodes -i racnode1

# Check node status
olsnodes -s

# Check clusterware name
olsnodes -c
```

```sh
# Display Global public and global cluster_conterconnect
oifcfg getif
```

```sh
# Check software version
# Binary version
crsctl query crs softwareversion
# Active version
crsctl query crs activeversion
```

```sh
# Check Voting Disk
crsctl query css votedisk
```

## Oracle Clusterware Logs

All clusterware logs reside under

```sh
$GRID_HOME/log/<hostname>
```

Clusterware alert log

```sh
$GRID_HOME/log/<hostname>/alertracn1.log
```

Other process log files

```sh
$GRID_HOME/log/<hostname>/crsd         --> crs log
$GRID_HOME/log/<hostname>/ctssd        --> cts log
$GRID_HOME/log/<hostname>/evmd         --> evmd log
$GRID_HOME/log/<hostname>/ohasd        --> ohasd log
$GRID_HOME/log/<hostname>/diskmon      --> diskmon log
```

CRSD Oraagent logs

```sh
$GRID_HOME/log/<hostname>/agent/crsd/oragent_<crs admin username>
```

CRSD Ora root agent logs

```sh
$GRID_HOME/log/<hostname>/agent/crsd/orarootagent_<crs admin username>
```

<https://docs.oracle.com/en/database/oracle/oracle-database/19/cwadd/oracle-clusterware-control-crsctl-utility-reference.html#GUID-1A6774CF-FD5F-463E-817F-34E4466443C7>
