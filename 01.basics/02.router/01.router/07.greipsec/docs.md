---
title: GRE/IPsec with dynamic IPs
taxonomy:
    category: docs
---
Here is the network map of the setup.
![Network Map for GRE/IPsec](/images/gre_ipsec.png)
Make sure that DDNS is enabled and adjust ips as needed (especially if you use Docker as it uses 172.16.0.0/24)
```
/ip cloud set ddns-enabled=yes
```
For the rest of this post, lets assume that X is site 1 routerboard serial number and Y is site 2 routerboard serial number. Also any firewall rules get added to the bottom of the chain, so make sure to move them above any global drop rule you may have.

On Site 1:
```
:global remotesite [:resolve Y.sn.mynetname.net]
:global localsite [:resolve X.sn.mynetname.net]
/interface gre add name=myGre remote-address=$remotesite local-address=$localsite
/ip address add address=172.16.1.1/30 interface=myGre
/ip route add dst-address=192.168.1.0/24 gateway=172.16.1.2

/ip ipsec proposal set [ find default=yes ] auth-algorithms=md5,sha1 enc-algorithms=aes-128-cbc,twofish
/ip ipsec peer add address=$remotesite nat-traversal=no secret=letshavefunwithipsec
/ip ipsec policy add dst-address=192.168.1.0/24 sa-dst-address=$remotesite sa-src-address=$localsite src-address=192.168.0.0/24 tunnel=yes

/ip firewall filter add chain=forward comment="Allow all Site 2 to Site 1 traffic" dst-address=192.168.0.0/24 src-address=192.168.1.0/24
/ip firewall filter add chain=forward comment="Allow all Site 1 to Site 2 traffic" dst-address=192.168.1.0/24 src-address=192.168.0.0/24

/ip firewall nat add chain=srcnat dst-address=192.168.1.0/24 src-address=192.168.0.0/24

/system script add name=SetGRE owner=admin policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive source=":global\
    \_remotesite [:resolve Y.sn.mynetname.net]\r\
    \n:global localsite [:resolve X.sn.mynetname.net]\r\
    \n/int gre set myGre local-address=\$localsite remote-address=\$remotesite\
    \r\
    \n/ip ipsec policy set 1 sa-dst-address=\$remotesite sa-src-address=\$loca\
    lsite\r\
    \n/ip ipsec peer set 0 address=\$remotesite\r\
    \n:log info \"SetGREscript:Changing IP\""

/system scheduler add disabled=yes interval=1m name=SetGRE on-event="system script run SetGRE" policy=read,write,test start-time=startup

/system script add name=EnaSched_1 policy=read,write,reboot,ftp,policy,test,password,sniff,sensitive source="sys sched ena SetGRE"
/system script add name=DisaSched_1 policy=read,write,reboot,ftp,policy,test,password,sniff,sensitive source="sys sched disa SetGRE"

/tool netwatch add disabled=no down-script=EnaSched_1 host=192.168.1.1 interval=15s timeout=1s up-script=DisaSched_1
```
On Site 2:
```
:global remotesite [:resolve X.sn.mynetname.net]
:global localsite [:resolve Y.sn.mynetname.net]

/ip cloud set ddns-enabled=yes
/ip cloud force-update

/interface gre add name=myGre remote-address=$remotesite local-address=$localsite
/ip address add address=172.16.1.2/30 interface=myGre
/ip route add dst-address=192.168.0.0/24 gateway=172.16.1.1

/ip ipsec proposal set [ find default=yes ] auth-algorithms=md5,sha1 enc-algorithms=aes-128-cbc,twofish
/ip ipsec peer add address=$remotesite nat-traversal=no secret=letshavefunwithipsec
/ip ipsec policy add dst-address=192.168.0.0/24 sa-dst-address=$remotesite sa-src-address=$localsite src-address=192.168.1.0/24 tunnel=yes

/ip firewall nat add chain=srcnat dst-address=192.168.0.0/24 src-address=192.168.1.0/24

/ip firewall filter add chain=forward comment="Allow all Site 2 to Site 1 traffic" dst-address=192.168.0.0/24 src-address=192.168.1.0/24
/ip firewall filter add chain=forward comment="Allow all Site 1 to Site 2 traffic" dst-address=192.168.1.0/24 src-address=192.168.0.0/24

/system script add name=SetGRE owner=admin policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive source=":global\
    \_remotesite [:resolve X.sn.mynetname.net]\r\
    \n:global localsite [:resolve Y.sn.mynetname.net]\r\
    \n/int gre set myGre local-address=\$localsite remote-address=\$remotesite\
    \r\
    \n/ip ipsec policy set 1 sa-dst-address=\$remotesite sa-src-address=\$loca\
    lsite\r\
    \n/ip ipsec peer set 0 address=\$remotesite\r\
    \n:log info \"SetGREscript:Changing IP\""

/system scheduler add disabled=yes interval=1m name=SetGRE on-event="system script run SetGRE" policy=read,write,test start-time=startup

/system script add name=EnaSched_1 policy=read,write,reboot,ftp,policy,test,password,sniff,sensitive source="sys sched ena SetGRE"
/system script add name=DisaSched_1 policy=read,write,reboot,ftp,policy,test,password,sniff,sensitive source="sys sched disa SetGRE"

/tool netwatch add disabled=no down-script=EnaSched_1 host=192.168.0.1 interval=15s timeout=1s up-script=DisaSched_1
```