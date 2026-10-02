# Inter-VLAN Routing

## Router-on-a-stick
```
interface GigabitEthernet0/0
 no shutdown
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 10.10.99.1 255.255.255.0
interface GigabitEthernet0/0.999
 encapsulation dot1Q 999 native
```
## L3 switch (SVI)
```
ip routing
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 ip helper-address 10.10.20.10      ! DHCP server
 ip access-group USERS-IN in
interface Vlan20
 ip address 10.10.20.1 255.255.255.0
! Routed uplink
interface Gi1/0/1
 no switchport
 ip address 10.255.1.2 255.255.255.252
ip route 0.0.0.0 0.0.0.0 10.255.1.1
```
## Verification
```
show ip interface brief
show ip route connected
show interfaces vlan 10
```
Security: apply ACLs on SVIs so users cannot reach the management VLAN 99 (see `../04-security/02-acls.md`).
