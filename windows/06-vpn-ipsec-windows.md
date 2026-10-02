# Windows VPN & IPsec

## Always-On / IKEv2 client VPN
```powershell
Add-VpnConnection -Name "CorpVPN" -ServerAddress vpn.corp.example -TunnelType IKEv2 `
  -AuthenticationMethod MachineCertificate -EncryptionLevel Maximum -SplitTunneling $false -RememberCredential $true
Set-VpnConnectionIPsecConfiguration -ConnectionName "CorpVPN" -AuthenticationTransformConstants GCMAES256 `
  -CipherTransformConstants GCMAES256 -EncryptionMethod AES256 -IntegrityCheckMethod SHA256 -DHGroup ECP256 -PfsGroup ECP256 -Force
Add-VpnConnectionRoute -ConnectionName "CorpVPN" -DestinationPrefix 10.10.0.0/16
rasdial CorpVPN ; Get-VpnConnection
```
## Server-to-server IPsec policy (Windows Firewall IPsec rules)
```powershell
$ph1 = New-NetIPsecPhase1AuthSet -DisplayName "P1" -Proposal (New-NetIPsecAuthProposal -Machine -Kerberos)
$ph2 = New-NetIPsecPhase2AuthSet -DisplayName "P2" -Proposal (New-NetIPsecAuthProposal -User -Kerberos)
New-NetIPsecRule -DisplayName "Require IPsec SQL" -InboundSecurity Require -OutboundSecurity Require `
  -LocalPort 1433 -Protocol TCP -RemoteAddress 10.10.10.0/24 -Phase1AuthSet $ph1.Name
Get-NetIPsecRule ; Get-NetIPsecMainModeSA ; Get-NetIPsecQuickModeSA
```
## Site-to-site with Cisco (RRAS)
```powershell
Install-WindowsFeature RemoteAccess,Routing -IncludeManagementTools
Install-RemoteAccess -VpnType VpnS2S
Add-VpnS2SInterface -Name BranchHQ -Protocol IKEv2 -Destination 198.51.100.1 -AuthenticationMethod PSKOnly -SharedSecret <CHANGE-ME> `
  -IPv4Subnet @("10.10.0.0/16:100") -NumberOfTries 0
```
Match IKE/IPsec parameters with the Cisco side (`cisco/03-vpn/02-ipsec-site-to-site.md`).

## Verification
```powershell
Get-VpnConnectionTrigger
Get-WinEvent -LogName "Microsoft-Windows-Security-Auditing" -MaxEvents 20 | ? Id -in 4653,4654,4655,4656
netsh advfirewall monitor show mmsa
netsh advfirewall monitor show qmsa
```
