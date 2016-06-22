---
title: Basic IPv4 firewall
taxonomy:
    category: docs
---
#####Prep work
To start, it would be a good idea to either adapt the following rules or to simply flush all the existing rules and completely replace them with what follows. The easiest way to remove a rule is `/ip firewall filter print`, followed by: `/ip firewall filter remove numbers=X`. Where X is the line numbers from print command that you want to remove.


#####Port knocking
The advantage of port knocking is that without knowing which ports to access and in which sequence it is really hard to guess because the rules only give you a limited time to do it.

This is easily accomplished with 3 simple rules:
```
/ip firewall filter
add action=add-src-to-address-list address-list=knock address-list-timeout=15s chain=input comment="Set up portknocking rule 1" dst-port=1337 protocol=tcp
add action=add-src-to-address-list address-list=safe address-list-timeout=15m chain=input comment="Set up portknocking rule 2" dst-port=7331 protocol=tcp src-address-list=knock
add chain=input comment="Allow access to router from safe IP" src-address-list=safe
```

First line instructs our router to listen to any tcp packets on port 1337 and to add any connecting client into a temporary list (15 seconds to be exact) called knock. The second line listens for tcp packets on port 7331 checks them against knock list and if the IP is on knock list, adds it to safe list, the safe list lasts 15 minutes. Third line checks any incoming traffic against safe list and if the source IP is on safe list, lets it through. In layman’s terms, you connect to port 1337, then within 15 seconds to port 7331 and you got unrestricted access to the router and LAN from that IP for 15 minutes (after 15 minutes any on going connections will remain, you will simply loose the ability to initiate new ones before repeating the knock).
#####Established/related traffic
```
/ip firewall filter
add chain=input comment="Accept established" connection-state=established
add chain=forward comment="Accept established internally" connection-state=established
add chain=input comment="Accept related" connection-state=related
add chain=forward comment="Accept related internally" connection-state=related
```
#####Broadcast traffic
```
/ip firewall filter
add chain=input comment="Allow Broadcast Traffic" dst-address-type=broadcast
```
#####IPTV
These are the rules covered in [IPTV with Bell Fibe section](../iptv):
```
/ip firewall filter
add chain=input comment="IPTV traffic input" in-interface=iptv protocol=udp src-address=10.2.0.0/16
add chain=forward comment="IPTV traffic forward" in-interface=iptv out-interface=bridge-local protocol=udp src-address=10.2.0.0/16
add chain=input comment="IPTV traffic input" in-interface=iptv protocol=udp src-address=10.242.0.0/16
add chain=forward comment="IPTV traffic forward" in-interface=iptv out-interface=bridge-local protocol=udp src-address=10.242.0.0/16
add chain=input comment="IPTV traffic igmp accept" in-interface=iptv protocol=igmp
add action=drop chain=input comment="Drop all packetes recieved from iptv that doesn't meet any other rule" in-interface=iptv
```
#####ICMP and ping flood
```
/ip firewall filter
add action=jump chain=input comment="jump to chain ICMP" jump-target=ICMP protocol=icmp
add chain=ICMP comment="Accept ping from TunnelBroker" protocol=icmp src-address=66.220.2.74
add chain=ICMP comment="0:0 and limit for 5pac/s" icmp-options=0 limit=5,5 protocol=icmp
add chain=ICMP comment="3:3 and limit for 5pac/s" icmp-options=3:3 limit=5,5 protocol=icmp
add chain=ICMP comment="3:4 and limit for 5pac/s" icmp-options=3:4 limit=5,5 protocol=icmp
add chain=ICMP comment="8:0 and limit for 5pac/s" icmp-options=8 limit=5,5 protocol=icmp
add chain=ICMP comment="11:0 and limit for 5pac/s" icmp-options=11 limit=5,5 protocol=icmp
add action=drop chain=ICMP comment="Drop everything else" protocol=icmp
```
The first line creates a custom chain named ICMP into which all icmp traffic will be dropped. So logically everything coming in and is icmp will go in ICMP chain and the rules will apply, everything else will simply skip these rules and go to the next one. The second line allows all icmp traffic from Tunnel Broker (to make sure IPv6 tunnel works). Line 4 through 8 limit different icmp packets to 5 per second to prevent ping flood attacks. Line 9 instructs the firewall to drop everything icmp related that did not meet any previous rule in the ICMP chain.
#####Scanners and DDoS
```
/ip firewall filter
add action=drop chain=input comment="Detect and drop port scan connections" protocol=tcp psd=21,3s,3,1
add action=tarpit chain=input comment="Suppress DoS attack" connection-limit=3,32 protocol=tcp src-address-list=black_list
add action=add-src-to-address-list address-list=black_list address-list-timeout=1d chain=input comment="Detect DoS attack" connection-limit=10,32 protocol=tcp
```
These 3 lines help with scanners and DoS (denial of service) attacks. First line drops packets that are used by tools like nmap to scan your IP and report information on open pots, operating system, etc. The second line holds tcp connections and third line blacklists the IP that tries to flood you.
#####Drop everything else
```
/ip firewall filter
add action=drop chain=input comment="Drop invalid packets" connection-state=invalid
add action=drop chain=forward comment="Drop invalid internal packets" connection-state=invalid
add action=drop chain=input comment="Drop all packetes recieved from internet that doesn't meet any other rule" in-interface=pppoe-bell log=yes log-prefix="pppoe drop"
add action=drop chain=input comment="Drop all packetes recieved from iptv that doesn't meet any other rule" in-interface=iptv log=yes log-prefix="iptv drop"
```