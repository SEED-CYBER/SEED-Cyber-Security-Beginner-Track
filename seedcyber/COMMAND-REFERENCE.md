# Cybersecurity Command Reference Guide

Quick reference for common security commands used throughout the seedcyber curriculum.

---

## Linux/WSL Essential Commands

### System Information

```bash
# Current user and privileges
whoami
id
sudo whoami

# System details
uname -a
hostnamectl
lsb_release -a

# Disk space
df -h
du -sh /path/to/directory

# Memory
free -h
top -b -n 1

# Uptime
uptime
```

### User & Group Management

```bash
# Create user
sudo useradd -m -s /bin/bash username
sudo useradd -m -s /bin/false serviceaccount

# Add to group
sudo usermod -aG groupname username
sudo usermod -aG sudo username

# Change password
sudo passwd username
sudo passwd

# Delete user
sudo userdel -r username

# Create group
sudo groupadd groupname

# List groups
getent group
id username

# Switch user
su - username
sudo -i

# List sudoers
sudo visudo
```

### File Permissions

```bash
# Change permissions (numeric)
chmod 755 file.txt   # rwxr-xr-x
chmod 644 file.txt   # rw-r--r--
chmod 600 file.txt   # rw-------

# Change ownership
chown user:group file.txt
chown user file.txt
chown :group file.txt

# Recursive change
chmod -R 755 /directory
chown -R user:group /directory

# View permissions
ls -la
stat file.txt

# Umask (default permissions)
umask
umask 0022  # Sets default to 755
```

### Network Commands

```bash
# IP configuration
ip addr show
ip route show
ifconfig (older systems)

# Ping
ping -c 4 host.com  # Linux/WSL
ping -n 4 host.com  # Windows

# DNS lookup
nslookup host.com
dig host.com
host host.com

# Netstat (view connections)
netstat -tulpn
netstat -an | grep LISTEN

# Modern equivalent
ss -tulpn
ss -an | grep LISTEN

# Traceroute
traceroute host.com
tracert host.com  # Windows

# Network monitoring
tcpdump -i eth0
tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap

# Bandwidth checking
iftop
nethogs
```

### SSH & Keys

```bash
# Generate SSH key
ssh-keygen -t ed25519 -f ~/.ssh/id_key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# SSH to host
ssh user@host
ssh -i ~/.ssh/key.pem user@host
ssh -p 2222 user@host

# Copy files via SSH
scp file.txt remote:/path/
scp -r directory/ remote:/path/
scp -P 2222 remote:/file.txt local/

# SSH config
nano ~/.ssh/config
# Create shortcuts

# SSH agent
eval $(ssh-agent)
ssh-add ~/.ssh/key

# View public key
cat ~/.ssh/id_rsa.pub

# SSH fingerprint
ssh-keygen -l -f ~/.ssh/id_rsa.pub
```

### File Operations

```bash
# List files
ls -la
ls -lah /directory/

# Find files
find / -name "file.txt"
find ~ -name "*.log" -mtime -7
find / -type f -perm /u+s  # SUID files

# Search content
grep "term" file.txt
grep -r "term" /directory/
grep -i "term" file.txt  # Case insensitive
grep -E "term1|term2" file.txt  # Regex

# File content
cat file.txt
head -20 file.txt
tail -20 file.txt
tail -f file.txt  # Follow (live)

# Text processing
sed 's/old/new/g' file.txt
awk '{print $1, $3}' file.txt
cut -d: -f1,3 /etc/passwd

# Check file type
file file.txt
stat file.txt

# Compare files
diff file1.txt file2.txt
```

---

## Security Testing Commands

### Nmap Scanning

```bash
# Basic scan
nmap target.com

# Scan specific ports
nmap -p 22,80,443 target.com

# All ports
nmap -p- target.com

# Service detection
nmap -sV target.com

# OS detection
nmap -O target.com

# Aggressive scan
nmap -A -T4 target.com
```

### Web Security Testing

```bash
# curl - HTTP requests
curl http://target.com
curl -X POST -d "data" http://target.com
curl -H "Header: value" http://target.com

# sqlmap - SQL injection testing
sqlmap -u "http://target/page?id=1" -p id

# nikto - Web server scanning
nikto -h http://target.com
```

---

## Quick Tips

- **Always verify you have authorization before testing**
- **Backup important files before making changes**
- **Log all security testing activities**
- **Keep security tools updated**
- **Test in isolated environments first**

For detailed explanations, refer to the chapter modules.
