---
title: MergerFS
taxonomy:
    category: docs
---

Instead of messing around with software or hardware RAID, one can use a union filesystem in order to join a bunch of disks in one big share. The main advantage of this over RAID is power consumption. MergerFS will only spin up the disk it writes to and not the whole array like RAID does. MergerFS does not provide any data protection like RAID 1,5,6 or 10 so if one of the disks fails you loose the data stored on that disk. The good news is the rest of your data will be safe, and paired with SnapRAID it represents a viable alternative to RAID. Luckily, mergerfs is easy to install.

Fist check that fuse is installed on your system as MergerFS depends on it.
```
debian :: ~ » apt-get install fuse
Reading package lists... Done
Building dependency tree
Reading state information... Done
fuse is already the newest version.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```
Next visit [MergerFS releases page](https://github.com/trapexit/mergerfs/releases) and download the latest version for desired architecture, at the moment of writing for Debian it is [mergerfs_2.12.3.debian-jessie_amd64.deb](https://github.com/trapexit/mergerfs/releases/download/2.12.3/mergerfs_2.12.3.debian-jessie_amd64.deb).

The install is as easy as invoking wget to download and dpkg to install.
```
debian :: ~ » wget https://github.com/trapexit/mergerfs/releases/download/2.12.3/mergerfs_2.12.3.debian-jessie_amd64.deb

mergerfs_2.12.3.debian-jessie_amd64.deb
100%[====================================================================================================================>]  79.21K  --.-KB/s   in 0.08s

2016-04-12 09:05:47 (1.03 MB/s) - ‘mergerfs_2.12.3.debian-jessie_amd64.deb’ saved [81112/81112]

debian :: ~ » dpkg -i mergerfs_2.12.3.debian-jessie_amd64.deb 
(Reading database ... 41457 files and directories currently installed.)
Preparing to unpack mergerfs_2.12.3.debian-jessie_amd64.deb ...
Unpacking mergerfs (2.12.3~debian-jessie) over (2.12.3~debian-jessie) ...
Setting up mergerfs (2.12.3~debian-jessie) ...
Processing triggers for man-db (2.7.0.2-5) ...
debian :: ~ »
```

Alternatively one can compile it from source:
```
debian :: ~ » cd
debian :: ~ » apt-get install g++ pkg-config git git-buildpackage pandoc debhelper libfuse-dev libattr1-dev -y
debian :: ~ » git clone https://github.com/trapexit/mergerfs.git 
debian :: ~ » cd mergerfs
debian :: ~/mergerfs ‹master› » make clean
debian :: ~/mergerfs ‹master› » make deb
```
Then simply install the compiled `.deb`
```
debian :: ~/mergerfs ‹master› » dpkg -i mergerfs*_amd64.deb
debian :: ~/mergerfs ‹master› » rm mergerfs*_amd64.deb mergerfs*_amd64.changes mergerfs*.dsc mergerfs*.tar.gz
debian :: ~ »
```
Let's now pool all the data disks setup for SnapRAID in the previous section. Open up `/etc/fstab` and add the following line (adjust the mount points)
```
# mergerfs setup for snapraid disks
/mnt/snapraid/data/*    /mnt/pool       fuse.mergerfs defaults,direct_io,allow_other,minfreespace=20G,fsname=mergerfsPool  0 0
```

This would pool all mounts in /mnt/snapraid/data and present them at /mnt/pool. This defaults to using the empfs mode and with the minfreespace option, the disks won't fill past 20GB remaining. fsname option is used so that `df -h` is short and usable.