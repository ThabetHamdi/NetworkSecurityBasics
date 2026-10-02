# Linux Networking Fundamentals (iproute2 / NetworkManager / netplan)

## Scenario
Linux gateway/server with VLANs, bonding, bridge and static routes. Commands are runtime; persist with NetworkManager or netplan.

## Inspect
```bash
ip -br addr                    # brief addresses
ip -br link
ip route; ip -6 route
ip neigh                       # ARP/NDP table
ip rule                        # policy routing rules
ss -tulpn                      # listening sockets + process
ss -s                          # socket summary
nmcli device status; nmcli con show
resolvectl status
cat /etc/resolv.conf
```
## Addressing / routes
```bash
ip addr add 10.10.10.5/24 dev eth0
ip addr del 10.10.10.5/24 dev eth0
ip link set eth0 up mtu 1500
ip route add 10.20.0.0/16 via 10.10.10.1 dev eth0
ip route add default via 10.10.10.1
ip route replace default via 10.10.10.1 metric 100
ip route add blackhole 192.0.2.0/24
sysctl -w net.ipv4.ip_forward=1
```
## VLAN / bridge / bond
```bash
ip link add link eth0 name eth0.10 type vlan id 10
ip addr add 10.10.10.2/24 dev eth0.10 && ip link set eth0.10 up

ip link add br0 type bridge vlan_filtering 1
ip link set eth1 master br0
bridge vlan add dev eth1 vid 10 pvid untagged
bridge vlan show

ip link add bond0 type bond mode 802.3ad miimon 100 lacp_rate fast xmit_hash_policy layer3+4
ip link set eth2 down; ip link set eth2 master bond0
ip link set eth3 down; ip link set eth3 master bond0
ip link set bond0 up
cat /proc/net/bonding/bond0
```
## Policy routing (multi-ISP)
```bash
echo "100 ispa" >> /etc/iproute2/rt_tables
ip route add default via 198.51.100.2 table ispa
ip rule add from 198.51.100.1 lookup ispa
ip rule add fwmark 0x1 lookup ispa           # mark with nftables
```
## NetworkManager (persistent)
```bash
nmcli con add type ethernet ifname eth0 con-name LAN ipv4.addresses 10.10.10.5/24 ipv4.gateway 10.10.10.1 ipv4.dns 10.10.20.53 ipv4.method manual
nmcli con add type vlan ifname eth0.10 dev eth0 id 10 ipv4.method manual ipv4.addresses 10.10.10.2/24
nmcli con mod LAN +ipv4.routes "10.20.0.0/16 10.10.10.1"
nmcli con up LAN
```
## Netplan (Ubuntu) /etc/netplan/01-net.yaml
```yaml
network:
  version: 2
  ethernets:
    eth0: {}
  vlans:
    vlan10:
      id: 10
      link: eth0
      addresses: [10.10.10.5/24]
      routes:
        - to: default
          via: 10.10.10.1
      nameservers:
        addresses: [10.10.20.53]
```
`netplan try` then `netplan apply`.

## Name resolution / misc
```bash
dig +short A example.com @10.10.20.53
dig +trace example.com
host -t MX example.com
getent hosts server1
ethtool eth0; ethtool -S eth0
ip -s link show eth0
```
