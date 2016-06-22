---
title: Joining Active Directory
taxonomy:
    category: docs
---
In this example, Active Directory domain is ad.gontar.net and the domain controller is pdc.ad.gontar.net, Linux box hostname is debian. sssd and realmd will be used to join AD.

Make sure the linux box uses AD DNS:

```
debian :: ~ » dig -t SRV _ldap._tcp.gontar.ca | grep -A2 "ANSWER SECTION"

;; ANSWER SECTION:
_ldap._tcp.gontar.ca. 600   IN      SRV     0 100 389 dc-srv.gontar.ca.
```

Install sssd and realmd:
```
debian :: ~ » apt-get -y install realmd sssd sssd-tools samba-common krb5-user packagekit samba-common-bin samba-libs adcli ntp libpam-sss libnss-sss
```

When prompted, type in your AD Kerberos realm. It should generally be your domain name in capital letters (“ad.gontar.net” becomes “AD.GONTAR.NET”). If your DNS is working properly, that should be all that is needed for the Kerberos client to work. If no promt appears during install, simply invoke dpkg: `$ sudo dpkg-reconfigure krb5-config`

Otherwise you may need to add your servers to **/etc/krb5.conf** here is what it should look like:

```
[libdefaults]
        default_realm = GONTAR.CA
        dns_lookup_realm = true
        dns_lookup_kdc = true

        krb4_config = /etc/krb.conf
        krb4_realms = /etc/krb.realms
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true

# The following libdefaults parameters are only for Heimdal Kerberos.
        v4_instance_resolve = false
        v4_name_convert = {
                host = {
                        rcmd = host
                        ftp = ftp
                }
                plain = {
                        something = something-else
                }
        }
        fcc-mit-ticketflags = true

[realms]
        GONTAR.CA = {
                kdc = dc-srv.gontar.ca
                admin_server = dc-srv.gontar.ca
                default_domain = gontar.ca
        }

[domain_realm]
        gontar.ca = GONTAR.CA
        .gontar.ca = GONTAR.CA

[login]
        krb4_convert = true
        krb4_get_tickets = false
```
In an Active Directory environment time sync with domain controller is important. The domain controllers in AD behave as ntp servers. Edit the **/etc/ntp.conf** file. Comment out the preset timeservers and add your Domain Controller instead:
```
...
# You do need to talk to an NTP server or two (or three).
#server ntp.your-provider.example
server dc-srv.gontar.ca

# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
# pick a different set every time it starts up.  Please consider joining the
# pool: <http://www.pool.ntp.org/join.html>
#server 0.debian.pool.ntp.org iburst
#server 1.debian.pool.ntp.org iburst
#server 2.debian.pool.ntp.org iburst
#server 3.debian.pool.ntp.org iburst
...
```
Once this is done, restart ntp service `debian :: ~ » service ntp restart`.

Try getting a Kerberos ticket as domain administrator:
```
debian :: ~ » kinit administrator@GONTAR.CA
Password for administrator@GONTAR.CA:
```

If the command was successful, you will not see any output. Verify that the ticket was obtaineds with `$ klist`, the output should look something like this:
```
debian :: ~ » klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@GONTAR.CA

Valid starting       Expires              Service principal
04/10/2016 00:49:23  04/10/2016 10:49:23  krbtgt/GONTAR.CA@GONTAR.CA
        renew until 04/11/2016 00:49:18
```

That shows Kerberos authentication is working fine with the domain controller. 

Next create a new **/etc/realmd.conf** to configure realmd realm enrollment tool which will be used to join the domain, with the following settings:
```
[users]
default-home = /home/%D/%U
default-shell = /bin/bash
[active-directory]
default-client = sssd
os-name = Debian Jessie
os-version = 8.4
[service]
automatic-install = no
[gontar.ca]
fully-qualified-names = no
automatic-id-mapping = yes
user-principal = yes
manage-system = no
```
| Directive | Explanation |
| --- | --- |
| default-home | set the default homedir for each Active Directory User. This example will create /home/gontar.ca/domainuser. |
| default-shell | the default shell used by the users. bash is usually the default shell. |
| default-client | sssd will be used in this scenario. winbind is also a possible option. |
| os-name | the operating system name as it will appear in our Active Directory. |
| os-version | the operating system version as it will appear in our Active Directory. |
| automatic-install | prevent realmd to try to install its dependencies. |
| fully-qualified-names | allow users to use just their username instead of GONTAR\domainuser or domainuser@ad.gontar.net. Could cause conflicts with local users, if they have the same username. | 
| automatic-id-mapping | this option will auto-generate the user and group ids (UID, GID) for newly created users, if set to yes. |
| user-principal | this will set the necessary attributes for the linux machine when it joins the domain. |
| manage-system | if you don’t want policies from the Active Directory environment to be applied on this machine, set this option to no. |

Let's see if realmd can discover the domain:
```
debian :: ~ » realm discover GONTAR.CA
gontar.ca
  type: kerberos
  realm-name: GONTAR.CA
  domain-name: gontar.ca
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
```
Time to join the domain, issue the following command `realm --verbose join gontar.ca --user-principal=file-srv/administrator@GONTAR.CA --unattended`

To check that everything went ok do `realm list`:
```
debian :: ~ » realm list
gontar.ca
  type: kerberos
  realm-name: GONTAR.CA
  domain-name: gontar.ca
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U
  login-policy: allow-realm-logins
```
When using realmd to join the machine in the domain, it creates the configuration for sssd in the **/etc/sssd/sssd.conf** file. Unfortunately realmd does not get everything right.

Modify the `access_provider = simple` option in the **/etc/sssd/sssd.conf** file, to read `access_provider = ad` and append `sudo_provider = none` at the end of the file. sudo_provider helps to avoid errors when using sudo as a domain user (you can still give sudo to domain users through visudo).

Restart sssd `service sssd restart`.

Check that everything is working with `id` and `getent`:
```
debian :: ~ » id administrator
uid=1068800500(administrator) gid=1068800513(domain users) groups=1068800513(domain users),1068800520(group policy creator owners),1068800519(enterprise admins),1068800512(domain admins),1068800518(schema admins),1068801128(users),1068800572(denied rodc password replication group)
debian :: ~ » getent group "domain users"
domain users:*:1068800513:vasyl,administrator,veronika,vitaliy,krbtgt,maryna,julie
```
If either one of the commands does not return valid entries append `enumerate = True` at the end of **/etc/sssd/sssd.conf**, restart sssd and try again. If it is still not working check the logs under **/var/log/sssd/**, **/var/log/auth** and **/var/log/syslog**. Also never forget that [Google](https://google.com) is your friend!

Finally add the pam_mkhomedir module, as the last module in the **/etc/pam.d/common-session** file in order to set up home directory autocreation for all users upon first login:
```
debian :: ~ » echo "session required pam_mkhomedir.so skel=/etc/skel/ umask=0077" >> /etc/pam.d/common-session
```

It is possible to also authenticate logins to Desktop using Active Directory accounts. The AD accounts will not show up in the pick list with local users, so lightdm will need to be modified. Edit the file `/etc/lightdm/lightdm.conf.d/50-unity-greeter.conf` and append the following two lines:
```
greeter-show-manual-login=true
greeter-hide-users=true
```
Afterward lightdm need to be restarted (by rebooting the computer).

If GSSAPI will be used to authenticate windows clients connecting to Debian (passwordless connection without need to set up keys) it is also required to enable delegation in Active Directory Users and Computers for the newly created principal (Trust this computer for delegation to any service (Kerberos only).
