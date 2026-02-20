# Part 1: SSH Server Hardening - Enhanced

## The SSH Hardening Problem

### Why Default SSH Is Dangerous

```
Default SSH Configuration:
├─ PermitRootLogin yes      (root can log in directly!)
├─ PasswordAuthentication yes (passwords can be brute-forced)
├─ MaxAuthTries undefined   (unlimited guesses!)
├─ X11Forwarding yes        (extra attack surface)
├─ Empty passwords OK       (null password backdoors!)
└─ Published on port 22     (easy to scan)

Consequences:
├─ Attackers scan the internet for open port 22
├─ Automated tools try default credentials
├─ Brute force attacks consume resources
├─ One successful compromise = root access
├─ Logs fill with noise from attacks
└─ Server under constant bombardment!

Reality Check (Actual Server):
A typical internet-exposed server gets:
├─ 50-100 login attempts PER MINUTE
├─ From hundreds of IP addresses
├─ Testing common passwords (admin, root, etc)
├─ Many from automated botnet scanners
└─ This starts within HOURS of going online!
```

### The Hardening Philosophy

```
Defense in Depth for SSH:

LAYER 1: Eliminate High-Value Targets
  ├─ Root cannot log in directly
  ├─ No password authentication (no guessable passwords)
  ├─ Only key holders can authenticate
  └─ Admin must use sudo for elevated tasks

LAYER 2: Limit Attack Surface
  ├─ Disable unnecessary features (X11 forwarding)
  ├─ Close unused ports
  ├─ Limit authentication attempts
  ├─ Reduce failed attempts per connection
  └─ Timeouts and delays

LAYER 3: Detect Attacks
  ├─ Monitor login attempts
  ├─ Alert on suspicious patterns
  ├─ Rate-limit aggressive scanners
  ├─ Block after repeated failures
  └─ Maintain audit trail

RESULT:
  ├─ Brute force becomes impractical
  ├─ Automated scanners get nowhere
  ├─ Manual attackers spend less time
  ├─ Damage from compromised account limited
  └─ Recovery possible and traceable
```

---

## SSH Configuration: First Principles

### Understanding /etc/ssh/sshd_config

**What this file IS:**
```
/etc/ssh/sshd_config = SSH Daemon Configuration
├─ Read when sshd starts
├─ Controls SSH server behavior
├─ Affects ALL SSH connections to this server
├─ Requires daemon restart to apply changes
└─ Syntax: Key Value (one per line)

Why this matters:
├─ Config syntax errors break SSH entirely
├─ Mistake = can't log in = admin access lost!
├─ Must be careful when editing
├─ Test syntax BEFORE restarting!
└─ Keep backup access open!
```

### Critical SSH Configuration Options Explained

#### 1. PermitRootLogin: Who Can Login as Root?

```
PermitRootLogin no
├─ Root CANNOT login via SSH directly
├─ Only regular users can log in
├─ Admin uses "sudo su -" to become root
├─ Why this matters:
│  ├─ Root compromise = TOTAL system compromise
│  ├─ If attacker gets root: All users owned
│  ├─ If attacker gets regular user: Limited damage
│  ├─ Principle: Protect highest-value account first
│  └─ Requires two steps to elevate (harder to automate)

Common alternatives:
├─ PermitRootLogin without-password (still dangerous!)
├─ PermitRootLogin forced-commands-only (very restrictive)
└─ PermitRootLogin yes (DEFAULT - INSECURE!)

Real-world attack:
1. Attacker gets password list (common on dark web)
2. Tries to SSH as root with each password
3. If PermitRootLogin yes: Once successful = SYSTEM OWNED
4. If PermitRootLogin no: Becomes root user account NOT accessible
   └─ Attacker needs ANOTHER exploit to escalate
   └─ Much harder!

Recommendation: PermitRootLogin no
```

#### 2. PubkeyAuthentication: Allow SSH Keys?

```
PubkeyAuthentication yes
├─ SSH keys work
├─ Symmetric part of authentication
├─ Must be enabled for key-based auth
├─ WHY:
│  ├─ Keys are mathematically proven (not guessable)
│  ├─ Enables automation (servers talk to servers)
│  ├─ More secure than passwords
│  └─ No keyboard input needed (prevents keyloggers!)

Default: yes (usually fine)

When to disable:
├─ Never. Leave this enabled.
└─ If you disable this, you can't use SSH keys!
```

#### 3. PasswordAuthentication: Allow Passwords?

```
PasswordAuthentication no
├─ Passwords as SSH auth method are DISABLED
├─ Users MUST use SSH keys
├─ WHY disable?
│  ├─ Passwords can be guessed (brute force)
│  ├─ Eliminate massive attack surface
│  ├─ If PasswordAuthentication yes: 50+ attacks/min
│  ├─ If PasswordAuthentication no: 0 password attacks!
│  └─ Principle: Remove attack vectors entirely

What happens if enabled (PasswordAuthentication yes):
├─ Attacker connects
├─ SSH says "password please"
├─ Attacker tries: admin
├─ Denied
├─ Attacker tries: password
├─ Denied
├─ Attacker tries: 123456
├─ Denied
├─ ... repeat 1000x per second
├─ Eventually: admin123 (BINGO!)
├─ Root access with default credentials!
└─ System compromised in seconds

What happens if disabled (PasswordAuthentication no):
├─ Attacker connects
├─ SSH says "public key please"
├─ Attacker has no key (doesn't have private keyfile)
├─ Connection refused
├─ User with SSH key: Works perfectly
└─ Automated attack impossible!

Recommendation:
├─ ONLY disable after:
│  ├─ Creating admin user
│  ├─ Installing SSH key for admin
│  └─ Testing that key authentication works!
├─ Never disable first without key ready
└─ Otherwise: LOCKED OUT OF OWN SERVER!
```

#### 4. PermitEmptyPasswords: Allow Null Passwords?

```
PermitEmptyPasswords no
├─ Users cannot have empty passwords
├─ Cannot authenticate with empty password
├─ Pretty much always: no
├─ WHY:
│  ├─ Null password = anyone can log in
│  ├─ Means account is broken
│  ├─ Should never happen
│  └─ Default: no (safe)

When would this matter?
├─ If user's password deleted by accident
├─ Account created with no password initially
├─ Account configuration mistake
└─ And PasswordAuthentication is yes

Setting: PermitEmptyPasswords no (always)
```

#### 5. MaxAuthTries: Limit Failed Attempts

```
MaxAuthTries 3
├─ User gets 3 failed authentication attempts
├─ After 3 failures: Connection dropped
├─ Must reconnect to try again
├─ WHY:
│  ├─ Slows down brute force
│  ├─ If password auth disabled: moot point
│  ├─ If password auth enabled: critical limit
│  └─ Discourages automated attacks

Attack timeline WITH MaxAuthTries 3:
├─ Attacker connects
├─ Try 1: admin
├─ Try 2: password
├─ Try 3: 123456
├─ Connection dropped
├─ Attacker reconnects
├─ Repeat up to 3 more
├─ Blocks progress

Attack timeline with MaxAuthTries 100 (default):
├─ Attacker gets 100 guesses per connection
├─ Can try 100 common passwords in one session
├─ Much higher chance of success
└─ Must reconnect after 100 failures

Recommendation: MaxAuthTries 3-5
└─ Slows attacks significantly
```

#### 6. MaxSessions: Limit Concurrent Connections

```
MaxSessions 10
├─ Only 10 concurrent SSH connections per IP
├─ Prevents resource exhaustion attacks
├─ WHY:
│  ├─ Attacker could open 1000 connections
│  ├─ Each takes server resources
│  ├─ Legitimate users can't connect
│  ├─ Denial of Service
│  └─ Limit prevents this

Attack: Connection Flood (without limit)
├─ Attacker opens SSH connection
├─ Doesn't authenticate
├─ Opens another
├─ And another
├─ And another (1000 times)
├─ Server memory exhausted
├─ Legitimate users: "Server not responding"
├─ Result: DoS attack

Setting MaxSessions 10:
├─ Can only have 10 unauthenticated connections
├─ After 10: New connections refused
├─ Legitimate users still get through
└─ Prevents connection exhaustion
```

#### 7. X11Forwarding: Disable Graphics

```
X11Forwarding no
├─ Graphical interface NOT forwarded through SSH
├─ Only terminal available
├─ WHY disable?
│  ├─ X11 protocol is complex and old
│  ├─ Potential security vulnerabilities
│  ├─ Servers don't need graphics anyway
│  ├─ Adds attack surface for no benefit
│  └─ Principle: Minimize what's exposed

What X11Forwarding does (if enabled):
├─ Allows graphical applications through SSH
├─ SSH tunnel becomes X11 server
├─ Remote applications appear locally
├─ Adds complexity and potential vulns
└─ Not needed for server administration

Recommendation: X11Forwarding no
└─ Services don't need graphics
```

#### 8. Port: Which Port to Listen On?

```
Port 22
├─ Standard SSH port (publicly known)
├─ Internet scanners look here first
├─ WHY 22?
│  ├─ IANA official SSH port
│  └─ Standard across all systems

Port 2222 (Alternative):
├─ Non-standard port
├─ Automated scanners often skip
├─ NOT security (just obscurity)
├─ WHY it doesn't help:
│  ├─ Attacker can scan all 65535 ports
│  ├─ Finding your SSH is trivial
│  ├─ Moving port = false security
│  ├─ Distracts from real hardening
│  └─ Better to fix fundamentals

Reality Check:
├─ Attacker scans your IP
├─ Port 22: SSH found in 1 second
├─ Port 2222: SSH found in 30 seconds
├─ Difference: almost meaningless
├─ Falls to other hardening measures
└─ Conclusion: Port doesn't matter much

Recommendation: Keep standard Port 22
└─ Don't distract with obscurity
└─ Focus on real security: keys only, no passwords!
```

---

## Hardening Workflow: Step-by-Step

### Phase 1: Preparation (Before Any Changes!)

```bash
# CRITICAL: Before editing sshd_config:
# 1. Keep SSH connection open in current terminal
# 2. Open NEW terminal for testing changes
# 3. Keep emergency backup access method

# Current SSH connection (keep open!)
ssh admin@server

# NEW terminal (for testing)
ssh -o StrictHostKeyChecking=no admin@server

# Why?
# If config is wrong:
# ├─ New connection fails
# ├─ But original connection still works
# ├─ You can fix the error
# └─ Not locked out
```

### Phase 2: Edit Configuration

```bash
# Edit on server
sudo nano /etc/ssh/sshd_config

# Make changes:
# ├─ PermitRootLogin no
# ├─ PubkeyAuthentication yes
# ├─ PasswordAuthentication no
# ├─ PermitEmptyPasswords no
# ├─ MaxAuthTries 3
# ├─ MaxSessions 10
# ├─ X11Forwarding no
# └─ Port 22 (leave standard)

# Save and exit (Ctrl+O, Enter, Ctrl+X)
```

### Phase 3: Verify Configuration Syntax

```bash
# CRITICAL: Test syntax BEFORE restarting!
sudo sshd -t

# Output: (nothing = success!)
# If error: Shows what's wrong

# If error exists:
# ├─ Don't restart!
# ├─ Fix the error
# ├─ Test again
# └─ Once "sudo sshd -t" produces no output: OK to restart

# Check current settings
sudo sshd -T | grep -E "permit|password|auth|max"
```

### Phase 4: Test the Connection Before Restarting

```bash
# In new terminal (don't close current one!):
ssh -v admin@server

# Should connect as normal
# If this works with old config: Good!
```

### Phase 5: Restart SSH Service

```bash
# Back in original terminal on server:
sudo systemctl restart sshd

# Output: (nothing = success)
```

### Phase 6: Verify Hardening Applied

```bash
# In new terminal connection:
ssh admin@server

# Try to connect as root (should fail):
ssh root@server
# Output: Permission denied
# Reason: PermitRootLogin no

# Try password (should fail):
ssh -o PubkeyAuthentication=no admin@server
# Output: Permission denied
# Reason: PasswordAuthentication no

# Normal SSH key connection (should work!):
ssh -i ~/.ssh/id_rsa admin@server
# Should work perfectly!
```

---

## Monitoring Hardened SSH

### What to Monitor

```
After hardening: Watch for:

1. Rate of failed attempts (should decrease from 50+/min to near 0)
2. Source IPs of attacks (can block with firewall)
3. Attack patterns (same IP ≠ random scanning)
4. Brute force attempts (shouldn't happen!)
5. Unusual connection times
```

### Monitoring Commands

```bash
# Real-time SSH log monitoring
sudo tail -f /var/log/auth.log | grep sshd

# Failed login attempts (last 24 hours)
sudo grep "Failed password\|Invalid user" /var/log/auth.log | tail -20

# Count failed attempts by IP
sudo grep "Failed password" /var/log/auth.log | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -rn | head -10

# Successful logins
sudo grep "Accepted publickey" /var/log/auth.log | tail -10

# Extract connection details
sudo journalctl -u ssh -n 100 | grep "session opened for"
```

---

## Common SSH Hardening Mistakes

### Mistake 1: Disabling Password Auth Before Keys Work

```
WRONG:
1. Edit sshd_config
2. PasswordAuthentication no
3. Restart sshd
4. Oops! Can't connect (no keys deployed)
5. LOCKED OUT!

CORRECT:
1. Ensure admin user created
2. Deploy SSH key with ssh-copy-id
3. Test new connection: ssh -i key admin@server
4. THEN change PasswordAuthentication no
5. Restart sshd
6. All good!
```

### Mistake 2: Not Testing Configuration Before Restarting

```
WRONG:
1. Edit sshd_config
2. Restart sshd immediately
3. If error: SSH breaks, locked out!

CORRECT:
1. Edit sshd_config
2. Run: sudo sshd -t
3. If output: Error (don't restart!)
4. If no output: OK to restart!
5. Test connection works before closing
```

### Mistake 3: Changing Port for Security

```
WRONG:
"We'll use port 2222 instead of 22 for security!"
(This is security theater, not real security!)

CORRECT:
"We disabled passwords, only keys work.
 Let's use standard port 22.
 Real security comes from crypto,
 not obscurity."
```

### Mistake 4: Not Monitoring After Hardening

```
WRONG:
1. Harden SSH
2. Assume it's secure
3. No monitoring
4. Attacker finds exploit months later
5. Breach goes unnoticed for weeks

CORRECT:
1. Harden SSH
2. Set up monitoring
3. Alert on suspicious activity
4. Review logs regularly
5. Investigate anomalies
6. Continuous hardening
```

---

## Advanced: SSH Certificates (Future Hardening)

```
For large deployments, SSH certificates provide:
├─ Centralized key management
├─ Time-limited access (expires automatically)
├─ Role-based permissions (different certs for different users)
├─ Audit trail (certificate tracking)
└─ More secure than raw public keys

But for initial hardening: Skip this
└─ Key-based auth sufficient for most
└─ Adds complexity not needed yet
```

---

This completes SSH server hardening at depth. Proper SSH security forms the foundation for all server security.
