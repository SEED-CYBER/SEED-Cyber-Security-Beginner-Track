# Part 2: Firewall Configuration - Enhanced Exercises

## Exercise 2.1: Understanding Your Current Network Exposure

**Objective:** Discover what ports are currently open and what's running on them.

### Part A: Scan Your Own Server

```bash
# From ANOTHER machine on the network (not the VPS itself):
# This shows what the internet sees

nmap -sV your.vps.ip

# Output example:
# PORT     STATE SERVICE VERSION
# 22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux)
# 80/tcp   open  http    nginx 1.18.0 (Ubuntu)
# 443/tcp  open  ssl/http nginx
# 3306/tcp open  mysql   MySQL 8.0.27

ANALYSIS:
  ├─ Port 22 (SSH): Open - OK, needed for admin
  ├─ Port 80 (HTTP): Open - OK, needed for web traffic
  ├─ Port 443 (HTTPS): Open - OK, needed for secure web
  ├─ Port 3306 (MySQL): Open - PROBLEM!
  │   ├─ Database should NOT expose to internet
  │   ├─ Anyone can try to connect
  │   ├─ Could commit SQL injection attacks
  │   └─ MUST block with firewall
  └─ Unknown ports: INVESTIGATE!

# Scan specific ports
nmap -p 3306,5432,6379 your.vps.ip
# Checks PostgreSQL, Redis, other databases

# Aggressive scan (more info, slower)
nmap -A your.vps.ip
# ├─ OS detection
# ├─ Version detection
# ├─ Script scanning
# └─ Traceroute
```

### Part B: Check Listening Services on VPS

```bash
# SSH into VPS first
ssh user@your.vps.ip

# What services are listening?
sudo ss -tlnp

# Output should show:
# LISTEN  0  128  0.0.0.0:22  0.0.0.0:*  users:(("sshd",pid=123))
# ├─ sshd listening on 0.0.0.0:22
# ├─ 0.0.0.0 = all interfaces
# ├─ Port 22 = SSH
# └─ PID 123 = process running this

# More detailed
sudo netstat -tlnp | grep LISTEN

# Identify all listening ports
sudo ss -tlnp | awk '{print $4}' | grep -o ':[0-9]*' | sort -u

# What many open ports indicate
netstat -tlnp | wc -l
# If > 5: You have unused services running!
# Kill them or block them with firewall
```

**Deliverable:**
- Nmap output showing all exposed ports
- List of which are necessary
- List of which should be blocked
- Explanation of security risk for each

---

## Exercise 2.2: Firewall Setup from Scratch

**Objective:** Build a complete firewall configuration using UFW.

### Part A: Initial Lockdown

```bash
# Connect to VPS
ssh user@your.vps.ip

# First, UNDERSTAND current state
echo "=== BEFORE FIREWALL ==="
sudo ufw status
# Probably shows: "Status: inactive"

# What can reach the server right now?
nmap -A your.vps.ip 2>/dev/null | grep "open " | wc -l
# Shows how many open ports visible from internet

# CRITICAL: Before enabling firewall, ensure SSH rule exists!
# Otherwise: Enable firewall and lock yourself out!

echo "=== SETUP STARTS HERE ==="

# Step 1: Reset to clean state
sudo ufw reset
# Answer 'y' to confirm
# This clears any existing iptables rules

# Step 2: Set default policies
sudo ufw default deny incoming
# Reject all incoming by default (whitelist approach)

sudo ufw default allow outgoing
# Allow all outgoing (you control what software runs)

# Step 3: BEFORE ENABLING - Add SSH rule!
sudo ufw allow 22/tcp
# CRITICAL: Do this BEFORE 'enable'!

# Step 4: Verify rule added
sudo ufw show added
# Should show: [22/tcp] ALLOW IN Anywhere

# Step 5: Enable firewall
sudo ufw enable
# Type 'y' to confirm

# Step 6: Verify it's active
sudo ufw status verbose
# Should show:
# Status: active
# Default: deny (incoming)
# Default: allow (outgoing)
# 22/tcp ALLOW Anywhere
```

### Part B: Add Required Services

```bash
# Add service rules one by one

# Web services (if running web server)
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS

# Verify each rule as you add
sudo ufw status numbered
# Shows rules with line numbers

# Test from external machine after each rule:
# Port 22: Should be reachable (admin)
nmap -p 22 your.vps.ip
# 22/tcp open ssh

# Port 80: Should be open (if web server)
curl -I http://your.vps.ip
# Should get HTTP headers

# Port 443: Should be open (if HTTPS)
curl -I https://your.vps.ip
# Should get HTTPS headers

# Database ports: Should be BLOCKED
nmap -p 3306,5432 your.vps.ip
# Should show: filtered (blocked by firewall)
```

### Part C: Document Complete Configuration

```bash
# Create firewall configuration document
cat > /tmp/firewall_config.md << 'EOF'
# Firewall Configuration Report

## Active Rules
EOF

sudo ufw status >> /tmp/firewall_config.md

cat >> /tmp/firewall_config.md << 'EOF'

## Rule Explanation

| Port | Protocol | Action | Purpose | Justification |
|------|----------|--------|---------|---------------|
| 22 | TCP | Allow | SSH Admin Access | Required for server management |
| 80 | TCP | Allow | HTTP Web | Public web traffic (non-secure) |
| 443 | TCP | Allow | HTTPS Web | Public web traffic (secure) |
| 3306 | TCP | Deny | MySQL Database | Should not be exposed to internet |
| 5432 | TCP | Deny | PostgreSQL | Should not be exposed to internet |

## Security Posture

- Default policy: Deny incoming (whitelist approach)
- Unnecessary services: BLOCKED
- Required services: ALLOWED
- Logging: [enabled/disabled]

EOF

cat /tmp/firewall_config.md
```

**Deliverable:**
- Screenshot of `ufw status verbose` showing final config
- Successful SSH login test
- HTTP/HTTPS connectivity test
- Database port blocked verification
- Configuration document

---

## Exercise 2.3: Firewall Rule Testing and Verification

**Objective:** Test firewall rules comprehensively.

### Part A: UFW Rule Scenarios

```bash
# Test 1: SSH from different IP
# Scenario: Only allow admin from specific office IP

CURRENT_RULE="Allow 22/tcp from anywhere"

# Add more restrictive rule
sudo ufw allow from 203.0.113.100 to any port 22

# Test:
# From 203.0.113.100: Should connect
# From 8.8.8.8: Should timeout
# (In lab: can't really test, but rule is in place)

# Check rules
sudo ufw status numbered
# Should show both the permissive and restrictive rule
# SSH from office: ALLOW (first rule to match)
# SSH from anywhere else: Handled by second matching rule

# DELETE the permissive rule, keep only office IP
sudo ufw delete allow 22/tcp
# Now only IP 203.0.113.100 can SSH

# Verify
sudo ufw status
# Should show: 22/tcp from 203.0.113.100


# Test 2: Block specific attacker IP
# Scenario: Someone is attacking you

ATTACKER_IP="198.51.100.5"

# Block them
sudo ufw deny from $ATTACKER_IP
# All their traffic: DENIED

# Verify
sudo ufw status numbered | grep $ATTACKER_IP
# Should show them in deny list

# Whitelist them later (if false positive)
sudo ufw delete deny from $ATTACKER_IP


# Test 3: Allow specific service only to specific IPs
# Scenario: Database should only be accessible from web server

WEB_SERVER_IP="10.0.1.5"
DB_PORT="3306"

# Currently MySQL on 3306 is blocked

# Allow only from web server
sudo ufw allow from $WEB_SERVER_IP to any port $DB_PORT

# Verify
sudo ufw status
# Shows: 3306/tcp from 10.0.1.5 ALLOW

# Test from web server: Would succeed (if MySQL runs)
# Test from internet: Would fail (blocked)
# Test from other internal IP: Would fail (not explicitly allowed)

# Restore: Remove MySQL rule again
sudo ufw delete allow from $WEB_SERVER_IP to any port $DB_PORT
sudo ufw deny 3306/tcp
```

### Part B: Logging and Monitoring

```bash
# Enable firewall logging
sudo ufw logging on
sudo ufw logging medium
# Options: low, medium, high

# View logs in real-time
sudo tail -f /var/log/ufw.log

# Simulate traffic to see logs
# From another machine:
nmap -p 22,80,443,3306 your.vps.ip

# In log file, see:
# [UFW BLOCK] IN=eth0 OUT= MAC=... SRC=... DST=... PROTO=TCP SPT=... DPT=3306
# ├─ BLOCK = Rule denied traffic
# ├─ SRC = Source IP (attacker/tester)
# ├─ DST = Destination IP (your VPS)
# ├─ DPT (Destination Port) = 3306
# └─ Indicates: Blocked attempt to scan MySQL

# Parse logs for analysis
grep "DPT=3306" /var/log/ufw.log | wc -l
# Shows: How many attempts on MySQL port?

grep "SRC=8.8.8.8" /var/log/ufw.log
# Shows: All blocked attempts from IP 8.8.8.8

# Disable logging when done (reduces disk usage)
sudo ufw logging off
```

### Part C: Rules Conflict Detection

```bash
# Test precedence: First matching rule wins

# Create conflicting rules (on purpose to learn)

# Rule 1: Allow SSH from anywhere
sudo ufw allow 22/tcp

# Rule 2: Deny SSH from 1.2.3.4  
sudo ufw deny from 1.2.3.4 to any port 22

# Check order
sudo ufw status numbered
# If permit comes first: 1.2.3.4 IS allowed (surprising!)
#   Because first rule (allow) matched, second rule ignored
# If deny comes first: 1.2.3.4 not allowed (expected)
#   Because first rule (deny) matched, second rule ignored

# The solution: More specific rules first!
sudo ufw reset

# Right way:
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Specific rules FIRST
sudo ufw deny from 1.2.3.4 to any port 22
# Now 1.2.3.4 is denied

# General rules AFTER
sudo ufw allow 22/tcp
# Now everyone else allowed

# Result:
sudo ufw status numbered
# 1. Deny from 1.2.3.4 to port 22
# 2. Allow 22/tcp
# Correct: Attacker blocked, everyone else OK!
```

**Deliverable:**
- Document showing test cases and results
- Firewall logs showing blocked traffic
- Proof of: Port 22 accessible, port 3306 blocked
- Rule conflict analysis

---

## Exercise 2.4: Advanced Filtering Scenarios

**Objective:** Configure firewall for complex real-world scenarios.

### Scenario 1: Multi-Tier Application Architecture

```
Typical Setup:
  Internet → Load Balancer (Ports 80, 443)
           → Web Server (Ports 80, 443)
           → App Server (Port 8080 - internal only)
           → Database (Port 3306 - internal only)

Firewall Rules by Server:

LOAD BALANCER:
  sudo ufw allow 80/tcp   # HTTP from internet
  sudo ufw allow 443/tcp  # HTTPS from internet
  sudo ufw allow from app_network to any port 8080
  └─ Only talk to app servers

WEB SERVER:
  sudo ufw deny 22/tcp    # No SSH!
  sudo ufw allow from load_balancer to any port 80
  sudo ufw allow from load_balancer to any port 443
  sudo ufw allow from internal_ops to any port 22
  └─ Only receive requests from load balancer

APP SERVER:
  sudo ufw allow from web_servers to any port 8080
  sudo ufw allow from db_master to any port 3306
  sudo ufw allow from internal_ops to any port 22
  └─ Receives only from web servers, sends only to DB

DATABASE:
  sudo ufw allow from app_servers to any port 3306
  sudo ufw allow from db_replica to any port 3306
  sudo ufw allow from internal_ops to any port 22
  sudo ufw deny incoming
  └─ Only accepts connections from app servers
```

### Scenario 2: Development vs Production

```bash
# Development environment (permissive-ish)
# Admin needs to debug, test, SSH in frequently

sudo ufw reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp          # SSH
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS
sudo ufw allow 3306/tcp        # MySQL (for local testing)
sudo ufw allow 5432/tcp        # PostgreSQL

sudo ufw enable

# Result: All necessary ports open for development


# Production environment (very restrictive)
# Only what running applications need

sudo ufw reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Only SSH from admin IPs
sudo ufw allow from 203.0.113.0/24 to any port 22

# HTTP/HTTPS from anywhere (application-facing)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Databases: NEVER directly from internet
# sudo ufw deny 3306/tcp  (not needed, already denied by default)
# sudo ufw deny 5432/tcp  (not needed, already denied by default)

# SSH from other servers (in database, job queue, etc.)
sudo ufw allow from 10.0.1.0/24 to any port 22
sudo ufw allow from 10.0.2.0/24 to any port 22

sudo ufw enable

# Result: Minimal ports, only what's needed
```

### Scenario 3: DDoS Protection

```bash
# Firewall alone doesn't stop DDoS, but can help

# Rate limiting (basic)
# Note: UFW doesn't have native rate limiting, need iptables

sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/minute -j ACCEPT
# Allow max 100 connections per minute on port 80
# Connection requests above 100/min: DROPPED

# In UFW (workaround):
# Write custom rules, restart regularly

# Better solution: Use DDoS protection service
# ├─ CloudFlare (reverse proxy)
# ├─ AWS Shield (AWS built-in)
# ├─ Imperva (enterprise)
# └─ These absorb traffic before it reaches your server

# What firewall CAN do:
# ├─ Block countries/ranges (geoblocking)
# ├─ Rate-limit by IP
# ├─ Block known attack patterns
# └─ Monitor for suspicious patterns
```

**Deliverable:**
- Firewall configs for all three scenarios
- documentation of what each rule protects against
- test results showing rules work

---

## Exercise 2.5: Firewall Troubleshooting Lab

**Objective:** Diagnose firewall problems.

### Creating Problems On Purpose

```bash
# Problem 1: SSH locked out
# Scenario: Admin forgets firewall rule before enabling

sudo ufw reset
sudo ufw default deny incoming
# CRITICAL MISTAKE: No SSH rule added!

sudo ufw enable
# Now firewall is enabled, but SSH is blocked!

# Recovery (on VPS console, not SSH):
sudo ufw reset
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw enable
# Now fixed!


# Problem 2: Applications can't communicate
# Scenario: Web app can't connect to database using firewall

sudo ufw deny from 10.0.1.5 to any port 3306
# APP Server (10.0.1.5) can't access DB

# Symptom: Web pages show database connection error
# Application logs: "Connection refused to 10.0.2.5:3306"

# Diagnosis:
# 1. Can you ping database server?
ping 10.0.2.5  # If yes, network is fine
#
# 2. Is database running?
ssh 10.0.2.5
sudo ss -tlnp | grep 3306
# If running, database is fine
#
# 3. Can you connect without firewall?
sudo ufw disable
mysql -h 10.0.2.5 -u root -p
# If connects, firewall WAS the problem!
#
# 4. Fix the firewall rule
sudo ufw allow from 10.0.1.5 to any port 3306
sudo ufw enable

# Test again
mysql -h 10.0.2.5 -u root -p
# Now connects!


# Problem 3: Port appears open but service unreachable
# Scenario: Firewall allows but app not running

sudo ufw allow 8080/tcp
# Allows external connections on 8080

# But your app isn't running!
sudo systemctl status myapp
# "inactive (dead)"

nmap your.vps.ip -p 8080
# Port shows: "filtered" or "closed"
# NOT "open" (because nothing listening)

# Fix:
sudo systemctl start myapp
# Now app is listening

nmap your.vps.ip -p 8080
# Port shows: "open"


# Problem 4: Wrong IP in firewall rule
# Scenario: Whitelist IP but got it wrong

sudo ufw allow from 192.168.1.5 to any port 22
# But admin is at 192.168.1.6!

# Symptom: Admin can't SSH
# Admin tries: ssh user@server
# Result: "Connection timeout"

# Diagnosis:
# Check rule:
sudo ufw status numbered
# Shows: 22/tcp from 192.168.1.5

# Check admin's actual IP:
# On admin's machine: curl icanhazip.com
# Output: 192.168.1.6

# Fix:
sudo ufw delete allow from 192.168.1.5 to any port 22
sudo ufw allow from 192.168.1.6 to any port 22
# Or: allow both
sudo ufw allow from 192.168.1.0/24 to any port 22
```

### Diagnostic Commands Checklist

```bash
cat > /tmp/firewall_diagnosis.sh << 'EOF'
#!/bin/bash
echo "=== FIREWALL DIAGNOSIS ==="

echo "1. Firewall Status:"
sudo ufw status

echo ""
echo "2. All Rules:"
sudo ufw status numbered

echo ""
echo "3. iptables Rules (if using):"
sudo iptables -L -n -v | head -20

echo ""
echo "4. Services Listening:"
sudo ss -tlnp

echo ""
echo "5. Recent Firewall Logs:"
sudo tail -20 /var/log/ufw.log

echo ""
echo "6. Test SSH:"
ssh localhost "echo SSH Works"

echo ""
echo "7. Test HTTP (if running):"
curl -I http://localhost:80 2>/dev/null

echo ""
echo "=== END DIAGNOSIS ==="
EOF

chmod +x /tmp/firewall_diagnosis.sh
/tmp/firewall_diagnosis.sh
```

**Deliverable:**
- Document showing each problem type
- Diagnosis steps taken
- Remediation applied
- Verification that service works after fix

---

## Exercise Summary

**Key Firewall Concepts Reinforced:**

```
1. DEFAULT DENY principle
   └─ Reject all, allow only what's needed
   └─ Whitelist > Blacklist

2. UFW vs iptables
   └─ UFW for simplicity (most cases)
   └─ iptables for advanced scenarios

3. Rule precedence matters!
   └─ First matching rule decides
   └─ Specific rules before general rules

4. Test before enforcing!
   └─ Don't lock yourself out
   └─ SSH rule BEFORE enabling

5. Monitor and audit
   └─ Enable logging to see blocked traffic
   └─ Regular review of rules
   └─ Document WHY each rule exists

6. Layer thinking
   └─ Firewall = first layer of defense
   └─ Data protection = deeper layers
   └─ Defense in depth, not single solution
```

---

**Excellent work!** You've now mastered firewall configuration from first principles.
