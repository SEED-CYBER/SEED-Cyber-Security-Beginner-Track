# part 1-Internet Fundamentals

## The Internet: How It Works End-to-End

Before securing systems, you must understand how they communicate. This module covers the fundamental protocols and concepts that enable internet connectivity.

## The OSI Model (7 Layers)

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

### IP (Internet Protocol)

**Purpose:** Route data between computers using IP addresses

```
IPv4 Address: 192.168.1.100
              └─ 32 bits total (4 octets of 8 bits each)
              └─ Allows ~4.3 billion unique addresses

IPv6 Address: 2001:0db8:85a3::8a2e:0370:7334
              └─ 128 bits total
              └─ Allows ~340 undecillion addresses
```

**IP Packet Structure:**
```
┌────────────────────────────────┐
│ Destination IP: 8.8.8.8        │
│ Source IP: 192.168.1.100       │
│ Protocol: TCP (6) or UDP (17)  │
│ Data: [actual message]         │
└────────────────────────────────┘
```

### TCP (Transmission Control Protocol)

**Purpose:** Reliable, ordered delivery of data

**Three-Way Handshake (Connection Establishment):**

```
Client                              Server
  │                                   │
  ├──── SYN (seq=X) ─────────────────▶
  │                                   │
  │◀──── SYN-ACK (seq=Y, ack=X+1) ───┤
  │                                   │
  ├──── ACK (seq=X+1, ack=Y+1) ──────▶
  │                                   │
  ├─── [Connection Established] ────▶
  │                                   │
  ├──── [Data Transfer] ─────────────▶
  │                                   │
  ├──── FIN ─────────────────────────▶
  │                                   │
  │◀──── FIN-ACK ─────────────────────┤
  │                                   │
```

**Advantages:**
✓ Guarantees all data arrives
✓ Maintains order
✓ Handles retransmission

**Disadvantages:**
✗ Slower (acknowledgment overhead)
✗ More processing

### UDP (User Datagram Protocol)

**Purpose:** Fast, unreliable delivery of data

**Characteristics:**
- No connection setup
- No guaranteed delivery
- No order guarantee
- Lower overhead, faster

**Use Cases:**
- Video streaming (okay to lose frames)
- Online gaming (need speed)
- DNS queries (resend if needed)
- VoIP (prefer speed over reliability)

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
