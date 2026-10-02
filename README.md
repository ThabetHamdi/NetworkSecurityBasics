# Network Engineering & Security Playbook

Scenario-based configuration reference for **Cisco**, **Linux** and **Windows** networking, with a security-first approach.
Every file follows the same layout: **Scenario → Topology → Configuration → Verification → Security notes / Troubleshooting**.

```
network/
├── cisco/
│   ├── 01-switching/   VLAN, trunking, STP, VTP, EtherChannel, HSRP/VRRP/GLBP, L2 security, inter-VLAN
│   ├── 02-routing/     Static, RIP, EIGRP, OSPF, BGP, redistribution/PBR, BFD
│   ├── 03-vpn/         GRE, IPsec (VTI + crypto-map), DMVPN
│   ├── 04-security/    Device hardening, ACLs, AAA/TACACS+, CoPP/uRPF/ZBF
│   ├── 05-services/    NAT, DHCP, NTP/Syslog/SNMPv3, QoS, IP SLA/NetFlow
│   └── 06-sdwan/       Cisco Catalyst SD-WAN + DIY SD-WAN (DMVPN + IP SLA + PBR)
├── linux/              iproute2, nftables, hardening, WireGuard/strongSwan, FRR, tcpdump, IDS
├── windows/            PowerShell networking, firewall, hardening, AD/DNS/DHCP, VPN, logging
└── docs/               Lab topology, IP plan, security checklist
```

## Conventions
- Lab addressing uses RFC 5737 documentation ranges (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) for "internet" and `10.x.x.x` for internal. See `docs/lab-topology.md`.
- Secrets are placeholders (`<CHANGE-ME>`). **Never commit real keys/passwords.**
- Commands target IOS / IOS-XE 16.x–17.x; differences are noted inline.
- Use only on systems you own or are authorized to test.
