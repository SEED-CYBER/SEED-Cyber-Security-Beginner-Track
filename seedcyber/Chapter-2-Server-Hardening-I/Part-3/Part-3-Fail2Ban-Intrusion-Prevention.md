# Part 3: Fail2Ban Intrusion Prevention

## What is Fail2Ban?

Monitors logs for suspicious activity and blocks offending IPs automatically.

### Installation

```bash
# Install
sudo apt update
sudo apt install fail2ban

# Start service
sudo systemctl start fail2ban

# Enable on boot
sudo systemctl enable fail2ban

# Check status
sudo systemctl status fail2ban
```

### Configuration

Main config: `/etc/fail2ban/jail.conf`

```bash
# View configuration
sudo cat /etc/fail2ban/jail.conf

# Key settings:
# [DEFAULT]
# bantime = 3600              # Ban for 1 hour
# findtime = 600              # Check last 10 minutes
# maxretry = 5                # Ban after 5 failures

# [sshd]
# enabled = true
# port = ssh
# logpath = /var/log/auth.log
```

### Useful Commands

```bash
# Check failed attempts
sudo fail2ban-client status sshd

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# View banned IPs
sudo iptables -L -n | grep DROP
```

---

**Next:** [Part 4: Network Scanning](../Part-4/Part-4-Network-Scanning.md)
