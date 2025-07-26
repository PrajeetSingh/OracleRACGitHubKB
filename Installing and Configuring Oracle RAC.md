# Installing and Configuring Oracle RAC

> **Prerequisites**
>
> GI must be installed before Oracle RAC installation and configuration.
>
> Ensure passwordless SSH communication between all nodes.

## Software Installation

* Download Database Software
* Run "runInstaller" from downloaded software
* Select "Install database software only"
* Select all Nodes on screen on which you want to install Oracle RAC software
* Select Software Installation Location
* Associate OS Groups with DBA groups. Like any member with OSDBA group, can login as DBA without providing username and password.
* Check `Perform Prerequisite Checks` screen, ensure all look good. Then click on Install button.
* At the end of the installation, you'll need to run root.sh script on all member nodes of Oracle RAC.

## Create the Cluster Database

Once Software is installed, we'll create Cluster database.

* Run dbca to create database
* Pick default or advanced option
* Configuration Type can be Policy Managed Database or Adminitrator Managed. For simplicity, pick Administrator Managed.
* Provide Global Database Name and check `Create as Container Database`
* Provide Server pool name. It a logical pool of servers.
* Check `Run Cluster Verfication Utility (CVU) Checks Periodically`
* Provide Database Administrative passwords for all accounts
* Select Disk Groups for `Database File Locations` and `Fast Recovery Area`. Check `Use Oracle-Managed Files` and specify size of `Fast Recovery Area Size`.
* Provide `Memory Size` for SGA and PGA, Processes and Character Set.
* Once all done, click Finish to create the database.

### Download and Install latest Oracle Patches

### Verify the Cluster Database configuration

```sh
srvctl config databae -db orcl

# You will find that it is a RAC database and Database is administrator managed.
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

## Verify DB

```sql
select instance_name from gv$instance;

select name from v$datafile;

select name from v$controlfile;
```
