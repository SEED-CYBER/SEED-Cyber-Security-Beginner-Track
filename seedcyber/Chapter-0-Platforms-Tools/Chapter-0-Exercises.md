# Chapter 0 Exercises

**Deadline:** Before Week 1 begins  
**Estimated Time:** 7-10 hours  
**Deliverable:** Complete documentation file

## Exercise Overview

These exercises ensure you have a complete, working setup before starting the main curriculum. All exercises build on the modules you just studied.

---

## Exercise 1: Cloud Account & VPS Deployment

**Time Estimate:** 2-3 hours  
**Objective:** Deploy your first VPS instance and verify connectivity

### Task 1.1: Choose & Create Cloud Account

Using the guidance from [Module 0](Part-0-Introduction-and-Cloud-Providers.md):

1. **Choose a provider** (recommended: DigitalOcean or Linode)
2. **Create an account** with your email
3. **Set up billing** (verify you have free credits or payment method)
4. **Note your account details** (safely store username/password)

**Documentation (Screenshot Required):**
- Take screenshot of your cloud provider dashboard home page
- Include provider name and any free credits shown
- File name: `1-1-cloud-account.png`

### Task 1.2: Deploy VPS Instance

**Configuration:**
- **OS Image:** Ubuntu 20.04 LTS or 22.04 LTS
- **Region:** Choose geographically close to you
- **Instance Type:** Minimal (1 vCPU, 1-2GB RAM, 20GB storage)
- **Firewall:** Enable (allow SSH port 22)
- **SSH Key:** Generate new key during deployment OR add existing key

1. Navigate to VPS deployment section
2. Follow provider's wizard
3. Select configurations above
4. Deploy instance
5. Wait 1-2 minutes for startup

**Documentation (Screenshot Required):**
- Deployment confirmation page
- Instance details showing:
  - IP address
  - Hostname
  - Specifications (CPU, RAM)
  - Status (Running)
- File name: `1-2-vps-deployed.png`

### Task 1.3: Verify VPS is Accessible

```bash
# From your local machine, test SSH access
ssh -i ~/.ssh/key.pem root@your.vps.ip.address

# If prompted about host key fingerprint:
# Type 'yes' to accept

# You should see the Ubuntu prompt
root@ubuntu-server:~#
```

**If it works:**
- Type: `exit` to disconnect
- This VPS is now your personal practice server

**Documentation:**
- Save the connection command used: `ssh -i ~/.ssh/key.pem root@YOUR.IP.HERE`
- Note it down for next exercise

---

## Exercise 2: Local Development Environment Setup

**Time Estimate:** 1-2 hours  
**Objective:** Configure your personal computer as the control center

### Task 2.1: SSH Key Pair Generation

```bash
# On your LOCAL machine (not the VPS)

# Generate key if you don't have one
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -C "your_email@example.com"

# When prompted:
# Enter passphrase: [create strong passphrase, e.g., Tr0pic@lStorm#2026]
# Confirm passphrase: [repeat it]
```

**Verification:**
```bash
# List keys
ls -la ~/.ssh/id_rsa*

# Should see:
# -rw------- id_rsa          (private - SECRET)
# -rw-r--r-- id_rsa.pub      (public - share)
```

**Documentation:**
```
Key Path: ~/.ssh/id_rsa
Key Type: RSA 4096-bit
Passphrase: [saved securely]
Date Created: [today's date]
```

### Task 2.2: Create SSH Config File

Create file: `~/.ssh/config`

```bash
# For Linux/macOS
nano ~/.ssh/config

# For Windows (PowerShell)
notepad $env:USERPROFILE\.ssh\config
```

Add this content (customize for YOUR VPS):

```
Host vps-training
    HostName your.vps.ip.address
    User ubuntu
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

**Verification:**
```bash
# Test connection
ssh vps-training

# Should connect without typing full command
```

**Documentation:**
- Save your SSH config file content
- File name: `2-2-ssh-config.txt`

### Task 2.3: Shell Environment Setup

Choose and verify your terminal:

**Linux/macOS:**
```bash
# Check default shell
echo $SHELL
# /bin/bash or /bin/zsh

# Both are fine for this course
```

**Windows - Choose One:**

**Option A: Git Bash**
```bash
# Download from: https://gitforwindows.org
# Install with default options
# Verify:
git bash --version
```

**Option B: WSL 2 (Recommended)**
```powershell
# In PowerShell as Administrator:
wsl --install
# Restart computer
# Open WSL terminal, you're good to go
```

**Documentation:**
- Shell type: [Bash/Zsh/WSL/Git Bash]
- Version: [output of shell --version]
- File name: `2-3-shell-setup.txt`

### Task 2.4: Text Editor for Configuration

Test at least one editor:

```bash
# Test Nano
nano ~/test.txt
# Type "Hello"
# Ctrl+O (save)
# Ctrl+X (exit)
# Verify file exists: cat ~/test.txt

# OR test Vim
vim ~/test.txt
# Press 'i' to insert
# Type "Hello"
# Press ESC
# Type ':wq' and Enter
```

**Documentation:**
```
Preferred Editor: [nano/vim/VS Code]
Reason: [why you chose it]
Test Result: [Successfully edited test file]
```

---

## Exercise 3: Linux Commands Practice

**Time Estimate:** 1-2 hours  
**Objective:** Verify comfort with essential Linux commands

### Task 3.1: File Operations

On your VPS, execute these commands and save outputs:

```bash
# Connect to VPS first
ssh vps-training

# File operations
mkdir -p ~/sec-practice/week1
cd ~/sec-practice/week1
touch exercise1.txt
ls -la

# Save output (copy from terminal)
```

**Documentation - save output of:**
1. `pwd`
2. `ls -la`
3. `whoami`
4. `uname -a`

File name: `3-1-file-operations.txt`

### Task 3.2: Permissions Practice

```bash
# Still on VPS

# Create test file
echo "This is a secret" > secret.txt
echo "This is public" > public.txt

# Change permissions
chmod 600 secret.txt    # Only your can read
chmod 644 public.txt    # You read/write, others read

# View permissions
ls -la secret.txt
ls -la public.txt
```

**Documentation: save output of:**
1. `ls -la secret.txt` (should show rw-------)
2. `ls -la public.txt` (should show rw-r--r--)

File name: `3-2-permissions.txt`

### Task 3.3: Viewing & Searching

```bash
# On VPS

# Create sample file
cat > sample.log << 'EOF'
2025-02-20 10:00:01 INFO User logged in
2025-02-20 10:05:23 ERROR Connection timeout
2025-02-20 10:10:45 INFO Process completed
2025-02-20 10:15:12 ERROR Authentication failed
2025-02-20 10:20:33 INFO User logged out
EOF

# View file
cat sample.log
head -n 3 sample.log
tail -n 2 sample.log

# Search for patterns
grep "ERROR" sample.log
grep -c "INFO" sample.log    # Count lines
```

**Documentation:**
1. Output of `grep "ERROR" sample.log` (should show 2 lines)
2. Output of `grep -c "INFO" sample.log` (should show 2)

File name: `3-3-viewing-searching.txt`

---

## Exercise 4: Tools Verification

**Time Estimate:** 1-2 hours  
**Objective:** Confirm all tools from Module 3 are accessible

### Task 4.1: Network Tools

On your VPS:

```bash
# Check if tools are installed
which curl wget nmap

# If any are missing, install:
sudo apt update
sudo apt install curl wget nmap

# Test them
curl -V                      # Show version
wget --version               # Show version
nmap --version              # Show version
```

**Documentation:**
```
curl version: [output]
wget version: [output]
nmap version: [output]
Installation date: [today]
```

### Task 4.2: System Administration Tools

```bash
# Check tool availability
systemctl --version
journalctl --version
sudo -V

# These should all work and show versions
```

**Documentation:**
```
systemctl: [available/missing]
journalctl: [available/missing]
sudo: [available/missing]
```

### Task 4.3: SSH Tools

```bash
# Local machine
ssh -V                       # OpenSSH version
ssh-keygen -?               # Help (should work)
scp -?                      # SCP help

# All should work without errors
```

**Documentation:**
```
ssh version: [output]
All SSH tools available: [yes/no]
```

---

## Exercise 5: Shared VPS Architecture Understanding

**Time Estimate:** 30 minutes  
**Objective:** Confirm you understand isolation and permissions

### Task 5.1: Workspace Directory Structure

```bash
# After getting assignment details from supervisor

# Check your workspace exists
ls -la /var/www/your_username/

# Check home directory
ls -la ~

# Try to access another student's directory (you shouldn't be able to)
ls -la /var/www/other_student/  # Permission denied (expected!)
```

**Documentation:**
1. Output of `ls -la /var/www/your_username/`
2. Output showing permission denied for other student workspace
3. Your assigned username
4. Your assigned port range

File name: `5-1-workspace-structure.txt`

### Task 5.2: Isolation Verification

```bash
# Check you can read your own files
cat ~/.bashrc

# Try directory listing of others (should fail)
sudo ls -la /home/other_student/  # Even sudo doesn't allow it
```

**Documentation:**
- Screenshot or output showing:
  - Your home directory is accessible
  - Attempt to access other students' directory fails
- File name: `5-2-isolation-verification.txt`

---

## Exercise 6: Credentials & Security Setup

**Time Estimate:** 30 minutes  
**Objective:** Secure your access methods

### Task 6.1: Change VPS Password

On the VPS:

```bash
# Change your initial password
passwd

# Prompt:
# Current password: [enter current password]
# New password: [enter NEW strong password]
# Retype new password: [confirm]

# Example of strong password:
# ✓ Tr0p!c@lThunder#2026Sec

# Password should:
# - Be 16+ characters
# - Mix uppercase, lowercase, numbers, symbols
# - NOT contain your username
# - NOT contain dictionary words
```

**Documentation:**
```
Password changed: [date]
Strength level: [Strong/Very Strong]
Note: Password stored securely, not shared
```

### Task 6.2: SSH Key Protection

On local machine:

```bash
# Verify key permissions
ls -la ~/.ssh/id_rsa

# Should show:
# -rw------- (600 permissions)

# If not, fix it:
chmod 600 ~/.ssh/id_rsa

# Verify
ls -la ~/.ssh/id_rsa
```

**Documentation:**
```
Private key permissions: 600 (rw-------)
Public key permissions: 644 (rw-r--r--)
Key passphrase: Set to strong value
Last key rotation: [today]
```

---

## Exercise 7: Documentation & Final Verification

**Time Estimate:** 1 hour  
**Objective:** Create deliverable showing complete setup

### Compile Your Deliverable

Create a single document with all findings:

**File name:** `Chapter-0-SUBMISSION.md`

**Contents:**

```markdown
# Chapter 0 Completion Submission

**Student Name:** [Your Name]
**Date:** [Today's Date]
**Submission Date:** [Today]

## 1. Cloud Provider & VPS
- Provider: [DigitalOcean/Linode/AWS/etc]
- VPS IP: [Your IP Address]
- Ubuntu Version: 20.04 LTS / 22.04 LTS
- Instance Size: [CPU/RAM/Storage]
- Status: ✓ Running and verified

## 2. Local Environment
- SSH Keys: ✓ Generated (4096-bit RSA)
- SSH Config: ✓ Created at ~/.ssh/config
- Shell: [bash/zsh/WSL/Git Bash]
- Editor: [nano/vim/VS Code]

## 3. Linux Fundamentals
- File Operations: ✓ Tested and working
- Permissions: ✓ Understood (chmod working)
- Viewing Files: ✓ cat/head/tail/grep functional
- System Info: ✓ whoami, uname, df working

## 4. Tools Verification
- Network Tools (curl, wget, nmap): ✓ Installed
- System Tools (systemctl, journalctl, sudo): ✓ Working
- SSH Tools (ssh, scp, ssh-keygen): ✓ Available

## 5. Shared VPS Setup
- Assigned Username: [your_username]
- Workspace Location: /var/www/[your_username]/
- Port Range: [8XXX-8XXX and 8XXX-8XXX]
- Isolation: ✓ Verified

## 6. Security
- VPS Password: ✓ Changed to strong value
- SSH Key Passphrase: ✓ Set to strong value
- File Permissions: ✓ Properly secured

## 7. Test Connections
### Via SSH Config:
```
$ ssh vps-training
[Successfully connects]
$
```

### Linux Commands:
```
$ whoami
your_username
$ pwd
/home/your_username
$ ls -la /var/www/your_username
[Shows your workspace]
```

## 8. Questions or Issues
[List any problems encountered and how they were resolved]

## 9. Ready for Week 1
✓ All exercises completed
✓ All tools verified
✓ Workspace confirmed
✓ Ready to begin!
```

### Task 7.1: Create Submission Document

Compile all your evidence:
- Screenshots from exercises
- Command outputs
- Configuration details
- Any issues and resolutions

### Task 7.2: Final System Check

```bash
# One final test
ssh vps-training
whoami
pwd
ls -la /var/www/$(whoami)/

# Should all work without issues
exit
```

**Documentation:**
- Full output of above commands
- File name: `7-2-final-verification.txt`

---

## Submission Checklist

Before submitting to supervisor, verify:

### Documentation Check
- [ ] All 7 exercises completed  
- [ ] All file names correct
- [ ] All command outputs captured
- [ ] Screenshots included
- [ ] Submission document complete

### Technical Check
- [ ] SSH connection works
- [ ] Workspace assigned and accessible
- [ ] File permissions correct
- [ ] Tools installed and tested
- [ ] Password changed and secured

### Security Check
- [ ] VPS password changed
- [ ] SSH key secured (600 permissions)
- [ ] No passwords in documents
- [ ] No credentials shared
- [ ] Backup of SSH keys created

---

## Submission Instructions

1. **Gather all files:** Create folder `Chapter-0-Submission/`
2. **Include all exercise outputs** (screenshots, txt files, etc.)
3. **Include the compilation document:** `Chapter-0-SUBMISSION.md`
4. **Verify completeness** using the checklist above
5. **Submit to supervisor** via email or course platform

## Next Steps After Submission

Once supervisor approves:
1. Receive full access credentials (if not already done)
2. Review supervisor feedback
3. Address any issues flagged
4. **Begin Week 1: Foundation – Linux & Internet Infrastructure**

---

**Congratulations!** Completing Chapter 0 means you're ready for the main cybersecurity curriculum!
