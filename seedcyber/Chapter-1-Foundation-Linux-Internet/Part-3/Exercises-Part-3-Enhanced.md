# Part 3: Users, Groups & Permissions - Enhanced Exercises

## Exercise 3.1: Understanding Your System's Users

**Objective:** Explore the multi-user nature of your Linux system and understand user classification.

### Part A: Discover All Users

```bash
# View all users on system
cat /etc/passwd | head -20

# Output will look like:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
...

# ANALYSIS QUESTIONS:
# 1. How many users have UID < 1000? (System users)
# 2. How many have login shell /usr/sbin/nologin? (Service accounts)
# 3. How many have UID >= 1000? (Regular users)

# Task: Count them!
cat /etc/passwd | grep "/usr/sbin/nologin" | wc -l  # How many service accounts?
cat /etc/passwd | grep "/bin/bash" | wc -l          # How many can actually log in?
```

**Deliverable:** 
- Document showing:
  - Total service accounts on your system
  - Total regular users (UID >= 1000)
  - Total system accounts
  - List 5 service users and explain WHY they exist

---

## Exercise 3.2: Permission Notation Breakdowns

**Objective:** Master permission notation and understand what each means.

### Part A: Parse Permission Strings

Given these files, convert them between formats:

```bash
# File 1: Regular document
-rw-r--r-- alice bob document.txt

# What permissions are these?
# ANALYSIS:
# ├─ -rw- (owner alice) = ?
# ├─ r-- (group bob) = ?
# ├─ r-- (others) = ?
# └─ Numeric form = ?

# Answer: 644
#  ├─ 6 = rw- (4+2+0) = owner
#  ├─ 4 = r-- (4+0+0) = group
#  └─ 4 = r-- (4+0+0) = others

# File 2: Executable script
-rwxr-xr-x root root script.sh
# Numeric form = 755

# File 3: Private data
-rw------- alice alice secret.txt
# Numeric form = 600

# File 4: Directory for team project
drwxrwx--- alice developers project_folder
# Numeric form = 770
```

### Part B: Practical Permission Setting

```bash
# Create test files
mkdir test_permissions
cd test_permissions

# Create different file types
echo "This is readable" > readable.txt
echo "#!/bin/bash\necho 'Hello'" > script.sh
echo "Private stuff" > secret.txt

# Task 1: Make readable.txt world-readable but not writable
chmod 644 readable.txt
ls -l readable.txt
# Expected: -rw-r--r-- 

# Task 2: Make script.sh executable by everyone
chmod 755 script.sh
ls -l script.sh
# Expected: -rwxr-xr-x
# Then test: ./script.sh (should run!)

# Task 3: Make secret.txt ONLY readable by owner
chmod 600 secret.txt
ls -l secret.txt
# Expected: -rw-------
# Try: cat secret.txt (works)
# Try: sudo -u different_user cat secret.txt (should fail!)

# Task 4: Directory permissions
mkdir shared_work
chmod 755 shared_work     # Others can enter but not modify
ls -ld shared_work
# Expected: drwxr-xr-x

mkdir private_work
chmod 700 private_work    # Only owner can access
ls -ld private_work
# Expected: drwx------
```

**Deliverable:**
- Screenshot showing final permissions of each file
- Explain why each permission setting makes sense for its use case

---

## Exercise 3.3: User and Group Management Scenarios

**Objective:** Practice creating users, adding groups, and managing permissions for real-world scenarios.

### Scenario: Multi-user Web Hosting Environment

You manage a web hosting company with three customers:
- **Company A:** Sales team website (3 employees)
- **Company B:** E-commerce site (5 employees)
- **Company C:** Blog (1 person)

Each needs separate access, but shares the same server.

### Part A: Create User Structure

```bash
# Create groups for each company
sudo groupadd company_a
sudo groupadd company_b
sudo groupadd company_c

# Create users for Company A
sudo useradd -m -s /bin/bash -c "Company A - Alice" alice_a
sudo useradd -m -s /bin/bash -c "Company A - Bob" bob_a
sudo useradd -m -s /bin/bash -c "Company A - Charlie" charlie_a

# Add them to company_a group
sudo usermod -aG company_a alice_a
sudo usermod -aG company_a bob_a
sudo usermod -aG company_a charlie_a

# Set passwords (required to log in)
# In real scenario, would use proper onboarding
sudo passwd alice_a  # (Enter password)
sudo passwd bob_a
sudo passwd charlie_a

# Similar for Company B (create 3 users)
sudo useradd -m -s /bin/bash -c "Company B - Diana" diana_b
sudo useradd -m -s /bin/bash -c "Company B - Eve" eve_b
sudo useradd -m -s /bin/bash -c "Company B - Frank" frank_b

sudo usermod -aG company_b diana_b
sudo usermod -aG company_b eve_b
sudo usermod -aG company_b frank_b

# Company C (single user)
sudo useradd -m -s /bin/bash -c "Company C - Grace" grace_c
sudo usermod -aG company_c grace_c

# Verification
cat /etc/passwd | grep "_a\|_b\|_c"  # See all new users
cat /etc/group | grep "company_"    # See all groups
```

### Part B: File Organization and Permissions

```bash
# Create web directory structure
sudo mkdir -p /var/www/clients
sudo mkdir -p /var/www/clients/company_a/website
sudo mkdir -p /var/www/clients/company_b/website
sudo mkdir -p /var/www/clients/company_c/website

# Company A directory setup
sudo chown -R alice_a:company_a /var/www/clients/company_a/
sudo chmod -R 770 /var/www/clients/company_a/
ls -ld /var/www/clients/company_a/
# Expected: drwxrwx--- alice_a company_a

# Principle: Each Company A member can read/write
# Company B members CANNOT access

# Company B directory setup  
sudo chown -R diana_b:company_b /var/www/clients/company_b/
sudo chmod -R 770 /var/www/clients/company_b/

# Company C directory setup (single user)
sudo chown -R grace_c:company_c /var/www/clients/company_c/
sudo chmod -R 700 /var/www/clients/company_c/
# grace_c only person who can access

# Test access control!
# Simulate alice_a accessing her company's files
sudo -u alice_a ls -la /var/www/clients/company_a/website
# Should work!

# Try alice_a accessing company_b files  
sudo -u alice_a ls -la /var/www/clients/company_b/website
# Should fail with "Permission denied"

# Verify in output
```

### Part C: Audit and Documentation

```bash
# Create a security audit report
cat > /tmp/user_audit.txt << 'EOF'
=== MULTI-USER HOSTING AUDIT ===

Users by Company:
EOF

# List Company A users
echo -e "\nCOMPANY A:" >> /tmp/user_audit.txt
groups alice_a bob_a charlie_a >> /tmp/user_audit.txt

echo -e "\nCOMPANY B:" >> /tmp/user_audit.txt
groups diana_b eve_b frank_b >> /tmp/user_audit.txt

echo -e "\nCOMPANY C:" >> /tmp/user_audit.txt
groups grace_c >> /tmp/user_audit.txt

# Check file permissions
echo -e "\n=== FILE PERMISSIONS ===" >> /tmp/user_audit.txt
ls -ld /var/www/clients/company_*/ >> /tmp/user_audit.txt

# View the audit
cat /tmp/user_audit.txt
```

**Deliverable:**
- Command history showing user and group creation
- Permission verification (screenshots)
- Security audit report
- Answer: Can alice_a read company_b files? Why/why not?

---

## Exercise 3.4: File Ownership Security Incident

**Objective:** Investigate and fix permission problems that cause real security issues.

### Scenario: Web Server File Permissions Crisis

A web server runs as user `www-data`, and someone left files writable by everyone!

```bash
# Create the problematic scenario
mkdir -p /tmp/web_crisis/
cd /tmp/web_crisis/

# Create files with WRONG permissions
echo "<?php echo 'Website'; ?>" > index.php
echo "database_password=super_secret123" > config.php
echo "secret_api_key=abc123" > api_keys.json

# Set dangerously open permissions (what NOT to do!)
chmod 666 *  # Everyone can read AND write!
ls -la

# Problem: File contents visible/modifiable by anyone!

# INVESTIGATION PHASE
echo "=== SECURITY AUDIT ==="

# 1. Find world-writable files (DANGEROUS!)
find /tmp/web_crisis -perm 777 -o -perm 666
# Should find our 3 files

# 2. Check who owns them (probably your user)
ls -la /tmp/web_crisis/
# Problem: Should be owned by www-data!

# 3. Understand the risks
cat << 'EOF'
CURRENT STATE - COMPLETELY EXPOSED:
✗ config.php is world-readable (database password visible!)
✗ api_keys.json is world-readable (API keys stolen!)
✗ index.php is world-writable (attacker could modify!)
✗ Wrong owner (should be www-data, not your user!)

RISKS:
├─ Database password leaked
├─ API keys compromised
├─ Attacker could modify website
├─ SQL injection is now possible
└─ CRITICAL SECURITY BREACH!
EOF
```

### Part B: Fix the Permissions

```bash
# Step 1: Fix ownership
# Files should be owned by user who manages them
# But web server should only read them

# Option A: Owner is developer, group is www-data
sudo chown -R your_user:www-data /tmp/web_crisis/
# (Replace your_user with your username)

# Step 2: Fix permissions
# ├─ Owner needs to modify (rw)
# ├─ www-data needs to read only (r)
# └─ Others need nothing (no permissions)

sudo chmod 640 /tmp/web_crisis/config.php
sudo chmod 640 /tmp/web_crisis/api_keys.json
sudo chmod 644 /tmp/web_crisis/index.php  # Public PHP file

ls -la /tmp/web_crisis/
# Expected:
# -rw-r----- developer www-data config.php
# -rw-r----- developer www-data api_keys.json
# -rw-r--r-- developer www-data index.php

# Step 3: Verify fix
echo "=== AFTER FIX ==="
echo "Can www-data read config.php? (should be yes)"
sudo -u www-data cat /tmp/web_crisis/config.php
# Should work (readable by group www-data)

echo "Can www-data write config.php? (should be no)"  
sudo -u www-data cp config.php /tmp/backup.php 2>&1 | grep "Permission denied"
# Should fail (not writable by group)

echo "Can others read config.php? (should be no)"
cat /tmp/web_crisis/config.php 2>&1 | grep "Permission denied"
# Should fail for non-www-data users
```

### Part C: Implement Best Practice

```bash
# Create a deployment script that fixes permissions automatically
cat > /tmp/fix_web_permissions.sh << 'EOF'
#!/bin/bash
# Fix web application permissions for security

WEB_ROOT="${1:-.}"

echo "Fixing permissions in: $WEB_ROOT"

# Set ownership to developer:www-data
sudo chown -R $USER:www-data "$WEB_ROOT"

# Remove all permissions first
find "$WEB_ROOT" -type f -exec chmod 000 {} \;
find "$WEB_ROOT" -type d -exec chmod 000 {} \;

# Directories: owner full, group rx, others none
find "$WEB_ROOT" -type d -exec chmod 750 {} \;

# Config/secret files: owner rw, group r, others none  
find "$WEB_ROOT" -name "*.conf" -o -name "*.config" -o -name "config.*" | \
  xargs -r chmod 640

# Regular files: owner rw, group r, others r (public)
find "$WEB_ROOT" -type f -name "*.php" -o -name "*.html" -o -name "*.css" | \
  xargs -r chmod 644

# API keys NEVER readable by others
find "$WEB_ROOT" -name "*api*" -o -name "*secret*" -o -name "*password*" | \
  xargs -r chmod 640

echo "Permissions fixed!"
find "$WEB_ROOT" -type f -ls | head -20  # Show sample
EOF

chmod +x /tmp/fix_web_permissions.sh

# Run it!
/tmp/fix_web_permissions.sh /tmp/web_crisis/
```

**Deliverable:**
- Screenshots showing BEFORE (world-writable) and AFTER (secure)
- Permissions audit showing fixed settings
- Explanation of why each setting is correct
- Script that can be reused for production web servers

---

## Exercise 3.5: SSH Key Permissions (Critical Security!)

**Objective:** Understand proper SSH key ownership and permissions.

### Scenario: SSH Key Security

SSH private keys are your digital identity. Wrong permissions = account compromise!

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N "passphrase"

# Check what was created
ls -la ~/.ssh/
# Output might be:
# -rw------- user user id_rsa        (PRIVATE KEY - SECRET!)
# -rw-r--r-- user user id_rsa.pub    (PUBLIC KEY - OK to share)

# CRITICAL ANALYSIS
echo "=== SSH KEY SECURITY ==="
echo "Private key: $(stat -c '%A' ~/.ssh/id_rsa)"
# Should be: -rw------- (600)
# Meaning: Only owner can read!

echo "Public key: $(stat -c '%A' ~/.ssh/id_rsa.pub)"
# Should be: -rw-r--r-- (644)
# Meaning: Everyone can read (safe to share)

# WHAT HAPPENS IF PRIVATE KEY PERMISSIONS ARE WRONG?

# Simulate bad permissions
sudo chmod 644 ~/.ssh/id_rsa  # OOPS! World readable!
ls -la ~/.ssh/id_rsa
# -rw-r--r-- (644) - ANYONE CAN READ!

# Try to use it - SSH will REFUSE!
ssh user@server.com
# Output: "WARNING: UNPROTECTED PRIVATE KEY FILE!"
# "It is required that your private key files are NOT accessible by others"
# Connection will be refused!

# Fix it back
chmod 600 ~/.ssh/id_rsa
ls -la ~/.ssh/id_rsa
# -rw------- (600) - Safe again!

# Now SSH works
```

### Part B: Directory Permissions Matter Too!

```bash
# Often overlooked: .ssh directory permissions!

# Correct permissions
chmod 700 ~/.ssh
ls -ld ~/.ssh
# Expected: drwx------ (700)
# Meaning: Only owner can enter directory!

# What if directory is world-readable?
chmod 755 ~/.ssh  # WRONG!
ls -ld ~/.ssh

# Try SSH - will fail!
ssh user@server.com
# "WARNING: UNPROTECTED PRIVATE KEY FILE!"
# Reason: SSH sees that OTHERS can read ~/.ssh directory
# So they COULD read the private key inside!

# The fix:
chmod 700 ~/.ssh
ls -ld ~/.ssh
# drwx------ - Now safe!
```

### Part C: Authorized Keys Permissions

```bash
# When accepting SSH connections, server checks:
# ~/.ssh/authorized_keys

# This file should be:
chmod 600 ~/.ssh/authorized_keys
ls -la ~/.ssh/authorized_keys
# -rw------- (600)

# Why?
# ├─ Contains public keys of people allowed to connect
# ├─ If world-writable, attacker could add their key!
# ├─ Attacker then has permanent SSH access
# └─ Must be read-only by owner!

# Check your server's authorized_keys
cat ~/.ssh/authorized_keys
# Should show lines like:
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB... user@hostname
```

**Deliverable:**
- SSH key permissions check showing proper values (600 for private, 644 for public)
- .ssh directory permissions (700)
- authorized_keys permissions (600)
- Document explaining what happens if these are wrong

---

## Exercise 3.6: Sudoers File and Privilege Escalation

**Objective:** Understand how sudo works and configure it safely.

### Part A: Understanding Your Sudo Access

```bash
# Check what you can run with sudo
sudo -l

# Output might be:
# User user may run the following commands (user-ALL=(ALL:ALL) ALL):
#     (ALL:ALL) ALL
# Meaning: Can run ANYTHING with sudo!

# More restrictive example:
# User deployuser may run the following commands (deployuser-localhost=(ALL) NOPASSWD:/usr/bin/systemctl):
#     (ALL) NOPASSWD: /usr/bin/systemctl
# Meaning: Can only run systemctl without password

# Check sudo log (which commands were run)
sudo journalctl -u sudo
# Shows history of all sudo commands!

# CHECK: Who can use sudo?
getent group sudo
# Or: cat /etc/group | grep sudo
# Shows: sudo:x:27:user1,user2
# Meaning: user1 and user2 can use sudo
```

### Part B: Test Privilege Escalation

```bash
# Create a scenario where you need elevated privileges

# Scenario: System administrator needs to restart services
# But doesn't need full sudo access

# First, let's see what requires privilege
systemctl restart nginx
# Fails: "Failed to restart nginx.service: Access denied"

# With sudo it works
sudo systemctl restart nginx
# Works! (if nginx installed)

# But giving full sudo is dangerous - what if user's account compromised?
# Better: Allow ONLY specific commands

# (This requires editing /etc/sudoers - see Part C)
```

### Part C: Audit User Privileges (Read-Only)

```bash
# List all users in sudoers group
getent group sudo 2>/dev/null || echo "sudo group not found"

# Find all users with UID >= 1000
getent passwd | awk -F: '$3 >= 1000 {print $1}'
# These are regular users

# Create a privilege audit report
cat > /tmp/privilege_audit.sh << 'EOF'
#!/bin/bash
echo "=== PRIVILEGE AUDIT REPORT ==="
echo ""
echo "Users with sudo access:"
getent group sudo | cut -d: -f4 | tr ',' '\n'

echo ""
echo "Users with UID >= 1000 (normal users):"
getent passwd | awk -F: '$3 >= 1000 {print $1 " (UID: " $3 ")"}'

echo ""
echo "System accounts (UID < 1000):"
getent passwd | awk -F: '$3 < 1000 && $3 != 0 {print $1 " (UID: " $3 ")"}'

echo ""
echo "Service accounts without login shell:"
getent passwd | grep "nologin" | wc -l
echo "Total service accounts"
EOF

chmod +x /tmp/privilege_audit.sh
/tmp/privilege_audit.sh
```

**Deliverable:**
- Output of `sudo -l` showing your permissions
- Privilege audit showing all users and their access levels
- Analysis: Who has sudo? Is it appropriate?

---

## Exercise 3.7: Security Audit - Find Permission Problems

**Objective:** Scan system for common permission security issues.

### Create Audit Scripts

```bash
# Script 1: Find world-writable files (DANGEROUS!)
cat > /tmp/find_world_writable.sh << 'EOF'
#!/bin/bash
echo "=== WORLD-WRITABLE FILES (SECURITY RISK!) ==="
echo "These files are writable by ANYONE - potential attack vector:"
echo ""

# Search common web directories
find /var/www -perm -002 2>/dev/null | head -20

# Search home directories
find /home -perm -002 2>/dev/null | head -20

# Search /tmp (usually OK since /tmp is temporary)
# But still dangerous for long-lived files
find /tmp -perm -002 -type f 2>/dev/null | head -20

echo ""
echo "Total world-writable files found:"
find / -perm -002 -type f 2>/dev/null | wc -l
EOF

chmod +x /tmp/find_world_writable.sh

# Script 2: Find files with exposed secrets in name
cat > /tmp/find_potential_secrets.sh << 'EOF'
#!/bin/bash
echo "=== FILES WITH SUSPICIOUS NAMES (Possible Secrets) ==="

find ~ -type f \( \
  -name "*password*" -o \
  -name "*secret*" -o \
  -name "*api*key*" -o \
  -name "*.key" -o \
  -name "*.pem" -o \
  -name "*credential*" \
2>/dev/null \) | while read file; do
  perms=$(stat -c "%A" "$file" 2>/dev/null)
  echo "$perms - $file"
done

echo ""
echo "Check: Are secret files world-readable? (They shouldn't be!)"
EOF

chmod +x /tmp/find_potential_secrets.sh

# Run the audit
echo "Running security audits..."
/tmp/find_world_writable.sh
echo ""
echo "---"
echo ""
/tmp/find_potential_secrets.sh
```

### Create Security Checklist

```bash
cat > /tmp/security_checklist.txt << 'EOF'
LINUX PERMISSIONS SECURITY CHECKLIST
====================================

[  ] SSH keys present and readable only by owner (600)
[  ] .ssh directory readable only by owner (700)
[  ] No world-writable files in home directory
[  ] Web application files not world-writable
[  ] Database config files readable only by DB user
[  ] API keys not included in source control
[  ] Private files not group-readable unnecessarily
[  ] No setuid bits on user-created binaries
[  ] sudo access limited to necessary users
[  ] sudo commands match job responsibilities
[  ] Old user accounts removed when people leave
[  ] Regular permission audits performed
[  ] Service accounts have no login shell
EOF

cat /tmp/security_checklist.txt
```

**Deliverable:**
- Run permission audit, document any findings
- Report showing:
  - Any world-writable files (and whether they're problematic)
  - Any suspicious secret files
  - Assessment of current system state
- Security checklist with self-assessment

---

## Exercise Summary and Key Takeaways

```
CRITICAL SECURITY PRINCIPLES FROM EXERCISES:

1. Users & Groups create security boundaries
   ├─ Separate users prevent cross-access
   ├─ Groups enable shared work safely
   └─ Always follow principle of least privilege

2. Permissions are your ONLY protection
   ├─ Even one wrong digit (666 vs 644) is critical
   ├─ Directory permissions matter (755 vs 700)
   └─ test -r file is how Linux checks access

3. Ownership matters!
   ├─ Files should be owned by appropriate user
   ├─ Web server files ≠ root files ≠ user files
   └─ Wrong owner = wrong (or no) access

4. SSH keys are your digital identity
   ├─ Private key: MUST be 600 (owner only)
   ├─ .ssh directory: MUST be 700 (owner only)
   ├─ One wrong permission = account compromised
   └─ Linux will refuse to use if permissions wrong!

5. Audit regularly!
   ├─ Find world-writable files
   ├─ Check who has sudo
   ├─ Monitor permission changes
   └─ Remove old accounts promptly

6. sudo is privilege escalation tool
   ├─ Grant specific commands, not full sudo
   ├─ Always log and audit sudo usage
   ├─ Compromised account with sudo = full system compromise
   └─ "With great privilege comes great responsibility"
```

---

**Congratulations!** You now understand permissions at depth, from first principles. This knowledge will protect every Linux system you manage.
