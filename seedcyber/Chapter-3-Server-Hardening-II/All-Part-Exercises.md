# Chapter 3 Part Exercises

## Part 1: DNS Setup Exercises

**Exercise 1: Domain Registration Check**
- [ ] Have domain registered
- [ ] Know registrar details
- [ ] Can access DNS settings

**Exercise 2: DNS Record Configuration**
```bash
# Create A record pointing to VPS IP
nslookup yourdomain.com
dig yourdomain.com +short

# Wait for propagation (24-48 hours typical)
# Check multiple times with:
host yourdomain.com
```

**Exercise 3: Subdomain Creation**
```bash
# Add CNAME for www
nslookup www.yourdomain.com

# Add other subdomains (mail, api, etc)
# Verify each resolves correctly
```

---

## Part 2: Nginx Exercises

**Exercise 1: Installation**
```bash
sudo apt install nginx
sudo systemctl status nginx
curl http://localhost
curl http://your.vps.ip:8001
```

**Exercise 2: Virtual Host Configuration**
- Create conf file in /etc/nginx/sites-available/
- Test configuration: sudo nginx -t
- Enable site
- Reload Nginx
- Access from browser

**Exercise 3: HTML Content**
```bash
echo "<h1>Welcome!</h1>" > /var/www/username/public_html/index.html
curl http://yoursite.com
```

---

## Part 3: HTTPS Exercises

**Exercise 1: Certificate Generation**
```bash
sudo certbot certonly --standalone -d yourdomain.com
# Check certificate location
ls /etc/letsencrypt/live/yourdomain.com/
```

**Exercise 2: Nginx HTTPS Configuration**
- Edit Nginx config with SSL paths
- Test configuration
- Reload Nginx
- Access via https://

**Exercise 3: Certificate Verification**
```bash
curl -v https://yourdomain.com  # See certificate details
# Check certificate expiration
sudo certbot certificates
```

---

## Part 4: Service Auditing Exercises

**Exercise 1: Running Services Audit**
```bash
systemctl list-units --type=service --state=running
# Document each service purpose
# Mark as necessary/unnecessary
```

**Exercise 2: Disable Unnecessary Services**
For each unnecessary service:
```bash
sudo systemctl disable servicename
sudo systemctl stop servicename
```

**Exercise 3: Log Analysis**
```bash
sudo tail -f /var/log/nginx/access.log
sudo grep "404" /var/log/nginx/access.log
# Analyze access patterns
```

---
