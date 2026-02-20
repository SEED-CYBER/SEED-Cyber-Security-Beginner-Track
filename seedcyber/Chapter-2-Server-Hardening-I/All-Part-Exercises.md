# Chapter 2 Part Exercises & Modules

## Part 1 Exercises: SSH Hardening

**Exercise 1: Audit Current SSH**
```bash
sudo sshd -T | grep -i auth
sudo systemctl status ssh
grep PermitRoot /etc/ssh/sshd_config
```

**Exercise 2: Harden SSH**
1. Disable root login
2. Enforce key-only auth
3. Test configuration
4. Monitor failed attempts

**Exercise 3: Monitor SSH Attacks**
```bash
sudo tail -f /var/log/auth.log | grep Failed
sudo grep "Failed" /var/log/auth.log | wc -l
```

---

## Part 2 Exercises: Firewall Configuration

**Exercise 1: UFW Setup**
```bash
sudo ufw status
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw enable
```

**Exercise 2: Test Firewall Rules**
```bash
nmap -p 22,80,443 your.vps.ip
ss -tlnp
```

**Exercise 3: Document Rules**
Create table of all firewall rules and justifications.

---

## Part 3 Exercises: Fail2Ban

**Exercise 1: Install & Configure**
```bash
sudo apt install fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status sshd
```

**Exercise 2: Simulate Attack**
```bash
# From another machine, wrong passwords
ssh -i wrongkey user@vps
# Multiple times...
```

**Exercise 3: Verify Bans**
```bash
sudo fail2ban-client status sshd
sudo iptables -L -n | grep fail2ban
```

---

## Part 4 Exercises: Network Scanning

**Exercise 1: Scan Your VPS**
```bash
nmap -sV your.vps.ip
nmap -p- your.vps.ip
```

**Exercise 2: Compare Results**
- External nmap scan
- Internal ss -tlnp
- Match ports with services

**Exercise 3: Security Analysis**
Document which ports/services are necessary.

---

