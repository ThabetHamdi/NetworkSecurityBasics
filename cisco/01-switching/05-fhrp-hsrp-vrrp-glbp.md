# First-Hop Redundancy: HSRP / VRRP / GLBP

## Scenario
CORE1 and CORE2 provide default gateway 10.10.10.1 for VLAN 10. CORE1 active; fail-over if its uplink dies (object tracking).

## HSRPv2 (Cisco)
```
! CORE1
track 1 interface GigabitEthernet1/0/1 line-protocol
interface Vlan10
 ip address 10.10.10.2 255.255.255.0
 standby version 2
 standby 10 ip 10.10.10.1
 standby 10 priority 110
 standby 10 preempt delay minimum 60
 standby 10 track 1 decrement 20
 standby 10 authentication md5 key-string <CHANGE-ME>
 standby 10 timers msec 250 msec 750      ! fast timers; or use BFD
 standby bfd
! CORE2
interface Vlan10
 ip address 10.10.10.3 255.255.255.0
 standby version 2
 standby 10 ip 10.10.10.1
 standby 10 priority 100
 standby 10 preempt
 standby 10 authentication md5 key-string <CHANGE-ME>
```
Load-share: make CORE2 active for VLAN 20 (priority 110) and CORE1 for VLAN 10 - align with STP root.

## VRRPv3 (multi-vendor)
```
fhrp version vrrp v3
interface Vlan20
 ip address 10.10.20.2 255.255.255.0
 vrrp 20 address-family ipv4
  address 10.10.20.1 primary
  priority 120
  preempt
  track 1 decrement 30
```
## GLBP (load balancing)
```
interface Vlan30
 ip address 10.10.30.2 255.255.255.0
 glbp 30 ip 10.10.30.1
 glbp 30 priority 110
 glbp 30 preempt
 glbp 30 load-balancing round-robin
 glbp 30 authentication md5 key-string <CHANGE-ME>
```
## Verification
```
show standby brief
show vrrp brief
show glbp brief
show track
```
