---
title: Basic configuration
taxonomy:
    category: docs
process:
	twig: true
---

####wbadmin.msc on Windows Server 2012r2 fails to start with error wbengine.exe 0XC000005

This error may cause backups to stop working. To correct it from an elevated command prompt run **wbadmin delete catalog**:
```
C:\Windows\system32>wbadmin delete catalog
wbadmin 1.0 - Backup command-line tool
(C) Copyright 2012 Microsoft Corporation. All rights reserved.

Are you sure that you want to delete the backup catalog? If you delete the
catalog, you will need to create a new set of backups.
[Y] Yes [N] No y

The backup catalog has been successfully deleted.

C:\Windows\system32>
```
Then recreate the backup job, as for the old backups they are still there, when you restore simply point Windows recovery to the correct disk/folder.

####How to change the default certificate used by Windows for remote sessions
It turns out that much of the configuration data for RDSH is stored in the Win32_TSGeneralSetting class in WMI in the root\cimv2\TerminalServices namespace. The configured certificate for a given connection is referenced by the Thumbprint value of that certificate on a property called SSLCertificateSHA1Hash.

In order to get the thumbprint value:

1. Open the properties dialog for your certificate and select the Details tab.
1. Scroll down to the Thumbprint field and copy the space delimited hex string into something like Notepad++.
1. Remove all the spaces from the string. You’ll also want to watch out for and remove a non-ascii character that sometimes gets copied just before the first character in the string.
1. This is the value you need to set in WMI. It should look something like this: 1ea1fd5b25b8c327be2c4e4852263efdb4d16af4.

Now with the thumbprint value, here’s a one-liner to set the value using wmic:
```
wmic /namespace:\\root\cimv2\TerminalServices PATH Win32_TSGeneralSetting Set SSLCertificateSHA1Hash="THUMBPRINT"
```
Or in PowerShell:
```
$path = (Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -Filter "TerminalName='RDP-tcp'").__path
Set-WmiInstance -Path $path -argument @{SSLCertificateSHA1Hash="THUMBPRINT"}
```