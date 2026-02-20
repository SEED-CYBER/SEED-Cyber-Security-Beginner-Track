# Part 1: SSH Server Hardening - Enhanced Exercises

## Exercise 1.1: Audit Current SSH Configuration

**Objective:** Assess the security level of current SSH setup.

### Part A: Extract Current Configuration

```bash
# Get actual SSH daemon configuration (including defaults)
sudo sshd -T > /tmp/sshd_current_config.txt

# View critical security settings
echo "=== SSH DAEMON CURRENT CONFIGURATION ==="
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|permitempptypasswords|maxauthtries|x11forwarding|port|listenaddress"

# Count total settings
echo ""
echo "Total settings: $(sudo sshd -T | wc -l)"

# Get version
ssh -V

# Check if defaults are insecure
echo ""
echo "=== SECURITY ASSESSMENT ==="
echo "If you see these values, the server is INSECURE:"
echo "├─ permitrootlogin yes (or without-password)"
echo "├─ passwordauthentication yes"
echo "├─ permitemptypasswords yes"
echo "└─ maxauthtries > 10"
```

### Part B: Analyze Security Posture

```bash
# Create security scoring script
cat > /tmp/assess_ssh_security.sh << 'EOF'
#!/bin/bash

echo "=== SSH SECURITY AUDIT REPORT ==="
echo "Date: $(date)"
echo ""

SCORE=0
MAX_SCORE=8

# Check 1: Root login
if sudo sshd -T | grep "permitrootlogin no" > /dev/null; then
    echo "[✓] Root login disabled"
    ((SCORE++))
else
    echo "[✗] Root login enabled (INSECURE)"
fi
((MAX_SCORE++))

# Check 2: Password authentication
if sudo sshd -T | grep "passwordauthentication no" > /dev/null; then
    echo "[✓] Password authentication disabled"
    ((SCORE++))
else
    echo "[✗] Password authentication enabled (VULNERABLE to brute force)"
fi
((MAX_SCORE++))

# Check 3: Public key authentication
if sudo sshd -T | grep "pubkeyauthentication yes" > /dev/null; then
    echo "[✓] Public key authentication enabled"
    ((SCORE++))
else
    echo "[✗] Public key authentication disabled"
fi
((MAX_SCORE++))

# Check 4: Empty passwords
if sudo sshd -T | grep "permitemptypasswords no" > /dev/null; then
    echo "[✓] Empty passwords disabled"
    ((SCORE++))
else
    echo "[✗] Empty passwords allowed (DANGEROUS)"
fi
((MAX_SCORE++))

# Check 5: Max auth tries
TRIES=$(sudo sshd -T | grep "maxauthtries" | awk '{print $NF}')
if [ "$TRIES" -le 5 ]; then
    echo "[✓] Max auth tries limited ($TRIES)"
    ((SCORE++))
else
    echo "[✗] Max auth tries too high ($TRIES)"
fi
((MAX_SCORE++))

# Check 6: X11 Forwarding
if sudo sshd -T | grep "x11forwarding no" > /dev/null; then
    echo "[✓] X11 forwarding disabled"
    ((SCORE++))
else
    echo "[✗] X11 forwarding enabled (reduces attack surface)"
fi
((MAX_SCORE++))

# Check 7: Protocol version
if sudo sshd -T | grep "protocol 2" > /dev/null; then
    echo "[✓] Protocol version 2 only"
    ((SCORE++))
else
    echo "[✗] Protocol version 1 or mixed (SSH1 is insecure)"
fi
((MAX_SCORE++))

# Check 8: Ciphers
CIPHER=$(sudo sshd -T | grep "ciphers" | head -1)
if echo "$CIPHER" | grep -i "aes\|chacha" > /dev/null; then
    echo "[✓] Strong ciphers configured"
    ((SCORE++))
else
    echo "[✗] Weak or default ciphers"
fi
((MAX_SCORE++))

echo ""
echo "SECURITY SCORE: $SCORE / $MAX_SCORE"

if [ $SCORE -eq $MAX_SCORE ]; then
    echo "STATUS: HARDENED ✓"
elif [ $SCORE -ge 6 ]; then
    echo "STATUS: GOOD (minor improvements possible)"
elif [ $SCORE -ge 4 ]; then
    echo "STATUS: MODERATE (needs work)"
else
    echo "STATUS: INSECURE! (immediate hardening required)"
fi
EOF

chmod +x /tmp/assess_ssh_security.sh
/tmp/assess_ssh_security.sh
```

**Deliverable:**
- Current SSH configuration audit
- Security score assessment
- List of items needing hardening

---

## Exercise 1.2: Harden SSH Configuration

**Objective:** Apply comprehensive SSH hardening to a test system.

### Part A: Create Hardened Configuration

```bash
# CRITICAL: Keep existing SSH connection open!
# This is your lifeline if something goes wrong!

# Create backup of original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
echo "Backed up original to /etc/ssh/sshd_config.backup"

# Create hardened version
cat > /tmp/sshd_config_hardened << 'EOF'
# Hardened SSH Configuration
# Based on security best practices

# Port and listening address
Port 22
ListenAddress 0.0.0.0
ListenAddress ::

# Protocol
Protocol 2

# Authentication
PermitRootLogin no
# Users must use private keys, not passwords:
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no

# Authorization
MaxAuthTries 3
MaxSessions 10

# Security filters
X11Forwarding no
PermitUserEnvironment no
IgnoreRhosts yes
HostbasedAuthentication no

# Cryptography (only strong ciphers)
Ciphers aes128-ctr,aes192-ctr,aes256-ctr,chacha20-poly1305@openssh.com
MACs hmac-sha2-256,hmac-sha2-512,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519

# Timeouts and keep-alive
ClientAliveInterval 60
ClientAliveCountMax 3

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Banner
Banner /etc/ssh/banner.txt

# Subsystem
Subsystem sftp /usr/lib/openssh/sftp-server

# Accept locale settings
AcceptEnv LANG LC_*
EOF

# Review changes before applying
echo "=== COMPARING CONFIGURATIONS ==="
diff /etc/ssh/sshd_config /tmp/sshd_config.hardened | head -30

# Show key differences
echo ""
echo "=== KEY HARDENING CHANGES ==="
echo "Root login: NO"
echo "Password auth: NO"
echo "Key auth: YES"
echo "Max auth tries: 3"
echo "Max sessions: 10"
echo "X11 forwarding: NO"
echo "Only strong ciphers/MACs/keys"
```

### Part B: Test Configuration Syntax

```bash
# CRITICAL: Do NOT apply changes without testing first!

echo "=== SYNTAX VALIDATION ==="

# Copy hardened config to temp location
sudo cp /tmp/sshd_config_hardened /tmp/sshd_config_test

# Test syntax
sudo /usr/sbin/sshd -t -f /tmp/sshd_config_test

# Output: (nothing = syntax OK)
# If error: Shows what's wrong (DON'T APPLY!)

# Extract and show key settings
echo ""
echo "=== VERIFIED HARDENED SETTINGS ==="
sudo /usr/sbin/sshd -T -f /tmp/sshd_config_test | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries|x11forwarding"
```

### Part C: Staged Application (Safe)

```bash
# SAFE PROCEDURE:

# Step 1: Create a comment in original about hardening
# Step 2: In NEW TERMINAL, establish connection
ssh admin@server

# Step 3: In NEW connection, apply changes
sudo cp /tmp/sshd_config_hardened /etc/ssh/sshd_config

# Step 4: Restart SSH
sudo systemctl restart sshd

# Output: (nothing = success)

# Step 5: STAY IN NEW CONNECTION and test
whoami
echo "Still connected!"

# Step 6: If in old connection and still works, you're safe!
```

### Part D: Verification

```bash
# Verify hardening applied
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|maxauthtries"

# Test root login rejected:
ssh root@server
# Output: Permission denied (publickey)

# Test password auth rejected:
ssh -o PreferredAuthentications=password admin@server
# Output: Permission denied

# Test key auth works:
ssh -i ~/.ssh/id_rsa admin@server
# Output: Should connect successfully!
```

**Deliverable:**
- Before/after SSH configuration comparison
- Hardening checklist with verification
- Screenshots showing successful hardening
- Documentation of changes applied

---

## Exercise 1.3: SSH Banner Implementation

**Objective:** Add security warnings via SSH login banner.

### Part A: Create SSH Banner

```bash
# Create security banner file
sudo cat > /etc/ssh/banner.txt << 'EOF'
╔════════════════════════════════════════════════════════════╗
║                     AUTHORIZED ACCESS ONLY                ║
║                                                            ║
║  Unauthorized access to this system is forbidden and will ║
║  be prosecuted by law. By accessing this system, you agree ║
║  that your actions may be monitored and recorded.          ║
║                                                            ║
║  This is a private network. Do not attempt to access       ║
║  accounts or files belonging to other users.              ║
║                                                            ║
║  All activities on this system are logged.                ║
║  Violators will be prosecuted to the fullest extent       ║
║  of the law.                                              ║
║                                                            ║
║                  ** PROCEED AT YOUR OWN RISK **           ║
╚════════════════════════════════════════════════════════════╝
EOF

# Set permissions (world readable)
sudo chmod 644 /etc/ssh/banner.txt

# Enable in sshd_config
sudo nano /etc/ssh/sshd_config
# Ensure: Banner /etc/ssh/banner.txt

# Restart SSH
sudo systemctl restart sshd

# Check it displays
ssh admin@server
# Server should display banner before login prompt
```

### Part B: Test Banner Display

```bash
# Connect and see banner
ssh -v admin@server 2>&1 | grep -A 5 "AUTHORIZED ACCESS\|banner"

# Output should show banner appears before login
```

**Deliverable:**
- Banner file content
- Screenshot showing banner on SSH connection
- Explanation of legal notice purpose

---

## Exercise 1.4: Monitor SSH Activity After Hardening

**Objective:** Set up active monitoring to track attacks.

### Part A: Configure SSH Logging

```bash
# Ensure high log verbosity
sudo sshd -T | grep loglevel

# Set to VERBOSE for more details
sudo sed -i 's/^#LogLevel.*/LogLevel VERBOSE/' /etc/ssh/sshd_config

# Restart to apply
sudo systemctl restart sshd

# Check log location
grep "^SyslogFacility" /etc/ssh/sshd_config
# Usually: SyslogFacility AUTH

# View logs
sudo tail -50 /var/log/auth.log
```

### Part B: Create Monitoring Script

```bash
cat > /tmp/monitor_ssh.sh << 'EOF'
#!/bin/bash

echo "=== SSH ACTIVITY MONITORING ==="
echo "Time: $(date)"
echo ""

echo "SUCCESSFUL CONNECTIONS (last 10):"
sudo grep "Accepted publickey" /var/log/auth.log | tail -10

echo ""
echo "FAILED LOGIN ATTEMPTS (last 24 hours):"
sudo grep "Failed password\|Invalid user" /var/log/auth.log | tail -10

echo ""
echo "BRUTE FORCE ANALYSIS (top 10 IPs):"
sudo grep "Failed password" /var/log/auth.log | \
  grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | \
  sort | uniq -c | sort -rn | head -10

echo ""
echo "REAL-TIME MONITORING:"
echo "sudo tail -f /var/log/auth.log"
EOF

chmod +x /tmp/monitor_ssh.sh
/tmp/monitor_ssh.sh
```

### Part C: Automated Alerting

```bash
# Create alert script for suspicious activity
cat > /tmp/ssh_alert.sh << 'EOF'
#!/bin/bash

# Alert threshold: 10 failed attempts in 5 minutes

CHECK_INTERVAL=300  # 5 minutes
THRESHOLD=10

while true; do
    FAILED=$(sudo grep "Failed password" /var/log/auth.log | \
             grep "$(date -d '5 minutes ago' '+%b %d %H:%M')" | \
             wc -l)
    
    if [ $FAILED -gt $THRESHOLD ]; then
        echo "ALERT: $FAILED failed login attempts in last 5 minutes!"
        echo "Top attacking IPs:"
        sudo grep "Failed password" /var/log/auth.log | \
          grep "$(date -d '5 minutes ago' '+%b %d %H:%M')" | \
          grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | \
          sort | uniq -c | sort -rn | head -5
    fi
    
    sleep $CHECK_INTERVAL
done
EOF

chmod +x /tmp/ssh_alert.sh
# Running this in background: nohup /tmp/ssh_alert.sh &
```

**Deliverable:**
- SSH monitoring script output
- Log analysis showing attack patterns
- Alert configuration
- Report of SSH activity trends

---

## Exercise 1.5: Incident Response - SSH Compromise

**Objective:** Respond to compromised SSH access.

### Scenario: Attacker Gained SSH Access

Assume attacker obtained SSH key and logged in. Response plan:

```bash
# IMMEDIATE ACTIONS (Within minutes):

# 1. Verify compromise
echo "=== VERIFY COMPROMISE ==="
sudo grep "Accepted publickey" /var/log/auth.log | tail -5
# Check timestamps and IPs

# 2. Check for backdoors
echo "=== CHECK FOR BACKDOORS ==="
sudo grep "session opened" /var/log/auth.log | tail -10
# Look for suspicious user sessions

# 3. List users added recently
echo "=== RECENT USER ADDITIONS ==="
sudo grep "useradd\|adduser" /var/log/auth.log
# OR: sudo lastlog

# 4. Check authorized keys
echo "=== REVIEW AUTHORIZED_KEYS ==="
for user in $(cut -f1 -d: /etc/passwd | grep -E "^[a-z]"); do
    if [ -f "/home/$user/.ssh/authorized_keys" ]; then
        echo ""
        echo "User: $user"
        wc -l "/home/$user/.ssh/authorized_keys"
        head -1 "/home/$user/.ssh/authorized_keys"
    fi
done
```

### Remediation Steps

```bash
# STEP 1: Remove compromised keys
cat > /tmp/remediate_ssh.sh << 'EOF'
#!/bin/bash

echo "=== SSH COMPROMISE REMEDIATION ==="
echo ""

# Backup current state (for forensics)
mkdir -p /tmp/forensics/
sudo cp /home/*/.ssh/authorized_keys /tmp/forensics/ 2>/dev/null
sudo cp /etc/ssh/sshd_config /tmp/forensics/

echo "1. Removing suspicious SSH keys..."
# Remove all authorized keys
for user in $(cut -f1 -d: /etc/passwd | grep -E "^[a-z]"); do
    if [ -f "/home/$user/.ssh/authorized_keys" ]; then
        echo "  - /home/$user/.ssh/authorized_keys"
        # Backup first
        sudo cp "/home/$user/.ssh/authorized_keys" "/home/$user/.ssh/authorized_keys.backup"
        # Clear
        sudo truncate -s 0 "/home/$user/.ssh/authorized_keys"
    fi
done

echo ""
echo "2. Checking for backdoor users..."
# List all users
cat /etc/passwd | grep -E ":0:0:|nologin|false"

echo ""
echo "3. Changing SSH host keys..."
# Regenerate host keys (ensures attacker can't use captured session)
sudo ssh-keygen -A

echo ""
echo "4. Redeploying legitimate SSH keys..."
echo "  Manual step: Re-add known-good SSH keys"
echo "  ssh-copy-id -i ~/.ssh/id_rsa user@server"

echo ""
echo "5. Restarting SSH daemon..."
sudo systemctl restart sshd

echo ""
echo "Remediation complete. Review carefully before resuming operations."
EOF

chmod +x /tmp/remediate_ssh.sh
```

### Post-Incident Analysis

```bash
# Create incident report
cat > /tmp/incident_report.md << 'EOF'
# SSH COMPROMISE INCIDENT REPORT

## Timeline
- [Date/Time] Compromise detected
- [Date/Time] Remediation started
- [Date/Time] System restored

## Attack Vector
- [ ] Compromised SSH key
- [ ] Weak password (if password auth enabled)
- [ ] SSH daemon vulnerability
- [ ] Other: ___

## Damage Assessment
- [ ] Unauthorized file access
- [ ] Data exfiltration
- [ ] Malware installation
- [ ] Configuration changes
- [ ] Other: ___

## Remediation Steps
1. Removed compromised SSH keys
2. Regenerated host keys
3. Reinstalled SSH daemon
4. Reviewed system logs
5. Redeployed SSH keys

## Prevention
1. Ensure password auth disabled
2. Ensure root login disabled
3. Implement monitoring
4. Regular security audits
5. SSH key rotation schedule

## Follow-up
- [ ] Update SSH configuration
- [ ] Implement monitoring
- [ ] Review logs older than incident
- [ ] Test backups
- [ ] Document lessons learned
EOF

cat /tmp/incident_report.md
```

**Deliverable:**
- Incident detection procedure
- Remediation steps checklist
- Incident report template
- Forensics collection procedure

---

## Exercise 1.6: SSH Key Rotation Program

**Objective:** Implement automated SSH key rotation.

### Part A: Create Rotation Script

```bash
cat > /tmp/rotate_ssh_keys.sh << 'EOF'
#!/bin/bash

# SSH Key Rotation Script
# Rotates SSH host keys and user keys

echo "=== SSH KEY ROTATION ==="
echo "Date: $(date)"
echo ""

# Backup current keys
BACKUP_DIR="/tmp/ssh_key_backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
sudo cp /etc/ssh/ssh_host_* "$BACKUP_DIR/"
cp ~/.ssh/id_* "$BACKUP_DIR/user_keys/"

echo "1. Backed up to: $BACKUP_DIR"

# Rotate host keys (server side)
echo "2. Rotating SSH host keys (requires sudo)..."
sudo rm -f /etc/ssh/ssh_host_*
sudo ssh-keygen -A
echo "   Host keys regenerated"

# Rotate user keys (client side)
echo "3. Rotating user SSH keys..."
echo "   Old fingerprint:"
ssh-keygen -l -f ~/.ssh/id_rsa.pub

# Generate new key
ssh-keygen -t ed25519 -f ~/.ssh/id_rsa_new -N ""
echo "   New fingerprint:"
ssh-keygen -l -f ~/.ssh/id_rsa_new.pub

# Swap keys
mv ~/.ssh/id_rsa ~/.ssh/id_rsa_old
mv ~/.ssh/id_rsa_new ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa

echo ""
echo "4. Deploy new public key to server..."
echo "   ssh-copy-id -i ~/.ssh/id_rsa user@server"

echo ""
echo "5. Verify new key works..."
echo "   ssh -i ~/.ssh/id_rsa user@server"

echo ""
echo "Rotation complete. Old keys backed up in: $BACKUP_DIR"
EOF

chmod +x /tmp/rotate_ssh_keys.sh
```

### Part B: Schedule Rotation

```bash
# Add to crontab for automatic rotation (monthly)
cat > /tmp/ssh_rotation_cron << 'EOF'
# Rotate SSH keys monthly (first Sunday of month)
0 2 1-7 * * test $(date +\%w) = 0 && /tmp/rotate_ssh_keys.sh

# Reminder email about key rotation
0 0 1 * * * echo "SSH Key Rotation Due" | mail -s "SSH Rotation Reminder" admin@domain.com
EOF

# View current crontab
crontab -l

# Add entry (manual or with editor)
# crontab -e
```

**Deliverable:**
- Key rotation script
- Cron schedule configuration
- Rotation procedure documentation
- Key rotation audit log

---

## Exercise Summary

```
SSH HARDENING MASTERY:

Key Achievements:
✓ SSH configuration audited and hardened
✓ Root login disabled
✓ Password authentication disabled
✓ Key-based authentication enforced
✓ SSH logging and monitoring active
✓ Security banner implemented
✓ Incident response procedures tested
✓ Key rotation automation established

Security Improvements:
├─ Brute force attacks ineffective
├─ Compromised passwords = no access
├─ Only valid SSH keys work
├─ All activities logged and monitored
├─ Quick incident response possible
├─ Backdoors detected via alerting
└─ System continuously hardened

Real-world Impact:
├─ Server security posture: STRONG
├─ Attack surface: MINIMAL
├─ Detection capability: EXCELLENT
├─ Recovery time: FAST
├─ Overall risk: LOW
└─ Compliance: EXCELLENT
```

---

**Congratulations!** SSH hardening is now mastered. This foundation protects every system you manage.
