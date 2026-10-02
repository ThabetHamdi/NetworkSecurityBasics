# Linux as Router: FRRouting (OSPF / BGP / BFD)

## Install & enable
```bash
apt install frr frr-pythontools      # or dnf install frr
sed -i 's/^ospfd=no/ospfd=yes/; s/^bgpd=no/bgpd=yes/; s/^bfdd=no/bfdd=yes/' /etc/frr/daemons
systemctl enable --now frr
sysctl -w net.ipv4.ip_forward=1
vtysh
```
## OSPF with MD5 + BFD
```
configure terminal
 router ospf
  ospf router-id 1.1.1.1
  passive-interface default
  no passive-interface eth1
  network 10.10.0.0/16 area 0
 exit
 interface eth1
  ip ospf area 0
  ip ospf network point-to-point
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 <CHANGE-ME>
  ip ospf bfd
 exit
```
## eBGP with filters
```
router bgp 65001
 bgp router-id 1.1.1.1
 no bgp default ipv4-unicast
 neighbor 198.51.100.2 remote-as 64500
 neighbor 198.51.100.2 password <CHANGE-ME>
 neighbor 198.51.100.2 ttl-security hops 1
 neighbor 198.51.100.2 bfd
 address-family ipv4 unicast
  network 192.0.2.0/24
  neighbor 198.51.100.2 activate
  neighbor 198.51.100.2 prefix-list MY-NETS out
  neighbor 198.51.100.2 maximum-prefix 1000
 exit-address-family
ip prefix-list MY-NETS seq 5 permit 192.0.2.0/24
```
## Verification
```
show ip ospf neighbor
show ip route
show bgp summary
show bgp ipv4 unicast
show bfd peers
write memory
```
