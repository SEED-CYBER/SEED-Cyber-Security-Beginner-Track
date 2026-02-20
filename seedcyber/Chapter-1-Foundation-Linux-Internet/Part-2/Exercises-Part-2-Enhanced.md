# Part 2 Exercises: Linux Command-Line Mastery Enhanced

**Time:** 3-4 hours  
**Platform:** Linux/WSL/macOS  
**Difficulty:** Beginner (fundamental skills)  
**Prerequisite:** Part 2 content and basic terminal familiarity

---

## Exercise 2.1: Directory Navigation Maze

**Objective:** Master absolute and relative paths, understand directory structure

**Why this matters:** 
- Incorrectly specifying paths is one of the most common beginner errors
- Security scripts rely on absolute paths to prevent hijacking
- Understanding filesystem hierarchy is essential for finding vulnerabilities

### Part A: Understanding Your Filesystem

**Step 1: Explore the root filesystem**

```bash
# Go to the very top (root)
cd /

# List what's there
ls -la
```

**Expected output:**
```
drwxr-xr-x   root   bin/       (holder for executable programs)
drwxr-xr-x   root   boot/      (boot files)
drwxr-xr-x   root   dev/       (device files)
drwxr-xr-x   root   etc/       (configuration files - CRITICAL!)
drwxr-xr-x   root   home/      (user home directories)
drwxr-xr-x   root   lib/       (system libraries)
drwxr-xr-x   root   media/     (removable media)
drwxr-xr-x   root   opt/       (optional software)
drwxr-xr-x   root   root/      (root user's home)
drwxr-xr-x   root   tmp/       (temporary files)
drwxr-xr-x   root   usr/       (user programs/data)
drwxr-xr-x   root   var/       (variable data - logs, cache)
```

**Questions for you:**
- Which directory contains user home folders? (answer: /home)
- Where would application configuration usually be? (answer: /etc)
- Where are application logs typically stored? (answer: /var/log)

**Step 2: Navigate using absolute paths**

```bash
# Go to /home directory
cd /home
pwd  # Should show: /home

# List files
ls -la

# Go to /etc directory
cd /etc
pwd  # Should show: /etc

# Go back to /home
cd /home
pwd  # Confirms you're at /home
```

**Step 3: Using relative paths**

```bash
# From /home, list files (you're already here)
ls -la

# Go INTO a subdirectory (if exists)
# Don't use / at start - that would be absolute!
cd username  # without / = relative path, relative to /home

# Confirm where you are
pwd

# Go up one level (to /home)
cd ..
pwd  # Should be /home again

# Go to home directory (shortcut)
cd ~
pwd  # Should be /home/username (your home)
```

### Part B: Path Navigation Exercises

**Create a test directory structure:**

```bash
# Start in home directory
cd ~

# Create nest directory
mkdir -p projects/security/week1/day1
mkdir -p projects/security/week1/day2

# Verify structure
tree projects/  # or use ls -R projects/
```

**Exercise 1: Absolute paths**
```bash
# From home, navigate using ABSOLUTE paths
cd /home/username/projects/security

# Confirm where you are
pwd  # Should show full path

# List contents using absolute path (not 'ls', but full path reference)
ls -la /home/username/projects/security

# Try from a different location
cd /tmp  # Go to tmp
ls -la /home/username/projects/security  # List without changing directory!
```

**Exercise 2: Relative paths**
```bash
# Start in /home/username
cd ~

# Navigate relative to home
cd projects/security/week1

# Confirm
pwd  # Should be /home/username/projects/security/week1

# Go up multiple levels
cd ../..  # Up 2 levels...

pwd  # Should be /home/username/projects

# Go sideways (to parallel directory)
cd security  # We're in projects, security is a subdirectory
cd week1/day1  # Relative path continues

pwd  # Should be /home/username/projects/security/week1/day1

# Go to day2 (sibling of day1)
cd ../day2  # .. = up to week1, then day2 = relative path

pwd  # Should be /home/username/projects/security/week1/day2

# Jump to home from anywhere
cd ~
pwd  # Should be /home/username (always works!)
```

**Deliverable:**
Create a document showing:
1. 5 absolute paths you navigated to
2. 5 relative paths you used
3. When would you use each?

---

## Exercise 2.2: File Manipulation Like a Pro

**Objective:** Master copy, move, delete operations safely

**Why this matters:**
- `rm` is permanent (no trash can!)
- One mistake can delete important files
- Security scripts rely on precise file operations

### Part A: Safe Deletion Practices

**Never blindly delete!**

```bash
# BAD (DANGEROUS):
rm -rf /tmp/*  # Could delete important files!

# GOOD:
ls -la /tmp/  # Check what's there FIRST
# Review carefully...
rm -rf /tmp/unwanted_folder/
```

**Step 1: Create test files**

```bash
cd ~/projects/security/week1/day1

# Create some test files
touch important.txt
touch temporary.txt
touch backup.txt

# Create test directories
mkdir logs
mkdir cache
mkdir temp

# Verify
ls -la
```

**Step 2: Practice safe deletion**

```bash
# Before deleting, list to confirm
ls -la temp/
# Does it look right?

# Then delete
rm -r temp/

# Verify it's gone
ls -la  # Should not show temp/
```

**Step 3: Deletion with confirmation**

```bash
# Use -i flag for interactive (asks before deleting)
rm -i temporary.txt
# Output: remove regular file 'temporary.txt'? 

# Type 'y' for yes, 'n' for no
# If file is important, say 'n'!
```

### Part B: Copying and Moving Files

**Step 1: Create files to work with**

```bash
cd ~/projects/security/week1

# Create some files
echo "Important data" > important.txt
echo "Log entry" > yesterday.log
echo "Temporary" > temp.txt
```

**Step 2: Copy files**

```bash
# Copy to same directory (rename)
cp important.txt important_backup.txt
ls -la  # Should show both

# Copy to different directory
cp important.txt day1/
ls -la day1/  # Should see important.txt

# Copy entire directory
cp -r day1/ day1_backup/
ls -la  # Should show day1 and day1_backup

# Verify contents match
ls -la day1/  
ls -la day1_backup/  # Should be identical
```

**Step 3: Move (rename) files**

```bash
# Rename file
mv temporary.txt temp_old.txt
ls -la  # Should see temp_old.txt, not temporary.txt

# Move file to directory
mv temp_old.txt day1/
ls -la  # temp_old.txt should be gone
ls -la day1/  # Should see temp_old.txt there now

# Move with rename
mv day2/file.txt day2/file_processed.txt
```

**Deliverable:**
Create a script that:
1. Creates 5 test files
2. Copies them with confirmation
3. Moves them to subdirectories
4. Lists results to confirm operations worked

---

## Exercise 2.3: Searching for Needles in Haystacks

**Objective:** Master grep and find for security analysis

**Why this matters:**
- Log analysis uses grep constantly
- Finding vulnerable files uses find
- These are essential security tools

### Part A: Grep - Text Pattern Searching

**Step 1: Create test log file**

```bash
cd ~/projects/security/week1/day1

cat > app.log << 'EOF'
2024-01-15 10:05:23 INFO User alice logged in
2024-01-15 10:15:44 INFO User bob logged in
2024-01-15 10:22:11 ERROR Database connection failed
2024-01-15 10:23:01 ERROR Database connection failed
2024-01-15 10:23:45 ERROR Database connection failed
2024-01-15 10:24:12 WARNING Slow query detected
2024-01-15 10:30:22 INFO User alice logged out
2024-01-15 10:31:05 ERROR Authentication failed for user charlie
2024-01-15 10:31:15 ERROR Authentication failed for user charlie
2024-01-15 10:31:25 ERROR Authentication failed for user charlie
2024-01-15 10:45:33 CRITICAL Unauthorized access attempt
EOF
```

**Step 2: Search for ERROR entries**

```bash
# Find all error lines
grep "ERROR" app.log

# Expected output: 4 lines with ERROR

# Count them
grep -c "ERROR" app.log
# Output: 4

# Show line numbers
grep -n "ERROR" app.log
# Shows which lines (3, 4, 5, 6)
```

**Step 3: Search patterns**

```bash
# Find entries starting with ERROR
grep "^ERROR" app.log  # ^ means start of line
# Output: Lines starting with ERROR

# Find entries ending with 23
grep "23$" app.log  # $ means end of line
# Output: Lines ending with 23

# Case-insensitive search
grep -i "error" app.log
grep -i "warning" app.log
# Case doesn't matter

# Lines that DON'T contain ERROR (inverse)
grep -v "ERROR" app.log
# All non-error lines
```

**Step 4: Search multiple files**

```bash
# Create another log file
cat > system.log << 'EOF'
System startup successful
ERROR: Memory allocation failed
WARNING: Disk space low
EOF

# Search both files
grep "ERROR" app.log system.log
# Shows which file each match came from:
# app.log:2024-01-15 10:22:11 ERROR Database connection failed
# system.log:ERROR: Memory allocation failed

# Search entire directory
grep -r "ERROR" .
# Searches all files in current and subfolders
```

### Part B: Find - File Locating

**Step 1: Create test file structure**

```bash
cd ~/projects/security

# Create directories with various files
mkdir -p documents logs backups
mkdir -p sensitive

touch documents/report.txt
touch documents/notes.txt
touch logs/app.log
touch logs/system.log
touch logs/access.log.gz  # Compressed
touch backups/backup_2024.tar
touch sensitive/passwords.txt

# Create large file
dd if=/dev/zero of=large_file.bin bs=1M count=10  # 10MB file

# Create files in subdirectories
find . -type f | wc -l  # See total files
```

**Step 2: Find by name**

```bash
# Find all .log files
find . -name "*.log"

# Find all .txt files
find . -name "*.txt"

# Find a specific file
find . -name "passwords.txt"
```

**Step 3: Find by type**

```bash
# Find only directories
find . -type d

# Find only files
find . -type f

# Find symbolic links
find . -type l
```

**Step 4: Find by size/date**

```bash
# Files larger than 1MB
find . -size +1M

# Files smaller than 10KB
find . -size -10K

# Files modified in last 7 days
find . -mtime -7

# Files accessed more than 30 days ago
find . -atime +30
```

**Step 5: Find by permissions (SECURITY!)**

```bash
# Find world-writable files (DANGEROUS!)
find . -perm 777
# These are security risks!

# Find files readable by others
find . -perm -004

# Find executable files
find . -perm /111
```

**Deliverable:**
Create a security report showing:
1. All .log files in project
2. All .txt files (might contain secrets!)
3. Files larger than 5MB
4. Any world-writable files
5. Files modified in last 7 days

---

## Exercise 2.4: Process Inspection & Control

**Objective:** Master process management for security monitoring

**Why this matters:**
- Detecting malware often means finding unexpected processes
- Performance issues require process analysis
- Terminating rogue processes is critical

### Part A: Understanding Processes

**Step 1: List all processes**

```bash
# Get all running processes
ps aux

# Output columns explained:
# USER       = Who's running it
# PID        = Process ID (unique identifier)
# %CPU       = CPU usage percentage
# %MEM       = Memory usage percentage
# VSZ        = Virtual memory size
# RSS        = Resident memory size
# STAT       = Process state
# START      = When it started
# COMMAND    = The command that started it
```

**Step 2: Find specific processes**

```bash
# Find processes containing "bash"
ps aux | grep bash

# Find process of specific user
ps aux | grep root

# Don't count grep itself
ps aux | grep bash | grep -v grep
```

**Step 3: Monitor with top**

```bash
# Start top (live monitoring)
top

# Key commands in top:
# q = quit
# u = show processes of specific user
# k = kill process (interactive)
# M = sort by memory
# P = sort by CPU
# ? = help
```

**Step 4: Process relationships**

```bash
# Show process tree
pstree

# With PIDs
pstree -p

# Shows parent-child relationships
# (bash) -> grep -> find -> etc
```

### Part B: Terminating Processes

**Step 1: Start background process**

```bash
# Start a long-running command
sleep 100 &
# Output: [1] 12345
# [1] = job number
# 12345 = PID

# Start another
sleep 200 &

# List jobs
jobs
# Output:
# [1]-  Running                 sleep 100 &
# [2]+  Running                 sleep 200 &
```

**Step 2: Kill gracefully**

```bash
# Kill by job number
kill %1  # Kills job 1 (sleep 100)

# OR kill by PID
kill 12345  # If PID is 12345

# Check status
jobs
# [1] is gone, [2] still running
```

**Step 3: Force kill (if needed)**

```bash
# Normal kill (graceful)
kill 12346  # sleep 200

# If process ignores, force it
kill -9 12346  # FORCE kill (nuclear option!)
```

**Deliverable:**
Create a script that:
1. Lists all running processes
2. Finds all bash processes
3. Starts a background sleep process
4. Terminates it
5. Verifies it's gone

---

## Exercise 2.5: Piping & Data Flow

**Objective:** Master the most powerful Linux feature

**Why this matters:**
- Piping is how you build complex analysis commands
- Log analysis requires chaining multiple commands
- Security tools often need pipeline processing

### Part A: Understanding Pipes

**Step 1: Simple pipe**

```bash
# Without pipe (see everything):
ps aux
# Output: 50+ lines, hard to find nginx

# With pipe (filter):
ps aux | grep nginx
# Output: Only lines with nginx!
```

**Step 2: Chaining pipes**

```bash
# Find nginx processes, show only the command
ps aux | grep nginx | grep -v grep

# Chain more:
ps aux | grep nginx | grep -v grep | wc -l
# Count how many nginx processes are running
```

### Part B: Real Log Analysis

**Step 1: Create realistic log file**

```bash
cat > /tmp/access.log << 'EOF'
192.168.1.100 - - [15/Jan/2024:10:05:23 +0000] "GET / HTTP/1.1" 200 1234
192.168.1.101 - - [15/Jan/2024:10:05:25 +0000] "GET /api HTTP/1.1" 404 567
192.168.1.100 - - [15/Jan/2024:10:05:27 +0000] "POST /login HTTP/1.1" 200 789
192.168.1.102 - - [15/Jan/2024:10:05:29 +0000] "GET /admin HTTP/1.1" 401 234
192.168.1.100 - - [15/Jan/2024:10:05:31 +0000] "GET / HTTP/1.1" 200 1234
192.168.1.103 - - [15/Jan/2024:10:05:33 +0000] "GET /admin HTTP/1.1" 401 234
192.168.1.104 - - [15/Jan/2024:10:05:35 +0000] "POST /api HTTP/1.1" 500 100
192.168.1.100 - - [15/Jan/2024:10:05:37 +0000] "GET / HTTP/1.1" 200 1234
192.168.1.102 - - [15/Jan/2024:10:05:39 +0000] "GET /admin HTTP/1.1" 401 234
192.168.1.100 - - [15/Jan/2024:10:05:41 +0000] "GET / HTTP/1.1" 200 1234
EOF
```

**Step 2: Analysis tasks**

```bash
# Task 1: Find all failed requests (400+)
cat /tmp/access.log | grep -E " (4|5)[0-9]{2} "

# Task 2: Find unique IPs
cat /tmp/access.log | cut -d' ' -f1 | sort | uniq

# Task 3: Count requests per IP
cat /tmp/access.log | cut -d' ' -f1 | sort | uniq -c

# Task 4: Find most active IP
cat /tmp/access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -1

# Task 5: Find failed requests from IP 192.168.1.102
cat /tmp/access.log | grep 192.168.1.102 | grep -E " (4|5)[0-9]{2} "
```

**Step 3: Complex analysis**

```bash
# Find IPs trying to access /admin (reconnaissance!)
cat /tmp/access.log | grep "/admin" | cut -d' ' -f1 | sort | uniq -c | sort -rn

# Failed login attempts
cat /tmp/access.log | grep "(401|401)" | wc -l

# Top 5 response codes
cat /tmp/access.log | awk '{print $9}' | sort | uniq -c | sort -rn | head -5
```

**Deliverable:**
Create security analysis report showing:
1. All requests with errors (4xx, 5xx)
2. Top 3 most active IPs
3. IPs attempting admin access
4. Failed authentication attempts
5. Unique IPs and request counts

---

## Exercise 2.6: Practical Security Tasks

**Objective:** Apply all skills to real security scenarios

### Task 1: System Audit Command

Create a script that collects security info:

```bash
#!/bin/bash

echo "=== SYSTEM SECURITY AUDIT ==="
echo

echo "Currently logged in users:"
w

echo -e "\nRecent login attempts:"
last -n 5

echo -e "\nListening ports:"
netstat -tln | grep LISTEN

echo -e "\nRunning processes:"
ps aux | wc -l

echo -e "\nDisk usage:"
df -h

echo -e "\nWorld-writable files (RISK!):"
find / -perm 777 2>/dev/null | head -10

echo -e "\nFiles changed recently (7 days):"
find /home -mtime -7 2>/dev/null | head -10
```

### Task 2: Log Analysis Pipeline

```bash
# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log | cut -d' ' -f11 | sort | uniq -c | sort -rn

# Find suspicious access patterns
grep "401\|403\|404\|500" /var/log/apache2/access.log | tail -20

# Check for port scans
grep "Connection reset" /var/log/syslog | tail -10
```

### Task 3: File Security Review

```bash
# Check for files modified by root in user directories
find /home -user root -mtime -7 2>/dev/null

# Find setuid binaries (potential privilege escalation risk)
find / -perm -4000 2>/dev/null

# Find unowned files (inconsistent permissions)
find / -nouser -o -nogroup 2>/dev/null | head -20
```

---

**Summary:** These exercises build practical Linux competency essential for security work.
