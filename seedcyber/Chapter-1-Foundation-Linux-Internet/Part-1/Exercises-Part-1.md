# Part 1 Exercises: Internet Fundamentals

**Time:** 2-3 hours  
**Platform:** All (Linux/Windows/macOS)  
**Prerequisites:** Understand the OSI model and TCP/IP basics from Part 1 content

---

## Exercise 1.1: OSI Model Mapping (Deep Dive)

**Objective:** Understand layer relationships and recognize protocols at each layer

**Why this matters:** In security, attacks often target specific layers. Knowing which layer a protocol operates at tells you which defensive measures apply.

### Part A: Protocol-to-Layer Mapping

**Task:**
Create a document that maps these protocols/concepts to OSI layers with explanations:

**Application Layer (Layer 7) - User Programs:**
```
Protocol        | Purpose                      | Security Consideration
─────────────────────────────────────────────────────────────────
HTTP            | Transfer web pages           | NO encryption (use HTTPS)
HTTPS           | "HTTP + TLS encryption"     | Encrypted end-to-end
SSH             | Secure remote shell          | Encrypted tunneling
Telnet          | Remote shell (legacy)        | PLAINTEXT - Never use!
DNS             | Name to IP translation       | Can be spoofed (DNS spoofing)
SMTP            | Send email                   | Often unencrypted
POP3/IMAP       | Retrieve email               | Should be encrypted
FTP             | File transfer (legacy)       | PLAINTEXT passwords
SFTP            | SSH + file transfer          | Encrypted file transfer
```

**Deliverable A:**
- Comprehensive document explaining EACH protocol
- For each protocol, explain:
  1. What problem does it solve?
  2. How does it work?
  3. What security risks exist?
  4. How would you protect it?

**Example answer for HTTP:**
```
HTTP (HyperText Transfer Protocol)
──────────────────────────────────
Problem it solves:
  - Need a standard way to request web pages
  - Server sends back HTML, images, etc.

How it works (fundamentals):
  1. Client opens TCP connection to port 80
  2. Client sends: GET /index.html HTTP/1.1
  3. Server responds with HTML content
  4. Connection closes after response

Security risks:
  - NO ENCRYPTION (anyone can intercept)
  - No authentication (imposters possible)
  - Headers visible in plaintext
  - Passwords transmitted in clear!

How to protect it:
  - ALWAYS use HTTPS instead
  - HTTPS = HTTP + TLS encryption
  - Your browser warns if site uses HTTP
```

### Part B: Data Flow Through Layers

**Task:**
Trace a single HTTP request through all 7 layers. When you visit http://example.com:

**Step 1: Application Layer (Layer 7)**
```
Your action:        Type www.example.com in browser
Browser creates:    GET /index.html HTTP/1.1
                    Host: www.example.com
                    User-Agent: Mozilla/5.0

Question for you:   What information is the browser sending?
```

**Step 2: Presentation Layer (Layer 6)**
```
Browser checks:     "Is this HTTPS?" → NO, so no encryption
                    "What character encoding?" → UTF-8

Question for you:   What WOULD happen if this was HTTPS?
                    (Hint: Read Part 3 content on TLS)
```

**Step 3: Session Layer (Layer 5)**
```
Browser says:       "Remember this is one conversation"
                    (Cookie management)

Question for you:   How do cookies track users?
```

**Step 4: Transport Layer (Layer 4 - TCP)**
```
OS adds:            Source port: 52891 (random, assigned by OS)
                    Dest port: 80 (HTTP default)
                    Sequence: 1000
                    Checksum: [calculated for reliability]

Package name:       SEGMENT

Question for you:   Why does the OS pick a random port?
                    Why not use the same port repeatedly?
```

**Step 5: Network Layer (Layer 3 - IP)**
```
OS adds:            Source IP: 192.168.1.100 (your computer)
                    Dest IP: 93.184.216.34 (example.com)
                    TTL: 64 (can hop through 64 routers)
                    Protocol: 6 (means TCP)

Package name:       PACKET

Question for you:   How would router know where to send this?
                    (Hint: routing tables, BGP protocol)
```

**Step 6: Data Link Layer (Layer 2 - Ethernet)**
```
NIC adds:           Source MAC: AA:BB:CC:DD:EE:FF (your computer)
                    Dest MAC: 11:22:33:44:55:66 (your router!)
                    Frame type: IPv4
                    CRC: [error checking]

Package name:       FRAME

Question for you:   Why is destination MAC your ROUTER?
                    Not example.com's computer directly?
                    (Hint: ARP, same network only)
```

**Step 7: Physical Layer (Layer 1)**
```
NIC converts:       Frame → electrical pulses on Ethernet
                    OR radio waves if WiFi
                    1010010101011... (bits on the wire)

Question for you:   What happens if 1s and 0s get corrupted
                    on the physical wire?
```

**Deliverable B:**
- Diagram showing data at each layer
- Explain what gets added at EACH layer
- Explain what gets removed when data arrives at destination

---

## Exercise 1.2: TCP Handshake Observation & Analysis

**Objective:** SEE the TCP three-way handshake happen in real time using packet capture

**Why this matters:** Understanding TCP connection establishment is critical for:
- Port scanning (how attackers find services)
- Connection hijacking attacks
- Firewall rules (how to block unwanted connections)
- Debugging network issues

### Prerequisites:
- Install either Wireshark (GUI) or tshark (command-line)
- OR use tcpdump if on Linux/macOS

### Part A: Windows Users (Wireshark GUI)

**Step 1: Install Wireshark**
```
Option A: Chocolatey (Windows package manager)
  choco install wireshark

Option B: Download from wireshark.org
  1. Go to https://www.wireshark.org/download/
  2. Download Windows x64 installer
  3. Run installer, accept defaults
  4. Might need to install Npcap (packet capture library)
```

**Step 2: Start Packet Capture**
```
1. Open Wireshark
2. Double-click your active network interface
   (Usually "Ethernet" or "Wi-Fi")
3. RED CIRCLE appears - capturing started!
```

**Step 3: Generate HTTP Traffic**
```
1. Open command prompt
2. Type: curl http://example.com
3. You should see HTML output
4. This creates network traffic to capture
```

**Step 4: Stop Capture & Filter**
```
1. Click the RED square to stop capture
2. In filter box at top, type: tcp.flags.syn==1 or tcp.flags.ack==1
3. This shows ONLY SYN/ACK packets (handshake)
```

**Step 5: Analyze the Capture**
```
Look for 3 lines:
1st line:  [Your IP]:52891 → 93.184.216.34:80  [SYN]
           └─ You initiating, flags show SYN only

2nd line:  93.184.216.34:80 → [Your IP]:52891  [SYN, ACK]
           └─ Server responding, SYN and ACK flags set

3rd line:  [Your IP]:52891 → 93.184.216.34:80  [ACK]
           └─ You confirming, ACK flag set
```

### Part B: Linux/WSL/macOS Users (Command Line)

**Step 1: Install tshark (if needed)**
```bash
# Ubuntu/Debian/WSL
sudo apt update
sudo apt install tshark

# macOS (with Homebrew)
brew install wireshark
```

**Step 2: Capture Traffic to File**
```bash
# Start capturing on default interface
# Ctrl+C after ~30 seconds

sudo tshark -i eth0 -w handshake.pcap tcp port 80

# In another terminal, generate traffic:
curl http://example.com

# Press Ctrl+C in tshark terminal to stop
```

**Step 3: Analyze the Capture**
```bash
# Read the capture file
tshark -r handshake.pcap

# See more details
tshark -r handshake.pcap -V | grep -A 5 "SYN\|ACK"
```

**Step 4: View in Wireshark GUI (Cross-Platform)**
```
Even on Linux, you can open handshake.pcap in Wireshark GUI
  wireshark handshake.pcap &
```

### Deliverable A:
```
❑ Screenshot of Wireshark showing:
  - SYN packet from your IP to destination
  - SYN-ACK packet returning
  - ACK packet from you

❑ Table showing for EACH packet:
  ┌─────────────────────────────────────────────┐
  │ Packet │ From        │ To          │ Flags   │
  ├─────────────────────────────────────────────┤
  │ 1      │ 192.168...  │ 93.184...   │ SYN     │
  │ 2      │ 93.184...   │ 192.168...  │ SYN,ACK │
  │ 3      │ 192.168...  │ 93.184...   │ ACK     │
  └─────────────────────────────────────────────┘

❑ Written explanation:
  - What does SYN mean?
  - What does ACK mean?
  - Why do we need all 3 packets?
  - What would happen if packet 2 never arrived?
```

### Analysis Questions:
1. **Sequence Numbers:** Look at the "Seq" and "Ack" fields
   - Do they match what we learned? (seq doubles on each response)
   - Why are they large numbers (not just 1, 2, 3)?

2. **Timing:** Look at the timestamps of the 3 packets
   - How long does a handshake typically take?
   - Why might this vary?

3. **After Handshake:** Scroll down in Wireshark
   - What packet comes AFTER the 3-way handshake?
   - That's the HTTP GET request!
   - See the HTTP data in the packet?

---

## Exercise 1.3: DNS Resolution Tracing (Complete Walk-Through)

**Objective:** Trace a DNS query from your computer → root nameserver → TLD → authoritative

**Why this matters:**
- DNS attacks can redirect you to fake websites
- Understanding DNS hierarchy prevents cache poisoning attacks
- DNS exfiltration is used in some data breaches
- DNS filtering is part of security defenses

### Part A: Simple DNS Query

**Step 1: Windows (PowerShell)**
```powershell
# Simple query
nslookup www.google.com

# Expected output:
# Server:  192.168.0.1          (your ISP's DNS)
# Address: 192.168.0.1#53       (port 53 = DNS default)
#
# Non-authoritative answer:
# Name:    www.google.com
# Addresses: 142.250.185.46
#            2607:f8b0:4004:...
```

**Step 2: Windows (Detailed)**
```powershell
# See MX records (email servers)
nslookup -type=MX google.com

# See NS records (nameservers)
nslookup -type=NS google.com

# See ALL records
nslookup -type=ANY google.com
```

**Step 3: Linux/WSL (dig command)**
```bash
# Simple query
dig www.google.com

# See the full hierarchy
dig www.google.com +trace

# Expected output shows:
# 1. Root nameserver query
# 2. TLD (.com) nameserver query
# 3. Authoritative nameserver query
# 4. Final IP address
```

### Part B: Complete DNS Hierarchy Trace

**Step 1: Do Full Trace (all platforms)**
```bash
# Linux/WSL/macOS
dig www.google.com +trace

# Windows (using dig if installed, or nslookup recursively)
nslookup www.google.com 8.8.8.8  (Google's public DNS)
```

**Output will show (example):**
```
; <<>> DiG 9.16.1-Ubuntu <<>> www.google.com +trace
;; global options: +cmd

;; Received 488 bytes from 127.0.0.1#53 in 0 ms

.               518400  IN      NS      a.root-servers.net.
.               518400  IN      NS      b.root-servers.net.
...               (* 13 root servers total *)

com.            172800  IN      NS      a.gtld-servers.net.
com.            172800  IN      NS      b.gtld-servers.net.
...               (* nameservers for .com domain *)

google.com.     172800  IN      NS      ns1.google.com.
google.com.     172800  IN      NS      ns2.google.com.
...

www.google.com. 300     IN      A       142.250.185.46
```

### Step-by-Step Explanation:

**Stage 1: Root Servers (.)**
```
Question: "Where can I find www.google.com?"
Root answer: "Don't know, but ask .com TLD servers"
└─ Shows 13 root servers (a.root-servers.net through m.root-servers.net)
```

**Stage 2: TLD Servers (.com)**
```
Question: "Where can I find www.google.com?"
.com answer: "Don't know about www, but ask google.com's nameservers"
└─ Shows .com nameservers (a.gtld-servers.net, etc)
```

**Stage 3: Authoritative Server (google.com)**
```
Question: "What's the IP for www.google.com?"
google.com answer: "It's 142.250.185.46"
└─ This is AUTHORITATIVE - google.com controls this domain!
```

### Deliverable B:
```
❑ Full output of dig +trace (or equivalent)

❑ Diagram showing:
  Your Computer
      ↓ (Q: Where's www.google.com?)
  Local Resolver (ISP DNS)
      ↓ (Q: Where's www.google.com?)
  Root Nameserver  
      ↓ (A: Ask .com TLD)
  TLD Nameserver (.com)
      ↓ (A: Ask google.com)  
  Authoritative NS (google.com)
      ↓ (A: 142.250.185.46)
  Back to your computer

❑ Written answers:
  - How many "hops" did the query take?
  - What's the difference between recursive and iterative queries?
  - Where is the answer cached? (resolver caches it!)
```

### Analysis Questions:
1. **Timing:** Run the same query twice
   ```bash
   time dig www.google.com      (first time - slower)
   time dig www.google.com      (second time - faster!)
   ```
   Why is the 2nd faster? (DNS caching by resolver)

2. **Cache TTL:** In dig output, see "300" next to final answer
   ```
   www.google.com. 300 IN A 142.250.185.46
                   ↑
                   TTL (Time To Live in seconds)
   ```
   - This means: cache for 300 seconds = 5 minutes
   - After 5 minutes, query again (cache expires)
   - Why might Google set TTL to 5 minutes? (allows load balancing)

3. **Security Issue - Spoofing:**
   - What if attacker intercepts DNS query?
   - Attacker responds: "www.google.com = 1.2.3.4" (fake IP)
   - Your browser connects to 1.2.3.4 (attacker's server!)
   - This is DNS spoofing / DNS hijacking
   - How to prevent? DNSSEC (DNS Security)

---

## Exercise 1.4: HTTP Request/Response Analysis

**Objective:** Inspect actual HTTP headers and understand what information is sent

**Why this matters:**
- HTTP headers contain metadata about your request
- Security headers (or lack thereof) indicate site security posture
- Attackers inspect headers to find exploitable versions
- Understanding HTTP helps with API security

### Part A: Basic HTTP Request

**Step 1: Simple Request (All Platforms)**
```bash
# Windows PowerShell
curl -i http://example.com

# Linux/macOS/WSL (curl or wget)
curl -i http://example.com
```

**Step 2: Verbose Request (See everything)**
```bash
curl -v http://example.com

# Output shows:
# > GET / HTTP/1.1         (request line)
# > Host: example.com      (headers)
# > Connection: keep-alive
# 
# < HTTP/1.1 200 OK        (response line)
# < Content-Type: text/html
# < Content-Length: 1234
```

**Step 3: Analyze Request Headers**
```bash
curl -v http://example.com 2>&1 | grep "^>"
```

**Expected output (your request):**
```
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/7.68.0
> Accept: */*
> Connection: keep-alive
```

### Understanding Each Header:

```
GET / HTTP/1.1
├─ GET = HTTP method (retrieve resource)
├─ / = path (root of website)
└─ HTTP/1.1 = protocol version

Host: example.com
├─ Required header (HTTP/1.1)
├─ Tells server which domain you want
└─ (important when server hosts multiple domains)

User-Agent: curl/7.68.0
├─ Identifies the client software
├─ Servers use this for compatibility
├─ Problem: Reveals browser type/version to attackers!

Accept: */*
├─ Tells server: "I'll accept any content type"
├─ Could be specific: Accept: text/html, application/json

Connection: keep-alive
├─ "Keep this TCP connection open"
├─ Multiple requests can reuse same connection
├─ Faster than opening new connection each time
```

### Part B: HTTP Response Headers

**Step 1: See Response Headers**
```bash
curl -i http://example.com

# OR just headers (no body)
curl -I http://example.com
```

**Step 2: Analyze Response**
```
HTTP/1.1 200 OK
├─ 200 = Success

Content-Type: text/html; charset=utf-8
├─ This is HTML content
├─ Uses UTF-8 character encoding

Content-Length: 1256
├─ Response body is 1256 bytes

Cache-Control: max-age=3600
├─ Browser can cache for 1 hour (3600 seconds)
├─ After 1 hour, request again from server

Server: Apache/2.4.41
├─ Web server software (Apache version 2.4.41)
├─ ⚠️ SECURITY ISSUE: Reveals server type!
└─ Attackers can look for known Apache vulnerabilities

Set-Cookie: sessionid=abc123; Path=/
├─ Browser stores cookie for future requests
├─ Cookie is sent with every request
├─ ⚠️ If not HttpOnly/Secure, JavaScript can steal it!
```

### Part C: Security Headers (or Lack Thereof)

**Step 1: Check Security Headers**
```bash
# GET https://example.com (HTTPS - important!)
curl -I https://example.com

# Look for these GOOD headers:
# ✓ Strict-Transport-Security: max-age=31536000
# ✓ X-Content-Type-Options: nosniff
# ✓ X-Frame-Options: DENY
# ✓ Content-Security-Policy: [policy]

# Missing = vulnerability!
```

**Deliverable C:**
```
❑ Full curl output of:
  - HTTP request to http://example.com
  - HTTPS request to https://example.com

❑ Table of response headers:
  ┌──────────────────────┬───────┬──────────────────────┐
  │ Header               │ Value │ What it means        │
  ├──────────────────────┼───────┼──────────────────────┤
  │ Content-Type         │ text/html │ HTML page       │
  │ Content-Length       │ 1234  │ Page size           │
  │ Server               │ Apache│ Server software     │
  │ ... (all headers)    │ ...   │ ...                 │
  └──────────────────────┴───────┴──────────────────────┘

❑ Security analysis:
  - What security headers are PRESENT?
  - What security headers are MISSING?
  - Is server revealing its version? (security issue?)
  - Is Strict-Transport-Security set? (HTTPS requirement)
```

### Analysis Questions:
1. **HTTP vs HTTPS:**
   ```bash
   curl -i http://example.com    (insecure)
   curl -i https://example.com   (secure)
   ```
   - What's the difference in headers?
   - Does HTTPS have Strict-Transport-Security?

2. **Cookies:**
   Look for Set-Cookie headers
   - Is HttpOnly flag set? (protects from JavaScript theft)
   - Is Secure flag set? (only sent over HTTPS)
   - Is SameSite set? (prevents CSRF attacks)

3. **Caching:**
   Look for Cache-Control header
   - If set to long TTL, means data doesn't change often
   - If max-age=0, means always refetch

---

## Exercise 1.5: Network Tools Comparison

**Objective:** Use 4+ tools to accomplish the same goal

**Why this matters:**
- Different tools have different use cases
- Security teams use multiple tools for redundancy
- Some tools are better for debugging, others for forensics
- Tool selection depends on the task

### Goal: Map the route to google.com

**Method A: Using Ping (ICMP Echo)**
```bash
# Windows
ping google.com -n 4

# Linux/macOS/WSL
ping -c 4 google.com

# Output shows:
# PING google.com (142.250.185.46) 56(84) bytes of data.
# 64 bytes from 142.250.185.46: icmp_seq=1 ttl=57 time=14.2 ms
# 64 bytes from 142.250.185.46: icmp_seq=2 ttl=57 time=13.8 ms
# ...
# Packet loss: 0%

# Interpretation:
# ✓ google.com is reachable
# ✓ 4 probes sent, 4 returned (0% loss = good)
# ✓ Each took ~14ms round-trip time (RTT)
# ✓ TTL=57 (started at 64, passed through 7 routers)
```

**Method B: Using Traceroute (ICMP with TTL)**
```bash
# Windows
tracert google.com

# Linux/macOS/WSL
traceroute google.com

# Output shows path:
# traceroute to google.com (142.250.185.46), 30 hops max
#  1  router.home (192.168.1.1)         1.234 ms
#  2  isp-gateway (10.0.0.1)             5.123 ms
#  3  isp-backbone (203.0.113.1)        12.456 ms
#  4  internet-exchange (206.152.14.1)  15.234 ms
#  5  google-edge (142.250.185.1)       14.567 ms
#  6  google-core (142.250.185.46)      14.789 ms

# Interpretation:
# ├─ Shows EVERY router between you and google.com
# ├─ 6 hops to reach Google
# ├─ Each hop shows response time
# ├─ If hop shows "*", that router doesn't respond (normal)
# └─ Useful for finding where network problems are!
```

**Method C: Using MTR (Better Traceroute)**
```bash
# Install if needed:
# Windows: choco install mtr
# Linux: apt install mtr-tiny
# macOS: brew install mtr

mtr -c 4 google.com

# Output shows continuously updated:
# HOST: mycomputer
# Packets: Sent = 4, Received = 4
#  1. 192.168.1.1      0% loss   4.5 ms
#  2. 10.0.0.1         0% loss   8.3 ms
#  3. 203.0.113.1      0% loss  12.4 ms
#  4. ... (shows all hops with stats)

# Advantages over traceroute:
# ✓ Continuous, real-time updates
# ✓ Shows loss percentage per hop
# ✓ Shows average/min/max time
# ✓ Better for spotting intermittent issues
```

**Method D: Using DNS (nslookup/dig)**
```bash
# First, get IP from domain
nslookup google.com
# Returns: 142.250.185.46

# Then reverse lookup
nslookup 142.250.185.46
# Returns: host name for that IP

# dig equivalent (more detailed)
dig google.com
# Shows query process, response, stats
```

### Deliverable D:

Create comparison table:

```
┌─────────────┬──────────────────┬───────────────────┬─────────────────┐
│ Tool        │ What it does      │ Best for          │ Limitations     │
├─────────────┼──────────────────┼───────────────────┼─────────────────┤
│ Ping        │ ICMP Echo test    │ Quick reachability│ No route info   │
│             │ Is host alive?    │ latency measure   │ Blocked by fw   │
├─────────────┼──────────────────┼───────────────────┼─────────────────┤
│ Traceroute  │ Maps full route   │ Find bad hop      │ Slow, 30 hops   │
│             │ Path to destina   │ Bottleneck       │ No stats        │
├─────────────┼──────────────────┼───────────────────┼─────────────────┤
│ MTR         │ Cont. traceroute  │ Real-time stats   │ Requires install│
│             │ Live hop stats    │ Intermittent issues│ Terminal only   │
├─────────────┼──────────────────┼───────────────────┼─────────────────┤
│ nslookup    │ DNS lookups       │ Domain → IP       │ Doesn't show    │
│             │ Reverse lookup    │ DNS issues        │ full query path │
├─────────────┼──────────────────┼───────────────────┼─────────────────┤
│ dig         │ Detailed DNS      │ DNS debugging     │ Complex output  │
│             │ Query details     │ Find nameservers  │ Not on Windows  │
└─────────────┴──────────────────┴───────────────────┴─────────────────┘
```

### Analysis Questions:
1. **When would you use each?**
   - User says "Website is slow" → Which tool? (traceroute or MTR)
   - Domain won't resolve → Which tool? (nslookup or dig)
   - Server not responding → Which tool? (ping)

2. **Interpreting Results:**
   - If traceroute stops at hop 8, what does it mean?
   - If one hop has 50% packet loss, is that bad?
   - If ping shows 200ms latency, is that slow?

3. **Security Perspective:**
   - What can attackers learn from traceroute output?
   - Why might firewall block ping/traceroute?
   - How could you use these for network reconnaissance?

---

## Exercise 1.6: Port & Service Identification

**Objective:** Understand port assignments and service mapping
- Port 80 (HTTP): What service? Risks?
- Port 443 (HTTPS): What service? Advantages?
- Port 3306 (MySQL): What service? Exposure risk?
- Port 5900 (VNC): What service? Security implications?

**All Platforms:**
```bash
# Windows (check services)
Get-Service | Where-Object {$_.Status -eq "Running"}

# Linux/macOS/WSL
ss -tlnp
netstat -tlnp
lsof -i -P -n | grep LISTEN
```

**Deliverable:**
- Table with ports, services, and risks
- Output from your system showing listening ports
- Security implications analysis

---

**Completion Checklist:**
- [ ] All exercises completed
- [ ] Deliverables documented
- [ ] Platform-specific commands tested
- [ ] Ready for Part 2

**Submission:** Part-1-Exercises-SUBMISSION.txt
