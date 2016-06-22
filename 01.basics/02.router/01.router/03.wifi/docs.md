---
title: Wifi and guest wifi
taxonomy:
    category: docs
---
Firstly, securing and customizing the main wifi. Change the name and channels to your preference.
```
/interface wireless
set [ find default-name=wlan1 ] band=2ghz-b/g/n channel-width=20/40mhz-eC \
    disabled=no frequency=2462 mode=ap-bridge ssid="Obi WLAN Kenobi" \
    wireless-protocol=802.11
```
For basic security let's enable wpa2. Don't forget to change your password (pre-shared-key)
```
/interface wireless security-profiles
set [ find default=yes ] authentication-types=wpa2-psk eap-methods="" mode=\
    dynamic-keys supplicant-identity=MikroTik wpa2-pre-shared-key=\
    "MySuperSecretPassword"
```
Moving on to guest wifi, start by creating a new wireless security profile, since it is a guest wifi we will leave it open to all.
```
/interface wireless security-profiles
add authentication-types=wpa-psk,wpa2-psk eap-methods="" name=passwordless supplicant-identity=""
```
Set up a virtual access point for the open wifi.
```
/interface wireless
add keepalive-frames=disabled master-interface=wlan1 multicast-buffering=disabled \
	name=ap-guest security-profile=passwordless ssid=\
    "Yell PENIS for password" wds-cost-range=0 wds-default-cost=0
```
Set an address for the new interface
```
/ip address
add address=192.168.10.1/24 interface=ap-guest network=192.168.10.0
```
Create a new ip pool for dhcp server
```
/ip pool
add name=ap-guest-pool ranges=192.168.10.2-192.168.10.254
```
Set up dhcp server using google dns
```
/ip dhcp-server
add address-pool=ap-guest-pool disabled=no interface=ap-guest name=ap-guest-dhcp
/ip dhcp-server network
add address=192.168.10.0/24 dns-server=8.8.8.8,8.8.4.4 gateway=192.168.10.1
```
In odrer to communicate with the outside additionnal NAT must be created
```
/ip firewall nat
add action=masquerade chain=srcnat comment="Allow NAT on guest wifi" out-interface=pppoe-bell src-address=192.168.10.0/2
```
Drop any communication between our main and guest network
```
/ip firewall filter
add action=drop chain=input comment="Drop all connections to/from guest wifi" dst-address=192.168.10.0/24 src-address=192.168.0.0/24
add action=drop chain=input comment="Drop all connections to/from guest wifi" dst-address=192.168.0.0/24 src-address=192.168.10.0/24
add action=drop chain=forward comment="Drop all connections to/from guest wifi" dst-address=192.168.10.0/24 src-address=192.168.0.0/24
add action=drop chain=forward comment="Drop all connections to/from guest wifi" dst-address=192.168.0.0/24 src-address=192.168.10.0/24
```
Finally, we limit guest wifi speed to 2Mbit/s up and down globally.
```
/queue simple
add max-limit=2M/2M name=guest-ap-queue target=ap-guest
```
