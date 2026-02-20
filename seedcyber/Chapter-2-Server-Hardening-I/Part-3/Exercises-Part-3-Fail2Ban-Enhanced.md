# Part 3: Fail2Ban Intrusion Prevention - Enhanced Exercises

## Exercise 3.1: Installation and Baseline Configuration

**Objective:** Install Fail2Ban and set up basic SSH protection.

### Part A: Installation and Verification

```bash
# Step 1: Install Fail2Ban
sudo apt update
sudo apt install fail2ban -y

# Step 2: Verify installation
which fail2ban-server
# Output: /usr/bin/fail2ban-server
# Confirms: Binary installed

fail2ban-server -V
# Output: Fail2Ban v0.11.2
# Shows: Version number

# Step 3: Start service
sudo systemctl start fail2ban

# Step 4: Enable on boot
sudo systemctl enable fail2ban

# Step 5: Check status
sudo systemctl status fail2ban
# Should show: active (running)

# Step 6: Check currently active jails
sudo fail2ban-client status
# Output:
# ├─ Status: running
# ├─ Number of jail: 2
# │   ├─ sshd (if default jail is enabled)
# │   └─ recidive
# └─ Jail list: sshd, recidive

# Step 7: Check specific jail
sudo fail2ban-client status sshd
# Shows:
# ├─ Filter: sshd[ssh] - enabled
# ├─ Currently failed: 0
# ├─ Currently banned: 0
# ├─ Total banned: 0
# └─ IP list: []
# (Empty = no attacks yet)
```

### Part B: Understand Default Configuration

```bash
# View default jail configuration
sudo cat /etc/fail2ban/jail.conf | grep -A 10 "^\[sshd\]"

# Output shows defaults:
# [sshd]
# enabled = false        (disabled!"
# port = ssh
# logpath = /var/log/auth.log

# Create local override (never edit jail.conf!)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# View defaults section
sudo grep -A 20 "^\[DEFAULT\]" /etc/fail2ban/jail.local | head -15

# Shows defaults like:
# bantime = 10m
# findtime = 10m
# maxretry = 5
```

**Deliverable:**
- Screenshot of `sudo fail2ban-client status` showing running
- SSH jail status showing 0 bans
- `systemctl status fail2ban` showing active

---

## Exercise 3.2: Create Custom SSH Protection Configuration

**Objective:** Configure stricter SSH protection based on security principles.

### Part A: Create Regional Jail Configuration

```bash
# Create override file (safer than editing jail.local)
sudo nano /etc/fail2ban/jail.d/sshd.local

# Add aggressive SSH protection:
EOF
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log

# SSH is critical: Be aggressive
maxretry = 3          # Only 3 failed attempts!
findtime = 600        # In 10-minute window
bantime = 3600        # Ban for 1 hour

# For development: Make whitelist list
ignoreip = 127.0.0.1/8 ::1

# For production: Add your IP
# ignoreip = 127.0.0.1/8 203.0.113.100 10.0.0.0/8
EOF

# Similarly, create recidive jail (repeat offenders)
sudo nano /etc/fail2ban/jail.d/recidive.local

EOF
[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log

# Person banned 2+ times? Longer ban
maxretry = 2
findtime = 604800     # 1 week window
bantime = 604800      # Ban for 1 week!
EOF

# Reload Fail2Ban with new config
sudo systemctl restart fail2ban

# Verify new config loaded
sudo fail2ban-client status
# Should now show: 2 jails (sshd, recidive)

sudo fail2ban-client status sshd
# Should show new settings applied
```

### Part B: Verify Configuration Applied

```bash
# Get specific jail setting
sudo fail2ban-client get sshd maxretry
# Output: 3 (our setting)

sudo fail2ban-client get sshd findtime
# Output: 600 (our setting)

sudo fail2ban-client get sshd bantime
# Output: 3600 (our setting)

# View whitelist (ignoreip)
sudo fail2ban-client get sshd ignoreip
# Output: 127.0.0.1/8 ::1

# Get all sshd settings
sudo fail2ban-client get sshd actions
# Shows: iptables-multiport (firewall blocker)
```

**Deliverable:**
- Copy of /etc/fail2ban/jail.d/sshd.local
- Copy of /etc/fail2ban/jail.d/recidive.local
- Output of jail configuration verification showing all settings applied correctly

---

## Exercise 3.3: Simulate and Detect SSH Attacks

**Objective:** Test Fail2Ban by simulating failed SSH attempts.

### Part A: Monitor Active Logs

```bash
# Terminal 1: Watch Fail2Ban logs in real-time
sudo tail -f /var/log/fail2ban.log

# Terminal 2: Watch auth logs in real-time
sudo tail -f /var/log/auth.log | grep -i "failed\|invalid"

# Terminal 3: Simulate attacks (from this terminal)
```

### Part B: Generate Failed Attempts

```bash
# Open new terminal or use &

# Method 1: Wrong password attempts (safe)
for i in {1..5}; do
  sshpass -p "wrongpassword" ssh user@localhost -o StrictHostKeyChecking=no 2>/dev/null
  echo "Attempt $i"
done

# OR: Manual attempts (you type wrong password)
# Open another terminal and run:
ssh user@localhost
# (type wrong password, press Ctrl+C after rejection)
# Do this 3-4 times in quick succession

# OR: Use expect script (programmatic)
expect << 'EOF'
spawn ssh user@localhost
expect "password:"
send "wrong1\r"
expect "password:"
send "wrong2\r"
expect "password:"
send "wrong3\r"
expect "password:"
send "wrong4\r"
expect "password:"
send "wrong5\r"
EOF
```

### Part C: Observe Fail2Ban Response

```bash
# Watch logs in real-time (already running)
# Output example:

# /var/log/auth.log shows:
# Feb 20 10:15:30 server sshd[1234]: Failed password for user from 127.0.0.1 port 54321 ssh2
# Feb 20 10:15:31 server sshd[1235]: Failed password for user from 127.0.0.1 port 54322 ssh2
# Feb 20 10:15:32 server sshd[1236]: Failed password for user from 127.0.0.1 port 54323 ssh2
# Feb 20 10:15:33 server sshd[1237]: Invalid user from 127.0.0.1 port 54324 ssh2

# /var/log/fail2ban.log shows:
# 2024-02-20 10:15:30,123 fail2ban.filter [pid]: INFO [sshd] Found 127.0.0.1
# 2024-02-20 10:15:31,456 fail2ban.filter [pid]: INFO [sshd] Found 127.0.0.1
# 2024-02-20 10:15:32,789 fail2ban.filter [pid]: INFO [sshd] Found 127.0.0.1
# 2024-02-20 10:15:33,321 fail2ban.actions [pid]: NOTICE [sshd] Ban 127.0.0.1

# Key observation:
# ├─ After 3 failures (our maxretry setting)
# ├─ IP 127.0.0.1 is banned!
# └─ Next SSH attempt will be rejected

# Verify ban is active
sudo fail2ban-client status sshd
# Output:
# ├─ Filter: sshd[ssh] enabled
# ├─ Currently failed: 0  (reset after ban)
# ├─ Currently banned: 1
# ├─ Total banned: 1
# └─ IP list: ['127.0.0.1']
```

### Part D: Test Ban Effectiveness

```bash
# Try to SSH from banned IP
ssh user@localhost
# Output: Connection refused or connection timeout
# (Not even prompt for password - banned at firewall level!)

# Check what iptables rule was added:
sudo iptables -L fail2ban-sshd -n
# Output:
# Chain fail2ban-sshd (1 references)
# target   prot opt source      destination
# REJECT   all  --  127.0.0.1   0.0.0.0/0     (our banned IP)
# RETURN   all  --  0.0.0.0/0   0.0.0.0/0     (default return)

# Explanation:
# ├─ Any traffic from 127.0.0.1 → REJECT
# ├─ All other traffic → RETURN (check other rules)
# └─ Ban is enforced at kernel level!

# After 1 hour (bantime = 3600 seconds):
# Rule automatically removed
# IP automatically unbanned
# No manual intervention needed!
```

**Deliverable:**
- Screenshots of both /var/log/fail2ban.log and /var/log/auth.log showing attack and blocking
- Evidence that IP was banned (`fail2ban-client status sshd` showing banned count)
- iptables rule screenshot showing REJECT rule for banned IP
- Proof that SSH connection was refused after ban

---

## Exercise 3.4: Whitelist and Exception Management

**Objective:** Manage who is protected from bans.

### Part A: Add Trusted IPs to Whitelist

```bash
# Scenario: Your office IP should never be banned

# Edit sshd jail config
sudo nano /etc/fail2ban/jail.d/sshd.local

# Find ignoreip line and add your IPs:
ignoreip = 127.0.0.1/8 ::1 203.0.113.100 192.168.0.0/24
# ├─ 127.0.0.1/8 = localhost (always!)
# ├─ ::1 = IPv6 localhost
# ├─ 203.0.113.100 = Your office IP
# └─ 192.168.0.0/24 = Your office network

# Reload
sudo systemctl reload fail2ban

# Verify change
sudo fail2ban-client get sshd ignoreip
# Should show all IPs in list
```

### Part B: Manual Ban/Unban

```bash
# Scenario: User locked out, needs immediate access

# Check current bans
sudo fail2ban-client status sshd
# Output shows: 127.0.0.1 is banned

# Unban specific IP
sudo fail2ban-client set sshd unbanip 127.0.0.1

# Verify unbanned
sudo fail2ban-client status sshd
# Output shows: Currently banned: 0 (empty list)

# User can now SSH!
ssh user@localhost
# Should get prompt for password now

# Scenario 2: Know attacker - ban them immediately

# Ban attacker IP
sudo fail2ban-client set sshd banip 192.0.2.5

# Verify banned
sudo fail2ban-client status sshd
# 192.0.2.5 in ban list (even though <3 failures)

# Won't get unbanned until bantime expires!
# (or manual unban)
```

### Part C: Create Whitelist Override File

```bash
# For permanent whitelists, create override:
sudo nano /etc/fail2ban/jail.local

# In [DEFAULT] section, add:
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 203.0.113.0/24 10.0.0.0/8
# Office network (203.0.113.0/24)
# Internal network (10.0.0.0/8)

# Reload
sudo systemctl reload fail2ban

# Check if applied to all jails
sudo fail2ban-client get sshd ignoreip
sudo fail2ban-client get apache-auth ignoreip
# Should be same (inherited from [DEFAULT])
```

**Deliverable:**
- /etc/fail2ban/jail.d/sshd.local showing whitelist configuration
- Evidence of trusting an IP and then confirming failed attempts don't ban it
- Manual ban/unban process showing commands and verification

---

## Exercise 3.5: Advanced Jail Creation

**Objective:** Create custom jail to protect application-specific attacks.

### Scenario: Web Application Bruteforce Protection

```bash
# Imagine: Your app's /api/login endpoint is attacked

# Step 1: Identify log format
# Check web server logs:
sudo tail -20 /var/log/nginx/access.log

# Example output:
# 192.168.1.5 - - [20/Feb/2024:10:15:30 +0000] "POST /api/login HTTP/1.1" 401 123
# 192.168.1.5 - - [20/Feb/2024:10:15:31 +0000] "POST /api/login HTTP/1.1" 401 123
# 192.168.1.5 - - [20/Feb/2024:10:15:32 +0000] "POST /api/login HTTP/1.1" 401 123

# Key: 401 = Unauthorized (failed login attempt)

# Step 2: Create filter for login bruteforce
sudo cat > /etc/fail2ban/filter.d/app-bruteforce.conf << 'EOF'
[Definition]
failregex = ^<HOST> .* "POST /api/login .* 401
ignoreregex =
EOF

# Step 3: Create jail for this filter
sudo cat > /etc/fail2ban/jail.d/app-bruteforce.conf << 'EOF'
[app-bruteforce]
enabled = true
port = http,https
filter = app-bruteforce
logpath = /var/log/nginx/access.log
maxretry = 10
findtime = 300        # 5 minute window
bantime = 600         # Ban for 10 minutes
EOF

# Step 4: Reload Fail2Ban
sudo systemctl reload fail2ban

# Step 5: Verify jail created
sudo fail2ban-client status
# Should show: app-bruteforce in jail list

sudo fail2ban-client status app-bruteforce
# Should show active and monitoring

# Step 6: Test it
# Simulate 10+ failed logins on /api/login endpoint
for i in {1..15}; do
  curl -X POST http://localhost/api/login -d "user=test&pass=wrong" -s | grep -q "401"
  echo "Login attempt $i"
  sleep 1
done

# After 10 attempts:
sudo fail2ban-client status app-bruteforce
# Should show: 127.0.0.1 banned!

# Next login attempt will be rejected at firewall
```

**Deliverable:**
- /etc/fail2ban/filter.d/app-bruteforce.conf (custom filter)
- /etc/fail2ban/jail.d/app-bruteforce.conf (custom jail)
- Logs showing attack detection and ban
- Test results showing application requests blocked after ban

---

## Exercise 3.6: Monitoring and Maintenance

**Objective:** Set up ongoing monitoring for Fail2Ban health.

### Part A: Status Monitoring Script

```bash
# Create monitoring script
cat > /tmp/fail2ban-monitor.sh << 'EOF'
#!/bin/bash
echo "=== FAIL2BAN STATUS REPORT ==="
date

echo ""
echo "Service Status:"
sudo systemctl status fail2ban | grep "Active:"

echo ""
echo "Overall Fail2Ban Status:"
sudo fail2ban-client status

echo ""
echo "Per-Jail Details:"
for jail in $(sudo fail2ban-client status | grep "Jail list:" | sed 's/.*\[\(.*\)\]/\1/' | tr ',' ' '); do
  echo "--- Jail: $jail ---"
  sudo fail2ban-client status "$jail" | grep -E "^Status:|Currently|Total|IP list"
done

echo ""
echo "Current iptables Blocks:"
sudo iptables -L | grep "fail2ban" -A 2

echo ""
echo "Recent Fail2Ban Log Activity:"
sudo tail -10 /var/log/fail2ban.log

echo ""
echo "=== REPORT END ==="
EOF

chmod +x /tmp/fail2ban-monitor.sh

# Run monitoring report
/tmp/fail2ban-monitor.sh
```

### Part B: Unban Script (Automated Recovery)

```bash
# Sometimes IP gets wrongly banned (false positive)
# Create recovery script

cat > /tmp/fail2ban-unban.sh << 'EOF'
#!/bin/bash

if [ -z "$1" ]; then
  echo "Usage: $0 <IP_ADDRESS>"
  echo "Example: $0 203.0.113.100"
  exit 1
fi

IP="$1"

echo "Unbanning IP: $IP"

# Check which jails have this IP banned
sudo fail2ban-client status | grep Jail | grep -oP '\[.*?\]' | tr -d '[]' | while read jail; do
  if sudo fail2ban-client status "$jail" | grep -q "$IP"; then
    echo "  Unbanning from jail: $jail"
    sudo fail2ban-client set "$jail" unbanip "$IP"
  fi
done

echo "Done! IP $IP unbanned"
echo "Verifying..."
sudo fail2ban-client status sshd
EOF

chmod +x /tmp/fail2ban-unban.sh

# Example usage:
/tmp/fail2ban-unban.sh 127.0.0.1
```

### Part C: Maintenance Tasks

```bash
# 1. Clean old ban records (monthly)
sudo fail2ban-client purge
# Removes old ban records from database
# Keeps current active bans

# 2. Check disk space (monthly)
du -sh /var/log/fail2ban.log
du -sh /var/lib/fail2ban/

# Archive old logs
sudo gzip /var/log/fail2ban.log

# 3. Review and adjust settings (quarterly)
# If too many false positives: Increase maxretry
sudo nano /etc/fail2ban/jail.d/sshd.local
# maxretry = 5  (was 3, too aggressive!)

# If attacks not blocked: Decrease bantime
# bantime = 86400  (1 day, attackers just wait)
# Change to: bantime = 3600  (1 hour, force them to change IP)

sudo systemctl reload fail2ban

# 4. Generate audit report
cat > /tmp/fail2ban-audit.sh << 'EOF'
#!/bin/bash
echo "=== FAIL2BAN AUDIT REPORT ==="
echo "Date: $(date)"
echo ""

echo "1. Configuration Summary:"
echo "  SSH maxretry: $(sudo fail2ban-client get sshd maxretry)"
echo "  SSH findtime: $(sudo fail2ban-client get sshd findtime)"
echo "  SSH bantime: $(sudo fail2ban-client get sshd bantime)"

echo ""
echo "2. Ban Statistics (lifetime):"
echo "  Total bans (sshd jail):"
grep "Ban " /var/log/fail2ban.log | wc -l

echo ""
echo "3. Most Banned IPs (all-time):"
grep "Ban " /var/log/fail2ban.log | awk '{print $(NF-1)}' | sort | uniq -c | sort -rn | head -5

echo ""
echo "4. Attacks by Hour (last 24 hours):"
tail -10000 /var/log/fail2ban.log | grep "Found" | cut -d' ' -f1 | sort | uniq -c

echo ""
echo "5. Current Load:"
echo "  Active bans: $(sudo fail2ban-client status sshd | grep "Currently banned" | awk '{print $(NF-1)}')"
echo "  Failed attempts being tracked: $(sudo fail2ban-client status sshd | grep "Currently failed" | awk '{print $(NF-1)}')"
EOF

chmod +x /tmp/fail2ban-audit.sh
/tmp/fail2ban-audit.sh
```

**Deliverable:**
- Output of `/tmp/fail2ban-monitor.sh`
- Documentation of maintenance schedule (monthly/quarterly tasks)
- Audit report showing statistics
- Copy of monitoring and maintenance scripts

---

## Exercise Summary

```
KEY CONCEPTS REINFORCED:

1. FAIL2BAN PROTECTION LAYERS:
   ├─ Filter: Pattern matching in logs
   ├─ Jail: Thresholds and enforcement
   ├─ Action: Firewall integration
   └─ Backend: Persistent storage

2. CONFIGURATION HIERARCHY:
   ├─ /etc/fail2ban/jail.conf (defaults - don't edit)
   ├─ /etc/fail2ban/jail.local (local overrides)
   ├─ /etc/fail2ban/jail.d/*.conf (per-jail overrides)
   └─ Most specific = wins

3. TUNING PARAMETERS:
   ├─ maxretry: Balance between usability and security
   ├─ findtime: How long to track failures
   ├─ bantime: How long to punish
   └─ ignoreip: Whitelist trusted sources

4. OPERATIONAL TASKS:
   ├─ Monitor: Regularly check status
   ├─ Maintain: Clean old records, review logs
   ├─ Adjust: Tune parameters based on data
   └─ Document: Track why each rule exists

5. FAILURE MODES:
   ├─ Too aggressive: Blocks legitimate users
   ├─ Too lenient: Allows brute-force attacks
   ├─ Wrong IP: Whitelisting fixes it quickly
   └─ Wrong service: Custom jails handle it
```

---

**Excellent!** You've mastered Fail2Ban from installation through advanced customization and maintenance.
