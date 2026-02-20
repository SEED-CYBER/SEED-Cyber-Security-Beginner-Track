# Chapter 4 Part Exercises

## Part 1: Log Analysis Exercises

**Exercise 1: Examine Critical Logs**
```bash
# Auth log
sudo tail -100 /var/log/auth.log
sudo grep "Failed" /var/log/auth.log | wc -l
sudo grep "Accepted" /var/log/auth.log | head -5

# System log
sudo tail -100 /var/log/syslog
sudo grep "ERROR" /var/log/syslog

# App logs
sudo tail -100 /var/log/nginx/access.log
sudo tail -100 /var/log/nginx/error.log
```

**Exercise 2: Parse Log Data**
```bash
# Extract IPs from auth.log
sudo grep "Accepted" /var/log/auth.log | awk '{print $NF}' | sort | uniq

# Count login attempts per user
sudo grep "Failed" /var/log/auth.log | awk '{print $9}' | sort | uniq -c | sort -rn

# Time-based analysis
sudo grep "Failed" /var/log/auth.log | cut -c1-11 | sort | uniq -c
```

**Exercise 3: Real-time Log Monitoring**
```bash
# Windows: Use WSL or Cygwin to follow logs
tail -f /var/log/auth.log

# Generate activity - watch logs update
# (In another terminal, attempt SSH)
ssh user@localhost
```

---

## Part 2: Centralized Logging Exercises

**Exercise 1: Understand Log Destinations**
- [ ] Identify all log sources on VPS
- [ ] Map to log files (/var/log/*)
- [ ] Document rotation policy

**Exercise 2: Setup Rsyslog (Simple)**
```bash
# Configure rsyslog to forward logs:
sudo nano /etc/rsyslog.conf

# Add:
*.* @@logserver.example.com:514
```

**Exercise 3: Log Retention**
```bash
# Check logrotate config
cat /etc/logrotate.d/nginx
cat /etc/logrotate.d/rsyslog

# Run rotation manually
sudo logrotate -f /etc/logrotate.conf

# Verify
ls -la /var/log/
```

---

## Part 3: Alerting Exercises

**Exercise 1: Monitor Failed SSH**
```bash
# Create simple alert script
cat > monitor.sh << 'EOF'
#!/bin/bash
FAILURES=$(grep "Failed" /var/log/auth.log | wc -l)
if [ $FAILURES -gt 50 ]; then
    echo "ALERT: $FAILURES failed logins!" | mail -s "SSH Alert" you@example.com
fi
EOF

chmod +x monitor.sh
```

**Exercise 2: Cron Job for Alerts**
```bash
# Add to crontab
crontab -e

# Add:
*/10 * * * * /home/user/monitor.sh
```

---

## Part 4: IDS Exercises

**Exercise 1: Monitor Network**
```bash
# Capture traffic
sudo tcpdump -i eth0 -w capture.pcap

# Analyze with tshark
sudo tshark -r capture.pcap | head -50

# Find suspicious traffic
sudo tshark -r capture.pcap | grep -i "syn"
```

---
