# NTP, Syslog, SNMPv3, NetFlow

## NTP (authenticated)
```
ntp authentication-key 1 hmac-sha2-256 <CHANGE-ME>
ntp authenticate
ntp trusted-key 1
ntp server 10.10.20.123 key 1 prefer
ntp source Loopback0
clock timezone CET 1
clock summer-time CEST recurring last Sun Mar 2:00 last Sun Oct 3:00
! NTP master in lab: ntp master 3
ntp access-group peer 11
```
## Syslog
```
logging buffered 64000 informational
logging host 10.10.99.70 transport udp port 514
logging source-interface Loopback0
logging trap informational
logging origin-id hostname
service sequence-numbers
login on-failure log
```
## SNMPv3 (no v1/v2c)
```
ip access-list standard SNMP-MGR
 permit host 10.10.99.60
snmp-server view RO-VIEW iso included
snmp-server group MONITOR v3 priv read RO-VIEW access SNMP-MGR
snmp-server user zabbix MONITOR v3 auth sha <AUTHPASS> priv aes 128 <PRIVPASS>
snmp-server host 10.10.99.60 version 3 priv zabbix
snmp-server enable traps config
snmp-server enable traps bgp
snmp-server enable traps ospf
snmp-server location HQ-DC1
```
## Flexible NetFlow
```
flow record FR
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 collect counter bytes long
 collect counter packets long
flow exporter FE
 destination 10.10.99.80
 transport udp 2055
 source Loopback0
flow monitor FM
 record FR
 exporter FE
interface Gi0/0
 ip flow monitor FM input
```
## Verification
```
show ntp status
show ntp associations
show logging
show snmp user
show flow monitor FM cache
```
