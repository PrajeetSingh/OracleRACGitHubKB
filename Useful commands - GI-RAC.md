# Useful commands for Oracle Grid Infrastructure and RAC

* To list all nodes in the cluster and their corresponding unique numeric IDs.

```sh
olsnodes -n
```

## Start or Stop Cluster

* Start clusterware (CRS + HAS) on the local node

```sh
crsctl start cluster
```

* Start clusterware on all nodes in the cluster.

```sh
crsctl start cluster -all
```
