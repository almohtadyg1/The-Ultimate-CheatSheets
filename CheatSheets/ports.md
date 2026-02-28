# Network Ports Cheat Sheet
> Complete Guide to TCP/UDP Ports and Network Protocols

---

## Overview

**What are Ports?** Network ports are virtual endpoints for communication. They range from 0–65535 and help route network traffic to the correct service or application.

### Port Ranges

| Range | Type | Description |
|-------|------|-------------|
| 0 – 1023 | System / Well-Known | Reserved for common services (HTTP, FTP, SSH, etc.) |
| 1024 – 49151 | Registered | Registered for specific applications and services |
| 49152 – 65535 | Dynamic / Private | Used for temporary connections and client-side ports |

---

## TCP vs UDP

### TCP (Transmission Control Protocol)
**Connection-oriented, Reliable**

- Guarantees delivery of packets
- Ensures packets arrive in order
- Error checking and retransmission
- 3-way handshake (SYN, SYN-ACK, ACK)
- Slower but more reliable
- Header size: 20–60 bytes

**Best for:** Web browsing, email, file transfers, remote access

---

### UDP (User Datagram Protocol)
**Connectionless, Fast**

- No delivery guarantee
- Packets may arrive out of order
- Minimal error checking
- No handshake — send and forget
- Faster with lower overhead
- Header size: 8 bytes

**Best for:** Streaming, gaming, VoIP, DNS, live broadcasts

---

### Detailed Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (handshake required) | Connectionless (no handshake) |
| Reliability | Guaranteed delivery with retransmission | No guarantee, packets may be lost |
| Speed | Slower (acknowledgments, error checking) | Faster (minimal overhead) |
| Data Flow | Stream-based (continuous flow) | Message-based (individual datagrams) |
| Ordering | Packets delivered in sequence | No ordering guarantee |
| Flow Control | Yes (sliding window, congestion control) | No flow control |
| Error Handling | Detection + correction via retransmission | Detection only (checksum) |
| Resource Usage | Higher (state tracking, buffers) | Lower (minimal overhead) |

---

## Common Ports

| Port | Protocol | Service / Description |
|------|----------|-----------------------|
| 20 | TCP/UDP | FTP (Data transfer) |
| 21 | TCP/UDP | FTP (Control / commands) |
| 22 | TCP/UDP | SSH, SFTP, SCP (secure remote access & transfer) |
| 23 | TCP | Telnet (unencrypted remote login) |
| 25 | TCP | SMTP (email transfer between servers) |
| 53 | TCP/UDP | DNS (domain name resolution) |
| 67 | UDP | DHCP Server |
| 68 | UDP | DHCP Client |
| 69 | UDP | TFTP (Trivial File Transfer Protocol) |
| 80 | TCP/UDP | HTTP (web traffic) |
| 110 | TCP | POP3 (email retrieval) |
| 123 | UDP | NTP (Network Time Protocol — time sync) |
| 135 | TCP | Microsoft RPC / DCOM services |
| 137 | TCP/UDP | NetBIOS Name Service |
| 138 | UDP | NetBIOS Datagram Service |
| 139 | TCP | NetBIOS Session Service |
| 143 | TCP | IMAP (email retrieval & management) |
| 161 | UDP | SNMP (network monitoring) |
| 162 | UDP | SNMP Trap (alerts) |
| 179 | TCP/UDP | BGP (Border Gateway Protocol) |
| 389 | TCP | LDAP (Lightweight Directory Access Protocol) |
| 443 | TCP/UDP | HTTPS (secure web traffic) |
| 445 | TCP | SMB / Microsoft-DS (file sharing, Active Directory) |
| 465 | TCP | SMTPS (SMTP over SSL) |
| 514 | UDP | Syslog (system logging) |
| 515 | TCP | LPD (Line Printer Daemon) |
| 587 | TCP | SMTP (mail submission, modern use) |
| 631 | TCP | IPP (Internet Printing Protocol) |
| 636 | TCP | LDAPS (LDAP over SSL) |
| 989 | TCP | FTPS Data (FTP over SSL) |
| 990 | TCP | FTPS Control (FTP over SSL) |
| 993 | TCP | IMAPS (IMAP over SSL) |
| 995 | TCP | POP3S (POP3 over SSL) |

---

## Web & Email

### 🌐 Web Traffic

| Port | Protocol | Service |
|------|----------|---------|
| 80 | TCP | HTTP — Unencrypted web traffic |
| 443 | TCP | HTTPS — Encrypted web traffic (SSL/TLS) |
| 8080 | TCP | HTTP Alternative — Often used for proxies |

### 📧 Email Services

| Port | Protocol | Service | Purpose |
|------|----------|---------|---------|
| 25 | TCP | SMTP | Server-to-server email transfer |
| 587 | TCP | SMTP | Email submission (modern standard) |
| 465 | TCP | SMTPS | SMTP over SSL (legacy) |
| 110 | TCP | POP3 | Email retrieval (downloads & deletes) |
| 995 | TCP | POP3S | POP3 over SSL |
| 143 | TCP | IMAP | Email retrieval (syncs, keeps on server) |
| 993 | TCP | IMAPS | IMAP over SSL |

---

## File Transfer & Remote Access

### 📁 File Transfer Protocols

| Port | Protocol | Service | Security |
|------|----------|---------|----------|
| 20–21 | TCP | FTP | ❌ Unencrypted |
| 22 | TCP | SFTP / SCP | ✅ Encrypted (SSH) |
| 989–990 | TCP | FTPS | ✅ Encrypted (SSL/TLS) |
| 69 | UDP | TFTP | ❌ Unencrypted (simple) |

### 🔐 Remote Access

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 22 | TCP | SSH | Secure shell — encrypted remote access |
| 23 | TCP | Telnet | ⚠️ Unencrypted — avoid using |
| 3389 | TCP | RDP | Windows Remote Desktop Protocol |
| 5900 | TCP | VNC | Virtual Network Computing |

---

## Database & Directory Services

### 🗄️ Database Ports

| Port | Protocol | Database |
|------|----------|----------|
| 3306 | TCP | MySQL / MariaDB |
| 5432 | TCP | PostgreSQL |
| 1433 | TCP | Microsoft SQL Server |
| 27017 | TCP | MongoDB |
| 6379 | TCP | Redis |
| 1521 | TCP | Oracle Database |

### 📂 Directory Services

| Port | Protocol | Service |
|------|----------|---------|
| 389 | TCP | LDAP (Lightweight Directory Access Protocol) |
| 636 | TCP | LDAPS (LDAP over SSL) |
| 88 | TCP/UDP | Kerberos Authentication |

---

## Streaming & Media

> **Why UDP?** Streaming services use UDP because speed is more important than perfect delivery. A few dropped frames are better than lag!

| Port | Protocol | Service |
|------|----------|---------|
| 554 | TCP/UDP | RTSP (Real-Time Streaming Protocol) |
| 1935 | TCP | RTMP (Real-Time Messaging Protocol) |
| 5060–5061 | TCP/UDP | SIP (Session Initiation Protocol — VoIP) |
| 5004–5005 | UDP | RTP (Real-time Transport Protocol) |

---

## Quick Reference

### Most Critical Ports to Remember

**🌐 Web & Secure**
- `80` — HTTP
- `443` — HTTPS
- `22` — SSH

**📧 Email**
- `25` — SMTP
- `587` — SMTP (Modern)
- `143` — IMAP
- `993` — IMAPS

**📁 File Transfer**
- `20–21` — FTP
- `22` — SFTP / SCP
- `445` — SMB

**🗄️ Databases**
- `3306` — MySQL
- `5432` — PostgreSQL
- `27017` — MongoDB

**🔧 Network Services**
- `53` — DNS
- `67–68` — DHCP
- `123` — NTP

**🖥️ Remote Access**
- `22` — SSH
- `3389` — RDP
- `5900` — VNC

---

### 🔒 Security Tips — Always Prefer Encrypted Protocols

| Instead of... | Use... |
|---------------|--------|
| HTTP (80) | **HTTPS (443)** |
| Telnet (23) | **SSH (22)** |
| FTP (20–21) | **SFTP (22)** |
| IMAP (143) | **IMAPS (993)** |
| SMTP (25) | **SMTPS (465 / 587)** |

---

## Port Scanning Commands

### Using Netstat

| Command | Description |
|---------|-------------|
| `netstat -tuln` | Show all listening TCP/UDP ports |
| `netstat -tupn` | Show all connections with process IDs |
| `netstat -an \| grep LISTEN` | Show only listening ports |

### Using ss (Modern Alternative)

| Command | Description |
|---------|-------------|
| `ss -tuln` | Show all listening TCP/UDP ports |
| `ss -tupn` | Show all connections with process names |
| `ss -ltn` | Show only TCP listening ports |

### Using Nmap

| Command | Description |
|---------|-------------|
| `nmap localhost` | Scan common ports on localhost |
| `nmap -p- localhost` | Scan all ports (1–65535) |
| `nmap -sV 192.168.1.1` | Scan with service version detection |
| `nmap -p 80,443 example.com` | Scan specific ports |
