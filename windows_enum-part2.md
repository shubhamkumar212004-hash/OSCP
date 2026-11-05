# Windows & Active Directory Privilege Escalation Enumeration Cheat Sheet

A practical Windows / Active Directory privilege escalation enumeration checklist for authorized penetration testing, OSCP preparation, and CTF labs.

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
- [19. Active Directory Module Cheat Sheet](#19-active-directory-module-cheat-sheet)
- [20. AD Recycle Bin Enumeration and Recovery](#20-ad-recycle-bin-enumeration-and-recovery)
- [21. Logon Scripts and Legacy Attribute Hunting](#21-logon-scripts-and-legacy-attribute-hunting)
- [22. Reverse Engineering Custom Credential Tools](#22-reverse-engineering-custom-credential-tools)
- [23. Enumeration Tools](#23-enumeration-tools)
- [24. Privilege Escalation Checklist](#24-privilege-escalation-checklist)

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

**Example output (`whoami /priv`):**

```text
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= =============================== =========
SeMachineAccountPrivilege     Add workstations to domain      Disabled
SeBackupPrivilege             Back up files and directories   Enabled
SeRestorePrivilege            Restore files and directories   Enabled
SeImpersonatePrivilege        Impersonate a client             Enabled
SeCreateGlobalPrivilege       Create global objects            Enabled
```

> `SeBackupPrivilege` + `SeRestorePrivilege` both Enabled here is a high-value finding — it can allow reading/writing files that would normally be access-denied, including `SAM`/`SYSTEM` hive copies.

### Check local administrator membership

```cmd
net localgroup Administrators
```

**Example output:**

```text
Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------
Administrator
CASCADE\Domain Admins
CASCADE\svc_backup
The command completed successfully.
```

> A service account (`svc_backup`) sitting directly in local Administrators is worth flagging — it's an unusual membership for a non-interactive account and a common privesc/lateral-movement target.

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

**Example output (`netstat -ano`):**

```text
Proto  Local Address          Foreign Address        State           PID
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       812
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
TCP    127.0.0.1:3306         0.0.0.0:0              LISTENING       2240
TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
```

> `127.0.0.1:3306` listening only on loopback is a classic lead — a MySQL instance that's unreachable remotely but may be reachable (and have weak local auth) from the current foothold.

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

**Example output:**

```text
Name    : BackupAgentSvc
StartName: LocalSystem
PathName : C:\Program Files\Backup Agent\backup service.exe --run
```

> Windows will try `C:\Program.exe`, then `C:\Program Files\Backup.exe`, before the real binary — if any of those intermediate paths is writable, this is directly exploitable.

### 6.2 Check service configuration

```cmd
sc qc <ServiceName>
sc sdshow <ServiceName>
```

**Example output (`sc qc BackupAgentSvc`):**

```text
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: BackupAgentSvc
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files\Backup Agent\backup service.exe --run
        SERVICE_START_NAME : LocalSystem
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

**Example output (vulnerable configuration):**

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1

HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1
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

---

## 19. Active Directory Module Cheat Sheet

`Get-ADUser`/`Get-ADObject` and friends come from the **ActiveDirectory** PowerShell module (RSAT). It is often already present on a Domain Controller, or installable via RSAT on a workstation. `PowerView` (`Get-DomainUser`, etc.) is a separate, non-native offensive toolkit — do not confuse the two. If `Get-DomainUser` is "not recognized," the native AD module is usually the faster path since it may already be on the box.

### Import and basic checks

```powershell
Import-Module ActiveDirectory
Get-Module ActiveDirectory
```

### User and attribute lookups

```powershell
Get-ADUser -Identity <username> -Properties *
Get-ADUser -Identity <username> -Properties ScriptPath | Select-Object ScriptPath
Get-ADUser -Filter * -Properties MemberOf, LastLogonDate, PasswordLastSet
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

**Example output (`Get-ADUser -Identity j.allen -Properties ScriptPath`):**

```text
ScriptPath
----------
MapAuditDrive.vbs
```

> A populated `ScriptPath` is a strong lead — pull the actual `.vbs`/`.bat`/`.ps1` file from SYSVOL/NETLOGON next (see Section 21) and read it for share paths or embedded creds.

### Group membership and privileged groups

```powershell
Get-ADGroupMember -Identity "Domain Admins"
Get-ADPrincipalGroupMembership -Identity <username>
Get-ADGroup -Filter * | Select-Object Name
```

### Computers and trusts

```powershell
Get-ADComputer -Filter * -Properties OperatingSystem
Get-ADTrust -Filter *
```

### ADSI fallback (no module needed, built into .NET)

Useful when the ActiveDirectory module isn't installed and you can't add it:

```powershell
$searcher = New-Object DirectoryServices.DirectorySearcher([ADSI]"LDAP://DC=domain,DC=local")
$searcher.Filter = "(sAMAccountName=<username>)"
$result = $searcher.FindOne()
$result.Properties["scriptpath"]
```

Or bind to a known DN directly:

```powershell
$user = [ADSI]"LDAP://CN=<username>,CN=Users,DC=domain,DC=local"
$user.scriptPath
```

**Look for:**
- `scriptPath` attributes pointing to logon scripts on SYSVOL/NETLOGON — these can embed share paths or credentials.
- Non-standard custom schema attributes (e.g., legacy migration fields) that may hold base64 or encrypted values.
- Group memberships that grant non-obvious rights (AD Recycle Bin, Backup Operators, DnsAdmins, etc.).

---

## 20. AD Recycle Bin Enumeration and Recovery

Membership in a group with AD Recycle Bin rights (or direct delegation on the "Deleted Objects" container) allows viewing — and potentially restoring — tombstoned/deleted AD objects, including their attributes.

### 20.1 List all deleted objects

```powershell
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects -Properties *
```

This can be noisy (GPOs, computer accounts, groups, etc. also show up as deleted). Narrow it down:

```powershell
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects -Properties * |
    Where-Object {$_.ObjectClass -eq "user"}
```

```powershell
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects -Properties * |
    Where-Object {$_.Name -like "*<partial-name>*"}
```

**Example output (truncated):**

```text
CanonicalName      : cascade.local/Deleted Objects/TempAdmin
                      DEL:f0cc344d-31e0-4866-bceb-a842791ca059
cascadeLegacyPwd    : YmFDVDNyMWFOMDBkbGVz
CN                  : TempAdmin
                      DEL:f0cc344d-31e0-4866-bceb-a842791ca059
DisplayName         : TempAdmin
isDeleted           : True
LastKnownParent     : OU=Users,OU=UK,DC=cascade,DC=local
ObjectClass         : user
ObjectGUID          : f0cc344d-31e0-4866-bceb-a842791ca059
sAMAccountName      : TempAdmin
userAccountControl  : 66048
```

> This is the finding to flag in a report: a deleted "temporary admin" account whose custom legacy-password attribute is still readable by any principal with Deleted-Objects visibility — long after the account was supposedly removed.

### 20.2 Inspect custom attributes on the deleted object

Deleted objects can retain custom schema attributes (depending on tombstone lifetime / recycle bin config). Look for anything unusual in the full property dump — custom fields are often named per-organization (e.g., a legacy migration attribute) and may hold base64-encoded or otherwise encoded credentials left behind from a migration or audit process.

```powershell
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects -Properties * |
    Where-Object {$_.Name -like "*<target-user>*"} |
    Format-List *
```

Decode any base64-looking value found (do this on your own analysis host, not by pasting secrets into shared notes):

```bash
echo "<base64-string>" | base64 -d
```

> Not every encoded value is base64 "plaintext" — some may be further encrypted (see Section 22). Confirm the encoding before assuming it's directly decodable.

### 20.3 Restoring a deleted object (legitimate recovery / defensive use)

If the goal is genuine object recovery rather than just reading leftover attributes:

```powershell
# Restore by piping from a filtered search
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects -Properties * |
    Where-Object {$_.Name -like "*<target-user>*"} |
    Restore-ADObject

# Or restore directly by ObjectGUID (safer when multiple matches exist)
Restore-ADObject -Identity "<ObjectGUID>"
```

Verify the restore:

```powershell
Get-ADUser -Identity <username> -Properties *
```

**Example output (successful restore):**

```text
DistinguishedName : CN=TempAdmin,OU=Users,OU=UK,DC=cascade,DC=local
Enabled           : True
GivenName         : TempAdmin
Name              : TempAdmin
ObjectClass       : user
ObjectGUID        : f0cc344d-31e0-4866-bceb-a842791ca059
SamAccountName    : TempAdmin
SID               : S-1-5-21-3332504370-1206983947-1165150453-1136
UserPrincipalName : TempAdmin@cascade.local
```

A successfully restored object returns to its last known parent OU with its prior attributes intact. This is the standard, supported recovery path when the AD Recycle Bin feature is enabled on the forest — no backup restore or reinstall required.

**Look for:**
- Who has delegated rights to view/restore Deleted Objects.
- Whether deleted *user* objects still expose sensitive attributes while tombstoned.
- Whether a "temporary" or "break-glass" admin account was deleted but its attributes (or a shared legacy password) remain discoverable.

---

## 21. Logon Scripts and Legacy Attribute Hunting

Logon scripts (`scriptPath` in AD) run automatically on interactive logon and are a common place to find mapped drives, shares, or (in poorly hardened environments) embedded credentials.

### 21.1 Find a user's configured logon script

```powershell
Get-ADUser -Identity <username> -Properties ScriptPath | Select-Object ScriptPath
```

### 21.2 Locate the script on SYSVOL / NETLOGON

```powershell
dir \\<domain>\SYSVOL\<domain>\scripts\
dir \\<DC-hostname>\NETLOGON\
```

Or search recursively if the exact subfolder is unknown:

```powershell
Get-ChildItem -Path \\<domain>\SYSVOL -Recurse -Filter "<scriptname>" -ErrorAction SilentlyContinue
```

### 21.3 Read the script content

```powershell
type \\<domain>\SYSVOL\<domain>\scripts\<scriptname>.vbs
```

**Example output:**

```vbs
'MapAuditDrive.vbs
Option Explicit
Dim oNetwork, strDriveLetter, strRemotePath
strDriveLetter = "F:"
strRemotePath = "\\CASC-DC1\Audit$"
Set oNetwork = CreateObject("WScript.Network")
oNetwork.MapNetworkDrive strDriveLetter, strRemotePath
WScript.Quit
```

> Note the UNC path uses the DC's **NetBIOS hostname** (`CASC-DC1`), not the domain FQDN — if `\\<domain>\Audit$` doesn't resolve/connect, retry with the hostname shown in the script itself.

Review `.vbs`/`.bat`/`.ps1` logon scripts for:
- `MapNetworkDrive` calls revealing share paths the account can reach.
- Hardcoded credentials (increasingly rare but still found in legacy environments).
- References to the **actual DC hostname** rather than the domain FQDN — UNC paths inside scripts sometimes use `\\HOSTNAME\Share$` rather than `\\domain.local\Share$`, so if a domain-based path 404s, retry with the DC's NetBIOS/hostname.

### 21.4 Explore shares revealed by the script

```powershell
net view \\<DC-hostname>
dir \\<DC-hostname>\<Share$>\
```

**Example output (`dir \\CASC-DC1\Audit$\`):**

```text
    Directory: \\CASC-DC1\Audit$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        1/28/2020   9:40 PM                DB
d-----        1/26/2020  10:25 PM                x64
d-----        1/26/2020  10:25 PM                x86
-a----        1/28/2020   9:46 PM          13312 CascAudit.exe
-a----        1/29/2020   6:00 PM          12288 CascCrypto.dll
-a----        1/28/2020  11:29 PM             45 RunAudit.bat
```

> `CascCrypto.dll` next to the audit tool is the tell: an internal tool with its own encryption library almost always means recoverable (often hardcoded-key) credentials inside a DB file in the adjacent `DB` folder — go to Section 22.

Download anything relevant for offline analysis:

```powershell
# from an Evil-WinRM session
download C:\path\to\file
```

### 21.5 Legacy / custom LDAP attributes holding credentials

Some environments carry forward custom schema attributes from old migrations (e.g., a `*LegacyPwd` style attribute) containing base64 or encrypted values. These can surface via:

```bash
ldapsearch -x -H ldap://<DC-IP> -D "<domain>\<user>" -w '<password>' \
  -b "DC=<domain>,DC=local" "(sAMAccountName=<target>)" <attributeName>
```

Or, with a null/anonymous bind if the domain permits it:

```bash
ldapsearch -x -H ldap://<DC-IP> -b "DC=<domain>,DC=local" | grep -Ei "passw|pwd"
```

**Example output:**

```text
cascadeLegacyPwd: clk0bjVldmE=
sAMAccountName: r.thompson
```

> A null/anonymous LDAP bind that still returns a custom password-like attribute on a real user object is a critical finding on its own — flag it regardless of whether the attribute decodes to something useful.

**Look for:**
- Any attribute name that doesn't look like a standard AD schema field.
- Values that decode cleanly from base64 vs. values that remain binary garbage after decoding (the latter usually means an additional encryption layer — see Section 22).

---

## 22. Reverse Engineering Custom Credential Tools

Internal/custom audit or admin tools found on shares sometimes encrypt stored credentials with a fixed, hardcoded key embedded in a companion DLL or EXE. This section covers the general workflow for identifying and using such a key — purely for recovering credentials you are authorized to access as part of an assessment, not for attacking third-party software.

### 22.1 Identify the binaries

```powershell
dir \\<server>\<Share$>\
```

Typical shape of a vulnerable internal tool: a main `.exe`, a supporting crypto `.dll`, and a `.bat`/config that shows how they're invoked together (which itself is a strong hint as to what calls what).

### 22.2 Static analysis — decompile without running anything

Any of the following work for inspecting .NET assemblies (EXE/DLL) without executing them:

| Tool | Platform | Notes |
|---|---|---|
| dnSpy / dnSpyEx | Windows | GUI, supports live debugging too |
| ILSpy / ILSpy CLI (`ilspycmd`) | Cross-platform | Good CLI option for Linux |
| dotPeek | Windows | JetBrains, free |
| `monodis` | Linux | Lightweight IL dump; can be unstable on some binaries |
| Online decompilers (e.g. decompiler.com) | Any | Convenient, but avoid uploading sensitive/client binaries to third-party services outside an authorized scope |

Look inside the relevant crypto class/method for hardcoded `key`/`IV` string literals — these are usually visible directly in the decompiled C# source with no further effort.

### 22.3 Dynamic analysis — debugging as a fallback

If static analysis alone doesn't make the key/IV obvious, running the tool under a debugger with a breakpoint on the decrypt call can reveal values at runtime. Common pitfalls when doing this:

- **Run the EXE from an actual command prompt first**, not just inside the debugger — this surfaces the program's own `Console.WriteLine` error messages (e.g., connection errors), which are often much more informative than a debugger's generic "unhandled exception, no message" popup.
- **Set the correct arguments and working directory** in the debugger's launch settings — many tools expect to be run from their own folder (for locating sibling DLLs, e.g. native SQLite interop DLLs) and will fail before reaching your breakpoint if launched from elsewhere.
- **Match bitness (32-bit vs 64-bit)** between the debugger process and the target binary/dependencies — a mismatch (e.g. 64-bit debugger loading a 32-bit-only native dependency) can cause an opaque crash before your breakpoint is ever hit.
- If a DLL fails to load with `FileLoadException` / `HRESULT 0x80131515` in PowerShell (`Add-Type -Path ...`), check:
  - Whether the file is flagged as downloaded-from-network (`Unblock-File -Path <dll>`).
  - Whether you're in the matching-bitness PowerShell host (`C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` for 32-bit).

### 22.4 Fastest path once key material is known: isolate just the decrypt call

Rather than fighting a complex application's full startup path (network calls, LDAP binds, etc.), load just the crypto assembly and call its public method directly:

```powershell
Add-Type -Path "C:\path\to\Crypto.dll"
$encrypted = "<base64-ciphertext-from-source>"
$key = "<recovered-key>"
$result = [Namespace.ClassName]::DecryptString($encrypted, $key)
Write-Host $result
```

**Example output:**

```text
Decrypted password: Examp1ePassw0rd
```

Or offline with Python, once the algorithm (commonly AES-CBC), key, and IV are known:

```python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

ciphertext = base64.b64decode("<base64-ciphertext>")
key = b"<key-bytes>"
iv  = b"<iv-bytes>"

cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = unpad(cipher.decrypt(ciphertext), 16)
print(plaintext.decode())
```

Or via CyberChef (no scripting needed): `From Base64` → `AES Decrypt` (Key/IV as UTF8, Mode CBC, Input/Output Raw).

### 22.5 Common gotchas

- **Case sensitivity matters.** A decrypted/recovered credential that "almost" matches across spraying attempts but fails everywhere may simply have a transcription case error (e.g., one letter copied in the wrong case). Re-verify the exact output of the decrypt step character-by-character before concluding the credential is invalid.
- **Don't assume failure means wrong key/IV** — confirm the algorithm, mode, padding scheme, and encoding (base64 vs hex) are all correct before troubleshooting the key itself.
- Validate recovered credentials broadly but carefully:
  ```bash
  nxc smb <target> -u <recovered-username> -p '<recovered-password>'
  nxc winrm <target> -u <recovered-username> -p '<recovered-password>'
  ```
  rather than only testing against one service, since different protocols can behave differently (lockout policies, protocol-specific auth requirements).

  **Example output (successful WinRM validation):**

  ```text
  WINRM   10.129.97.25   5985   CASC-DC1   [*] Windows 7 / Server 2008 R2 Build 7601 (name:CASC-DC1) (domain:cascade.local)
  WINRM   10.129.97.25   5985   CASC-DC1   [+] cascade.local\arksvc:Examp1ePassw0rd (Pwn3d!)
  ```

---

## 23. Enumeration Tools

Use trusted, version-appropriate tools only within your authorization scope.

| Tool | Purpose |
|---|---|
| [WinPEAS](https://github.com/peass-ng/PEASS-ng) | Automated Windows privilege escalation enumeration |
| [Seatbelt](https://github.com/GhostPack/Seatbelt) | Windows security posture and configuration enumeration |
| [PrivescCheck](https://github.com/itm4n/PrivescCheck) | PowerShell-based privilege escalation checks |
| [AccessChk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) | Inspect object and access permissions |
| [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) | Windows troubleshooting and inspection utilities |
| [SharpUp](https://github.com/GhostPack/SharpUp) | C# privilege escalation checks |
| [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) | Offensive AD recon (`Get-DomainUser`, etc.) — not built-in, must be loaded/uploaded |
| [BloodHound](https://github.com/SpecterOps/BloodHound) | AD attack-path graphing and analysis |
| [Impacket](https://github.com/fortra/impacket) | Python toolkit for AD/Windows protocol interaction (secretsdump, GetUserSPNs, etc.) |
| [dnSpyEx](https://github.com/dnSpyEx/dnSpy) | .NET debugger/decompiler |
| [ILSpy](https://github.com/icsharpcode/ILSpy) | .NET decompiler, cross-platform CLI available |

### Suggested enumeration order

1. Run basic system and user enumeration.
2. Check `whoami /all` and token privileges.
3. Enumerate services and scheduled tasks.
4. Inspect executable paths and permissions.
5. Check registry configuration and startup locations.
6. Review credentials and deployment files.
7. Identify patches and security mitigations.
8. Investigate domain configuration where applicable (including logon scripts and AD Recycle Bin rights).
9. Validate potential findings manually.
10. Document evidence, impact, and remediation.

Automated tools may produce false positives. Confirm each result before reporting it.

## 24. Privilege Escalation Checklist

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
- [ ] User logon scripts (`scriptPath`) identified and reviewed
- [ ] AD Recycle Bin rights checked; deleted objects enumerated for leftover attributes
- [ ] Custom/legacy LDAP attributes hunted for encoded or encrypted credentials
- [ ] Internal credential tools (if present) reverse engineered for hardcoded keys, where in scope
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
- Restrict AD Recycle Bin / Deleted Objects visibility to the minimum necessary group membership.
- Purge or rotate any legacy custom LDAP attributes that still carry encoded/encrypted credentials.
- Replace hardcoded encryption keys in internal tools with properly managed secrets (e.g., a vault or DPAPI-protected store), since a key embedded in a shipped binary provides no real confidentiality.

**Disclaimer:** This cheat sheet is for education, authorized penetration testing, and defensive security assessments. Enumeration results should be validated before drawing conclusions about exploitability.
