# Oracle Grid Infrastructure and RAC Tidbits

This page contains miscellaneous but valuable pieces of information, quick tips, and observations concerning Oracle Grid Infrastructure and RAC. Each entry is designed to be a brief, standalone point.

## Oracle Grid Infrastructure

* We can have multiple Switches to manage multiple Private Interconnect networks to provide redundancy, which is called NIC bonding.

* Till Oracle 11.1, ASM was part of RAC software but from 11.2, ASM and Clusterware are part of Oracle Grid Infrastructure software.

* To store Oracle database files, we can use ASM and to use Non-Oracle database files, we can use OCFS (Oracle Cloud File System, previously known as ACFS) in shared file system.

* OCFS can also be used to store, data files, control files, online redo log files, etc.

* Another benefit of OCFS is that we can take snapshot of it and then snapshots of snapshots.
  * If we have a PDB database of say 10 TB, we can take a snapshot of this PDB, which will take seconds. This way we can clone our database very quickly, especially for our testing and QA databases.
  * OCFS has a feature of Tagging. So, we can set a group of directories or files as a logical group.
  * So, ACFS with DB Files, Tagging, Auditing, Encryption (TDE) and Snapshotting, is OCFS.

* We can install Oracle Clusterware in Standard Architecture or in Oracle Flex Architecture.

* If we want to consolidate Database Tier and Application Tier into one Oracle Cluster, then we can think of Flex Clusters.
  * Database runs on Hub Nodes and Applications run on Leaf Nodes.
  * Startup and Shutdown dependencies of Database and Application are also taken care by Flex Cluster, as dependencies can be defined here.
  * Leaf Nodes and Hub Nodes leverage HANFS, to ensure HA of storage.

* Data Files, OCR, Voting Disks, etc, all are on ASM or OCFS storage.

* The interconnect (Private Network) adapter must support UDP (User Datagram Protocol) or RDS (Reliable Data Socket) - **IMPORTANT**.

* In the Public Subnet Network (eth0), we'll see many VIPs and this subnet is shared by all of these services.
  * Node VIPs
  * SCAN VIPs
  * GNS VIPs
  * Apps VIPs
  * In Standard Cluster, GNS VIPs are optional but in Flex Clusters, it is mandatory.

* When using SCAN, client configuration looks like:
  * `tnsnames.ora:` app = cluster01-scan.domain, 1521, TCP, servicename.
  * Here SCAN (cluster01-scan.domain), is an alias or associated with VIPs node1-vip, node2-vip, node3-vip in DNS.
  * node1-vip, node2-vip and node3-vip are addresses of SCAN Listener.
  * In Oracle 11.1, when there was no SCAN, these VIPs pointed to local listeners and Oracle used Random Access based Load Balancing, so Nodes were unevenly loaded.
  * In case of SCAN, DNS uses round robin method to use these VIPs.
  * Now, whether we add or remove nodes from Cluster, as SCAN name remains same, client side configurations remain same too and addition or removal of nodes become transparent.
