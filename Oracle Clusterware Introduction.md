# Oracle Clusterware 
A cluster consists of a group of independent but interconnected computers. A common cluster feature is that it should appear to an application as though it were a single server. Most cluster architectures use a dedicated network (interconnect) for communication and coordination between cluster nodes.

Oracle supports a shared disk cluster architecture providing load balancing and failover capabilities.

In an Oracle cluster, all nodes must share teh same processor architecture and run the same operating system. 

## Oracle Clusterware is:
* A key part of Oracle Grid Infrastructure
* Integrated with Oracle ASM (Automatic Storage Management)
* The basis for OCFS (Oracle Cloud File System)
* A foundation for Oracle RAC (Real Application Cluster)

Oracle Clusterware is also a foundation for the ASM Cluster File System (ACFS), a generalized Cluster File System, that can be used for most file based data such as documents, spreadsheets and reports.

Oracle Clusterware also manages resources, such as virtual IP (VIP) addresses, databases, listeners, services and so on.

## Clusterware Architecture and Clusterware Services
* Shared disk cluster architecture supporting application load balancing and failover
* Services include:
  - **Cluster management**, which allows cluster services and application resources to be monitored and managed from any node in the cluster 
  - **Node monitoring**, which provides real-time information regarding which nodes are currently available and the resources they support. Cluster integrity is also protected by evicting or fencing unresponsive nodes.
  - **Event services**, which publish cluster events so that applications are aware of changes in the cluster.
  - **Time synchronization**, which synchronizes the time on all nodes of the cluster.
  - **Network management**, which provisions and manages Virtual IP (VIP) addresses that are associated with cluster nodes or application resources to provide a consistent network identity regardless of which nodes are available. In addition, Grid Naming Service (GNS) manages network naming within the cluster.
  - **High availability**, which services, monitors, and restarts all other resources are required.
  - **Cluster Interconnect Link Aggregation (HAIP)**

## Oracle Clusterware Networking
* Each node must have at least two network adapters
* Each public network must support TCP/IP
* The interconnect (Private Network) adapter must support UDP (User Datagram Protocol) or RDS (Reliable Data Socket)
* All platforms use Grid Interprocess Communication (GIPc)

Each node must have at least two network adapters: one for the public network interface and the other for the private network interface or interconnect.

In addition, the interface names associated with network adapters for each network must be the same on all nodes. For example, in a two node cluster, you cannot configure network adapters on node1 with eth0 as the public interface, but on node2 have eth1 as the public interface. Public interface names must be the same, so you must configure eth0 as public on both nodes.
You should configure private interfaces on the same network adapters as well. If eth1 is the private interface for node1, eth1 should be the private interface for node2.

You can configure IP addresses with one of the following options:
* GNS (Oracle Grid Naming Service), using one static address defined during installation, which dynamically allocates VIP addresses using DHCP, which must be running on the network.
You must select the Advanced Oracle Clusterware installation option to use GNS.

* Static addresses that network administrators assign on DNS or each node. To use the Typical Oracle Clusterware installation option, yo must use static addresses.

Grid Interprocess Communication (GIPc) is used for Grid (Clusterware) interprocess communication. GIPc is a common communications infrastructure to replace CLSC/NS. It provides full control of the communicatoins stack from the operating system up to whatever client library uses it. GIPc can support multiple communications types: CLSC, TCP, UDP and IPC.

Use high-speed network adapters for the Interconnects and switches that support TCP/IP. Gigabit Ethernet or an equivalent is recommended.

If you have multiple available network interfaces, Oracle recommends that you use the Redundant Interconnect Usage feature to make use of multiple interfaces for the Private Network.

Note: Cross-over cables are not supported for use with Oracle Clusterware interconnects.

### Oracle Clusterware Initialization
The *Oracle High Avaialbility Service Daemon (OHASD) is a critical process in Oracle Clusterware, that manages high availability services.

In Oracle Linux 7, OHASD works as a systemd service.

### Single-Client Access Name (SCAN)
* The SCAN is the address used by clients connecting to the cluster.
* The SCAN is a fully qualified host name (host name + domain) registered to three IP addresses.

If you use GNS, and you have DHCP support, then the GNS will assign addresses dynamically to the SCAN.

If you do not use GNS, the SCAN should be defined in the DNS to resolve the three addresses assigned to that name. This should be done before we install Oracle Grid Infrastructure. 
The SCAN and its associated IP addresses provide a stanbel name for clients to use for connections, independent of the ndes that make up the cluster.

SCANs function like a cluster alias. However, SCANs are resolved on any node in the cluster. The SCAN addresses resolve to the cluster. Nodes can be added to or removed from the cluste rwithout affecting the SCAN address configuration.

During installation, listeners are created on each node for the SCAN IP addresses. Oracle Clusterware routes application requests to the cluster SCAN to the least loaded instance providing the service.

SCANs provide location independence for databases so that the client configuratoin does not have to depend on which nodes run a particular database.
The SCAN defaults to *clustername-scan.current_domain*, if a GNS domain is not specified at installation.