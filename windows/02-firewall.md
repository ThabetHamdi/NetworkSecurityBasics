# Windows Defender Firewall (PowerShell / netsh)

## Scenario
Server baseline: all profiles ON, inbound default-block, only required ports from management/LAN subnets, logging enabled.

```powershell
# Profiles
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True `
  -DefaultInboundAction Block -DefaultOutboundAction Allow `
  -LogAllowed False -LogBlocked True -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log" -LogMaxSizeKilobytes 16384

# Rules
New-NetFirewallRule -DisplayName "Allow RDP from Mgmt" -Direction Inbound -Protocol TCP -LocalPort 3389 -RemoteAddress 10.10.99.0/24 -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "Allow HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
New-NetFirewallRule -DisplayName "Allow WinRM HTTPS" -Direction Inbound -Protocol TCP -LocalPort 5986 -RemoteAddress 10.10.99.0/24 -Action Allow
New-NetFirewallRule -DisplayName "Block Telnet Out" -Direction Outbound -Protocol TCP -RemotePort 23 -Action Block
New-NetFirewallRule -DisplayName "Block SMB Out to Internet" -Direction Outbound -Protocol TCP -RemotePort 445 -RemoteAddress Internet -Action Block
New-NetFirewallRule -DisplayName "App rule" -Program "C:\App\app.exe" -Direction Inbound -Action Allow

# Manage
Get-NetFirewallRule -Enabled True -Direction Inbound | ft DisplayName,Profile,Action
Get-NetFirewallRule -DisplayName "Allow RDP*" | Get-NetFirewallPortFilter
Disable-NetFirewallRule -DisplayGroup "File and Printer Sharing"
Remove-NetFirewallRule -DisplayName "Block Telnet Out"
Export: netsh advfirewall export "C:\fw-backup.wfw"
Import: netsh advfirewall import "C:\fw-backup.wfw"
```
## netsh equivalents
```cmd
netsh advfirewall show allprofiles
netsh advfirewall set allprofiles state on
netsh advfirewall firewall add rule name="Allow SSH" dir=in action=allow protocol=TCP localport=22 remoteip=10.10.99.0/24
netsh advfirewall firewall delete rule name="Allow SSH"
netsh advfirewall reset
```
## Logs to hunt
`C:\Windows\System32\LogFiles\Firewall\pfirewall.log`  |  Event IDs **5152/5157** (blocked) via `auditpol /set /subcategory:"Filtering Platform Connection" /failure:enable`.

Manage centrally with GPO: *Computer Config → Windows Settings → Security Settings → Windows Defender Firewall with Advanced Security*.
