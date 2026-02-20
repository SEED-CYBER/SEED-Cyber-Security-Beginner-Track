# Part 4: SSH & VPS - Enhanced Exercises

## Exercise 4.1: Understanding SSH Key Generation

**Objective:** Generate SSH keys and understand the cryptography involved.

### Part A: Generate Your Key Pair

```bash
# Check if you already have keys
ls -la ~/.ssh/

# Generate new RSA key if needed
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_test1

# Prompt: Enter passphrase (create strong one!)
# Should output:
# Generating public/private rsa key pair.
# Your identification has been saved in ~/.ssh/id_rsa_test1
# Your public key has been saved in ~/.ssh/id_rsa_test1.pub
# ...

# View the pair (different representations of same key)
echo "=== PUBLIC KEY ==="
cat ~/.ssh/id_rsa_test1.pub
# Output: ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB...very long string... user@hostname

echo "=== PRIVATE KEY ==="
head ~/.ssh/id_rsa_test1
# Output: -----BEGIN RSA PRIVATE KEY-----
#         Proc-Type: 4,ENCRYPTED
#         DEK-Info: AES-128-CBC,...
#         [encrypted data]
```

### Part B: Analyze Key Structure

```bash
# Check permissions
ls -la ~/.ssh/id_rsa_test1*
# Private key should be: -rw------- (600)
# Public key can be: -rw-r--r-- (644)

# Get key file sizes
du -h ~/.ssh/id_rsa_test1*
# Private key: depends on encryption, usually ~3.2K
# Public key: usually ~1K

# Extract key information
ssh-keygen -l -f ~/.ssh/id_rsa_test1.pub
# Output shows: fingerprint (unique identifier for this key)
# Example: 4096 SHA256:aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890+= user@hostname

# Get different fingerprint format (baby mode)
ssh-keygen -l -f ~/.ssh/id_rsa_test1.pub -E md5
# More compact, older SSH clients understand this

# Get full key details
ssh-keygen -l -f ~/.ssh/id_rsa_test1.pub -v
# Shows: ├─ Key bits
#        ├─ Fingerprint
#        ├─ Key type
#        └─ Randomart visualization (for visual verification!)

# Get SSH public key file format info
head -1 ~/.ssh/id_rsa_test1.pub
# Should say: ssh-rsa
# (or ssh-ed25519 if using Ed25519 keys)
```

### Part C: Compare Key Types

```bash
# Generate other key types to compare

# Ed25519 (modern, smaller, faster)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_test -N "passphrase"

# ECDSA (elliptic curve)
ssh-keygen -t ecdsa -b 521 -f ~/.ssh/id_ecdsa_test -N "passphrase"

# Compare file sizes
echo "Key type comparisons:"
ls -lh ~/.ssh/id_*test*
# RSA-4096: ~3.2K (largest)
# Ed25519: ~1.8K (smallest)
# ECDSA-521: ~2.5K (medium)

# Compare fingerprints
echo "=== RSA Fingerprint ==="
ssh-keygen -l -f ~/.ssh/id_rsa_test1.pub
echo ""
echo "=== Ed25519 Fingerprint ==="
ssh-keygen -l -f ~/.ssh/id_ed25519_test.pub
echo ""
echo "=== ECDSA Fingerprint ==="
ssh-keygen -l -f ~/.ssh/id_ecdsa_test.pub

# Security implications
cat << 'EOF'
RSA-4096:
  ├─ Industry standard
  ├─ Large key size (older systems need this)
  ├─ Slower operations
  └─ Largest file size

Ed25519:
  ├─ Modern standard
  ├─ Smallest key size (security per bit is highest)
  ├─ Fastest operations
  ├─ Not supported by very old systems
  └─ Recommended for new keys

ECDSA:
  ├─ Middle-ground option
  ├─ Common in cloud deployments
  ├─ Smaller than RSA, larger than Ed25519
  └─ Wide support
EOF
```

**Deliverable:**
- Screenshots showing key fingerprints
- Table comparing key types (size, fingerprint, type)
- Explanation of which key type you'd recommend why

---

## Exercise 4.2: SSH Connection Handshake Analysis

**Objective:** Observe and understand the SSH handshake process in detail.

### Part A: Verbose SSH Connection

```bash
# Connect with verbose output (shows handshake)
# Create dummy server locally if needed, or use any accessible server

# Connect to a test server with maximum verbosity
ssh -vvv user@testserver.com 2>&1 | head -50

# Key output to look for:

# 1. Version Exchange
# OpenSSH_8.2p1 Ubuntu 4ubuntu0.5, OpenSSL 1.1.1f
# SSH-2.0 line shows server version

# 2. Key Exchange Algorithms
# debug1: Authentications that can continue: publickey,password
# Shows what auth methods server accepts

# 3. Server Key Verification
# The server's ECDSA key fingerprint is SHA256:abcdef...
# debug1: Checking host key in /home/user/.ssh/known_hosts
# This is verification step!

# 4. Authentication
# debug1: Authentication succeeded (publickey)
# Shows which method worked

# Full verbose output
ssh -vvv user@testserver.com 2>&1 | tee /tmp/ssh_handshake.log

# Analyze specific parts
grep "Server host key" /tmp/ssh_handshake.log
grep "Authentications that" /tmp/ssh_handshake.log
grep "Authentication succeeded" /tmp/ssh_handshake.log
```

### Part B: Analyze Handshake Stages

```bash
# Create a detailed handshake tracker
cat > /tmp/analyze_ssh.sh << 'EOF'
#!/bin/bash
# Analyze SSH handshake stages

echo "=== SSH HANDSHAKE ANALYSIS ==="
echo ""
echo "Stage 1: Initial Connection"
echo "├─ TCP connection to port 22"
echo "├─ Banner exchange (version strings)"
echo "└─ Algorithm negotiation"
echo ""

echo "Stage 2: Key Exchange"
echo "├─ Generate shared session key"
echo "├─ Both sides verify they computed same key"
echo "├─ All future traffic will use this key"
echo "└─ Session key NOT the same as SSH key!"
echo ""

echo "Stage 3: Server Authentication"
echo "├─ Server presents host key"
echo "├─ Client verifies via known_hosts"
echo "├─ First time: User must accept"
echo "└─ Later times: Automatic verification"
echo ""

echo "Stage 4: User Authentication"
echo "├─ Either publickey or password method"
echo "├─ Publickey:"
echo "│   ├─ Client sends public key"
echo "│   ├─ Server checks authorized_keys"
echo "│   ├─ Server sends challenge"
echo "│   ├─ Client signs with private key"
echo "│   └─ Server verifies signature"
echo "└─ Password:"
echo "    ├─ Client sends username"
echo "    ├─ Server responds 'password please'"
echo "    └─ Client sends password (encrypted!)"
echo ""

echo "Stage 5: Channel Open"
echo "├─ Authenticated session established"
echo "├─ Shell or command execution begins"
echo "└─ All communication encrypted with session key!"
EOF

chmod +x /tmp/analyze_ssh.sh
/tmp/analyze_ssh.sh
```

### Part C: Examine known_hosts

```bash
# View your collected host keys
cat ~/.ssh/known_hosts

# Format: hostname ssh-key-type hostname-key-encrypted
# Example line:
# server.example.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQDx+...

# What happens when fingerprint mismatches?
echo "=== FINGERPRINT MISMATCH SCENARIO ==="
echo "If you get:"
echo "'WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!'"
echo ""
echo "This means:"
echo "├─ known_hosts has one fingerprint"
echo "├─ Server now has different fingerprint"
echo "└─ Possibilities:"
echo "   ├─ Server was reinstalled (normal)"
echo "   ├─ Server was compromised (serious!)"
echo "   └─ This is a MITM attack (very serious!)"
echo ""
echo "What to do:"
echo "1. Contact server admin: Did you reinstall?"
echo "2. If yes: ssh-keygen -R server.com"
echo "3. Connect fresh (accept new key)"
echo "4. Verify fingerprint via phone/email"

# To remove and re-add:
# ssh-keygen -R server.com
# Then SSH connects and you re-accept the key
```

**Deliverable:**
- SSH verbose output showing all stages
- Handshake analysis diagram (text or image)
- Explanation of what happens at each stage

---

## Exercise 4.3: Managing Multiple SSH Keys

**Objective:** Understand key management for different servers and purposes.

### Part A: Organize SSH Keys

```bash
# Create a structured key management system
mkdir -p ~/.ssh/keys/production
mkdir -p ~/.ssh/keys/development
mkdir -p ~/.ssh/keys/personal

# Generate keys for different purposes
# NEVER reuse the same key for multiple servers!
# If one server compromised, all others accessible

# Production server key (high security)
ssh-keygen -t ed25519 -f ~/.ssh/keys/production/id_prod \
  -N "complex_passphrase_here" \
  -C "production-key-$(date +%Y%m%d)"

# Development server key (lower security, easier passphrase)
ssh-keygen -t ed25519 -f ~/.ssh/keys/development/id_dev \
  -N "dev_passphrase" \
  -C "development-key-$(date +%Y%m%d)"

# Personal projects (relaxed security)
ssh-keygen -t ed25519 -f ~/.ssh/keys/personal/id_personal \
  -N "personal_phrase" \
  -C "personal-key-$(date +%Y%m%d)"

# List organized keys
echo "=== KEY ORGANIZATION ==="
find ~/.ssh/keys -name "id_*" -o -name "id_*.pub" | sort

# View permissions (all should be 600 for private keys)
ls -la ~/.ssh/keys/*/*
```

### Part B: SSH Config for Key Management

```bash
# Create ~/.ssh/config to use different keys automatically

cat > ~/.ssh/config << 'EOF'
# Production servers (high security)
Host prod-*
    User admin
    IdentityFile ~/.ssh/keys/production/id_prod
    IdentityFile ~/.ssh/keys/production/id_prod_backup
    # Will try first key, then backup if needed
    
Host prod-web1
    HostName 203.0.113.100
    User admin
    # Inherits rules from prod-*
    
Host prod-db1
    HostName 203.0.113.101
    User admin

# Development servers (convenient, less paranoid)
Host dev-* 
    User developer
    IdentityFile ~/.ssh/keys/development/id_dev
    IdentityFile ~/.ssh/keys/development/id_dev_backup
    
Host dev-app
    HostName 198.51.100.50
    User developer

# Personal projects
Host personal-*
    User user
    IdentityFile ~/.ssh/keys/personal/id_personal
    
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/keys/personal/id_github
    # GitHub-specific key

# Global defaults
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    HashKnownHosts yes
    # Hash hostnames in known_hosts for privacy
EOF

# Verify config syntax
ssh -G prod-web1
# Should show all config for this host

# Usage examples:
# ssh prod-web1              (no need to type full hostname!)
# ssh dev-app                (uses dev key automatically)
# ssh personal-server        (uses personal key)
```

### Part C: Key Rotation Script

```bash
# Create automated key rotation
cat > /tmp/rotate_ssh_keys.sh << 'EOF'
#!/bin/bash

echo "=== SSH KEY ROTATION ==="
echo ""

SERVER="$1"
if [ -z "$SERVER" ]; then
    echo "Usage: $0 server_name"
    echo "Example: $0 prod-web1"
    exit 1
fi

# Get current key info
echo "Current key on $(echo $SERVER | tr '_' ' '):"
ssh -G "$SERVER" | grep IdentityFile | head -1

# Generate new key
echo ""
echo "Generating new key for $SERVER..."
NEW_KEY_PATH="~/.ssh/keys/$(echo $SERVER | cut -d'-' -f1)/${SERVER}-$(date +%Y%m%d)"

# ssh-keygen -t ed25519 -f "$NEW_KEY_PATH" -N "new_passphrase"

# TODO: Implement these steps
cat << 'STEPS'
Key Rotation Steps:
1. Generate new key
2. Add new key to ~/.ssh/keys/...
3. SSH to server
4. Append new public key to ~/.ssh/authorized_keys
5. Verify new key works (SSH test)
6. Remove old public key from authorized_keys
7. Update ~/.ssh/config to use new key
8. Verify old key no longer works
9. Delete old key file (now unused)
STEPS
EOF

chmod +x /tmp/rotate_ssh_keys.sh
```

**Deliverable:**
- Directory structure showing organized keys
- SSH config file for multiple servers
- Key information showing different purposes
- Documentation of key rotation process

---

## Exercise 4.4: SSH Server Security Audit

**Objective:** Analyze SSH security configuration and identify vulnerabilities.

### Part A: Audit SSH Configuration

```bash
# Get SSH daemon configuration
sudo sshd -T
# Shows actual SSH config (including defaults)

# Check critical security settings
echo "=== CRITICAL SSH SECURITY SETTINGS ==="

echo "Root login allowed?"
sudo sshd -T | grep "permitrootlogin"
# Should be: permitrootlogin no

echo "Password authentication allowed?"
sudo sshd -T | grep "passwordauthentication"  
# Should be: passwordauthentication no

echo "Public key authentication enabled?"
sudo sshd -T | grep "pubkeyauthentication"
# Should be: pubkeyauthentication yes

echo "Max auth attempts:"
sudo sshd -T | grep "maxauthtries"
# Should be: maxauthtries 3 or 5

echo "SSH listening port:"
sudo sshd -T | grep "port"
# Usually: port 22

# Full security audit
echo ""
echo "=== FULL SSH SECURITY AUDIT ==="
sudo sshd -T | grep -E "permit|password|key|auth|port" | sort
```

### Part B: Check for Security Issues

```bash
# Check who can SSH
cat > /tmp/ssh_audit.sh << 'EOF'
#!/bin/bash

echo "=== SSH SECURITY AUDIT ==="
echo ""

echo "1. SSH Users and Groups:"
sudo grep "^AllowUsers\|^DenyUsers\|^AllowGroups" /etc/ssh/sshd_config

echo ""
echo "2. SSH Key Files and Permissions:"
ls -la ~/.ssh/
echo ""
printf "Private key permissions: "
stat -c '%A' ~/.ssh/id_rsa 2>/dev/null || echo "Not found"
echo ""
printf ".ssh directory permissions: "
stat -c '%A' ~/.ssh 2>/dev/null || echo "Not found"

echo ""
echo "3. Public Keys Authorized:"
ls -la ~/.ssh/authorized_keys 2>/dev/null || echo "Not found"
echo ""
echo "Number of authorized public keys:"
wc -l ~/.ssh/authorized_keys 2>/dev/null || echo "None"

echo ""
echo "4. Known Hosts:"
wc -l ~/.ssh/known_hosts 2>/dev/null || echo "None"

echo ""
echo "5. Failed SSH Attempts (last 24 hours):"
sudo journalctl -u ssh -n 100 --since "24 hours ago" | grep -i "failed"
EOF

chmod +x /tmp/ssh_audit.sh
/tmp/ssh_audit.sh
```

### Part C: Security Recommendations

```bash
# Create security checklist
cat > /tmp/ssh_security_checklist.txt << 'EOF'
SSH SECURITY CHECKLIST
======================

Client-Side (Your Machine):
[  ] Private key exist and protected (600 permissions)
[  ] SSH directory protected (700 permissions)
[  ] Passphrase used on private key
[  ] Different keys for different purposes
[  ] Keys rotated at least annually
[  ] ssh-agent in use (caches keys)
[  ] ~/.ssh/config managing multiple keys
[  ] ~/.ssh/known_hosts has expected servers
[  ] No unexpected entries in known_hosts

Server-Side (VPS/Server):
[  ] SSH daemon uses port 22 (or documented alternate)
[  ] PermitRootLogin set to no
[  ] PasswordAuthentication set to no
[  ] PubkeyAuthentication set to yes
[  ] MaxAuthTries limited (3-5)
[  ] ~/.ssh/authorized_keys protected (600)
[  ] ~/.ssh directory protected (700)
[  ] Only necessary users listed in AllowUsers
[  ] SSH daemon restarted after config changes
[  ] SSH service enabled on startup
[  ] SSH access logged and monitored
[  ] fail2ban or similar rate limiting active

Monitoring:
[  ] SSH activity logged
[  ] Failed login attempts monitored
[  ] Suspicious activity investigated
[  ] Regular key audits performed
[  ] Access logs reviewed monthly
[  ] Intrusion attempts detected and blocked

Incident Response:
[  ] Recovery procedure documented
[  ] Backup authentication method available
[  ] Emergency access account configured
[  ] Key rotation procedure documented
EOF

cat /tmp/ssh_security_checklist.txt
```

**Deliverable:**
- SSH configuration audit report
- Security settings verification (screenshot)
- Checklist with self-assessment
- Recommendations for improvements

---

## Exercise 4.5: SSH Public Key Cryptography Hands-On

**Objective:** Verify how public key cryptography actually works.

### Part A: Demonstrate One-Way Function

```bash
# Public key cryptography is based on one-way functions
# Easy to compute one direction: Hard to reverse

# Simulate with hashing (related concept)
echo "Password123!" | sha256sum
# Output: hash (can't reverse to get password)

# Try different inputs
echo "Password123!" | sha256sum  # Hash A
echo "password123!" | sha256sum  # Hash B (very different!)

# Key insight:
echo "Same input = same hash"
echo "Different input = completely different hash"
echo "Hash doesn't contain password (can't reverse)"

# With SSH keys:
cat ~/.ssh/id_ed25519.pub
# This IS your public key (can be shared)
# No way to reverse it to get private key
# Math doesn't work backward
```

### Part B: Verify Signature Process

```bash
# Create a test file
echo "This is a message to sign" > /tmp/message.txt

# Sign it with private key
openssl dgst -sha256 -sign ~/.ssh/id_rsa /tmp/message.txt > /tmp/message.sig 2>/dev/null || \
echo "Note: Need to convert SSH key format for this demo"

# In actual SSH:
# 1. Server generates random challenge
# 2. Client signs challenge with private key
# 3. Server verifies signature with public key
# If signature verifies: Client PROVEN to have private key!

# Safe because:
# ├─ Only private key can create signature
# ├─ Public key verifies it (but can't create it)
# └─ Attacker would need private key (has it? No!)
```

### Part C: Test Authentication Without Package Password

```bash
# Connect to server with public key
ssh -i ~/.ssh/id_ed25519 user@server.com

# If successful, no password entered!
# Entire authentication happened via cryptography

# Verify you're authenticated
echo "Authentication successful if you see this!"
whoami
hostname

# If it fails with "permission denied":
# ├─ Private key doesn't match authorized_keys public key
# ├─ authorized_keys has wrong permissions
# ├─ Private key has wrong permissions
# └─ Server has key-only auth enabled
```

**Deliverable:**
- Demonstration of one-way function (hashing)
- Successful SSH connection using public key
- Explanation of why it's secure
- Comparison: password auth vs key auth

---

## Exercise 4.6: VPS Initial Hardening

**Objective:** Perform complete hardening of a fresh VPS.

### Part A: Create VPS Deployment Checklist

```bash
# Create complete hardening script (read-only for now)
cat > /tmp/vps_hardening_checklist.md << 'EOF'
# VPS INITIAL HARDENING CHECKLIST

## Phase 1: Initial Access (First 5 minutes)

[ ] Connect as root via SSH
[ ] Verify server fingerprint
[ ] Change root password
[ ] Document new root password (secure storage)

## Phase 2: System Updates (First 10 minutes)

[ ] apt update
[ ] apt upgrade -y
[ ] apt autoremove -y
[ ] Note: Takes 5-10 minutes on fresh instance

## Phase 3: Create Admin User (First 15 minutes)

[ ] useradd -m -s /bin/bash admin
[ ] passwd admin (strong password)
[ ] usermod -aG sudo admin
[ ] Test: sudo -u admin sudo -l

## Phase 4: SSH Key Deployment (First 30 minutes)

Client:
[ ] ssh-keygen -t ed25519 -f ~/.ssh/id_prod_vps1

Server:
[ ] mkdir -p ~/.ssh
[ ] chmod 700 ~/.ssh

Client:
[ ] ssh-copy-id -i ~/.ssh/id_prod_vps1.pub admin@vps.ip

[ ] Test connection (NEW TERMINAL): ssh -i ~/.ssh/id_prod_vps1 admin@vps.ip

## Phase 5: Disable Root SSH (First 35 minutes)

Server:
[ ] sudo nano /etc/ssh/sshd_config
[ ] PermitRootLogin no
[ ] PubkeyAuthentication yes
[ ] PasswordAuthentication no
[ ] PermitEmptyPasswords no
[ ] MaxAuthTries 3

[ ] sudo systemctl restart sshd
[ ] TEST in NEW TERMINAL before closing current!

## Phase 6: Firewall Configuration (First 40 minutes)

[ ] sudo ufw enable
[ ] sudo ufw allow 22/tcp
[ ] sudo ufw default deny incoming
[ ] sudo ufw status

## Phase 7: System Hardening (First 50 minutes)

[ ] sudo hostnamectl set-hostname my-vps-name
[ ] sudo timedatectl set-timezone UTC
[ ] sudo apt install -y fail2ban
[ ] sudo systemctl enable fail2ban

[ ] Configure sudo to require password on every use
[ ] Configure automatic security updates

## Phase 8: Verification (Final 60 minutes)

[ ] All SSH connections tested
[ ] Firewall rules verified
[ ] Only necessary ports open
[ ] Root password changed
[ ] Admin user created and functional
[ ] No password-based SSH allowed
[ ] Key-based SSH working

## Time Budget:
├─ Phase 1-2: 10 minutes
├─ Phase 3-4: 20 minutes
├─ Phase 5-6: 10 minutes
├─ Phase 7-8: 20 minutes
└─ Total: ~60 minutes for hardened VPS

## Expected Result:
✓ Secure VPS ready for application deployment
✓ SSH key-only access
✓ Firewall enabled
✓ Automatic security updates
✓ fail2ban protecting from brute force
✓ Ready for application stack
EOF

cat /tmp/vps_hardening_checklist.md
```

### Part B: Simulate VPS Setup (Local)

```bash
# Create a local simulation of hardening workflow
cat > /tmp/simulate_vps_hardening.sh << 'EOF'
#!/bin/bash

echo "=== VPS HARDENING SIMULATION ==="
echo ""
echo "This script demonstrates the entire hardening process"
echo "(It doesn't actually modify anything)"
echo ""

echo "STEP 1: Initial system updates"
echo "$ sudo apt update && apt upgrade -y"
echo ""

echo "STEP 2: Create admin user"
echo "$ sudo useradd -m -s /bin/bash admin"
echo "$ sudo passwd admin"
echo ""

echo "STEP 3: Create SSH key (on your machine)"
echo "$ ssh-keygen -t ed25519 -f ~/.ssh/vps_key"
echo ""

echo "STEP 4: Deploy key to VPS"
echo "$ ssh-copy-id -i ~/.ssh/vps_key.pub admin@vps.ip"
echo ""

echo "STEP 5: Edit SSH config on VPS"
echo "$ sudo nano /etc/ssh/sshd_config"
echo "  Set:"
echo "    PermitRootLogin no"
echo "    PasswordAuthentication no"
echo "    PubkeyAuthentication yes"
echo ""

echo "STEP 6: Restart SSH"
echo "$ sudo systemctl restart sshd"
echo ""

echo "STEP 7: Enable firewall"
echo "$ sudo ufw enable"
echo "$ sudo ufw allow 22/tcp"
echo ""

echo "STEP 8: Install fail2ban"
echo "$ sudo apt install fail2ban"
echo "$ sudo systemctl enable fail2ban"
echo ""

echo "VERIFICATION:"
echo "$ ssh -i ~/.ssh/vps_key admin@vps.ip"
echo "'Welcome to hardened VPS!'"
EOF

chmod +x /tmp/simulate_vps_hardening.sh
/tmp/simulate_vps_hardening.sh
```

**Deliverable:**
- VPS hardening documentation
- Complete checklist for fresh VPS
- Time estimates for each step
- Screenshots of successful hardening (if actual VPS available)

---

## Exercise 4.7: SSH Security Incident Investigation

**Objective:** Learn to detect and investigate SSH security incidents.

### Part A: Detect Suspicious Activity

```bash
# Check failed SSH attempts
echo "=== FAILED SSH ATTEMPTS (Last 24 hours) ==="
sudo journalctl -u ssh -n 1000 --since "24 hours ago" | grep -i "failed\|invalid"

# Check successful SSH connections
echo ""
echo "=== SUCCESSFUL SSH CONNECTIONS ==="
sudo journalctl -u ssh -n 100 --since "24 hours ago" | grep -i "accepted"

# Analyze specific attempts
echo ""
echo "=== BRUTE FORCE DETECTION ==="

# Count failed attempts by IP
sudo journalctl -u ssh -n 10000 | grep "Invalid\|Failed" | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -rn | head -20

# View last login attempts
echo ""
echo "=== LAST LOGINS ==="
last -20  # Local
sudo journalctl -u ssh | grep "session opened" | tail -20  # SSH

# Check currently logged-in users
echo ""
echo "=== CURRENT SESSIONS ==="
w
```

### Part B: Create Security Monitoring Script

```bash
cat > /tmp/ssh_security_monitor.sh << 'EOF'
#!/bin/bash

echo "=== SSH SECURITY MONITORING SCRIPT ==="
echo "Date: $(date)"
echo ""

# Monthly security report
echo "MONTHLY SSH SECURITY REPORT"
echo "============================"
echo ""

echo "1. Failed Authentication Attempts:"
sudo journalctl -u ssh --since "30 days ago" | grep -i "failed\|invalid" | wc -l
echo ""

echo "2. Unique IPs Attempting Access:"
sudo journalctl -u ssh --since "30 days ago" | grep -i "invalid\|failed" | \
  grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort -u | wc -l
echo ""

echo "3. Top 10 Attacking IPs:"
sudo journalctl -u ssh --since "30 days ago" | grep -i "invalid\|failed" | \
  grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -rn | head -10
echo ""

echo "4. Successful SSH Sessions:"
sudo journalctl -u ssh --since "30 days ago" | grep "session opened" | wc -l
echo ""

echo "5. Last SSH Attempts (Success + Failure):"
sudo journalctl -u ssh --since "1 hour ago" | tail -10
EOF

chmod +x /tmp/ssh_security_monitor.sh
/tmp/ssh_security_monitor.sh
```

**Deliverable:**
- SSH security audit report
- Failed attempt analysis
- Potential threat identification
- Recommendations for addressing threats

---

## Exercise Summary

```
SSH & VPS SECURITY MASTERY:

Key Learnings:
1. SSH uses public key cryptography (not password-based)
2. Private key MUST remain 600 permissions (owner only)
3. Server host key verification prevents MITM
4. SSH config enables key management for multiple servers
5. VPS hardening is systematic: Updates > Users > SSH > Firewall
6. Monitor SSH logs for intrusions
7. Key rotation periodic prevents compromise impact
8. Different keys for different purposes (isolation)

Practical Skills Gained:
✓ Generate, manage, and rotate SSH keys
✓ Understand cryptography fundamentals
✓ Harden SSH servers
✓ Analyze SSH handshakes
✓ Detect security incidents
✓ Deploy VPS securely
✓ Manage multiple SSH keys for multiple servers

Security Principles Applied:
├─ Defense in Depth (layers of security)
├─ Least Privilege (only necessary access)
├─ Key Isolation (separate keys for separate purposes)
├─ Continuous Monitoring (watch for attacks)
├─ Incident Response (detect and investigate)
└─ Security Hardening (eliminate attack vectors)
```

---

**Congratulations!** You understand SSH security from first principles. You can now securely manage remote servers and understand the cryptography protecting your systems.
