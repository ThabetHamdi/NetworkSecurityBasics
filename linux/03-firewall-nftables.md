# Linux Firewall: nftables (primary), iptables, ufw, firewalld

## Scenario
Linux server/gateway: default-drop inbound, allow SSH from management subnet, rate-limit, log drops, NAT for LAN.

## /etc/nftables.conf
```nft
#!/usr/sbin/nft -f
flush ruleset

define MGMT = 10.10.99.0/24
define LAN  = 10.10.10.0/24
define WAN_IF = "eth0"
define LAN_IF = "eth1"

table inet filter {
  set blocklist { type ipv4_addr; flags timeout; }

  chain input {
    type filter hook input priority 0; policy drop;
    iif lo accept
    ct state established,related accept
    ct state invalid drop
    ip saddr @blocklist drop
    ip protocol icmp icmp type { echo-request, destination-unreachable, time-exceeded } limit rate 10/second accept
    ip6 nexthdr icmpv6 accept
    ip saddr $MGMT tcp dport 22 ct state new limit rate 5/minute accept
    tcp dport 22 ct state new add @blocklist { ip saddr timeout 1h } drop  # excess brute force
    tcp dport { 80, 443 } accept
    udp dport 51820 accept            # WireGuard
    udp dport { 500, 4500 } accept ; ip protocol esp accept   # IPsec
    limit rate 5/second log prefix "nft-drop-in: " flags all
  }
  chain forward {
    type filter hook forward priority 0; policy drop;
    ct state established,related accept
    iifname $LAN_IF oifname $WAN_IF accept
    iifname $LAN_IF ip daddr 10.10.99.0/24 drop
  }
  chain output { type filter hook output priority 0; policy accept; }
}
table ip nat {
  chain postrouting {
    type nat hook postrouting priority 100;
    oifname $WAN_IF masquerade
  }
  chain prerouting {
    type nat hook prerouting priority -100;
    iifname $WAN_IF tcp dport 8443 dnat to 10.10.20.10:443
  }
}
```
```bash
nft -c -f /etc/nftables.conf          # syntax check
nft -f /etc/nftables.conf
systemctl enable --now nftables
nft list ruleset
nft list table inet filter -a
nft add rule inet filter input tcp dport 8080 accept
nft delete rule inet filter input handle 12
```
## iptables (legacy)
```bash
iptables -P INPUT DROP; iptables -P FORWARD DROP; iptables -P OUTPUT ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp -s 10.10.99.0/24 --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "IPT-DROP: "
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -L -n -v --line-numbers
iptables-save > /etc/iptables/rules.v4
```
## ufw / firewalld
```bash
ufw default deny incoming; ufw default allow outgoing
ufw allow from 10.10.99.0/24 to any port 22 proto tcp
ufw limit 22/tcp ; ufw enable ; ufw status verbose

firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.10.99.0/24 service name=ssh accept'
firewall-cmd --permanent --remove-service=cockpit
firewall-cmd --reload ; firewall-cmd --list-all
```
