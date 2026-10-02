# Cisco

| Folder | Files |
|---|---|
| 01-switching | VLANs/trunking, STP, VTP, EtherChannel, HSRP/VRRP/GLBP, L2 security, inter-VLAN |
| 02-routing | Static+IP SLA, RIPv2, EIGRP, OSPF v2/v3, BGP, redistribution/PBR, BFD |
| 03-vpn | GRE, IPsec IKEv2 VTI/crypto-map, DMVPN Phase 3 |
| 04-security | Hardening baseline, ACLs, AAA/TACACS+, CoPP/uRPF/ZBF |
| 05-services | NAT, DHCP, NTP/Syslog/SNMPv3/NetFlow, QoS, IP SLA/EEM |
| 06-sdwan | Catalyst SD-WAN concepts + CLI, DIY SD-WAN (DMVPN+IP SLA+PBR+Ansible) |

Suggested lab order: switching → routing → vpn → security → services → sdwan (EVE-NG / GNS3 / CML).
Save configs with `copy running-config startup-config`; roll back with `configure replace flash:backup force`.
