# Part 1 Exercises: Internet Fundamentals

**Time:** 2 hours  
**Platform:** All (Linux/Windows/macOS)

## Exercise 1.1: OSI Model Mapping

**Objective:** Understand layer relationships

**Task:**
Map the following protocols/tools to OSI layers:
- HTTP, HTTPS, SSH, Telnet (Layer 7)
- TCP, UDP (Layer 4)
- IP, ICMP, Routing (Layer 3)
- Ethernet, MAC Addresses, ARP (Layer 2)
- Physical Cables, WiFi Signals (Layer 1)

**Deliverable:**
- Document showing each with layer description
- Explain why each belongs to that layer
- Real-world example of data flow through layers

## Exercise 1.2: TCP Handshake Visualization

**Objective:** Understand TCP connection establishment

**All Platforms:**

```bash
# Linux/macOS/Windows with Wireshark:
# 1. Install Wireshark
# 2. Start packet capture
# 3. Open browser to http://example.com
# 4. Stop capture
# 5. Filter: tcp.flags.syn==1 OR tcp.flags.ack==1
# 6. Observe SYN, SYN-ACK, ACK packets
```

**Windows Users (WSL):**
```bash
# In WSL terminal
# Install tshark (command-line Wireshark)
sudo apt update
sudo apt install tshark

# Capture packets
sudo tshark -i eth0 -w capture.pcap tcp

# View in Windows Wireshark GUI using WSL file
```

**Deliverable:**
- Screenshot of SYN, SYN-ACK, ACK packets
- Annotated diagram showing handshake
- Explanation of each step

## Exercise 1.3: DNS Resolution Tracing

**Objective:** Trace DNS query path

**All Platforms:**

```bash
# Windows (PowerShell)
nslookup www.google.com
nslookup -querytype=MX gmail.com

# Linux/macOS/WSL
nslookup www.google.com 8.8.8.8
dig www.google.com +trace
dig www.google.com +short

# Detailed trace
dig @8.8.8.8 www.google.com +trace
```

**Deliverable:**
- Output from dig +trace (shows all hierarchy)
- Explanation of each nameserver level
- Recursive resolver IP identification
- Time taken for resolution

## Exercise 1.4: HTTP Request/Response Analysis

**Objective:** Understand HTTP protocol

**All Platforms:**

```bash
# Windows PowerShell
curl -i http://example.com
curl -i https://example.com

# Linux/macOS/WSL
curl -v http://example.com
curl -i https://example.com
curl -L http://example.com  # Follow redirects

# See only headers
curl -I https://example.com

# Custom headers
curl -H "User-Agent: MyClient/1.0" https://example.com
```

**Deliverable:**
- Full HTTP request and response
- Identify all headers (explain each)
- Response codes explanation
- Security headers present/missing (Strict-Transport-Security, etc)

## Exercise 1.5: Network Tools Comparison

**Objective:** Use multiple tools for same goal

**All Platforms:**

```bash
# Ping (ICMP)
ping google.com
ping -c 4 google.com        # Linux/macOS: 4 packets
ping -n 4 google.com        # Windows: 4 packets

# Traceroute path
tracert google.com          # Windows
traceroute google.com       # Linux/macOS
mtr google.com              # Better traceroute (install via package manager)

# DNS queries
nslookup google.com
dig google.com
host google.com             # If installed

# HTTP testing
curl https://google.com
wget https://google.com     # Alternative to curl
```

**Deliverable:**
- Output from each tool
- Comparison matrix (strengths/weaknesses)
- Tool selection criteria per scenario

## Exercise 1.6: Port & Service Identification

**Objective:** Understand port assignments

**Task:**
Research and create table:
- Port 22 (SSH): What service? Why this port?
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
