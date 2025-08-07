# Oracle RAC and Multi-tenant Architecture

* In a MultiTenant architecture, there are:
  * Several instances per CDB
  * One UNDO tablespace per CDB instance
  * At least two groups of REDO log files per CDB instance

* Each container is exposed as a Service
  * The root
  * Each PDB

* Management of Services
  * Default database services
  * Policy managed databases
    * Singletons
    * Uniform across all servers in a server pool
