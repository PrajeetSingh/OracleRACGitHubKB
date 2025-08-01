# Managing High Availability of Services

To manage workloads or a group of applications, we can define services for a particular application or a subset of an application's operations.

We can also group work by Type under services. For example, OLTP users can use one service while batch processing can use another to connect to the database.

Use `srvctl` to manage services, not DBMS_SERVICE.

> By default, in an RAC environment, a SQL statement run in parallel can run across all of the nodes in the cluster.
> We can control parallel execution in a RAC environment with the `PARALLEL_FORCE_LOCAL` initialization parameter. The parallel queries will execute on the same Oracle RAC Node where the SQL statement was started.

**Restricted Service Registration** allows listener registration only from local IP addresses, by default.

## Creating Services with `srvctl`

To create a service called `srvc1` with preferred instance RAC02 and an available instance RAC01.

```sh
srvctl add service -db RACDB -service SRVC1 -preferred RAC02 -available RAC01
```

Here, service `srvc1` will work on node RAC02 but if it fails, the service will failover to RAC01.

>Note: When services are created with `srvctl`, tnsnames.ora is not updated and the service is not started.
