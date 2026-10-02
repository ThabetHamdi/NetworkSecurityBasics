# DMVPN Phase 3 (Hub & Spoke with Spoke-to-Spoke)

## Scenario
HQ is the hub; BR1 and BR2 are spokes with dynamic public IPs. Spokes build direct tunnels on demand (NHRP shortcuts). All encrypted with IPsec, routing with EIGRP/OSPF/BGP.

## Shared IPsec profile (all nodes)
```
crypto ikev2 keyring DMVPN-KR
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key <CHANGE-ME>
crypto ikev2 profile DMVPN-PROF
 match identity remote address 0.0.0.0
 authentication local pre-share
 authentication remote pre-share
 keyring local DMVPN-KR
crypto ipsec transform-set TS esp-gcm 256
 mode transport
crypto ipsec profile DMVPN-IPSEC
 set transform-set TS
 set ikev2-profile DMVPN-PROF
```
## Hub (HQ-R1)
```
interface Tunnel100
 ip address 172.16.100.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 ip nhrp network-id 100
 ip nhrp authentication <NHRPKEY>
 ip nhrp map multicast dynamic
 ip nhrp redirect
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN-IPSEC
 ! EIGRP specific: no ip split-horizon eigrp 100 / no ip next-hop-self eigrp 100 (classic)
```
## Spoke (BR1-R1)
```
interface Tunnel100
 ip address 172.16.100.2 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 ip nhrp network-id 100
 ip nhrp authentication <NHRPKEY>
 ip nhrp nhs 172.16.100.1 nbma 198.51.100.1 multicast
 ip nhrp shortcut
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN-IPSEC
```
Hub advertises a summary/default to spokes; spokes learn specifics via NHRP shortcut.

## Verification
```
show dmvpn
show ip nhrp
show crypto ikev2 sa
show ip route nhrp
```
