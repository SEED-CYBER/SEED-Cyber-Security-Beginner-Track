# Part 2: Linux Command Line Mastery - Enhanced

## Why Command Line? (The Conceptual Foundation)

**Reality:** 96% of cloud servers don't have graphical interfaces. Most cybersecurity work happens in terminals.

But more importantly: **The command line is POWER.**

### The Shell: Your Operating System Interface

Your computer has layers:
```
┌─────────────────────────────────────┐
│ You (the user)                      │
├─────────────────────────────────────┤
│ SHELL (bash, zsh, sh) ← YOU ARE HERE│
│ └─ How you talk to OS               │
├─────────────────────────────────────┤
│ KERNEL (Linux kernel)               │
│ └─ Controls hardware, files, memory │
├─────────────────────────────────────┤
│ HARDWARE (CPU, RAM, disk)           │
└─────────────────────────────────────┘

When you type `ls`, the SHELL:
1. Parses your command
2. Asks KERNEL to list files
3. KERNEL accesses the disk
4. Returns list to SHELL
5. SHELL displays to you
```

### Why Learn Command Line for Security?

```
Problem: Attackers use command line
Solution: You must understand it better

├─ Penetration testers use bash commands
├─ Red teamers write complex shell scripts
├─ Malware often spawns shells
├─ Log analysis requires grep, sed, awk
├─ Forensics requires understanding file systems
├─ Exploit chains are bash scripts
└─ Many tools have NO GUI

Bottom line: Command line literacy = Security literacy
```

### Fundamental Concepts Before Commands

**Concept 1: Absolute vs Relative Paths**
```
Absolute path: Starts with / (root of entire system)
  /home/user/documents/file.txt
  │    │    │         
  │    │    └─ File in documents folder
  │    └─────── In user's home folder
  └──────────── From root

Relative path: Relative to CURRENT directory
  If you're in /home/user:
    documents/file.txt (same file, but relative)
    
  If you're in /home:
    user/documents/file.txt (same file, from here)

Security implication:
  Scripts using relative paths can be hijacked!
  Smart attackers create malicious files in PATH
  Always use absolute paths in security scripts
```

**Concept 2: Standard Streams (stdin, stdout, stderr)**
```
Every Linux command has 3 data channels:

stdin (file descriptor 0):  Input to command
  └─ Where command reads from
  └─ Keyboard by default, but can redirect!

stdout (file descriptor 1): Normal output
  └─ What command writes (successful)
  └─ Shown on screen by default

stderr (file descriptor 2): Error output
  └─ Error messages from command
  └─ Separate from normal output!

Examples:
  ls > files.txt      # Redirect stdout (normal output)
  grep 2> errors.log  # Redirect stderr (errors)
  find 2>/dev/null    # Send errors to /dev/null (discard)
  command 2>&1        # Combine stderr AND stdout
```

**Concept 3: Exit Codes (How Commands Communicate Success/Failure)**
```
Every command returns an "exit code" when done:
  0 = Success
  1-255 = Various errors

Example:
  grep "root" /etc/passwd
  echo $?  # Shows: 0 (found it!)
  
  grep "nosuchuser" /etc/passwd
  echo $?  # Shows: 1 (not found!)

Why it matters for security:
  ├─ Scripts check exit codes to know if something worked
  ├─ Exploit scripts fail if dependency missing
  ├─ Forensics depends on exit codes
  └─ Attackers use exit codes to know if command succeeded

You'll often see in scripts:
  if command; then
    # Exit code was 0 (success)
    echo "Command worked!"
  else
    # Exit code wasn't 0 (failed)
    echo "Command failed!"
  fi
```

**Concept 4: Permissions (preview for now)**
```
Every file has ownership and permissions:
  -rw-r--r-- 1 root root 1234 Jan 1 12:00 file.txt
  │││││││││ │ │    │    │    │   │  │    │
  │││││││││ │ │    │    │    │   │  │    └─ Filename
  │││││││││ │ │    │    │    │   │  └─ Day
  │││││││││ │ │    │    │    │   └─ Time
  │││││││││ │ │    │    │    └─ File size
  │││││││││ │ │    │    └─ Modification date
  │││││││││ │ │    └─ Group owner
  │││││││││ │ └─ User owner
  │││││││││ └─ Link count
  └││││││││─ Permission bits

Security: These permissions control WHO can do WHAT
  For security work, you MUST understand this!
  (Covered fully in Part 3)
```

---

## Essential Command Categories

### Navigation & File Management (Understanding Paths)

**Core Concept:** You're always IN a directory. Commands affect what's there.

```bash
# WHERE AM I?
pwd    # Print Working Directory - shows path you're currently in
# Output: /home/username/documents
# This means you're IN the documents folder

# WHAT'S HERE?
ls           # List files in current directory
ls -l        # Long format - see permissions, size, dates
ls -la       # Include HIDDEN files (start with .)
ls -lh       # Human readable - 2.3K instead of 2348

# Why -l matters:
# drwxr-xr-x 2 user user 4096 Jan 1 12:00 dirname
# ├─ d = directory (not file)
# ├─ rwxr-xr-x = permissions (we'll detail in Part 3)
# ├─ user user = owner:group
# ├─ 4096 = size in bytes
# └─ Jan 1 = last modified

# MOVING AROUND
cd /home       # Go TO /home (absolute path)
cd docs        # Go INTO docs folder (relative - must exist here!)
cd ..          # Go UP one level
cd ~           # Go to home directory (~=/home/username)
cd -           # Go to PREVIOUS directory (toggles back/forth!)

# CREATING DIRECTORIES
mkdir myfolder
mkdir -p path/to/nested/dir    # -p = creates parent directories too
# Without -p: would fail if parent doesn't exist
# With -p: creates whole chain

# COPY FILES
cp file.txt copy_of_file.txt   # Copy in same directory
cp file.txt /tmp/file.txt      # Copy to different directory
cp -r sourcedir/ destdir/      # -r = recursive (copy whole folder)

# MOVE/RENAME (same command!)
mv oldname.txt newname.txt     # Rename in same place
mv file.txt /new/path/         # Move to different directory
mv * /media/backup/            # Move ALL files

# DELETE (CAREFUL! No trash can!)
rm file.txt                    # Delete file (PERMANENT!)
rm -f file.txt                 # Force delete (no confirmation)
# ⚠️ DANGER: rm -rf directory/  would delete EVERYTHING recursively
# Pro tip: Let's USE `ls` before `rm` to be safe
ls path_to_delete/    # See what's there first
rm -r path_to_delete/ # NOW delete

# CHECK SIZES (for quota management)
du -sh file.txt        # File size human-readable
du -sh directory/      # Folder size (includes all contents)
df -h                  # Disk usage of mounted drives
```

### Text Viewing & Analyzing (How to Inspect Content)

```bash
# READ ENTIRE FILE
cat file.txt           # Show whole file at once
# Use only for SHORT files!
# For large files, use less (so you don't scroll forever)

# READ WITH CONTROL
less file.txt          # Pagination - press Space for next page
                       # q to quit, /pattern to search
more file.txt          # Similar to less (older)

# READ FIRST/LAST LINES (For log analysis!)
head -n 5 file.txt     # First 5 lines
head file.txt          # First 10 lines (default)
tail -n 10 file.txt    # Last 10 lines
tail file.txt          # Last 10 lines
tail -f file.txt       # FOLLOW mode - watch new lines as added!
# tail -f is GREAT for watching log files in real-time

# COUNT STATISTICS
wc -l file.txt         # Line count
wc -w file.txt         # Word count  
wc file.txt            # Lines words characters

# Why wc matters:
#   Detecting log inflation attacks
#   Monitoring log file growth
#   Counting number of users in file

# EDIT FILES
nano file.txt          # User-friendly editor (start here!)
vim file.txt           # Powerful editor (steeper learning curve)

# All text editors:
# Ctrl+X to save/exit (nano) or :q! to quit (vim)
```

### Searching & Filtering (Finding Information in Chaos)

**Core Concept:** grep SEARCHES for patterns, find SEARCHES for file names

```bash
# SEARCH CONTENT (grep = find text patterns)
grep "ERROR" file.txt          # Lines containing ERROR
grep -i "error" file.txt       # Case-insensitive
grep -n "ERROR" file.txt       # Show line numbers
grep -c "ERROR" file.txt       # COUNT matching lines
grep -v "ERROR" file.txt       # Inverse - lines WITHOUT ERROR

# Pattern matching:
grep "^ERROR" file.txt         # Lines STARTING with ERROR
grep "ERROR$" file.txt         # Lines ENDING with ERROR
grep "E.*R" file.txt           # ERROR with anything between

# Recursive search (through folders):
grep -r "password" /etc/       # Search whole /etc directory
grep -r "password" . 2>/dev/null  # Search current dir, hide errors

# Why grep matters for security:
#   Finding suspicious patterns in logs
#   Searching for hardcoded credentials
#   Analyzing configuration files
#   Building forensic timelines

# FIND FILES (find = locate files by name/attributes)
find /path -name "*.txt"       # All .txt files
find /path -type f -name "*.log"  # Type f = files only
find /path -type d -name "tmp"    # Type d = directories only
find /path -name "*.exe" 2>/dev/null  # Hide permission errors

# By size/date:
find /path -size +100M         # Files larger than 100MB
find /path -size -1K           # Files smaller than 1KB
find /path -mtime -7           # Modified in last 7 days
find /path -atime +30          # Accessed more than 30 days ago

# By owner/permissions:
find /path -user root          # Files owned by root
find /path -group sudo         # Files owned by sudo group
find /path -perm 777           # World-writable files (security risk!)

# Why find matters:
#   Identifying compromised binaries
#   Finding world-writable files
#   Locating suspicious uploads
#   Backup/recovery operations
```

### Process Management (Finding & Controlling Running Programs)

```bash
# LIST RUNNING PROCESSES
ps aux                 # ALL processes, detailed
ps aux | grep nginx    # Find specific process
ps aux | grep user     # Find process running as user

# Understanding ps output:
USER        PID %CPU %MEM COMMAND
root        1   0.0  0.1  /sbin/init
├─ USER = who's running it
├─ PID = process ID (unique identifier)
├─ %CPU = CPU usage
├─ %MEM = memory usage
└─ COMMAND = what it's running

# MONITORING IN REAL-TIME
top                    # Live process monitor
# Shows CPU, memory, running processes
# Press q to quit, ? for help

htop                   # Better version of top (if installed)
# More user-friendly, colors, sorting

# TERMINATING PROCESSES (DANGEROUS!)
kill 1234              # Graceful shutdown of PID 1234
kill -9 1234           # FORCE kill (give no chances!)
pkill nginx            # Kill by process NAME (all matching)

# ⚠️ Security: Know what you're killing!
# Killing wrong process = denial of service
# Killing init (PID 1) = system crash

# BACKGROUND/FOREGROUND
command &              # Run in background (get prompt back)
Ctrl+Z                 # Suspend current process
bg                     # Resume in background
fg                     # Bring to foreground
jobs                   # List background jobs

# Why this matters:
#   Running multiple tools simultaneously
#   Long-running processes (downloads, scans)
#   Preventing lockup during slow operations
```

### System Information (Know Your Target)

```bash
# System basics:
uname -a               # Full system info
uname -s               # Just kernel name (Linux)
uname -r               # Kernel version (4.19.0)

# User info:
whoami                 # Current logged-in user
id                     # User ID and groups
w                      # Currently logged-in users
last                   # Login history

# Time/Date:
date                   # Current date/time
uptime                 # How long system has been up

# CPU/Memory:
nproc                  # Number of CPU cores
free -h                # RAM usage human-readable
free -m                # RAM in MB
top -b -n 1            # Single snapshot of top

# Disk:
df -h                  # Mounted drives and usage
du -sh /path           # Size of directory

# Network:
ip addr show           # IP addresses
ip route show          # Routing table
netstat -an            # Network connections (if installed)
ss -tln                # Network sockets (modern replacement)

# Why system info matters:
#   Kernel version determines exploits available
#   CPU/memory = bottleneck analysis
#   Network config = network mapping
#   User list = privilege escalation vectors
```

### Piping & Redirection (Connecting Commands Together)

**Core Concept:** Commands output data. You can:
- Save it to file
- Send it to another command  
- Ignore it entirely

```bash
# REDIRECTION (send output to file):
command > output.txt           # OVERWRITE file
command >> output.txt          # APPEND to file
command 2> errors.txt          # Redirect ERRORS only
command > out.txt 2>&1         # BOTH output and errors

# Practical:
ls -la > directory_listing.txt  # Save full listing
grep "ERROR" log.txt > errors.txt  # Save errors

# PIPING (|): Connect command output to another command:
ps aux | grep nginx            # ps outputs list | grep filters it
cat file.txt | wc -l           # cat outputs content | wc counts lines
grep "ERROR" log.txt | head -5  # grep finds errors | head shows first 5

# Chain MANY pipes:
cat huge_log.txt | grep "ERROR" | grep -v "DEBUG" | sort | uniq | wc -l
# ├─ cat: read file
# ├─ grep "ERROR": find errors
# ├─ grep -v "DEBUG": remove debug messages
# ├─ sort: arrange alphabetically
# ├─ uniq: remove duplicates
# └─ wc -l: count unique errors

# Why this matters:
#   Log analysis (most common security task!)
#   Data processing and filtering
#   Building analysis pipelines
#   Writing shellscripts for automation
```

---

## Advanced Command-Line Techniques

### Command Substitution

```bash
# Using $() syntax
echo "Today is $(date)"

# Using backticks (older syntax)
echo "Today is `date`"

# List files with file sizes
for file in $(ls); do wc -l $file; done
```

### Variables

```bash
# Set variable
MESSAGE="Hello World"

# Use variable
echo $MESSAGE

# Command output in variable
IP=$(hostname -I)
echo $IP

# Environment variables
echo $HOME
echo $USER
echo $PATH
```

### Aliases (Create shortcuts)

```bash
# Create temporary alias
alias l='ls -lah'

# Make permanent (add to ~/.bashrc)
nano ~/.bashrc
# Add: alias l='ls -lah'
source ~/.bashrc
```

## Common Command Combinations

```bash
# Find large files
du -sh * | sort -h

# Search log files
grep "ERROR" /var/log/*.log | head -20

# Monitor system resources
watch -n 1 'free -h && echo "---" && df -h'

# List currently logged in users
w

# Show user login history
last

# Find files opened by process
lsof -p 1234

# Network connections
netstat -an | grep ESTABLISHED
ss -tln | grep LISTEN
```

---

**Next:** [Part 3: Users, Groups & Permissions](../Part-3/Part-3-Users-Groups-Permissions.md)
