# Part 4: Service Auditing & Logging

## List Running Services

```bash
# All services
systemctl list-units --type=service

# Enabled services
systemctl list-unit-files --type=service | grep enabled

# Check specific service
systemctl status nginx

# View service logs
journalctl -u nginx -f
```

## Disable Unnecessary Services

```bash
# Find installed packages
apt list --installed | wc -l

# Disable service
sudo systemctl disable service-name
sudo systemctl stop service-name

# Verify disabled
sudo systemctl status service-name
```

## Nginx Logging

```bash
# Configure in nginx.conf
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;

# View logs
sudo tail -f /var/log/nginx/access.log
sudo grep "404" /var/log/nginx/access.log

# Log rotation
ls /etc/logrotate.d/ | grep nginx
```

## Auditd for System Auditing

```bash
# Install
sudo apt install auditd

# Start
sudo systemctl start auditd

# Audit specific file
sudo auditctl -w /var/www/ -p wa -k webroot

# View audit logs
sudo ausearch -k webroot
```

---

**Next:** [Week 3 Exercises](../Week-3-Exercises.md)
