# Windows Server Network Roles: AD DS, DNS, DHCP, NPS

## Scenario
Windows Server 2022 DC at HQ provides AD DS + DNS + DHCP; NPS provides RADIUS for 802.1X/VPN.

## AD DS + DNS
```powershell
Install-WindowsFeature AD-Domain-Services,DNS -IncludeManagementTools
Install-ADDSForest -DomainName corp.local -DomainNetbiosName CORP -InstallDns -SafeModeAdministratorPassword (Read-Host -AsSecureString)
Get-ADDomainController ; dcdiag /v ; repadmin /replsummary
# DNS
Add-DnsServerPrimaryZone -Name "lab.corp.local" -ReplicationScope Domain
Add-DnsServerResourceRecordA -Name srv1 -ZoneName corp.local -IPv4Address 10.10.20.10
Add-DnsServerForwarder -IPAddress 1.1.1.1,9.9.9.9
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00
Set-DnsServerDiagnostics -All $true          # troubleshooting only
Add-DnsServerResponseRateLimiting -Mode Enable
```
Hardening: secure-only dynamic updates (`Set-DnsServerPrimaryZone -DynamicUpdate Secure`), restrict zone transfers (`-SecureSecondaries TransferToZoneNameServer`), enable DNSSEC where applicable.

## DHCP
```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName dc1.corp.local -IPAddress 10.10.20.53
Add-DhcpServerv4Scope -Name USERS -StartRange 10.10.10.50 -EndRange 10.10.10.250 -SubnetMask 255.255.255.0 -LeaseDuration 8:00:00
Set-DhcpServerv4OptionValue -ScopeId 10.10.10.0 -Router 10.10.10.1 -DnsServer 10.10.20.53 -DnsDomain corp.local
Add-DhcpServerv4ExclusionRange -ScopeId 10.10.10.0 -StartRange 10.10.10.1 -EndRange 10.10.10.20
Add-DhcpServerv4Reservation -ScopeId 10.10.10.0 -IPAddress 10.10.10.30 -ClientId "AA-BB-CC-DD-EE-FF"
Add-DhcpServerv4Failover -ComputerName dc1 -PartnerServer dc2 -Name HQ-FO -ScopeId 10.10.10.0 -LoadBalancePercent 50 -SharedSecret <CHANGE-ME>
Get-DhcpServerv4Lease -ScopeId 10.10.10.0
Set-DhcpServerv4DnsSetting -DynamicUpdates OnClientRequest -DeleteDnsRRonLeaseExpiry $true
```
## NPS (RADIUS)
```powershell
Install-WindowsFeature NPAS -IncludeManagementTools
New-NpsRadiusClient -Name "SW1" -Address 10.10.99.11 -SharedSecret <CHANGE-ME>
netsh nps show config
```
Cisco side: see `cisco/04-security/03-aaa-tacacs.md` and `cisco/01-switching/06-l2-security.md`.

## Group Policy
```powershell
gpupdate /force ; gpresult /r ; gpresult /h C:\gp.html
Get-GPO -All | select DisplayName,GpoStatus
```
