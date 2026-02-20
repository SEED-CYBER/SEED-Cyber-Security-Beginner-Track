# Week 4 Comprehensive Exercises

## Overview
Week 4 focuses on **server hardening completion** (SSH, firewalls, fail2ban, network scanning) from Chapters 2-3. This week transitions from defensive basics to practical hardening in production-like environments.

---

## Exercise W4-1: Complete SSH Hardening Configuration

**Time: 120 minutes | Difficulty: Intermediate**

### Objectives
- Disable password authentication
- Configure key-based authentication
- Implement timeout settings
- Set up port forwarding restrictions
- Enable audit logging

### Setup
```bash
# Create fresh Ubuntu VM or container
docker run -it -p 2222:22 ubuntu:22.04 /bin/bash
# or
vagrant init ubuntu/jammy && vagrant up

# Install SSH server
sudo apt update && sudo apt install -y openssh-server openssh-client
sudo systemctl start ssh
sudo systemctl enable ssh
```

### Step-by-Step Configuration

**Step 1: Backup Original Config** (5 min)
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

**Step 2: Create Non-Root User with SSH Key**  (15 min)
```bash
# Create regular user
sudo useradd -m -s /bin/bash admin
sudo usermod -aG sudo admin

# Generate SSH key pair (ON ATTACKER/ADMIN MACHINE):
# Linux/WSL:
ssh-keygen -t ed25519 -f ~/.ssh/admin_key -N "SecurePassphrase123!"

# Windows PowerShell:
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\admin_key -N "SecurePassphrase123!"

# Copy public key to server
ssh-copy-id -i ~/.ssh/admin_key.pub -p 2222 admin@target-server

# or manually:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-ed25519 AAAA... label" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Step 3: Harden SSH Configuration** (20 min)
```bash
sudo nano /etc/ssh/sshd_config

# Apply these settings:
```

```ini
# Disable root login
PermitRootLogin no

# Disable password authentication  
PasswordAuthentication no
PubkeyAuthentication yes

# Disable X11 forwarding
X11Forwarding no

# Limited authentication attempts
MaxAuthTries 3

# Connection timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable empty passwords
PermitEmptyPasswords no

# Disable hostname authentication
IgnoreRhosts yes

# Enable strict mode
StrictModes yes

# Limit concurrent sessions
MaxSessions 10

# Port (change from 22 to 2222)
Port 2222

# Host keys
HostKey /etc/ssh/ssh_host_ed25519_key

# Use strong key exchange algorithms
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Use strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com

# Use strong MAC algorithms
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

```bash
# Check syntax
sudo sshd -T | grep -E "permitrootlogin|passwordauth|port"

# Restart SSH
sudo systemctl restart ssh
```

**Step 4: Enable SSH Audit Logging** (10 min)
```bash
# Configure auditd rules for SSH access
sudo apt install auditd
sudo nano /etc/audit/rules.d/ssh.rules
```

```bash
# Add these lines:
-w /etc/ssh/sshd_config -p wa -k ssh_config_changes
-w /var/log/auth.log -p wa -k ssh_auth_log
-a always,exit -F arch=b64 -S execve -F exe=/usr/sbin/sshd -k ssh_execution
```

```bash
# Apply rules
sudo systemctl restart auditd

# View SSH access logs
sudo ausearch -k ssh_execution -ts recent
sudo grep sshd /var/log/auth.log
```

**Step 5: Verify Configuration** (20 min)
```bash
# Test that password auth is disabled
ssh -o PasswordAuthentication=yes admin@localhost
# Expected: Permission denied (publickey only)

# Test key-based auth works
ssh -i ~/.ssh/admin_key -p 2222 admin@target-server
# Expected: Successful login

# Test root login is disabled
ssh -i root_key.pem root@target-server
# Expected: Permission denied

# Verify no weak ciphers
ssh -p 2222 -Q ciphers

# Check active connections
ss -tulpn | grep ssh
```

**Step 6: Document Configuration** (10 min)
Create a report including:
- [ ] Before/after sshd_config diff
- [ ] Screenshot of successful key-based login
- [ ] Screenshot of failed password auth attempt
- [ ] Audit log entries showing access
- [ ] Security checklist with all hardening steps

### Deliverables
- [ ] Hardened sshd_config file
- [ ] SSH key pair (public key)
- [ ] Audit rule configuration
- [ ] Configuration report (500+ words)
- [ ] Test results summary

**Success Criteria**
- ✓ Root login disabled
- ✓ Password auth disabled  
- ✓ Port changed from 22
- ✓ Strong algorithms only
- ✓ Audit logging enabled
- ✓ All tests pass

---

## Exercise W4-2: Firewall Configuration & Testing

**Time: 90 minutes | Difficulty: Intermediate**

### Objectives
- Configure UFW firewall
- Define allow/deny rules
- Test with Nmap
- Create firewall policy document

### Configuration

```bash
# Step 1: Enable firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable

# Step 2: Allow SSH (critical!)
sudo ufw allow 2222/tcp comment "SSH hardened port"

# Step 3: Allow web services
sudo ufw allow 80/tcp comment "HTTP"
sudo ufw allow 443/tcp comment "HTTPS"

# Step 4: Allow specific service (e.g., database)
sudo ufw allow from 10.0.0.0/24 to any port 3306 comment "MySQL from DB subnet"

# Step 5: Deny specific ports
sudo ufw deny 23/tcp comment "Telnet denied"
sudo ufw deny 25/tcp comment "SMTP blocked globally"

# Step 6: View active rules
sudo ufw status verbose

# Step 7: Log firewall activity
sudo nano /etc/ufw/before.rules
# Add logging:
-A ufw-before-input -m limit --limit 5/min -j LOG --log-prefix "[UFW BLOCK] "

# Or use:
sudo nano /etc/ufw/sysctl.conf
# Uncomment: net/ipv4/ip_forward=1
```

### Testing with Nmap

```bash
# From another machine:

# Step 1: SYN scan
nmap -sS target-server

# Step 2: Service detection on allowed ports
nmap -sV -p 22,80,443 target-server

# Step 3: Test denied port (should be filtered)
nmap -p 23,25 target-server
# Expected: filtered (no response)

# Step 4: Comprehensive scan
nmap -A -T4 target-server | tee nmap-results.txt
```

### Verify & Document

```bash
# Capture firewall logs
sudo journalctl -u ufw | tail -50

# Export UFW rules
sudo ufw status numbered > firewall-rules.txt

# Test each rule
netstat -tulpn  # View listening ports
ss -tulpn       # Modern alternative
```

### Deliverables
- [ ] UFW configuration with all rules
- [ ] Nmap scan results (before/after)
- [ ] Screenshots showing:
  - ufw status output
  - Blocked connection attempts
  - Allowed connections working
- [ ] Firewall policy document (3-5 pages)

---

## Exercise W4-3: Multi-System Hardening Lab

**Time: 120 minutes | Difficulty: Advanced**

### Scenario
Deploy a mini three-tier architecture and harden each layer:

```
     Internet
        ↓
   [Firewall VM]
        ↓
   [Web Server]
        ↓
   [Database Server]
```

### Setup

```bash
# Create 3 VMs or containers:
for i in {1..3}; do
  docker run -d --name hardening-lab-$i ubuntu:22.04 sleep 999999
done

# Or containers with networking:
docker network create hardening-lab
docker run -d --name firewall --network hardening-lab ubuntu:22.04
docker run -d --name webserver --network hardening-lab ubuntu:22.04
docker run -d --name database --network hardening-lab ubuntu:22.04
```

### Hardening Each Layer

**Firewall VM:**
```bash
# Install ufw, enable forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Masquerade for NAT
sudo nano /etc/ufw/before.rules
# Add: -A POSTROUTING -t nat -j MASQUERADE
```

**Web Server:**
```bash
# Install Nginx
sudo apt install nginx
sudo systemctl enable nginx

# Harden SSH (as in Exercise W4-1)
# Change port to 2222
# Disable password auth

# Create UFW rules
sudo ufw allow 22/tcp
sudo ufw allow from 10.0.0.0/8 to any port 80
sudo ufw allow from 10.0.0.0/8 to any port 443
sudo ufw deny 22/tcp from any  # After hardening SSH
```

**Database Server:**
```bash
# Install MySQL
sudo apt install mysql-server

# Configure for remote access from web tier only
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
# Set: bind-address = 10.0.0.10  (or internal IP)

# Create limited database user
mysql> CREATE USER 'webapp'@'10.0.0.x' IDENTIFIED BY 'StrongPass';
mysql> GRANT SELECT, INSERT, UPDATE ON database.* TO 'webapp'@'10.0.0.x';

# UFW rules
sudo ufw allow from web-server-ip to any port 3306
```

### Testing

```bash
# From attacker machine:
# Test each connection succeeds/fails as expected
nmap -A firewall-ip
curl http://webserver-ip
mysql -h database-ip -u webapp -p

# Verify traffic blocking
# Denied: SSH to web, direct SSH to DB
# Allowed: Web to DB
```

### Deliverables
- [ ] Architecture diagram
- [ ] Configuration files for all 3 systems
- [ ] Network topology documentation
- [ ] Test results (connectivity matrix)

---

## Exercise W4-4: Fail2Ban Configuration

**Time: 60 minutes | Difficulty: Intermediate**

### Objectives
- Install and configure Fail2Ban
- Create custom filters
- Test with simulated attacks
- Monitor bans

### Configuration

```bash
# Install
sudo apt install fail2ban

# Step 1: Create jail configuration
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
destemail = admin@example.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = 2222
maxretry = 3

[nginx-http-auth]
enabled = true

[nginx-limit-req]
enabled = true

[sshd-invalid-user]
enabled = true
maxretry = 1
```

```bash
# Step 2: Create custom filter for SQL injection
sudo nano /etc/fail2ban/filter.d/sqli.conf
```

```ini
[Definition]
failregex = ^<HOST>.*(\bUNION\b|\bSELECT\b|' OR '.*).*HTTP/
ignoreregex =
```

```bash
# Step 3: Enable in jail
sudo nano /etc/fail2ban/jail.local

[sqli]
enabled = true  
filter = sqli
logpath = /var/log/apache2/access.log
maxretry = 2
findtime = 600
bantime = 3600
```

```bash
# Step 4: Start service
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Step 5: Monitor
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo tail -f /var/log/fail2ban.log
```

### Testing

```bash
# Simulate failed SSH logins
for i in {1..4}; do
  ssh -p 2222 baduser@target &
done

# Verify ban
sudo fail2ban-client status sshd
# Should show banned IP

# Check iptables rules
sudo iptables -L | grep f2b
```

### Deliverables
- [ ] jail.local configuration
- [ ] Custom filter definitions
- [ ] Screenshots of:
  - Active bans (fail2ban-client status)
  - Iptables rules
  - Log file entries
- [ ] Monitoring report

---

## Exercise W4-5: Network Scanning & Vulnerability Assessment

**Time: 90 minutes | Difficulty: Intermediate**

### Objectives
- Perform port scanning
- Identify services
- Run vulnerability scans
- Create remediation plan

### Scanning Tasks

```bash
# Step 1: Basic Nmap scan
nmap -sV -sC -O target-network

# Step 2: Service-specific scans
nmap -p- --open target  # All open ports
nmap -sU target  # UDP ports
nmap -n target  # No DNS resolution

# Step 3: Timing and output
nmap -T4 -A -oA results target
# Results in: results.nmap, results.xml, results.gnmap

# Step 4: Vulnerability scanning
# Try Nessus, OpenVAS, or Nikto
nikto -h target
```

### Analysis

```bash
# Parse Nmap results
grep "open" results.nmap

# Identify services
grep "Service" results.xml

# Cross-reference CVEs
# For each service version found, check:
# https://www.cvedetails.com/
```

### Deliverables
- [ ] Nmap XML output
- [ ] Service inventory table
- [ ] Vulnerability list with:
  - Service name/version
  - Identified CVEs
  - CVSS scores
  - Remediation steps
- [ ] Scan report (2-3 pages)

---

## Exercise W4-6: Comprehensive Weekly Assessment

**Time: 180 minutes | Difficulty: Advanced**

### Objective: Complete hardened server deployment

Deploy a fully hardened web server with:

**Requirements:**
- ✓ Hardened SSH (key-based, strong crypto, audit logging)
- ✓ UFW firewall with specific rules
- ✓ Fail2Ban protection
- ✓ Web server (Nginx) running securely
- ✓ TLS/HTTPS configured
- ✓ System monitoring enabled
- ✓ Security audit completed

### Implementation

```bash
# Create deployment script
cat > hardened-deploy.sh << 'EOF'
#!/bin/bash
set -e

echo "Starting hardened server deployment..."

# 1. Update system
sudo apt update && sudo apt full-upgrade -y

# 2. Harden SSH
# (apply configurations from Exercise W4-1)

# 3. Configure firewall
# (apply configurations from Exercise W4-2)

# 4. Install Fail2Ban
# (apply configurations from Exercise W4-4)

# 5. Install web server
sudo apt install -y nginx

# 6. Configure HTTPS
# Generate self-signed cert for testing
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/server.key \
  -out /etc/ssl/certs/server.crt

# 7. Nginx hardening
# [apply security headers, disable server signature, etc.]

# 8. Enable monitoring
sudo systemctl enable ssh nginx auditd fail2ban

echo "Deployment complete!"
EOF

chmod +x hardened-deploy.sh
./hardened-deploy.sh
```

### Verification Checklist

Create a comprehensive verification report verifying:
- [ ] SSH accessible only via keys
- [ ] No services exposed except allowed
- [ ] HTTPS working with strong ciphers
- [ ] Fail2Ban active and protecting
- [ ] Audit logs capturing security events
- [ ] Monitoring alerts configured
- [ ] All hardening requirements met

### Deliverables
- [ ] Deployment script (fully automated)
- [ ] Hardened server configuration bundle
- [ ] Verification report (2000+ words)
- [ ] Screenshots of:
  - All hardening measures in place
  - Security scan results (before/after)
  - Working HTTPS connection
  - Active monitoring dashboard
- [ ] Remediation checklist (100% complete)

---

## Week 4 Capstone: Professional Security Report

**Consolidate all Week 4 exercises into a single professional report:**

### Report Structure

1. **Executive Summary** (2 pages)
   - Hardening scope
   - Key improvements
   - Risk reduction

2. **Technical Implementation** (10 pages)
   - SSH hardening details
   - Firewall architecture
   - Fail2Ban configuration
   - Web server hardening

3. **Verification Results** (5 pages)
   - Security scan (before/after)
   - Penetration test results
   - Configuration validation

4. **Operations** (5 pages)
   - Monitoring setup
   - Alert thresholds
   - Incident response procedures

5. **Maintenance** (3 pages)
   - Update procedures
   - Backup strategy
   - Disaster recovery

### Submission

- [ ] Professional report (25-30 pages)
- [ ] All configuration files
- [ ] Supporting evidence (logs, screenshots)
- [ ] Presentation slides (if required)

---

## Week 4 Summary

By end of Week 4, you should:
✓ Understand multi-layer defense strategy  
✓ Implement defense-in-depth with SSH hardening  
✓ Configure firewalls for specific requirements  
✓ Deploy intrusion prevention (Fail2Ban)  
✓ Perform security assessment  
✓ Create actionable hardening plans  

**Estimated Effort: 8-10 hours**  
**Difficulty Progression: ⭐⭐⭐ Intermediate**
