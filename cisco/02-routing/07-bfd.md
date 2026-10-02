# BFD (Bidirectional Forwarding Detection)

## Scenario
Sub-second failure detection (300 ms x 3) on HQ↔Branch links so OSPF/EIGRP/BGP/HSRP reconverge in under 1 second instead of waiting for dead timers.

## Configuration
```
! Interface timers
interface GigabitEthernet0/1
 bfd interval 300 min_rx 300 multiplier 3

! Template (single-hop/multi-hop)
bfd-template single-hop BFD-FAST
 interval min-tx 300 min-rx 300 multiplier 3
 authentication sha-1 keychain BFD-KEYS    ! BFD authentication

! Bind to protocols
router ospf 1
 bfd all-interfaces
router eigrp CORP
 address-family ipv4 unicast autonomous-system 100
  af-interface GigabitEthernet0/1
   bfd
router bgp 65001
 neighbor 198.51.100.2 fall-over bfd
interface Vlan10
 standby bfd
! Static routes
ip route static bfd GigabitEthernet0/1 172.16.255.2
ip route 10.20.0.0 255.255.0.0 GigabitEthernet0/1 172.16.255.2
```
## Verification
```
show bfd neighbors detail
show bfd summary
show ip ospf neighbor
```
Notes: BFD is CPU-assisted on some platforms; keep timers realistic (e.g. ≥300 ms on software routers, ≥50 ms only on hardware offload).
