# DHCP Server / Relay

```
! Server on router/L3 switch
ip dhcp excluded-address 10.10.10.1 10.10.10.20
ip dhcp pool USERS
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.20.53 10.10.20.54
 domain-name corp.local
 lease 0 8
ip dhcp pool PRINTER-RESERVED
 host 10.10.10.30 255.255.255.0
 client-identifier 0100.aabb.ccdd.ee
ip dhcp conflict logging
ip dhcp ping packets 2

! Relay (SVI toward users, server elsewhere)
interface Vlan10
 ip helper-address 10.10.20.10

! Option 43/66/150 for IP phones / APs
ip dhcp pool VOICE
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 option 150 ip 10.10.20.100
```
```
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip dhcp server statistics
```
Pair with DHCP Snooping (`01-switching/06-l2-security.md`).
