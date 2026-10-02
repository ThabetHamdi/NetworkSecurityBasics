# Windows Network Hardening

## Disable legacy / weak protocols
```powershell
# SMBv1 off, SMB signing & encryption on
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
Set-SmbServerConfiguration -EnableSMB1Protocol $false -RequireSecuritySignature $true -EncryptData $true -Force
Set-SmbClientConfiguration -RequireSecuritySignature $true -Force
Get-SmbServerConfiguration | select EnableSMB1Protocol,RequireSecuritySignature,EncryptData

# LLMNR off
New-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Force | Out-Null
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" EnableMulticast 0 -Type DWord
# NetBIOS over TCP/IP off
Get-CimInstance Win32_NetworkAdapterConfiguration -Filter "IPEnabled=true" | Invoke-CimMethod -MethodName SetTcpipNetbios -Arguments @{TcpipNetbiosOptions=2}
# mDNS off
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters" EnableMDNS 0 -Type DWord
# WPAD / auto-proxy off
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp" DisableWpad 1 -Type DWord -ErrorAction SilentlyContinue
# NTLM: send NTLMv2 only, refuse LM/NTLMv1
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" LmCompatibilityLevel 5 -Type DWord
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" NoLMHash 1 -Type DWord
# LSA protection
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" RunAsPPL 1 -Type DWord
# LDAP signing / channel binding (DC): set via GPO "Domain controller: LDAP server signing requirements = Require signing"
# IPv6 only if unused: prefer disabling per-binding rather than registry global
Disable-NetAdapterBinding -Name "Ethernet" -ComponentID ms_tcpip6
```
## TLS / protocols (SCHANNEL)
```powershell
$p = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols"
foreach ($proto in "SSL 2.0","SSL 3.0","TLS 1.0","TLS 1.1") {
  foreach ($side in "Server","Client") {
    New-Item "$p\$proto\$side" -Force | Out-Null
    New-ItemProperty "$p\$proto\$side" -Name Enabled -Value 0 -PropertyType DWord -Force | Out-Null
    New-ItemProperty "$p\$proto\$side" -Name DisabledByDefault -Value 1 -PropertyType DWord -Force | Out-Null
  }
}
```
## RDP
```powershell
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" UserAuthentication 1   # NLA
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" SecurityLayer 2       # TLS
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" MinEncryptionLevel 3
# Never expose 3389 to Internet: use VPN / RD Gateway / Bastion
```
## WinRM / PowerShell remoting over HTTPS
```powershell
$cert = New-SelfSignedCertificate -DnsName $env:COMPUTERNAME -CertStoreLocation Cert:\LocalMachine\My
New-Item WSMan:\localhost\Listener -Transport HTTPS -Address * -CertificateThumbPrint $cert.Thumbprint -Force
Set-Item WSMan:\localhost\Service\AllowUnencrypted $false
Set-Item WSMan:\localhost\Service\Auth\Basic $false
Enable-PSRemoting -SkipNetworkProfileCheck
# PowerShell logging
New-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force | Out-Null
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" EnableScriptBlockLogging 1
```
## Defender / ASR / audit
```powershell
Get-MpComputerStatus
Set-MpPreference -EnableNetworkProtection Enabled -PUAProtection Enabled -MAPSReporting Advanced
Add-MpPreference -AttackSurfaceReductionRules_Ids 56a863a9-875e-4185-98a7-b882c64b5ce5 -AttackSurfaceReductionRules_Actions Enabled  # block abused vulnerable drivers
Update-MpSignature ; Start-MpScan -ScanType QuickScan
Get-MpThreatDetection
auditpol /get /category:*
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```
## Baselines & quick audits
```powershell
Get-Service | ? {$_.StartType -eq 'Automatic' -and $_.Status -ne 'Running'}
Get-LocalUser ; Get-LocalGroupMember Administrators
Get-ScheduledTask | ? State -ne Disabled | select TaskName,TaskPath
Get-HotFix | sort InstalledOn -desc | select -first 10
secedit /export /cfg C:\sec.cfg
```
Use Microsoft Security Baselines / CIS Benchmarks via GPO or Intune; verify with Microsoft Security Compliance Toolkit (LGPO.exe, Policy Analyzer).
