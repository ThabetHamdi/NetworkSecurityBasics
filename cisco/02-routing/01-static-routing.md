# Static Routing

## Scenario
HQ-R1 has two ISPs. ISP-A is primary, ISP-B is a floating backup. Branch networks are reached statically; the backup default is withdrawn automatically when ISP-A stops responding (IP SLA tracking).

## Configuration
```
! Basic
ip route 10.20.0.0 255.255.0.0 172.16.255.2
ip route 10.30.0.0 255.255.0.0 172.16.255.6

! Default routes (primary + floating AD 250)
ip route 0.0.0.0 0.0.0.0 198.51.100.2
ip route 0.0.0.0 0.0.0.0 203.0.113.2 250

! Tracked primary default (IP SLA)
ip sla 1
 icmp-echo 198.51.100.2 source-interface GigabitEthernet0/0
 frequency 5
 threshold 1000
ip sla schedule 1 life forever start-time now
track 10 ip sla 1 reachability
no ip route 0.0.0.0 0.0.0.0 198.51.100.2
ip route 0.0.0.0 0.0.0.0 198.51.100.2 track 10

! Summary / discard route (loop prevention before advertising)
ip route 10.10.0.0 255.255.0.0 Null0

! Recursive vs directly attached (prefer next-hop + interface on Ethernet)
ip route 192.168.50.0 255.255.255.0 GigabitEthernet0/1 192.168.1.2

! IPv6
ipv6 unicast-routing
ipv6 route ::/0 2001:db8:100::2
```
## Verification
```
show ip route static
show ip route 0.0.0.0
show ip sla statistics
show track 10
traceroute 8.8.8.8 source 10.10.10.1
```
Security: add `ip verify unicast source reachable-via rx` on ISP links (uRPF).
