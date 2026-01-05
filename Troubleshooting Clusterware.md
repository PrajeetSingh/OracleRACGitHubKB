# Troubleshooting Oracle Clusterware

* Working with Log Files
* Working with OCRDUMP
* Working with CLUVFY

## Working with Log Files

### Overview 

Here you examine the Oracle Clusterware alert log and then package various log files into an archive format suitable to send to My Oracle Support.

In oracle clusterware 12c (12.1.0.2), the location of alert log has been changed to $ORACLE_BASE/diag/crs/<hostname>/crs/trace/

12.1.0.2 standard cluster

* Name of alert log : alert.log
* location of alert log: $ORACLE_BASE/diag/crs/host01/crs/trace

1. Connect to host01 as the grid user, locate and view the contents of the Oracle Clusterware alert log.

```sh
[root@node1]$ . oraenv
ORACLE_SID = [grid] ? +ASM1
[root@node1 trace]# cd /u01/app/grid/diag/crs/node1/crs/trace/ 
[root@node1 trace]# tail -f alert.log
```

2. Navigate to the Oracle Cluster Synchronization Services daemon log directory and determine whether any log archives exist.

```sh
[grid@node1 trace]$ ls -ltr *.trc | tail -10

[grid@node1 trace]$ tail -5 crsd.trc

[grid@node1 trace]$ tail -5 ocssd.trc
```

3. Switch to the root user and set up the environment variables for the Grid Infrastructure. Change to the ORACLE_HOME/bin directory and run the diagcollection.pl script to gather all log files that can be sent to My Oracle Support for problem analysis.

```sh
[grid@node1 cssd]$ su - root

[root@node1]# . oraenv
ORACLE_SID = [root] ? +ASM1

[root@node1]# cd $ORACLE_HOME/bin

[root@node1 bin]# ./diagcollection.pl --collect $ORACLE_HOME
```

4. List the resulting log file archives that were generated with the diagcollection.pl script.
[root@node1 bin]# ls -la *.gz

## Working with OCRDUMP

### Overview

Work with the OCRDUMP utility and dump the binary file into both text and XML representations.

1. While connected to the grid account, dump the contents of the OCR to the standard output and count the number of lines of output.

```sh
[grid@node1]$ . oraenv
ORACLE_SID = [grid] ? +ASM1

[grid@node1]$ ocrdump -stdout | wc -l
```

2. Switch to the root user, dump the contents of the OCR to the standard output and count the number of lines of output. Compare your results with the previous step. How do the results differ?

```sh
[root@node1]# cd /u01/app/12.1.0.2/grid/bin/

[root@node1 bin]# ocrdump -stdout | wc -l
```

3. Dump the first 15 lines of the OCR to standard output using XML format

```sh
[root@node1 bin]# ocrdump -stdout -xml |head -15
```

4. Create an XML file dump of the OCR in the /tmp directory. Name the dump file ocr_current_dump.xml.

```sh
[root@node1 bin]# ocrdump -xml /tmp/ocr_current_dump.xml

[root@node1 bin]# ll /tmp/ocr*
-rw------- 1 root root 185642 Aug 17 17:26 ocr_current_dump.xml
```

5. Find the node and the directory that contains the automatic backup of the OCR from 24 hours ago.

```sh
[root@node1 bin]# ocrconfig –showbackup
```

6. Copy the 24-hour-old automatic backup of the OCR into the /tmp directory. This is not a dump, but rather an actual backup of the OCR. (If the daily backup of the OCR is not there, use the oldest backup on the list.)

Note: It may be necessary to use scp if the file is located on a different node.

```sh
[root@node1 bin]# cp /u01/app/12.1.0.2/grid/cdata/clust1/day.ocr /tmp/day.ocr
```

If you have day.ocr on node2 copy on node1 using SCP

```sh
[root@node1 bin]# scp node2:/u01/app/12.1.0.2/grid/cdata/cluster01/day.ocr 
/tmp/day.ocr 
```

7. Dump the contents of the day.ocr backup OCR file in the XML format saving the file in the /tmp directory. Name the file as day_ocr.xml.

```sh
[root@node1 bin]# ocrdump -xml -backupfile /tmp/day.ocr /tmp/day_ocr.xml
```

8. Compare the differences between the day_ocr.xml file and the ocr_current_dump.xml file to determine all changes made to the OCR in the last 24 hours. Log out as root when finished.

```sh
[root@node1 bin]# diff /tmp/day_ocr.xml /tmp/ocr_current_dump.xml | more
```

## Working with CLUVFY

### Overview 

Work with CLUVFY to verify the state of various cluster components.
