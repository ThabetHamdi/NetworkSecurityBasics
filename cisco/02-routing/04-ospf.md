# OSPF (multi-area, v2 & v3)

## Scenario
Area 0 = HQ core; Area 10 = HQ campus (stub-ish); Area 20 = Branch 1 (NSSA because it redistributes a local static/ISP route). Authentication, summarization, DR/BDR control, BFD.

## ABR (HQ-R1)
```
key chain OSPF-KEYS
 key 1
  key-string <CHANGE-ME>
  cryptographic-algorithm hmac-sha-256

router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 100000      ! 100 Gbps (same on all routers)
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface Tunnel0
 network 1.1.1.1 0.0.0.0 area 0
 network 10.255.0.0 0.0.255.255 area 0
 network 10.10.0.0 0.0.255.255 area 10
 area 10 stub no-summary                    ! totally stubby
 area 20 nssa
 area 20 range 10.20.0.0 255.255.0.0        ! summarize at ABR
 area 0 authentication message-digest       ! legacy MD5 alternative
 default-information originate always metric 10
 bfd all-interfaces
 max-lsa 10000 80 warning-only

interface GigabitEthernet0/1
 ip ospf network point-to-point
 ip ospf authentication key-chain OSPF-KEYS ! SHA (IOS-XE 16.x+)
 ip ospf hello-interval 3
 ip ospf dead-interval 9
 ip ospf cost 10
 ip ospf priority 100                       ! DR election (0 = never)
 bfd interval 100 min_rx 100 multiplier 3

! Legacy MD5 per-interface
interface GigabitEthernet0/2
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 <CHANGE-ME>
```
## OSPFv3 (IPv4+IPv6 AF)
```
ipv6 unicast-routing
router ospfv3 1
 router-id 1.1.1.1
 address-family ipv6 unicast
interface GigabitEthernet0/1
 ospfv3 1 ipv6 area 0
 ospfv3 authentication ipsec spi 256 sha1 <40-hex-chars>
```
## Virtual-link / Stub / Totally NSSA reminders
```
area 30 virtual-link 2.2.2.2                ! avoid unless unavoidable
area 20 nssa no-summary
area 20 nssa default-information-originate
```
## Verification
```
show ip ospf neighbor
show ip ospf interface brief
show ip ospf database
show ip route ospf
show ip ospf border-routers
show bfd neighbors
```
## Troubleshooting (neighbor stuck)
| State | Typical cause |
|---|---|
| INIT | ACL blocking, one-way multicast, auth mismatch |
| 2WAY | Normal on broadcast between DROTHERs |
| EXSTART/EXCHANGE | MTU mismatch (`ip ospf mtu-ignore` only as workaround) |
| Not forming | Area ID, hello/dead, stub flag, subnet mask, auth mismatch |
