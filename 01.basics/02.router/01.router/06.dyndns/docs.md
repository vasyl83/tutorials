---
title: Dynamic DNS
taxonomy:
    category: docs
---
This section is useless for anyone with a static ip, but since residential Bell Fibe service does not include a static ip here is an easy way to freely configure a dynamic dns. RouetrOS has this feature built in.
```
/ip cloud
set ddns-enabled=yes
```
Now your router is reachable from outside with whatever dns-name the following input gives you (the first block is your RB serial number): `/ip cloud print`

As backup and a more robust solution, my suggestion is to use free DNS service provided by [Hurricane Electric](https://dns.he.net/) and a little script to regularly update any hosts set up on dns.he.net with DDNS flag enabled.

My script only works if you have same password for every DDNS entry (just make sure not to use the same password that you used to register with dns.he.net)

Here is the script as is for gontar.net
```
# Enter comma separated dns.he.net hosts that have DDNS active
:local DNShosts "gontar.net,gontar.ca"
#dns.he.net DDNS host password same one must be set for every host
:local DNSpass "MySuperSecretPassword"
#Tunnel Broker user that is used to log into the site
:local HEuser "MySuperSecretUser"
#Tunnel Broker password
:local HEpass "MySuperSecretPassword"
#Name of the tunnel as seen on main Tunnel Broker
:local HEhost "vasyl83-1.tunnel.tserv21.tor1.ipv6.he.net"
#Name of the 6in4 interface used for Tunnel Broker tunnel
:local HEtunnelinterface "sit1"
#WAN interface used to connect to the internet and having the IP you want to be set to your hosts
:local WANinterface "pppoe-bell"


#-----------------------------------------

:global previousIP;
:if ([/interface get $WANinterface value-name=running]) do={
	# Get the current IP on the interface
    :local currentIP [/ip address get [find interface="$WANinterface" disabled=no] address];
	# Strip the net mask off the IP address
    :for i from=( [:len $currentIP] - 1) to=0 do={
        :if ( [:pick $currentIP $i] = "/") do={ 
            :set currentIP [:pick $currentIP 0 $i]
        } 
    }
	:if ($currentIP != $previousIP) do={
        :log info "IPv6 and DNS update is needed"
        :set previousIP $currentIP
		
		# The update URL. Note the "\3F" is hex for question mark (?). Required since ? is a special character in commands.
		:local url "https://dyn.dns.he.net/nic/update\3Fmyip=$currentIP"
		:local dnshostarray;
		:set dnshostarray [:toarray $DNShosts];
		
		# Updating dns.he.net hosts
		:foreach host in=$dnshostarray do={
			:log info "dns.he.net: sending update for $host"
			/tool fetch url=($url . "&hostname=$host") user=$host password=$DNSpass mode=https dst-path=("dns.he.net-" . $host . ".txt")
			:log info "dns.he.net: host $host updated with IP $currentIP"
		}
		
		#Updating Tunnel Brocker IPv4 endpoint
		:log info "tunnelbroker: sending update for $HEhost"
		/tool fetch url=("https://ipv4.tunnelbroker.net/nic/update\3Fmyip=$currentIP" . "&hostname=$HEhost") user=$HEuser password=$HEpass mode=https dst-path=("tunnebroker.txt")
		:log info "tunnelbroker: $HEhost updated with IP $currentIP"
		
		#Updating 6in4 local-address
		/interface 6to4 {
			:if ([get ($HEtunnelinterface) local-address] != $currentIP) do={
			:log info ("Updating " . $HEtunnelinterface . " local-address with new IP " . $currentIP . "...")
			set ($HEtunnelinterface) local-address=$currentIP
				}
			}
		} 	else={
        :log info "dns.he.net: Previous IP $previousIP and current IP equal, no update need"
		:log info "tunnelbroker: Previous IP $previousIP and current IP equal, no update need"
    	}
} else={
    :log info "$inetinterface is not currently running, therefore will not update tunnelbroker nor dns.he.net"
}
```
Here it is formatted to RouterOS CLI entry
```
/system script
add name=he.net-update owner=admin policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive source="# Enter comma separated dns.he.net hosts that have DDNS\
    \_active\r\
    \n:local DNShosts \gontar.net,gontar.ca\"\r\
    \n#dns.he.net DDNS host password same one must be set for every host\r\
    \n:local DNSpass \"MySuperSecretPassword\"\r\
    \n#Tunnel Broker user that is used to log into the site\r\
    \n:local HEuser \"MySuperSecretUser\"\r\
    \n#Tunnel Broker password\r\
    \n:local HEpass \"MySuperSecretPassword\"\r\
    \n#Name of the tunnel as seen on main Tunnel Broker\r\
    \n:local HEhost \"vasyl83-1.tunnel.tserv21.tor1.ipv6.he.net\"\r\
    \n#Name of the 6in4 interface used for Tunnel Broker tunnel\r\
    \n:local HEtunnelinterface \"sit1\"\r\
    \n#WAN interface used to connect to the internet and having the IP you want to be set to your hosts\r\
    \n:local WANinterface \"pppoe-bell\"\r\
    \n\r\
    \n\r\
    \n#-----------------------------------------\r\
    \n\r\
    \n:global previousIP;\r\
    \n:if ([/interface get \$WANinterface value-name=running]) do={\r\
    \n\t# Get the current IP on the interface\r\
    \n    :local currentIP [/ip address get [find interface=\"\$WANinterface\" disabled=no] address];\r\
    \n\t# Strip the net mask off the IP address\r\
    \n    :for i from=( [:len \$currentIP] - 1) to=0 do={\r\
    \n        :if ( [:pick \$currentIP \$i] = \"/\") do={ \r\
    \n            :set currentIP [:pick \$currentIP 0 \$i]\r\
    \n        } \r\
    \n    }\r\
    \n\t:if (\$currentIP != \$previousIP) do={\r\
    \n        :log info \"IPv6 and DNS update is needed\"\r\
    \n        :set previousIP \$currentIP\r\
    \n\t\t\r\
    \n\t\t# The update URL. Note the \"\\3F\" is hex for question mark (\?). Required since \? is a special character in commands.\r\
    \n\t\t:local url \"https://dyn.dns.he.net/nic/update\\3Fmyip=\$currentIP\"\r\
    \n\t\t:local dnshostarray;\r\
    \n\t\t:set dnshostarray [:toarray \$DNShosts];\r\
    \n\t\t\r\
    \n\t\t# Updating dns.he.net hosts\r\
    \n\t\t:foreach host in=\$dnshostarray do={\r\
    \n\t\t\t:log info \"dns.he.net: sending update for \$host\"\r\
    \n\t\t\t/tool fetch url=(\$url . \"&hostname=\$host\") user=\$host password=\$DNSpass mode=https dst-path=(\"dns.he.net-\" . \$host . \".txt\")\r\
    \n\t\t\t:log info \"dns.he.net: host \$host updated with IP \$currentIP\"\r\
    \n\t\t}\r\
    \n\t\t\r\
    \n\t\t#Updating Tunnel Brocker IPv4 endpoint\r\
    \n\t\t:log info \"tunnelbroker: sending update for \$HEhost\"\r\
    \n\t\t/tool fetch url=(\"https://ipv4.tunnelbroker.net/nic/update\\3Fmyip=\$currentIP\" . \"&hostname=\$HEhost\") user=\$HEuser password=\$HEpass mode=https dst\
    -path=(\"tunnebroker.txt\")\r\
    \n\t\t:log info \"tunnelbroker: \$HEhost updated with IP \$currentIP\"\r\
    \n\t\t\r\
    \n\t\t#Updating 6in4 local-address\r\
    \n\t\t/interface 6to4 {\r\
	\n\t\t\t:if ([get (\$HEtunnelinterface) local-address] != \$currentIP) do={\r\
    \n\t\t\t:log info (\"Updating \" . \$HEtunnelinterface . \" local-address with new IP \" . \$currentIP . \"...\")\r\
    \n\t\t\tset (\$HEtunnelinterface) local-address=\$currentIP\r\
    \n\t\t\t\t}\r\
    \n\t\t\t}\r\
    \n\t\t} \telse={\r\
    \n        :log info \"dns.he.net: Previous IP \$previousIP and current IP equal, no update need\"\r\
    \n\t\t:log info \"tunnelbroker: Previous IP \$previousIP and current IP equal, no update need\"\r\
    \n    \t}\r\
    \n} else={\r\
    \n    :log info \"\$inetinterface is not currently running, therefore will not update tunnelbroker nor dns.he.net\"\r\
    \n}"
```
Another small script is needed in order to force periodic updates even if nothing changed, for the paranoid people mostly.
```
:global previousIP;
:set previousIP ""

:log info "Cleared previousIP to force IP update on next run."
```
Again, here it is in CLI format
```
/system script
add name=clearpreviousip owner=admin policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive source=\
    ":global previousIP;\r\
    \n:set previousIP \"\"\r\
    \n\r\
    \n:log info \"Cleared previousIP to force IP update on next run.\""
```
Finally, set the following schedules to run both scripts (adjust as needed)
```
/system scheduler
add interval=1h name="forceclear old ip" on-event="/system script run clearpreviousip" policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive start-time=startup
add interval=5m name=he.net-update on-event="/system script run he.net-update" policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive start-time=startup
```