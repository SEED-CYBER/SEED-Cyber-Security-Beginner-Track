# Part 1: Log Analysis & Interpretation

## Important Log Files

```
/var/log/syslog          - System messages
/var/log/auth.log        - Authentication attempts
/var/log/kern.log        - Kernel messages
/var/log/nginx/access.log - Web server access
/var/log/nginx/error.log  - Web server errors
/var/log/fail2ban.log     - Intrusion prevention
```

## Viewing Logs

```bash
# Recent entries
sudo tail -n 50 /var/log/auth.log

# Follow log
sudo tail -f /var/log/auth.log

# Search pattern
sudo grep "Failed" /var/log/auth.log

# Count occurrences
sudo grep "ERROR" /var/log/syslog | wc -l

# Date/time filtering
sudo journalctl --since "2 hours ago"
sudo journalctl --since "2025-02-20"
```

## Understanding Log Formats

```
Feb 20 10:30:45 ubuntu sshd[1234]: Failed password for root from 192.168.1.100 port 54321
│       │        │      │       │    │
│       │        │      │       │    └─ Message
│       │        │      │       └─────── Process/function
│       │        │      └──────────────── Process ID
│       │        └─────────────────────── Hostname
│       └────────────────────────────────── Time
└──────────────────────────────────────────── Date
```

## Parsing Logs

```bash
# Extract specific fields
awk '{print $1, $2, $3, $NF}' /var/log/auth.log

# Get failed logins, show IP
grep "Failed" /var/log/auth.log | awk '{print $11}' | sort | uniq -c

# Failed SSH by IP
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```

---

**Next:** [Part 2: Centralized Logging](../Part-2/Part-2-Centralized-Logging.md)
