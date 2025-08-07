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
# or
srvctl add service -d RACDB -s SRVC1 -p RAC02 -available RAC01
```

Here, service `srvc1` will work on node RAC02 but if it fails, the service will failover to RAC01.

>Note: When services are created with `srvctl`, tnsnames.ora is not updated and the service is not started.

## Managing Services with `srvctl`

* Start / Stop a named service on all configured instances

```sh
srvctl start service -db RACDB -service srvc
srvctl stop service -db RACDB -service srvc
srvctl status service -db RACDB -service srvc
# or
srvctl start service -d RACDB -s srvc
srvctl stop service -d RACDB -s srvc
srvctl status service -d RACDB -s srvc
```

```sql
sqlplus /@cluster01-scan.net:1521/srvc as sysdba
select instance_name from v$instance;
show parameter service_names
```

* Disable/Enable a service at a named instance

```sh
srvctl disable service -db RACDB -service srvc -instance RAC01
# or
srvctl enable service -d RACDB -s srvc -i RAC01
```

## Services and Transparent Application Failover

* We can define TAF policy for a service and all connections using this service will automatically have TAF enabled.

* The TAF setting on a service can be NONE, BASIC or PRECONNECT and overrides any TAF setting in the client definition.

>Note: TAF applies only to an admin-managed database, not to policy-managed database

## Using Services with the Scheduler

* Services are associatd with Scheduler classes
* Scheduler jobs have service affinity
  * High Availability
  * Load Balancing

Just as in other environments, the Scheduler in a RAC environment uses one job table for each database and one job coordinator (CJQ0 process) for each instance. The job coordinators communicate with each other to keep information current.

The Schduler can use the services and the benefits they offer in a RAC environment. The service that a specific job class uses is defined when the job class is created.
During execution, jobs are assigned to job classes and job classes run within services.
Using services with job classes ensures that the work of the Scheduler is identified for workload management and performance tuning.
For example, jobs inherit server generated alerts and performance thresholds for the servie they under.

For high availability, the Scheduler offers service affinity instead of instance affinity. Jobs are not scheduled to run on a specific instance. They are scheduled to run under a service.
