# NAT / PAT

## Scenario
HQ users reach the Internet via PAT on the ISP-A interface; a web server is published with static NAT; VPN traffic is exempted.

```
interface GigabitEthernet0/1
 ip nat inside
interface GigabitEthernet0/0
 ip nat outside

! PAT with interface overload
ip access-list extended NAT-ACL
 deny   ip 10.10.0.0 0.0.255.255 10.20.0.0 0.0.255.255     ! VPN exemption
 permit ip 10.10.0.0 0.0.255.255 any
ip nat inside source list NAT-ACL interface GigabitEthernet0/0 overload

! Static (port forward)
ip nat inside source static tcp 10.10.20.10 443 198.51.100.1 443
ip nat inside source static 10.10.20.20 192.0.2.20

! Dynamic pool
ip nat pool PUBPOOL 192.0.2.50 192.0.2.60 netmask 255.255.255.0
ip nat inside source list NAT-ACL pool PUBPOOL overload

! Route-map NAT (dual ISP)
route-map NAT-ISPA permit 10
 match ip address NAT-ACL
 match interface GigabitEthernet0/0
ip nat inside source route-map NAT-ISPA interface GigabitEthernet0/0 overload
```
## Verification
```
show ip nat translations
show ip nat statistics
clear ip nat translation *
debug ip nat           ! lab only
```
