# Windows Logging & Detection (Security-relevant)

## Key Event IDs
| ID | Meaning |
|---|---|
| 4624 / 4625 | Logon success / failure (LogonType 3 = network, 10 = RDP) |
| 4648 | Explicit-credential logon |
| 4672 | Special privileges assigned |
| 4688 | Process creation (enable command-line auditing) |
| 4697 / 7045 | Service installed |
| 4720 / 4728 / 4732 | User created / added to privileged group |
| 4768 / 4769 / 4771 | Kerberos TGT / service ticket / pre-auth failure (Kerberoasting: 4769 RC4) |
| 4776 | NTLM authentication |
| 5140 / 5145 | Network share access |
| 1102 | Audit log cleared |
| 5152 / 5157 | Firewall blocked packet / connection |

## Enable useful auditing
```powershell
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f
wevtutil sl Security /ms:1073741824              # 1 GB log
```
## Hunting with PowerShell
```powershell
# Failed logons by source IP (brute force)
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625;StartTime=(Get-Date).AddHours(-1)} |
 % { $_.Properties[19].Value } | group | sort Count -desc | select -first 10
# RDP logons
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4624} | ? { $_.Properties[8].Value -eq 10 } | select TimeCreated
# New services
Get-WinEvent -FilterHashtable @{LogName='System';Id=7045} | select TimeCreated,Message -first 10
```
## Sysmon (network connections = Event ID 3)
```powershell
sysmon64 -accepteula -i sysmonconfig.xml
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -FilterXPath "*[System[EventID=3]]" -MaxEvents 20
```
Use a community config (SwiftOnSecurity / Olaf Hartong modular) as a starting point.
## Forwarding
```powershell
wecutil qc ; winrm quickconfig            # Windows Event Collector / WEF
```
Or ship with the Wazuh agent / Azure Monitor Agent / Defender for Endpoint to a SIEM (KQL examples: `DeviceNetworkEvents | where ActionType == "ConnectionFailed"`).
