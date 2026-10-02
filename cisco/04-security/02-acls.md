# Access Control Lists

## Scenario
Segment HQ: Users (VLAN 10) may reach servers on specific ports and the Internet, never management (VLAN 99). Internet edge ACL blocks spoofing and only allows required inbound.

## Rules of thumb
- Standard ACL: source only → place **close to destination**. Extended: place **close to source**.
- Implicit deny at end; add explicit `deny ip any any log` while testing.
- Wildcard mask = inverse of subnet mask. Order matters (first match).

## Named extended ACL (users → servers)
```
ip access-list extended USERS-IN
 remark Allow DHCP
 permit udp any eq bootpc any eq bootps
 remark DNS
 permit udp 10.10.10.0 0.0.0.255 host 10.10.20.53 eq domain
 permit tcp 10.10.10.0 0.0.0.255 host 10.10.20.53 eq domain
 remark Web & file servers
 permit tcp 10.10.10.0 0.0.0.255 10.10.20.0 0.0.0.255 eq 80 443 445
 remark Block management VLAN
 deny   ip 10.10.10.0 0.0.0.255 10.10.99.0 0.0.0.255 log
 remark Block other internal RFC1918 except above
 deny   ip any 10.0.0.0 0.255.255.255
 deny   ip any 172.16.0.0 0.15.255.255
 deny   ip any 192.168.0.0 0.0.255.255
 remark Internet
 permit ip 10.10.10.0 0.0.0.255 any
interface Vlan10
 ip access-group USERS-IN in
```
## Internet edge - inbound anti-spoofing
```
ip access-list extended WAN-IN
 deny ip 10.0.0.0 0.255.255.255 any log
 deny ip 172.16.0.0 0.15.255.255 any log
 deny ip 192.168.0.0 0.0.255.255 any log
 deny ip 127.0.0.0 0.255.255.255 any log
 deny ip 169.254.0.0 0.0.255.255 any log
 deny ip 224.0.0.0 15.255.255.255 any log
 deny ip host 0.0.0.0 any log
 deny ip 192.0.2.0 0.0.0.255 any log             ! our own public space as source
 permit tcp any host 192.0.2.10 eq 443            ! published web server
 permit udp host 203.0.113.2 host 198.51.100.1 eq isakmp
 permit esp host 203.0.113.2 host 198.51.100.1
 permit icmp any any echo-reply
 permit icmp any any unreachable
 permit icmp any any time-exceeded
 permit tcp any any established                   ! stateless; prefer ZBF/CBAC
 deny ip any any log
interface GigabitEthernet0/0
 ip access-group WAN-IN in
```
## Time-based ACL
```
time-range WORK-HOURS
 periodic weekdays 8:00 to 18:00
ip access-list extended GUEST
 permit tcp 10.10.50.0 0.0.0.255 any eq 80 443 time-range WORK-HOURS
```
## Object-groups (cleaner)
```
object-group network SERVERS
 host 10.10.20.10
 host 10.10.20.11
object-group service WEB
 tcp eq 80
 tcp eq 443
ip access-list extended OG-ACL
 permit object-group WEB 10.10.10.0 0.0.0.255 object-group SERVERS
```
## VTY / SNMP / control-plane ACL
```
access-list 10 permit 10.10.99.0 0.0.0.255
line vty 0 15
 access-class 10 in
snmp-server community <RO> RO 10      ! avoid v2c; see SNMPv3
```
## Verification
```
show access-lists
show ip access-lists USERS-IN
show ip interface Vlan10 | include access list
clear ip access-list counters
```
