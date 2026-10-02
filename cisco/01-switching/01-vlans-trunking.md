# VLANs & Trunking

## Scenario
HQ access switches carry Users (10), Servers (20), Voice (30), Mgmt (99). Uplinks to core are 802.1Q trunks.
Unused ports go to a black-hole VLAN 999.

## Configuration (access / distribution switch)
```
vlan 10
 name USERS
vlan 20
 name SERVERS
vlan 30
 name VOICE
vlan 99
 name MGMT
vlan 999
 name BLACKHOLE

! Access port with voice VLAN
interface range GigabitEthernet1/0/1-24
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 30
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 description USER-PORT

! Trunk to core
interface range GigabitEthernet1/0/47-48
 switchport trunk encapsulation dot1q   ! not needed on IOS-XE 17+/3650+
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,99
 switchport nonegotiate                 ! disable DTP
 description TRUNK-TO-CORE

! Unused ports
interface range GigabitEthernet1/0/25-46
 switchport mode access
 switchport access vlan 999
 shutdown
 description UNUSED

! Management SVI
interface Vlan99
 ip address 10.10.99.11 255.255.255.0
 no shutdown
ip default-gateway 10.10.99.1
```

## Verification
```
show vlan brief
show interfaces trunk
show interfaces Gi1/0/48 switchport
show mac address-table dynamic vlan 10
```

## Security notes
- Native VLAN ≠ 1 and never used for user traffic (stops VLAN-hopping via double tagging).
- `switchport nonegotiate` kills DTP (switch-spoofing attack).
- Prune VLANs with `allowed vlan`; do not leave `all`.

## Troubleshooting
| Symptom | Check |
|---|---|
| VLAN not crossing trunk | `show int trunk` – allowed/active VLANs, VLAN exists? |
| Native VLAN mismatch log | Same native VLAN both ends |
