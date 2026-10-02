# Spanning Tree (Rapid-PVST+ / MST)

## Scenario
Two core switches, redundant access uplinks. CORE1 is root for VLANs 10,20; CORE2 for 30,99 (load sharing).

## Rapid-PVST+
```
spanning-tree mode rapid-pvst
spanning-tree extend system-id
! CORE1
spanning-tree vlan 10,20 root primary
spanning-tree vlan 30,99 root secondary
! CORE2 (mirror)
spanning-tree vlan 30,99 root primary
spanning-tree vlan 10,20 root secondary
! or explicit
spanning-tree vlan 10 priority 4096

! Access switch protections
interface range Gi1/0/1-24
 spanning-tree portfast
 spanning-tree bpduguard enable
spanning-tree portfast bpduguard default       ! global alternative
spanning-tree loopguard default                ! on point-to-point uplinks

! Core downlinks to access: never let access become root
interface Gi1/0/10
 spanning-tree guard root

! Cost tuning / UDLD
interface Gi1/0/48
 spanning-tree vlan 10 cost 10
udld enable
errdisable recovery cause bpduguard
errdisable recovery interval 300
```

## MST (many VLANs)
```
spanning-tree mode mst
spanning-tree mst configuration
 name HQ-MST
 revision 1
 instance 1 vlan 10,20
 instance 2 vlan 30,99
spanning-tree mst 1 priority 4096
spanning-tree mst 2 priority 8192
```

## Verification
```
show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree inconsistentports
show spanning-tree root
```

## Security notes
BPDU Guard on edge, Root Guard toward untrusted, Loop Guard/UDLD on fiber uplinks, avoid `spanning-tree bpdufilter` casually.
