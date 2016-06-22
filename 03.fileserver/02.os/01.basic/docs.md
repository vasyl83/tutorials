---
title: Basic configuration
taxonomy:
    category: docs
process:
	twig: true
---

#####Static IP
```
debian :: ~ » nano /etc/network/interfaces

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).
    
source /etc/network/interfaces.d/*
    
# The loopback network interface
auto lo
iface lo inet loopback
    
# The primary network interface
auto eth0
iface eth0 inet static
address 192.168.0.3
netmask 255.255.255.0
gateway 192.168.0.1
network 192.168.0.0
broadcast 192.168.0.255
dns-nameservers 192.168.0.2
dns-search ad.gontar.net
iface eth0 inet6 static
address 2001:470:1d:1fe::3
netmask 64
gateway 2001:470:1d:1fe::1
```
#####Nano highlighting
Create needed files and folders (must be done for each user, or):
```
debian :: ~ » mkdir -p ~/.nano/syntax
debian :: ~ » touch ~/.nano/syntax/syntax.nanorc
debian :: ~ » touch ~/.nanorc
```
Paste the following in `~/.nano/syntax/syntax.nanorc`:
```
# config file highlighting

syntax "conf" syntax "conf" "(\.(conf|config|cfg|cnf|rc|lst|list|defs|ini|desktop|mime|types|preset|cache|seat|service|htaccess)$|(^|/)(\w*crontab|mirrorlist|group|hosts|passwd|rpc|netconfig|shadow|fstab|inittab|inputrc|protocols|sudoers)$|conf.d/|.config/|nginx/)"

# default text
color magenta "^.*$"
# special values
icolor brightblue "(^|\s|=)(default|true|false|on|off|yes|no)(\s|$)"
# keys
icolor cyan "^\s*(set\s+)?[A-Z0-9_\/\.\%\@+-]+\s*([:]|\>)"
# commands
color blue "^\s*set\s+\<"
# punctuation
color cyan "[.]"
# numbers
color red "(^|\s|[[/:|<>(){}=,]|\])[-+]?[0-9](\.?[0-9])*%?($|\>)"
# keys
icolor cyan "^\s*(\$if )?([A-Z0-9_\/\.\%\@+-]|\s)+="
# punctuation
color blue "/"
color brightwhite "(\]|[()<>[{},;:=])"
color brightwhite "(^|\[|\{|\:)\s*-(\s|$)"
# section headings
icolor brightyellow "^\s*(\[([A-Z0-9_\.-]|\s)+\])+\s*$"
color brightcyan "^\s*((Sub)?Section\s*(=|\>)|End(Sub)?Section\s*$)"
color brightcyan "^\s*\$(end)?if(\s|$)"
# URLs
icolor green "\b(([A-Z]+://|www[.])[A-Z0-9/:#?&$=_\.\-]+)(\b|$| )"
# XML-like tags
icolor brightcyan "</?\w+((\s*\w+\s*=)?\s*("[^"]*"|'[^']*'|!?[A-Z0-9_:/]))*(\s*/)?>"
# strings
color yellow "\"(\\.|[^"])*\"" "'(\\.|[^'])*'"
# comments
color white "#.*$"
color blue "^\s*##.*$"
color white "^;.*$"
color white start="<!--" end="-->"
```
Append any files or directories in `syntax` section to enable highlighting in that file/location. For exemple the `|nginx/` at the end causes all the files in any nginx folder anywhere to be highlighted. 

Paste the following into `~/.nanorc`:
```
include ~/.nano/syntax/syntax.nanorc
```
#####Prefer IPv4 over IPv6 for aptitude.
Find the following line in `/etc/gai.conf` and uncomment it:
```
precedence ::ffff:0:0/96  100
```
```
