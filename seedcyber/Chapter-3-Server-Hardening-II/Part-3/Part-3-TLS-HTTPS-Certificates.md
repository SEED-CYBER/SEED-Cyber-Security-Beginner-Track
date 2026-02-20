# Part 3: TLS/HTTPS Certificates with Let's Encrypt

## Certbot Installation

```bash
sudo apt install certbot python3-certbot-nginx
```

## Get Certificate

```bash
# Interactive
sudo certbot certonly --standalone -d example.com -d www.example.com

# Enter email when prompted
# Agree to terms
# Certificate stored in /etc/letsencrypt/live/example.com/
```

## Configure Nginx for HTTPS

```nginx
server {
    listen 8401 ssl http2;
    server_name example.com www.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    
    root /var/www/username/public_html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 8001;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

## Auto-Renewal

```bash
# Test renewal
sudo certbot renew --dry-run

# Renewal runs automatically via cron
# Check renewal timer
sudo systemctl list-timers | grep certbot
```

---

**Next:** [Part 4: Service Auditing & Logging](../Part-4/Part-4-Service-Auditing-Logging.md)
