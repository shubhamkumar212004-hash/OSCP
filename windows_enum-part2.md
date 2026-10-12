# Windows Privilege Escalation Enumeration Cheat Sheet

A practical Windows privilege escalation enumeration checklist for authorized penetration testing, OSCP preparation, and CTF labs.

> **Scope:** Use only on systems you own or are explicitly authorized to assess.

## Table of Contents

- [1. System Information](#1-system-information)
- [2. Current User and Privileges](#2-current-user-and-privileges)
- [3. Users and Groups](#3-users-and-groups)
- [4. Network Enumeration](#4-network-enumeration)
- [5. Processes and Services](#5-processes-and-services)
- [6. Service Misconfiguration Checks](#6-service-misconfiguration-checks)
- [7. Files and Permissions](#7-files-and-permissions)
- [8. Registry Enumeration](#8-registry-enumeration)
- [9. Scheduled Tasks](#9-scheduled-tasks)
- [10. PowerShell Enumeration](#10-powershell-enumeration)
- [11. Credential and Secret Hunting](#11-credential-and-secret-hunting)
- [12. AutoLogon and AlwaysInstallElevated](#12-autologon-and-alwaysinstallelevated)
- [13. Token Privileges](#13-token-privileges)
- [14. Patch and Security Enumeration](#14-patch-and-security-enumeration)
- [15. Startup and Persistence Locations](#15-startup-and-persistence-locations)
- [16. UAC and Security Controls](#16-uac-and-security-controls)
- [17. Domain Enumeration](#17-domain-enumeration)
- [18. PowerShell History and Environment](#18-powershell-history-and-environment)
- [19. Enumeration Tools](#19-enumeration-tools)
- [20. Privilege Escalation Checklist](#20-privilege-escalation-checklist)

---

## 1. System Information

Identify the operating system, architecture, hostname, domain, and installed updates.

### CMD

```cmd
systeminfo
hostname
ver
whoami
whoami /all
echo %username%
echo %userdomain%
echo %computername%
echo %USERPROFILE%
net time /domain
```

### Check OS and architecture

```cmd
wmic os get Caption,Version,BuildNumber,OSArchitecture
wmic computersystem get Domain,PartOfDomain,Manufacturer,Model
```

> Note: WMIC is deprecated on newer Windows versions. Use PowerShell CIM commands where available.

```powershell
Get-ComputerInfo
Get-CimInstance Win32_OperatingSystem |
    Select-Object Caption, Version, BuildNumber, OSArchitecture

Get-CimInstance Win32_ComputerSystem |
    Select-Object Domain, PartOfDomain, Manufacturer, Model
```

**Look for:**
- Outdated Windows builds and missing security updates.
- Unusual domain membership or hostname information.
- Virtual machines and server roles.
- Your current security context.

## 2. Current User and Privileges

```cmd
whoami
whoami /user
whoami /groups
whoami /priv
whoami /all
```

### Check local administrator membership

```cmd
net localgroup Administrators
```

### Check effective access to a directory

```cmd
icacls C:\ProgramData
icacls C:\Windows\Temp
```

**Important:** A listed privilege or group membership does not automatically mean it can be exploited. Check whether the privilege is enabled, whether the account has an applicable token, and whether Windows mitigations apply.

## 3. Users and Groups

### CMD

```cmd
net user
net user <username>
net localgroup
net localgroup Administrators
net accounts
```

### Domain users and groups

```cmd
net user /domain
net group /domain
net group "Domain Admins" /domain
net group "Domain Users" /domain
net group "Enterprise Admins" /domain
```

### WMIC enumeration

```cmd
wmic useraccount list full
wmic group list brief
```

### PowerShell

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"
Get-LocalGroupMember -Group "Remote Desktop Users"
Get-LocalGroupMember -Group "Remote Management Users"
```

**Look for:**
- Unexpected local administrators.
- Service accounts and dormant accounts.
- Groups with access to RDP, WinRM, or administrative tools.
- Accounts with privileges beyond their intended roles.

## 4. Network Enumeration

```cmd
ipconfig /all
ipconfig /displaydns
netstat -ano
netstat -anob
arp -a
route print
net share
nbtstat -n
nbtstat -A <IP>
```

### PowerShell

```powershell
Get-NetIPConfiguration
Get-NetIPAddress
Get-NetRoute
Get-NetTCPConnection
Get-NetUDPEndpoint
Get-NetNeighbor
```

### Identify processes listening on network ports

```cmd
netstat -ano
tasklist /fi "PID eq <PID>"
```

```powershell
Get-NetTCPConnection -State Listen |
    Select-Object LocalAddress, LocalPort, OwningProcess
```

**Look for:**
- Services bound only to localhost.
- Unexpected listening ports.
- Internal interfaces and network routes.
- Shares and network connections relevant to the assessment.

## 5. Processes and Services

### Process enumeration

```cmd
tasklist
tasklist /v
tasklist /svc
wmic process list full
```

### Service enumeration

```cmd
sc query
sc queryex type= service state= all
sc qc <ServiceName>
sc query <ServiceName>
```

### PowerShell

```powershell
Get-Process
Get-Service
Get-CimInstance Win32_Service |
    Select-Object Name, DisplayName, State, StartMode, StartName, PathName
```

### Find services running as privileged accounts

```powershell
Get-CimInstance Win32_Service |
    Where-Object {$_.StartName -match 'LocalSystem|LocalService|NetworkService'} |
    Select-Object Name, State, StartName, PathName
```

**Look for:**
- Services running as `LocalSystem` or another privileged account.
- Executables in writable directories.
- Custom services and third-party applications.
- Services whose executable paths or configurations can be modified by your current user.

## 6. Service Misconfiguration Checks

### 6.1 Unquoted service paths

```powershell
Get-CimInstance Win32_Service |
    Where-Object {
        $_.PathName -match '\s' -and
        $_.PathName -notmatch '^\s*"' -and
        $_.PathName -notmatch '^\s*\\SystemRoot\\'
    } |
    Select-Object Name, StartName, PathName
```

An unquoted path containing spaces may create an opportunity for Windows to resolve the executable incorrectly. It is **not automatically exploitable**; check the actual executable path, directory write permissions, and service restart conditions.

### 6.2 Check service configuration

```cmd
sc qc <ServiceName>
sc sdshow <ServiceName>
```

Review the service binary path, account, startup type, and security descriptor.

### 6.3 Check file and directory permissions

```cmd
icacls "C:\Path\To\Service.exe"
icacls "C:\Path\To\ServiceDirectory"
```

### 6.4 Find potentially writable application directories

```powershell
Get-CimInstance Win32_Service |
    Select-Object Name, PathName, StartName
```

Inspect each relevant path manually with `icacls`.

**Potential findings:**
- Weak service configuration permissions.
- Weak permissions on service executables.
- Writable parent directories.
- Unquoted paths with exploitable resolution opportunities.
- Service accounts with excessive privileges.

## 7. Files and Permissions

### Browse hidden files and directories

```cmd
dir /a C:\
dir /a C:\ProgramData
dir /a C:\Users
dir /a "%TEMP%"
```

### Search for interesting filenames

```cmd
where /r C:\ unattend.xml
where /r C:\ web.config
where /r C:\ sysprep.inf
where /r C:\ *.kdbx
where /r C:\ *.ps1
```

Searches across an entire drive can take a long time and may encounter access-denied errors.

### Inspect permissions

```cmd
icacls C:\
icacls C:\ProgramData
icacls C:\Windows\Temp
icacls "C:\Path\To\Application"
```

### Common locations to inspect

```text
C:\Users\
C:\ProgramData\
C:\Windows\Temp\
C:\Windows\System32\Tasks\
C:\inetpub\wwwroot\
C:\xampp\htdocs\
C:\Windows\Panther\
C:\Windows\System32\sysprep\
```

### Search for configuration and deployment files

Look for:
- `unattend.xml`
- `Unattend.xml`
- `sysprep.inf`
- `web.config`
- `.config`
- `.ini`
- `.xml`
- `.ps1`
- `.bat`
- `.cmd`

Only inspect files you are authorized to access. Configuration files can contain credentials, connection strings, API keys, or other secrets.

## 8. Registry Enumeration

### Search registry values for relevant strings

```cmd
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query HKLM /f username /t REG_SZ /s
reg query HKLM /f secret /s
```

### Winlogon configuration

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

### Installed applications

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" /s
reg query "HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall" /s
```

### Service configuration

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Services" /s
```

### PowerShell

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"
```

**Look for:** insecure application configuration, sensitive values, weak registry permissions, and software that has known security issues.

## 9. Scheduled Tasks

### CMD

```cmd
schtasks /query /fo LIST /v
schtasks /query /fo TABLE
schtasks /query /tn "<TaskName>" /fo LIST /v
```

### PowerShell

```powershell
Get-ScheduledTask
Get-ScheduledTask | Where-Object {$_.TaskPath -notlike "\Microsoft*"}
```

### Check task details

```powershell
Get-ScheduledTask | ForEach-Object {
    [PSCustomObject]@{
        TaskName = $_.TaskName
        TaskPath = $_.TaskPath
        State = $_.State
        UserId = $_.Principal.UserId
        RunLevel = $_.Principal.RunLevel
        Execute = ($_.Actions.Execute -join "; ")
        Arguments = ($_.Actions.Arguments -join "; ")
    }
}
```

**Look for:**
- Tasks running with elevated privileges.
- Scripts stored in writable locations.
- Weak permissions on task executables.
- Credentials or secrets embedded in task arguments.
- Tasks that execute frequently or at system startup.

A task running as SYSTEM is not exploitable unless you can affect an executable, script, configuration, or other relevant resource used by it.

## 10. PowerShell Enumeration

```powershell
$PSVersionTable
Get-ExecutionPolicy -List
Get-Module -ListAvailable
Get-Command
Get-ChildItem Env:
$env:PATH -split ';'
[System.Environment]::OSVersion
```

### Services

```powershell
Get-Service | Where-Object {$_.Status -eq 'Running'}
```

### Local accounts and groups

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember Administrators
```

### CIM information

```powershell
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_ComputerSystem
Get-CimInstance Win32_StartupCommand
Get-CimInstance Win32_Service
```

Avoid using `Win32_Product` for routine software enumeration. Querying it can trigger Windows Installer consistency checks and potentially repair or reconfigure installed applications.

## 11. Credential and Secret Hunting

### Credential Manager

```cmd
cmdkey /list
vaultcmd /list
```

### PowerShell history

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

CMD equivalent:

```cmd
type "%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
```

### Environment variables

```cmd
set
```

```powershell
Get-ChildItem Env:
```

### Look for configuration files

```cmd
where /r C:\ web.config
where /r C:\ unattend.xml
where /r C:\ *.config
```

### Common locations

```text
%APPDATA%
%LOCALAPPDATA%
%TEMP%
C:\ProgramData\
C:\Windows\Panther\
C:\Windows\System32\sysprep\
C:\inetpub\wwwroot\
```

Check application logs, deployment scripts, backup files, and connection configurations where permitted.

**Important:** A saved credential is not necessarily an administrator credential. Validate the account, scope, and permissions without exposing secrets in your notes or GitHub output.

## 12. AutoLogon and AlwaysInstallElevated

### 12.1 AutoLogon configuration

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
```

A missing value or access-denied response does not establish whether credentials exist elsewhere.

### 12.2 AlwaysInstallElevated

Check both registry locations:

```cmd
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated
reg query "HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated
```

If both values are enabled (`0x1`), Windows Installer packages can be installed with elevated privileges under the applicable configuration.

Treat this as a high-priority misconfiguration finding. Validate it only in an authorized lab and follow the assessment's change-control requirements.

## 13. Token Privileges

### Enumerate the current token

```cmd
whoami /priv
whoami /all
```

Review privileges such as:

| Privilege | Why it matters |
|---|---|
| `SeImpersonatePrivilege` | May permit impersonation-based escalation in applicable configurations |
| `SeAssignPrimaryTokenPrivilege` | Relevant to assigning primary tokens |
| `SeBackupPrivilege` | May allow access to protected data through backup semantics |
| `SeRestorePrivilege` | May permit privileged restore operations |
| `SeDebugPrivilege` | Can enable powerful process access |
| `SeTakeOwnershipPrivilege` | May permit taking ownership of objects |
| `SeLoadDriverPrivilege` | Can be dangerous if exploitable driver-loading conditions exist |
| `SeCreateTokenPrivilege` | Highly sensitive token-creation privilege |

Privileges alone do not prove an escalation path. Check whether they are enabled, usable by the current token, and applicable to the OS build.

## 14. Patch and Security Enumeration

### Installed hotfixes

```cmd
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

PowerShell:

```powershell
Get-HotFix | Sort-Object InstalledOn -Descending
```

### Defender status

```powershell
Get-MpComputerStatus
Get-MpPreference
```

### Firewall profiles

```powershell
Get-NetFirewallProfile
```

### Windows services related to security

```cmd
sc query WinDefend
sc query BFE
```

### Drivers

```cmd
driverquery /v
```

PowerShell:

```powershell
Get-CimInstance Win32_SystemDriver |
    Select-Object Name, State, StartMode, PathName
```

**Look for:**
- Missing security updates.
- Unsupported OS versions.
- Vulnerable third-party applications and drivers.
- Security controls that are disabled or incorrectly configured.

Do not assume that a particular CVE is exploitable just because a patch appears missing. Verify the affected build, prerequisites, and mitigations.

## 15. Startup and Persistence Locations

Enumeration of startup locations can reveal privileged programs launched automatically.

### Startup directories

```cmd
dir /a "%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup"
dir /a "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

### Registry startup entries

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"
```

### PowerShell

```powershell
Get-CimInstance Win32_StartupCommand |
    Select-Object Name, Command, Location, User
```

Check whether an automatically executed file or its parent directory is writable by your current account.

## 16. UAC and Security Controls

### Check UAC configuration

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System"
```

### PowerShell

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" |
    Select-Object EnableLUA, ConsentPromptBehaviorAdmin, PromptOnSecureDesktop
```

### Current integrity level

```cmd
whoami /groups
```

Look for integrity labels such as:
- Low
- Medium
- High
- System

UAC is a security boundary mechanism for administrative elevation, but not every UAC bypass technique constitutes a vulnerability or works on every Windows version. Avoid treating a single registry value as proof of exploitability.

## 17. Domain Enumeration

First establish whether the machine belongs to a domain.

```cmd
systeminfo
echo %USERDOMAIN%
echo %LOGONSERVER%
nltest /dsgetdc:<domain>
nltest /dclist:<domain>
nltest /domain_trusts
gpresult /r
gpresult /scope computer /r
net accounts /domain
net user /domain
net group /domain
```

### PowerShell

```powershell
$env:USERDOMAIN
$env:LOGONSERVER
Get-CimInstance Win32_ComputerSystem |
    Select-Object Domain, PartOfDomain
```

### Active Directory module, if installed and authorized

```powershell
Get-ADDomain
Get-ADForest
Get-ADUser -Filter * -Properties MemberOf
Get-ADGroup -Filter *
Get-ADComputer -Filter *
```

### Legacy directory commands

```cmd
dsquery user
dsquery group
```

These commands may require the Active Directory DS tools and appropriate permissions.

### Group Policy

```cmd
gpresult /h "%TEMP%\gpresult.html"
```

Review applied policies, local administrative rights, security configurations, and delegated permissions.

**Look for:**
- Excessive local administrator access.
- Misconfigured Group Policy.
- Privileged groups and service accounts.
- Weak service account configurations.
- Unnecessary domain trusts and permissions.

## 18. PowerShell History and Environment

### History locations

```powershell
(Get-PSReadLineOption).HistorySavePath
```

```cmd
dir /a "%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine"
```

### PATH enumeration

```cmd
echo %PATH%
```

```powershell
$env:PATH -split ';'
```

Check PATH directories for unsafe write permissions and unexpected executable resolution. A writable PATH directory is only relevant if a privileged process actually resolves an executable from it.

### Temporary directories

```cmd
echo %TEMP%
echo %TMP%
```

```powershell
Get-Acl $env:TEMP
Get-Acl C:\Windows\Temp
```

## 19. Enumeration Tools

Use trusted, version-appropriate tools only within your authorization scope.

| Tool | Purpose |
|---|---|
| [WinPEAS](https://github.com/peass-ng/PEASS-ng) | Automated Windows privilege escalation enumeration |
| [Seatbelt](https://github.com/GhostPack/Seatbelt) | Windows security posture and configuration enumeration |
| [PrivescCheck](https://github.com/itm4n/PrivescCheck) | PowerShell-based privilege escalation checks |
| [AccessChk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) | Inspect object and access permissions |
| [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) | Windows troubleshooting and inspection utilities |
| [SharpUp](https://github.com/GhostPack/SharpUp) | C# privilege escalation checks |

### Suggested enumeration order

1. Run basic system and user enumeration.
2. Check `whoami /all` and token privileges.
3. Enumerate services and scheduled tasks.
4. Inspect executable paths and permissions.
5. Check registry configuration and startup locations.
6. Review credentials and deployment files.
7. Identify patches and security mitigations.
8. Investigate domain configuration where applicable.
9. Validate potential findings manually.
10. Document evidence, impact, and remediation.

Automated tools may produce false positives. Confirm each result before reporting it.

## 20. Privilege Escalation Checklist

Use this checklist during authorized lab work or penetration testing.

- [ ] OS version, build, architecture, and patches identified
- [ ] Current user, groups, and token privileges enumerated
- [ ] Local administrator membership checked
- [ ] Processes and services enumerated
- [ ] Service executable paths and ACLs inspected
- [ ] Unquoted service path candidates reviewed
- [ ] Scheduled tasks and task permissions checked
- [ ] AlwaysInstallElevated configuration checked
- [ ] AutoLogon configuration reviewed
- [ ] Registry and application configuration inspected
- [ ] Credential Manager and PowerShell history checked
- [ ] Startup locations and PATH permissions inspected
- [ ] Writable directories and sensitive file permissions reviewed
- [ ] Drivers and installed security updates identified
- [ ] UAC and security controls reviewed
- [ ] Domain membership, Group Policy, and privileges checked
- [ ] Potential findings manually validated
- [ ] Evidence and remediation documented

---

## Reporting Template

For every suspected privilege escalation path, record:

```text
Finding:
Affected Host:
Current User:
Target Privilege:
Affected Component:
Misconfiguration:
Evidence:
Required Conditions:
Potential Impact:
Reproducibility:
Recommended Remediation:
```

### Important Remediation Examples

- Remove unnecessary local administrator membership.
- Apply security updates and upgrade unsupported software.
- Restrict service and scheduled task modification permissions.
- Quote service executable paths and protect their parent directories.
- Remove plaintext credentials from scripts and configuration files.
- Disable insecure AutoLogon and AlwaysInstallElevated configurations.
- Apply least privilege to service accounts.
- Review Group Policy and domain delegation permissions.

**Disclaimer:** This cheat sheet is for education, authorized penetration testing, and defensive security assessments. Enumeration results should be validated before drawing conclusions about exploitability.
