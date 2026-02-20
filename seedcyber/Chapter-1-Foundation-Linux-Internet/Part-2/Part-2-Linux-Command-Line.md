# Part 2: Linux Command Line Mastery

## Why Command Line?

Most servers run without graphical interfaces. As a cybersecurity professional, you'll manage systems exclusively via command line.

## Essential Command Categories

### Navigation & File Management

```bash
# Print working directory
pwd

# List files
ls           # Simple list
ls -l        # Long format (permissions, owner, size, date)
ls -la       # Include hidden files
ls -lh       # Human-readable file sizes

# Change directory
cd /path/to/dir
cd ..        # Parent directory
cd ~         # Home directory
cd -         # Previous directory

# Print path in readable format
tree                # Tree view (if installed)

# Create directories
mkdir dirname
mkdir -p path/to/nested/dir    # Create parents

# Copy files
cp source.txt destination.txt
cp -r sourcedir/ destdir/      # Recursive (directories)

# Move/rename
mv oldname.txt newname.txt
mv file.txt /new/path/

# Remove files
rm file.txt
rm -r directory/               # Remove directory
rm -f file.txt                 # Force (no confirmation)

# Create empty file
touch filename.txt

# Show file size
du -sh filename
du -sh directory/              # Directory size
```

### Text Viewing & Editing

```bash
# View entire file
cat file.txt

# View with pagination
less file.txt                  # More control, press 'q' to exit
more file.txt                 # Simpler pagination

# View first/last lines
head -n 5 file.txt             # First 5 lines
tail -n 10 file.txt            # Last 10 lines
tail -f file.txt               # Follow (watch new lines)

# Count lines, words, characters
wc -l file.txt                 # Line count
wc -w file.txt                 # Word count
wc file.txt                    # All three

# Edit files
nano file.txt                  # User-friendly editor
vim file.txt                   # Advanced editor
# Or use VS Code Remote SSH
```

### Searching & Filtering

```bash
# Search text in files
grep "pattern" file.txt
grep -i "pattern" file.txt     # Case-insensitive
grep -r "pattern" directory/   # Recursive search
grep -n "pattern" file.txt     # Show line numbers
grep "^pattern" file.txt       # Lines starting with pattern
grep "pattern$" file.txt       # Lines ending with pattern

# Find files
find /path -name "*.txt"       # Find all .txt files
find /path -type f -size +1M   # Files larger than 1MB
find /path -type d             # Find directories only
find /path -user username      # Files owned by user

# Count occurrences
grep -c "pattern" file.txt     # Count matching lines
```

### Process Management

```bash
# List running processes
ps aux                         # All processes
ps aux | grep nginx            # Find specific process

# Real-time process monitor
top                            # Press 'q' to exit
htop                           # Better interface (if installed)

# Kill process
kill 1234                      # Kill process ID 1234
kill -9 1234                   # Force kill
pkill nginx                    # Kill by process name

# Run process in background
command &                      # Append & to background it

# Get job info
jobs                           # List background jobs
fg                             # Bring to foreground
bg                             # Continue in background
```

### System Information

```bash
# Kernel and system info
uname -a                       # All system info
uname -s                       # Kernel name
uname -r                       # Kernel release

# Hostname
hostname                       # System hostname
whoami                         # Current user
id                            # User ID and groups

# Uptime
uptime                        # System uptime
date                          # Current date/time

# CPU info
nproc                         # Number of CPUs
lscpu                         # Detailed CPU info

# Memory usage
free -h                       # Memory in human format
free -m                       # Memory in MB

# Disk usage
df -h                         # Mounted filesystems
du -sh /path                  # Directory size

# Network interfaces
ip addr show                  # IP addresses
ip route show                 # Routing table
ifconfig                      # Legacy command (older Linux)
```

### Piping & Redirection

```bash
# Redirect output to file
command > output.txt          # Overwrite file
command >> output.txt         # Append to file

# Redirect errors
command 2> errors.txt         # Redirect stderr
command > output.txt 2>&1     # Both stdout and stderr

# Pipe (|) output of one command to another
ps aux | grep nginx
cat file.txt | grep "ERROR" | wc -l

# Chain multiple pipes
cat log.txt | grep "ERROR" | head -n 10 | sort
```

### File Permissions (Review)

```bash
# View permissions
ls -la file.txt

# Change permissions
chmod 644 file.txt            # rw-r--r--
chmod 755 script.sh           # rwxr-xr-x
chmod 600 private.txt         # rw-------

# Change owner
chown username file.txt
chown username:groupname file.txt

# Recursive
chmod -R 755 directory/
chown -R username:groupname directory/
```

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

## Practice Exercises

1. Navigate to `/etc` directory
2. List all files with detailed permissions
3. Find all files larger than 1MB
4. Count lines in all `.conf` files
5. Create directory structure: `~/projects/security/week1`
6. Create a file with multiline content using `cat > file.txt << EOF`
7. Search for `root` in passwd file
8. Monitor running processes with `top`

---

**Next:** [Part 3: Users, Groups & Permissions](../Part-3/Part-3-Users-Groups-Permissions.md)
