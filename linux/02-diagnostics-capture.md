# Diagnostics, Scanning & Packet Capture

## Reachability / path
```bash
ping -c4 10.10.10.1
ping -M do -s 1472 8.8.8.8          # path-MTU test
traceroute -n 8.8.8.8; tracepath 8.8.8.8
mtr -rwzbc 100 8.8.8.8
curl -vk https://host --resolve host:443:10.10.20.10
curl -I https://example.com
openssl s_client -connect host:443 -servername host </dev/null | openssl x509 -noout -dates -subject
nc -zv host 22 443                  # port test
iperf3 -s   /   iperf3 -c server -P 4 -t 30
```
## tcpdump
```bash
tcpdump -ni eth0 host 10.10.10.5 and port 443
tcpdump -ni any icmp
tcpdump -ni eth0 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'
tcpdump -ni eth0 -w /tmp/cap.pcap -C 100 -W 5      # ring buffer
tcpdump -ni eth0 'vlan and port 53'
tcpdump -nnvvXSs0 -i eth0 arp
tcpdump -r cap.pcap -nn 'src net 10.10.10.0/24 and dst port 22'
```
## tshark / Wireshark filters
```bash
tshark -r cap.pcap -Y "dns.qry.name contains example" -T fields -e ip.src -e dns.qry.name
tshark -r cap.pcap -q -z conv,tcp
tshark -r cap.pcap -Y "tls.handshake.type==1" -T fields -e tls.handshake.extensions_server_name
```
Display filters: `ip.addr==10.10.10.5`, `tcp.flags.syn==1 && tcp.flags.ack==0`, `http.request`, `arp.duplicate-address-detected`, `ospf`, `bgp`.

## Nmap (authorized scanning only)
```bash
nmap -sn 10.10.10.0/24                          # host discovery
nmap -sS -sV -O -p- --min-rate 1000 -oA full 10.10.20.10
nmap -sU --top-ports 50 10.10.20.10
nmap -Pn -p 22,80,443 --script ssh2-enum-algos,ssl-enum-ciphers 10.10.20.10
nmap --script vuln 10.10.20.10
nmap -sV --script smb-security-mode,smb-protocols 10.10.20.0/24
nmap -p 161 -sU --script snmp-info 10.10.99.0/24
nmap -iL targets.txt -T3 -oG - | grep open
```
## Other
```bash
arp-scan -l                                    # L2 discovery
arpwatch -i eth0                               # detect ARP changes
masscan 10.10.0.0/16 -p80,443 --rate 1000
openvas / gvm-start                            # vulnerability scanning
ss -tnp state established
lsof -i -P -n
iftop -i eth0 ; nethogs eth0 ; nload
```
## Network attack simulation (LAB ONLY - to validate defenses)
```bash
# Validate DHCP snooping/DAI/port-security alarms in your own lab
yersinia -I                       # L2 protocol testing
ettercap -T -M arp:remote /10.10.10.1// /10.10.10.5//     # lab ARP-spoof test (should be blocked by DAI)
hping3 -S -p 80 --flood 10.10.20.10                       # test storm/CoPP limits
```
