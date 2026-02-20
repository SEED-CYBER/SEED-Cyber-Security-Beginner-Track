# Part 2: Firewall Configuration

## UFW (Uncomplicated Firewall)

Simple interface to configure iptables on Ubuntu.

### Basic UFW Commands

```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Check status
sudo ufw status
sudo ufw status verbose

# Allow port (TCP by default)
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443

# Deny port
sudo ufw deny 23

# Delete rule
sudo ufw delete allow 22
sudo ufw delete allow 80/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Priority rules
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Example Setup

```bash
# Start from scratch
sudo ufw reset

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (critical!)
sudo ufw allow 22/tcp

# Allow web services
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check rules
sudo ufw status numbered

# Enable
sudo ufw enable
```

## Iptables (Advanced)

Direct firewall rule manipulation.

```bash
# View rules
sudo iptables -L -n -v

# Allow input on port 22
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Drop all other input
sudo iptables -A INPUT -j DROP

# Save rules
sudo iptables-save > rules.txt
sudo iptables-save > /etc/iptables/rules.v4
```

## Port Scanning Your Own Rules

```bash
# From external machine
nmap -p 22,80,443 your.vps.ip

# Shows which ports respond

# On VPS, verify listening
sudo ss -tlnp
```

---

**Next:** [Part 3: Fail2Ban Intrusion Prevention](../Part-3/Part-3-Fail2Ban-Intrusion-Prevention.md)
