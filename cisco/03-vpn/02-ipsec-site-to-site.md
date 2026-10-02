# IPsec Site-to-Site VPN

## Scenario
Encrypt HQ ↔ Branch 1 traffic over the Internet. Two designs: **(A) IKEv2 route-based VTI** (recommended, supports routing protocols) and **(B) legacy crypto-map** with ACL.

## (A) IKEv2 + IPsec VTI - HQ-R1
```
crypto ikev2 proposal PROP-AES256
 encryption aes-cbc-256
 integrity sha256
 group 19 20
crypto ikev2 policy POL1
 proposal PROP-AES256
crypto ikev2 keyring KR-BR1
 peer BR1
  address 203.0.113.2
  pre-shared-key local <CHANGE-ME>
  pre-shared-key remote <CHANGE-ME>
crypto ikev2 profile PROF-BR1
 match identity remote address 203.0.113.2 255.255.255.255
 authentication local pre-share
 authentication remote pre-share
 keyring local KR-BR1
 lifetime 28800
 dpd 10 3 on-demand

crypto ipsec transform-set TS-AES256 esp-gcm 256
 mode tunnel
crypto ipsec profile IPSEC-PROF
 set transform-set TS-AES256
 set ikev2-profile PROF-BR1
 set pfs group19
 set security-association lifetime seconds 3600

interface Tunnel1
 ip address 172.16.255.5 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source GigabitEthernet0/0
 tunnel destination 203.0.113.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-PROF

! Underlay must allow ESP/IKE (see ACL)
ip access-list extended WAN-IN
 permit udp host 203.0.113.2 host 198.51.100.1 eq isakmp
 permit udp host 203.0.113.2 host 198.51.100.1 eq non500-isakmp
 permit esp host 203.0.113.2 host 198.51.100.1
```
BR1-R1 mirrors it with source/destination, keyring address and `match identity` swapped. Run OSPF/EIGRP over Tunnel1.

Prefer certificates (PKI/SCEP) over PSK in production: `authentication local rsa-sig` + `pki trustpoint`.

## (B) Legacy crypto map (policy-based)
```
ip access-list extended VPN-TRAFFIC
 permit ip 10.10.0.0 0.0.255.255 10.20.0.0 0.0.255.255
crypto map CMAP 10 ipsec-isakmp
 set peer 203.0.113.2
 set transform-set TS-AES256
 set pfs group19
 match address VPN-TRAFFIC
 set ikev2-profile PROF-BR1
interface GigabitEthernet0/0
 crypto map CMAP
! NAT exemption so VPN traffic bypasses PAT
ip access-list extended NAT-ACL
 deny   ip 10.10.0.0 0.0.255.255 10.20.0.0 0.0.255.255
 permit ip 10.10.0.0 0.0.255.255 any
```
## Verification
```
show crypto ikev2 sa
show crypto ipsec sa | include encaps|decaps|peer
show crypto session
show interfaces tunnel 1
debug crypto ikev2          ! lab only; undebug all after
```
## Troubleshooting
| Symptom | Cause |
|---|---|
| IKE SA down | PSK/identity mismatch, proposal mismatch, UDP 500/4500 blocked |
| IKE up, no traffic | ACL/proxy-ID mismatch, NAT not exempted, MTU/MSS |
| Flaps | DPD too aggressive, asymmetric routing, ISP filtering ESP |
