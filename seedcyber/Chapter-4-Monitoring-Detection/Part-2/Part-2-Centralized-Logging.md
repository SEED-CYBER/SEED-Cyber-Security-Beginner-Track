# Part 2: Centralized Logging (Rsyslog)

## Rsyslog Configuration

Default: `/etc/rsyslog.conf`

```bash
# Send all logs to central location
*.* @@logserver.example.com:514

# Or store locally
*.* /var/log/centralized.log
```

## Logrotate Configuration

```bash
# /etc/logrotate.d/custom
/var/log/myapp.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 root root
    postrotate
        systemctl reload rsyslog > /dev/null 2>&1 || true
    endscript
}
```

Rotate logs:
```bash
sudo logrotate -f /etc/logrotate.d/custom
```

---

**Next:** [Part 3: Alerting & Anomaly Detection](../Part-3/Part-3-Alerting-Anomaly-Detection.md)
