---
title: Samba and filesharing
taxonomy:
    category: docs
---
This section assumes that sssd and reamld were used to join AD domain. 

The following smb.conf uses the same nomenclature as the [AD section](../sso).

```
[global]
server string = file server
netbios name = FILE-SRV

workgroup = GONTAR
realm = GONTAR.CA
security = ADS
server role = member server
encrypt passwords = yes
passdb backend = tdbsam
server signing = auto
client signing = auto

domain master = no
local master = no
prefered master = no
os level = 0

kerberos method = secrets and keytab

load printers = yes
#disable spoolss = yes
printing = cups
printcap name = cups

admin users = root @"domain admins"

[example]
        comment = example share
        path = /mnt/example
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
        hide unreadable = yes
        #hide unwriteable files = yes
        access based share enum = yes

[printers]
        comment=All Printers
        path=/var/spool/samba
        browseable=yes
        guest ok=no
        writable=no
        printable=yes
        create mode=0700
        write list=vasyl @"domain users"

[print$]
        path = /opt/samba/printer_drivers
        comment = Printer drivers
        write list=vasyl @"domain users"
        browseable = yes
        writeable = yes

```
Request a new Kerberos ticket for Administrator if the one from [AD section](../sso) isn't valid anymore by issuing `kinit administrator` and then instruct samba to join AD with `net ads join -k` (may not be required but I still do it just in case).

Restart samba services:
```
# systemctl restart smbd.service nmbd.service
```

All done.
