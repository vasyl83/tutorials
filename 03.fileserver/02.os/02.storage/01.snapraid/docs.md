---
title: SnapRAID
taxonomy:
    category: docs
---

This is mostly a resume of a wonderfull writeup on the topic that Zack Reed did, you can find it [here](http://zackreed.me/articles/72-snapraid-on-ubuntu-12-04)

[SnapRAID](http://www.snapraid.it/) is a backup program for disk arrays. It stores parity information of your data and it recovers from up to six disk failures. 
SnapRAID is mainly targeted for a home media center, with a lot of big files that rarely change.

There is no `.deb` package for SnapRAID, so it must be compiled.

Install needed tools:
```
debian :: ~ » apt-get install gcc git make -y
```
Then download the most current version from [here](https://github.com/amadvance/snapraid/releases) version 10 is the latest at the moment of writing:
```
debian :: ~ » cd
debian :: ~ » wget https://github.com/amadvance/snapraid/releases/download/v10.0/snapraid-10.0.tar.gz
debian :: ~ » tar xzvf snapraid-10.0.tar.gz
debian :: ~ » cd snapraid-10.0/
debian :: ~/snapraid-10.0 » ./configure
debian :: ~/snapraid-10.0 » make
debian :: ~/snapraid-10.0 » make check
debian :: ~/snapraid-10.0 » make install
debian :: ~/snapraid-10.0 » cp ~/snapraid-10.0/snapraid.conf.example /etc/snapraid.conf
debian :: ~/snapraid-10.0 » cd ..
debian :: ~ »
```
Using disks and mount points created in the previous section, edit `/etc/snapraid.conf` to look something like this:
```
parity /mnt/snapraid/parity/parity1/snapraid.parity

content /var/snapraid.content
content /mnt/snapraid/data/disk1/snapraid.content
content /mnt/snapraid/data/disk2/snapraid.content
content /mnt/snapraid/data/disk3/snapraid.content
content /mnt/snapraid/data/disk4/snapraid.content
content /mnt/snapraid/data/disk5/snapraid.content
content /mnt/snapraid/data/disk6/snapraid.content

disk d1 /mnt/snapraid/data/disk1/
disk d2 /mnt/snapraid/data/disk2/
disk d3 /mnt/snapraid/data/disk3/
disk d4 /mnt/snapraid/data/disk4/
disk d5 /mnt/snapraid/data/disk5/
disk d5 /mnt/snapraid/data/disk6/

exclude *.bak
exclude *.unrecoverable
exclude /tmp/
exclude /lost+found/
exclude .AppleDouble
exclude ._AppleDouble
exclude .DS_Store
exclude .Thumbs.db
exclude .fseventsd
exclude .Spotlight-V100
exclude .TemporaryItems
exclude .Trashes
exclude .AppleDB

blocksize 256
```

Run `snapraid sync` to create the initial snapshot (warning this may take a while).

Finally use [this](http://zackreed.me/articles/83-updated-snapraid-sync-script) script to set up a cronjob (invoke `crontab with crontab -e`)as follows to scrub and sync at 4:30 am each day:
```
# Run a SnapRAID diff and then sync
30 4 * * * /root/scripts/snapraid_diff_n_sync.sh
```
A little note, if after running the script, debian complains about smartctl not being found, make sure smartmontools are installed (guide [here](../smartmontools)) and add the following line to the script:
```
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```