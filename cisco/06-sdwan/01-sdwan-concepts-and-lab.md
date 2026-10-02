# Cisco Catalyst SD-WAN (Viptela) - Concepts & CLI

## Scenario
HQ + 2 branches connected over MPLS and Internet (dual transport). Centralized policy: voice prefers MPLS, bulk traffic uses Internet, automatic failover on SLA violation.

## Architecture (planes)
| Component | Role |
|---|---|
| **vManage** | Management plane (GUI, templates, monitoring) |
| **vSmart** | Control plane (OMP, policies, route/key distribution) |
| **vBond** | Orchestrator, authentication & NAT discovery |
| **WAN Edge (cEdge/vEdge)** | Data plane routers (IPsec/GRE tunnels, BFD) |

Key terms: **TLOC** (system-ip, color, encap), **OMP** (route/TLOC/service advertisement), **VPN 0** (transport), **VPN 512** (mgmt), **VPN 1+** (service/LAN), BFD probes measure loss/latency/jitter per tunnel, **control policy** (vSmart) vs **data policy** (edge), **App-aware routing (AAR)** policy with SLA classes.

## Controller bootstrap (vBond/vSmart/vManage) - typical lab (CLI)
```
system
 host-name vSmart1
 system-ip 10.255.255.2
 site-id 100
 organization-name "LAB-ORG"
 vbond 192.0.2.250
!
vpn 0
 interface eth0
  ip address 192.0.2.251/24
  tunnel-interface
   allow-service all
  no shutdown
 ip route 0.0.0.0/0 192.0.2.1
```
## WAN Edge (IOS-XE SD-WAN, cEdge) in autonomous/CLI-bootstrap mode
```
system
 system-ip             11.11.11.11
 site-id               20
 organization-name     "LAB-ORG"
 sp-organization-name  "LAB-ORG"
 vbond 192.0.2.250
!
sdwan
 interface GigabitEthernet1
  tunnel-interface
   encapsulation ipsec
   color mpls restrict
   allow-service dhcp
   allow-service dns
   allow-service icmp
  exit
 interface GigabitEthernet2
  tunnel-interface
   encapsulation ipsec
   color biz-internet
   allow-service dhcp
   allow-service dns
   allow-service icmp
   allow-service https
  exit
!
vrf definition 1
 rd 1:1
 address-family ipv4
  route-target export 1:1
  route-target import 1:1
 exit-address-family
interface GigabitEthernet3
 vrf forwarding 1
 ip address 10.20.1.1 255.255.255.0
!
! Service-side routing in VPN 1 (OSPF/BGP/static)
router ospf 1 vrf 1
 router-id 11.11.11.11
 network 10.20.1.0 0.0.0.255 area 0
```
## Policies (vSmart CLI view)
```
policy
 sla-class VOICE-SLA
  loss 1
  latency 150
  jitter 30
 !
 app-route-policy AAR-VOICE
  vpn-list VPN1
   sequence 10
    match
     dscp 46
    !
    action
     sla-class VOICE-SLA preferred-color mpls
    !
   !
   default-action sla-class VOICE-SLA
  !
 !
 lists
  vpn-list VPN1
   vpn 1
  site-list ALL-SITES
   site-id 10-30
 !
apply-policy
 site-list ALL-SITES
  app-route-policy AAR-VOICE
 !
```
## Verification (edge)
```
show sdwan control connections
show sdwan omp routes
show sdwan omp tlocs
show sdwan bfd sessions
show sdwan app-route stats
show sdwan control local-properties
show sdwan certificate installed
show ip route vrf 1
```
## Security in SD-WAN
- Certificates (device identity), DTLS/TLS control connections, AES-256-GCM IPsec data plane with key rotation.
- Segmentation by VPN (VRF) end-to-end; use `allow-service` minimalism on tunnels; ZBF / Snort IPS / URL filtering via security policy on cEdge.
- Disable unused `allow-service`, restrict vManage via ACL/mgmt VPN, enforce RBAC and MFA.

## Troubleshooting
| Symptom | Check |
|---|---|
| control connection `connect` / `tear_down` | Time sync, certs/org-name, vBond reachable, firewall UDP 12346+ |
| BFD down | Colors restrict mismatch, NAT type, underlay MTU |
| Routes missing | OMP advertise, VPN membership, control-policy on vSmart |
