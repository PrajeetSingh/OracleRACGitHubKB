# Useful commands for Oracle Grid Infrastructure and RAC

* To list all nodes in the cluster and their corresponding unique numeric IDs.

```sh
olsnodes -n
```

## Start or Stop Cluster

```sh
# Start clusterware (CRS + HAS) on the local node
crsctl start cluster
```

```sh
# Start clusterware on all nodes in the cluster.
crsctl start cluster -all
```
