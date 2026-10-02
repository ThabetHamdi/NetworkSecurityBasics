# Lab Topology & IP Plan

```
                         ┌────────── ISP-A (198.51.100.0/30) ──────────┐
   [HQ-R1]───────────────┤                                              ├──────────[BR1-R1]
     │  \                └────────── ISP-B (203.0.113.0/30)  ───────────┘             │
     │   \__[HQ-R2]  (HSRP / dual-ISP)                                              [BR1-SW1]
  [HQ-CORE-SW1]═══(Po1)═══[HQ-CORE-SW2]                                              Users/VoIP
        │                       │                                      [BR2-R1]── Branch 2
  [HQ-ACC-SW1]            [HQ-ACC-SW2]
```

| Site | Prefix | Notes |
|------|--------|-------|
| HQ users | 10.10.10.0/24 (VLAN 10) | |
| HQ servers | 10.10.20.0/24 (VLAN 20) | |
| HQ voice | 10.10.30.0/24 (VLAN 30) | |
| HQ mgmt | 10.10.99.0/24 (VLAN 99) | OOB/management |
| Branch 1 | 10.20.0.0/16 | |
| Branch 2 | 10.30.0.0/16 | |
| Loopbacks | 1.1.1.1 HQ-R1, 2.2.2.2 HQ-R2, 11.11.11.11 BR1, 22.22.22.22 BR2 | /32, router-IDs |
| Tunnels | 172.16.255.0/24 | /30 per P2P tunnel |
| Native/blackhole VLAN | 999 | unused |
| AS numbers | HQ 65001, BR1 65002, ISP-A 64500, ISP-B 64501 | private ASNs |
