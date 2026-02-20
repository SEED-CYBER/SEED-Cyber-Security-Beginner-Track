# Part 4 Exercises: VPS Deployment & SSH Setup

**Time:** 4 hours  
**Platform:** All (Windows/macOS/Linux with WSL)

## Exercise 4.1: VPS Initial Setup

**All Platforms (use SSH client):**

```bash
# Windows PowerShell, macOS Terminal, Linux, or WSL
ssh root@your.vps.ip

# Update system
apt update && apt upgrade -y

# Set hostname
hostnamectl set-hostname training-vps

# All platforms verify:
hostname
uname -a
df -h
```

**Deliverable:**
- Screenshot of SSH connection success
- System info output
- Hostname verification

## Exercise 4.2: SSH Key Generation & Deployment

**Local Machine Setup (All Platforms):**

```bash
# Windows PowerShell, macOS Terminal, WSL, Linux
ssh-keygen -t rsa -b 4096 -f ~/.ssh/vps_key -C "student@training"

# View public key
cat ~/.ssh/vps_key.pub

# On VPS:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste contents of vps_key.pub

chmod 600 ~/.ssh/authorized_keys

# Test connection (from local machine)
ssh -i ~/.ssh/vps_key ubuntu@vps.ip
```

**Deliverable:**
- Key pair generated successfully
- SSH config file created
- Public key deployed to VPS
- Passwordless login verified

## Exercise 4.3: Regular User Account Setup

```bash
# On VPS as root/sudo:
sudo useradd -m -s /bin/bash trainee
sudo usermod -aG sudo trainee
sudo passwd trainee

# Create workspace
sudo mkdir -p /var/www/trainee/public_html
sudo mkdir /var/www/trainee/logs
sudo chown -R trainee:trainee /var/www/trainee
sudo chmod 750 /var/www/trainee

# Verify
sudo su - trainee
pwd
ls -la /var/www/trainee/
exit
```

**Deliverable:**
- User creation and sudo access verified
- Workspace directory structure
- Permission verification (ls -la)

## Exercise 4.4: Disable Password Authentication

```bash
# On VPS:
sudo nano /etc/ssh/sshd_config

# Change/add:
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no

# Test config
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd

# Test from local machine (don't close this session first!)
ssh -i ~/.ssh/vps_key trainee@vps.ip
```

**Deliverable:**
- Modified sshd_config file
- Configuration test successful
- Password-based login disabled
- Key-based login functional

## Exercise 4.5: SSH Client Configuration

**Local Machine Setup (All Platforms):**

**Linux/macOS/WSL:**
```bash
nano ~/.ssh/config

# Add:
Host training-vps
    HostName your.vps.ip.address
    User trainee
    IdentityFile ~/.ssh/vps_key
    Port 22
```

**Windows PowerShell:**
```powershell
notepad $env:USERPROFILE\.ssh\config
# Add same content as above
```

**Test:**
```bash
ssh training-vps
```

**Deliverable:**
- SSH config file created
- Simplified SSH connection working
- Connection details documented

## Exercise 4.6: Workspace Testing

```bash
# Connect via SSH
ssh training-vps

# Verify workspace access
cd /var/www/trainee/public_html
pwd
ls -la

# Create test file
echo "<h1>Test Page</h1>" > index.html

# Verify file created
ls -la
cat index.html

# Disconnect
exit
```

**Deliverable:**
- Workspace directory accessible
- Can create files
- File permissions correct
- Workspace assignment confirmed

---

**Completion Checklist:**
- [ ] VPS deployed and updated
- [ ] SSH keys generated (4096-bit RSA)
- [ ] Public key deployed to VPS
- [ ] Regular user created with sudo
- [ ] Workspace assigned and accessible
- [ ] Password authentication disabled
- [ ] SSH config file created
- [ ] Can connect to VPS without typing IP/user

**Submission:** Part-4-Exercises-SUBMISSION.txt
