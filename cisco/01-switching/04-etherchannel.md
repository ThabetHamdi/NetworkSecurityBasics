# EtherChannel (LACP / PAgP / L3)

## Scenario
Bundle 2x10G between CORE1 and CORE2 (and each access switch to both cores) for bandwidth + redundancy.

## L2 LACP
```
interface range TenGigabitEthernet1/1/1-2
 channel-group 1 mode active          ! LACP active; 'passive' passive; 'on' static (avoid)
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,99
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,99

port-channel load-balance src-dst-ip
lacp system-priority 100
```
## PAgP (Cisco-only)
```
interface range Gi1/0/1-2
 channel-group 2 mode desirable
```
## L3 EtherChannel (routed)
```
interface Port-channel3
 no switchport
 ip address 10.255.0.1 255.255.255.252
interface range Te1/1/3-4
 no switchport
 channel-group 3 mode active
```
## Multi-chassis
Use StackWise Virtual / VSS so access switches bundle across both cores with one Port-channel (no STP-blocked links).

## Verification
```
show etherchannel summary          ! flags: SU = good, SD/I = problem
show lacp neighbor
show interfaces port-channel 1
```
Mismatch causes: speed/duplex/VLAN/trunk settings differ between members.
