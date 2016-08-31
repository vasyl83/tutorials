---
title: Router setup with Bell Fibe
taxonomy:
    category: docs
---
Connect to your router through ssh, telnet or use winbox terminal in order to paste those settings. Adjust as needed, some of these commands are not needed if the firmware was reset to defaults.

Set link speed, master ports, rename ether1 interface to wan and disable sfp:
```
/interface ethernet
set [ find default-name=ether2 ] speed=1Gbps
set [ find default-name=ether3 ] speed=1Gbps
set [ find default-name=ether4 ] speed=1Gbps
set [ find default-name=ether5 ] speed=1Gbps
set [ find default-name=ether7 ] master-port=ether6
set [ find default-name=ether8 ] master-port=ether6
set [ find default-name=ether9 ] master-port=ether6
set [ find default-name=ether10 ] master-port=ether6
set [ find default-name=sfp1 ] disabled=yes
set [ find default-name=ether1 ] name=wan speed=1Gbps
```
Join all the interfaces and wifi into a bridge
```
/interface bridge port
add bridge=bridge-local interface=ether6
add bridge=bridge-local interface=wlan1
add bridge=bridge-local interface=ether5
add bridge=bridge-local interface=ether2
add bridge=bridge-local interface=ether3
add bridge=bridge-local interface=ether4
```
Set bridge ip address and network
```
/ip address
add address=192.168.0.1/24 interface=bridge-local network=192.168.0.0
```
Set up dhcp (if needed)
```
/ip pool
add name=local-dhcp ranges=192.168.0.101-192.168.0.200
/ip dhcp-server
add add-arp=yes address-pool=local-dhcp \
    interface=bridge-local lease-time=1h name=default
/ip dhcp-server network
add address=192.168.0.0/24 dns-server=192.168.0.1 gateway=192.168.0.1 \
    netmask=24
```
Setup local dns-server
```
/ip dns
set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
```
Disable unneeded services
```
/ip service
set telnet address=192.168.0.0/24 disabled=yes
set ftp address=192.168.0.0/24 disabled=yes
set www address=192.168.0.0/24 disabled=yes
set api address=192.168.0.0/24 disabled=yes
set api-ssl address=192.168.0.0/24 disabled=yes
```
Create VLAN35 and attach it to wan
```
/interface vlan
add interface=wan name=internet vlan-id=35
```
Setup PPPoE over VLAN35 (don't forget to input your own username and password)
```
/interface pppoe-client
add add-default-route=yes default-route-distance=1 disabled=no interface=\
    internet max-mru=1490 max-mtu=1490 name=pppoe-bell password=********* \
    user=b1******
```
Not sure if it is really needed or not, disable neighbor discovery
```
/ip neighbor discovery
set internet discover=no
set pppoe-bell discover=no
```
Enable UPnP (if needed)
```
/ip upnp
set allow-disable-external-interface=yes enabled=yes
/ip upnp interfaces
add interface=bridge-local type=internal
add interface=pppoe-bell type=external
```
Set router name
```
/system identity
set name=router
```
Setup NTP (adjust timezone and ntp servers that correspond to your location)
```
/system clock
set time-zone-name=America/Toronto
/system logging
add topics=script
/system ntp client
set enabled=yes primary-ntp=198.50.239.55 secondary-ntp=208.73.56.29
/system ntp server
set broadcast=yes enabled=yes
```
Set up basic firewall
``` markup
/ip firewall filter
add chain=input comment="Accept established" connection-state=established
add chain=forward comment="Accept established internally" connection-state=\
    established
add chain=input comment="Accept related" connection-state=related
add chain=forward comment="Accept related internally" connection-state=\
    related
add chain=input comment="Allow Broadcast Traffic" dst-address-type=broadcast
add action=jump chain=input comment="jump to chain icmp" jump-target=icmp \
    protocol=icmp
add chain=icmp comment="Accept ping from TunnelBroker" protocol=icmp \
    src-address=66.220.2.74
add chain=icmp comment="0:0 and limit for 5pac/s" icmp-options=0 limit=\
    5,5:packet protocol=icmp
add chain=icmp comment="3:3 and limit for 5pac/s" icmp-options=3:3 limit=\
    5,5:packet protocol=icmp
add chain=icmp comment="3:4 and limit for 5pac/s" icmp-options=3:4 limit=\
    5,5:packet protocol=icmp
add chain=icmp comment="8:0 and limit for 5pac/s" icmp-options=8 limit=\
    5,5:packet protocol=icmp
add chain=icmp comment="11:0 and limit for 5pac/s" icmp-options=11 limit=\
    5,5:packet protocol=icmp
add action=drop chain=icmp comment="Drop everything else" protocol=icmp
add action=drop chain=input comment="Detect and drop port scan connections" \
    protocol=tcp psd=21,3s,3,1
add action=tarpit chain=input comment="Suppress DoS attack" connection-limit=\
    3,32 protocol=tcp src-address-list=black_list
add action=add-src-to-address-list address-list=black_list \
    address-list-timeout=1d chain=input comment="Detect DoS attack" \
    connection-limit=10,32 protocol=tcp
add action=drop chain=input comment="Drop invalid packets" connection-state=\
    invalid
add action=drop chain=forward comment="Drop invalid internal packets" \
    connection-state=invalid
add action=drop chain=input comment="Drop all packets recieved from internet \
    that doesn't meet any other rule" in-interface=pppoe-bell log-prefix=\
    "pppoe drop"

/ip firewall nat
add action=masquerade chain=srcnat comment="Allow NAT on main network" \
	out-interface=pppoe-bell src-address=192.168.0.0/24
```
Internet and local network should be working now.