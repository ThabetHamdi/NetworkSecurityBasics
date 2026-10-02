# Layer-2 Security

## Scenario
Harden the access layer against rogue DHCP, ARP spoofing, MAC flooding, rogue switches, loops and unauthorized devices.

## Port-Security
```
interface range Gi1/0/1-24
 switchport port-security
 switchport port-security maximum 2           ! PC + phone
 switchport port-security mac-address sticky
 switchport port-security violation restrict  ! or shutdown
 switchport port-security aging time 10
errdisable recovery cause psecure-violation
```
## DHCP Snooping + DAI + IP Source Guard
```
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
interface Gi1/0/48                              ! uplink toward DHCP server
 ip dhcp snooping trust
 ip arp inspection trust
interface range Gi1/0/1-24
 ip dhcp snooping limit rate 15
 ip verify source                               ! IPSG

ip arp inspection vlan 10,20,30
ip arp inspection validate src-mac dst-mac ip
! Static hosts without DHCP:
arp access-list STATIC-HOSTS
 permit ip host 10.10.20.10 mac host aaaa.bbbb.cccc
ip arp inspection filter STATIC-HOSTS vlan 20
```
## Storm control / misc
```
interface range Gi1/0/1-24
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 switchport block unicast
no ip http server
lldp run
```
## 802.1X (basic)
```
aaa new-model
aaa authentication dot1x default group radius
aaa authorization network default group radius
dot1x system-auth-control
radius server ISE1
 address ipv4 10.10.99.50 auth-port 1812 acct-port 1813
 key <CHANGE-ME>
interface Gi1/0/5
 switchport mode access
 authentication port-control auto
 authentication order dot1x mab
 authentication priority dot1x mab
 dot1x pae authenticator
 spanning-tree portfast
```
## Private VLAN (isolate hosts in same subnet)
```
vtp mode transparent
vlan 201
 private-vlan isolated
vlan 200
 private-vlan primary
 private-vlan association 201
interface Gi1/0/10
 switchport mode private-vlan host
 switchport private-vlan host-association 200 201
interface Gi1/0/1
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping 200 201
```
## Verification
```
show port-security interface Gi1/0/1
show ip dhcp snooping binding
show ip arp inspection statistics
show ip verify source
show errdisable recovery
show authentication sessions
```
