# BGP (eBGP dual-homed + iBGP)

## Scenario
HQ (AS 65001) is multi-homed to ISP-A (AS 64500, primary) and ISP-B (AS 64501, backup). HQ-R1 ↔ HQ-R2 are iBGP peers. Inbound traffic is steered with AS-path prepending; outbound with local-preference. Prefix filtering prevents becoming a transit AS.

## HQ-R1 (peering with ISP-A)
```
! Filters
ip prefix-list MY-NETS seq 5 permit 192.0.2.0/24
ip prefix-list BOGONS seq 5 deny 0.0.0.0/8 le 32
ip prefix-list BOGONS seq 10 deny 10.0.0.0/8 le 32
ip prefix-list BOGONS seq 15 deny 127.0.0.0/8 le 32
ip prefix-list BOGONS seq 20 deny 172.16.0.0/12 le 32
ip prefix-list BOGONS seq 25 deny 192.168.0.0/16 le 32
ip prefix-list BOGONS seq 30 deny 169.254.0.0/16 le 32
ip prefix-list BOGONS seq 35 deny 224.0.0.0/3 le 32
ip prefix-list BOGONS seq 40 deny 0.0.0.0/0 ge 25          ! too-specific
ip prefix-list BOGONS seq 100 permit 0.0.0.0/0 le 24
ip as-path access-list 10 permit ^$                         ! only locally originated

route-map TO-ISPA-OUT permit 10
 match ip address prefix-list MY-NETS
 match as-path 10
route-map FROM-ISPA-IN permit 10
 match ip address prefix-list BOGONS
 set local-preference 200                                   ! prefer ISP-A outbound
route-map TO-ISPB-OUT permit 10
 match ip address prefix-list MY-NETS
 set as-path prepend 65001 65001 65001                      ! de-prefer inbound via ISP-B

router bgp 65001
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 no bgp default ipv4-unicast
 neighbor 198.51.100.2 remote-as 64500
 neighbor 198.51.100.2 description ISP-A
 neighbor 198.51.100.2 password <CHANGE-ME>                 ! MD5; TCP-AO on newer IOS-XE
 neighbor 198.51.100.2 ttl-security hops 1                  ! GTSM
 neighbor 198.51.100.2 fall-over bfd
 neighbor 2.2.2.2 remote-as 65001                           ! iBGP to HQ-R2
 neighbor 2.2.2.2 update-source Loopback0
 neighbor 2.2.2.2 password <CHANGE-ME>
 address-family ipv4
  network 192.0.2.0 mask 255.255.255.0
  neighbor 198.51.100.2 activate
  neighbor 198.51.100.2 route-map FROM-ISPA-IN in
  neighbor 198.51.100.2 route-map TO-ISPA-OUT out
  neighbor 198.51.100.2 maximum-prefix 1000 80 restart 5
  neighbor 198.51.100.2 soft-reconfiguration inbound
  neighbor 2.2.2.2 activate
  neighbor 2.2.2.2 next-hop-self
  neighbor 2.2.2.2 send-community both
 exit-address-family

ip route 192.0.2.0 255.255.255.0 Null0                      ! anchor route for network statement
```
## Useful BGP tools
```
! MED
route-map SET-MED permit 10
 set metric 50
! Communities (e.g. blackhole / local-pref signalling)
route-map TAG permit 10
 set community 65001:100 additive
ip bgp-community new-format
! Route reflector
router bgp 65001
 address-family ipv4
  neighbor 10.255.0.2 route-reflector-client
! Confederation / peer-groups
neighbor PEERS peer-group
! Graceful restart / soft clear
clear ip bgp 198.51.100.2 soft in
```
## Verification
```
show ip bgp summary
show ip bgp
show ip bgp neighbors 198.51.100.2 advertised-routes
show ip bgp neighbors 198.51.100.2 received-routes
show ip bgp 8.8.8.0/24
show ip route bgp
```
## Best-path order (memory aid)
Weight → Local-Pref → Locally originated → AS-path length → Origin → MED → eBGP over iBGP → IGP metric → router-ID.

## Security notes
Never advertise learned prefixes to another ISP (transit leak): outbound prefix-list/as-path filters, max-prefix, GTSM, MD5/TCP-AO, RPKI origin validation (`bgp rpki server tcp ...`), uRPF strict on edges.
