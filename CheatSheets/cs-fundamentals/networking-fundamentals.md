# Networking Fundamentals: A Complete Progressive Tutorial

---

## 1. What & Why

Networking is how computers communicate. Every web request, database query, API call, SSH connection, and file download relies on a layered stack of protocols working together. Understanding these fundamentals is essential for every software engineer — not just system administrators.

Why does a backend developer need to understand networking? Because when your application times out, drops packets, has latency spikes, or fails to connect to a database, the cause is almost always in the network layer. Debugging these problems requires understanding TCP handshakes, IP routing, DNS resolution, TLS negotiation, and port binding. Without this knowledge, you are guessing.

The three ideas that explain all networking: addressing (how we identify endpoints), routing (how data finds its way), and protocols (the rules governing communication at each layer).

---

## 2. Mental Model

Networking is organized into layers. Each layer has a specific job, communicates only with adjacent layers, and adds a header to the data as it passes through.

```
OSI Layer        TCP/IP Layer      What It Does                  Examples
─────────────────────────────────────────────────────────────────────────
7. Application                     User-facing protocols          HTTP, DNS, SMTP, SSH
6. Presentation  Application       Encoding, encryption, format   TLS, JSON, JPEG
5. Session                         Session management             RPC, WebSockets
4. Transport     Transport         End-to-end delivery, ports     TCP, UDP
3. Network       Internet          IP addressing, routing         IP, ICMP
2. Data Link     Network Access    MAC addressing, frames         Ethernet, Wi-Fi
1. Physical                        Signals, wires, bits           Copper, fiber, radio

When you send an HTTP request:
  Browser creates HTTP message (layer 7)
  TLS encrypts it (layer 6)
  TCP wraps it in a segment with ports (layer 4)
  IP wraps that in a packet with addresses (layer 3)
  Ethernet wraps that in a frame with MACs (layer 2)
  Sent as electrical signals on the wire (layer 1)

Each layer removes its header on the receiving end — "de-encapsulation."
```

The real-world model is TCP/IP, which collapses OSI's 7 layers into 4. OSI is the conceptual reference. TCP/IP is what the internet actually runs.

---

## 3. Progressive Examples

### Level 1: IP Addressing and Subnetting

```
IPv4 is 32 bits written as four decimal octets: 192.168.1.100

In binary:
  192      168       1        100
11000000.10101000.00000001.01100100

Every IPv4 address has two parts: network prefix + host ID.
The subnet mask tells you how many bits are the network prefix.

CIDR notation: 192.168.1.100/24
  /24 means: first 24 bits = network, remaining 8 bits = host
  Subnet mask: 255.255.255.0

To find the network address: IP AND subnet_mask
  192.168.1.100 = 11000000.10101000.00000001.01100100
  255.255.255.0 = 11111111.11111111.11111111.00000000
  Network addr  = 11000000.10101000.00000001.00000000 = 192.168.1.0

To find the broadcast address: set all host bits to 1
  Broadcast = 192.168.1.255

Usable hosts: 192.168.1.1 to 192.168.1.254 (254 hosts)
Formula: 2^(32 - prefix) - 2 (subtract network and broadcast addresses)
```

```python
# Subnet math in Python
import ipaddress

network = ipaddress.IPv4Network("192.168.1.0/24")
print(f"Network:   {network.network_address}")     # 192.168.1.0
print(f"Netmask:   {network.netmask}")              # 255.255.255.0
print(f"Broadcast: {network.broadcast_address}")    # 192.168.1.255
print(f"Hosts:     {network.num_addresses - 2}")    # 254

# Check if an address is in the network
ip = ipaddress.IPv4Address("192.168.1.50")
print(f"In network: {ip in network}")   # True

# Split a network into subnets
for subnet in network.subnets(prefixlen_diff=2):  # /24 → four /26 subnets
    print(subnet, f"({subnet.num_addresses - 2} hosts)")

# Common CIDR prefixes
cidr_table = {
    8:  ("255.0.0.0",       16_777_214),
    16: ("255.255.0.0",        65_534),
    24: ("255.255.255.0",         254),
    25: ("255.255.255.128",       126),
    26: ("255.255.255.192",        62),
    27: ("255.255.255.224",        30),
    28: ("255.255.255.240",        14),
    29: ("255.255.255.248",         6),
    30: ("255.255.255.252",         2),
}
```

**Special IPv4 addresses you must know:**

```
10.0.0.0/8          Private (RFC 1918) — 16M addresses, large internal networks
172.16.0.0/12       Private (RFC 1918) — 1M addresses
192.168.0.0/16      Private (RFC 1918) — 65K addresses, home/office networks
127.0.0.0/8         Loopback — 127.0.0.1 always means "this machine"
0.0.0.0             "Any" address (listen on all interfaces)
255.255.255.255     Limited broadcast (all hosts on local subnet)
169.254.0.0/16      Link-local (APIPA) — no DHCP server found
```

### Level 2: TCP — The Reliable Transport

```
TCP is connection-oriented. Before any data flows, both sides go through
a three-way handshake to establish synchronized sequence numbers.

THREE-WAY HANDSHAKE:
                    Client                    Server
                       │                         │
           SYN ────────┼──── SYN ───────────────>│  "I want to connect,
                       │                         │   my seq# = 1000"
                       │<─── SYN-ACK ────────────│  "OK, my seq# = 5000,
     SYN-ACK ──────────┼                         │   I acknowledge 1001"
                       │──── ACK ───────────────>│  "Acknowledged 5001"
           ACK ────────┼                         │
                       │════ DATA FLOWS ═════════│
                       │
CONNECTION TEARDOWN (four-way):
  Client: FIN  →  Server: ACK  →  Server: FIN  →  Client: ACK
  (Server may still send data after the first FIN)

KEY MECHANISMS:
  Sequence numbers: every byte has a number; receiver reorders out-of-order segments
  Acknowledgments:  receiver confirms bytes received; sender retransmits on timeout
  Flow control:     receiver advertises window size — how much it can buffer
  Congestion control: sender reduces rate when packet loss detected (AIMD algorithm)

TCP STATES (important for debugging):
  LISTEN      : waiting for connection (server side)
  SYN_SENT    : handshake started (client side)
  ESTABLISHED : connection active, data flowing
  TIME_WAIT   : connection closed, waiting for late packets (lasts 2×MSL ≈ 60-120s)
  CLOSE_WAIT  : remote side closed, waiting for local application to close
  FIN_WAIT_2  : waiting for remote FIN
```

```bash
# Inspect TCP connections
ss -tuln                        # listening sockets
ss -tupn                        # established connections with process names
ss -s                           # summary statistics
netstat -an | grep TIME_WAIT | wc -l  # count TIME_WAIT sockets

# Capture and analyze TCP traffic
sudo tcpdump -i eth0 port 443 -n         # capture HTTPS traffic
sudo tcpdump -i eth0 'tcp[13] & 2 != 0' # capture SYN packets only
sudo tcpdump -w capture.pcap -i eth0     # save to file for Wireshark analysis
```

### Level 3: DNS — The Internet's Phone Book

```
DNS translates human-readable names (example.com) to IP addresses (93.184.216.34).
Without DNS, you'd memorize IP addresses for every service.

RESOLUTION HIERARCHY:
  1. Browser cache (recent lookups, respects TTL)
  2. OS cache (/etc/hosts, systemd-resolved)
  3. Recursive resolver (your ISP's or 8.8.8.8)
  4. Root nameservers (13 clusters, know who manages .com, .org, etc.)
  5. TLD nameservers (know who manages example.com)
  6. Authoritative nameservers (know the actual IPs for example.com)

QUERY TRACE for "www.example.com":
  Your machine → Recursive resolver (cache miss)
  Resolver → Root NS: "Who manages .com?" → [.com TLD NS IPs]
  Resolver → .com TLD NS: "Who manages example.com?" → [example.com NS IPs]
  Resolver → example.com NS: "What's the IP of www.example.com?" → 93.184.216.34
  Resolver → Your machine: "93.184.216.34" (caches result for TTL seconds)

DNS RECORD TYPES:
  A     : hostname → IPv4 address
  AAAA  : hostname → IPv6 address
  CNAME : hostname → another hostname (alias)
  MX    : domain → mail server (with priority)
  TXT   : arbitrary text (used for SPF, DKIM, verification)
  NS    : zone → authoritative nameservers
  PTR   : IP → hostname (reverse DNS)
  SOA   : zone metadata (primary NS, admin email, serial, TTL values)
  SRV   : service location (_service._proto.name TTL IN SRV priority weight port target)
```

```bash
# DNS diagnostic commands
dig example.com                    # A record
dig example.com AAAA               # IPv6 address
dig example.com MX                 # mail servers
dig example.com NS                 # nameservers
dig example.com TXT                # text records

dig @8.8.8.8 example.com          # query specific resolver (Google DNS)
dig +short example.com             # output just the IP
dig +trace example.com             # full resolution trace from roots
dig -x 93.184.216.34              # reverse DNS (PTR lookup)

# Check your local DNS resolver
cat /etc/resolv.conf
systemd-resolve --status | grep "DNS Servers"
```

### Level 4: HTTP and HTTPS

```
HTTP is a request-response protocol. The client sends a request with:
  - Method: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
  - URL: /api/users?page=2
  - Headers: Host, Content-Type, Authorization, Accept, ...
  - Body: (for POST/PUT/PATCH)

The server responds with:
  - Status code: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized,
                 403 Forbidden, 404 Not Found, 429 Too Many Requests,
                 500 Internal Server Error, 502 Bad Gateway, 503 Unavailable
  - Headers: Content-Type, Content-Length, Set-Cookie, Cache-Control, ...
  - Body

HTTP VERSIONS:
  HTTP/1.1 (1997): persistent connections, pipelining (rarely used)
                   one request/response at a time per connection (head-of-line blocking)
  HTTP/2  (2015):  multiplexing (many requests over one TCP connection),
                   header compression (HPACK), server push
  HTTP/3  (2022):  QUIC protocol (UDP-based), eliminates TCP head-of-line blocking,
                   faster connection setup (0-RTT), better mobile performance

TLS (HTTPS): TLS 1.3 handshake
  1. Client Hello: cipher suites, TLS version, random, extensions
  2. Server Hello: chosen cipher, certificate, server random
  3. Certificate verification: client verifies against trusted CAs
  4. Key exchange: both derive session keys (using ECDHE)
  5. Finished: encrypted from here on
  
  TLS 1.3: 1-RTT handshake (down from 2-RTT in 1.2)
  0-RTT resumption: for returning clients, data in first packet
```

```bash
# HTTP testing and inspection
curl -v https://example.com              # verbose: show headers
curl -I https://example.com             # HEAD request only (headers)
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

curl -w "\nTime: %{time_total}s\nDNS: %{time_namelookup}s\nTLS: %{time_appconnect}s\n" \
  -o /dev/null -s https://example.com   # timing breakdown

# Check TLS certificate
openssl s_client -connect example.com:443 -servername example.com < /dev/null 2>/dev/null \
  | openssl x509 -text -noout | grep -E "Subject:|Not After|Issuer"

# HTTP/2 check
curl -I --http2 https://example.com | grep "HTTP/"

# Check headers for security
curl -s -I https://example.com | grep -E "Strict-Transport|X-Frame|Content-Security|X-Content-Type"
```

### Level 5: Routing, NAT, and Common Network Topologies

```
ROUTING: How packets find their way across networks

Every router has a routing table: a list of (destination network, next hop) pairs.
When a packet arrives, the router does longest-prefix match and forwards accordingly.

Static routes: manually configured (used for small, stable networks)
Dynamic routing protocols:
  OSPF (Open Shortest Path First): link-state, interior gateway protocol
  BGP  (Border Gateway Protocol):  path-vector, the internet's routing protocol
                                    runs between autonomous systems (ISPs, CDNs)

NAT (Network Address Translation):
  The mechanism that lets millions of devices with private IPs share a few public IPs.
  
  Your router has: public IP = 203.0.113.5
  Your laptop has: private IP = 192.168.1.100 (not routable on internet)
  
  When you make a request to 93.184.216.34:80:
  1. Router rewrites source: 192.168.1.100:54321 → 203.0.113.5:54321
  2. Packet goes to internet
  3. Response arrives at router's public IP:54321
  4. Router rewrites destination back: → 192.168.1.100:54321
  5. Packet delivered to your laptop
  
  Router maintains a NAT table to track these mappings.
  This is called NAPT (Network Address Port Translation) or PAT.

FIREWALL RULES:
  Stateful firewall: tracks connection state
  - Allows established sessions (TCP SYN-ACK, established)
  - Allows related traffic (FTP data connection for a control connection)
  - Default: block all inbound, allow all outbound

  Rule order matters: first matching rule wins.
```

```bash
# View routing table
ip route show
# default via 192.168.1.1 dev eth0    ← default gateway
# 192.168.1.0/24 dev eth0 src 192.168.1.100  ← local network

# Add a static route
ip route add 10.10.0.0/16 via 192.168.1.254

# Trace the route to a destination
traceroute google.com       # Linux/macOS
tracert google.com          # Windows
mtr google.com              # combined ping + traceroute, live updating

# Check iptables firewall rules (Linux)
iptables -L -n -v           # list all rules
iptables -L INPUT -n -v     # INPUT chain only

# Test connectivity through the stack
ping 192.168.1.1             # Layer 3: is the gateway reachable?
ping 8.8.8.8                 # Layer 3: can we reach the internet?
ping google.com              # Layer 3 + 7: does DNS work too?
nc -zv database.internal 5432  # Layer 4: is port 5432 accessible?
```

### Level 6: Diagnosing Network Problems Systematically

```bash
# Systematic troubleshooting: start at Layer 1, work up

# Layer 1 (Physical): Is the interface up?
ip link show eth0
# "state UP" = good, "state DOWN" = cable/port issue

# Layer 3 (Network): Do we have an IP?
ip addr show eth0
# If no address: DHCP failure (run: dhclient eth0)

# Layer 3: Can we reach the gateway?
ip route show | grep default    # find gateway IP
ping -c 3 <gateway_ip>
# If fails: local network issue (switch, cable, VLAN)

# Layer 3: Can we reach the internet?
ping -c 3 8.8.8.8
# Works: internet reachable
# Fails: routing or NAT issue

# Layer 7 (DNS): Does name resolution work?
dig +short google.com
# Returns IP: DNS works
# Empty/SERVFAIL: DNS issue (check /etc/resolv.conf)

# Layer 4: Is the target port open?
nc -zv <host> <port>
# Connection succeeded: port is open
# Connection refused: nothing listening
# Connection timed out: firewall dropping packets

# Application layer: does the service respond correctly?
curl -v http://target:8080/health

# Performance diagnosis
ss -s                           # TCP statistics (retransmits, errors)
netstat -s | grep -i "retransmit\|error\|failed"  # TCP error rates
cat /proc/net/sockstat          # socket usage summary
sar -n DEV 1 5                 # network interface stats (1-sec intervals, 5 times)
iftop                          # live bandwidth by connection
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Confusing latency and bandwidth**

```
Bandwidth: how much data can flow per second (a pipe's diameter)
Latency:   how long it takes data to travel (the pipe's length)

High bandwidth does NOT mean low latency.
A satellite link can have 100 Mbps bandwidth but 600ms RTT latency.
A fiber link can have 1 Gbps bandwidth with 1ms RTT latency.

For interactive applications (web, games, SSH): latency matters more.
For bulk transfers (backups, video streaming): bandwidth matters more.

TCP's throughput is bounded by: Bandwidth × RTT (the bandwidth-delay product).
A 1 Gbps link with 100ms RTT can transfer at most ~12.5 MB of data in flight.
TCP window scaling was invented to overcome this for long-distance fast links.
```

**Mistake 2: Thinking private IP addresses are inherently secure**

Private IP addresses (192.168.x.x, 10.x.x.x) are not routable on the internet — but they are NOT secure from internal attackers. An attacker who compromises any machine inside your network can reach all other machines on private subnets. Defense: use firewalls and network segmentation internally, not just at the perimeter.

**Mistake 3: DNS TTL is not instantaneous**

When you change a DNS record (e.g., updating an A record to point to a new IP), the old record persists in caches worldwide until its TTL expires. A TTL of 3600 means stale records persist for up to an hour. Best practice: reduce TTL to 300 (5 minutes) several days before a planned migration, then change the record, then restore TTL afterward.

**Mistake 4: TCP's TIME_WAIT state is not a bug**

When you see thousands of TIME_WAIT connections, you might try to "fix" it. Don't. TIME_WAIT prevents old packets from being confused with new connections that reuse the same 4-tuple (src_ip:src_port → dst_ip:dst_port). Eliminating it with `SO_REUSEPORT` or `tcp_tw_reuse` can cause subtle data corruption bugs. High TIME_WAIT count is normal for busy servers.

---

## 5. The "Why Does This Work" Layer

### Why TCP's Three-Way Handshake Needs Three Steps, Not Two

The handshake must establish sequence numbers in both directions. Client picks ISN_C (Initial Sequence Number), server picks ISN_S. Each side must confirm it received the other's ISN.

A two-way handshake (SYN + SYN-ACK) would confirm ISN_C at the server and ISN_S at the client, but the server would never know if the client received ISN_S. With the third ACK, the server knows the client is ready to receive data starting at ISN_S + 1, making the connection fully duplex and synchronized.

### How BGP Runs the Internet

BGP (Border Gateway Protocol) is how the ~90,000 autonomous systems (ISPs, cloud providers, enterprises) that make up the internet exchange routing information. Each AS announces the IP prefixes it owns and the ASes it can reach them through. BGP routers maintain a table of millions of prefixes and select the best path based on attributes like AS path length, local preference, and origin.

When a CDN like Cloudflare announces 1.1.1.1/32 from dozens of data centers worldwide, BGP ensures your DNS query goes to the geographically closest one — not because of geographic routing logic, but because the AS path to the nearest data center is shorter.

### Why NAT Breaks End-to-End Connectivity

The internet's original design assumed every device has a globally routable IP address. NAT violated this: devices behind NAT have private IPs and cannot be directly addressed from the internet. This broke peer-to-peer connectivity (file sharing, VoIP, gaming) and forced developers to use relay servers, STUN/TURN protocols, and hole-punching techniques.

IPv6 (128-bit addresses, 340 undecillion unique addresses) was designed to eliminate NAT by giving every device a globally routable IP. Adoption is increasing but slow — over 40% of internet traffic as of 2024, but most enterprise infrastructure still runs on IPv4 with NAT.

---

## 6. Quick Reference

### OSI / TCP-IP Layer Summary

| Layer | Unit | Devices | Protocols | Addresses |
|-------|------|---------|-----------|-----------|
| Application | Data | — | HTTP, DNS, SMTP, SSH | URLs, domain names |
| Transport | Segment | — | TCP, UDP | Ports (0-65535) |
| Network | Packet | Router | IP, ICMP | IP addresses |
| Data Link | Frame | Switch, AP | Ethernet, Wi-Fi | MAC addresses |
| Physical | Bits | Hub, NIC | — | — |

### IP Address Classes (Quick Reference)

| CIDR | Hosts | Common Use |
|------|-------|-----------|
| /30 | 2 | Point-to-point links |
| /29 | 6 | Very small subnets |
| /28 | 14 | Small teams/segments |
| /27 | 30 | Small departments |
| /26 | 62 | Medium segments |
| /25 | 126 | Medium subnets |
| /24 | 254 | Standard subnet |
| /22 | 1,022 | Large networks |
| /16 | 65,534 | Huge networks |

### Common Diagnostic Commands

```bash
ip addr show               # network interfaces and IPs
ip route show              # routing table
ss -tulnp                  # listening ports with processes
ss -tupn                   # established connections
dig +short hostname        # DNS lookup
dig +trace hostname        # full DNS trace
ping -c 4 host            # basic reachability
traceroute host           # path to destination
mtr host                  # continuous path analysis
nc -zv host port          # port connectivity test
tcpdump -i eth0 port 80   # capture traffic
curl -v https://host      # HTTP debugging
```
