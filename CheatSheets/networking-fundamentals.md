# 🌐 The Complete Networking Fundamentals Cheat Sheet
### From Zero to Expert

---

## 1. What is a Network?

A network is a collection of devices that can communicate with each other by exchanging data. Every networking concept — from WiFi to the global internet — builds on three fundamental ideas:

```
Addressing:   How do we identify who we want to talk to?
Routing:      How does data find its way from source to destination?
Protocols:    What language do devices speak, and what rules do they follow?
```

### Types of Networks

```
PAN  (Personal Area Network):   ~1–10m     Bluetooth, USB
LAN  (Local Area Network):      ~100m      Home/office Ethernet, WiFi
MAN  (Metropolitan Area Network): ~city    ISP infrastructure, fiber rings
WAN  (Wide Area Network):       global     The internet, leased lines, MPLS
```

### Key Concepts Before Everything Else

```
Packet switching:
  Data broken into packets. Each packet routed independently.
  Packets may take different paths; reassembled at destination.
  Efficient: network capacity shared among many users.
  The internet is a packet-switched network.

Circuit switching:
  Dedicated physical path established before communication.
  Path reserved for entire duration of call.
  Guaranteed bandwidth; wasted capacity when silent.
  Traditional telephone network (PSTN).

Bandwidth vs Throughput vs Latency:
  Bandwidth:   maximum capacity of a link (e.g., 1 Gbps)
  Throughput:  actual data transferred per second (always ≤ bandwidth)
  Latency:     time for one packet to travel from A to B
  RTT:         Round-Trip Time = latency there + latency back + processing

Jitter:
  Variation in latency. Kills real-time audio/video.
  Caused by: variable queue depths, different routing paths.
  Fixed by: jitter buffer (introduces intentional delay to smooth playback).
```

---

## 2. The OSI & TCP/IP Models

Models describe how networking is organized into layers. Each layer has a specific job and communicates only with the layers directly above and below it.

### OSI Model (7 Layers)

```
Layer  Name            Unit        Function                    Protocols
  7    Application     Data        User-facing services        HTTP, SMTP, DNS, FTP
  6    Presentation    Data        Encryption, compression,    TLS, SSL, JPEG, ASCII
                                   format translation
  5    Session         Data        Manage sessions,            RPC, NetBIOS
                                   dialog control
  4    Transport       Segment     End-to-end delivery,        TCP, UDP, SCTP
                                   port numbers, reliability
  3    Network         Packet      Logical addressing,         IP, ICMP, OSPF, BGP
                                   routing between networks
  2    Data Link       Frame       Physical addressing,        Ethernet, WiFi (802.11),
                                   access to medium            ARP, PPP
  1    Physical        Bits        Electrical/optical signals  Copper, fiber, radio

Mnemonic: "Please Do Not Throw Sausage Pizza Away" (layers 1–7)
      or: "All People Seem To Need Data Processing"  (layers 7–1)
```

### TCP/IP Model (4 Layers)

```
TCP/IP Layer     Corresponds to OSI     Protocols
Application      5 + 6 + 7              HTTP, HTTPS, DNS, SMTP, FTP, SSH, DHCP
Transport        4                      TCP, UDP
Internet         3                      IP (v4/v6), ICMP, ARP
Network Access   1 + 2                  Ethernet, WiFi, PPP

This is what the actual internet uses.
OSI is a conceptual model; TCP/IP is the implementation.
```

### Encapsulation & De-encapsulation

```
When sending data, each layer WRAPS the data with its own header (and sometimes trailer):

Application data:    [          HTTP request           ]
Transport adds:      [TCP header][  HTTP request        ]  ← segment
Network adds:        [IP header ][TCP header][  HTTP   ]  ← packet
Data Link adds:      [ETH hdr   ][IP header][TCP][HTTP][ETH trailer]  ← frame
Physical:            101010101010... (bits on wire)

At the receiver, each layer strips its own header and passes data up.

This is why:
  A router only needs to read up to layer 3 (IP header) to forward packets.
  A switch only needs to read up to layer 2 (MAC address) to forward frames.
  A firewall may read up to layer 4 or 7 depending on type.
```

---

## 3. Physical & Data Link Layer

### Physical Layer

```
Transmission media:

Copper (twisted pair):
  Cat5e:  1 Gbps, 100m
  Cat6:   10 Gbps, 55m (or 1 Gbps, 100m)
  Cat6a:  10 Gbps, 100m
  Used for: LAN (Ethernet), DSL internet connections

Fiber optic:
  Single-mode (SMF): 1 laser, long distance (km–100km), expensive
  Multi-mode (MMF): LED/multiple paths, short distance (<550m), cheaper
  Bandwidth: 10 Gbps – 100+ Tbps per fiber
  Used for: ISP backbone, data center interconnects, long-haul

Wireless (radio):
  2.4 GHz: longer range, more interference, slower (WiFi b/g/n)
  5 GHz:   shorter range, less interference, faster (WiFi a/n/ac/ax)
  6 GHz:   WiFi 6E — least interference, highest bandwidth
  Encoding: modulation schemes convert bits to radio waves (OFDM, QAM)

Signal concepts:
  Attenuation:   signal weakens with distance
  Noise:         unwanted signals corrupt data
  SNR:           Signal-to-Noise Ratio (higher = better)
  Bandwidth:     max data rate determined by Shannon's theorem:
                 C = B × log₂(1 + SNR)
                 C=capacity, B=bandwidth in Hz, SNR=signal-to-noise ratio
```

### Data Link Layer

```
Purpose: reliable transmission between two DIRECTLY connected devices.
Handles: framing, error detection, medium access control.
```

#### Ethernet (IEEE 802.3)

```
Ethernet Frame:
┌──────────┬──────────┬──────┬──────────┬──────┬──────────┐
│ Preamble │  Dst MAC │ Src  │ EtherType│ Data │   FCS    │
│  8 bytes │  6 bytes │ MAC  │ 2 bytes  │46-1500│ 4 bytes │
│          │          │6 bytes│         │ bytes│(checksum)│
└──────────┴──────────┴──────┴──────────┴──────┴──────────┘

EtherType common values:
  0x0800 = IPv4
  0x0806 = ARP
  0x86DD = IPv6
  0x8100 = VLAN tagged (802.1Q)

MTU (Maximum Transmission Unit): 1500 bytes for standard Ethernet
Jumbo frames: up to 9000 bytes (used in data centers for better throughput)
```

#### MAC Addresses

```
48-bit hardware address burned into NIC.
Format: XX:XX:XX:XX:XX:XX (hex, colon-separated)
        or XX-XX-XX-XX-XX-XX (Windows style)

First 3 bytes (OUI): Organizationally Unique Identifier = manufacturer
Last 3 bytes: device-specific

Special addresses:
  FF:FF:FF:FF:FF:FF = Broadcast (all devices on LAN receive)
  01:xx:xx:xx:xx:xx = Multicast (specific group of devices)
  Locally administered bit (bit 1 of byte 0) = set for virtual/spoofed MACs

MAC is layer 2; used only within a single network segment.
Routers don't forward based on MAC addresses.
```

#### ARP — Address Resolution Protocol

```
Problem: you know the IP address of the next hop, but need its MAC address.
ARP maps IP → MAC on a local network.

ARP Request (broadcast):
  "Who has IP 192.168.1.1? Tell 192.168.1.100"
  Sent to FF:FF:FF:FF:FF:FF — everyone on LAN receives it.

ARP Reply (unicast):
  "192.168.1.1 is at AA:BB:CC:DD:EE:FF"
  Sent directly back to requester.

ARP Cache: stores recent IP→MAC mappings (typically 20 min timeout)
  arp -n  (Linux)  or  arp -a  (Windows)

Gratuitous ARP: device announces its own IP→MAC mapping
  Used to detect IP conflicts, update caches after MAC change.

ARP Spoofing/Poisoning attack:
  Attacker sends fake ARP replies to poison caches.
  Result: traffic intended for victim routed to attacker (Man-in-the-Middle).
  Defense: Dynamic ARP Inspection (DAI) on managed switches.
```

#### WiFi (IEEE 802.11)

```
802.11 standards:
  802.11b  (1999): 2.4 GHz, 11 Mbps
  802.11g  (2003): 2.4 GHz, 54 Mbps
  802.11n  (2009): 2.4/5 GHz, 600 Mbps  (WiFi 4)
  802.11ac (2013): 5 GHz,     3.5 Gbps   (WiFi 5)
  802.11ax (2019): 2.4/5/6 GHz, 9.6 Gbps (WiFi 6/6E)
  802.11be (2024): 2.4/5/6 GHz, 46 Gbps  (WiFi 7)

CSMA/CA (Carrier Sense Multiple Access / Collision Avoidance):
  WiFi can't detect collisions while transmitting (unlike wired Ethernet).
  Before transmitting: listen to check if medium is free.
  If busy: wait random backoff time, try again.
  If free: optionally send RTS (Request to Send), get CTS (Clear to Send).
  
SSID: Network name (Service Set Identifier)
BSSID: MAC address of the access point
Channel: specific frequency band (2.4 GHz: channels 1–14, non-overlapping: 1,6,11)

Security:
  WEP:  broken (don't use)
  WPA:  TKIP — deprecated
  WPA2: AES-CCMP — current standard (use this minimum)
  WPA3: SAE (Simultaneous Authentication of Equals) — latest, better against brute force
```

#### VLANs (Virtual LANs)

```
Logically segment one physical switch into multiple isolated networks.
Benefits: security isolation, broadcast domain reduction, easier management.

802.1Q VLAN tagging: 4-byte tag inserted in Ethernet frame after src MAC.
  ┌──────────┬──────────┬───────────┬──────┬──────────┐
  │ Dst MAC  │ Src MAC  │ 802.1Q tag│ Type │   Data   │
  │          │          │TPID+VLAN ID│     │          │
  └──────────┴──────────┴───────────┴──────┴──────────┘

VLAN ID: 12-bit field → up to 4094 VLANs (0 and 4095 reserved)

Access port: carries traffic for ONE VLAN (untagged) — connects to end devices
Trunk port:  carries traffic for MULTIPLE VLANs (tagged) — connects to other switches/routers
Native VLAN: untagged traffic on a trunk port belongs to this VLAN (default: VLAN 1)
```

---

## 4. IP — The Network Layer

### IPv4

```
32-bit addresses, written as 4 octets in dotted-decimal.
  192.168.1.100
  11000000.10101000.00000001.01100100

Total address space: 2³² = 4,294,967,296 (~4.3 billion)
```

#### IPv4 Header

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─────────┬───────────┬──────────────────────────────────────────┤
│ Version │    IHL    │  DSCP/ECN  │         Total Length        │
│  (4)    │(hdr len/4)│            │                             │
├─────────┴───────────┴──────────────────────────────────────────┤
│             Identification        │Flags│  Fragment Offset     │
├─────────────────────────────────────────────────────────────────┤
│      TTL      │    Protocol       │       Header Checksum      │
├─────────────────────────────────────────────────────────────────┤
│                         Source IP Address                       │
├─────────────────────────────────────────────────────────────────┤
│                      Destination IP Address                     │
└─────────────────────────────────────────────────────────────────┘

Key fields:
  TTL (Time To Live):  decremented by each router; packet discarded at 0
                       Prevents infinite routing loops
  Protocol:            0x06=TCP, 0x11=UDP, 0x01=ICMP
  Flags:               DF (Don't Fragment), MF (More Fragments)
  Fragment Offset:     position of this fragment in original packet
  DSCP:                QoS marking (Differentiated Services)
```

#### Subnetting

```
An IP address has two parts: Network prefix + Host suffix
The subnet mask separates them.

Example: 192.168.1.100/24
  Subnet mask:    255.255.255.0  = /24 (24 ones)
  Network:        192.168.1.0
  Broadcast:      192.168.1.255  (all host bits = 1)
  Usable hosts:   192.168.1.1 – 192.168.1.254  (2²⁴⁻²⁴ - 2 = 254)

CIDR Notation & Subnet Math:
  /prefix → subnet mask → hosts per subnet

  /8   = 255.0.0.0          → 16,777,214 hosts  (Class A)
  /16  = 255.255.0.0        →     65,534 hosts  (Class B)
  /24  = 255.255.255.0      →        254 hosts  (Class C)
  /25  = 255.255.255.128    →        126 hosts
  /26  = 255.255.255.192    →         62 hosts
  /27  = 255.255.255.224    →         30 hosts
  /28  = 255.255.255.240    →         14 hosts
  /29  = 255.255.255.248    →          6 hosts
  /30  = 255.255.255.252    →          2 hosts  (point-to-point)
  /31  = 255.255.255.254    →          2 hosts  (no network/broadcast: RFC 3021)
  /32  = 255.255.255.255    →          1 host   (single host route)

Formula: hosts = 2^(32 - prefix) - 2   (subtract network and broadcast)

To find network address: IP AND subnet_mask
  192.168.1.100 & 255.255.255.0 = 192.168.1.0  ✓

Subnetting example: split 192.168.1.0/24 into 4 subnets
  Need: 4 subnets → borrow 2 bits → /26
  192.168.1.0/26    hosts: .1–.62
  192.168.1.64/26   hosts: .65–.126
  192.168.1.128/26  hosts: .129–.190
  192.168.1.192/26  hosts: .193–.254
```

#### Special IPv4 Addresses

```
Private (RFC 1918) — not routable on internet:
  10.0.0.0/8          Class A private
  172.16.0.0/12       Class B private (172.16–172.31.x.x)
  192.168.0.0/16      Class C private

Reserved:
  127.0.0.0/8         Loopback (127.0.0.1 = "localhost")
  169.254.0.0/16      Link-local / APIPA (no DHCP response)
  0.0.0.0/8           This network
  255.255.255.255     Limited broadcast (all devices on local subnet)
  224.0.0.0/4         Multicast (224.0.0.0–239.255.255.255)
  240.0.0.0/4         Reserved for future use
```

#### NAT — Network Address Translation

```
Problem: IPv4 exhaustion. Only ~4.3 billion addresses for billions of devices.
Solution: NAT allows many private IPs to share one public IP.

How NAPT (Network Address Port Translation) works:
  Private: 192.168.1.100:12345 → NAT device → Public: 1.2.3.4:54321
  
  Outbound packet: replace (192.168.1.100:12345) with (1.2.3.4:54321)
  NAT table: 1.2.3.4:54321 ↔ 192.168.1.100:12345
  Inbound response: look up 54321 → forward to 192.168.1.100:12345

Types:
  Static NAT:    one-to-one mapping (one private IP = one public IP)
  Dynamic NAT:   pool of public IPs; assigned on demand
  PAT/NAPT:      many private IPs share ONE public IP using different ports (most common)

NAT breaks end-to-end connectivity (devices behind NAT can't accept inbound connections):
  Solutions: port forwarding, UPnP, STUN/TURN (for WebRTC), IPv6
```

### IPv6

```
128-bit addresses. Fixes IPv4 exhaustion permanently.
2¹²⁸ = 340 undecillion addresses (~340 × 10³⁶)

Format: 8 groups of 4 hex digits, colon-separated
  2001:0db8:85a3:0000:0000:8a2e:0370:7334

Compression rules:
  1. Leading zeros in each group can be omitted
  2. One consecutive run of all-zero groups can be replaced by ::

  2001:db8:85a3::8a2e:370:7334  (compressed form above)
  ::1  =  0000:...:0001  (loopback, equivalent to 127.0.0.1)
  ::   =  all zeros  (unspecified)

Special addresses:
  ::1/128         Loopback
  fe80::/10       Link-local (auto-configured, not routable)
  fc00::/7        Unique local (like RFC 1918 private, not routable)
  2000::/3        Global unicast (routable internet addresses)
  ff00::/8        Multicast
  ::ffff:0:0/96   IPv4-mapped IPv6 (::ffff:192.168.1.1)

Key improvements over IPv4:
  No NAT needed (every device gets globally unique address)
  No broadcast (replaced by multicast)
  Stateless Address Autoconfiguration (SLAAC): device generates own address
  Built-in IPSec support
  Simplified header (fixed 40 bytes; no fragmentation by routers)
  Flow label field for QoS
  
Dual stack: device runs both IPv4 and IPv6 simultaneously (transition mechanism)
```

### ICMP — Internet Control Message Protocol

```
Used for: error reporting, diagnostics (not data transport)
Carried inside IP packets. Protocol number: 1 (IPv4), 58 (IPv6)

Common ICMP messages:
  Type 0:  Echo Reply          (ping response)
  Type 3:  Destination Unreachable (with codes: 0=net, 1=host, 3=port, 4=frag needed)
  Type 5:  Redirect             (router tells host to use a better route)
  Type 8:  Echo Request         (ping)
  Type 11: Time Exceeded        (TTL=0; used by traceroute)
  Type 12: Parameter Problem

ping:
  Sends ICMP Echo Request; measures RTT from Echo Reply.
  ping 8.8.8.8   → tests reachability and measures latency

traceroute / tracert:
  Sends packets with increasing TTL (1, 2, 3...).
  Each router that discards (TTL=0) sends ICMP Time Exceeded back.
  Result: path of routers from source to destination.
  Linux: traceroute uses UDP by default; traceroute -I uses ICMP
  Windows: tracert uses ICMP
```

---

## 5. TCP & UDP — The Transport Layer

Transport layer provides end-to-end communication between applications using **port numbers**.

### Port Numbers

```
16-bit value (0–65535) identifies a specific application/service.

Well-known ports (0–1023): assigned by IANA, require root/admin to bind
  20/21   FTP (data/control)
  22      SSH
  23      Telnet (insecure — don't use)
  25      SMTP (email sending)
  53      DNS
  67/68   DHCP (server/client)
  80      HTTP
  110     POP3
  143     IMAP
  443     HTTPS
  465/587 SMTP over TLS
  993/995 IMAP/POP3 over TLS
  3306    MySQL
  5432    PostgreSQL
  6379    Redis
  27017   MongoDB

Registered ports (1024–49151): assigned by IANA for specific services
Ephemeral ports (49152–65535): dynamically assigned to clients

Socket = IP address + port number
  Uniquely identifies a communication endpoint.
  Connection = (src_IP, src_port, dst_IP, dst_port, protocol)
```

### TCP — Transmission Control Protocol

```
Reliable, ordered, connection-oriented byte stream.
Guarantees: delivery, ordering, no duplicates, error detection.
```

#### TCP Header

```
Source Port (16)        Destination Port (16)
Sequence Number (32)
Acknowledgment Number (32)
Data Offset(4) | Reserved(3) | Flags(9) | Window Size(16)
Checksum (16)           Urgent Pointer (16)
Options (variable)

Key fields:
  Sequence Number:  byte offset of first byte in this segment
  Ack Number:       next byte the receiver expects (cumulative ACK)
  Flags:
    SYN:  synchronize sequence numbers (connection setup)
    ACK:  acknowledgment number is valid
    FIN:  sender finished sending (connection teardown)
    RST:  reset connection (abort)
    PSH:  push data to application immediately (don't buffer)
    URG:  urgent pointer field is valid
    ECE:  ECN echo (congestion notification)
    CWR:  congestion window reduced
  Window Size: how many bytes the receiver can accept (flow control)
```

#### TCP Three-Way Handshake

```
Client                              Server
  │                                    │
  │──── SYN (seq=x) ──────────────────►│  "I want to connect; my seq starts at x"
  │                                    │
  │◄─── SYN-ACK (seq=y, ack=x+1) ─────│  "OK, my seq starts at y; I got your x"
  │                                    │
  │──── ACK (seq=x+1, ack=y+1) ───────►│  "Got it. Connection established."
  │                                    │
  │◄═══════ data flows ════════════════►│

SYN flooding attack:
  Attacker sends many SYN packets, never completes handshake.
  Server's SYN queue fills up; legitimate connections refused.
  Defense: SYN cookies (server encodes state in ISN instead of storing it)
```

#### TCP Four-Way Teardown

```
Client                              Server
  │                                    │
  │──── FIN ──────────────────────────►│  "I'm done sending"
  │◄─── ACK ───────────────────────────│  "OK"
  │◄─── FIN ───────────────────────────│  "I'm done sending too"
  │──── ACK ──────────────────────────►│  "OK"
  │                                    │
  │ [TIME_WAIT: 2×MSL ≈ 60s]          │  (wait for stray packets)

Half-close: each direction closed independently.
TIME_WAIT: prevents old duplicate packets from being accepted by new connections.
  If you're seeing "too many connections in TIME_WAIT": tune net.ipv4.tcp_tw_reuse
```

#### TCP Flow Control

```
Receiver advertises how much buffer space it has (rwnd: receive window).
Sender cannot send more than rwnd unacknowledged bytes.

Silly Window Syndrome: sending tiny segments (e.g., 1 byte) wastes bandwidth.
Fixes:
  Nagle's Algorithm (sender): accumulate small writes into one segment.
    Sends immediately if: no unacked data OR data fills MSS.
    Good for: bulk transfers. Bad for: interactive apps (SSH, games).
    Disable with: TCP_NODELAY socket option.
  Clark's Algorithm (receiver): don't advertise tiny windows.
```

#### TCP Congestion Control

```
Problem: TCP must not overwhelm the network.
Solution: estimate network capacity and back off when congestion detected.

State: cwnd (congestion window) — sender's self-imposed limit.
Effective window = min(cwnd, rwnd)

Phases:
  Slow Start:
    cwnd starts at 1 MSS (Maximum Segment Size, typically 1460 bytes).
    On each ACK: cwnd += 1 MSS  (doubles each RTT — exponential growth).
    Until cwnd reaches ssthresh (slow start threshold).

  Congestion Avoidance:
    cwnd > ssthresh → linear growth: cwnd += MSS²/cwnd per ACK (~1 MSS/RTT).
    "Additive increase" phase.

  Congestion Detection:
    Packet loss (timeout):  ssthresh = cwnd/2; cwnd = 1; restart slow start.
    3 duplicate ACKs (fast retransmit): ssthresh = cwnd/2; cwnd = ssthresh; congestion avoidance.
    "Multiplicative decrease" → AIMD behavior.

Modern algorithms:
  Reno:    classic (above)
  CUBIC:   default Linux; uses cubic function for cwnd growth; better on high-BDP links
  BBR:     Google's algorithm; models bandwidth and RTT directly; better in wireless/lossy
  QUIC:    Google/IETF; congestion control at application level over UDP

  cwnd
   │          ╱╲       ╱╲
   │         ╱  ╲     ╱  ╲
   │        ╱    ╲   ╱    ╲
   │       ╱      ╲ ╱
   │      ╱        ╲
   │     ╱ ssthresh  \
   │────╱─────────────────
   └──────────────────────► time
   slow start → congestion avoidance → loss event → repeat
```

#### TCP State Machine

```
CLOSED → LISTEN → SYN_RCVD → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
         (server)                (both)      (active close)

Client side connect():  CLOSED → SYN_SENT → ESTABLISHED
Server side accept():   LISTEN → SYN_RCVD → ESTABLISHED

netstat -an  or  ss -an   →   shows all connections and their states
Useful states to recognize:
  ESTABLISHED:  active connection
  TIME_WAIT:    connection closed, waiting for stragglers (~60s)
  CLOSE_WAIT:   remote side closed; local hasn't yet
  LISTEN:       server waiting for connections
  SYN_SENT:     client sent SYN, waiting for SYN-ACK
```

### UDP — User Datagram Protocol

```
Unreliable, unordered, connectionless datagrams.
No handshake. No ACKs. No retransmission.
8-byte header: Source Port | Dest Port | Length | Checksum

Use UDP when:
  • Speed matters more than reliability
  • Application handles its own error recovery
  • Broadcast/multicast (TCP is unicast only)
  • Low overhead needed (IoT, embedded)

Common UDP use cases:
  DNS:        query/response; retry at application level
  DHCP:       bootstrap before IP address known
  SNMP:       network management; loss OK
  NTP:        time sync; loss OK
  QUIC:       HTTP/3 runs over UDP (implements reliability itself)
  Gaming:     position updates; stale data worse than no data
  Video/VoIP: lost frames → glitch; retransmission would arrive too late
  Streaming:  one-way; no receiver to ACK; multicast possible

UDP vs TCP summary:
  TCP: reliable, ordered, connection setup, flow/congestion control, higher overhead
  UDP: unreliable, unordered, no connection, no flow control, minimal overhead
```

### QUIC — Quick UDP Internet Connections

```
HTTP/3's transport. Built on UDP. Developed by Google, standardized as RFC 9000.

Problems with TCP that QUIC solves:
  1. Head-of-line blocking: in HTTP/2, one lost TCP packet blocks ALL streams.
     QUIC: each stream is independent; loss in one doesn't block others.
  2. Connection migration: TCP connections tied to src IP:port.
     QUIC: connection ID in header; survives IP change (e.g., WiFi → LTE).
  3. Handshake latency: TCP 1-RTT + TLS 1-RTT = 2 RTT to first data.
     QUIC: 1-RTT for new connections, 0-RTT for resumptions.
  4. Ossification: TCP middleboxes block TCP extensions.
     QUIC: fully encrypted (including headers) → middleboxes can't interfere.

QUIC provides: reliability, ordering, flow control, congestion control (per-stream),
               multiplexing, TLS 1.3, connection migration.
```

---

## 6. DNS — The Internet's Phone Book

DNS translates human-readable names (google.com) to IP addresses (142.250.80.46).

### DNS Hierarchy

```
Root (.)
   │
   ├── .com  .org  .net  .io  .gov  ...   (TLDs — Top Level Domains)
   │
   ├── google.com   amazon.com   github.com  ...   (Second Level Domains)
   │
   └── www.google.com   mail.google.com  ...   (Subdomains)

DNS is a distributed, hierarchical, eventually consistent database.

Name servers:
  Root name servers:     13 logical servers (a-m.root-servers.net)
                         Actually hundreds of physical servers via anycast.
                         Know where all TLD servers are.
  TLD name servers:      operated by registries (.com → Verisign)
                         Know where authoritative NS for each domain are.
  Authoritative NS:      holds the actual DNS records for a domain.
                         (your web host, Route53, Cloudflare DNS, etc.)
  Recursive resolver:    your ISP's or 8.8.8.8 (Google) — does the work for clients.
```

### DNS Resolution Process

```
Client wants to resolve www.example.com:

1. Check local cache (OS + browser cache)  → hit? done!
2. Ask recursive resolver (e.g., 8.8.8.8)
3. Resolver checks its cache                → hit? done!
4. Resolver asks root server: "who handles .com?"
5. Root replies: "ask a.gtld-servers.net"
6. Resolver asks TLD server: "who handles example.com?"
7. TLD replies: "ask ns1.example.com"
8. Resolver asks authoritative NS: "what is www.example.com?"
9. Authoritative NS replies: "192.0.2.1, TTL 3600"
10. Resolver caches result, returns to client
11. Client connects to 192.0.2.1

Total: ~10–100ms for uncached, ~1–5ms for cached.
```

### DNS Record Types

```
A:       Maps name to IPv4 address
         www.example.com.  IN  A  93.184.216.34

AAAA:    Maps name to IPv6 address
         www.example.com.  IN  AAAA  2606:2800:220:1:248:1893:25c8:1946

CNAME:   Canonical name (alias) — must point to another name, not IP
         blog.example.com.  IN  CNAME  example.com.
         ⚠ Cannot CNAME the zone apex (example.com itself)

MX:      Mail exchanger — where to send email for the domain
         example.com.  IN  MX  10  mail.example.com.
         (priority 10 — lower = higher priority)

NS:      Name server — which servers are authoritative for domain
         example.com.  IN  NS  ns1.example.com.

TXT:     Arbitrary text — used for verification, SPF, DKIM, DMARC
         example.com.  IN  TXT  "v=spf1 include:_spf.google.com ~all"

PTR:     Reverse DNS — IP to name (used in 1.2.3.4 → in-addr.arpa zone)
         34.216.184.93.in-addr.arpa.  IN  PTR  www.example.com.

SOA:     Start of Authority — metadata about the zone (primary NS, email, serial)

SRV:     Service location — host + port for a named service
         _https._tcp.example.com.  IN  SRV  0 5 443  www.example.com.

CAA:     Certification Authority Authorization — which CAs can issue certs
         example.com.  IN  CAA  0 issue "letsencrypt.org"
```

### DNS Security

```
DNS Spoofing / Cache Poisoning:
  Attacker injects fake DNS responses into resolver's cache.
  Client gets wrong IP → redirected to malicious server.
  
DNSSEC (DNS Security Extensions):
  Adds digital signatures to DNS records.
  Resolver can verify record authenticity.
  Chain of trust from root → TLD → domain.
  Doesn't encrypt — just authenticates.
  
DNS over HTTPS (DoH):
  DNS queries sent over HTTPS (port 443).
  Encrypted from client to resolver.
  Used by: browsers (Firefox, Chrome), Cloudflare 1.1.1.1, Google 8.8.8.8
  
DNS over TLS (DoT):
  DNS queries over TLS on dedicated port 853.
  More transparent than DoH (easy to filter at network level).

TTL (Time To Live):
  How long a record can be cached (seconds).
  Low TTL (60s): changes propagate fast, but more DNS queries.
  High TTL (86400s = 1 day): fewer queries, slow propagation.
  Before changing DNS: lower TTL to 60s → wait for old TTL to expire → change → raise TTL.
```

---

## 7. HTTP & HTTPS

### HTTP Fundamentals

```
Hypertext Transfer Protocol. Application layer. Port 80 (HTTP), 443 (HTTPS).
Stateless: each request is independent (no memory of past requests).
Text-based (HTTP/1.1), binary-framed (HTTP/2, HTTP/3).
```

#### HTTP Request Structure

```
GET /index.html HTTP/1.1\r\n
Host: www.example.com\r\n
User-Agent: Mozilla/5.0\r\n
Accept: text/html,application/xhtml+xml\r\n
Accept-Encoding: gzip, deflate, br\r\n
Connection: keep-alive\r\n
\r\n
[optional body — for POST/PUT]

Method:  what operation to perform
Path:    resource being requested (URL path + query string)
Version: HTTP/1.1, HTTP/2, HTTP/3
Headers: key-value metadata
Body:    payload (for POST, PUT, PATCH)
```

#### HTTP Methods

```
GET:     Retrieve a resource. No body. Idempotent. Cacheable.
POST:    Submit data. Has body. Not idempotent. Not cacheable.
PUT:     Replace a resource at a URL. Idempotent. Has body.
PATCH:   Partially update a resource. Has body.
DELETE:  Remove a resource. Idempotent. Optionally has body.
HEAD:    Like GET but response has no body. Used to check existence/headers.
OPTIONS: Ask what methods are supported. Used for CORS preflight.
CONNECT: Establish a tunnel (used for HTTPS through HTTP proxy).
TRACE:   Echo the request (diagnostic; usually disabled for security).

Idempotent: calling N times = calling once (same result).
Safe: doesn't modify state on the server (GET, HEAD, OPTIONS).
```

#### HTTP Status Codes

```
1xx — Informational:
  100 Continue          (server received headers, client should send body)
  101 Switching Protocols  (HTTP → WebSocket upgrade)

2xx — Success:
  200 OK                (standard success)
  201 Created           (POST created a resource; Location header has URL)
  204 No Content        (success, no body; common for DELETE)
  206 Partial Content   (range request fulfilled)

3xx — Redirection:
  301 Moved Permanently (bookmark the new URL; SEO: passes link equity)
  302 Found             (temporary redirect; don't update bookmark)
  304 Not Modified      (cached version is current; use it)
  307 Temporary Redirect (like 302 but method must not change)
  308 Permanent Redirect (like 301 but method must not change)

4xx — Client Error:
  400 Bad Request       (malformed request)
  401 Unauthorized      (authentication required — misleading name)
  403 Forbidden         (authenticated but not authorized)
  404 Not Found         (resource doesn't exist)
  405 Method Not Allowed
  409 Conflict          (resource state conflict, e.g., duplicate)
  410 Gone              (permanently deleted; like 404 but explicit)
  422 Unprocessable Entity (validation error — common in REST APIs)
  429 Too Many Requests (rate limited)

5xx — Server Error:
  500 Internal Server Error  (unhandled exception; server bug)
  502 Bad Gateway            (upstream server returned invalid response)
  503 Service Unavailable    (server overloaded or down for maintenance)
  504 Gateway Timeout        (upstream server didn't respond in time)
```

#### HTTP Headers (Important Ones)

```
Request headers:
  Host:               target server hostname (required in HTTP/1.1)
  Authorization:      credentials (Bearer token, Basic auth)
  Content-Type:       media type of request body (application/json, multipart/form-data)
  Accept:             media types client can handle
  Accept-Encoding:    compression algorithms client supports (gzip, br)
  Cookie:             send cookies to server
  Cache-Control:      caching directives (no-cache, max-age=3600)
  If-None-Match:      conditional GET using ETag (cache validation)
  If-Modified-Since:  conditional GET using date (cache validation)
  Origin:             source origin (for CORS)
  X-Forwarded-For:    original client IP (added by proxy/load balancer)

Response headers:
  Content-Type:       media type of response body
  Content-Length:     size of response body in bytes
  Content-Encoding:   compression applied (gzip, br)
  Set-Cookie:         set a cookie on client
  Cache-Control:      caching directives (no-store, public, max-age)
  ETag:               resource version identifier (for cache validation)
  Last-Modified:      when resource was last modified
  Location:           URL for redirect (3xx) or new resource (201)
  WWW-Authenticate:   auth challenge (401 responses)
  Access-Control-Allow-Origin:  CORS header
  Strict-Transport-Security:    HSTS (enforce HTTPS)
  Content-Security-Policy:      CSP (restrict resource loading)
```

### HTTP Versions

```
HTTP/1.0:
  New TCP connection for every request. No persistent connections.
  Very inefficient.

HTTP/1.1 (1997, still common):
  Persistent connections (Connection: keep-alive default).
  Pipelining: send multiple requests without waiting for each response.
    In practice disabled (head-of-line blocking: one slow response blocks all).
  Host header mandatory.

HTTP/2 (2015):
  Binary framing (not text-based).
  Multiplexing: multiple requests/responses over ONE TCP connection simultaneously.
    No head-of-line blocking at HTTP level (but still at TCP level!).
  Header compression (HPACK): dramatically reduces header overhead.
  Server push: server proactively sends resources client will need.
  Stream prioritization.
  Requires TLS in practice (all browsers enforce this).

HTTP/3 (2022):
  Uses QUIC (UDP) instead of TCP.
  No head-of-line blocking at all (QUIC streams are independent).
  Faster handshake (0-RTT or 1-RTT).
  Better on mobile (connection migration).
  Mandatory TLS 1.3.
  ~25% of web traffic as of 2024.
```

### HTTPS & TLS

```
HTTPS = HTTP + TLS (Transport Layer Security).
TLS encrypts the HTTP data in transit.

TLS 1.3 Handshake (simplified):
  Client → Server:  ClientHello (TLS version, cipher suites, key share)
  Server → Client:  ServerHello (chosen cipher, key share, certificate)
                    Server sends Certificate + CertificateVerify + Finished
  Client → Server:  Finished
  [1 RTT — data can flow]

  Session resumption (0-RTT):
    Client stores session ticket.
    On reconnect, sends ticket + early data in first message.
    Server accepts early data immediately.
    ⚠ Replay attacks possible with 0-RTT — only for idempotent requests.

TLS 1.2 vs 1.3:
  1.2: 2 RTT handshake, weaker ciphers available, no 0-RTT
  1.3: 1 RTT, only strong ciphers, 0-RTT resumption, forward secrecy mandatory

Certificates:
  X.509 certificate: public key + identity (domain) + CA signature + expiry
  CA (Certificate Authority): trusted third party that vouches for identity
    Root CAs: DigiCert, Let's Encrypt, Comodo, GlobalSign
    Let's Encrypt: free, automated (90-day certs, auto-renewal)
  
  Certificate chain:
    Root CA → Intermediate CA → End-entity cert
    Browser trusts root; verifies chain of signatures to end-entity.
  
  Certificate Transparency (CT): public log of all issued certificates
    Browsers require certs to be in CT logs.
    Enables detection of misissued certs.

TLS verification:
  Check server's certificate:
    Signature valid? (signed by trusted CA)
    Domain matches? (CN or SAN matches requested hostname)
    Not expired?
    Not revoked? (OCSP or CRL check)
    In CT log?
```

### Cookies & Sessions

```
HTTP is stateless. Cookies maintain state across requests.

Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict; Max-Age=3600

Cookie attributes:
  HttpOnly:           JavaScript can't access (protects against XSS theft)
  Secure:             only sent over HTTPS
  SameSite=Strict:    only sent for same-site requests (blocks CSRF)
  SameSite=Lax:       sent for top-level navigations and same-site (default modern browsers)
  SameSite=None:      cross-site (requires Secure)
  Max-Age / Expires:  when cookie expires (session cookie if absent)
  Domain:             which domains receive the cookie
  Path:               which paths receive the cookie

Session management:
  Server-side sessions: cookie stores session ID; actual data in server memory/DB.
    + Easy to invalidate (delete from server)
    - Doesn't scale horizontally without shared session store (Redis)
  
  JWT (JSON Web Token): self-contained token; no server storage needed.
    Header.Payload.Signature (base64url encoded, period-separated)
    Server verifies signature; no DB lookup.
    + Stateless; scales easily
    - Hard to revoke before expiry (need token blacklist or short TTL)
```

### REST & API Design

```
REST (Representational State Transfer) constraints:
  Stateless:        server holds no client state between requests
  Client-Server:    separation of concerns
  Cacheable:        responses must define cacheability
  Uniform Interface: consistent resource URLs and HTTP method semantics
  Layered System:   client doesn't know if it's talking to origin or proxy

RESTful URL design:
  Resources = nouns; methods = verbs.
  GET    /users          → list users
  POST   /users          → create user
  GET    /users/42       → get user 42
  PUT    /users/42       → replace user 42
  PATCH  /users/42       → update fields of user 42
  DELETE /users/42       → delete user 42
  GET    /users/42/posts → list user 42's posts

CORS (Cross-Origin Resource Sharing):
  Browser blocks JS from reading responses to cross-origin requests by default.
  Server opts in by sending:
    Access-Control-Allow-Origin: https://app.example.com
    Access-Control-Allow-Methods: GET, POST, PUT
    Access-Control-Allow-Headers: Content-Type, Authorization
  
  Preflight: browser sends OPTIONS request first for non-simple requests.
  Simple request: GET/POST with safe headers (no preflight needed).
```

---

## 8. Network Security

### Common Network Attacks

```
Man-in-the-Middle (MitM):
  Attacker intercepts traffic between two parties.
  Methods: ARP spoofing, rogue WiFi AP, DNS poisoning, BGP hijacking.
  Defense: TLS/HTTPS, certificate pinning, HSTS, DNSSEC.

DDoS (Distributed Denial of Service):
  Flood target with traffic from many sources to exhaust resources.
  Types:
    Volumetric: fill bandwidth (UDP flood, ICMP flood, amplification attacks)
    Protocol: exploit protocol weaknesses (SYN flood, Ping of Death)
    Application layer (L7): HTTP floods, slow attacks (Slowloris)
  Amplification: attacker spoofs victim's IP, sends small queries to servers
                 that reply with large responses (DNS: 50×, NTP: 500×, memcached: 50000×)
  Defense: CDN/scrubbing centers, rate limiting, anycast, ingress filtering (BCP38).

Packet Sniffing:
  Capture traffic on shared medium (WiFi, hub).
  Tool: Wireshark, tcpdump.
  Defense: encryption (TLS), switched networks.

Port Scanning:
  Probe which ports are open on a target to discover services.
  Tools: nmap.
  nmap -sS target  (SYN scan — half-open, fast, less detectable)
  nmap -sV target  (service version detection)
  Defense: firewall, intrusion detection, minimize exposed services.

SQL Injection, XSS (cross-site scripting), CSRF:
  Application-layer attacks, not pure network, but cross boundaries.
  Defense: input validation, CSP, SameSite cookies, CSRF tokens.
```

### Firewalls

```
Packet filter (stateless):
  Allow/deny based on: src/dst IP, port, protocol.
  Fast. Doesn't track connection state.
  Example rule: ALLOW TCP 0.0.0.0/0 → 10.0.1.5:443

Stateful firewall:
  Tracks connection state (SYN/SYN-ACK/ACK).
  Allows return traffic for established connections automatically.
  Blocks unsolicited inbound traffic even if port is "open".

Application firewall (Layer 7 / WAF):
  Inspects HTTP payload.
  Can block SQL injection, XSS, specific URL patterns.
  Examples: AWS WAF, Cloudflare, ModSecurity.

iptables (Linux firewall):
  Tables: filter (default), nat, mangle, raw
  Chains: INPUT, OUTPUT, FORWARD (in filter table)
  
  # Allow established connections
  iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  # Allow SSH
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT
  # Drop everything else
  iptables -A INPUT -j DROP
  
  nftables: modern replacement for iptables (Linux 3.13+)
```

### VPN — Virtual Private Network

```
Creates an encrypted tunnel between client and VPN server.
Traffic appears to come from VPN server's IP.

Use cases:
  Remote access: employee connects to corporate network over internet.
  Site-to-site: connect two office networks securely.
  Privacy: hide traffic from ISP; appear to be in different location.

Protocols:
  IPSec:    network-layer; standard; used in enterprise; complex setup
  OpenVPN:  SSL/TLS-based; flexible; crosses most firewalls; slower
  WireGuard: modern; simple; extremely fast; small codebase (~4000 lines)
             Uses Curve25519, ChaCha20, BLAKE2s, SipHash24
  L2TP/IPSec: older; widely supported; double encapsulation = overhead
  SSTP:     Microsoft; uses HTTPS port 443 (crosses most firewalls)

How WireGuard works:
  Peers pre-share public keys.
  UDP-based; no handshake overhead after initial key exchange.
  Connection-less: no state; just send encrypted packets.
  ~Gbps throughput with minimal CPU.
```

### TLS/SSL Details

```
Cipher suite: combination of algorithms used in TLS session.
  TLS_AES_256_GCM_SHA384    (TLS 1.3)
  └─ Key exchange: ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)
  └─ Bulk cipher: AES-256-GCM (authenticated encryption)
  └─ Hash/MAC: SHA-384

Forward secrecy (PFS):
  Each session uses a temporary key pair (ephemeral).
  Compromise of server's long-term private key doesn't expose past sessions.
  Achieved by: ECDHE or DHE key exchange.
  TLS 1.3 mandates forward secrecy.

Certificate pinning:
  App hardcodes expected certificate/public key.
  Prevents MitM even with rogue trusted CA.
  ⚠ Hard to update when certificate changes.

HSTS (HTTP Strict Transport Security):
  Server tells browser: always use HTTPS for this domain.
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  Browser refuses plain HTTP connections even if user types http://.
```

---

## 9. Routing & Switching

### Switching (Layer 2)

```
Switch: learns which MAC address is on which port.
MAC address table (CAM table): {MAC → port} mappings.

Learning process:
  1. Frame arrives on port X with source MAC = AA:BB:CC:DD:EE:FF
  2. Switch records: AA:BB:CC:DD:EE:FF → port X
  3. Destination MAC in table? Forward to specific port. Not in table? Flood all ports.
  4. Unknown, broadcast, or multicast → flood all ports (except source port).

Spanning Tree Protocol (STP, 802.1D):
  Problem: redundant links between switches → broadcast storms (loops).
  STP elects a root bridge and blocks redundant ports to prevent loops.
  Convergence: slow (~50 seconds to detect failure + reconverge).
  RSTP (802.1w): Rapid STP — converges in ~1–2 seconds.
  MSTP (802.1s): Multiple STP — separate spanning trees per VLAN group.
```

### Routing (Layer 3)

```
Router: forwards packets between different networks based on IP address.
Routing table: {destination network → next hop IP + outgoing interface}

Longest prefix match:
  Router matches destination IP to most specific (longest) prefix.
  10.0.0.0/8    → interface A
  10.1.0.0/16   → interface B   ← more specific
  10.1.2.0/24   → interface C   ← most specific
  0.0.0.0/0     → default GW    ← catch-all (default route)
  Packet to 10.1.2.5 → matches /24 first  ✓
```

#### Static vs Dynamic Routing

```
Static routing:
  Admin manually configures routes.
  + Simple, predictable, no overhead.
  - Doesn't adapt to failures; doesn't scale.
  Use: small networks, stub networks, default routes.

Dynamic routing:
  Routers automatically discover and share routes.
  Adapts to failures automatically.
  
  Distance Vector (RIP, EIGRP):
    Routers share their routing tables with neighbors.
    Metric: hop count (RIP) or composite (EIGRP).
    Slow convergence; can cause routing loops.
    RIP: max 15 hops (>15 = unreachable).
  
  Link State (OSPF, IS-IS):
    Each router advertises state of its directly connected links.
    Every router builds identical topology map (Link State Database).
    Runs Dijkstra's algorithm to find shortest path.
    Fast convergence. Scales to large networks.
    
  Path Vector (BGP):
    Used between autonomous systems (the internet).
    Shares full path (sequence of AS numbers), not just distances.
    Policy-based: choose routes based on business relationships.
```

#### OSPF — Open Shortest Path First

```
Interior Gateway Protocol (IGP) — within one organization's network.
Link State protocol. Uses Dijkstra's algorithm. Metric: cost (based on bandwidth).

Areas: OSPF networks divided into areas.
  Area 0 (backbone): all other areas must connect to it.
  ABR (Area Border Router): connects area to backbone.
  
Packet types:
  Hello:     discover/maintain neighbor relationships
  DBD:       Database Description — summary of LSDB
  LSR:       Link State Request — ask for specific LSAs
  LSU:       Link State Update — contains LSAs
  LSAck:     acknowledge LSUs

DR/BDR election (on broadcast networks):
  Designated Router and Backup DR reduce LSA flooding.
  All routers form adjacencies with DR/BDR only (not with each other).
```

#### BGP — Border Gateway Protocol

```
The routing protocol of the internet.
Used between Autonomous Systems (AS) — independently managed networks.
  Each AS has an ASN (Autonomous System Number): 1–64511 (public), 64512–65535 (private)

eBGP: between different ASes (external BGP)
iBGP: within the same AS (internal BGP)

BGP attributes (selection criteria, in order):
  1. Weight (Cisco proprietary; highest wins; local only)
  2. Local Preference (highest wins; influences outbound traffic within AS)
  3. Locally originated (prefer routes originated by this router)
  4. AS Path length (shorter wins; loop prevention)
  5. Origin (IGP < EGP < incomplete)
  6. MED (Multi-Exit Discriminator; lowest wins; hint to neighbors for inbound)
  7. eBGP over iBGP
  8. IGP metric to next hop
  9. Oldest route (for stability)

BGP hijacking:
  AS announces routes it doesn't own → traffic redirected.
  Famous incidents: Pakistan Telecom (YouTube, 2008), AWS (MyEtherWallet, 2018).
  Defense: RPKI (Resource Public Key Infrastructure) — cryptographic route origin validation.
```

### Network Address Translation (revisited in context)

```
SNAT (Source NAT): translate source IP (used for outbound internet traffic)
DNAT (Destination NAT): translate destination IP (used for inbound, port forwarding)

Load balancer NAT:
  DST: client → VIP:80 → DNAT → real server:80
  SRC: real server → client (direct return possible with DSR)
```

---

## 10. Network Infrastructure & Protocols

### DHCP — Dynamic Host Configuration Protocol

```
Automatically assigns IP addresses to devices on a network.
UDP: client port 68, server port 67.

DORA process:
  Discover:  client broadcasts "I need an IP" (src: 0.0.0.0, dst: 255.255.255.255)
  Offer:     server offers "use this IP: 192.168.1.50"
  Request:   client requests "I'll take 192.168.1.50"
  Acknowledge: server confirms "it's yours for 24 hours"

DHCP lease: IP is assigned for a duration (lease time).
  Client renews at 50% of lease time.
  If renewal fails: retry at 87.5%.
  If lease expires: stop using IP, restart DORA.

DHCP options:
  Router (option 3):     default gateway
  DNS servers (option 6): DNS resolvers to use
  Domain name (option 15): search domain
  NTP servers (option 42)
  TFTP server (option 66): for network boot (PXE)

DHCP relay agent:
  DHCP uses broadcasts; routers don't forward broadcasts.
  Relay agent (on router) unicasts DHCP packets to DHCP server.
  Allows one DHCP server to serve multiple subnets.
```

### NAT64 & IPv6 Transition

```
Mechanisms to coexist as IPv4 → IPv6 transition:

Dual Stack: device has both IPv4 and IPv6 addresses.
            Connects via whichever is available (prefer IPv6 — RFC 6724).

Tunneling: carry IPv6 inside IPv4 packets (6in4, 6to4, Teredo).
           Used for IPv6 connectivity over IPv4-only networks.

NAT64:     translate IPv6 ↔ IPv4 at gateway.
           IPv6-only devices can reach IPv4-only hosts.
           Prefix: 64:ff9b::/96 + IPv4 address.
```

### MPLS — Multiprotocol Label Switching

```
High-performance packet forwarding using short fixed-length labels instead of IPs.
Used by ISPs and enterprise WANs for traffic engineering.

Label: 20-bit value inserted between Layer 2 and Layer 3 headers.
LSR (Label Switch Router): forwards based on label, not IP lookup.
LER (Label Edge Router): adds/removes labels at network edge.

Benefits:
  Traffic engineering: route traffic around congestion.
  VPNs: MPLS L3VPN / L2VPN (separate routing tables per customer).
  QoS: CoS bits in label stack.
  Faster than IP routing (label lookup vs longest-prefix match).
```

### Load Balancing

```
Distributes traffic across multiple backend servers.

L4 Load Balancer (transport layer):
  Routes based on IP + port. Fast. No HTTP awareness.
  Protocols: TCP, UDP.
  Examples: AWS NLB, HAProxy (TCP mode), Linux LVS.

L7 Load Balancer (application layer):
  Reads HTTP headers, cookies, URL. Can route intelligently.
  SSL termination, HTTP/2 support, health checks.
  Examples: AWS ALB, nginx, HAProxy (HTTP mode), Envoy.

Algorithms:
  Round Robin:          distribute sequentially
  Weighted RR:          distribute proportionally to weight
  Least Connections:    send to server with fewest active connections
  Least Response Time:  send to fastest-responding server
  IP Hash:              same client IP → same server (session affinity)
  Random:               randomly pick a backend

Health checks:
  LB periodically probes backends (TCP connect, HTTP GET /health).
  Failed backends removed from rotation automatically.

Session persistence (sticky sessions):
  Some apps require same user → same server (stateful sessions).
  Methods: cookie-based, IP-hash, URL rewriting.
  Better: stateless apps / centralized session store (Redis).
```

### CDN — Content Delivery Network

```
Network of edge servers distributed globally.
Serves content from the server closest to the user.

Benefits:
  Reduced latency: edge server in same city instead of origin in another country.
  Reduced origin load: static assets cached at edge.
  DDoS protection: absorb traffic across global network.
  Global availability: redundancy across data centers.

How it works:
  1. DNS: CDN nameservers return IP of nearest edge node (GeoDNS or anycast).
  2. Client connects to edge server.
  3. Cache hit: edge serves directly.
  4. Cache miss: edge fetches from origin, caches, serves.

Cache-Control headers for CDN:
  Cache-Control: public, max-age=86400, s-maxage=604800
    s-maxage: CDN-specific TTL (overrides max-age for shared caches)
  Cache-Control: no-store     → never cache
  Surrogate-Control: max-age=3600  (some CDNs use this)
  Vary: Accept-Encoding        → cache separate versions per encoding

Cache invalidation:
  URL versioning: /app.js?v=abc123 (change hash when content changes)
  Cache purge API: CDN API call to invalidate specific URLs.

Examples: Cloudflare, AWS CloudFront, Fastly, Akamai, Vercel Edge Network.
```

### NTP — Network Time Protocol

```
Synchronizes clocks across the network.
Port 53 UDP. Hierarchical: stratum 0 (atomic clocks) → stratum 1 (GPS) → stratum 2+ (servers).

Why time matters:
  TLS certificate expiry checks, Kerberos authentication (5-minute window),
  Log correlation, distributed system ordering (causality).

Modern: Chrony or systemd-timesyncd on Linux.
PTP (IEEE 1588): sub-microsecond precision for financial/industrial systems.
```

---

## 11. Performance & Troubleshooting

### Network Performance Metrics

```
Bandwidth:   max capacity of a link (Gbps)
Throughput:  actual data rate achieved (always ≤ bandwidth)
Latency:     one-way delay from sender to receiver
RTT:         Round-Trip Time (includes both directions + processing)
Jitter:      variation in delay
Packet loss: % of packets that don't arrive
BDP:         Bandwidth-Delay Product = bandwidth × RTT
             = amount of data "in flight" at any time
             TCP must have cwnd ≥ BDP to fully utilize a link

Example: 1 Gbps link, 100ms RTT:
  BDP = 1×10⁹ bits/s × 0.1s = 100 Mbit = 12.5 MB
  TCP window must be ≥ 12.5 MB to fill the pipe
  Default TCP window: 64 KB — would only use 0.5% of capacity!
  Solution: Window Scaling (RFC 7323) — up to 1 GB window
```

### Essential Network Tools

```
ping:
  Tests connectivity and measures RTT.
  ping -c 5 8.8.8.8          → 5 packets to Google DNS
  ping -s 1400 192.168.1.1   → test with specific packet size (MTU test)

traceroute / tracert:
  Shows path to destination (each hop).
  traceroute -n 8.8.8.8      → -n skips DNS lookups (faster)
  mtr 8.8.8.8                → combines ping + traceroute, real-time

nslookup / dig:
  DNS lookup tools.
  dig google.com              → A record
  dig google.com AAAA         → IPv6 address
  dig google.com MX           → mail servers
  dig @8.8.8.8 google.com    → use specific resolver
  dig +trace google.com       → show full resolution chain
  dig -x 8.8.8.8             → reverse DNS lookup

nmap:
  Network scanner.
  nmap 192.168.1.0/24        → scan entire subnet (host discovery)
  nmap -p 80,443 target      → scan specific ports
  nmap -sV -O target         → service + OS detection
  nmap -A target             → aggressive (OS, version, scripts)

netstat / ss:
  Show network connections and listening ports.
  ss -tlnp                   → TCP listening sockets + PIDs
  ss -an                     → all connections
  netstat -rn                → routing table

ip / ifconfig:
  Network interface configuration.
  ip addr show               → list interfaces and addresses
  ip route show              → routing table
  ip neigh show              → ARP cache

tcpdump:
  Capture and analyze packets (command line Wireshark).
  tcpdump -i eth0             → capture on eth0
  tcpdump -i any port 80     → all interfaces, HTTP traffic
  tcpdump host 192.168.1.1   → traffic to/from specific host
  tcpdump -w cap.pcap        → write to file (open in Wireshark)
  tcpdump -n -X              → no DNS resolution, hex+ASCII output

curl:
  HTTP client.
  curl -v https://example.com        → verbose (headers + body)
  curl -I https://example.com        → HEAD request (headers only)
  curl -o /dev/null -w "%{time_total}\n" https://example.com  → timing
  curl -H "Authorization: Bearer token" https://api.example.com
  curl --resolve example.com:443:93.184.216.34 https://example.com  → bypass DNS

openssl:
  TLS diagnostics.
  openssl s_client -connect example.com:443   → test TLS connection
  openssl s_client -connect example.com:443 -showcerts  → view full chain
  echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

### MTU & Path MTU Discovery

```
MTU (Maximum Transmission Unit): largest packet a link can carry.
  Ethernet: 1500 bytes
  Jumbo frames: 9000 bytes (within data center)
  PPPoE: 1492 bytes (8 bytes for PPPoE overhead)
  VPN overhead: usually 1400–1450 bytes effective MTU

If a packet exceeds link MTU:
  Fragmentation (IPv4): router fragments packet; reassembled at destination.
    Expensive; DF bit prevents it.
  IPv6: routers don't fragment! Source must use correct size.

Path MTU Discovery:
  Send large packet with DF (Don't Fragment) bit set.
  If too big: router sends ICMP "Fragmentation Needed" with its MTU.
  Source reduces packet size accordingly.
  
PMTU Black hole: firewall blocks ICMP → DF packets dropped silently.
  Symptom: connection hangs after TCP handshake (large data packets lost).
  Fix: MSS clamping on firewall (adjust TCP MSS option to fit path MTU).
```

### Common Networking Problems & Diagnosis

```
Problem: Can't reach a host
  1. ping localhost              → is TCP/IP stack working?
  2. ping default gateway         → is local network working?
  3. ping 8.8.8.8                → is internet routing working?
  4. ping google.com             → is DNS working?
  5. traceroute google.com       → where does it fail?

Problem: DNS not resolving
  dig @127.0.0.1 example.com    → is local DNS working?
  dig @8.8.8.8 example.com      → does external DNS work?
  cat /etc/resolv.conf           → what DNS servers configured?
  systemd-resolve --status       → systemd DNS status

Problem: High latency
  mtr target                    → find which hop has high latency/loss
  If loss at one hop but not subsequent: ICMP rate-limiting (normal)
  If loss at one hop and all subsequent: actual packet loss

Problem: Port not accessible
  nc -zv 192.168.1.1 80          → test if port is open
  nmap -p 80 192.168.1.1         → port scan
  ss -tlnp | grep :80            → is something listening locally?
  iptables -L -n -v              → is firewall blocking?

Problem: Slow file transfer
  iperf3 -s (server)             → bandwidth test server
  iperf3 -c server               → test throughput
  Check: duplex mismatch (auto-negotiate failure), cable quality, MTU
```

---

## 12. Quick Reference

### Protocol & Port Number Reference

| Protocol | Port | Transport | Purpose |
|----------|------|-----------|---------|
| FTP | 20/21 | TCP | File transfer (data/control) |
| SSH | 22 | TCP | Secure shell |
| SMTP | 25 | TCP | Email sending |
| DNS | 53 | UDP/TCP | Name resolution |
| DHCP | 67/68 | UDP | IP assignment |
| HTTP | 80 | TCP | Web |
| POP3 | 110 | TCP | Email retrieval |
| NTP | 123 | UDP | Time sync |
| IMAP | 143 | TCP | Email retrieval |
| SNMP | 161/162 | UDP | Network management |
| HTTPS | 443 | TCP | Secure web |
| SMB | 445 | TCP | File sharing (Windows) |
| SMTP/TLS | 587 | TCP | Email submission |
| IMAPS | 993 | TCP | Secure email |
| MySQL | 3306 | TCP | Database |
| RDP | 3389 | TCP | Remote desktop |
| PostgreSQL | 5432 | TCP | Database |
| Redis | 6379 | TCP | Cache/message broker |
| HTTP alt | 8080 | TCP | Dev/proxy |
| HTTPS alt | 8443 | TCP | Dev/proxy |
| MongoDB | 27017 | TCP | Database |

### OSI Layer Quick Reference

| Layer | Name | Device | Protocol | PDU |
|-------|------|--------|----------|-----|
| 7 | Application | — | HTTP, DNS, SMTP | Data |
| 6 | Presentation | — | TLS, JPEG | Data |
| 5 | Session | — | RPC | Data |
| 4 | Transport | — | TCP, UDP | Segment |
| 3 | Network | Router | IP, ICMP, BGP | Packet |
| 2 | Data Link | Switch, Bridge | Ethernet, ARP | Frame |
| 1 | Physical | Hub, NIC, Cable | — | Bits |

### Subnet Quick Reference

| CIDR | Subnet Mask | Hosts | Usable |
|------|------------|-------|--------|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /16 | 255.255.0.0 | 65536 | 65534 |
| /8 | 255.0.0.0 | 16M | ~16M |

### TCP vs UDP Summary

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Yes (3-way handshake) | No |
| Reliability | Guaranteed delivery | Best-effort |
| Ordering | Maintained | Not maintained |
| Error correction | Yes (retransmission) | No |
| Flow control | Yes (window) | No |
| Congestion control | Yes (AIMD) | No |
| Header size | 20–60 bytes | 8 bytes |
| Speed | Slower | Faster |
| Use case | HTTP, email, file transfer | DNS, VoIP, video, games |

### HTTP Status Code Summary

| Range | Category | Key Codes |
|-------|----------|-----------|
| 1xx | Informational | 101 Switching Protocols |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Permanent, 302 Temporary, 304 Not Modified |
| 4xx | Client Error | 400, 401, 403, 404, 422, 429 |
| 5xx | Server Error | 500, 502, 503, 504 |

### IPv4 Private Address Ranges

| Range | CIDR | Addresses |
|-------|------|-----------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16.7 million |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1.04 million |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 |

### Latency Reference (Approximate)

| Operation | Latency |
|-----------|---------|
| Same datacenter (LAN) | ~0.5 ms |
| Cross-country US | ~40 ms |
| US ↔ Europe | ~80 ms |
| US ↔ Asia | ~150 ms |
| Earth circumference (speed of light) | ~133 ms min |
| DNS resolution (cached) | ~1–5 ms |
| DNS resolution (uncached) | ~20–100 ms |
| TCP handshake | 1 RTT |
| TLS 1.3 handshake | 1 RTT (0-RTT for resumption) |
| TLS 1.2 handshake | 2 RTT |

---

*For deeper study: "Computer Networks" (Tanenbaum & Wetherall) for theory. "TCP/IP Illustrated Vol 1" (Stevens) for protocol deep-dives. RFC documents for authoritative protocol specifications. Julia Evans' networking zines for accessible practical knowledge.*