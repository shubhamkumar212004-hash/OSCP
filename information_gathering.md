# Reconnaissance & Enumeration 

> A practical, GitHub‑ready reconnaissance and enumeration notes collection for pentesting & CTFs.

---

## Reconnaissance

### Host Discovery

#### Nmap (Ping Scan)

```bash
nmap -sn 10.10.0.0/16
```

#### Static Binaries (Useful for restricted systems)

```
https://github.com/andrew-d/static-binaries/tree/master/binaries
```

#### CrackMapExec (SMB Host Discovery)

```bash
crackmapexec smb 192.168.0.0/24
```

---

### Ping Sweep

#### PowerShell

```powershell
for ($i=1;$i -lt 255;$i++) { ping -n 1 192.168.0.$i | findstr "TTL" }
```

#### Bash

```bash
for i in {1..255}; do (ping -c 1 192.168.0.$i | grep "bytes from" &); done
```

---

## Port Scanning

### Bash (TCP Scan)

```bash
for i in {1..65535}; do (echo > /dev/tcp/192.168.1.1/$i) >/dev/null 2>&1 && echo "$i open"; done
```

### Netcat

```bash
nc -zvn <IP> 1-1000
```

Static ncat binary:

```
https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/ncat
```

### Nmap Full Scan

```bash
nmap -sC -sV -A -Pn -T5 -p- <IP>
```

### RustScan

#### Install

```bash
sudo docker pull rustscan/rustscan:2.1.1
alias rustscan='sudo docker run -it --rm rustscan/rustscan:2.1.1'
```

#### Scan

```bash
rustscan -a <IP> -- -A -Pn
```

---

## DNS Enumeration

### Host Records

```bash
host <domain>
host -t mx megacorpone.com
host -t txt <domain>
```

### Forward Lookup Bruteforce

```bash
for i in $(cat wordlist.txt); do host $i.<domain>; done
```

### Reverse Lookup Bruteforce

```bash
for i in $(seq 50 100); do host 192.168.0.$i; done | grep -v "not found"
```

### Get DNS Servers

```bash
host -t ns megacorpone.com | cut -d " " -f 4
```

### DNS Zone Transfer

```bash
host -l <domain> <dns-server>
```

#### Automated Zone Transfer

```bash
for ns in $(host -t ns $1 | cut -d ' ' -f 4); do host -l $1 $ns; done
```

### dnsrecon

```bash
dnsrecon -d <domain> -t axfr
dnsrecon -d megacorpone.com -D wordlist.txt -t brt
```

---

## SMB Enumeration

```bash
nmap -v -p 139,445 -oG smb.txt 10.11.1.1-254
sudo nbtscan -r 10.11.1.0/24
```

### enum4linux

```bash
enum4linux <IP>
enum4linux -a -u "" -p "" <IP>
enum4linux -a -u "guest" -p "" <IP>
```

---

## NFS Enumeration

### NFS Version

```bash
nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254
rpcinfo <IP> | grep nfs
```

### List Shares

```bash
nmap -p 111 --script nfs* <IP>
showmount -e <IP>
```

### Mount Share

```bash
mkdir /tmp/nfs
sudo mount -t nfs -o vers=4 <IP>:/folder /tmp/nfs -o nolock
```

### Config Files

```
/etc/exports
/etc/lib/nfs/etab
```

---

## LDAP Enumeration

```bash
nmap -n -sV --script "ldap* and not brute" <IP>
ldapsearch -H <IP> -b "DC=domain,DC=com"
```

---

## SNMP Enumeration

```bash
sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
```

### Community Strings

```bash
echo public > community
echo private >> community
echo manager >> community
```

### onesixtyone

```bash
onesixtyone -c community -i ips
```

### SNMP Walk Examples

```bash
snmpwalk -c public -v1 <IP>
snmpwalk -c public -v1 <IP> 1.3.6.1.4.1.77.1.2.25
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.4.2.1.2
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.6.13.1.3
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.6.3.1.2
```

---

## FTP Enumeration

### Default Credentials

```
anonymous : anonymous
```

### Version Grab

```bash
nc <IP> 21
```

### Nmap Scripts

```bash
nmap --script ftp-* -p 21 <IP>
```

### FTP Modes

```bash
ftp <IP>
binary
ascii
```

---

## RDP Enumeration

```bash
nmap --script rdp-ntlm-info,rdp-enum-encryption,rdp-vuln-ms12-020 -p 3389 <IP>
```

### Connect

```bash
rdesktop -u <username> <IP>
xfreerdp /d:<domain> /u:<username> /p:<password> /v:<IP>
```

---

## POP3 Enumeration

```bash
nmap --script pop3-capabilities,pop3-ntlm-info -sV -p 110 <IP>
```

```bash
telnet <IP> 110
USER user
PASS password
list
retr 1
```

---

## SMTP Enumeration

```bash
nmap -p25 --script smtp-commands,smtp-open-relay <IP>
```

### Send Mail

```bash
nc -C <IP> 25
HELO
MAIL FROM:user@local
RCPT TO:user@local
DATA
Subject: Test

http://<IP>/malware.exe
.
QUIT
```

### Username Enumeration

```bash
hydra smtp-enum://<IP>/vrfy -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

---

## Web Recon

### Technology Detection

* Wappalyzer → [https://www.wappalyzer.com/](https://www.wappalyzer.com/)
* WhatWeb

```bash
whatweb site.com
```

### Directory Bruteforce

```bash
ffuf -u http://site.com/FUZZ -w /usr/share/wordlists/dirb/big.txt
gobuster dir -u <IP> -w /usr/share/wordlists/dirb/common.txt
```

### File Extension Fuzzing

```bash
ffuf -u "https://site.com/indexFUZZ" -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt
```

### Parameter Fuzzing

```bash
ffuf -u "https://site.com/index.php?FUZZ=test" -w wordlist.txt
ffuf -u "https://site.com/index.php" -X POST -d 'FUZZ=test' -w wordlist.txt
```

Wordlists:

```
https://github.com/danielmiessler/SecLists
```

### Nikto

```bash
nikto -h http://site.com
```

### HTTP Enum (Nmap)

```bash
nmap -p80 --script=http-enum <IP>
```

---

## CMS Enumeration

### WordPress

```bash
wpscan --url http://site.com --api-token <TOKEN> --enumerate u,vp
```

### Joomla

```bash
python main.py -u <target>
```

Repo: [https://github.com/oppsec/juumla](https://github.com/oppsec/juumla)

### Drupal

```bash
droopescan scan drupal -u <target>
```

Repo: [https://github.com/SamJoan/droopescan](https://github.com/SamJoan/droopescan)

### Magento

```bash
php magescan.phar scan:all www.example.com
```

Repo: [https://github.com/steverobbins/magescan](https://github.com/steverobbins/magescan)

---

✅ **End of Recon & Enumeration Notes**

