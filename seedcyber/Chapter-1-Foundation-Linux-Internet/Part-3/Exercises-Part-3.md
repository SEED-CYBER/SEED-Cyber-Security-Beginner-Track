# Part 3 Exercises: Users, Groups & Permissions

**Time:** 2 hours  
**Platform:** Linux/macOS/WSL

## Exercise 3.1: User Account Lifecycle

**Objective:** Create, modify, delete users

```bash
# Create users
sudo useradd -m -s /bin/bash testuser1
sudo useradd -m -s /bin/bash testuser2

# Set passwords
sudo passwd testuser1
sudo passwd testuser2

# Verify creation
cat /etc/passwd | grep test

# Modify user
sudo usermod -c "Test User 1" testuser1

# Add to group
sudo usermod -aG sudo testuser1

# List user groups
groups testuser1

# Delete user (keep home)
sudo userdel testuser2

# Delete with home directory
sudo userdel -r testuser1
```

**Deliverable:**
- /etc/passwd entries for created users
- /etc/shadow entry (sanitized)
- Group membership verification
- Before/after of user modifications

## Exercise 3.2: Group Management

**Objective:** Create and manage groups

```bash
# Create group
sudo groupadd developers
sudo groupadd qa

# Create users for testing
sudo useradd -m -s /bin/bash dev1
sudo useradd -m -s /bin/bash dev2
sudo useradd -m -s /bin/bash qa1

# Add users to groups
sudo usermod -aG developers dev1
sudo usermod -aG developers dev2
sudo usermod -aG qa qa1

# View group membership
getent group developers
getent group qa

# Change user's primary group
sudo usermod -g qa dev1

# Verify
id dev1
groups dev1
```

**Deliverable:**
- Group list from /etc/group
- Group membership matrix
- Primary and secondary groups for each user

## Exercise 3.3: File Ownership & Permissions

**Objective:** Manage file-level access control

```bash
# Create test files
mkdir ~/access-test
cd ~/access-test

echo "Private" > private.txt
echo "Shared" > shared.txt
echo "Public" > public.txt

# View initial ownership
ls -la

# Create groups for testing
sudo groupadd project-team
sudo usermod -aG project-team $(whoami)

# Change ownership
chmod 600 private.txt      # Only owner
chmod 640 shared.txt       # Owner and group
chmod 644 public.txt       # Everyone reads

# Change group
sudo chown :project-team shared.txt

# Verify
ls -la

# Test access (newsh for group changes to take effect)
cat private.txt     # Works (you own it)
cat shared.txt      # Works (in group)
cat public.txt      # Works (everyone can read)
```

**Deliverable:**
- Permission listing (ls -la output)
- Explanation of each permission set
- Group ownership verification

## Exercise 3.4: Sudo Access & Privilege Escalation

**Objective:** Configure and use sudo safely

```bash
# View available sudo commands
sudo -l

# Create specific sudo rule
sudo visudo
# Add line: testuser ALL=(ALL) NOPASSWD:/usr/bin/systemctl restart nginx

# Test sudo without password (if configured)
sudo -u testuser sudo systemctl restart nginx

# Audit sudo usage
sudo grep "sudo:" /var/log/auth.log | tail -10

# Configure sudoers for specific user
# In visudo:
# user1 ALL=(ALL) ALL                    # All commands
# user2 ALL=(root) NOPASSWD:/bin/ls      # Specific command
# %admins ALL=(ALL) NOPASSWD:ALL         # All for group
```

**Deliverable:**
- Current sudoers configuration
- Explanation of each sudo rule
- Audit log showing sudo usage
- Security implications

## Exercise 3.5: Permission Security Analysis

**Objective:** Identify and fix permission issues

```bash
# Find files with overly permissive permissions
find ~ -perm 777    # Dangerous - all permissions
find ~ -perm 666    # Dangerous - world writable
find ~ -perm 644    # Safe - owner rw, others ro

# Find SUID binaries (potential privilege escalation)
sudo find / -perm -4000 2>/dev/null

# Check critical system files
ls -la /etc/passwd
ls -la /etc/shadow    # Should be 640 or 600
ls -la /etc/sudoers
ls -la ~/.ssh/id_rsa  # Should be 600
```

**Windows Guide (Concepts):**
```
Windows uses ACLs (Access Control Lists) instead of Unix permissions.
Concepts:
- NTFS permissions (similar to chmod)
- Ownership (User/Group similar to chown)
- Inheritance (similar to directory permissions applying to contents)

PowerShell commands:
Get-Acl filename
Set-Acl filename ACL
```

**Deliverable:**
- Permission scan results
- Security recommendations
- Fixes applied to dangerous files

## Exercise 3.6: Practical Workspace Isolation

**Objective:** Apply user/group security to shared workspace

```bash
# Simulate multi-user workspace
sudo mkdir -p /var/workspace/{shared,user1,user2}

# Create users
sudo useradd workspace-user1
sudo useradd workspace-user2
sudo groupadd workspace-team

# Add to group
sudo usermod -aG workspace-team workspace-user1
sudo usermod -aG workspace-team workspace-user2

# Set ownership and permissions
sudo chown workspace-user1:workspace-user1 /var/workspace/user1
sudo chown workspace-user2:workspace-user2 /var/workspace/user2
sudo chown root:workspace-team /var/workspace/shared

# Permissions
sudo chmod 700 /var/workspace/user1     # Private to user1
sudo chmod 700 /var/workspace/user2     # Private to user2
sudo chmod 770 /var/workspace/shared    # Group writable shared

# Verification
ls -la /var/workspace/
ls -la /var/workspace/user1/
ls -la /var/workspace/shared/
```

**Deliverable:**
- Directory listing showing isolation
- Explanation of each permission
- Proof that isolation works (can't access others' dirs)

---

**Completion Checklist:**
- [ ] User lifecycle commands understood
- [ ] Group management functional
- [ ] File permissions properly set
- [ ] Sudo configuration safe
- [ ] Permission security issues identified
- [ ] Workspace isolation working

**Submission:** Part-3-Exercises-SUBMISSION.txt
