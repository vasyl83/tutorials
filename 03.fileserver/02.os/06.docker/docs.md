---
title: Docker
taxonomy:
    category: docs
---
Docker is convenient way to run applications in containers separate from your host OS. Docker is not a replacement for virtual machines or all local applications, but it can help modularize your system and keep your host OS neat and tidy. Running apps in containers keeps things like the Mono libraries, Ruby dependences (and gems), Java, etc. from being stored on host OS. It make porting these containers to a new system and backups very easy.

####Installation
Simply follow [the official docs](https://docs.docker.com/engine/installation/linux/debian/)

####Configuration
Create a new user for Docker (it is not strictly necessary, but it is cleaner this way and limits who can access container files saved outside the container, it is also possible your actual user) `adduser --system  --gecos "Docker User for containers" --disabled-password --group --home /opt/docker docker`. This command will not only create the user but will create a home folder for it directly under /. Now whenever you --create or --run a container you can centralise all the external data in one folder. 

If it is the first system user created on Debian it should have UID 108 and GID 113. Check UID and GID with:
```
» id docker
uid=108(docker) gid=113(docker) groups=113(docker)
```
Now all is left is to start the containers with the apps we want. The individual apps will be covered in [HTPC Software](../htpc), [Remote backup](../backup/remote) and in [Chapter 4](../../../webserver).

Here is the global list:

|Container|Function|
|----|----|
|[linuxserver/deluge](https://hub.docker.com/r/linuxserver/deluge/)|Deluge bittorrent client|
|[linuxserver/plex](https://hub.docker.com/r/linuxserver/plex/)|Plex media server|
|[linuxserver/couchpotato](https://hub.docker.com/r/linuxserver/couchpotato/)|Automatic video library manager for movies|
|[linuxserver/sickrage](https://hub.docker.com/r/linuxserver/sickrage/)|Automatic video library manager for tv shows|
|[linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr/)|Sonarr (formerly NZBdrone) is a more modern looking alternative to SickRage, il still lacks some of Sickrage features but its modern UI more than makes up for it|
|[linuxserver/plexpy](https://hub.docker.com/r/linuxserver/plexpy/)|Plex usage tracker|
|[lsiodev/plexrequests](https://hub.docker.com/r/lsiodev/plexrequests/)|Automated way for users to request new content for Plex, works with SickRage and Couchpotato|
|[linuxserver/mariadb](https://hub.docker.com/r/linuxserver/mariadb/)|MariaDB database|
|[glyptodon/guacamole](https://hub.docker.com/r/glyptodon/guacamole/)|Guacamole - HTML5 clientless remote desktop gateway|
|[glyptodon/guacd](https://hub.docker.com/r/glyptodon/guacd/)|Guacamole - HTML5 clientless remote desktop daemon|
|[linuxserver/nginx](https://hub.docker.com/r/linuxserver/nginx/)|Nginx webserver with SSL and PHP|
|[linuxserver/sabnzbd](https://hub.docker.com/r/linuxserver/sabnzbd/)|SabNZBd open source binary newsreader|
|[linuxserver/muximux](https://hub.docker.com/r/linuxserver/muximux/)|Lightweight portal to view & manage your HTPC|
|[linuxserver/gsm-ts3](https://hub.docker.com/r/linuxserver/gsm-ts3/)|Teamspeak server|
|[jrcs/crashplan](https://hub.docker.com/r/jrcs/crashplan/)|Crashplan backup|
|[zuhkov/observium](https://hub.docker.com/r/zuhkov/observium/)|Observium snmp monitoring suite|
|[linuxserver/jackett](https://hub.docker.com/r/linuxserver/jackett/)|Jackett works as a proxy server: it translates queries from apps (Sonarr, SickRage, CouchPotato, Mylar, etc) into tracker-site-specific http queries|
|[linuxserver/syncthing](https://hub.docker.com/r/linuxserver/syncthing/)|An open source sync agent|
|[linuxserver/pydio](https://hub.docker.com/r/linuxserver/pydio/)|Pydio is a mature open source software solution for file sharing and synchronization|
|[linuxserver/openvpn-as](https://hub.docker.com/r/linuxserver/openvpn-as/)|OpenVPN Access Server is a full featured secure network tunneling VPN software solution|
|[linuxserver/codiad](https://hub.docker.com/r/linuxserver/codiad/)|Codiad is a web-based IDE framework|
|[hurricane/ubooquity](https://hub.docker.com/r/hurricane/ubooquity/)|Ubooquity is a free, lightweight and easy-to-use home server for your comics and ebooks|