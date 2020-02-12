# fail2ban config examples

#### Ubuntu/Debian install:
```
# apt install fail2ban
# systemctl enable fail2ban
```
#### Configure:

/etc/fail2ban/jail.local
```
[DEFAULT] ignoreip = 127.0.0.1/8 ::1
bantime = 3600
findtime = 1800
maxretry = 15

[sshd] enabled = true
banaction = iptables-multiport
port = 22,5561

[sshd-ddos]
enabled = true
port = 22
protocol = tcp
filter = sshd-ddos
backend = systemd
bantime = 86400
findtime = 3
maxretry = 300
action = iptables-allports[name=sshd-ddos, protocol=all]

[http-ddos]
enabled = true
port = 80,443
filter = http-ddos
# logpath = [путь к access логу вашего сайта, например /var/log/httpd/access.log]
maxretry = 45000
findtime = 3
bantime = 300
#action = iptables-dos[name=http, port=http, protocol=all]
action = iptables-allports[name=http-ddos, protocol=all]

```
/etc/fail2ban/filter.d/http-ddos.conf
```
# Fail2Ban configuration file
#
# Author: http://www.adminhelp.pro
#
[Definition]

# Option: failregex
# Note: This regex will match any GET and POST entry in your logs, so basically all valid and not valid entries are a match.
# You should set up in the jail.conf file, the maxretry and findtime carefully in order to avoid false positives.

failregex = ^<HOST> -.*"(GET|POST).*

# Option: ignoreregex
# Notes.: regex to ignore. If this regex matches, the line is ignored.
# Values: TEXT
#
ignoreregex =

```
/etc/fail2ban/action.d/iptables-dos.conf
```
# Fail2Ban configuration file
#
# Author: Cyril Jaquier and http://www.adminhelp.pro
#
#

[INCLUDES]

before = iptables-blocktype.conf

[Definition]

# Option: actionstart
# Notes.: command executed once at the start of Fail2Ban.
# Values: CMD
#
actionstart = iptables -N fail2ban-<name>
iptables -A fail2ban-<name> -j RETURN
iptables -I <chain> -p <protocol> --dport <port> -j fail2ban-<name>

# Option: actionstop
# Notes.: command executed once at the end of Fail2Ban
# Values: CMD
#
actionstop = iptables -D <chain> -p <protocol> --dport <port> -j fail2ban-<name>
iptables -F fail2ban-<name>
iptables -X fail2ban-<name>

# Option: actioncheck
# Notes.: command executed once before each actionban command
# Values: CMD
#
actioncheck = iptables -n -L <chain> | grep -q 'fail2ban-<name>[ \t]'

# Option: actionban
# Notes.: command executed when banning an IP. Take care that the
# command is executed with Fail2Ban user rights.
# Tags: See jail.conf(5) man page
# Values: CMD
#
actionban = iptables -I fail2ban-<name> 1 -s <ip> -j <blocktype>

# Option: actionunban
# Notes.: command executed when unbanning an IP. Take care that the
# command is executed with Fail2Ban user rights.
# Tags: See jail.conf(5) man page
# Values: CMD
#
actionunban = iptables -D fail2ban-<name> -s <ip> -j <blocktype>

[Init]

# Default name of the chain
#
name = default
### blocktype
blocktype = DROP

# Option: port
# Notes.: specifies port to monitor
# Values: [ NUM | STRING ] Default:

#
port = ssh

# Option: protocol
# Notes.: internally used by config reader for interpolations.
# Values: [ tcp | udp | icmp | all ] Default: tcp
#
protocol = tcp

# Option: chain
# Notes specifies the iptables chain to which the fail2ban rules should be
# added
# Values: STRING Default: INPUT
chain = INPUT
```
#### Start fail2ban and check stats:
```
systemctl start fail2ban

fail2ban-client status
Status
|- Number of jail:      3
`- Jail list:   http-ddos, sshd, sshd-ddos

fail2ban-client status sshd-ddos
Status for the jail: sshd-ddos
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
   
fail2ban-client status http-ddos
Status for the jail: http-ddos
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- File list:
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
```
