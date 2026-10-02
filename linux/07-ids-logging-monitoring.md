# IDS/IPS, Logging & Monitoring (Suricata, Wazuh, rsyslog, Zabbix)

## Suricata
```bash
apt install suricata ; suricata-update ; suricata-update list-sources
# /etc/suricata/suricata.yaml -> af-packet: interface: eth0 ; HOME_NET: "[10.10.0.0/16]"
suricata -T -c /etc/suricata/suricata.yaml        # config test
systemctl enable --now suricata
tail -f /var/log/suricata/fast.log
jq 'select(.event_type=="alert")' /var/log/suricata/eve.json
# Custom rule: /etc/suricata/rules/local.rules
alert tcp any any -> $HOME_NET 22 (msg:"SSH brute force"; flow:to_server; detection_filter:track by_src, count 5, seconds 60; sid:1000001; rev:1;)
alert icmp any any -> $HOME_NET any (msg:"ICMP large"; dsize:>800; sid:1000002; rev:1;)
# IPS inline (NFQUEUE)
nft add rule inet filter forward queue num 0
suricata -q 0
```
## Wazuh agent
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
WAZUH_MANAGER="10.10.99.70" apt install wazuh-agent
systemctl enable --now wazuh-agent
/var/ossec/bin/agent_control -l
```
## rsyslog forwarding
```
# /etc/rsyslog.d/50-forward.conf
*.* action(type="omfwd" target="10.10.99.70" port="514" protocol="tcp" action.resumeRetryCount="-1" queue.type="LinkedList" queue.size="10000")
```
`rsyslogd -N1 && systemctl restart rsyslog`

## Zabbix/SNMP quick tests
```bash
snmpwalk -v3 -l authPriv -u zabbix -a SHA -A <AUTHPASS> -x AES -X <PRIVPASS> 10.10.99.1 sysDescr
```
## Journald / log hunting
```bash
journalctl -u sshd --since "1 hour ago" | grep -i "failed"
journalctl -k | grep -i nft-drop
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head
```
