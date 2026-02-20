# Module 1: Development Tools & Environment Setup

## Your Local Environment

Your personal computer is your **control center** for managing the VPS. Before you deploy anything, ensure your local system is properly configured.

## SSH Client Setup

SSH (Secure Shell) is your primary tool for connecting to your VPS. It provides encrypted, secure command-line access.

### Linux & macOS (Built-in)

SSH comes pre-installed:

```bash
# Check if SSH is installed
ssh -V

# You should see output like: OpenSSH_8.2p1 Ubuntu...
```

### Windows 10+

Windows now includes OpenSSH by default, but verify:

```powershell
# In PowerShell as Administrator
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

# If not installed:
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

**Alternative:** Install Git Bash or WSL for native Linux experience.

## Generating SSH Keys

SSH keys consist of two files:
- **Private key** (keep secret, like a password) - stored on your computer
- **Public key** (share freely) - stored on the VPS

### Generate Your Key Pair

```bash
# Generate RSA 4096-bit key (recommended)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -C "your_email@example.com"

# When prompted for passphrase, create a strong one
# Output:
# Your identification has been saved in ~/.ssh/id_rsa
# Your public key has been saved in ~/.ssh/id_rsa.pub
```

### View Your Public Key

```bash
# Display your public key (you'll paste this onto the VPS)
cat ~/.ssh/id_rsa.pub

# Output looks like:
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDx/yB... your_email@example.com
```

### Protect Your Private Key

```bash
# Set correct permissions (Linux/macOS)
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Verify permissions
ls -la ~/.ssh/
```

## SSH Configuration File

Create a config file to simplify connections:

```bash
# Create/edit ~/.ssh/config

Host vps-training
    HostName your.vps.ip.address
    User ubuntu
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

After setup, connect simply with:

```bash
ssh vps-training
```

## Terminal & Shell Setup

### Linux/macOS

Your terminal is already set up. Recommended shells:

```bash
# Check current shell
echo $SHELL

# Switch to bash (if using zsh)
bash

# Or set it as default shell
chsh -s /bin/bash
```

### Windows: Three Options

**Option 1: PowerShell (Native)**
- Modern, built-in
- Less Unix-like but improving

**Option 2: Git Bash**
- UNIX commands on Windows
- Install from https://gitforwindows.org
- Good for Windows users learning Linux

**Option 3: WSL 2 (Recommended)**
- Windows Subsystem for Linux
- Full Linux environment
- Best Windows option for Linux development

```powershell
# Install WSL 2 in PowerShell (Administrator)
wsl --install

# This installs Ubuntu by default
# Restart computer after installation
```

## Text Editors for Configuration

You'll need to edit configuration files on your VPS. Learn one of these:

### Nano (Easiest)

```bash
# Open file in nano
nano filename.txt

# Commands while in nano:
# Ctrl+O = Save
# Ctrl+X = Exit
# Ctrl+K = Cut line
# Ctrl+U = Paste line
```

**Best for:** Beginners, simple edits

### Vim (Powerful but steep learning curve)

```bash
# Open file in vim
vim filename.txt

# Modes:
# Press 'i' to enter INSERT mode (edit text)
# Press 'ESC' to exit INSERT mode
# Type ':wq' to write and quit
# Type ':q!' to quit without saving
```

**Best for:** Experienced developers, advanced edits

### VS Code with SSH Extension

Install VS Code locally, then use the Remote - SSH extension:

```
1. Install VS Code
2. Install "Remote - SSH" extension
3. VS Code can edit files directly on your VPS
```

**Best for:** Powerful editing with GUI, modern features

## Package Managers

Your local system uses a package manager to install software:

### Ubuntu/Debian

```bash
# Update package list
sudo apt update

# Install a package
sudo apt install package-name

# Example: Install curl
sudo apt install curl
```

### macOS (Homebrew)

```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL \
  https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install package
brew install package-name
```

### Windows (Chocolatey)

```powershell
# Install Chocolatey (PowerShell as Administrator)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = \
  [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex \
  ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install package
choco install package-name
```

## Terminal Multiplexer (Optional but Helpful)

Keep multiple SSH sessions without opening new windows:

### Tmux

```bash
# Install tmux
sudo apt install tmux  # Linux
brew install tmux      # macOS

# Start a new session
tmux new-session -s work

# Inside tmux:
# Ctrl+B, C = Create new window
# Ctrl+B, N = Next window
# Ctrl+B, D = Detach session
# Ctrl+B, [ = Scroll/copy mode

# Reconnect to session
tmux attach-session -t work
```

## Logging & Output Capture

You'll want to save command outputs for documentation:

```bash
# Redirect output to file
command > output.txt

# Append to file
command >> output.txt

# Capture both output and errors
command > output.txt 2>&1

# View output in real-time and save it
command | tee output.txt
```

## Setup Summary Checklist

Before proceeding, verify:

- [ ] SSH client installed and working (`ssh -V`)
- [ ] SSH keys generated (`~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`)
- [ ] SSH config file created
- [ ] Preferred shell/terminal selected
- [ ] Text editor chosen and tested
- [ ] Package manager working
- [ ] Can save command outputs to files

## Your Development Machine Ready!

Your local setup is now complete for managing the VPS. In the exercises, you'll:
1. Test SSH key generation
2. Create your SSH config file
3. Practice SSH connections (even to test servers)
4. Save outputs from commands

---

**Next:** [Module 2: Linux Fundamentals Review](Part-2-Linux-Fundamentals-Review.md)
