# Part 1: Internet Fundamentals

## Critical Foundational Concept

**Why this matters for security:** You cannot secure what you don't understand. Network attacks happen at all seven OSI layers. To defend against them, you must know:
- How data actually moves from one computer to another
- Where vulnerabilities exist at each layer
- Which security measures protect which layers
- How protocols interact to enable communication

This chapter doesn't just teach "what is the OSI model?" but **why it exists** and **how each layer works at the most fundamental level**.

---

## The OSI Model (7 Layers): From Fundamentals to Implementation

### What IS a "Layer"? (The Foundational Concept)

Before we discuss 7 layers, you must understand what we mean by "layer":

**Layer = A conceptual boundary where data gets transformed or addressed differently**

Think of sending a letter:
```
Layer Analogy:
┌─────────────────────────────────────────┐
│ Application: You write a letter (content)│
├─────────────────────────────────────────┤
│ Envelope processing: Put letter in      │
│ envelope, address it                    │
├─────────────────────────────────────────┤
│ Transport: Postal service determines    │
│ best route (fastest vs cheapest)        │
├─────────────────────────────────────────┤
│ Network: Sorting facility sorts by      │
│ postal codes                            │
├─────────────────────────────────────────┤
│ Link: Delivery truck carries packages   │
├─────────────────────────────────────────┤
│ Physical: Roads, vehicles, fuel         │
└─────────────────────────────────────────┘
```

Each layer:
- Only cares about ITS job
- Doesn't need to know details of other layers
- Adds/reads its own "headers" (addressing info)
- Hands off to the next layer when done

### The Network Encapsulation Process (How Data Actually Gets Sent)

When you type "google.com" in your browser, here's **exactly** what happens:

```
LAYER 7 (Application - HTTP Protocol):
  Your browser creates HTTP request:
  ┌─────────────────────────────────┐
  │ GET / HTTP/1.1                  │
  │ Host: google.com                │
  │ User-Agent: Chrome              │
  │ [Your web request data] ────► [YOUR MESSAGE]
  └─────────────────────────────────┘

LAYER 6 (Presentation):
  Browser handles:
  - "Is google.com using HTTPS?" → Encrypt it
  - "What character encoding?" → UTF-8
  [Message now ENCRYPTED if HTTPS]

LAYER 5 (Session):
  Browser says "Remember this connection"
  - Keeps track that this is the same user
  - Manages multiple requests on one connection

LAYER 4 (Transport - TCP Protocol adds):
  ┌───────────────────────────────┐
  │ Source Port: 52891 (random)   │
  │ Dest Port: 443 (HTTPS)        │
  │ Sequence: 1001                 │
  │ [Checksum for error detection]│
  │ ┌──────────────────────────┐  │
  │ │ [Layer 7-5 data here]    │  │
  │ └──────────────────────────┘  │
  └───────────────────────────────┘
  This is now called a "SEGMENT"

LAYER 3 (Network - IP Protocol adds):
  ┌────────────────────────────────────┐
  │ Source IP: 192.168.1.100 (your PC) │
  │ Dest IP: 142.250.185.46 (Google)  │
  │ TTL: 64 (max hops: 64 routers)     │
  │ Protocol: 6 (TCP)                   │
  │ ┌──────────────────────────────┐   │
  │ │ [Layer 4-7 data here]        │   │
  │ └──────────────────────────────┘   │
  └────────────────────────────────────┘
  This is now called a "PACKET"

LAYER 2 (Data Link - Ethernet adds):
  ┌──────────────────────────────────────┐
  │ Source MAC: AA:BB:CC:DD:EE:FF       │
  │ Dest MAC: 11:22:33:44:55:66 (router)│
  │ Frame Type: IPv4                     │
  │ ┌────────────────────────────────┐   │
  │ │ [Layer 3-7 data here]          │   │
  │ └────────────────────────────────┘   │
  │ CRC Checksum                         │
  └──────────────────────────────────────┘
  This is now called a "FRAME"

LAYER 1 (Physical):
  Network card converts to electrical signals:
  011010101100 (your frame as 1s and 0s)
  └─ Sent over Ethernet cable as voltage pulses
     (or WiFi as radio waves)
```

**Key insight:** Each layer wraps the previous layer's data, adding its own header. This is called **encapsulation**. It's like preparing a gift:
1. Put your gift in a box (your message)
2. Wrap the box as a present
3. Put present in a shipping box
4. Label the shipping box with address

---

## The OSI Model (7 Layers): Complete Reference

The OSI (Open Systems Interconnection) model describes how data moves through networks:

```
Layer 7: Application Layer
├─ HTTP, HTTPS, DNS, SSH, Telnet
├─ User-facing applications
└─ Data from user programs

Layer 6: Presentation Layer
├─ Data encryption/decryption
├─ Compression
└─ Format translation

Layer 5: Session Layer
├─ Session management
├─ Dialog control
└─ Synchronization

Layer 4: Transport Layer
├─ TCP (reliable, ordered, connection-oriented)
├─ UDP (fast, unreliable, connectionless)
└─ Port numbers (combination with IP = socket)

Layer 3: Network Layer
├─ IP addresses (routing)
├─ Routers
├─ IPv4, IPv6
└─ ICMP (ping)

Layer 2: Data Link Layer
├─ MAC addresses
├─ Switches
├─ Ethernet, WiFi
└─ Frames

Layer 1: Physical Layer
├─ Cables, fiber optics
├─ Wireless signals
├─ Bits (0s and 1s)
└─ Voltage levels
```

### Memory Aid: "All People Seem To Need Data Processing"

## TCP/IP Protocol Suite

The modern internet uses TCP/IP, which combines multiple protocols:

### IP (Internet Protocol): The Addressing System

**Conceptual Foundation:** IP is how the internet finds YOUR computer among billions

At its core, IP addresses answer one question: **"Where should this data go?"**

```
┌─ Analogy: Mailing System ─┐
│ IP Address = Street Address
│ 192.168.1.100
│ ├─ 192.168 = Your city district (network)
│ └─ 1.100 = Your house number (host)
│
│ Subnet Mask = Defines what's "local"
│ 255.255.255.0
│ ├─ If mask = 255.255.255.0
│ ├─ Then 192.168.1.X is all local
│ └─ Anything else: send to router
└────────────────────────────────┘

How routing works:
Your computer wants to send data to 8.8.8.8
├─ Checks: Is 8.8.8.8 local? (local network?)
├─ Answer: NO (different from 192.168.1.X)
├─ So: Give to router (gateway: 192.168.1.1)
├─ Router checks: Is 8.8.8.8 mine?
├─ NO, pass to next router
├─ [Packet hops through internet]
├─ [Each router: "Where does 8.8.8.8 live?"]
├─ [Eventually reaches Google's network]
└─ Google's router: "That's me!" → delivers to 8.8.8.8
```

**IPv4 vs IPv6 (The Evolution):**
```
IPv4: 192.168.1.100 (32 bits = ~4 billion addresses)
├─ Problem: Internet is running OUT of IPv4 addresses!
├─ That's why IPv6 was created
└─ But IPv4 still dominates (security policy reason)

IPv6: 2001:0db8:85a3::8a2e:0370:7334 (128 bits)
├─ ~340 undecillion addresses (never run out)
├─ Better security built-in (IPSec support)
├─ But: Most organizations still prioritize IPv4
└─ You'll see: IPv4 in practice, IPv6 for future
```

### TCP (Transmission Control Protocol): The Reliability Protocol

**Conceptual Foundation:** TCP guarantees your data arrives correctly and in order

Most important concept: **TCP cares about RELIABILITY at the cost of SPEED**

```
Why reliability matters:
You email "BUY 100 stocks" (must arrive)
  ├─ Can't afford to lose that message
  ├─ Must arrive exactly as sent
  ├─ Must arrive in right order
  └─ → Use TCP

Streaming video (speed matters more):
  ├─ Okay to lose frame 500
  ├─ Okay if frame 600 arrives before 599
  ├─ Can't wait for retransmissions
  └─ → Use UDP
```

**The TCP Three-Way Handshake: Breaking it Down**

Before ANY data is sent, TCP must establish a connection. Here's exactly what happens:

```
STEP 1: Client sends SYN (synchronization request)
┌─────────────────────────────────────────┐
│ "Hello server, I want to connect"       │
│                                         │
│ Client sends: SYN, seq=1000             │
│ ├─ SYN = "This is a connection request" │
│ ├─ seq=1000 = "Start counting from 1000"│
│ └─ Client says "Let's coordinate our   │
│    numbering starting from 1000"        │
│                                         │
│ Why? TCP numbers EVERY byte sent!       │
│ ├─ If you send "Hello World" (11 bytes) │
│ ├─ Bytes are numbered 1000-1010         │
│ ├─ If byte 1005 is lost, we know it     │
│ └─ We ask client to resend byte 1005    │
└─────────────────────────────────────────┘

STEP 2: Server responds with SYN-ACK
┌──────────────────────────────────────────┐
│ "I heard you, I'm ready too"             │
│                                          │
│ Server sends: SYN-ACK                    │
│ ├─ seq=2000 = "I'm starting from 2000"  │
│ ├─ ack=1001 = "I got your seq 1000"     │
│ │            "Next byte I expect: 1001" │
│ └─ Both sides now have numbering system  │
└──────────────────────────────────────────┘

STEP 3: Client acknowledges server
┌──────────────────────────────────────────┐
│ "Got it, we're ready!"                   │
│                                          │
│ Client sends: ACK                        │
│ ├─ seq=1001 = "Next byte FROM me: 1001" │
│ ├─ ack=2001 = "I got seq 2000"          │
│ │            "Next byte I expect: 2001" │
│ └─ Connection established!               │
│    (All data from now uses these seq #s) │
└──────────────────────────────────────────┘

┌─ Timeline ─────────────────────────────────────┐
│ t=0:  Client ──SYN(seq=1000)──→ Server        │
│ t=1:  Server ←──SYN-ACK(seq=2000,ack=1001)──← │
│ t=2:  Client ──ACK(seq=1001,ack=2001)──→ Server│
│ t=3:  [Connection ready for data]             │
│ t=4:  Client ──HTTP GET REQUEST(seq=1001)──→  │
└────────────────────────────────────────────────┘
```

**Why this is secure (and vulnerable):**
✓ Sequence numbers prevent packet forgery
✓ Both sides confirmed existence
✗ BUT: Numbers are predictable (security issue)
✗ BUT: No authentication yet (TLS adds that)

**Data Transfer with Sequence Numbers:**
```
Client wants to send "HELLO" (5 bytes)
├─ Bytes numbered: 1001, 1002, 1003, 1004, 1005
├─ Sends segment: "HELLO" with seq=1001, ack=2001
├─ Server receives, checks:
│  ├─ Is seq=1001? YES (expected)
│  ├─ Data valid? YES
│  └─ Send back: ACK seq=1001, ack=1006
│     (means: got 1001-1005, expect next byte 1006)
│
└─ If packet lost:
   ├─ Client sends "WORLD" with seq=1006
   ├─ Server receives seq=1006 but expected 1006!
   ├─ Means no packets lost
   └─ Process continues
   
   BUT if packet 1001-1005 is lost:
   ├─ Client sends "WORLD" with seq=1006
   ├─ Server says: "I expected 1001!" 
   ├─ Client resends "HELLO" with seq=1001
   └─ Continue
```

**TCP State Diagram (Connection Lifecycle):**
```
CLOSED
  │
  ├─ [Client initiates]
  │  └─> SYN_SENT (waiting for response)
  │        │
  │        └─ [Gets SYN-ACK]
  │           └─> ESTABLISHED
  │                │
  │                ├─ [Data transfer here]
  │                │
  │                └─ [Client done, sends FIN]
  │                   └─> FIN_WAIT_1
  │                        │
  │                        └─ [Gets FIN-ACK]
  │                           └─> FIN_WAIT_2
  │                                │
  │                                └─ [Gets FIN from other side]
  │                                   └─> TIME_WAIT (2 min wait)
  │                                        │
  │                                        └─> CLOSED
  │
  └─ [Server listens passively]
     └─> LISTEN
          │
          └─ [Gets SYN]
             └─> SYN_RECEIVED
                  │
                  └─ [Sends SYN-ACK, gets ACK]
                     └─> ESTABLISHED
                          │
                          ├─ [Data transfer]
                          │
                          └─ [Client sends FIN]
                             └─> CLOSE_WAIT
                                  │
                                  └─ [Server app closes]
                                     └─> LAST_ACK
                                          │
                                          └─ [Gets ACK]
                                             └─> CLOSED
```

**Security Implications of TCP:**
- TCP sequence numbers are used in attacks (TCP/IP spoofing)
- Connection state can be exploited
- TCPDump shows full conversation (plaintext if not HTTPS)
- Port scanning works because of TCP behavior

### UDP (User Datagram Protocol): The Speed Protocol

**Conceptual Foundation:** UDP sends data WITHOUT guarantees, optimizing for speed

```
UDP Philosophy:
┌─────────────────────────────────┐
│ Send it and hope for the best!  │
│                                 │
│ Characteristics:                │
│ ├─ NO connection setup          │
│ ├─ NO acknowledgment             │
│ ├─ NO sequence numbers           │
│ ├─ NO retransmission             │
│ └─ NO ordering guarantee         │
│                                 │
│ Result:                          │
│ ├─ 50-60% faster than TCP       │
│ ├─ 90% less overhead            │
│ └─ But: Data WILL be lost      │
└─────────────────────────────────┘

Why use UDP?
├─ Video streaming: Drop 1 frame, no big deal
├─ Online gaming: Need fast position updates
├─ DNS: If no response, just ask again
├─ VoIP: Slight audio drop better than delay
└─ IoT sensors: Send status periodic

Why NOT use UDP?
├─ Email: Can't lose message
├─ Banking: Need verified delivery
├─ File transfer: Want every byte
└─ Any critical data
```

**Advantages:**
✓ Guarantees all data arrives
✓ Maintains order
✓ Handles retransmission

**Disadvantages:**
✗ Slower (acknowledgment overhead)
✗ More processing

### Port Numbers: How Multiple Services Coexist on One IP

**Fundamental Concept:** Port numbers allow ONE IP address to host MANY services

```
┌─ Why ports needed ─────────────────────────────┐
│ Your computer: 192.168.1.100                    │
│ ├─ Running web server                           │
│ ├─ Running email server                         │
│ ├─ Running SSH server                           │
│ ├─ Running 50 browser connections               │
│                                                 │
│ Problem: Packet arrives for 192.168.1.100      │
│ Question: Which program gets it?               │
│ Answer: Look at the PORT NUMBER in the packet! │
└──────────────────────────────────────────────────┘

Port Range:
├─ 0-1023:     Well-known ports (require root/admin)
│  ├─ 22:      SSH (secure shell)
│  ├─ 80:      HTTP (web)
│  ├─ 443:     HTTPS (secure web)
│  ├─ 53:      DNS (name resolution)
│  └─ 25:      SMTP (email)
│
├─ 1024-49151: Registered ports (user apps)
│  ├─ 3306:    MySQL
│  ├─ 5432:    PostgreSQL
│  ├─ 8080:    Common app port
│  └─ 27017:   MongoDB
│
└─ 49152-65535: Dynamic/ephemeral ports
   ├─ Assigned by OS when you connect OUT
   ├─ When you visit a website:
   │  ├─ You connect FROM port 52891 (random)
   │  ├─ TO port 443 (HTTPS)
   │  └─ Server knows to reply to 52891
   └─ Not listening ports, just temp addresses

Socket = IP + Port + Protocol
  Example: 192.168.1.100:22/TCP
  ├─ IP: 192.168.1.100
  ├─ Port: 22
  └─ Protocol: TCP
```

## DNS (Domain Name System)

**Purpose:** Translate human-readable domain names to IP addresses

### DNS Query Process

```
Step 1: You type in browser
├─ www.example.com

Step 2: Recursive Resolver
├─ Asks "What's the IP for www.example.com?"
├─ Usually your ISP's DNS server

Step 3: Root Nameserver
├─ Resolver asks root nameserver
├─ Root responds: "Ask the TLD server"

Step 4: TLD Nameserver
├─ Resolver asks TLD nameserver (.com, .org, etc)
├─ TLD responds: "Ask example.com's nameserver"

Step 5: Authoritative Nameserver
├─ Resolver asks example.com's nameserver
├─ Gets back: "www.example.com = 93.184.216.34"

Step 6: Browser Gets IP
├─ Resolver caches: www.example.com → 93.184.216.34
├─ Browser receives IP address
└─ Browser connects to 93.184.216.34:80

Step 7: HTTP Request
├─ Browser connects to IP on port 80 (HTTP) or 443 (HTTPS)
└─ Sends HTTP request
```

### DNS Records

```
A Record:     www.example.com → 93.184.216.34 (IPv4)
AAAA Record:  www.example.com → 2606:2800:220:1:... (IPv6)
CNAME Record: blog.example.com → www.example.com
MX Record:    example.com → mail.example.com (Email)
NS Record:    example.com → ns1.example.com (Nameserver)
TXT Record:   example.com → "v=spf1..." (SPF, DKIM)
```

### DNS in Linux

```bash
# Query DNS (get IP from domain)
nslookup www.example.com
dig www.example.com

# Reverse DNS (get domain from IP)
nslookup 93.184.216.34

# Shows query process
dig www.example.com +trace
```

## HTTP and HTTPS

### HTTP (Hypertext Transfer Protocol)

**Purpose:** Transfer web pages and resources

**Request Structure:**
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Connection: close

[body content if POST]
```

**Response Structure:**
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

[HTML content here]
```

**Common Status Codes:**
```
200 OK          = Success
301 Moved       = Permanent redirect
304 Not Changed = Cached version is still valid
401 Unauthorized= Need authentication
403 Forbidden   = Access denied
404 Not Found   = Page doesn't exist
500 Server Error= Server problem
503 Unavailable = Server temporarily down
```

### HTTPS (HTTP Secure)

**Difference:** Adds encryption and authentication via TLS/SSL

**How HTTPS Works:**

```
Browser                          Server
  │                                │
  ├── ClientHello ────────────────▶(List of supported ciphers)
  │                                │
  │◀── ServerHello ────────────────┤(Choose cipher, send certificate)
  │                                │
  ├── Verify Certificate ──────────▶(Check certificate is valid)
  │                                │
  ├── Exchange Encryption Keys ───▶(Both generate session key)
  │                                │
  ├═══ Encrypted Connection ══════▶(All data now encrypted)
  │
```

**Benefits of HTTPS:**
✓ Encryption (data can't be read in transit)
✓ Authentication (you're talking to real server)
✓ Integrity (data can't be modified)

## Network Protocols Summary

| Protocol | Layer | Purpose | Reliable | Speed |
|----------|-------|---------|----------|-------|
| TCP | 4 | Reliable connection | Yes | Slower |
| UDP | 4 | Fast transfer | No | Faster |
| IP | 3 | Routing | No | Medium |
| DNS | 7 | Name resolution | Yes | Fast |
| HTTP | 7 | Web transfer | Yes | Medium |
| HTTPS | 7 | Secure web | Yes | Medium |
| SSH | 7 | Remote access | Yes | Medium |

## Practical: Test Internet Connectivity

### Command 1: Ping (Test Reachability)

```bash
ping 8.8.8.8
# ICMP Echo Request to Google DNS

ping www.example.com
# Resolves domain, then pings IP

# Responses show:
# - Host is reachable
# - Latency (milliseconds)
# - No packet loss
```

### Command 2: Traceroute (See Hops to Destination)

```bash
traceroute www.example.com

# Output shows path:
# 1 → Your local gateway
# 2 → ISP router
# 3 → Backbone network
# ... → Intermediate routers
# n → Destination server
```

### Command 3: Nslookup (Resolve Domain to IP)

```bash
nslookup www.example.com

# Returns:
# Server: 8.8.8.8 (DNS server used)
# Address: 93.184.216.34 (Result)
```

### Command 4: Curl (Test HTTP Connection)

```bash
# GET request
curl http://www.example.com

# See response headers
curl -i http://www.example.com

# Follow redirects
curl -L http://www.example.com

# HTTPS with certificate verification
curl https://www.example.com
```

## Key Takeaways

**Remember:**
1. **IP addresses** identify computers on network
2. **Ports** identify services/applications on a computer
3. **TCP** ensures reliable delivery, **UDP** is faster
4. **DNS** translates names to IP addresses
5. **HTTP/HTTPS** transfers web pages
6. **Encryption (HTTPS)** protects data in transit

## Next Steps

- Practice the commands above
- Understand how a URL request travels across the internet
- You'll use this knowledge for firewall rules (Week 2)

---

**Next:** [Module 2: Linux Command Line Mastery](../Part-2/Part-2-Linux-Command-Line.md)
