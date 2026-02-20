# Part 2: Nginx Web Server Installation & Configuration

## Installation

```bash
sudo apt update
sudo apt install nginx

# Start service
sudo systemctl start nginx

# Enable on boot
sudo systemctl enable nginx

# Verify
sudo systemctl status nginx

# Test connection
curl http://localhost
```

## Basic Configuration

```bash
# Main config file
sudo nano /etc/nginx/nginx.conf

# Site configs
sudo nano /etc/nginx/sites-available/default
```

## Simple Virtual Host

```nginx
server {
    listen 8001;
    server_name your.vps.ip;
    
    root /var/www/username/public_html;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
}
```

Test and reload:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

**Next:** [Part 3: TLS/HTTPS Setup](../Part-3/Part-3-TLS-HTTPS-Certificates.md)
