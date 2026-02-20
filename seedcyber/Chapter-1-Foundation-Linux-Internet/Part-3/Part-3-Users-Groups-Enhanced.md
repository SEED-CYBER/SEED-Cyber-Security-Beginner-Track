# Part 3: Users, Groups & Permission Management - Enhanced

## The Permission Problem (Why This Exists)

### Multi-User Systems: The Core Challenge

**Fundamental question:** When multiple people use the same computer, how do you prevent them from interfering with each other?

```
Computer with 3 users:
  ├─ Alice (works on financial data)
  ├─ Bob (works on customer database)
  └─ Charlie (system administrator)

Requirements:
  ├─ Alice's data should be private
  ├─ Bob's data should be private
  ├─ Neither should accidentally (or maliciously) delete the other's work
  ├─ Charlie needs special admin powers
  └─ System files should be protected from users

Solution: LINUX PERMISSIONS
```

### The Three-Level Permission System

**Permission applies to THREE categories:**

```
FILE PERMISSIONS:
  -rw-r--r--
  │││││││││
  └┬┘└┬┘└┬┘
   │  │  └─ OTHERS (everyone else)
   │  └──── GROUP (users in file's group)
   └─────── OWNER (user who owns file)

Each category has 3 bits (read/write/execute)
```

---

## Linux User System: The Architecture

### Understanding Users at the Deepest Level

**What IS a "user" in Linux?**

```
A user is:
  ├─ An identity (username)
  ├─ A numeric ID (UID)
  ├─ A home directory
  ├─ A default group
  ├─ A login shell
  └─ A set of permissions/restrictions

Users serve to:
  ├─ Organize file ownership
  ├─ Control resource access
  ├─ Create audit trails
  ├─ Enforce security isolation
  └─ Enable multi-tenancy
```

### The Different Types of Users (Conceptual)

```
ROOT USER (UID 0):
  ├─ Can do ANYTHING
  ├─ No restrictions
  ├─ Can read/write all files
  ├─ Can kill any process
  ├─ Can install software
  ├─ Can change permissions
  └─ DANGEROUS: Only use when necessary!

SYSTEM USERS (UID 1-999):
  ├─ Run system services
  ├─ Examples:
  │   ├─ nginx (UID 33) runs web server
  │   ├─ mysql (UID 105) runs database
  │   └─ postgres (UID 109) runs database
  ├─ Usually no login shell
  ├─ Usually no home directory
  └─ Principle: Each service has its own user

REGULAR USERS (UID 1000+):
  ├─ Real people who log in
  ├─ Have home directory (/home/username)
  ├─ Have login shell (usually /bin/bash)
  ├─ Can own files
  ├─ Limited permissions (can't access others' files)
  └─ Can't install software globally (usually)
```

### Why Multiple Users Matters for Security

```
SCENARIO: Web server compromised
  ├─ Attacker gains control of nginx process
  ├─ Nginx runs as user "www-data"
  ├─ Attacker can only access files owned by www-data
  ├─ Can't read admin files (owned by root)
  ├─ Can't read user Alice's documents (owned by alice)
  └─ Damage CONTAINED!

VS. if everything ran as root:
  ├─ Same compromise happens
  ├─ Attacker now has ROOT access
  ├─ Can read EVERYTHING
  ├─ Can delete EVERYTHING
  ├─ Can install backdoors
  └─ TOTAL SYSTEM COMPROMISE!

Security principle: Run everything with MINIMUM privilege
```

---

## Understanding Users and Groups Files

### /etc/passwd: User Identity Database

**What it is:** Text file that maps usernames to UIDs and home directories

```
Format: username:password:UID:GID:GECOS:home:shell

Example:
root:x:0:0:root:/root:/bin/bash
  ├─ username: root
  ├─ password: x (means it's in /etc/shadow)
  ├─ UID: 0 (root user)
  ├─ GID: 0 (root group)
  ├─ GECOS: root (description/full name)
  ├─ home: /root (root's home directory)
  └─ shell: /bin/bash (what shell to use)

ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
  ├─ username: ubuntu
  ├─ UID: 1000 (first regular user)
  ├─ home: /home/ubuntu (separate from root)
  └─ shell: /bin/bash (can log in)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
  ├─ username: www-data (web server user)
  ├─ UID: 33
  ├─ home: /var/www (web directory)
  └─ shell: /usr/sbin/nologin (CAN'T LOG IN INTERACTIVELY!)
     └─ Security: Service account can't be used to break in
```

**Why readable by everyone:**
- Login processes need to look up username → UID
- Must be readable to work at all
- **Passwords are NOT here** (security!)

### /etc/shadow: Encrypted Password Storage

**What it is:** Password hashes, readable only by root

```
Format: username:password_hash:last_change:min:max:warn:grace:disable

Example:
ubuntu:$6$salt$...long.hash...encrypted:18974:0:99999:7:::
  ├─ $6$ = SHA-512 encryption method
  ├─ salt = random data added for security
  ├─ ...hash = one-way encrypted password
  ├─ 18974 = days since Jan 1 1970 (password last changed)
  ├─ 0 = minimum days before can change again
  ├─ 99999 = maximum age before must change
  ├─ 7 = days warning before expiration
  ├─ (empty) = grace period
  └─ (empty) = disabled (unset)

Why one-way encrypted?:
  ├─ File might be compromised
  ├─ Attacker can read hashes
  ├─ But can't reverse hash to get password
  ├─ Can only brute-force (try billions of guesses)
  └─ Good hashing + salt + strong passwords = secure!

Why only root reads it?:
  ├─ Attackers often get limited account access
  ├─ Can't read /etc/shadow without root
  ├─ Can't easily steal passwords
  └─ Root must approve password changes
```

### /etc/group: Group Membership Database

**What it is:** Maps group names to GIDs and members

```
Format: groupname:password:GID:members

Example:
root:x:0:
  ├─ Group name: root
  ├─ GID: 0
  └─ members: (empty, but root user is primary member)

ubuntu:x:1000:alice,bob
  ├─ Group name: ubuntu
  ├─ GID: 1000
  ├─ members: alice, bob (can access files owned by ubuntu group)
  └─ Useful for: Shared project files

sudo:x:27:ubuntu,alice
  ├─ Group name: sudo
  ├─ members: ubuntu, alice
  └─ Special meaning: Members can run commands with sudo!
     (Controlled by /etc/sudoers)
```

---

## The Permission System at Depth

### Permission Bits: Breaking Down -rw-r--r--

```
-rw-r--r--
│││││││││
│││││││││
││└────┬──┘ OTHERS (everyone else) = r-- = read-only (4)
││├───┬─┘   GROUP = r-- = read-only (4)
││└┬┘       OWNER = rw- = read+write (6)
│└─ FILE TYPE:
│    ├─ - = regular file
│    ├─ d = directory
│    ├─ l = symbolic link
│    └─ c = character device
│
└─ (position 0 is the type bit)

NUMERIC REPRESENTATION:
r = 4 (read)
w = 2 (write)
x = 1 (execute)

-rw-r--r-- = 644
 │││││││││
 ├─ 6 (rw-) = 4+2+0 = Owner
 ├─ 4 (r--) = 4+0+0 = Group
 └─ 4 (r--) = 4+0+0 = Others

EXAMPLE: 755
 ├─ 7 (rwx) = 4+2+1 = Owner (all permissions)
 ├─ 5 (r-x) = 4+0+1 = Group (read + execute)
 └─ 5 (r-x) = 4+0+1 = Others (read + execute)
```

### What Each Permission Means (Different for Files vs Directories)

#### For REGULAR FILES:

```
READ (r):
  ├─ Owner: Can view/open file
  ├─ Group: Can view/open file
  ├─ Others: Can view/open file
  └─ Security: File contents visible if readable

WRITE (w):
  ├─ Owner: Can modify/delete file
  ├─ Group: Can modify/delete file
  ├─ Others: Can modify/delete file
  └─ Security: DANGEROUS to give write to untrusted users!

EXECUTE (x):
  ├─ Owner: Can run file as program
  ├─ Group: Can run file as program
  ├─ Others: Can run file as program
  └─ Security: Don't give execute unless file is a program!
```

#### For DIRECTORIES:

```
READ (r):
  ├─ Owner: Can list directory contents (ls)
  ├─ Group: Can list directory contents
  ├─ Others: Can list directory contents
  └─ If NOT set: Can't use 'ls' to see what's inside!

WRITE (w):
  ├─ Owner: Can create/delete files in directory
  ├─ Group: Can create/delete files in directory
  ├─ Others: Can create/delete files in directory
  └─ VERY DANGEROUS: Users could delete each other's files!

EXECUTE (x):
  ├─ Owner: Can ENTER directory (cd into it)
  ├─ Group: Can ENTER directory
  ├─ Others: Can ENTER directory
  └─ If NOT set: Can't cd, can't access files inside!
     └─ Even if you could read/write files inside!

COMMON PERMISSION COMBOS:

drwx------  (700)
  ├─ Owner: full access
  ├─ Group: no access
  ├─ Others: no access
  └─ Use case: Private directory (only owner can access)

drwxr-xr-x  (755)
  ├─ Owner: full access
  ├─ Group: can read and enter
  ├─ Others: can read and enter
  └─ Use case: Shared directory (others can see but not modify)

drwxrwx---  (770)
  ├─ Owner: full access
  ├─ Group: full access
  ├─ Others: no access
  └─ Use case: Shared team directory
```

---

## Special Permissions (The Advanced Bits)

### SETUID (Set User ID)

**What it does:** Binary runs AS THE OWNER, not the user who started it!

```
EXAMPLE: passwd command
-rwsr-xr-x  root  passwd
 ││││││││
 │││││││└─ Others can execute
 ││││││└── Group can execute
 │││││└─── Owner can execute
 ││││└──── Owner can write
 │││└───── Owner can read
 ││└────── FILE TYPE (regular file)
 │└─────── SPECIAL BIT: SETUID (enabled)
 └──────── FILE TYPE bit

WHY NEEDED?
  ├─ passwd command needs to write to /etc/shadow
  ├─ /etc/shadow owned by root
  ├─ Regular users can't write there
  ├─ BUT: passwd has setuid bit set
  ├─ So when user runs passwd: runs AS ROOT
  ├─ Can write to /etc/shadow
  ├─ Then returns to normal user
  └─ Secure: Only specific program has elevated privileges

EXAMPLE: sudo command
-rwsr-xr-x  root  sudo
  ├─ Runs as root regardless of who runs it
  ├─ Can do admin tasks
  ├─ Logs what user did
  └─ Controlled by /etc/sudoers configuration

SECURITY DANGER:
  ├─ If setuid binary has vulnerability
  ├─ Attacker can exploit it with elevated privileges
  ├─ Serious security risk!
  └─ Never carelessly set setuid on programs!
```

### SETGID (Set Group ID)

**For files:** Similar to SETUID but runs as the GROUP
**For directories:** New files inherit directory's group instead of user's group

```
EXAMPLE: Directory with setgid
drwxrws--- developers shared_project
 ┌──────────────────── SETGID bit enabled

When alice (in developers group) creates file:
  ├─ File created as: alice:developers (not alice:alice)
  ├─ Makes teamwork easier
  └─ Everyone's files are in same group

Use case:
  ├─ Shared project directory
  ├─ All members need to access each other's files
  ├─ Setgid ensures consistent group ownership
  └─ No need to remember to change group permission
```

### Sticky Bit

**For directories:** Only owner (and root) can delete files in it

```
Example: /tmp directory
drwxrwxrwt root root /tmp
        │
        └─ Sticky bit enabled

Why needed?:
  ├─ /tmp is writable by EVERYONE
  ├─ Without sticky bit: Anyone can delete anyone's files!
  ├─ With sticky bit: Only owner can delete their own files
  ├─ Alice can't delete Bob's temp files
  └─ Prevents sabotage

How to set:
chmod +t directory/  # Add sticky bit

Or numerically:
chmod 1777 directory/  # 1 = sticky, 777 = rwx all
```

---

## User Management Commands (with Context)

### Creating Users

```bash
# Simple user creation
sudo useradd username
# Creates user, but:
# ├─ No password set (can't log in)
# ├─ No home directory
# └─ Uses /bin/sh (missing features)

# Proper user creation
sudo useradd -m -s /bin/bash username
# ├─ -m = create home directory
# ├─ -s /bin/bash = use bash shell
# └─ Creates /home/username

# Set password (required!)
sudo passwd username
# Prompts for password
# Encrypts and stores in /etc/shadow

# Better: All at once
sudo useradd -m -s /bin/bash -c "Full Name" newuser
# ├─ -c = comment (full name)
# └─ Then run: sudo passwd newuser
```

### Adding Users to Groups

```bash
# Add to secondary group (append)
sudo usermod -aG groupname username
# ├─ -a = append (keep existing groups)
# ├─ -G = supplementary groups
# └─ User now member of groupname

# Example: Add user to sudo group
sudo usermod -aG sudo username
# User can now use sudo!

# View user's groups
groups username
# Output: username : groupname1 groupname2 groupname3

# Change primary group (DANGEROUS!)
sudo usermod -g primarygroup username
# ├─ Don't do this usually
# └─ Can break file ownership
```

### Modifying Permissions

```bash
# Set owner+group write, others read only
chmod 664 file.txt
# -rw-rw-r--

# Set owner full, others none (private)
chmod 700 file.txt
# -rwx------

# Set for directory and all contents
chmod -R 755 directory/
# ├─ -R = recursive
# └─ Changes directory and all files/subdirs

# Using symbolic (more intuitive)
chmod u+x file.txt      # Add execute for owner
chmod g-w file.txt      # Remove write for group
chmod o+r file.txt      # Add read for others
chmod a+x script.sh     # Add execute for all
```

### Changing Ownership

```bash
# Change owner
sudo chown newuser file.txt
# file.txt now owned by newuser

# Change owner and group
sudo chown newuser:newgroup file.txt
# In group newgroup

# Recursive (whole directory)
sudo chown -R newuser:newgroup directory/

# Practical example: Fix web server files
sudo chown -R www-data:www-data /var/www/html
# Web server can now read/write files
```

---

## Security Best Practices with Permissions

### Principle: Defense in Depth

```
Layer 1: Separate users
  ├─ Don't run multiple services as root
  ├─ Each service has own user
  └─ Compromised service ≠ total compromise

Layer 2: Principle of Least Privilege
  ├─ Every user gets MINIMUM permissions needed
  ├─ Alice doesn't need to read /root
  ├─ Web server doesn't need to change passwords
  └─ Limits damage if user compromised

Layer 3: Group-based access
  ├─ Use groups for shared access
  ├─ Better than giving permissions to individuals
  └─ Easier to manage large teams

Layer 4: Regular permission audits
  ├─ Check for unintended world-writable files
  ├─ Review who can use sudo
  ├─ Remove unused accounts
  └─ Update permissions when roles change
```

### Security Anti-patterns (Don't Do These!)

```
❌ Run everything as root
  └─ One compromise = total system compromise

❌ User permissions too permissive (777)
  └─ Anyone can read/modify/delete files

❌ Shared passwords for accounts
  └─ Can't track who did what (audit trail lost)

❌ Never change passwords
  └─ Old employees still have access

❌ Use sudo without logging
  └─ Can't investigate security incidents

❌ Giving sudo to untrusted users
  └─ Becomes equivalent to root access
```

### Security Best Practices (Do These!)

```
✓ Use sudo carefully
  └─ Only for necessary commands
  └─ Log and audit sudo usage

✓ Regular security audits
  └─ Find world-writable files: find / -perm 777
  └─ Check sudo access: sudo -l
  └─ Review active users: w, last

✓ Remove old accounts
  └─ When employee leaves: userdel -r username
  └─ Don't leave orphaned files

✓ Use groups for shared access
  └─ More manageable than individual permissions
  └─ Don't use world-readable/writable

✓ SSH key-only authentication
  └─ Better than passwords
  └─ Requires proper permissions (600 for private key!)

✓ Monitor for privilege escalation
  └─ Check for setuid files: find / -perm -4000
  └─ Disable if not needed
```

---

This completes the conceptual foundation of users, groups, and permissions in Linux security.
