# Network Ports Reference Guide

> A complete reference for TCP/UDP ports, network protocols, and common services.

---

## What Are Ports?

Network ports are virtual communication endpoints. Every connection between two networked machines is identified by a combination of IP address and port number, allowing the operating system to route traffic to the correct service or application.

Ports are numbered from 0 to 65535 and are divided into three ranges:

| Range | Classification | Description |
|---|---|---|
| 0 – 1023 | Well-Known (System) | Reserved for standard, widely-used services such as HTTP, FTP, and SSH |
| 1024 – 49151 | Registered | Assigned to specific applications and services by IANA |
| 49152 – 65535 | Dynamic / Private | Used for temporary client-side connections and ephemeral ports |

---

## TCP vs UDP

### TCP — Transmission Control Protocol

TCP is connection-oriented and prioritizes reliability over speed. Before data is exchanged, a connection is established using a three-way handshake (SYN, SYN-ACK, ACK).

Key characteristics:
- Guarantees delivery — lost packets are retransmitted automatically.
- Ensures packets are received and reassembled in order.
- Includes flow control and congestion control mechanisms.
- Higher overhead due to acknowledgments and state tracking.
- Header size: 20–60 bytes.

Best suited for: web browsing, email, file transfers, and remote access — any scenario where data integrity matters.

---

### UDP — User Datagram Protocol

UDP is connectionless and prioritizes speed over reliability. Data is sent without establishing a connection or waiting for acknowledgment.

Key characteristics:
- No delivery guarantee — packets may be lost or arrive out of order.
- No handshake or retransmission logic.
- Minimal overhead and lower latency.
- Header size: 8 bytes.

Best suited for: video streaming, online gaming, VoIP, DNS queries, and live broadcasts — scenarios where speed matters more than perfect delivery.

---

### Side-by-Side Comparison

| Feature | TCP | UDP |
|---|---|---|
| Connection model | Connection-oriented (handshake required) | Connectionless (no handshake) |
| Delivery guarantee | Yes — lost packets are retransmitted | No — packets may be dropped |
| Speed | Slower due to acknowledgments and error checking | Faster due to minimal overhead |
| Data model | Stream-based (continuous flow) | Datagram-based (individual packets) |
| Packet ordering | Guaranteed in-order delivery | No ordering guarantee |
| Flow control | Yes (sliding window, congestion control) | No |
| Error handling | Detection and correction via retransmission | Detection only (checksum) |
| Resource usage | Higher (state tracking, buffers) | Lower (minimal overhead) |

---

## Common Ports

| Port | Protocol | Service |
|---|---|---|
| 20 | TCP/UDP | FTP — Data transfer channel |
| 21 | TCP/UDP | FTP — Command and control channel |
| 22 | TCP/UDP | SSH, SFTP, SCP — Secure remote access and file transfer |
| 23 | TCP | Telnet — Unencrypted remote login (deprecated) |
| 25 | TCP | SMTP — Email transfer between mail servers |
| 53 | TCP/UDP | DNS — Domain name resolution |
| 67 | UDP | DHCP — Server side (assigns IP addresses) |
| 68 | UDP | DHCP — Client side (requests IP addresses) |
| 69 | UDP | TFTP — Trivial File Transfer Protocol |
| 80 | TCP/UDP | HTTP — Unencrypted web traffic |
| 110 | TCP | POP3 — Email retrieval |
| 123 | UDP | NTP — Network time synchronization |
| 135 | TCP | Microsoft RPC / DCOM services |
| 137 | TCP/UDP | NetBIOS Name Service |
| 138 | UDP | NetBIOS Datagram Service |
| 139 | TCP | NetBIOS Session Service |
| 143 | TCP | IMAP — Email retrieval with server-side sync |
| 161 | UDP | SNMP — Network device monitoring |
| 162 | UDP | SNMP Trap — Alert notifications from devices |
| 179 | TCP/UDP | BGP — Border Gateway Protocol (inter-AS routing) |
| 389 | TCP | LDAP — Lightweight Directory Access Protocol |
| 443 | TCP/UDP | HTTPS — Encrypted web traffic (TLS) |
| 445 | TCP | SMB — Windows file sharing and Active Directory |
| 465 | TCP | SMTPS — SMTP over SSL (legacy) |
| 514 | UDP | Syslog — System and application logging |
| 515 | TCP | LPD — Line Printer Daemon |
| 587 | TCP | SMTP — Modern mail submission (with STARTTLS) |
| 631 | TCP | IPP — Internet Printing Protocol |
| 636 | TCP | LDAPS — LDAP over SSL |
| 989 | TCP | FTPS — Data channel (FTP over SSL) |
| 990 | TCP | FTPS — Control channel (FTP over SSL) |
| 993 | TCP | IMAPS — IMAP over SSL |
| 995 | TCP | POP3S — POP3 over SSL |

---

## Web and Email

### Web Traffic

| Port | Protocol | Service |
|---|---|---|
| 80 | TCP | HTTP — Standard unencrypted web traffic |
| 443 | TCP | HTTPS — Encrypted web traffic over TLS |
| 8080 | TCP | HTTP (alternative) — Commonly used for proxies and development servers |

### Email Services

Understanding which port to use depends on the role of the connection — whether it is transferring mail between servers, submitting mail from a client, or retrieving mail from a mailbox.

| Port | Protocol | Service | Purpose |
|---|---|---|---|
| 25 | TCP | SMTP | Server-to-server mail transfer |
| 587 | TCP | SMTP | Client mail submission (current standard, uses STARTTLS) |
| 465 | TCP | SMTPS | SMTP over SSL — legacy, still used by some providers |
| 110 | TCP | POP3 | Retrieves email and removes it from the server |
| 995 | TCP | POP3S | POP3 over SSL |
| 143 | TCP | IMAP | Retrieves email while keeping it synced on the server |
| 993 | TCP | IMAPS | IMAP over SSL |

---

## File Transfer and Remote Access

### File Transfer Protocols

| Port | Protocol | Service | Encryption |
|---|---|---|---|
| 20–21 | TCP | FTP | None — credentials and data sent in plaintext |
| 22 | TCP | SFTP / SCP | Encrypted via SSH |
| 989–990 | TCP | FTPS | Encrypted via SSL/TLS |
| 69 | UDP | TFTP | None — simple, minimal protocol for local networks |

### Remote Access

| Port | Protocol | Service | Notes |
|---|---|---|---|
| 22 | TCP | SSH | Encrypted shell access — the standard for secure remote administration |
| 23 | TCP | Telnet | Unencrypted — should not be used on any production system |
| 3389 | TCP | RDP | Windows Remote Desktop Protocol |
| 5900 | TCP | VNC | Virtual Network Computing — cross-platform remote desktop |

---

## Database and Directory Services

### Database Ports

These ports are typically not exposed to the public internet. They should be accessible only from trusted internal hosts or application servers.

| Port | Protocol | Database |
|---|---|---|
| 1433 | TCP | Microsoft SQL Server |
| 1521 | TCP | Oracle Database |
| 3306 | TCP | MySQL / MariaDB |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 27017 | TCP | MongoDB |

### Directory Services

| Port | Protocol | Service |
|---|---|---|
| 88 | TCP/UDP | Kerberos — Network authentication protocol |
| 389 | TCP | LDAP — Directory queries and authentication |
| 636 | TCP | LDAPS — LDAP over SSL |

---

## Streaming and Media

Streaming protocols favor UDP because latency is more disruptive to the user experience than occasional packet loss. A momentary drop in video quality is preferable to buffering caused by waiting for retransmission.

| Port | Protocol | Service |
|---|---|---|
| 554 | TCP/UDP | RTSP — Real-Time Streaming Protocol (camera feeds, media servers) |
| 1935 | TCP | RTMP — Real-Time Messaging Protocol (live streaming platforms) |
| 5004–5005 | UDP | RTP — Real-time Transport Protocol (audio/video over IP) |
| 5060–5061 | TCP/UDP | SIP — Session Initiation Protocol (VoIP call setup) |

---

## Quick Reference

### High-Priority Ports to Know

**Web and Security**

| Port | Service |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |

**Email**

| Port | Service |
|---|---|
| 25 | SMTP (server-to-server) |
| 587 | SMTP (client submission) |
| 143 | IMAP |
| 993 | IMAPS |

**File Transfer**

| Port | Service |
|---|---|
| 20–21 | FTP |
| 22 | SFTP / SCP |
| 445 | SMB |

**Databases**

| Port | Service |
|---|---|
| 3306 | MySQL / MariaDB |
| 5432 | PostgreSQL |
| 27017 | MongoDB |

**Network Services**

| Port | Service |
|---|---|
| 53 | DNS |
| 67–68 | DHCP |
| 123 | NTP |

**Remote Access**

| Port | Service |
|---|---|
| 22 | SSH |
| 3389 | RDP |
| 5900 | VNC |

---

## Security Recommendations

Where possible, always prefer the encrypted variant of a protocol. Unencrypted protocols transmit credentials and data in plaintext, making them vulnerable to interception on any shared or untrusted network.

| Avoid | Use Instead | Reason |
|---|---|---|
| HTTP (80) | HTTPS (443) | Encrypts all traffic with TLS |
| Telnet (23) | SSH (22) | Encrypts the session and authenticates the server |
| FTP (20–21) | SFTP (22) | Encrypts both credentials and file data |
| IMAP (143) | IMAPS (993) | Encrypts email retrieval |
| SMTP (25) | SMTP with STARTTLS (587) | Encrypts mail submission |

Additional hardening steps to consider:
- Close or firewall any port not actively required by your services.
- Change default ports for high-value services such as SSH to reduce automated scanning noise.
- Use network-level access controls (firewall rules, VPNs) to restrict database ports to known internal hosts only.
- Audit listening ports regularly using the commands in the next section.

---

## Port Scanning Commands

### netstat

`netstat` is available on most Unix-like and Windows systems. It reports active connections and listening ports.

| Command | Description |
|---|---|
| `netstat -tuln` | List all listening TCP and UDP ports |
| `netstat -tupn` | List all active connections with associated process IDs |
| `netstat -an \| grep LISTEN` | Filter output to show only listening sockets |

### ss — Socket Statistics

`ss` is the modern replacement for `netstat` on Linux. It is faster and provides more detail.

| Command | Description |
|---|---|
| `ss -tuln` | List all listening TCP and UDP ports |
| `ss -tupn` | List all connections with process names and PIDs |
| `ss -ltn` | List only listening TCP ports |

### nmap — Network Mapper

`nmap` is a dedicated network scanning tool. Use only on systems and networks you have explicit permission to scan.

| Command | Description |
|---|---|
| `nmap localhost` | Scan commonly used ports on localhost |
| `nmap -p- localhost` | Scan all 65535 ports on localhost |
| `nmap -sV 192.168.1.1` | Scan with service and version detection |
| `nmap -p 80,443 example.com` | Scan specific ports on a target host |
