# Network Ports: A Complete Progressive Tutorial

---

## 1. What & Why

A network port is a number from 0 to 65535 that identifies a specific process or service on a machine. When two computers communicate, data flows between an IP address and port number on each end — the four-tuple (source IP, source port, destination IP, destination port) uniquely identifies each connection.

Why do ports exist? An IP address identifies a machine. A port identifies which program on that machine should receive the data. Without ports, your computer couldn't run a web server and an SSH daemon simultaneously — all incoming traffic would pile up at the IP layer with no way to route it to the right process.

Understanding ports is mandatory for: configuring firewalls, debugging connectivity issues, analyzing network traffic, building services, securing servers, and doing penetration testing. When you're troubleshooting why your application can't connect to a database, or why a service is refusing connections, ports are almost always the starting point.

---

## 2. Mental Model

Think of an IP address as a building's street address and ports as numbered offices inside the building.

```
Building (IP): 192.168.1.100
│
├── Office 22  → SSH daemon   (waiting for secure shell connections)
├── Office 80  → Nginx        (serving HTTP)
├── Office 443 → Nginx        (serving HTTPS)
├── Office 5432 → PostgreSQL  (accepting database connections)
└── Office 8080 → Your app   (development server)

A letter (packet) addressed to:
  192.168.1.100 : 443
goes directly to the office managing HTTPS.
No other process sees it.
```

When you run `curl https://example.com`, your OS:
1. Opens a socket on a random ephemeral port on your machine (e.g., 54231)
2. Connects to example.com port 443
3. The connection is: (your IP:54231) ↔ (example.com:443)
4. When the response arrives at your IP:54231, the OS delivers it to curl

The kernel's socket layer routes incoming packets to the right process based on which process called `bind()` on that port.

---

## 3. Progressive Examples

### Level 1: TCP vs UDP — Choosing the Right Protocol

```
TCP (Transmission Control Protocol)
────────────────────────────────────
Connection-oriented: Three-way handshake before data flows.

  Client              Server
    │                    │
    │──── SYN ──────────>│   "I want to connect"
    │<─── SYN-ACK ───────│   "OK, I'm ready"
    │──── ACK ──────────>│   "Connection established"
    │                    │
    │═══ DATA FLOWS ═════│   Reliable, ordered, error-checked
    │                    │
    │──── FIN ──────────>│   Graceful teardown
    │<─── FIN-ACK ───────│

Guarantees:
  - Every byte arrives (retransmitted if lost)
  - Bytes arrive in order
  - No duplicates
  - Flow control (receiver can throttle sender)

Cost:
  - Minimum 1.5 RTT to establish connection
  - ~20-60 byte header overhead per packet
  - State maintained at both ends

Use for: HTTP/HTTPS, SSH, SMTP, databases, file transfer
         — any time data integrity matters more than speed.

UDP (User Datagram Protocol)
─────────────────────────────
Connectionless: Send and forget. No handshake.

  Client              Server
    │                    │
    │══ DATAGRAM ════════│   Sent immediately, no setup
    │══ DATAGRAM ════════│   May arrive out of order
    │   (lost) ──────────│   May not arrive at all
    │══ DATAGRAM ════════│   No notification of loss

Guarantees: none.

Benefits:
  - Zero connection setup latency
  - 8-byte header (vs TCP's 20-60)
  - Multicast/broadcast support (one sender, many receivers)
  - Application controls retransmission logic

Use for: DNS, video streaming, gaming, VoIP, NTP
         — when speed matters more than perfect delivery,
           or when you implement reliability yourself (QUIC does this).
```

### Level 2: The Port Number Ranges

```
Range           Classification    Who Uses It
──────────────────────────────────────────────────────────
0 – 1023        Well-Known        IANA-reserved for standard services.
                (System ports)    Require root/admin to bind on Linux/macOS.
                                  Never use these for your own applications.

1024 – 49151    Registered        IANA-registered for specific applications.
                                  Don't need root to bind.
                                  Avoid well-known registered ports (3306, 5432, 6379)
                                  for your own services unless that IS the service.

49152 – 65535   Dynamic/          OS-assigned ephemeral ports for outgoing connections.
                Ephemeral         You never bind here; the kernel picks from this range
                                  when your program opens a client socket.
```

```bash
# See which ports are currently listening on your system
ss -tuln          # Linux (fast, modern)
netstat -tuln     # older Linux/macOS
lsof -iTCP -sTCP:LISTEN  # macOS, shows process names

# See which process owns a specific port
lsof -i :8080
ss -tulnp | grep 8080

# Check the ephemeral port range on Linux
cat /proc/sys/net/ipv4/ip_local_port_range
# Typically: 32768   60999

# Connect to a port manually (test if it's accepting connections)
nc -zv example.com 443    # -z: scan, -v: verbose
telnet localhost 5432      # works but insecure; nc is preferred

# Port scan your own machine
nmap -p 1-65535 localhost  # scan all ports
nmap -sV localhost          # detect service versions
```

### Level 3: The Essential Ports Every Developer Must Know

```
REMOTE ACCESS
─────────────────────────────────────────────────────────────
Port 22   TCP   SSH (Secure Shell)
                The standard for encrypted remote server access.
                Also used by SCP (secure copy) and SFTP (secure FTP).
                Default target for brute-force attacks — consider changing
                to a high port number (e.g., 2222) on internet-facing servers.

Port 23   TCP   Telnet (OBSOLETE — never use)
                Sends everything including passwords in plaintext.
                Still found in legacy IoT devices and industrial equipment.

WEB
─────────────────────────────────────────────────────────────
Port 80   TCP   HTTP
                Unencrypted web traffic.
                Modern practice: always redirect 80 → 443.
                Still used inside private networks and for health checks.

Port 443  TCP   HTTPS (HTTP over TLS)
                All modern web traffic should use this.
                Also used by HTTP/3 over UDP (QUIC protocol).

Port 8080 TCP   HTTP (alternative / development)
                Convention for running a second HTTP server or dev server.
                Not a system port — no root required.
                Also used by some Java app servers and API proxies.

Port 8443 TCP   HTTPS (alternative)
                Convention for HTTPS on a non-system port.

EMAIL
─────────────────────────────────────────────────────────────
Port 25   TCP   SMTP — server-to-server email relay
                ISPs typically block this for residential IPs to stop spam.
                Your app should NOT use this to send email.

Port 587  TCP   SMTP submission (STARTTLS)
                The correct port for sending email from an application.
                Requires authentication. Uses STARTTLS to upgrade to TLS.

Port 465  TCP   SMTPS (legacy — SMTP over SSL)
                Older approach. Some services still require it.

Port 993  TCP   IMAPS (IMAP over SSL)
                Email client retrieval with server-side message storage.
                Use this, not 143 (unencrypted IMAP).

Port 995  TCP   POP3S (POP3 over SSL)
                Downloads email to local client, deletes from server.
                Use this, not 110 (unencrypted POP3).

DNS
─────────────────────────────────────────────────────────────
Port 53   TCP/UDP   DNS (Domain Name System)
                    UDP for queries (fast, small).
                    TCP for large responses (DNSSEC, zone transfers).
                    DNS over HTTPS (DoH): port 443.
                    DNS over TLS (DoT): port 853.

DATABASES
─────────────────────────────────────────────────────────────
Port 3306  TCP   MySQL / MariaDB
Port 5432  TCP   PostgreSQL
Port 1433  TCP   Microsoft SQL Server
Port 1521  TCP   Oracle Database
Port 27017 TCP   MongoDB
Port 6379  TCP   Redis
Port 5672  TCP   RabbitMQ (AMQP)
Port 9200  TCP   Elasticsearch (HTTP API)
Port 9300  TCP   Elasticsearch (cluster communication)

RULE: Database ports must NEVER be exposed to the internet.
Always bind to 127.0.0.1 or use firewall rules to restrict access.

INFRASTRUCTURE
─────────────────────────────────────────────────────────────
Port 67/68 UDP   DHCP (server/client)
                 Automatic IP address assignment.

Port 123   UDP   NTP (Network Time Protocol)
                 Time synchronization. Critical for certificates, logs,
                 distributed systems — time drift breaks authentication.

Port 161   UDP   SNMP (Simple Network Management Protocol)
                 Monitoring network devices (routers, switches).

Port 514   UDP   Syslog
                 Centralized logging.

Port 2181  TCP   ZooKeeper
Port 2379  TCP   etcd (Kubernetes configuration store)
Port 6443  TCP   Kubernetes API server
Port 10250 TCP   Kubernetes kubelet
```

### Level 4: Diagnosing Connectivity Problems

```bash
# Step-by-step: debugging "connection refused" or "connection timed out"

# 1. Is the service running?
systemctl status nginx
ps aux | grep postgres

# 2. Is it listening on the right port and address?
ss -tulnp | grep 5432
# Look for: 127.0.0.1:5432 (only localhost) vs 0.0.0.0:5432 (all interfaces)
# If it shows 127.0.0.1:5432, remote connections will be refused.
# postgres config: listen_addresses = '*'  (or specific IP)

# 3. Is the firewall blocking it?
ufw status verbose           # Ubuntu
firewall-cmd --list-all      # CentOS/RHEL
iptables -L -n | grep 5432   # raw iptables

# Allow a port through ufw:
ufw allow 5432/tcp

# 4. Can you reach the port from the connecting machine?
nc -zv database-host 5432    # -z: test only, -v: verbose
# "Connection refused" = port closed or firewall dropping immediately
# "Connection timed out" = firewall dropping silently (worse for debugging)

# 5. Check if a high-level connection works
psql -h database-host -U myuser -d mydb

# 6. Trace the full path
traceroute database-host     # shows each hop to destination
mtr database-host            # continuous trace with loss/latency stats

# 7. Capture traffic to see what's actually happening
tcpdump -i eth0 port 5432 -n
# Shows every packet — SYN, SYN-ACK, RST (reset = connection refused)

# Common "connection refused" causes:
# - Service not running
# - Service bound to 127.0.0.1 but connecting remotely
# - Wrong port number
# - Firewall rejecting (RST) vs dropping (timeout)
```

### Level 5: Port Security Patterns

```bash
# Principle of least exposure: close everything, open only what's needed

# See your current attack surface
ss -tulnp

# Secure SSH (the highest-value target)
# /etc/ssh/sshd_config:
Port 2222                  # non-standard port reduces noise from scanners
PermitRootLogin no         # never login as root directly
PasswordAuthentication no  # key-only authentication
AllowUsers alice bob       # whitelist specific users
MaxAuthTries 3             # limit brute-force attempts

# After changes:
systemctl reload sshd

# Block everything, allow specific ports (ufw example)
ufw default deny incoming
ufw default allow outgoing
ufw allow 2222/tcp    # SSH (your non-standard port)
ufw allow 80/tcp      # HTTP
ufw allow 443/tcp     # HTTPS
ufw enable

# Allow only specific source IPs to reach a port
ufw allow from 10.0.0.0/8 to any port 5432  # PostgreSQL from private network only

# Bind database to localhost only (postgres example)
# /etc/postgresql/*/main/postgresql.conf:
# listen_addresses = 'localhost'
# (restart postgres after change)

# Use SSH tunneling to access a remote database safely
# (never expose database port — tunnel through SSH instead)
ssh -L 5432:localhost:5432 user@database-server
# Now connect to localhost:5432 — it tunnels through SSH to the remote postgres
psql -h localhost -p 5432 -U myuser mydb

# Check which ports are exposed to the internet from outside
nmap -sV your-server-ip   # scan from a remote machine
# Any open port is a potential attack surface. Know what's open and why.
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Binding a database to 0.0.0.0 and forgetting to firewall it**

```bash
# WRONG: PostgreSQL config
listen_addresses = '*'    # binds to all interfaces, including the internet

# If firewall is not configured, anyone on the internet can attempt to connect.
# Default credentials or weak passwords → complete database compromise.

# CORRECT: bind only to needed interfaces
listen_addresses = 'localhost, 10.0.1.5'   # localhost + internal IP only

# OR: bind to all but firewall the port
ufw allow from 10.0.0.0/8 to any port 5432
ufw deny 5432
```

**Mistake 2: Using port 25 to send application email**

```bash
# WRONG: connecting to an SMTP server on port 25 from your app
# Port 25 is for server-to-server relay, not client submission.
# Most ISPs and cloud providers block outbound port 25.

# CORRECT: use port 587 (submission) with authentication
# Python example:
import smtplib
from email.mime.text import MIMEText

msg = MIMEText("Hello")
msg['Subject'] = 'Test'
msg['From'] = 'sender@example.com'
msg['To'] = 'recipient@example.com'

with smtplib.SMTP('smtp.gmail.com', 587) as s:   # port 587
    s.starttls()          # upgrade to TLS
    s.login('user', 'app-password')
    s.send_message(msg)
```

**Mistake 3: "Connection refused" vs "Connection timed out" — misdiagnosing**

```
Connection refused = RST packet received immediately
  → The machine is reachable but nothing is listening on that port
  → Or: firewall sends RST (reject, not drop)

Connection timed out = no response
  → Machine unreachable (wrong IP, routing issue)
  → Or: firewall silently drops packets (DROP rule, not REJECT)
  → Network-level block between you and the server

"Connection refused" is actually BETTER for debugging — the server is up and responding.
"Timed out" means you can't even reach the machine.
```

**Mistake 4: Assuming port numbers identify the protocol definitively**

```bash
# Port 443 is HTTPS by convention, but anything can run there.
# Nmap's -sV flag detects the ACTUAL service running on a port.

nmap -sV -p 443 target.example.com
# Might reveal: ssh (some sysadmins move SSH to 443 to bypass port-blocking firewalls)
# Might reveal: a custom protocol running over TLS

# Never assume: always verify with service detection.
```

---

## 5. The "Why Does This Work" Layer

### How the OS Routes Packets to the Right Process

When a process calls `bind(socket, 0.0.0.0:443)`, the kernel registers that socket in a hash table keyed by (protocol, local_IP, local_port). When a packet arrives at the network interface:

1. The NIC delivers the frame to the kernel's network stack
2. Layer 3 processing: kernel reads the destination IP, determines it's for this machine
3. Layer 4 processing: kernel reads the destination port number from the TCP/UDP header
4. Hash table lookup: kernel finds the socket bound to that port
5. The packet's payload is delivered to that socket's receive buffer
6. The process's `recv()` or `read()` call wakes up and reads the data

If no process is bound to that port: the kernel sends a TCP RST packet (for TCP) or an ICMP port unreachable message (for UDP) — this is what you see as "connection refused."

### Why Ephemeral Ports Matter for Connection Tracking

When you open a TCP connection as a client, you don't bind to a port — the OS assigns one from the ephemeral range. The connection is tracked by the full 4-tuple: (client_IP:54231, server_IP:443).

This matters for firewalls. A stateful firewall tracks these connections and knows that incoming packets from server_IP:443 to client_IP:54231 are responses to an established connection — not an unsolicited incoming connection. This is why your laptop can browse the web without opening any inbound firewall rules: the firewall permits response traffic because it tracks the outgoing connections.

---

## 6. Quick Reference

### Critical Ports (Memorize These)

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 22 | TCP | SSH | Key-based auth only; consider changing |
| 25 | TCP | SMTP relay | Block inbound; don't use for app email |
| 53 | TCP/UDP | DNS | UDP for queries, TCP for large responses |
| 80 | TCP | HTTP | Redirect to 443 |
| 443 | TCP | HTTPS | Primary web port |
| 587 | TCP | SMTP submission | Use for sending email from apps |
| 3306 | TCP | MySQL | Never expose to internet |
| 5432 | TCP | PostgreSQL | Never expose to internet |
| 6379 | TCP | Redis | Never expose to internet |
| 27017 | TCP | MongoDB | Never expose to internet |

### TCP vs UDP Decision

| Use TCP when... | Use UDP when... |
|----------------|----------------|
| Data must arrive intact | Speed is critical |
| Order matters | Some loss is acceptable |
| You need flow control | You implement your own reliability |
| Reliability is non-negotiable | Multicast/broadcast needed |
| Web, email, databases, SSH | DNS, gaming, video, VoIP, NTP |

### Diagnostic Commands

```bash
# What's listening?
ss -tulnp

# Who owns port X?
lsof -i :X
ss -tulnp | grep :X

# Can I reach a port?
nc -zv host port

# Full packet capture
tcpdump -i eth0 port X -n

# Firewall status
ufw status verbose
iptables -L -n -v
```
