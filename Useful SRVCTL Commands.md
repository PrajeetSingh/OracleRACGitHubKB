# Startup and Shutdown RAC Instances

## Start & Stop RAC Databases

```sh
# Display the database registered in the repository
srvctl config database

# Display configuration details of database
srvctl config database -d RAC
```

```sh
# Start/Stop/Check database
srvctl status database -d RAC
srvctl start database -d RAC -o nomount / mount / open
srvctl stop database -d RAC -o immediate

srvctl status service -d RAC
```

```sh
# Start/Stop/Check RAC Instances
srvctl start instance -d RAC -i instancename
srvctl stop instance -d RAC -i instancename
srvctl status instance -d RAC -i instancename
```

```sh
# Start/Stop/Check ASM
srvctl status asm 
srvctl status asm -a
srvctl status asm -n racnode1 -a

srvctl start asm 
srvctl start asm -n racnode1
srvctl stop asm
srvctl stop asm -n racnode1 -f
```

```sh
# Start/Stop/Check Listener
srvctl start listener -n racnode1
srvctl stop listener -n racnode1
srvctl status listener -n racnode1
```

```sh
# Check Nodeapps, like VIP, Network, and ONS
srvctl status nodeapps
srvctl status nodeapps -n racnode1 
srvctl config nodeaspps
srvctl config nodeapps -n racnode2
```

```sh
# Check SCAN listener
srvctl status scan
srvctl config scan

srvctl status scan_listener
srvctl config scan_listener
```

```sh
# Check Oracle Service
srvctl config service -db RAC
```

`` https://valehagayev.wordpress.com/2016/04/16/useful-oracle-rac-commands/