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

