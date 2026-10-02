# Redistribution, Route Filtering & Policy-Based Routing

## Scenario
Branch 2 runs RIP; HQ runs OSPF. HQ-R2 is the boundary: mutual redistribution with route-tags to prevent loops. Also steer VoIP via ISP-B using PBR.

## Two-point-safe mutual redistribution
```
route-map RIP-TO-OSPF deny 10
 match tag 110                             ! never re-inject what came from OSPF
route-map RIP-TO-OSPF permit 20
 set tag 120
 set metric-type 1
route-map OSPF-TO-RIP deny 10
 match tag 120
route-map OSPF-TO-RIP permit 20
 set tag 110
 set metric 3

router ospf 1
 redistribute rip subnets route-map RIP-TO-OSPF
router rip
 redistribute ospf 1 metric 3 route-map OSPF-TO-RIP

! Seed metric for EIGRP
router eigrp CORP
 address-family ipv4 unicast autonomous-system 100
  topology base
   redistribute ospf 1 metric 100000 10 255 1 1500
```
## Filtering with distribute-list / prefix-list
```
ip prefix-list ALLOW seq 10 permit 10.30.0.0/16 le 24
router ospf 1
 distribute-list prefix ALLOW in GigabitEthernet0/1
! Administrative distance tweak
router ospf 1
 distance 115 1.1.1.1 0.0.0.0
```
## Policy-Based Routing (VoIP via ISP-B)
```
ip sla 20
 icmp-echo 203.0.113.2
 frequency 5
ip sla schedule 20 life forever start-time now
track 20 ip sla 20 reachability
ip access-list extended VOIP
 permit udp 10.10.30.0 0.0.0.255 any range 16384 32767
route-map PBR-VOIP permit 10
 match ip address VOIP
 set ip next-hop verify-availability 203.0.113.2 1 track 20
interface Vlan30
 ip policy route-map PBR-VOIP
ip local policy route-map PBR-VOIP      ! for router-originated traffic (optional)
```
## Verification
```
show ip route | include tag
show route-map PBR-VOIP
show ip policy
debug ip policy          ! lab only
```
