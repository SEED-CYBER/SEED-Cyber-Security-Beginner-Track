# Part 4: VPS Deployment & SSH Setup

## VPS Initial Configuration

Once your VPS is deployed, perform these initial setup steps.

## Initial Connection

```bash
# First SSH as root (using password or key)
ssh root@your.vps.ip.address

# If prompted about ECDSA key, type 'yes' to accept
```

## System Updates

```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git htop vim nano
```

## Set Hostname

```bash
# Current hostname
hostname

# Change hostname
sudo hostnamectl set-hostname new-hostname

# Edit hosts file
sudo nano /etc/hosts
# Add: 127.0.0.1  new-hostname

# Verify
hostname
```

## Create Regular User Account

```bash
# Create user
sudo useradd -m -s /bin/bash username

# Set password
sudo passwd username

# Add to sudo group
sudo usermod -aG sudo username

# Verify
sudo -u username sudo -l
```

## SSH Key Setup

### Generate Keys Locally

```bash
# On your local machine
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Copy contents of public key
cat ~/.ssh/id_rsa.pub
```

### Deploy Public Key to VPS

```bash
# Method 1: Using ssh-copy-id (automated)
ssh-copy-id -i ~/.ssh/id_rsa.pub username@vps.ip

# Method 2: Manual addition
# On VPS:
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Create/edit authorized_keys
nano ~/.ssh/authorized_keys
# Paste your public key here

# Set permissions
chmod 600 ~/.ssh/authorized_keys
```

### Disable Password Authentication

```bash
# On VPS, edit SSH config
sudo nano /etc/ssh/sshd_config

# Find and change these lines:
# PermitRootLogin no          (disable root login)
# PubkeyAuthentication yes    (enable key auth)
# PasswordAuthentication no   (disable password)
# PermitEmptyPasswords no     (disable empty pass)

# Restart SSH daemon
sudo systemctl restart sshd

# Test new connection (DON'T CLOSE EXISTING SESSION)
# In new terminal:
ssh -i ~/.ssh/id_rsa username@vps.ip
```

## Basic Firewall Setup

```bash
# Enable UFW
sudo ufw enable

# Allow SSH (critical!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# View rules
sudo ufw status

# Remove rule if needed
sudo ufw delete allow 22/tcp
```

## Workspace Directory Structure

### Create Workspace Directories

```bash
# As root or sudo:
sudo mkdir -p /var/www/username/public_html
sudo mkdir -p /var/www/username/logs
sudo mkdir -p /var/www/username/backup

# Set ownership
sudo chown -R username:username /var/www/username

# Set permissions
sudo chmod 750 /var/www/username
sudo chmod 755 /var/www/username/public_html
```

### User Setup in Workspace

```bash
# As the regular user:
cd /var/www/username

# Create basic structure
mkdir -p private
mkdir -p temp

# Create a README
cat > README.md << 'EOF'
# My Security Training Workspace

This is my isolated workspace for cybersecurity training.

## Directory Structure
- public_html/  : Web content
- logs/         : Application logs
- backup/       : Backup files
- private/      : Private files (not web-accessible)
EOF

# Verify
ls -la /var/www/username/
```

## Testing Your Setup

### SSH Connection Test

```bash
# From local machine
ssh -i ~/.ssh/id_rsa username@vps.ip

# Should show:
# username@hostname:~$
```

### Workspace Access

```bash
# Once connected via SSH
pwd
# Should show: /home/username

# Access workspace
cd /var/www/username
ls -la
# Should show your workspace structure
```

### Basic Commands on VPS

```bash
# Check system info
uname -a
df -h
free -h
whoami

# Check uptime
uptime

# See recent logins
last
```

## SSH Client Configuration (Optional)

Create `~/.ssh/config` on your local machine:

```
Host vps-training
    HostName your.vps.ip.address
    User username
    IdentityFile ~/.ssh/id_rsa
    Port 22
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Now you can simply type: `ssh vps-training`

## Workspace Port Assignment

For multi-tenant setup:

```bash
# Your assigned ports
HTTP Port: 8001 (or assigned number)
HTTPS Port: 8401 (or 8000 + 401)

# Configure services to listen on these ports
# (More in Week 3)
```

## Troubleshooting

### Can't SSH?

```bash
# Verify IP is correct
nslookup vps.domain.com

# Test connectivity
ping vps.ip.address

# Check SSH explicitly
ssh -v username@vps.ip
# Look for error messages
```

### Permission Denied?

```bash
# Check key permissions
ls -la ~/.ssh/id_rsa
# Should be: -rw------- (600)

# Check if key is added to agent
ssh-add ~/.ssh/id_rsa
```

### Port Already in Use?

```bash
# Check what's using a port
sudo lsof -i :22
sudo netstat -tlnp | grep 22

# Kill process if needed
sudo kill -9 <PID>
```

---

**Next:** [Week 1 Exercises](../Week-1-Exercises.md)
