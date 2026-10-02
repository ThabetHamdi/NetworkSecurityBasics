# Windows Networking with PowerShell / CMD

## Inspect
```powershell
Get-NetIPConfiguration
Get-NetAdapter | ft Name,Status,LinkSpeed,MacAddress
Get-NetIPAddress -AddressFamily IPv4
Get-NetRoute -AddressFamily IPv4
Get-NetNeighbor                          # ARP table
Get-DnsClientServerAddress
Get-NetTCPConnection -State Listen | sort LocalPort
Get-NetTCPConnection -State Established | select LocalAddress,LocalPort,RemoteAddress,RemotePort,OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 443).OwningProcess
ipconfig /all ; route print ; arp -a ; netstat -ano ; nbtstat -n
```
## Configure
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.20.10 -PrefixLength 24 -DefaultGateway 10.10.20.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.20.53,10.10.20.54
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
New-NetRoute -DestinationPrefix 10.20.0.0/16 -InterfaceAlias "Ethernet" -NextHop 10.10.20.1
Remove-NetRoute -DestinationPrefix 10.20.0.0/16 -Confirm:$false
Rename-NetAdapter -Name "Ethernet" -NewName "LAN"
Set-NetAdapterAdvancedProperty -Name LAN -DisplayName "Jumbo Packet" -DisplayValue "9014"
# VLAN tag / NIC teaming (LBFO deprecated on client, SET on Hyper-V)
Set-NetAdapter -Name LAN -VlanID 10
New-NetLbfoTeam -Name TEAM1 -TeamMembers NIC1,NIC2 -TeamingMode LACP -LoadBalancingAlgorithm Dynamic
New-VMSwitch -Name SET -NetAdapterName NIC1,NIC2 -EnableEmbeddedTeaming $true
# Network profile & discovery
Set-NetConnectionProfile -InterfaceAlias LAN -NetworkCategory Private
ipconfig /flushdns ; ipconfig /release ; ipconfig /renew ; netsh winsock reset ; netsh int ip reset
```
## DNS client
```powershell
Resolve-DnsName example.com -Type A -Server 10.10.20.53
Clear-DnsClientCache
Get-DnsClientCache
Add-DnsClientNrptRule -Namespace ".corp.local" -NameServers 10.10.20.53
```
## Test connectivity
```powershell
Test-NetConnection 10.10.20.10 -Port 443
Test-NetConnection 8.8.8.8 -TraceRoute
1..254 | % { Test-Connection "10.10.10.$_" -Count 1 -Quiet -TimeoutSeconds 1 }
pathping 8.8.8.8 ; tracert -d 8.8.8.8
```
