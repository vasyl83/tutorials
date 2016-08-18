---
title: HTPC software
taxonomy:
    category: docs
---
All HTPC software I use is dockerized. I prefer using docker --run instead of docker --create because I like to use --restart parameter and it cannot be used in conjunction with --create.

The most important piece of software for HTPC is a media server. My server of choice is [Plex](https://plex.tv/). In order to use the docker container, here is how I have it set up:
```
docker run --name=plex -d \
--restart=always --net=host \
-e VERSION=latest -e PUID=108 -e PGID=113 \
-v /opt/docker/plex:/config \
-v /mnt/pool:/data linuxserver/plex
```
For the stats collection I always run [PlexPy](https://github.com/drzoidberg33/plexpy) here is the setup:
```
docker run --name=plexpy -d \
--restart=always -v /etc/localtime:/etc/localtime:ro \
-v /opt/docker/plexpy:/config \
-v /opt/docker/plex/Library/Application\ Support/Plex\ Media\ Server/Logs:/logs:ro \
-e PUID=108 -e PGID=113 -p 8181:8181 \
linuxserver/plexpy
```
The /logs folder must be mapped to the location of [Plex logs folder](https://support.plex.tv/hc/en-us/articles/200250417-Plex-Media-Server-Log-Files). If you have several Plex servers simply snip up another PlexPy container and point it to a different port and a different /logs folder location. You can even mount a cifs share and have PlexPy poll Plex server from a windows machine (here is an example of fstab for read only cifs share:`//MACMINI/plexlogs /opt/docker/plexpy/plexlogs cifs domain=GONTAR,username=vasyl,password=mysupersecretpassword,iocharset=utf8,sec=ntlm,ro  0  0`).

Next PlexRequests, so that our Plex users can request new content.
```
docker run --name=plexrequests -d \
--restart=always -v /etc/localtime:/etc/localtime:ro \
-v /opt/docker/plexrequests:/config \
-e PUID=108 -e PGID=113 -e URL_BASE=/plexrequests \
-p 3000:3000 lsiodev/plexrequests
```

Now that media sharing, requesting and logging is done, time to automate tv and movie downloads.

First, [Deluge](http://deluge-torrent.org/) for torrents:
```
docker run --name=deluge -d \
--restart=always --net=host \ 
-e PUID=108 -e PGID=113 \
-v /mnt/storage/downloads:/downloads \
-v /opt/docker/deluge:/config \
-v /etc/localtime:/etc/localtime:ro \
linuxserver/deluge
```
Next, [SabNZBd](http://sabnzbd.org/) for usenet:
```
docker run --name=sabnzbd -d \
--restart=always -e PUID=108 -e PGID=113 \
-v /opt/docker/sabnzbd:/config \
-v /mnt/storage/downloads:/downloads \
-v /mnt/storage/incomplete:/incomplete-downloads \
-v /etc/localtime:/etc/localtime:ro \
-p 8080:8080 linuxserver/sabnzbd
```
Now that the downloaders are working time to install [SickRage](https://github.com/SickRage/SickRage/graphs/contributors) for tv shows automation and [Couchpotato](https://couchpota.to/) for movie automation.

For SickRage:
```
docker run --name=sickrage -d \
--restart=always -v /etc/localtime:/etc/localtime:ro \
-v /opt/docker/sickrage:/config \
-v /mnt/storage/downloads/tv:/downloads \
-v /mnt/pool/tv:/tv -e PUID=108 -e PGID=113 \
-p 8081:8081 linuxserver/sickrage
```
For Couchpotato:
```
docker run --name=couchpotato -d \
--restart=always -v /etc/localtime:/etc/localtime:ro \
-v /opt/docker/couchpotato:/config \
-v /mnt/storage/downloads/movies:/downloads \
-v /mnt/pool/movies:/movies \
-e PUID=108 -e PGID=113 -p 5050:5050 \
linuxserver/couchpotato
```

Lastly, as you can see SickRage and Couchpotato use subfolders inside `/mnt/storage/downloads` to make usre that the content gets put into appropriate folder they are set up to label everything they download and then SabNZBd and Deluge are set up in a way that labeled content once completed is moved to the correct folder.