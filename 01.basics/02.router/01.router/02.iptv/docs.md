---
title: IPTV with Bell Fibe
taxonomy:
    category: docs
---
Setting up IPTV with Bell Fibe on RouterOS is extremely simple.

ITVP uses VLAN36, we need to create it
```
/interface vlan
add interface=wan name=iptv vlan-id=36
```
Disable neighbor discovery
```
/ip neighbor discovery
set iptv discover=no
```
Setup dhcp client for newly created VLAN36
```
/ip dhcp-client
add add-default-route=no dhcp-options=hostname,clientid disabled=no \
    interface=iptv use-peer-ntp=no
```
Add needed firewall and nat rules (make sure that they are placed correctly, especyally if you used my firewall rules, input and forward chains should always be before all the drop rules)
```
/ip firewall filter
add chain=input comment="IPTV traffic input" in-interface=iptv protocol=udp \
    src-address=10.2.0.0/16
add chain=forward comment="IPTV traffic forward" in-interface=iptv \
    out-interface=bridge-local protocol=udp src-address=10.2.0.0/16
add chain=input comment="IPTV traffic input" in-interface=iptv protocol=udp \
    src-address=10.242.0.0/16
add chain=forward comment="IPTV traffic forward" in-interface=iptv \
    out-interface=bridge-local protocol=udp src-address=10.242.0.0/16
add chain=input comment="IPTV traffic igmp accept" in-interface=iptv \
    protocol=igmp
add action=drop chain=input comment=\
    "Drop all packetes recieved from iptv that doesn't meet any other rule" \
    in-interface=iptv
```
```
/ip firewall nat
add action=masquerade chain=srcnat comment="Allow NAT on iptv" out-interface=\
    iptv
```
The following routes are not strictly necessary, but without them I get errors when Im watching tv. The gateway should be the same as the one that pppoe client gets assigned to it, to check use `/ip dhcp-client print detail`
```
/ip route
add distance=1 dst-address=10.2.0.0/16 gateway=10.242.160.1
add distance=1 dst-address=10.242.0.0/16 gateway=10.242.160.1
```
Lastly, IGMP Proxy must be installed and configured. By default multicast package is not installed, so you need to go to [Mikrotik Downloads page](http://www.mikrotik.com/download) and get Extra Packages for your architecture (you only need routing-x.xx.x-architecture.npk), you can check if the package is installed with `/system package print`. If it isnt installed, download the package copy it to your router and enable it by running `/system package enable multicast` (reboot may be necessary).

Configure IGMP Proxy.
```
/routing igmp-proxy
set query-interval=1m
/routing igmp-proxy interface
add alternative-subnets=10.0.0.0/8,239.0.0.0/8,224.0.0.0/8 interface=iptv \
    upstream=yes
add interface=bridge-local
```
Since most Router Boards only have 2.4 GHz wifi and it seems to get killed by multicast (read IPTV), it is necessary to drop multicast packets from wifi.
```
/interface bridge filter
add action=drop chain=output out-interface=wlan1 packet-type=multicast
```
It is also wise to drop multicast from all the interfaces where iptv packets do not belong. Presuming your STB is connected to ether3 and ether7 to 10 are slaves to ether6 and nothing else is connected to bridge-local. This may also break some aspects of network discovery for your windows machines.
```
/interface bridge filter
add action=drop chain=output out-interface=ether6 packet-type=multicast
add action=drop chain=output out-interface=ether5 packet-type=multicast
add action=drop chain=output out-interface=ether2 packet-type=multicast
add action=drop chain=output out-interface=ether3 packet-type=multicast
```
TV should be fully operational now, even the apps. It woulde be prudent to fully isolate IPTV from ther rest of your network, it can easily be accomplished with VLANs and will be covered in later sections.