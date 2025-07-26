# RAC Databases Overview and Architecture

## Types of Instances

There are three type of Oracle database instances:

* Single Instance databases
* RAC One Node databases
* RAC databases

### Single Instance database

These are Non-High Availability single instance databaeses.

### RAC One Node

* This is Single-Instance High Availability.
* These are two Active-Passive nodes/instances. At one time, there can be only 1 active node/instance but in case of node failure, database will failover to passive node. 
* Here we need to remember that, in this case node with Passive instance is lying idle and is a wastage of resource but is a lot cheaper than RAC database.
* It provides Online rolling patch application.
* It also provides Rolling Upgrades for the Operating System and Oracle Clusterware. We can do online database relocation between nodes for this any maintenance operation.
* SCAN allows clients to connect to the database, regardless of where the service is located.
* An Oracle RAC One Node database can be easily scaled up to a full Oracle RAC database if conditions demand it.
* Same like RAC database, it has Shared Everything Architecture.
* When we convert RAC One database to RAC database, we may need to change things like UNDO Tablespaces, etc., to manage multiple instances.
* UDP protocol is required for the cluster interconnect communication (private interconnect) on Linux/Unix platforms.

### RAC Database

* It is Active - Active configuration. There can be two or more nodes in a cluster.
* It provides *High Availability*, as if 1 node goes down, appliction will continue working from other node.
* It provides *Scalability*. If 1 instance can handle 100 transactions per minute, then 2 instances can handle 200 transactions.
* In RAC database, we can have separate workload for different departments or clients on different Instances using services but we can also share the workload among them.
* It is Pay as you grow architecture. We can scale the environment by adding more nodes is work load increases.
* If PARALLEL_FORCE_LOCAL parameter is set to TRUE, then parallel queries will run on the local node only but if it is set to FALSE, then SQL statement executed in parallel can run across all of the nodes in the cluster.

### Performance Overviews

Performance depends on a lot of things, not "just" RAC. Like:

* Gigabit, 10 Gitabit or Infinit Gigabit communication for Private Interconnect.
* Disk I/O for communication of RAC instances with Shared Storage.
* Number of CPUs on the Nodes.
* Application Design.
* Throughput support by Switches, Disk Controllers, etc. in use.
* Design the system to avoid any I/O bottleneck.

So, Performance Benefit is "possible" but we definitely do get the Scale Up benefit.

### To check Resource Utilization

```sql
select resource_name, current_utilization, max_utilization from v$resource_limit where resource_name like 'g%s_%';

select * from v$sgastat where name like 'g_s%' or name like 'KCL%';
```

## Background Processes Specific to Oracle RAC

* **ACMS** - Atomic Control File to Memory Service
* **GTX[0-j]** - Global Transaction Process
* **LMON** - Global Enqueue Service Monitor
* **LMD** - Global Enqueue Service Daemon
* **LMS** - Global Cache Service Process
* **LCK0** - Instance Enqueue Process
* **LMHB** - Global Cache/Enqueue Service Heartbeat Monitor
* **PING** - Interconnect Latency Measurement Process
* **RCBG** - Result Cache Background Process
* **RMSn** - Oracle RAC Management  Process
* **RSMN** - Remote Slave Monitor