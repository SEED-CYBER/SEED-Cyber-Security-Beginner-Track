# Part 2: Firewall Configuration - Enhanced

## The Network Access Problem (Why Firewalls Exist)

### Before Firewalls: Total Exposure

**Challenge:** Your server is connected to the internet. Every attacker can see it.

```
NETWORK REALITY:
  ├─ Port 22 (SSH): Open by default after installation
  ├─ Port 80 (HTTP): Open if web server running
  ├─ Port 443 (HTTPS): Open if web server running
  ├─ Port 3306 (MySQL): Open if database running
  ├─ Port 5432 (PostgreSQL): Open if database running
  └─ Port 6379 (Redis): Open if cache running

PROBLEM:
  ├─ Any of these ports = potential attack vector
  ├─ Attacker scans for open ports
  ├─ Finds vulnerable application
  ├─ Exploits and gains access
  └─ Server compromised!

EXAMPLE ATTACK:
  ├─ Script scans internet for port 3306 (MySQL)
  ├─ Finds your server
  ├─ Connects to MySQL
  ├─ Default credentials? Success!
  ├─ Attacker now reads database
  └─ Customer data stolen!
```

### What is a Firewall?

**Definition: Software that controls all network traffic in/out of server**

```
FIREWALL PRINCIPLE:
  "DEFAULT DENY, EXPLICIT ALLOW"

Concept:
  ├─ Reject ALL incoming connections by default
  ├─ Only allow specific ports/IPs explicitly
  ├─ Everything else gets blocked
  └─ Server still initiates outgoing (you make requests)

Physical Analogy:
  ├─ Company office with security guard
  ├─ Guard's job: Let authorized people in, turn others away
  ├─ No visitor? Guard says: "You're not authorized, go away"
  ├─ Employee ID? Guard says: "Welcome in!"
  └─ Firewall: Guard for network traffic

Real Example:

BEFORE FIREWALL:
  Internet Attacker → Port 22 (OPEN) → SSH Server (VULNERABLE)
                   → Port 3306 (OPEN) → MySQL (EXPLOITED)
                   → Port 5432 (OPEN) → PostgreSQL (COMPROMISED)
  Result: ANY attacker can probe/exploit!

AFTER FIREWALL:
  Internet Attacker → Port 22 (BLOCKED by firewall)
                   → Port 3306 (BLOCKED by firewall)
                   → Port 5432 (BLOCKED by firewall)
  
  But:
  Legitimate Client → Port 22 (ALLOWED to admin user IP)
  Web User         → Port 80 (ALLOWED to anyone)
  Web User         → Port 443 (ALLOWED to anyone)
  
  Result: Only necessary services exposed!
```

### Firewall Rules Explain Concept

```
RULE SYNTAX (Conceptual):

INBOUND RULE:
  IF (source IP, protocol, port, direction) THEN (action)
  
Example 1: Allow SSH from anywhere
  └─ Source: Any IP
  └─ Protocol: TCP
  └─ Port: 22
  └─ Direction: Incoming
  └─ Action: ACCEPT
  
Example 2: Allow HTTP web traffic
  └─ Source: Any IP
  └─ Protocol: TCP
  └─ Port: 80
  └─ Direction: Incoming
  └─ Action: ACCEPT
  
Example 3: Block MySQL from internet
  └─ Source: Any IP
  └─ Protocol: TCP
  └─ Port: 3306
  └─ Direction: Incoming
  └─ Action: DROP/REJECT
  
Example 4: Allow SSH only from office
  └─ Source IP: 192.168.1.0/24 (office network)
  └─ Protocol: TCP
  └─ Port: 22
  └─ Direction: Incoming
  └─ Action: ACCEPT
  
Example 5: Drop everything else
  └─ Source: Any IP
  └─ Protocol: Any
  └─ Port: Any
  └─ Direction: Incoming
  └─ Action: DROP

RULE EVALUATION (Stateful):
  1. Packet arrives from 192.168.1.5 on port 22
  2. Firewall checks rules top-to-bottom
  3. Rule 4 matches: "Allow SSH from office" - ACCEPTED!
  
  1. Packet arrives from 8.8.8.8 on port 3306
  2. Firewall checks rules
  3. Rule 3 matches: "Block MySQL from internet" - DROPPED!
  
  1. Packet arrives from 1.2.3.4 on port 25000
  2. Firewall checks rules
  3. No rules match
  4. Default rule (5) applies: DROP
  5. Connection refused
```

---

## Linux Firewall Architecture

### Kernel-Level Packet Filtering: Netfilter

```
LINUX NETWORK STACK:

Internet → Ethernet NIC (Network card)
         → Linux Kernel
           ├─ PREROUTING (nat, mangle)
           ├─ INPUT (filter)
           ├─ FORWARD (filter)
           ├─ OUTPUT (filter)
           └─ POSTROUTING (nat, mangle)
         → User Applications (sshd, nginx, etc)
         → Response back

IPTABLES:
  ├─ Low-level tool to write kernel rules
  ├─ Complex syntax (many options)
  ├─ Direct control over packet filtering
  ├─ Rules lost on reboot (need persistence)
  └─ Hard to manage (100+ rules = unmaintainable)

UFW (Uncomplicated Firewall):
  ├─ Wrapper around iptables
  ├─ Much simpler syntax
  ├─ Same power underneath
  ├─ Rules persistent (auto-saves)
  ├─ Suitable for most scenarios
  └─ Recommended for learning/production

ADVANTAGE OF UFW:
  Instead of: sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
  Just type:  sudo ufw allow 22
  └─ Much clearer what it does!
```

### Connection Tracking (Stateful Firewall)

```
STATELESS (Old):
  Rule: Allow port 22 incoming
  Result: ALL port 22 traffic allowed
  Problem: Response to YOUR commands gets blocked!
  
STATEFUL (Modern - UFW uses this):
  Rule: Allow port 22 incoming
  Tracking:
    ├─ Connection from 8.8.8.8:12345 → 10.0.0.5:22 ESTABLISHED
    ├─ Response from 10.0.0.5:22 → 8.8.8.8:12345 automatically allowed!
    ├─ Connection from 1.2.3.4:54321 → 10.0.0.5:22 NEW → Check rules
    └─ Connection from random:random → 10.0.0.5:22 NEW → Check rules

WHY THIS MATTERS:
  ├─ You SSH in from 8.8.8.8
  ├─ Connection state: ESTABLISHED
  ├─ Response packets automatically pass through
  ├─ You can type commands!
  ├─ Attacker tries to connect from different IP
  ├─ Connection state: NEW
  ├─ Firewall checks rules
  ├─ If rule allows SSH, they can connect
  ├─ If rule doesn't allow, connection rejected
  └─ Tracking prevents "response blocking" problem!
```

---

## UFW: Practical Configuration

### Understanding UFW Defaults

```
UFW DEFAULT POLICY (Defaults matter!):

  sudo ufw default deny incoming
  └─ Reject ALL incoming = DEFAULT DENY
  └─ Must explicitly allow what you need
  └─ SECURE: Whitelist approach

  sudo ufw default allow outgoing
  └─ Allow ALL outgoing = DEFAULT ALLOW
  └─ Your server can initiate connections
  └─ Makes sense: You control what runs on server

WHY?
  Incoming: You can't control who attacks, so reject all
  Outgoing: You control what software runs, so allow all
  └─ If malware installed, you have bigger problems!
```

### Step-by-Step Setup

```bash
STEP 1: Start fresh (if switching from iptables)
sudo ufw reset
# This clears all existing rules
# Answer 'y' when prompted

STEP 2: Set defaults
sudo ufw default deny incoming
# Default policy: reject all incoming
# Required for whitelist approach

sudo ufw default allow outgoing
# Default policy: allow all outgoing
# Server can initiate connections

STEP 3: Enable logging (optional but useful)
sudo ufw logging on
sudo ufw logging medium
# 'low', 'medium', 'high' verbosity
# Check logs: sudo tail -f /var/log/ufw.log

STEP 4: Critical rule - SSH (before enabling!)
sudo ufw allow 22/tcp
# MUST do this before 'enable'!
# Otherwise: firewall activated and SSH blocked!
# You're locked out!
# (Recovery needed: reboot, manual iptables, etc.)

STEP 5: Web services (if running web server)
sudo ufw allow 80/tcp
# HTTP port
sudo ufw allow 443/tcp
# HTTPS port

STEP 6: Review before activation
sudo ufw show added
# Shows rules about to be added
# Check everything looks right!

STEP 7: Enable
sudo ufw enable
# Firewall now active!
# All rules enforced
# Response: Type 'y' to confirm

STEP 8: Verify
sudo ufw status verbose
# Lists all rules
# Should show:
# ├─ Default: deny (incoming)
# ├─ Default: allow (outgoing)
# ├─ 22/tcp (ALLOW)
# ├─ 80/tcp (ALLOW)
# └─ 443/tcp (ALLOW)

STEP 9: Test from external machine
nmap -p 22,80,443,3306,5432 your.vps.ip
# Port 22: open (SSH allowed by UFW)
# Port 80: open (HTTP allowed by UFW)
# Port 443: open (HTTPS allowed by UFW)
# Port 3306: filtered (MySQL blocked by UFW)
# Port 5432: filtered (PostgreSQL blocked by UFW)
```

### Advanced UFW Rules

```bash
ALLOW SPECIFIC TYPES:

# TCP only (default)
sudo ufw allow 22/tcp
# ├─ UDP request on 22? Blocked
# └─ TCP request on 22? Allowed

# UDP only
sudo ufw allow 53/udp
# ├─ DNS uses UDP
# ├─ TCP request on 53? Blocked
# └─ UDP request on 53? Allowed

# Both TCP and UDP
sudo ufw allow 5353
# No protocol specified = both TCP and UDP

ALLOW FROM SPECIFIC IP:

sudo ufw allow from 192.168.1.100
# Only 192.168.1.100 can SSH
# Any other IP: connection refused
# Use case: Admin IP whitelisting

sudo ufw allow from 192.168.1.100 to any port 22
# Only 192.168.1.100 on port 22
# Other IPs on port 22: blocked

sudo ufw allow from 10.0.0.0/24
# Allow from entire network (10.0.0.0 - 10.0.0.255)
# /24 = subnet mask (256 IPs)
# Use case: Company office network

DENY SPECIFIC:

sudo ufw deny 123/udp
# Block NTP
# Prevents time synchronization
# Why? If don't need it, disable it

sudo ufw deny from 1.2.3.4
# Block ALL ports from this IP
# Use case: Blocking known attacker

RULES IN ORDER:

sudo ufw allow from 192.168.1.100 to any port 22
sudo ufw deny from any to any port 22
# First rule matches? ALLOW
# Second rule: ignored (already decided)
# Result: Only 192.168.1.100 can SSH

VERSUS reverse order:
sudo ufw deny from any to any port 22
sudo ufw allow from 192.168.1.100 to any port 22
# First rule matches EVERYONE? DENY
# Second rule: never reached
# Result: Everyone blocked (admin too!)
# DANGER!
```

### Managing Rules

```bash
VIEW RULES:

sudo ufw status
# Simple list

sudo ufw status verbose
# More detailed

sudo ufw status numbered
# With line numbers (needed for deletion)

DELETE RULES:

# By number (from 'status numbered')
sudo ufw delete 2
# Deletes rule #2

# By description
sudo ufw delete allow 22/tcp
# Deletes the SSH rule

# Delete all and reset
sudo ufw reset
# Clears everything
# Warning: You'll be locked out if no SSH rule!
# Reboot from console to fix

MODIFY RULES:

# Update rule: delete old, add new
sudo ufw delete allow 22/tcp
sudo ufw allow from 192.168.1.100 to any port 22
# Now more restrictive

DISABLE/ENABLE:

# Disable temporarily (testing, troubleshooting)
sudo ufw disable
# Removes all rules
# Server fully open
# Use for 5 minutes only!

# Enable again
sudo ufw enable
# Restores rules from saved config
```

---

## Iptables: Advanced Firewall Control

### When to Use Iptables

```
UFW vs IPTABLES:

USE UFW IF:
  ├─ Simple allow/deny rules
  ├─ Protecting single VPS
  ├─ New to Linux firewalls
  ├─ Need to remember commands
  └─ Rules need to survive reboots

USE IPTABLES IF:
  ├─ Complex NAT/port forwarding
  ├─ Load balancing across servers
  ├─ Deep packet inspection
  ├─ VPN tunnel setup
  ├─ Advanced filtering logic
  └─ Need ultimate control

RELATIONSHIP:
  UFW commands → generates iptables rules
  ufw allow 22 → iptables -A INPUT -p tcp --dport 22 -j ACCEPT
  
  If you use both: They interfere!
  Recommendation: Pick one. Use UFW for simplicity.
```

### Iptables Concepts

```
CHAINS: Processing queues

INPUT:
  ├─ Packets destined for THIS machine
  ├─ Example: Connection to your SSH server
  └─ Firewall decision: ACCEPT or DROP?

FORWARD:
  ├─ Packets passing through (not for you)
  ├─ Example: Server acting as router
  ├─ Common in: VPNs, load balancers
  └─ Usually not needed for single VPS

OUTPUT:
  ├─ Packets generated BY this machine
  ├─ Example: Server connecting to database
  ├─ Usually: Allow all (you control software)
  └─ Can restrict: Advanced security (rare)

TARGETS: What happens to packet

ACCEPT:
  └─ Let packet through

DROP:
  └─ Reject silently (sender gets timeout)

REJECT:
  └─ Reject with error message (ICMP)

LOG:
  └─ Log packet details before applying target
  ├─ Useful for: Debugging, monitoring
  └─ Set in kernel logs (dmesg, journalctl)

TABLES: Different rule categories

FILTER:
  └─ Accept/Drop (what we usually care about)

NAT:
  ├─ Network Address Translation
  ├─ Rewrite IP addresses
  ├─ Example: Port forwarding

MANGLE:
  ├─ Modify packet headers
  └─ Advanced (usually not needed)
```

### Basic Iptables Usage

```bash
VIEW CURRENT RULES:

sudo iptables -L
# Lists rules in filter table
# Format is hard to read

sudo iptables -L -n -v
# ├─ -L = list rules
# ├─ -n = numerics (no DNS lookups)
# ├─ -v = verbose (more columns)
# └─ Output shows: packets, bytes, target, proto, in, out, source, destination

sudo iptables -S
# Shows rules as commands (can be replayed)
# Format: iptables [options]
# Better for: Understanding what will happen

APPEND RULE (To INPUT chain):

sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# ├─ -A INPUT = Append rule to INPUT chain
# ├─ -p tcp = Protocol is TCP
# ├─ --dport 22 = Destination port 22
# ├─ -j ACCEPT = Jump to ACCEPT (allow)
# └─ Result: SSH traffic ACCEPTED

RULES ARE PROCESSED IN ORDER:

First matching rule = decision made
Subsequent rules ignored

ADD RULE AT SPECIFIC POSITION:

sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
# ├─ -I = Insert (not append)
# ├─ INPUT 1 = At position 1 (first rule)
# └─ Result: New rule becomes first, others shift down

DELETE RULE:

sudo iptables -D INPUT 1
# Delete rule at position 1

sudo iptables -D INPUT -p tcp --dport 22 -j ACCEPT
# Delete rule matching this specification

FLUSH (Clear) RULES:

sudo iptables -F INPUT
# Clears all INPUT rules
# WARNING: You'll be locked out if SSH in there!

sudo iptables -F
# Clears all rules in all chains
# DANGER: Server fully open!

SET DEFAULT POLICY:

sudo iptables -P INPUT DROP
# Default: DROP (reject incoming)
# Better than full open

sudo iptables -P OUTPUT ACCEPT
# Default: ACCEPT (allow outgoing)

SAVE RULES (Persistent):

sudo iptables-save > /tmp/iptables.backup
# Backup current rules

sudo iptables-save > /etc/iptables/rules.v4
# Save system-wide (Ubuntu)
# Need iptables-persistent package first:
# sudo apt install iptables-persistent

RESTORE RULES:

sudo iptables-restore < /etc/iptables/rules.v4
# Restore on boot (if iptables-persistent installed)
```

---

## Testing Firewall Rules

### Port Scanning Your Own Server

```bash
FROM EXTERNAL MACHINE:

# Scan with nmap (shows open ports)
nmap -p 22,80,443,3306 your.vps.ip

Possible outputs:
├─ PORT     STATE  SERVICE
├─ 22/tcp   open   ssh       (firewall allows)
├─ 80/tcp   open   http      (firewall allows)
├─ 443/tcp  open   https     (firewall allows)
├─ 3306/tcp closed mysql     (firewall blocks)
└─ Filtered = responded (host up but port blocked) marked as

# Verbose scan (shows version)
nmap -sV -p 22,80,443 your.vps.ip
# Tries to identify service running on port

# UDP scan
nmap -sU -p 53,5353 your.vps.ip
# Checks UDP ports

FROM VPS ITSELF:

# Check what's actually listening
sudo ss -tlnp
# ├─ -t = TCP
# ├─ -l = Listening
# ├─ -n = Numeric ports
# └─ -p = Process name
# 
# Output shows:
# LISTEN    0      128         0.0.0.0:22        0.0.0.0:*     users:(("sshd",pid=...))
# Process "sshd" listening on port 22
#
# IMPORTANT: Service listens ≠ firewall allows
# ├─ Service listening: Application ready for connections
# └─ Firewall allows: Kernel passes traffic to application

# Verify firewall blocking
sudo ufw status
sudo iptables -L -n | grep
```

### Troubleshooting Firewall Issues

```
SYMPTOM: "Connection timeout" when connecting

Diagnosis:
  1. Is service running?
     sudo systemctl status sshd
     
  2. Is service listening on correct port?
     sudo ss -tlnp | grep sshd
     
  3. Is firewall rule in place?
     sudo ufw status | grep 22
     
  4. Can you reach from VPS itself?
     ssh localhost
     # If this works: App is fine, firewall is blocking external
     
  5. Firewall rules evaluated correctly?
     sudo iptables -L -n | grep 22
     
  6. Are you on right network/IP?
     Check: Can you ping the server?
     ping your.vps.ip (should respond)
     
Fix Process:
  ├─ Step 1: Start service
  │   sudo systemctl start sshd
  │
  ├─ Step 2: Verify listening
  │   sudo ss -tlnp | grep sshd
  │
  ├─ Step 3: Check firewall
  │   sudo ufw allow 22/tcp
  │
  ├─ Step 4: Test locally
  │   ssh localhost (should work)
  │
  ├─ Step 5: Test remotely
  │   ssh your.vps.ip (from another machine)
  │
  └─ If still blocked: Check ISP/router firewall!

SYMPTOM: "Connection refused" (different from timeout)

Meaning:
  ├─ Server is reachable
  ├─ Firewall is passing traffic
  ├─ Service is NOT listening
  └─ Application problem (not firewall)

Fix:
  ├─ Start service: sudo systemctl start sshd
  └─ Check logs: sudo journalctl -u sshd -n 20
```

---

This completes foundational firewall understanding from first principles.
