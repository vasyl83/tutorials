---
title: S.M.A.R.T tools
taxonomy:
    category: docs
---

####What is SMART?
Self-Monitoring, Analysis, and Reporting Technology (S.M.A.R.T.) is a supplementary component build into many modern storage devices through which devices monitor, store, and analyse the health of their operation.

####Installation
```
debian :: ~ » apt-get update && apt-get install smartmontools hddtemp
```

####Basic usage
Smartctl is a command line utility designed to perform SMART tasks such as printing the SMART self-test and error logs, enabling and disabling SMART automatic testing, and initiating device self-tests.

#####Scan and Try to Open Devices
```
debian :: ~ » smartctl --scan
/dev/sda -d scsi # /dev/sda, SCSI device
/dev/sdb -d scsi # /dev/sdb, SCSI device
/dev/sdc -d scsi # /dev/sdc, SCSI device
/dev/sdd -d scsi # /dev/sdd, SCSI device
/dev/sde -d scsi # /dev/sde, SCSI device
/dev/sdf -d scsi # /dev/sdf, SCSI device
/dev/sdg -d scsi # /dev/sdg, SCSI device
/dev/sdh -d scsi # /dev/sdh, SCSI device

debian :: ~ » smartctl --scan-open
/dev/sda -d sat # /dev/sda [SAT], ATA device
/dev/sdb -d sat # /dev/sdb [SAT], ATA device
/dev/sdc -d sat # /dev/sdc [SAT], ATA device
/dev/sdd -d sat # /dev/sdd [SAT], ATA device
/dev/sde -d sat # /dev/sde [SAT], ATA device
/dev/sdf -d sat # /dev/sdf [SAT], ATA device
/dev/sdg -d sat # /dev/sdg [SAT], ATA device
/dev/sdh -d sat # /dev/sdh [SAT], ATA device
```

#####Check for SMART support
```
debian :: ~ » smartctl -i /dev/sda | grep support
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

#####Show full device info
```
debian :: ~ » smartctl -i /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate NAS HDD
Device Model:     ST4000VN000-1H4168
Serial Number:    Z305RY74
LU WWN Device Id: 5 000c50 087cda214
Firmware Version: SC46
User Capacity:    4,000,787,030,016 bytes [4.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    5900 rpm
Form Factor:      3.5 inches
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ACS-2, ACS-3 T13/2161-D revision 3b
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Apr 19 08:42:59 2016 EDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

#####Show SMART Capabilities
```
debian :: ~ » smartctl -c /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
General SMART Values:
Offline data collection status:  (0x82) Offline data collection activity
                                        was completed without error.
                                        Auto Offline Data Collection: Enabled.
Self-test execution status:      (   0) The previous self-test routine completed
                                        without error or no self-test has ever
                                        been run.
Total time to complete Offline
data collection:                (  117) seconds.
Offline data collection
capabilities:                    (0x7b) SMART execute Offline immediate.
                                        Auto Offline data collection on/off support.
                                        Suspend Offline collection upon new
                                        command.
                                        Offline surface scan supported.
                                        Self-test supported.
                                        Conveyance Self-test supported.
                                        Selective Self-test supported.
SMART capabilities:            (0x0003) Saves SMART data before entering
                                        power-saving mode.
                                        Supports SMART auto save timer.
Error logging capability:        (0x01) Error logging supported.
                                        General Purpose Logging supported.
Short self-test routine
recommended polling time:        (   1) minutes.
Extended self-test routine
recommended polling time:        ( 503) minutes.
Conveyance self-test routine
recommended polling time:        (   2) minutes.
SCT capabilities:              (0x10bd) SCT Status supported.
                                        SCT Error Recovery Control supported.
                                        SCT Feature Control supported.
                                        SCT Data Table supported.
```
#####Show SMART Health Status
```
debian :: ~ » smartctl -H /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED
```
#####Show SMART errors
```
debian :: ~ » smartctl -l error /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Error Log Version: 1
No Errors Logged
```
#####Perform SMART short test
```
debian :: ~ » smartctl -t short /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Short self-test routine immediately in off-line mode".
Drive command "Execute SMART Short self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 1 minutes for test to complete.
Test will complete after Tue Apr 19 08:48:15 2016

Use smartctl -X to abort test.
```
#####Check SMART test results
```
debian :: ~ » smartctl -a /dev/sda
smartctl 6.4 2014-10-07 r4002 [x86_64-linux-3.16.0-4-amd64] (local build)
Copyright (C) 2002-14, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate NAS HDD
Device Model:     ST4000VN000-1H4168
Serial Number:    Z305RY74
LU WWN Device Id: 5 000c50 087cda214
Firmware Version: SC46
User Capacity:    4,000,787,030,016 bytes [4.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    5900 rpm
Form Factor:      3.5 inches
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ACS-2, ACS-3 T13/2161-D revision 3b
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Tue Apr 19 08:48:47 2016 EDT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x82) Offline data collection activity
                                        was completed without error.
                                        Auto Offline Data Collection: Enabled.
Self-test execution status:      (   0) The previous self-test routine completed
                                        without error or no self-test has ever
                                        been run.
Total time to complete Offline
data collection:                (  117) seconds.
Offline data collection
capabilities:                    (0x7b) SMART execute Offline immediate.
                                        Auto Offline data collection on/off support.
                                        Suspend Offline collection upon new
                                        command.
                                        Offline surface scan supported.
                                        Self-test supported.
                                        Conveyance Self-test supported.
                                        Selective Self-test supported.
SMART capabilities:            (0x0003) Saves SMART data before entering
                                        power-saving mode.
                                        Supports SMART auto save timer.
Error logging capability:        (0x01) Error logging supported.
                                        General Purpose Logging supported.
Short self-test routine
recommended polling time:        (   1) minutes.
Extended self-test routine
recommended polling time:        ( 503) minutes.
Conveyance self-test routine
recommended polling time:        (   2) minutes.
SCT capabilities:              (0x10bd) SCT Status supported.
                                        SCT Error Recovery Control supported.
                                        SCT Feature Control supported.
                                        SCT Data Table supported.

SMART Attributes Data Structure revision number: 10
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   117   099   006    Pre-fail  Always       -       157203384
  3 Spin_Up_Time            0x0003   092   091   000    Pre-fail  Always       -       0
  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always       -       13
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  7 Seek_Error_Rate         0x000f   100   253   030    Pre-fail  Always       -       695908
  9 Power_On_Hours          0x0032   100   100   000    Old_age   Always       -       776
 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always       -       13
184 End-to-End_Error        0x0032   100   100   099    Old_age   Always       -       0
187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always       -       0
188 Command_Timeout         0x0032   100   100   000    Old_age   Always       -       0
189 High_Fly_Writes         0x003a   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0022   071   068   045    Old_age   Always       -       29 (Min/Max 23/32)
191 G-Sense_Error_Rate      0x0032   100   100   000    Old_age   Always       -       0
192 Power-Off_Retract_Count 0x0032   100   100   000    Old_age   Always       -       9
193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always       -       14
194 Temperature_Celsius     0x0022   029   040   000    Old_age   Always       -       29 (0 21 0 0 0)
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0

SMART Error Log Version: 1
No Errors Logged

SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short offline       Completed without error       00%       776         -
# 2  Short offline       Completed without error       00%       771         -
# 3  Short offline       Completed without error       00%       746         -
# 4  Short offline       Completed without error       00%       722         -
# 5  Extended offline    Completed without error       00%       706         -
# 6  Short offline       Completed without error       00%       674         -
# 7  Short offline       Completed without error       00%       650         -
# 8  Short offline       Completed without error       00%       626         -
# 9  Short offline       Completed without error       00%       602         -
#10  Short offline       Completed without error       00%       579         -
#11  Short offline       Completed without error       00%       555         -
#12  Extended offline    Completed without error       00%       539         -
#13  Short offline       Completed without error       00%       507         -
#14  Short offline       Completed without error       00%       483         -
#15  Short offline       Completed without error       00%       459         -
#16  Short offline       Completed without error       00%       435         -
#17  Short offline       Completed without error       00%       411         -
#18  Short offline       Completed without error       00%       387         -
#19  Extended offline    Completed without error       00%       371         -
#20  Short offline       Completed without error       00%       361         -
#21  Short offline       Completed without error       00%       337         -

SMART Selective self-test log data structure revision number 1
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.
```
#####Configure SMART Disk Monitoring Daemon
First, edit `/etc/default/smartmontools` uncomment start_smartd=yes and add following options `smartd_opts="-q never -i 7200"`. -q instructs smartd to only stop if fatal error occurs and -i tells smartd how ofter to spin up disks in order to test them (usefull with `hdparm` when spinning disks down to save power).

Create a script to execute if any disk problems are detected `nano /root/scripts/smartd_mail.sh`:
```
#!/bin/bash

# Save the email message (STDIN) to a file:
cat > /root/smartdmsg

# Append the output of smartctl -a to the message:
/usr/sbin/smartctl -a -d $SMARTD_DEVICETYPE $SMARTD_DEVICE >> /root/smartdmsg

# Now email the message to the user at address ADD:
/usr/bin/mail -s "$SMARTD_SUBJECT" $SMARTD_ADDRESS < /root/smartdmsg
```
Make sure the paths defined are correct. Double-check if unsure:
```
debian :: ~ » which smartctl mail
/usr/sbin/smartctl
/usr/bin/mail
```
Make the script file executable with `chmod 0755 /root/scripts/smartd_mail.sh`

Replace the contents of `/etc/smartd.conf` with this one liner:
```
debian :: ~ » echo "DEVICESCAN -S on -o on -a -n standby,q -I 194 -W 4,35,40 -d sat -s (S/../.././02|L/../../6/03) -m root -M exec /root/scripts/smartd_mail.sh" > /etc/smartd.conf
```

This line enables monitoring on all the detected SAT hard drives and sets some sane options for self tests and reporting. For more infor refer to [smartd man page](https://www.smartmontools.org/browser/trunk/smartmontools/smartd.8.in).

Firanlly restart smartd with `systemctl restart smartd.service`

For more severly paranoid people you can also follow [this article](http://zackreed.me/articles/46-monitor-hard-disks-temperature-in-ubuntu-debian) to closer monitor HDD temperatures.