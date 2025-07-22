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
# Check cluster services in table format
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
```

https://support.dbagenesis.com/oracle-rac/oracle-cluster-registry-ocr-administration
