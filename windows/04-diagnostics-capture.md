# Diagnostics & Packet Capture on Windows

```powershell
# Built-in capture (no Wireshark needed)
pktmon filter add -p 443
pktmon start --capture --pkt-size 0 -f C:\cap.etl
pktmon stop
pktmon etl2pcap C:\cap.etl --out C:\cap.pcapng
pktmon list ; pktmon counters

netsh trace start capture=yes tracefile=C:\trace.etl maxsize=512 persist=no
netsh trace stop

# Port/process mapping
netstat -anob
Get-NetTCPConnection -State Listen | select LocalAddress,LocalPort,@{n='Proc';e={(Get-Process -Id $_.OwningProcess).Name}}
Get-NetUDPEndpoint | sort LocalPort

# DNS / DHCP / ARP
nslookup -type=any corp.local 10.10.20.53
Resolve-DnsName _ldap._tcp.dc._msdcs.corp.local -Type SRV
nltest /dsgetdc:corp.local ; nltest /sc_verify:corp.local
arp -d * ; netsh interface ip delete arpcache

# Perf / counters
Get-Counter '\Network Interface(*)\Bytes Total/sec' -SampleInterval 2 -MaxSamples 5
Get-NetAdapterStatistics
Get-NetAdapterQos
# Scan (authorized)
nmap -sV -p- 10.10.20.10      # same Nmap syntax as Linux
```
Events: `Get-WinEvent -LogName System -MaxEvents 50 | ? ProviderName -match 'Tcpip|NDIS|Dhcp'`
