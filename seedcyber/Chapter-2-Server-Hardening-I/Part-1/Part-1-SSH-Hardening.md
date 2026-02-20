# Part 1: SSH Security Hardening

## Current SSH Vulnerabilities

By default, SSH allows:
- Root login (most privileged account exposed)
- Password authentication (brute-force attacks possible)
- Empty passwords
- Multiple authentication attempts

## SSH Configuration File

Located at: `/etc/ssh/sshd_config`

### Critical Settings

```bash
# Disable root login
PermitRootLogin no

# Require key-based authentication
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no

# Disable X11 forwarding
X11Forwarding no

# Limit authentication attempts
MaxAuthTries 3
MaxSessions 10

# Change default port (optional)
Port 22  # or change to 2222, etc

# Restrict listening address
ListenAddress 0.0.0.0
ListenAddress ::

# Protocol version
Protocol 2

# Use strong ciphers
Ciphers aes256-ctr,aes128-ctr

# Banner
Banner /etc/ssh/banner.txt
```

### Hardening Steps

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Make changes above

# Test configuration
sudo sshd -t

# Restart SSH daemon
sudo systemctl restart sshd

```

## SSH Key Strength

```bash
# Generate strong key locally
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -C "user@host"

# Deploy to server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@vps.ip
```

## Disabling Password Auth

Once keys are deployed:

```bash
# On VPS, edit SSH config
PasswordAuthentication no

# Restart SSH
sudo systemctl restart sshd
```

## Monitoring SSH Attempts

```bash
# View authentication logs
sudo tail -f /var/log/auth.log

# Failed login attempts
sudo grep "Failed" /var/log/auth.log

# Count failed attempts
sudo grep "Failed" /var/log/auth.log | wc -l
```

---

**Next:** [Part 2: Firewall Configuration](../Part-2/Part-2-Firewall-Configuration.md)
