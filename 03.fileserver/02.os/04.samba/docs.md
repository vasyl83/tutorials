---
title: Samba and filesharing
taxonomy:
    category: docs
---
This section assumes that sssd and reamld were used to join AD domain. 

The following smb.conf uses the same nomenclature as the [AD section](../sso).

```
[global]
server string = File Server (Samba %v)
netbios name = DEBIAN

workgroup = GONTAR
security = ads
realm = GONTAR.CA

domain master = no
local master = no
prefered master = no
os level = 0

kerberos method = secrets and keytab

load printers = no
disable spoolss = yes
printing = bsd
printcap name = /dev/null


[storage]
        comment = 4Tb - downloads
        path = /mnt/storage
        browseable = yes
        writable = yes
        read only = no
        inherit acls = yes
        inherit permissions = yes
        create mask = 775
        directory mask = 775
        valid users = @"domain users"
        admin users = @"domain admins"
        hide files = /lost+found/

[pool]
        comment = SNAPraid pool
        path = /mnt/pool
        browseable = yes
        writable = yes
        read only = no
        inherit acls = yes
        inherit permissions = yes
        create mask = 775
        directory mask = 775
        valid users = @"domain users"
        admin users = @"domain admins"
        hide files = /lost+found/snapraid.content/

```
Request a new Kerberos ticket for Administrator if the one from [AD section](../sso) isn't valid anymore by issuing `kinit Administrator` and then instruct samba to join AD with `net ads join -k` (may not be required but I still do it just in case).

Restart samba services:
```
debian :: ~ » systemctl restart smbd.service
debian :: ~ » systemctl restart nmbd.service
```
All done.
