---
title: IPv6 and Tunnel Broker
taxonomy:
    category: docs
---
Bell in their infinite wisdom, still haven't deployed IPv6 to their customer base. To remedy this, let's setup a 6in4 tunnel so that our network can access IPv6 internet without anyhelp from Bell.

First, we need to register a free 6in4 tunnel with [Hurricane Electric](https://tunnelbroker.net/). Once that is done, the setup is quite easy. Simply go to Example Configurations in their interface and select Mikrotik. Execute the 3 commands and the tunnel is setup. All that is left to do is to instruct our network on how to use IPv6.

Here is the input from Hurricane Electric for my tunnel
```
/interface 6to4 add comment="Hurricane Electric IPv6 Tunnel Broker" disabled=no local-address=76.65.86.64 mtu=1280 name=sit1 remote-address=216.66.38.58
/ipv6 route add comment="" disabled=no distance=1 dst-address=2000::/3 gateway=2001:470:1c:1fe::1 scope=30 target-scope=10
/ipv6 address add address=2001:470:1c:1fe::2/64 advertise=no disabled=no eui-64=no interface=sit1
```
We need to assign IPv6 address to our bridge. Since the bridge will serve as main gateway as it does on IPv4 it is quite logical to assign the first ip from the routed IPv6 preffix
```
/ipv6 address
add address=2001:470:1d:1fe::1 interface=bridge-local
```
Next comes Neighbor Discovery and also a choice. The following block will instruct RouterOS to use dhcpv6 server, basically stateful configuration. Do no forget to set up dhcpv6 server on your network with this config.
```
/ipv6 nd
set [ find default=yes ] interface=bridge-local managed-address-configuration=yes other-configuration=yes
/ipv6 nd prefix default
set autonomous=no
```
For stateless configuration use the following.
```
/ipv6 nd
set [ find default=yes ] advertise-dns=yes interface=bridge-local managed-address-configuration=no other-configuration=no
/ipv6 nd prefix default
set autonomous=yes
```
Now that IPv6 is enabled and all the clients have access to internet through IPv6, the firewall must be setup and enabled. Here is a basic firewall config:
```
/ipv6 firewall filter
add chain=input comment="Allow ICMPv6" protocol=icmpv6
add chain=forward comment="Accept forward ICMPv6" protocol=icmpv6
add chain=input comment="Accept established connections" connection-state=\
    established
add chain=forward comment="Accept forwarded established connections" \
    connection-state=established
add chain=input comment="Accept related connections" connection-state=related
add chain=forward comment="Accept forwarded related connections" \
    connection-state=related
add chain=input comment="Accept UDP" protocol=udp
add chain=forward comment="Accept forward UDP" protocol=udp
add chain=input comment="Accept from LAN" in-interface=bridge-local
add chain=forward comment="Accept forwarded from LAN" in-interface=\
    bridge-local
add action=drop chain=input comment="Drop invalid connections" \
    connection-state=invalid
add action=drop chain=input comment="Drop everything else from outside"
add action=drop chain=forward comment="Drop invalid forward connections" \
    connection-state=invalid
add action=drop chain=forward comment="Drop anything else forwarded"
```