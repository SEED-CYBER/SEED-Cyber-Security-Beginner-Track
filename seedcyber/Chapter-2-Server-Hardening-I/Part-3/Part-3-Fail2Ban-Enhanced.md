# Part 3: Fail2Ban Intrusion Prevention - Enhanced

## The Brute-Force Attack Problem (Why Fail2Ban Exists)

### Understanding SSH Attacks

**Scenario: Every second, attackers try to break into your server**

```
REAL-WORLD STATISTICS:
  ├─ Honeypot servers get 1000+ SSH attempts PER HOUR
  ├─ Attempts from automated bots (not humans)
  ├─ Most common usernames tried:
  │   ├─ root (most privileged)
  │   ├─ admin (common default)
  │   ├─ ubuntu (common for cloud servers)
  │   └─ test (default test account)
  └─ Most common passwords tried:
      ├─ 123456
      ├─ password
      ├─ admin
      ├─ letmein
      └─ qwerty

ATTACK PATTERN:
  Attacker → port 22 → SSH server
  ├─ Try: root / 123456
  ├─ Try: root / password
  ├─ Try: root / admin123
  ├─ Try: admin / qwerty
  ├─ ... (thousands of combinations)
  └─ Eventually: Give up or move to next IP/server

PROBLEM: Without protection
  ├─ Weak password? Eventually tried
  ├─ SSH key misconfiguration? Exploitable
  ├─ Server slows down processing login requests
  └─ Legitimate users blocked by attack traffic!
```

### The Solution: Fail2Ban

```
CONCEPT: Ban IPs that attack repeatedly

HOW IT WORKS:

1. Monitor logs (e.g., /var/log/auth.log)
2. Look for patterns:
   ├─ SSH: Multiple failed login attempts
   ├─ HTTP: Multiple 404 errors (probing)
   ├─ FTP: Password authentication failures
   └─ Custom: Any pattern you define

3. When threshold exceeded:
   ├─ Attacker IP blocked with firewall
   ├─ Duration: configurable (1 hour, 1 day, permanent)
   ├─ Ban is automatic (no manual intervention)
   └─ Legitimate users unaffected

4. After ban expires:
   ├─ IP can try again
   ├─ If they attack again: Blocked again!
   └─ Repeat offenders: Ban longer each time

BENEFIT:
  ├─ Stops brute-force attacks cold
  ├─ Requires almost no configuration
  ├─ Lightweight (minimal CPU/memory)
  ├─ Configurable thresholds
  └─ Works with any login system
```

---

## Fail2Ban Architecture

### How Fail2Ban Monitors

```
COMPONENTS:

1. FILTER (Pattern matching):
   ├─ Reads log file repeatedly
   ├─ Searches for failure patterns
   ├─ Examples:
   │   ├─ SSH: "Failed password for user"
   │   ├─ Apache: "404 Not Found"
   │   └─ FTP: "530 Login incorrect"
   └─ Extracts attacker IP from log line

2. JAIL (Action enforcement):
   ├─ Name: sshd, apache-auth, recidive, etc.
   ├─ Monitors: Specific service/log
   ├─ Matches: Specific failure pattern
   ├─ Action: Ban IP when threshold met
   ├─ Duration: How long to ban
   └─ Maxretry: Failures before banning

3. ACTION (Firewall integration):
   ├─ Fail2Ban calls iptables
   ├─ Inserts firewall rules
   ├─ Blocks traffic from banned IP
   ├─ Duration: Auto-expires rule
   └─ Example: iptables -A fail2ban-sshd -s 192.168.1.5 -j REJECT

4. BACKEND (Rule persistence):
   ├─ Database: sqlite (persistent bans)
   ├─ Tracks: Which IPs banned, when, for how long
   ├─ Survives: Server reboots
   └─ Useful for: "Why can't I SSH in?" diagnostics
```

### Filter Files

```
LOCATION: /etc/fail2ban/filter.d/

EXAMPLE: sshd.conf

  [Definition]
  failregex = ^<HOST> .* Failed password for .* port .* ssh2?$
              ^<HOST> .* Invalid user .* port .* ssh2?$
              ^<HOST> .* Connection closed by authenticating user .* port .*$
  
  Explanation:
  ├─ <HOST> = IP address to extract
  ├─ ^$ = Start and end of line
  ├─ .* = Any characters (wildcard)
  ├─ Multiple patterns = OR logic (any match = failure)
  └─ Example match: "192.168.1.5 ssh: Failed password for ubuntu port 54321"

WHY MULTIPLE PATTERNS?
  ├─ SSH fails in multiple ways:
  │   ├─ Invalid username (user doesn't exist)
  │   ├─ Valid username, wrong password
  │   ├─ Authentication timeout
  │   └─ Protocol errors
  └─ Each = Different log message = Different pattern needed

EXAMPLE: Apache-auth.conf

  [Definition]
  failregex = ^\<HOST\> .* ".*" .*401.*$
              ^\<HOST\> .* ".*" .*403.*$
  
  Explanation:
  ├─ 401 = Unauthorized (wrong credentials)
  ├─ 403 = Forbidden (access denied)
  └─ Multiple attempts = Attacker trying to guess password
```

### Jail Configuration

```
LOCATION: /etc/fail2ban/jail.conf and jail.d/

EXAMPLE: sshd jail configuration

  [sshd]
  enabled = true
  port = ssh
  filter = sshd
  logpath = /var/log/auth.log
  maxretry = 5
  findtime = 600
  bantime = 3600
  
EXPLANATION:

enabled = true
  └─ This jail is active

port = ssh
  ├─ Service name or number
  ├─ References protocol for detecting
  └─ Used in firewall rules

filter = sshd
  ├─ Which filter file to use
  ├─ Looks for /etc/fail2ban/filter.d/sshd.conf
  └─ Defines patterns to match

logpath = /var/log/auth.log
  ├─ Which log file to monitor
  ├─ Fail2Ban tails (reads continuously)
  ├─ Ubuntu: /var/log/auth.log
  ├─ CentOS: /var/log/secure
  └─ Custom app: /var/log/myapp.log

maxretry = 5
  ├─ Failures before banning
  ├─ Count: Failures within findtime window
  ├─ Example: 5+ failed SSH logins = ban
  └─ Lower = More aggressive (might block legitimate users!)

findtime = 600
  ├─ Time window in seconds (600 = 10 minutes)
  ├─ Failures within THIS window count
  ├─ Example:
  │   ├─ Failure at 10:00
  │   ├─ Failure at 10:05
  │   ├─ Failure at 10:09 (within 10 min, counts!)
  │   ├─ Failure at 10:11 (outside window, first failure counted!)
  │   └─ So: 4 failures in sliding window
  └─ Prevents: Attacks spread over hours/days bypassing limit

bantime = 3600
  ├─ Duration to ban in seconds (3600 = 1 hour)
  ├─ After 1 hour: IP automatically unbanned
  ├─ If attack repeats: Banned again
  ├─ Longer ban = More annoying for user (but safer)
  └─ Example penalties:
      ├─ First offense: 1 hour
      ├─ Second offense: 1 day
      ├─ Third offense: 1 week
      └─ Uses: recidive jail
```

---

## Fail2Ban Setup and Configuration

### Installation and Startup

```bash
STEP 1: Install
sudo apt update
sudo apt install fail2ban

STEP 2: Start service
sudo systemctl start fail2ban

STEP 3: Enable on boot
sudo systemctl enable fail2ban

STEP 4: Check status
sudo systemctl status fail2ban
# Should show: active (running)

STEP 5: Verify it's working
sudo fail2ban-client status
# Shows: 1 jail, 0 bans currently

STEP 6: After some time, check active bans
sudo fail2ban-client status sshd
# Shows:
# ├─ Total banned: 5
# ├─ Banned IP list: [192.168.1.5, 10.0.0.2, ...]
# └─ Currently banned: N
```

### Basic Configuration

```bash
# Main config directory
ls -la /etc/fail2ban/

# Key files:
# ├─ fail2ban.conf = global settings
# ├─ jail.conf = jail defaults + jail definitions
# ├─ jail.d/ = override specific jails
# ├─ filter.d/ = pattern definitions
# └─ action.d/ = ban/unban scripts

# Copy default jail config (never edit original!)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit local version
sudo nano /etc/fail2ban/jail.local

RECOMMENDED SETTINGS IN [DEFAULT]:

[DEFAULT]
bantime  = 3600
# Ban for 1 hour initially

findtime  = 600
# Check failures in last 10 minutes

maxretry  = 5
# Ban after 5 failures

destemail = your-email@example.com
# Email for alerts (if configured)

sendername = Fail2Ban
# Name in alert emails

RECOMMENDED SETTINGS FOR [sshd]:

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
# SSH: Be strict (only 3 attempts)
# Legitimate users won't fail 3+ times
# Attackers always will

findtime = 300
# 5 minute window (faster detection)

bantime = 86400
# Ban for 1 day (SSH attacks are serious!)
```

### Reload Configuration

```bash
# After editing jail.local:

sudo systemctl restart fail2ban
# OR

sudo fail2ban-client reload
# Reloads without stopping service

# Verify changes
sudo fail2ban-client status
# Should show updated jails/settings
```

---

## Monitoring and Troubleshooting

### Checking Ban Status

```bash
# See all jails
sudo fail2ban-client status

# Check specific jail
sudo fail2ban-client status sshd
# Shows:
# ├─ Filter: sshd[ssh]
# ├─ Currently failed: 2
# ├─ Currently banned: 1
# └─ Total banned: 5

# See currently banned IPs
sudo fail2ban-client set sshd banip

# See failed attempts (not yet banned)
sudo fail2ban-client get sshd logpath

# Get ban for specific IP
sudo fail2ban-client get sshd actionips
```

### Viewing Logs

```bash
# Fail2Ban own logs
sudo tail -f /var/log/fail2ban.log

# Example output:
# 2024-02-20 10:15:32,456 fail2ban.filter [1234]: INFO [sshd] Found 192.168.1.5
# 2024-02-20 10:15:45,123 fail2ban.filter [1234]: INFO [sshd] Found 192.168.1.5
# 2024-02-20 10:15:47,892 fail2ban.filter [1234]: INFO [sshd] Found 192.168.1.5
# 2024-02-20 10:15:49,231 fail2ban.actions [1234]: NOTICE [sshd] Ban 192.168.1.5
# ├─ Found = Matched failure pattern
# ├─ Ban = Applied firewall rule
# └─ All from same IP within short time

# System logs (server side SSH attempts)
sudo tail -f /var/log/auth.log | grep "Failed password"

# Example output:
# Feb 20 10:15:32 server sshd[1234]: Failed password for invalid user admin from 192.168.1.5 port 54321 ssh2
# Feb 20 10:15:33 server sshd[1235]: Failed password for ubuntu from 192.168.1.5 port 54322 ssh2
# Feb 20 10:15:35 server sshd[1236]: Failed password for ubuntu from 192.168.1.5 port 54323 ssh2

# Filter logs to time period
sudo journalctl -u fail2ban --since "10 minutes ago"

# See iptables rules Fail2Ban created
sudo iptables -L fail2ban-sshd -n

# Example output:
# Chain fail2ban-sshd (1 references)
# target  prot opt source         destination
# REJECT  all  --  192.168.1.5    0.0.0.0/0
# REJECT  all  --  10.0.0.2       0.0.0.0/0
# RETURN  all  --  0.0.0.0/0      0.0.0.0/0
# ├─ REJECT = Actively reject traffic
# └─ RETURN = Check normal firewall rules
```

### Manual IP Management

```bash
# Unban specific IP (if user locked out)
sudo fail2ban-client set sshd unbanip 192.168.1.5

# Verify unbanned
sudo fail2ban-client set sshd banip
# Should not show that IP anymore

# Ban specific IP manually
sudo fail2ban-client set sshd banip 8.8.8.8
# Useful for: Blocking known attackers

# Set jail to inactive temporarily
sudo fail2ban-client set sshd inactive

# Reactivate
sudo fail2ban-client set sshd active

# Check jail state
sudo fail2ban-client get sshd active
# Shows: true / false
```

---

## Advanced Fail2Ban Scenarios

### Recidivism: Repeat Offender Detection

```
PROBLEM: Attacker unbanned after 1 hour, attacks again

SOLUTION: recidive jail

[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log

# Bans IPs that are banned multiple times
# Second offense: Longer ban
# Third offense: Even longer ban

EXAMPLE:
  ├─ Attacker IP 192.168.1.5 attacks SSH
  ├─ Fail2Ban bans for 1 hour
  ├─ After 1 hour: Automatically unbanned
  ├─ Attacker tries again immediately
  ├─ Banned again (sshd jail)
  ├─ recidive jail detects repeat
  ├─ Bans for 1 WEEK this time!
  └─ Attacker learns: "This server is protected"

Configuration:
  maxretry = 2
  findtime = 86400  (24 hours)
  bantime = 604800  (1 week)
  
  Meaning: Ban someone for 1 week if they're banned 2+ times in 24 hours
```

### Custom Jails for Applications

```
SCENARIO: Nginx showing attacks on /admin endpoint

# Create filter
sudo cat > /etc/fail2ban/filter.d/nginx-noscript.conf << 'EOF'
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*/admin.*" 40[13]
ignoreregex =
EOF

# Create jail
sudo cat > /etc/fail2ban/jail.d/nginx-noscript.conf << 'EOF'
[nginx-noscript]
enabled = true
port = http,https
filter = nginx-noscript
logpath = /var/log/nginx/access.log
maxretry = 5
bantime = 3600
EOF

# Reload
sudo systemctl restart fail2ban

RESULT:
  ├─ Anyone probing /admin endpoint
  ├─ 5+ attempts = banned
  ├─ Stops automated vulnerability scanners
  └─ Protects application
```

### Email Alerts (Optional)

```
SETUP: Get notified when Fail2Ban bans someone

STEP 1: Install mail utility
sudo apt install mailutils

STEP 2: Edit jail.local [DEFAULT] section
destemail = admin@example.com
sendername = Fail2Ban
mta = sendmail

STEP 3: Add email action
[sshd]
enabled = true
...
action = %(action_mwl)s
# This adds email + whois + log action

RESULT:
  └─ Email when ban applied:
     ├─ IP address banned
     ├─ Service (sshd)
     ├─ Number of failures
     ├─ WHOIS information (who owns IP)
     └─ Recent logs (what they did)
```

---

## Best Practices and Limits

### Avoiding False Positives

```
PROBLEM: Legitimate users get banned!

SCENARIO:
  ├─ User forgets password, tries 5 times
  ├─ Gets banned!
  ├─ Now can't access server
  ├─ Admins get angry calls
  └─ Not a security win if users blocked!

PREVENTION:

1. GENEROUS maxretry for some services:
   [sshd]
   maxretry = 10  # Users get 10 tries, admins have time

2. LONGER findtime for user-facing services:
   [apache-auth]
   findtime = 3600  # 1 hour window
   # User forgetting password is rare
   # Will likely remember within 1 hour

3. WHITELIST trusted IPs:
   [sshd]
   ignoreip = 127.0.0.1/8 192.168.1.0/24 203.0.113.100
   # IPs in ignore list never banned
   # Use for: Office, VPN, known services

4. RATE LIMITING instead of banning:
   # Some apps better with rate limit than ban
   # Example: API endpoints (allow requests, just slow them down)
   # Fail2Ban: Prefers hard bans

5. MONITORING:
   # Regular check:
   sudo fail2ban-client status
   # Are legitimate IPs getting banned?
   # Adjust rules if needed
```

### Performance Considerations

```
CPU USAGE:
  ├─ Low: Fail2Ban typically < 1% CPU
  ├─ Scales: linearly with number of jails
  ├─ Log checking: Every few seconds
  └─ Iptables updates: Almost instant

MEMORY USAGE:
  ├─ Small: ~50-100 MB typical
  ├─ Varies: by number of jails + banned IPs
  ├─ Tracking: 10,000+ banned IPs manageable
  └─ Database: SQLite3 (lightweight)

DISK USAGE:
  ├─ Fail2Ban logs: Fairly small
  ├─ Fail2Ban DB: Grows over time
  ├─ Cleanup: sudo fail2ban-client purge
  │   ├─ Removes very old ban records
  │   ├─ Keeps current bans
  │   └─ Run monthly
  └─ Log rotation: Let logrotate handle /var/log/

RELIABILITY:
  ├─ Service survives reboots (enabled via systemctl)
  ├─ Bans survive reboots (SQLite persistence)
  ├─ Handles log rotation (Fail2Ban aware)
  ├─ Safe to run 24/7 (battle-tested)
  └─ CPU/memory: Leaves plenty for applications
```

---

This completes Fail2Ban fundamentals from first principles.
