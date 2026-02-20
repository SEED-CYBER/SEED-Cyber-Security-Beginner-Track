# Module 3: Security Tools Overview

## Introduction to Cybersecurity Tools

Throughout this 6-week internship, you'll use professional security tools used by real security engineers. This module introduces them without going deep—details come in each week's modules.

## Network Tools

### Nmap (Network Mapper)

Purpose: Discover hosts and services, identify open ports

```bash
# Basic scan (discover open ports)
nmap target.com

# Scan specific range
nmap 192.168.1.0/24

# Service version detection
nmap -sV target.com

# Full TCP scan
nmap -sS target.com

# Syn scan (stealthier)
nmap -sS target.com

# Output to file
nmap target.com -oN results.txt
```

**When you'll use it:** Week 2 (firewall testing), Week 5 (reconnaissance)

### Wireshark

Purpose: Capture and analyze network traffic packet-by-packet

```bash
# Install
sudo apt install wireshark

# Run Wireshark (GUI)
wireshark &

# Command-line capture (tshark)
sudo tshark -i eth0              # Capture on interface
sudo tshark -i eth0 -w packets.pcap  # Save to file
```

**When you'll use it:** Week 4 (monitoring traffic), Week 5 & 6 (analyzing attacks)

### Netstat

Purpose: Show network connections and listening ports

```bash
# List listening ports
netstat -tlnp

# Show all connections
netstat -an

# Modern alternative (ss command)
ss -tlnp
ss -an
```

**When you'll use it:** Week 1 & 2 (verify network configuration)

### Curl & Wget

Purpose: Transfer data using HTTP/HTTPS, test web services

```bash
# Fetch webpage
curl https://example.com

# Save to file
curl https://example.com -o file.html

# Send POST request
curl -X POST -d "data" https://example.com

# With headers
curl -H "Authorization: Bearer token" https://api.example.com

# Wget alternative
wget https://example.com/file.zip
```

**When you'll use it:** Week 3 (test Nginx), Week 6 (web testing)

## System Administration Tools

### Systemctl

Purpose: Manage services and system startup

```bash
# Start/stop services
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Enable at boot
sudo systemctl enable nginx

# Check status
systemctl status nginx

# View logs
journalctl -u nginx -f
```

**When you'll use it:** Week 1 (start services), Week 3 (mange web stack)

### Journalctl

Purpose: Query system logs and journaling service

```bash
# Recent logs
journalctl -n 50

# Specific service
journalctl -u nginx

# Follow logs (like tail -f)
journalctl -f

# By date
journalctl --since "2 hours ago"

# Search for errors
journalctl -p err
```

**When you'll use it:** Week 1 & 4 (system troubleshooting), Week 4 (monitoring)

### Auditd

Purpose: Comprehensive system auditing and compliance

```bash
# Install
sudo apt install auditd

# Start audit daemon
sudo systemctl start auditd

# Check rules
sudo auditctl -l

# View audit logs
sudo tail -f /var/log/audit/audit.log
```

**When you'll use it:** Week 3 (audit services), Week 4 (comprehensive monitoring)

## SSH & Access Control

### SSH Protocol

Purpose: Secure remote command execution and file transfer

```bash
# SSH to server
ssh user@hostname

# With specific key
ssh -i ~/.ssh/key.pem user@hostname

# Execute remote command
ssh user@hostname "whoami"

# Scp (secure copy)
scp file.txt user@host:/remote/path
scp -r user@host:/remote/dir local/dir

# SFTP (secure file transfer)
sftp user@host
```

**When you'll use it:** Week 1 (connect to VPS), every week (remote work)

### Ssh-keygen

Purpose: Generate cryptographic keys for passwordless authentication

```bash
# Generate key pair
ssh-keygen -t rsa -b 4096

# Generate key with comment
ssh-keygen -t rsa -b 4096 -C "work-key"

# Get public key
cat ~/.ssh/id_rsa.pub
```

**When you'll use it:** Week 1 (setup), Key management throughout

### Sudo

Purpose: Execute commands with elevated privileges

```bash
# Run command as root
sudo command

# View available commands
sudo -l

# Run as specific user
sudo -u username command

# Edit files as root (careful!)
sudo nano /etc/file.conf
```

**When you'll use it:** Every week (nearly every task requires sudo)

## Security Frameworks & Scanners

### OpenVAS (Open Vulnerability Assessment System)

Purpose: Automated vulnerability scanning and reporting

```bash
# Installation and deployment
# Usually run as web-based app
# Scan targets and generates detailed vulnerability reports
```

**When you'll use it:** Week 5 (vulnerability assessment)

### Metasploit Framework

Purpose: Exploitation framework for penetration testing

```bash
# Start Metasploit console
msfconsole

# Search for exploits
search cve-number

# Use specific exploit
use module/exploit/path

# Show options
show options

# Run exploit
run
```

**When you'll use it:** Week 5 & 6 (controlled exploitation exercises)

### Burp Suite Community

Purpose: Web application security testing proxy

```bash
# Install from official website
# https://portswigger.net/burp/community

# Launch Burp
java -jar burpsuite_community.jar

# Configure browser proxy to Burp (localhost:8080)
# Intercept and modify web requests
```

**When you'll use it:** Week 6 (web application testing)

## Monitoring & Logging

### ELK Stack (Elasticsearch, Logstash, Kibana)

Purpose: Centralized log collection, analysis, and visualization

- **Elasticsearch:** Stores and indexes logs
- **Logstash:** Collects and processes logs
- **Kibana:** Visualizes logs in dashboards

**When you'll use it:** Week 4 (implement monitoring stack)

### Prometheus

Purpose: Metrics collection and time-series database

```bash
# Prometheus configuration
# Scrapes metrics from targets
# AlertManager sends alerts based on rules
```

**When you'll use it:** Week 4 (monitoring system metrics)

## Firewall & IDS/IPS

### UFW (Uncomplicated Firewall)

Purpose: Simple firewall rules on Ubuntu

```bash
# Enable UFW
sudo ufw enable

# Allow port
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Deny port
sudo ufw deny 23/tcp

# Status and rules
sudo ufw status
sudo ufw status verbose

# Delete rule
sudo ufw delete allow 22/tcp
```

**When you'll use it:** Week 2 (firewall hardening)

### Iptables

Purpose: Advanced firewall rules and packet filtering

```bash
# View current rules
sudo iptables -L -n

# Allow incoming HTTP
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Save rules
sudo iptables-save > /etc/iptables.rules
```

**When you'll use it:** Week 2 (advanced filtering)

### Fail2Ban

Purpose: Prevent brute-force attacks by monitoring and blocking

```bash
# Install
sudo apt install fail2ban

# Start service
sudo systemctl start fail2ban

# Check status
sudo systemctl status fail2ban

# View blocked IPs
sudo fail2ban-client status sshd
```

**When you'll use it:** Week 2 (brute-force protection)

## Web Servers & TLS

### Nginx

Purpose: High-performance web server and reverse proxy

```bash
# Install
sudo apt install nginx

# Start service
sudo systemctl start nginx

# Configuration location
/etc/nginx/nginx.conf

# Test configuration
sudo nginx -t

# Reload config without restart
sudo systemctl reload nginx
```

**When you'll use it:** Week 3 (deploy hardened web stack)

### Let's Encrypt & Certbot

Purpose: Free SSL/TLS certificates for HTTPS

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot certonly --standalone -d example.com

# Renew certificate
sudo certbot renew
```

**When you'll use it:** Week 3 (enable HTTPS)

## Command-Line Utilities

### Grep

Purpose: Search text in files

```bash
grep "search_term" file.txt
grep -r "term" /path/          # Recursive
grep -i "term" file.txt        # Case-insensitive
grep "^pattern" file.txt       # Starts with pattern
```

### Sed

Purpose: Stream editor for text transformation

```bash
sed 's/old/new/g' file.txt     # Replace text
sed -i 's/old/new/g' file.txt  # Replace in place
```

### Awk

Purpose: Extract and format columns from text

```bash
awk '{print $1}' file.txt      # Print first column
awk -F: '{print $1}' /etc/passwd  # Use : as delimiter
```

## Summary: When You'll Use Each Tool

| Week | Primary Tools |
|------|---------------|
| Week 1 | SSH, systemctl, journalctl |
| Week 2 | UFW, iptables, Fail2Ban, Nmap |
| Week 3 | Nginx, Certbot, Auditd |
| Week 4 | Journalctl, ELK/Prometheus, Wireshark |
| Week 5 | Nmap, OpenVAS, Wireshark, Metasploit |
| Week 6 | Burp Suite, Curl, Metasploit |

## Next Steps

- Don't memorize all these tools now
- Refer back to this module when you need tool details
- Each week will teach specific tool usage
- Practice hands-on during exercises

---

**Next:** [Module 4: Shared VPS Architecture](Part-4-Shared-VPS-Architecture.md)
