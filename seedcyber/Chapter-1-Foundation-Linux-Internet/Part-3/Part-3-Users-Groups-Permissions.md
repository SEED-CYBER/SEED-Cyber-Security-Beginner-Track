# Part 3: Users, Groups & Permission Management

## Understanding Linux User System

Linux supports multiple users with separate permissions and file ownership.

## User Types

```
Root (UID 0)
└─ Unrestricted access to everything

System Users (UID < 1000)
└─ For running services (nginx, mysql, etc)
└─ Usually no login shell

Regular Users (UID >= 1000)
└─ Real people who can log in
└─ Home directory and shell
```

## User & Group Files

### /etc/passwd

Stores user account information:

```
username:password:UID:GID:GECOS:home:shell
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
```

- `x` indicates password stored in `/etc/shadow` (secure)
- UID: User ID (root=0)
- GID: Default group ID
- GECOS: Full name/description
- home: Home directory
- shell: Login shell

### /etc/shadow

Stores encrypted passwords (readable only by root):

```
username:encrypted_password:last_change:min:max:warn:grace:disable
ubuntu:$6$salt$hash...:18974:0:99999:7:::
```

### /etc/group

Stores group information:

```
groupname:password:GID:members
ubuntu:x:1000:user1,user2
```

## User Management Commands

### Creating Users

```bash
# Create user with home directory
sudo useradd -m -s /bin/bash newuser

# Set password
sudo passwd newuser

# Create user with specific options
sudo useradd -m -s /bin/bash -c "Full Name" -d /home/custom newuser

# Add user to group
sudo usermod -aG groupname username
```

### Deleting Users

```bash
# Remove user (keep home directory)
sudo userdel username

# Remove user and home directory
sudo userdel -r username
```

### Modifying Users

```bash
# Change shell
sudo usermod -s /bin/bash username

# Add to group
sudo usermod -aG sudo username

# Lock account
sudo usermod -L username

# Unlock account
sudo usermod -U username
```

## Group Management

### Creating Groups

```bash
# Create a group
sudo groupadd groupname

# Add user to group
sudo usermod -aG groupname username

# Remove user from group
sudo deluser username groupname
```

### User Groups

```bash
# See user's groups
groups username

# See current user's groups
groups

# Change primary group (advanced)
sudo usermod -g groupname username
```

## Sudo Access

### What is sudo?

`sudo` (superuser do) allows authorized users to run commands as root while maintaining audit trails.

```bash
# Run command as root
sudo command

# Verify sudo access
sudo -l

# Run command as specific user
sudo -u otheruser command

# Edit sudoers (carefully!)
sudo visudo
```

### Sudoers File

Located at `/etc/sudoers` and edited with `visudo`:

```bash
# Allow user to run all commands without password
ubuntu ALL=(ALL) NOPASSWD:ALL

# Allow user to run specific command
ubuntu ALL=(ALL) NOPASSWD:/usr/bin/systemctl restart nginx

# Allow group to run all commands
%admin ALL=(ALL) ALL
```

## File Ownership & Permissions

### Understanding Permissions

```
-rw-r--r-- 1 owner group size date time filename
│││││││││ │ │    │    │
│││││││││ │ │    │    └─ File name
│││││││││ │ │    └────── Group owner
│││││││││ │ └──────────── User owner
│││││││││ └────────────── Hard links count
││││└────────────────────── Others: r(4)
│││└─────────────────────── Group: w(2)
││└──────────────────────── Group: r(4)
│└───────────────────────── User: x(1)
└────────────────────────── User: w(2)
└─────────────────────────── User: r(4)
└──────────────────────────── Type: - (file), d (directory), l (link)
```

### Changing Ownership

```bash
# Change user owner
chown newuser file.txt

# Change group owner
chown :newgroup file.txt

# Change both
chown newuser:newgroup file.txt

# Recursive
chown -R newuser:newgroup directory/

# Only change group
chgrp newgroup file.txt
```

## Permission Levels

```
Owner (User)   = Most specific
Group          = Medium scope
Others         = Least specific
```

When checking permissions, Linux checks in order:
1. Are you the owner? → Use owner permissions
2. Are you in the group? → Use group permissions
3. Otherwise → Use others permissions

## Common Permission Scenarios

```bash
# Readable file (user and group can read)
chmod 640 config.txt           # rw-r-----

# Readable directory (must have execute permission)
chmod 750 directory/           # rwxr-x---

# Private file (only user)
chmod 600 secret.txt           # rw-------

# Public readable file
chmod 644 public.txt           # rw-r--r--

# Executable script
chmod 755 script.sh            # rwxr-xr-x

# Very private file
chmod 700 private/             # rwx------
```

## Practical Examples

### Workspace Setup

```bash
# Create user for intern
sudo useradd -m -s /bin/bash student1

# Set password
sudo passwd student1

# Create workspace
sudo mkdir -p /var/www/student1/public_html
sudo mkdir /var/www/student1/logs

# Set owner
sudo chown -R student1:student1 /var/www/student1

# Set permissions
sudo chmod 750 /var/www/student1
sudo chmod 755 /var/www/student1/public_html
```

### Restricted Access

```bash
# File only user can read
chmod 600 .ssh/private_key

# File group can write to
chmod 770 shared_project/

# Remove all permissions (temporarily disable)
chmod 000 problematic_file

# Restore
chmod 644 problematic_file
```

## Checking Permissions

```bash
# View file permissions
ls -l filename

# View directory permissions
ls -ld directory/

# View recursive
ls -lR directory/

# Find files by permission
find /path -perm 777          # Exact match
find /path -perm -755         # At least rwxr-xr-x
find /path -perm /600         # User read OR write
```

---

**Next:** [Part 4: VPS Deployment & SSH Setup](../Part-4/Part-4-VPS-Deployment-SSH.md)
