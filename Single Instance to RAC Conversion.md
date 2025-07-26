# Single Instance to RAC Conversion

* Single-Instance databases can be converted to RAC by using
  * DBCA
  * Enterprise Manager
  * RCONFIG utility
* DBCA automates most of the conversion tasks
* Before conversion, ensure that:
  * Your hardware and operating system are supported
  * Your Cluster Nodes have access to shared storage.

## Verify Database

```sql
select instance_name from v$instance;

select name from v$datafile;

select name from v$controlfile;
```

## Conversion of Single Instance DB to a RAC Instance on a new system

### Take backup using DBCA of your source database

* Run dbca on source node
* Check Manage Templates radio button
* Check `Create a database template` and `From an existing database (structure as well as data)` radio buttons
* Select database instance you want to convert
* Give Template Name. It'll create backup of database in .dfb format.
* Check `Maintain the file locations` for same directory structure or `Covert the file locations to use OFA structure` to use OMF.
* Review summary and click Finish to start the process.
* Now files with .ctl, .dbc and .dfb are created. .dbc file will contain the template and structure information. .dfb is actual backup file. .ctl is the control file.

### Start Conversion Process on destination database node

* Move backup files to Destination Node from Source Node.
* Copy .dbc file to $ORACLE_HOME/assistants/dbca/
* vi your .dbc file and provide location of .dfb file in "Location" field.
* Now, Run dbca on destination node.
* Check Create Database
* Check Advanced option
* In `Database Template` window, you'll see Template with the name of your file DBCA template backup file. Select it.
* Finish the process. It'll import database to RAC environment.
