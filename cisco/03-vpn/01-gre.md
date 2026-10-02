# GRE Tunnels

## Scenario
Transport routing protocols (OSPF/EIGRP) and multicast between HQ and Branch 1 over the Internet. GRE alone is **not encrypted** - protect it with IPsec (next file).

## HQ-R1
```
interface Tunnel0
 description GRE-TO-BR1
 ip address 172.16.255.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source GigabitEthernet0/0
 tunnel destination 203.0.113.2
 keepalive 5 3
 ! tunnel key 100                    ! optional demux, NOT security
```
## BR1-R1
```
interface Tunnel0
 ip address 172.16.255.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source GigabitEthernet0/0
 tunnel destination 198.51.100.1
```
## Verification
```
show interfaces tunnel 0
ping 172.16.255.2 df-bit size 1400
show ip route 10.20.0.0
```
Pitfalls: recursive routing (tunnel destination learned through the tunnel) → keep underlay and overlay in separate protocols/VRF (use `tunnel vrf` / FVRF).
