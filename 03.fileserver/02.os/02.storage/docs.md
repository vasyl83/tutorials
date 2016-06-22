---
title: Storage configuration
taxonomy:
    category: docs
---
This part will setup storage in a way that is software and hardware (read mobo and raid card) independant. As Debian does by default, it is always better to mount all you disks by the UUID, in order to find your disk UUID just invoke `blkid`
```
debian :: ~ » blkid
/dev/sdb1: UUID="5a5955fe-cf29-4f94-aabb-b50c384aa559" TYPE="ext4" PARTUUID="79b1245e-90a0-4880-940d-564b2ada7b06"
/dev/sdc1: UUID="3376d7a0-5961-4a57-8603-515fb0933402" TYPE="ext4" PARTUUID="6a53a7f0-99d6-4441-aa7b-d157fd046ded"
/dev/sdd1: UUID="585A-AB7D" TYPE="vfat" PARTUUID="8ff03d3d-b0fd-4a1c-8712-e15fd7b84ec0"
/dev/sdd2: UUID="494571fe-13f8-4514-b99b-52e2918948af" TYPE="ext4" PARTUUID="247adb2c-b071-4b75-9967-afdd1ff2e4a1"
/dev/sdd3: UUID="3c397f62-6731-4219-9b62-77caf7060401" TYPE="swap" PARTUUID="5d1e0f73-e1d6-4359-b94c-7bd99024e55d"
/dev/sde1: UUID="4e92e8b5-2e89-49ff-9848-597cf41072b3" TYPE="ext4" PARTUUID="64410aeb-0c33-483d-889a-20e8ab0b78c8"
/dev/sdf1: UUID="27c2133f-ff01-4969-b350-cde6ac34428c" TYPE="ext4" PARTUUID="66e50028-7613-495d-b821-1ae65ade0f1f"
/dev/sdg1: UUID="e79e172f-3433-40bc-93c4-596bcda0e02f" TYPE="ext4" PARTUUID="50290fac-9e00-41b8-b2f5-6d4542cbcb24"
/dev/sdh1: UUID="0091acce-c54f-4d93-848a-9882ef1f1b91" TYPE="ext4" PARTUUID="10050d78-34ef-443c-a153-74fcda2c5911"
/dev/sdi1: UUID="abbd32b7-ae92-412a-a0ba-6c726f7fb196" TYPE="ext4" PARTUUID="d0a079ea-451d-4a5b-8cdc-57d2f7451a00"
```
You an get info on which disk is which with `lsblk`
```
debian :: ~ » lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdb      8:16   0   3.7T  0 disk
└─sdb1   8:17   0   3.7T  0 part /mnt/snapraid/data/disk4
sdc      8:32   0   3.7T  0 disk
└─sdc1   8:33   0   3.7T  0 part /mnt/snapraid/data/disk6
sdd      8:48   0 111.8G  0 disk
├─sdd1   8:49   0   512M  0 part /boot/efi
├─sdd2   8:50   0  99.6G  0 part /
└─sdd3   8:51   0  11.7G  0 part [SWAP]
sde      8:64   0   3.7T  0 disk
└─sde1   8:65   0   3.7T  0 part /mnt/snapraid/data/disk5
sdf      8:80   0   1.8T  0 disk
└─sdf1   8:81   0   1.8T  0 part /mnt/snapraid/data/disk1
sdg      8:96   0   1.8T  0 disk
└─sdg1   8:97   0   1.8T  0 part /mnt/snapraid/data/disk2
sdh      8:112  0   1.8T  0 disk
└─sdh1   8:113  0   1.8T  0 part /mnt/snapraid/data/disk3
sdi      8:128  0   3.7T  0 disk
└─sdi1   8:129  0   3.7T  0 part /mnt/storage
```
Once you have all the info gathered, create the folders for your mount points
```
mkdir -p /mnt/storage
mkdir -p /mnt/snapraid/data/{disk1,disk2,disk3,disk4,disk5,disk6}
mkdir -p /mnt/snapraid/parity/parity1
```
Now we need to format the disks. If there is no partition table use `cfdisk /dev/sda` (or sdb or sdc etc) and create a GPT then make a single partition, write changes to disk exit and repeat for every drive that isn't initialised. 

I manly use ext4 as file system, I don't need any fancy features and SnapRAID seems to like it. Setup a filesystem on each data disk with `mkfs.ext4 -m 2 -T largefile4 /dev/sdb1`(Note, -n 2 flag will reserve 2% of the disks space so that the parity overhead can fit on the parity disk and -T largefile4 will instruct mkfs that the partition will be used to large files). You can set the reserved space to 0% if your parity disk(s) are all larger than your data disks (i.e. you have 5TB parity disks and 4TB data disks). For the parity disk, simply format it with `mkfs.ext4 -T largefile4 /dev/sdf1` with no reserve.

Finally, adjut your `/etc/fstab` to look something like this:
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sdd2 during installation
UUID=494571fe-13f8-4514-b99b-52e2918948af /               ext4    errors=remount-ro,discard 0       1
# /boot/efi was on /dev/sdd1 during installation
UUID=585A-AB7D  /boot/efi       vfat    umask=0077      0       1
# swap was on /dev/sdd3 during installation
UUID=3c397f62-6731-4219-9b62-77caf7060401 none            swap    sw              0       0

# snapraid disks
UUID=27c2133f-ff01-4969-b350-cde6ac34428c /mnt/snapraid/data/disk1 ext4 defaults 0 2
UUID=e79e172f-3433-40bc-93c4-596bcda0e02f /mnt/snapraid/data/disk2 ext4 defaults 0 2
UUID=0091acce-c54f-4d93-848a-9882ef1f1b91 /mnt/snapraid/data/disk3 ext4 defaults 0 2
UUID=5a5955fe-cf29-4f94-aabb-b50c384aa559 /mnt/snapraid/data/disk4 ext4 defaults 0 2
UUID=4e92e8b5-2e89-49ff-9848-597cf41072b3 /mnt/snapraid/data/disk5 ext4 defaults 0 2
UUID=3376d7a0-5961-4a57-8603-515fb0933402 /mnt/snapraid/data/disk6 ext4 defaults 0 2

# snapraid parity
UUID=e03a853d-2354-4558-9ac7-7370b8b62512 /mnt/snapraid/parity/parity1 ext4 defaults 0 2

# other storage
UUID=abbd32b7-ae92-412a-a0ba-6c726f7fb196 /mnt/storage ext4 defaults 0 2
```
Later on a pool share will unite all the disks into one big share using [MergerFS](./mergerfs).