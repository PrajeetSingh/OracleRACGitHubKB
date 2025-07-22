# Oracle Cluster Registry (OCR) Administration

## OCR Management

* Verify OCR

```sh
ocrcheck

# To see the contents of OCR
ocrdump ocr.bkp
ocrdump -xml /tmp/ocr.bkp
```

* Check OCR Backups

```sh
ocrconfig -showbackup auto

# Manual backup of OCR
ocrconfig -manualbackup

# To change OCR backup location
ocrconfig -backuploc <new_loc> 

# To restore OCR file from backup file
ocrconfig -restore <filename>
```

## OLR Management

```sh
# Default OLR location
cat /etc/oracle/olr.loc

# Default OLR configuration File
$GRID_HOME/cdata/hostname.olr

# Check OLR configuration file location using ocrcheck
ocrcheck -config local

# Check the contents of OLR
ocrdump -local /tmp/olr.bkp
ocrdump -local /xml/olr.bkp

# Manual backup of OLR
ocrconfig -local -manualbackup
```
