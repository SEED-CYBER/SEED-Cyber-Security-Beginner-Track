# Windows & WSL Setup Guide for seedcyber

This guide provides comprehensive instructions for running the seedcyber cybersecurity curriculum on **Windows 10/11** systems using either **WSL2 (Windows Subsystem for Linux)** or **native Windows tools**.

> **Note:** WSL2 is strongly recommended for the best experience. It provides a full Linux environment while maintaining access to Windows tools.

---

## Table of Contents

1. [WSL2 Installation](#wsl2-installation)
2. [Windows PowerShell Setup](#windows-powershell-setup)
3. [Tool Installation Guide](#tool-installation-guide)
4. [Platform-Specific Command Equivalents](#platform-specific-command-equivalents)
5. [Troubleshooting](#troubleshooting-common-issues)

---

## WSL2 Installation

### Prerequisites

- Windows 10 Version 1909+ or Windows 11
- Minimum 4GB RAM (8GB+ recommended)
- Administrative access

### Step 1: Enable Required Features

**Via PowerShell (Administrator):**

```powershell
# Enable required features
Enable-WindowsOptionalFeature -FeatureName Microsoft-Windows-Subsystem-Linux -Online -NoRestart
Enable-WindowsOptionalFeature -FeatureName VirtualMachinePlatform -Online -NoRestart

# Restart required
Restart-Computer
```

**Or via Command Prompt (Administrator):**

```cmd
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --set-default-version 2
```

### Step 2: Install Linux Distribution

```powershell
# List available distributions
wsl --list --online

# Install Ubuntu 22.04 (recommended)
wsl --install -d Ubuntu-22.04

# Or from Microsoft Store
# Open Microsoft Store > Search "Ubuntu" > Install "Ubuntu 22.04 LTS"
```

### Step 3: Initial Setup

```bash
# WSL will launch automatically after installation
# Create login credentials when prompted

# Set as default
wsl --set-default Ubuntu-22.04

# Update package lists
sudo apt update && sudo apt upgrade -y

# Install essentials (in WSL terminal)
sudo apt install -y git curl wget nano build-essential python3-pip
```

### Step 4: Install Docker & Tools

```bash
# Docker Desktop for Windows (GUI option)
# Download from: https://www.docker.com/products/docker-desktop

# Or in WSL:
sudo apt install -y docker.io
sudo usermod -aG docker $USER

# Verify installation
docker --version
python3 --version
git --version
```

---

## Windows Native Alternative

If you prefer not to use WSL2, most tools have Windows versions:

### Git

```powershell
# Install Git for Windows
# Download from: https://git-scm.com/download/win
# Or use Chocolatey:
choco install git -y
```

### Python

```powershell
# Download Python 3.10+ from python.org
# Or use Chocolatey:
choco install python -y

# Verify installation
python --version
pip --version
```

### SSH

```powershell
# OpenSSH is built into Windows 10+ (PowerShell 7+)
# Or install Git Bash which includes OpenSSH

# Generate SSH key
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\admin_key
```

---

## Tool Installation Guide

### Linux (Ubuntu via WSL2) - PREFERRED

```bash
#!/bin/bash
# Complete toolchain installation for seedcyber

# Update system
sudo apt update && sudo apt upgrade -y

# Security & Network Tools
sudo apt install -y \
  nmap \
  netcat \
  curl \
  wget \
  dnsutils \
  whois \
  tcpdump \
  iptables \
  ufw \
  openssl

# System Administration
sudo apt install -y \
  openssh-server \
  openssh-client \
  vim \
  nano \
  htop \
  systemctl \
  auditd

# Monitoring & Logging
sudo apt install -y \
  sysstat \
  logwatch \
  fail2ban

# Development
sudo apt install -y \
  git \
  build-essential \
  python3-pip \
  python3-dev

# Security Testing
sudo apt install -y \
  hashcat \
  john \
  sqlmap

# Optional but useful
sudo apt install -y \
  tshark \
  wireshark \
  jq

# Install Python packages
pip3 install --upgrade pip
pip3 install requests paramiko pycryptodome scapy

# Verify all installations
echo "=== Verification ==="
nmap --version
curl --version
python3 --version
git --version
```

### Windows PowerShell

```powershell
# Using Chocolatey (https://chocolatey.org/)

# Install Chocolatey (if not already installed)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install tools
choco install -y git python nmap curl putty wireshark 7zip vscode

# Alternative: Use Windows Package Manager (winget)
winget install -e --id Git.Git
winget install -e --id Python.Python.3.10
winget install -e --id Nmap.Nmap
```

---

## Platform-Specific Command Equivalents

### Basic Navigation & File Operations

| Task | Linux/WSL | Windows PowerShell |
|------|-----------|------------------|
| Show current directory | `pwd` | `Get-Location` or `cd` |
| List files | `ls -la` | `Get-ChildItem -Force` or `dir /s` |
| Create directory | `mkdir test` | `New-Item -ItemType Directory -Name test` |
| Create file | `touch file.txt` | `New-Item -ItemType File -Name file.txt` |
| Copy file | `cp file.txt copy.txt` | `Copy-Item -Path file.txt -Destination copy.txt` |
| Delete file | `rm file.txt` | `Remove-Item -Path file.txt` |
| View file | `cat file.txt` | `Get-Content file.txt` |
| Search in file | `grep "term" file.txt` | `Select-String -Pattern "term" file.txt` |

### Network Commands

| Task | Linux/WSL | Windows PowerShell |
|------|-----------|------------------|
| Show IP config | `ip addr` or `ifconfig` | `Get-NetIPAddress` |
| Ping | `ping host.com` | `Test-NetConnection -ComputerName host.com` |
| Port scan | `nmap target.com` | `nmap target.com` (if installed) |
| Check open ports | `netstat -tulpn` | `netstat -ano` or `Get-NetTCPConnection` |
| DNS lookup | `nslookup host.com` | `Resolve-DnsName host.com` |
| HTTP request | `curl http://target` | `Invoke-WebRequest -Uri http://target` |
| SSH connect | `ssh user@host` | `ssh user@host` (PowerShell 7+) |
| Download file | `wget url` | `Invoke-WebRequest -Uri url -OutFile file` |

### System Commands

| Task | Linux/WSL | Windows PowerShell |
|------|-----------|------------------|
| Show processes | `ps auxww` | `Get-Process` |
| Kill process | `kill -9 PID` | `Stop-Process -Id PID -Force` |
| Check disk space | `df -h` | `Get-PSDrive` |
| Show memory | `free -h` | `Get-ComputerInfo \| Select Total*` |
| List services | `systemctl list-units` | `Get-Service` |
| Start service | `sudo systemctl start ssh` | `Start-Service -Name sshd` |
| View logs (auth) | `tail -f /var/log/auth.log` | `Get-WinEvent -FilterHashtable ...` |
| Environment vars | `echo $PATH` | `$env:PATH` |

### Searching & Finding

| Task | Linux/WSL | Windows PowerShell |
|------|-----------|------------------|
| Find file | `find / -name file.txt` | `Get-ChildItem -Path C:\ -Filter file.txt -Recurse` |
| Search content | `grep -r "term" .` | `Select-String -Pattern "term" -Path *.txt -Recurse` |
| Find recently modified | `find . -mtime -1` | `Get-ChildItem \| Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)}` |

### Security & Cryptography

| Task | Linux/WSL | Windows PowerShell |
|------|-----------|------------------|
| SSH key generate | `ssh-keygen -t ed25519` | `ssh-keygen -t ed25519` (PowerShell 7+) |
| Hash file (SHA256) | `sha256sum file.txt` | `Get-FileHash -Path file.txt -Algorithm SHA256` |
| Encrypt file | `gpg -c file.txt` | `Protect-CmsMessage -Content ...` |
| Base64 encode | `echo "text" \| base64` | `[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("text"))` |
| Base64 decode | `echo "b64" \| base64 -d` | `[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("b64"))` |

---

## WSL2-Specific Tips & Tricks

### Running Linux Programs from Windows

```powershell
# Call Linux commands directly from PowerShell
wsl ls -la

# Run script in WSL
wsl bash -c "echo 'Hello from Linux'"

# Mount Windows drives in WSL
# Linux: /mnt/c = C: drive, /mnt/d = D: drive
```

### File System Access

```bash
# Access Windows files from WSL
cd /mnt/c/Users/YourUsername/Desktop
cp /mnt/c/file.txt ~/

# Access WSL files from Windows
# \\wsl$\Ubuntu-22.04\home\username\
# Can be opened in File Explorer
```

### Integrated Development

```powershell
# Launch VS Code with WSL integration
wsl code .

# Launch terminal in WSL
wsl --cd /home/username
```

### Network Access

```bash
# WSL2 has its own IP address
ip addr show eth0 | grep "inet "

# Access WSL services from Windows:
# Use the WSL IP or localhost (localhost forwarding enabled in Windows 11 22419+)

# From Windows: http://localhost:8080 → WSL:8080
# From WSL: curl http://localhost:8080
```

### Systemd in WSL2 (Ubuntu 22.04+)

```bash
# Enable systemd
sudo nano /etc/wsl.conf
# Add:
# [boot]
# systemd=true

# Restart WSL
wsl --shutdown
wsl

# Now systemctl works fully
sudo systemctl status ssh
sudo systemctl start fail2ban
```

---

## Docker on Windows

### Docker Desktop for Windows

```powershell
# Download and install Docker Desktop
# https://www.docker.com/products/docker-desktop

# Verify installation
docker --version
docker run hello-world

# Run Linux containers in WSL2 backend (recommended)
```

### Running Security Tools in Containers

```bash
# Pull vulnerable web app
docker pull vulnerables/web-dvwa

# Run with port mapping
docker run -d -p 8080:80 vulnerables/web-dvwa

# Access: http://localhost:8080

# View logs
docker logs [container_id]

# Stop container
docker stop [container_id]
```

---

## IDE & Editor Setup

### Visual Studio Code

```powershell
# Install VS Code
choco install vscode -y
# Or download from https://code.microsoft.com/

# Essential extensions for security work:
# - Remote - WSL (ms-vscode-remote.remote-wsl)
# - Python (ms-python.python)
# - Markdown Preview (yzhang.markdown-all-in-one)
# - Hex Editor (ms-vscode.hexeditor)
# - REST Client (humao.rest-client)
```

### With WSL Remote

```powershell
# Install Remote Development extension
code --install-extension ms-vscode-remote.remote-wsl

# Open VSCode with WSL
code --remote wsl+Ubuntu-22.04

# All terminal commands run in WSL
```

---

## SSH Setup on Windows

### PowerShell 7+

```powershell
# SSH is built-in to PowerShell 7+
ssh user@remote.com

# Generate key
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\admin_key

# Copy to remote
# If that doesn't work:
Add-Content $env:USERPROFILE\.ssh\authorized_keys "$(Get-Content $env:USERPROFILE\.ssh\admin_key.pub)"
```

### Git Bash

```bash
# Use Git Bash terminal (installed with Git for Windows)
ssh-keygen -t ed25519 -f ~/.ssh/admin_key
ssh user@remote.com
```

### PuTTY (GUI)

```powershell
# Install PuTTY
choco install putty -y

# Or download: https://www.putty.org/

# Create key in PuTTY Key Generator
# Save with .ppk extension
# Load in PuTTY configuration for authentication
```

---

## Troubleshooting Common Issues

### WSL2 Issues

**Problem:** WSL2 not available or slow performance

```powershell
# Check WSL status
wsl --list --verbose

# Update WSL kernel
wsl --update

# Set WSL2 as default
wsl --set-default-version 2

# Check available RAM
wsl -l -v

# If running slowly, configure .wslconfig
# Create: C:\Users\YourName\.wslconfig
# Content:
# [wsl2]
# memory=4GB
# processors=4
# swap=2GB

# Then restart:
wsl --shutdown
```

**Problem:** WSL2 localhost access from Windows

```powershell
# Windows 11 22419+ has localhost forwarding enabled
# For earlier versions, use WSL IP:
wsl hostname -I  # Get WSL IP (usually 172.x.x.x)

# Or set up firewall rules for port forwarding
```

### Tool Installation Issues

**Problem:** Git command not found

```powershell
# Install Git
choco install git -y

# Or add to PATH manually
# Control Panel > System > Advanced > Environment Variables
# Add: C:\Program Files\Git\bin to PATH
```

**Problem:** Python not recognized

```powershell
# Install Python
choco install python -y

# Verify PATH includes Python
[Environment]::GetEnvironmentVariable("Path")

# Add Python to PATH if needed
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Python310", "User")
```

**Problem:** SSH key permissions error

```powershell
# SSH requires specific permissions in WSL
# If you get: "Permissions for '/home/user/.ssh/config' are too open"

wsl chmod 700 ~/.ssh
wsl chmod 600 ~/.ssh/*
wsl chmod 644 ~/.ssh/*.pub
```

### Network Issues

**Problem:** Can't reach external hosts from WSL

```bash
# Check DNS configuration
cat /etc/resolv.conf

# Force nameserver
sudo nano /etc/resolv.conf
# Change to:
# nameserver 8.8.8.8
# nameserver 8.8.4.4

# Or in WSL config (Windows side):
wsl --shutdown
# Edit C:\Users\YourName\.wslconfig
# [wsl2]
# kernelCommandLine = ip=dhcp

# Try restarting network
sudo ip route delete default
sudo ip route add default via $(ip route | grep '^default' | awk '{print $3}')
```

**Problem:** Port already in use

```powershell
# Find process using port
netstat -ano | findstr :8080

# Kill process
taskkill /PID [PID] /F

# Or use different port in application
```

### File System Performance

**Problem:** Slow performance with files in /mnt/c/

```bash
# WSL2 file access to Windows drives is slower
# Keep Linux projects in WSL filesystem: ~/projects/

# Don't do:
cd /mnt/c/Users/Desktop/project  # SLOW

# Do this:
cp -r /mnt/c/Users/Desktop/project ~/
cd ~/project  # FAST
```

---

## Microsoft Defender Integration

### Exclude WSL Directories

WSL directories should be excluded from Windows Defender scanning to improve performance:

```powershell
# Add exclusion
Add-MpPreference -ExclusionPath "\\wsl$\Ubuntu-22.04"

# Verify exclusion
Get-MpPreference | Select-Object ExclusionPath
```

---

## Dual-Boot Alternative

For users who need full Linux performance:

### Linux VM Setup

```powershell
# Option 1: VirtualBox (Free)
choco install virtualbox -y

# Option 2: Hyper-V (Built into Windows Pro/Enterprise)
Enable-WindowsOptionalFeature -FeatureName Hyper-V -Online -NoRestart

# Option 3: VMware Workstation Player (Free for personal use)
# Download from: https://www.vmware.com/products/workstation-player.html
```

### Download Linux ISO

```powershell
# Ubuntu Server (smaller, headless)
# https://ubuntu.com/download/server

# Ubuntu Desktop (full GUI)
# https://ubuntu.com/download/desktop

# Kali Linux (security-focused - for offensive tools)
# https://www.kali.org/get-kali/
```

---

## Summary

| Method | Pros | Cons | Recommendation |
|--------|------|------|-----------------|
| **WSL2** | No overhead, integrated with Windows, good performance | Slight learning curve | ✅ **BEST** |
| **Native PowerShell** | No extra setup | Missing some tools, manual equivalents | Basic tasks only |
| **Git Bash** | Familiar Unix environment | Limited compared to WSL | Good complement to WSL |
| **Linux VM** | Full Linux environment | Overhead, more resources needed | If WSL not available |

---

**Most Recommended Setup:**
1. WSL2 (Ubuntu 22.04) for primary development
2. VS Code with Remote - WSL extension
3. Docker Desktop (WSL2 backend) for containerized tools
4. Git for Windows for git commands

This combination provides the best balance of Linux tooling and Windows integration for the seedcyber curriculum.
