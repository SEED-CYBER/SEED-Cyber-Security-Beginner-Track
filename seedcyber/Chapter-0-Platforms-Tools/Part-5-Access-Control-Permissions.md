# Module 5: Access Control & Permissions

## Authentication & Authorization

Before you can work on the VPS, you need proper authentication (proving who you are) and authorization (what you're allowed to do).

## SSH Key-Based Authentication

SSH keys are more secure than passwords. You'll use a key pair:

### How SSH Keys Work

```
Private Key (Your Computer)        Public Key (VPS Server)
─────────────────────────          ──────────────────────
secret_key.pem (keep safe!)  ←→    Goes in ~/.ssh/authorized_keys

When you SSH:
1. Your client sends connection request
2. Server sends random challenge
3. Your private key signs the challenge (proves you own the key)
4. Server verifies with public key
5. Access granted!
```

### Your SSH Keys

By the end of Chapter 0, you will have:

```bash
~/.ssh/id_rsa           # Your private key (SECRET! Like password)
~/.ssh/id_rsa.pub       # Your public key (share with VPS)
```

### Key Generation (If Not Done Yet)

```bash
# Generate key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -C "your_email@example.com"

# When prompted:
# Enter passphrase: [create strong passphrase]
# Confirm passphrase: [repeat it]

# Verify keys created
ls ~/.ssh/
# id_rsa
# id_rsa.pub
```

### Deploying Your Public Key

Your supervisor will add your public key to the VPS:

```bash
# Your public key content (share this with supervisor)
cat ~/.ssh/id_rsa.pub
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDx... your_email@example.com
```

The supervisor places it in `/home/your_username/.ssh/authorized_keys` on the VPS.

## SSH Connection to Your Workspace

Once your key is deployed, connect via SSH:

```bash
# Basic SSH connection
ssh -i ~/.ssh/id_rsa your_username@shared.vps.ip

# With SSH config file (easier)
ssh vps-training

# Where ~/.ssh/config contains:
# Host vps-training
#     HostName 123.45.67.89
#     User your_username
#     IdentityFile ~/.ssh/id_rsa
#     Port 22
```

## Workspace Permissions

Once logged in, your workspace is protected:

### Your Directories

```bash
# Home directory - only you can access
ls -la /home/your_username/
# drwx------ your_username students 4096 /home/your_username

# Web workspace - only you can access
ls -la /var/www/your_username/
# drwx------ your_username students 4096 /var/www/your_username
```

### Permission Breakdown

```
drwx------ = 700
d = directory
rwx = user (you) can read, write, execute
--- = group cannot access
--- = others cannot access
```

This means:
✓ You can access your files
✓ Other students cannot
✓ Group members (administrators) can if they use sudo

## Sudo Access

You have limited sudo access for administrative tasks in your workspace:

```bash
# Check what you can run with sudo
sudo -l

# Example output:
# User your_username may run the following commands:
#   (root) NOPASSWD: /usr/bin/systemctl start nginx-your_username
#   (root) NOPASSWD: /usr/bin/systemctl stop nginx-your_username
#   (root) NOPASSWD: /usr/bin/systemctl restart nginx-your_username
```

### Using Sudo

```bash
# Start your service
sudo systemctl start nginx-your_username

# It runs as root, but only for your service
sudo systemctl status nginx-your_username

# You CANNOT do this
sudo systemctl stop nginx-student2  # Permission denied!
sudo rm -rf /                       # Permission denied!
```

## File Ownership

Check who owns files in your workspace:

```bash
ls -l /var/www/your_username/

-rw-r--r-- 1 your_username students 12345 Feb 20 10:30 index.html
                    ↑         ↑
                  owner     group
```

### Ownership Rules

- **Owner:** You (user `your_username`)
- **Group:** `students` (all interns in the group)
- **Permissions:** You can read/write, group reads, others cannot

## Changing Permissions on Your Files

You can modify permissions of your own files:

```bash
# Make file readable only by you
chmod 600 secret.txt
# rw-------

# Make file readable by your group
chmod 640 config.ini
# rw-r-----

# Make directory traversable
chmod 755 public_html/
# rwxr-xr-x

# Recursive - change directory and contents
chmod 744 scripts/
chmod -R 755 public_html/
```

## Password-Based Access (Backup)

For backup access if SSH key fails:

```bash
# Your account has a password set by supervisor
# Never share this password
# Change it immediately:

passwd
# Enter old password: [current password]
# Enter new password: [strong new password]
# Retype new password: [confirm]

# Example: avoid these passwords:
# ❌ 12345678
# ❌ password
# ❌ your_username
# ❌ student123

# ✓ Good: Tr0pic@lStorm#2026Lab!
```

## Credential Security Checklist

### Keep Safe

- [ ] Private SSH key (`~/.ssh/id_rsa`)
- [ ] SSH key passphrase
- [ ] VPS password

### Don't Share

- [ ] Private key files
- [ ] Credentials in chat/email
- [ ] Credentials in code commits
- [ ] Credentials in screenshots

### Good Practices

✓ **Passphrase protected SSH key:** protects if key file leaks
✓ **Strong password:** 16+ characters, mix of types
✓ **SSH config file:** easier than typing command each time
✓ **SSH key rotation:** generate new keys periodically
✓ **Regular backups:** of your key pair (password protected)

## Accessing Services on Your Port

### Web Services

Once you deploy a web server in Week 3:

```bash
# On your local machine, access your service
curl http://shared.vps.ip:8001/     # If you're student with port 8001

# From inside VPS
curl http://localhost:8001/
```

### Port Verification

```bash
# Check what's listening on your port
sudo ss -tlnp | grep 8001

# See all open ports
sudo ss -tlnp
```

## Troubleshooting Access

### Can't SSH?

```bash
# Check key exists
ls ~/.ssh/id_rsa

# Check permissions (should be 600)
ls -la ~/.ssh/id_rsa
# -rw------- (correct)

# Try with verbose output
ssh -v your_username@vps.ip
# Look for any error messages
```

### Permission Denied?

```bash
# Check if file is in your workspace
ls /var/www/your_username/file.txt

# Check your user
whoami
# Should be: your_username

# Check file ownership
ls -la /var/www/your_username/file.txt
# your_username should be owner
```

### Port Access Issues?

```bash
# Check if service is running
sudo systemctl status nginx-your_username

# Check if port is open
ss -tlnp | grep 8001

# Test locally first
curl http://localhost:8001/
```

## Access Control Summary

By the end of Chapter 0:

- [ ] SSH key pair generated
- [ ] Public key deployed by supervisor
- [ ] SSH connection tested and working
- [ ] Workspace assignment received
- [ ] File permissions verified
- [ ] Sudo access confirmed
- [ ] Password changed to secure value

## Next Steps

Once all access is confirmed:
1. Test SSH connection multiple times
2. Verify workspace permissions
3. Complete Chapter 0 Exercises
4. Get supervisor approval for Week 1

---

**Next:** [Chapter 0 Exercises](Chapter-0-Exercises.md)
