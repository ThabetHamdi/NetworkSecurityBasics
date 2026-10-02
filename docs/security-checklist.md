# Network Security Checklist

## Device / Control plane
- [ ] SSHv2 only, Telnet/HTTP disabled, VTY ACL, `exec-timeout`
- [ ] AAA (TACACS+/RADIUS) + local fallback, command authorization, accounting
- [ ] `service password-encryption`, `enable algorithm-type scrypt secret`
- [ ] NTP authenticated, Syslog to SIEM, SNMPv3 authPriv only
- [ ] CoPP enabled; unused services off (`no ip http server`, `no cdp` on edge, `no ip source-route`)
- [ ] Login banner, config backups, archive/rollback

## Layer 2
- [ ] DTP off (`switchport nonegotiate`), native VLAN ≠ 1, unused ports shutdown in blackhole VLAN
- [ ] PortFast + BPDU Guard on edge, Root Guard on uplinks to untrusted
- [ ] DHCP Snooping + DAI + IP Source Guard, Port-Security / 802.1X
- [ ] Storm-control, VTP off/transparent (or v3 with password)

## Layer 3
- [ ] Routing protocol auth (MD5/SHA/HMAC-SHA-256), passive-interface default
- [ ] BGP: prefix-lists, max-prefix, TTL-security, MD5/TCP-AO, bogon filter
- [ ] uRPF, infrastructure ACLs, anti-spoofing, BFD for fast failure detection
- [ ] IPsec: IKEv2, AES-256, SHA-256+, DH group 19/20/21, PFS

## Hosts
- [ ] Default-deny host firewall (nftables / Windows Firewall)
- [ ] SMBv1, LLMNR, NBT-NS, mDNS disabled; NLA for RDP; WinRM over HTTPS
- [ ] SSH key-only auth, fail2ban/CrowdSec, auditd / Sysmon
- [ ] Central logging, IDS/IPS (Suricata), vuln scanning (OpenVAS/Nmap)
