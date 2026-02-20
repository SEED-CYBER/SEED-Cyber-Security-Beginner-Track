# Module 2: Linux Fundamentals Review

## Essential Linux Concepts

This module reviews critical Linux concepts you need for the internship. If you're already comfortable with these, you can move quickly through this section.

## User Types in Linux

Linux has three user categories:

### Root User

The **superuser** with complete system control:

```bash
# Check if you're root
whoami
# Output: root

# Root prompt shows #
root@host:~#
```

**Dangers of root:**
- Can accidentally delete entire system
- No safeguards against mistakes
- Vulnerable to malicious scripts
- Best practice: avoid using root directly

### Sudo Users

Regular users who can perform administrative tasks with `sudo`:

```bash
# As regular user (prompt shows $)
student@host:~$ whoami
student

# Use sudo for admin task
sudo apt update
# Requires password for the 'student' user

# Check if current user has sudo
sudo -l
```

**Benefits of sudo:**
- Safer than root (commands are logged)
- Each user has own password
- Can grant specific permissions
- Prompt reminds you of elevated privileges

### Regular Users

Standard accounts with limited permissions:

```bash
student@host:~$ whoami
student

student@host:~$ cat /etc/shadow
# Permission denied - regular users can't read password hashes
```

## File Paths & Directory Structure

Linux uses forward slashes `/` for paths:

```
/                    Root of entire file system
├── home/            User home directories
│   └── username/    Individual user's home
├── etc/             System configuration files
├── var/             Variable data (logs)
├── opt/             Optional software
├── srv/             Service data
└── tmp/             Temporary files
```

### Absolute vs Relative Paths

```bash
# Absolute path (starts with /)
/home/student/projects/app.py

# Relative path (depends on current location)
projects/app.py          # If you're in /home/student
../other/file.txt        # Go up one directory

# Home directory shortcut
~/projects               # Expands to /home/student/projects
```

### Navigation Commands

```bash
# Print working directory
pwd

# List files
ls                   # Simple list
ls -l                # Long format (permissions, size, date)
ls -la               # Include hidden files (start with .)
ls -lh               # Human-readable sizes

# Change directory
cd /path/to/dir      # Absolute path
cd projects          # Relative path
cd ..                # Parent directory
cd ~                 # Home directory
cd -                 # Previous directory
```

## File Permissions

Permissions control who can read, write, and execute files:

```
-rw-r--r-- 1 student students 1024 Feb 20 10:30 file.txt
│││││││││ user group   size
│││││││││
│││││└── Others: read(4)
│││└──── Group: read(4) write(2)
└─────── User: read(4) write(2)

- = regular file
d = directory
l = symbolic link
```

### Understanding Permissions

```
r (read)    = 4
w (write)   = 2
x (execute) = 1

Total combinations:
7 = 4+2+1 = rwx (all permissions)
6 = 4+2   = rw- (read and write)
5 = 4+1   = r-x (read and execute)
4 = 4     = r-- (read only)
0 = ---   (no permissions)
```

### Changing Permissions

```bash
# Format: chmod [user type][+/-][permission] file
# User types: u(user) g(group) o(others) a(all)

# Add execute permission for user
chmod u+x script.sh

# Remove write permission for others
chmod o-w file.txt

# Set specific permissions (numeric)
chmod 644 file.txt       # rw-r--r--
chmod 755 script.sh      # rwxr-xr-x
chmod 700 private.txt    # rwx------
```

### Changing Ownership

```bash
# Change file owner
chown newuser file.txt

# Change owner and group
chown newuser:newgroup file.txt

# Requires sudo
sudo chown root:root /etc/config.txt

# Recursive - change directory and contents
sudo chown -R student:students /home/student/projects
```

## Working with Packages

Linux distributions include package managers to install software:

### Ubuntu/Debian (apt)

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Install a package
sudo apt install package-name

# Remove a package
sudo apt remove package-name

# Search for a package
apt search keyword

# Show package information
apt show package-name
```

### Example: Installing Common Tools

```bash
# Update first
sudo apt update

# Install useful tools
sudo apt install curl wget git nano vim

# Install development tools
sudo apt install build-essential python3 python3-pip
```

## System Information Commands

```bash
# Hostname (VPS name)
hostname

# Kernel and system info
uname -a

# CPU information
nproc                    # Number of CPUs
lscpu                    # Detailed CPU info

# Memory usage
free -h                  # Human-readable format

# Disk usage
df -h                    # File system sizes
du -sh folder/           # Size of specific folder

# Current user and groups
whoami                   # Current user
id                       # User ID and groups

# Uptime
uptime                   # How long system has been running
```

## Essential Command-Line Tools

### File Operations

```bash
# Copy files
cp source.txt destination.txt
cp -r source_dir dest_dir    # Recursive (directories)

# Move/rename files
mv oldname.txt newname.txt
mv file.txt /new/location/

# Remove files
rm file.txt
rm -r directory/              # Remove directory
rm -f file.txt                # Force remove (no confirmation)

# Create directory
mkdir new_directory
mkdir -p path/to/nested/dir   # Create parents if needed
```

### Viewing File Contents

```bash
# View entire file
cat file.txt

# View file with pagination
less file.txt                  # Press 'q' to quit
more file.txt

# View first lines
head -n 10 file.txt           # First 10 lines

# View last lines
tail -n 10 file.txt           # Last 10 lines
tail -f file.txt              # Follow new lines (useful for logs)
```

### Searching & Filtering

```bash
# Search text in files
grep "search_term" file.txt
grep -r "term" directory/     # Recursive search
grep -i "term" file.txt       # Case-insensitive

# Search for files
find /path -name "*.txt"      # Find all .txt files
find /path -type f -size +1M  # Find files larger than 1MB

# Count lines in file
wc -l file.txt
```

### Piping Commands

Chain commands together:

```bash
# Pipe output of one command to another
command1 | command2

# Example: search in output
ps aux | grep nginx            # Find nginx processes

# Chain multiple pipes
cat log.txt | grep "ERROR" | wc -l  # Count error lines
```

## Managing Services

Services are background programs (like web servers, databases):

```bash
# Start a service
sudo systemctl start service-name

# Stop a service
sudo systemctl stop service-name

# Restart a service
sudo systemctl restart service-name

# Enable service at boot
sudo systemctl enable service-name

# Disable service at boot
sudo systemctl disable service-name

# Check service status
sudo systemctl status service-name

# List all services
systemctl list-units --type=service

# View service logs
sudo journalctl -u service-name
sudo journalctl -u service-name -f   # Follow logs
```

## File Editing Over SSH

You can edit files on the VPS from your local machine:

### Option 1: Nano (Simple)

```bash
# Edit file directly
nano /path/to/file.txt

# Inside nano:
# Ctrl+O = Save (press Enter)
# Ctrl+X = Exit
```

### Option 2: Vim (Advanced)

```bash
# Edit file
vim /path/to/file.txt

# Press 'i' to insert (edit)
# Press ESC to exit edit mode
# Type ':wq' to save and quit
# Type ':q!' to quit without saving
```

### Option 3: VS Code Remote SSH (Best)

Install "Remote - SSH" extension in VS Code and edit files with a modern editor directly on the VPS.

## Checking Logs

Logs are critical for understanding system behavior:

```bash
# System logs
sudo journalctl -n 50                    # Last 50 entries
sudo journalctl -u service-name -f       # Follow service logs

# Authorization logs (login attempts)
sudo tail -f /var/log/auth.log

# Syslog
sudo tail -f /var/log/syslog

# Search logs
sudo grep "ERROR" /var/log/syslog
```

## Summary of Critical Commands

By week 1, you should be comfortable with:

```bash
# Navigation & files
pwd, cd, ls, mkdir, cp, mv, rm, cat, nano

# Permissions
chmod, chown, sudo

# System info
whoami, uname, df, free, ps, top

# Package management
apt update, apt upgrade, apt install

# Services
systemctl start/stop/status

# Logs
journalctl, tail, grep
```

## Practice Exercises

Test your knowledge:

1. Create a directory structure: `~/projects/security/week1/`
2. Create a file and view its permissions: `ls -la`
3. Change permissions to: user=rwx, group=rx, others=none
4. Try viewing a protected file (you'll get "Permission denied")
5. Use `grep` to search for a pattern in a system file

---

**Next:** [Module 3: Security Tools Overview](Part-3-Security-Tools-Overview.md)
