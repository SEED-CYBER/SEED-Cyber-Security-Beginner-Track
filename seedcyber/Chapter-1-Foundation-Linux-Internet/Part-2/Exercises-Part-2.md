# Part 2 Exercises: Linux Command Line Mastery

**Time:** 3 hours  
**Platform:** Linux/macOS/Windows with WSL

## Exercise 2.1: File System Navigation

**All Platforms (Linux/macOS/WSL):**

```bash
# Navigate directory structure
cd /
pwd
ls -la
cd /etc
ls -l | head -20
cd ~
pwd
ls -la
```

**Deliverable:**
- Output showing directory traversal
- Explanation of each directory's purpose (/etc, /var, /home, /opt)
- File count in /etc directory

## Exercise 2.2: File Permissions Mastery

**Objective:** Understand and modify permissions

```bash
# Create test directory
mkdir ~/permission-test
cd ~/permission-test

# Create files with different content
echo "Secret" > secret.txt
echo "Public" > public.txt
echo "#!/bin/bash" > script.sh
echo "echo Hello" >> script.sh

# Check initial permissions
ls -la

# Modify permissions
chmod 600 secret.txt      # Owner only
chmod 644 public.txt      # Owner rw, others r
chmod 755 script.sh       # Owner rwx, others rx

# Verify
ls -la

# Try to read as different permission levels
cat secret.txt
cat public.txt
./script.sh
```

**Deliverable:**
- Output of ls -la showing all permission changes
- Explanation of 600, 644, 755 meanings
- Table showing permission changes and effects

## Exercise 2.3: Text Processing Pipeline

**Objective:** Master piping and filtering

```bash
# Create sample data
cat > data.log << 'EOF'
2026-02-20 10:00:00 INFO User123 logged in
2026-02-20 10:05:00 ERROR Database connection timeout
2026-02-20 10:10:00 INFO User456 logged in
2026-02-20 10:15:00 ERROR Authentication failed for User789
2026-02-20 10:20:00 WARNING High memory usage
2026-02-20 10:25:00 INFO User123 logged out
2026-02-20 10:30:00 ERROR File not found: /data/config.ini
EOF

# Extract and count errors
grep "ERROR" data.log
grep "ERROR" data.log | wc -l

# Extract users who logged in
grep "logged in" data.log | cut -d' ' -f5

# Sort logs by time
sort -k1,2 data.log

# Extract and format
grep "User" data.log | awk '{print $5}' | sort | uniq

# Complex pipeline
cat data.log | grep -v "INFO" | wc -l
```

**Windows (PowerShell):**
```powershell
# Create file
@"
2026-02-20 10:00:00 INFO User123 logged in
2026-02-20 10:05:00 ERROR Database connection timeout
"@ | Set-Content data.log

# Filter
Select-String -Path data.log "ERROR"
Select-String -Path data.log "ERROR" | Measure-Object
```

**Deliverable:**
- All command outputs
- Explanation of pipes and filters
- Complex query that combines multiple operations

## Exercise 2.4: Process Management

**All Platforms (Linux/macOS/WSL):**

```bash
# View running processes
ps aux
ps aux | grep bash

# Start background process
sleep 100 &

# List jobs
jobs

# View process details
top -b -n 1 | head -20

# Process hierarchy
ps -ejH | head -20

# Find process by name
pgrep sleep

# Kill process
kill %1    # Kill job 1
pkill sleep
```

**Windows (PowerShell):**
```powershell
# List processes
Get-Process
Get-Process | Where-Object {$_.Name -like "*powershell*"}

# Get process details
Get-Process -Name explorer | Select-Object ProcessName, Id, Memory

# Stop process
Stop-Process -Name notepad
```

**Deliverable:**
- Process listing with explanation
- Demonstration of background job management
- Process hierarchy understanding

## Exercise 2.5: System Information Commands

**Objective:** Gather system data

```bash
# Comprehensive system check
uname -a
hostname
whoami
date
uptime
free -h
df -h
top -bn1 | head -20
lscpu
```

**Windows (PowerShell):**
```powershell
# System information
systeminfo
Get-ComputerInfo
Get-Process | Measure-Object -Property WorkingSet -Sum
Get-Volume
```

**Deliverable:**
- Complete system profile
- Compare CPU, memory, disk usage
- Document OS and kernel version

## Exercise 2.6: Advanced Command Combinations

**Objective:** Master complex pipelines

```bash
# Find largest files
find ~ -type f -exec ls -lh {} \; | sort -k5 -hr | head -10

# Count files by extension
find ~ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Monitor disk usage
du -sh */ | sort -hr | head -5

# List user login history
last | head -20

# Find files modified today
find ~ -type f -mtime 0

# Get top 10 lines containing error
grep -i "error" /var/log/syslog | head -10

# Count events by hour
grep "ERROR" data.log | cut -c1-13 | sort | uniq -c
```

**Deliverable:**
- Output from 3 complex queries
- Explanation of each component
- Real-world use case for each query

---

**Completion Checklist:**
- [ ] All navigation exercises complete
- [ ] File permissions understood (chmod/chown)
- [ ] Text processing pipelines working
- [ ] Process management tested
- [ ] System info gathered
- [ ] Complex commands functional

**Submission:** Part-2-Exercises-SUBMISSION.txt
