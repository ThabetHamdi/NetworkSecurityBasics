# DIY SD-WAN with IOS (DMVPN + IP SLA + PBR + Ansible)

## Scenario
No SD-WAN license: build an SD-WAN-like overlay with two DMVPN clouds (MPLS & Internet), SLA-based path selection with IP SLA/PBR, and automated provisioning through Ansible.

## Overlay design
- Tunnel100 -> DMVPN over ISP-A (`172.16.100.0/24`), Tunnel200 -> DMVPN over ISP-B (`172.16.200.0/24`). See `../03-vpn/03-dmvpn.md` for each cloud.
- Routing: eBGP/iBGP or EIGRP over both tunnels, prefer cloud 1 via metric/local-pref.

## SLA-based steering (spoke)
```
ip sla 100
 udp-jitter 172.16.100.1 16384 codec g711ulaw
 frequency 10
ip sla schedule 100 life forever start-time now
ip sla 200
 udp-jitter 172.16.200.1 16384 codec g711ulaw
 frequency 10
ip sla schedule 200 life forever start-time now
track 100 ip sla 100 reachability
track 200 ip sla 200 reachability

ip access-list extended VOICE
 permit udp any any range 16384 32767
route-map PBR-SDWAN permit 10
 match ip address VOICE
 set ip next-hop verify-availability 172.16.100.1 1 track 100
 set ip next-hop verify-availability 172.16.200.1 2 track 200
route-map PBR-SDWAN permit 20
 set ip next-hop 172.16.200.1               ! bulk over Internet
interface GigabitEthernet0/1
 ip policy route-map PBR-SDWAN
```
## Performance Routing alternative
Use **PfRv3** (`domain default`, `master hub`, `border`) for built-in SLA monitoring on IOS-XE.
```
domain default
 vrf default
  master hub
   source-interface Loopback0
   site-prefixes prefix-list SITE-PFX
  collector 10.10.99.80 port 9995
```
## Ansible provisioning (inventory + playbook)
```yaml
# inventory.yml
all:
  children:
    spokes:
      hosts:
        br1: { ansible_host: 172.16.255.2 }
        br2: { ansible_host: 172.16.255.6 }
  vars:
    ansible_network_os: cisco.ios.ios
    ansible_connection: network_cli
    ansible_user: admin
    ansible_password: "{{ vault_pass }}"
```
```yaml
# deploy-sdwan.yml
- hosts: spokes
  gather_facts: false
  tasks:
    - name: Push SLA probes
      cisco.ios.ios_config:
        lines:
          - ip sla 100
          - udp-jitter 172.16.100.1 16384 codec g711ulaw
          - frequency 10
        parents: []
    - name: Backup config
      cisco.ios.ios_config:
        backup: true
```
Run: `ansible-playbook -i inventory.yml deploy-sdwan.yml --ask-vault-pass`
## Verification
```
show ip sla statistics
show track
show ip policy
show dmvpn
```
