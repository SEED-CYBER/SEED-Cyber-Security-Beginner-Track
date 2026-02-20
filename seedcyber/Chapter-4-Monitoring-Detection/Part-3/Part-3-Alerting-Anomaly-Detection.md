# Part 3: Alerting & Anomaly Detection

## Threshold-Based Alerting

```bash
# Monitor high CPU
top -bn1 | grep "Cpu(s)" | awk '{print $2}'

# Check memory
free | awk 'NR==2{print $3/$2 * 100}' | if ($(cat) > 80) alert

# Disk usage
df | awk 'NR>1{print $5}' | while read percent; do
    if [ ${percent%\%} -gt 80 ]; then
        echo "ALERT: Disk at $percent"
    fi
done
```

## Log-Based Alerts

```bash
# Failed logins threshold
FAILED=$(grep "Failed" /var/log/auth.log | wc -l)
if [ $FAILED -gt 100 ]; then
    # Send alert
fi

# Error detection
grep "ERROR" /var/log/app.log | tail -10
```

---

**Next:** [Part 4: Network Monitoring](../Part-4/Part-4-IDS-Network-Monitoring.md)
