# Part 4: SSH & VPS - Enhanced Conceptual Foundation

## The Problem SSH Solves

### Remote Login: The Original Security Nightmare

**Before SSH (the 1980s-1990s):**

```
Telnet Protocol (ANCIENT):
  ├─ User connects to remote server
  ├─ Server asks for username
  ├─ User types username (sent PLAIN TEXT)
  ├─ Server asks for password
  ├─ User types password (sent PLAIN TEXT)
  ├─ Connection now open
  └─ Everything sent UNENCRYPTED!

PROBLEM: Anyone on the network could see:
  ├─ Your username
  ├─ Your password
  ├─ Everything you do on the remote server
  ├─ Every command you run
  ├─ Every file you access
  └─ COMPLETELY VISIBLE!

Scenario:
  You ──[password: myp@ssw0rd]──> Attacker ──> Server
  
Attacker sees your password! ⚠️
```

### The SSH Solution: Encryption + Authentication

```
SSH (Secure Shell):
  ├─ Encrypts ALL traffic (attacker sees gibberish)
  ├─ Authenticates server (know you're talking to right place)
  ├─ Authenticates user (only right users can log in)
  └─ Prevents tampering (if attacker modifies data, cryptography detects it)

Modern SSH with keys:
  ├─ No passwords needed
  ├─ Uses public key cryptography
  ├─ Even if password stolen, they can't log in (no password!)
  └─ Perfect for automated systems
```

---

## Public Key Cryptography: The Foundation

### The Key Pair Concept (Simplified)

**Imagine two special boxes:**

```
PUBLIC KEY (the mailbox):
  ├─ Anyone can use it
  ├─ Anyone can "lock" something in it
  ├─ You POST this publicly
  └─ Think: Public mailbox on street

PRIVATE KEY (your key):
  ├─ Only you have it
  ├─ Can UNLOCK things encrypted with public key
  ├─ Keep this SECRET (never share!)
  ├─ Think: Key to your private mailbox
  └─ If stolen, attacker can read all your mail

How it works:
  1. Alice creates key pair (public + private)
  2. Alice posts public key everywhere
  3. Bob wants to send Alice secret message
  4. Bob uses Alice's PUBLIC key to encrypt message (locks mailbox)
  5. Only Alice's PRIVATE key can decrypt it (opens mailbox)
  6. Only Alice can read the message!
```

### Why Public Key Solves Password Problem

```
OLD WAY (Telnet with password):
  User ──[password]──> Server
  Problem: If attacker intercepts, they have password!

NEW WAY (SSH with public keys):
  1. User generates key pair locally (public + private)
  2. User sends PUBLIC key to server (safe to send!)
  3. Server stores public key in ~/.ssh/authorized_keys
  4. User keeps PRIVATE key locally (secret!)
  5. When connecting:
     ├─ Server sends challenge (random data)
     ├─ User signs challenge with PRIVATE key
     ├─ Server verifies signature with PUBLIC key
     └─ No password ever sent!
  
  KEY INSIGHT:
  ├─ If attacker intercepts: They only see PUBLIC key (useless!)
  ├─ If attacker steals connection: They don't have PRIVATE key (can't authenticate!)
  ├─ If attacker cracks password: Doesn't matter, no password used!
  └─ Much more secure!
```

### Math Behind It (Conceptually)

```
RSA Algorithm (common public key system):

SETUP (computer does this):
  ├─ Pick two large prime numbers (p and q)
  ├─ Multiply them: N = p × q (very hard to factor back!)
  ├─ Choose public exponent (usually 65537)
  ├─ Calculate private exponent using p, q
  └─ Public key: (N, 65537) - can print, post, share
     Private key: (N, private exponent) - KEEP SECRET!

ENCRYPTION:
  ciphertext = message^public_exponent mod N
  (Math operation, easy to compute, hard to reverse!)

DECRYPTION:
  message = ciphertext^private_exponent mod N
  (Only possible if you have private exponent!)

SECURITY basis:
  └─ Factoring large N (product of primes) is EXTREMELY hard
     └─ RSA-2048: ~2^2048 possible values (roughly)
     └─ Brute force: Would take universe's lifespan to try them all!
```

---

## SSH Handshake: How Connection Establishes

### Step-by-Step SSH Connection

```
CLIENT (your computer)                      SERVER (VPS)

Step 1: Initial Contact
├─ Client connects to server port 22
└─> Server: "Hello, I'm SSH server version 2.0"

Step 2: Key Exchange
├─ Client & Server negotiate encryption
├─ Both agree on cipher (AES-256, ChaCha20, etc.)
├─ Both agree on hash algorithm (SHA-256, etc.)
├─ Client & Server generate session keys
└─ All future traffic encrypted with session keys

Step 3: Server Authentication
├─ Server presents its public key to client
├─ Client asks: "Is this the right server?"
│  └─ Checks known_hosts file: ~/.ssh/known_hosts
│  └─ First time: "Are you sure?" (must confirm manually)
│  └─ Later: "I remember you!" (checks stored fingerprint)
└─ Mutual agreement: This IS the right server

Step 4: User Authentication (PKA)
├─ Client: "I'm user@hostname, prove I belong"
├─ Client sends PUBLIC key to server
├─ Server: "Show me you have the private key"
├─ Server generates random challenge
├─ Client signs challenge using PRIVATE key
├─ Server verifies signature using PUBLIC key
├─ Result: Client PROVEN! (has private key)
└─ No password needed!

Step 5: Authenticated Session
├─ Client & Server now have encrypted, authenticated connection
├─ All communication encrypted
├─ Both trust each other
└─ User can run commands securely!

Total time: Usually < 1 second!
```

### Man-in-the-Middle Attack (Why Server Verification Matters)

```
ATTACK SCENARIO (without host verification):

Client (you)          Attacker              Real Server
    │                    │                      │
    └──[SSH connect]────>│                      │
                         │──[SSH connect]──────>│
    <──[Server key]──────┤
    (attacker's key!)    │<──[Server key]───────┤
                         │   (real server key)  │

Attacker intercepts both directions!
├─ Client thinks attacker IS the server
├─ Attacker can read all your commands
├─ Attacker can modify server responses
├─ YOUR ACCOUNT HAS BEEN HACKED!

SOLUTION: Host verification
├─ ~/.ssh/known_hosts contains server fingerprints
├─ Client checks: "Do I recognize this server's key?"
├─ If fingerprint doesn't match: ERROR! Connection rejected.
├─ Attacker can't easily forge legitimate server key
└─ First-time connection: Manual verification (human choice)
```

---

## VPS Architecture & Security

### What IS a VPS?

```
Physical Data Center:
  └─ Host Server (powerful physical machine)
      ├─ Virtual Machine 1 (thinks it's own computer)
      │   ├─ Operating System (Linux)
      │   ├─ User accounts
      │   ├─ Services
      │   └─ Network interface (virtual)
      │
      ├─ Virtual Machine 2
      │   ├─ Operating System (Linux)
      │   ├─ Different users
      │   ├─ Different services
      │   └─ Network interface (virtual)
      │
      └─ Virtual Machine 3 (your VPS)
          ├─ Complete operating system
          ├─ Root access
          ├─ Your applications
          ├─ Your users
          └─ Full responsibility!

Key insight:
  ├─ YOU own the Linux instance
  ├─ YOU control users and permissions
  ├─ YOU manage security
  ├─ Host provider: Manages hypervisor, hardware, isolation
  └─ YOU ARE RESPONSIBLE for application security!
```

### VPS Security Model: Defense in Depth

```
LAYER 1: Hardware/Hypervisor (Provider responsibility)
  ├─ Physical data center security
  ├─ Hypervisor isolation (VM-to-VM)
  ├─ DDoS protection (sometimes)
  └─ Backup infrastructure

LAYER 2: Network & Firewall (Hybrid responsibility)
  ├─ Port 22 open (SSH access to you)
  ├─ All other ports closed by default
  ├─ UFW firewall (you control this!)
  ├─ Rate limiting (prevent brute force)
  └─ Intrusion detection (sometimes)

LAYER 3: OS & SSH (YOUR responsibility)
  ├─ Strong SSH security
  │   ├─ Disable root login
  │   ├─ Disable password auth
  │   ├─ Only key-based auth
  │   └─ SSH on non-standard port (optional obscurity)
  │
  ├─ Regular patching
  │   ├─ OS updates (security patches)
  │   ├─ Application updates
  │   └─ Library updates
  │
  └─ Monitoring & logging
      ├─ Watch for break-in attempts
      ├─ Monitor resource usage
      ├─ Check failed login attempts
      └─ Investigate suspicious activity

LAYER 4: Application Security (YOUR responsibility)
  ├─ Secure coding practices
  ├─ No hardcoded secrets
  ├─ Web application firewall
  ├─ Input validation
  └─ Regular backups

If Layer 1 compromised: All VPS on server affected
If Layer 2 compromised: Your VPS can be accessed
But Layers 3 & 4 prevent further damage!
```

---

## SSH Security Deep Dive

### SSH Configuration Security

```
/etc/ssh/sshd_config (SSH daemon configuration)

CRITICAL SETTINGS:

PermitRootLogin no
  ├─ Root CANNOT log in directly via SSH
  ├─ Why? Root compromise = total system compromise
  ├─ Admin users use sudo instead
  └─ Severely limits damage if password stolen

PubkeyAuthentication yes
PasswordAuthentication no
  ├─ Only allow SSH keys
  ├─ Passwords disabled entirely
  ├─ Brute force attacks impossible
  └─ Must have private key to log in

PermitEmptyPasswords no
  ├─ Users must have passwords set
  ├─ Prevents null-password backdoors
  └─ Standard config (usually default)

MaxAuthTries 3
  ├─ Only 3 failed auth attempts
  ├─ After 3 fails: Connection dropped
  ├─ Slows down brute force
  └─ Might need adjustment if SSH key issues

Port 22
  ├─ Standard SSH port
  ├─ Visible to network scanning
  ├─ Moving to non-standard port adds obscurity
  │   └─ PORT 2222: ssh user@server -p 2222
  └─ Note: Obscurity ≠ security (defense in depth!)

X11Forwarding no
  ├─ Disable X11 graphical interface forwarding
  ├─ Reduces attack surface
  └─ Most servers don't need GUI anyway

AllowUsers user1 user2
  ├─ Explicitly list who can SSH in
  ├─ Reject all others
  ├─ Better than just PermitRootLogin no
  └─ Principle: Default deny, explicit allow
```

### SSH Key Security

```
PRIVATE KEY PROTECTION:

Location: ~/.ssh/id_rsa (or ~/.ssh/id_ed25519)
Permissions: MUST be 600 (-rw-------)
  ├─ Only owner can read
  ├─ No group/other access
  ├─ Linux will refuse connection if wrong permissions!
  └─ This is ENFORCED (you can't bypass it)

Passphrase protection:
  ├─ When generating key: ssh-keygen prompts for passphrase
  ├─ Private key stored encrypted
  ├─ Even if file stolen, still protected
  ├─ ssh-agent caches passphrase (only enter once per session)
  └─ Trade-off: Security vs convenience

SSH Agent (caches keys):
  ├─ ssh-agent loads key into memory
  ├─ Prompts for passphrase once
  ├─ Reuses key for all SSH connections
  ├─ Automatic in modern systems
  └─ Compromise of agent = loss of key access (but not key itself)


PUBLIC KEY MANAGEMENT:

Location on server: ~/.ssh/authorized_keys
Anyone with this file can:
  ├─ See who can access you
  ├─ Can't use them (public key can't authenticate)
  ├─ But CAN potentially add new keys!
  └─ File permissions MUST be 600 (-rw-------)

Directory permissions:
  ├─ ~/.ssh MUST be 700 (drwx------)
  ├─ Only owner can read directory contents
  ├─ If writable by others: SSH keys could be modified!
  └─ Linux checks this and refuses SSH if wrong

Key rotation:
  ├─ Generate new key periodically
  ├─ Add new public key to authorized_keys
  ├─ Remove old public key
  ├─ Disconnect with old key (won't work anymore)
  └─ Limits damage if old key compromised
```

### Common SSH Attacks & Defenses

```
ATTACK 1: Brute Force Password Attack

What happens:
  ├─ Attacker connects repeatedly
  ├─ Tries thousands of password guesses
  ├─ Eventually gets lucky
  └─ Gains access

Defense: Disable passwords entirely!
  ├─ PasswordAuthentication no
  ├─ Brute force impossible
  ├─ Must have private key (not guessable)
  └─ Most secure approach

Additional defense:
  ├─ fail2ban (automatic IP blocking after N failures)
  ├─ Rate limiting (only X connections per minute)
  └─ Change default port (slight obscurity)


ATTACK 2: Private Key Theft

What happens:
  ├─ Attacker somehow gets your private key
  ├─ Can now log in as you
  └─ No password needed

Defense: Key file permissions!
  ├─ Private key 600 (-rw-------)
  ├─ Can't be read by non-owner
  ├─ Attacker can't copy if they compromise non-root account
  └─ Root compromise = everything lost (but that's worst case)

Additional defense:
  ├─ Passphrase-protect private key
  ├─ Even if file stolen, still encrypted
  ├─ ssh-agent loads key once per session
  └─ Longer passphrase = more secure


ATTACK 3: Man-in-the-Middle (MITM)

What happens:
  ├─ Attacker intercepts connection
  ├─ Impersonates server
  ├─ You think you're talking to server, but talking to attacker
  ├─ All your commands visible to attacker
  └─ Total compromise

Defense: Server host key verification!
  ├─ SSH verifies server's identity
  ├─ First time: Manual verification (check fingerprint!)
  ├─ Later: ~/.ssh/known_hosts prevents MITM
  ├─ Fingerprint mismatch = ALERT! Connection refused
  └─ Attacker would need to predict server's private key (impossible!)

Additional defense:
  ├─ Check known_hosts entry before connecting new server
  ├─ Verify fingerprint out-of-band (phone call, etc.)
  └─ Use DNS/DNSSEC for hostname verification


ATTACK 4: Server Compromise

What happens:
  ├─ Server is hacked
  ├─ Attacker modifies sshd_config
  ├─ Attacker leaves backdoor account
  ├─ Attacker extracts all users' public keys from authorized_keys
  └─ Attacker can impersonate server to any user

Defense: Regular auditing!
  ├─ Review ~/.ssh/authorized_keys regularly
  ├─ Remove public keys that shouldn't be there
  ├─ Rotate keys periodically
  ├─ Monitor sshd_config changes
  └─ Check file modification times (stat command)

Additional defense:
  ├─ Hardware security key (physical device)
  ├─ Requires attacker to also steal physical key
  ├─ Impossible to compromise remotely
  └─ Getting more common for high-security environments
```

---

## Initial VPS Deployment: Security-First Approach

### What You DO First (Security Foundation)

```
STEP 1: Connect as root
sudo ssh root@vps.ip
├─ Initial connection (password OK for first time)
├─ Must verify server fingerprint
└─ Check it matches provider's public value

STEP 2: Change root password
passwd
├─ Create strong root password
├─ You'll disable password auth later
└─ Emergency-only backup

STEP 3: Update system
apt update && apt upgrade -y
├─ Install all security patches
├─ Important: Older distros may be vulnerable
└─ Critical step!

STEP 4: Create admin user
useradd -m -s /bin/bash -G sudo admin
passwd admin
├─ Create regular user that can use sudo
├─ Set strong password
├─ Add to sudoers group

STEP 5: Key-Based Authentication (security)
# On server:
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# On client:
ssh-copy-id -i ~/.ssh/id_rsa admin@vps.ip

# Test the key works (new terminal)
ssh -i ~/.ssh/id_rsa admin@vps.ip

STEP 6: Disable Password Auth
sudo nano /etc/ssh/sshd_config
# Set these:
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
PermitEmptyPasswords no

sudo systemctl restart sshd

# TEST before closing! (in new terminal)
ssh -i ~/.ssh/id_rsa admin@vps.ip

STEP 7: Firewall
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw default deny incoming

STEP 8: System configuration
hostnamectl set-hostname my-server
systemctl enable fail2ban  # If available

RESULT: Hardened VPS
├─ Root cannot SSH in
├─ Password auth disabled
├─ Key-based auth only
├─ Firewall enabled
├─ Backups configured
└─ Ready for Application deployment
```

---

## SSH Aliases & Efficiency

### ~/.ssh/config for comfort and security

```
Host alias-name
    HostName actual.server.address.com
    User admin
    IdentityFile ~/.ssh/id_rsa_server1
    Port 22
    ServerAliveInterval 60
    ServerAliveCountMax 3
    # Reconnect: Don't drop after inactivity
    
Host production_*
    # Rules for all servers matching production_*
    User readonly_user
    StrictHostKeyChecking accept-new
    # Only accept NEW host keys (added to known_hosts)

Host secure-server
    HostName 203.0.113.45
    User security_admin
    IdentityFile ~/.ssh/keys/secure_key.pem
    # Specific key for this server
    Port 2222
    # Non-standard port

# Now you can:
# ssh alias-name          (instead of full command)
# ssh production_api1     (multiple servers)
```

---

This completes the conceptual foundation of SSH and VPS security. VPS management requires understanding both the technology and security principles.
