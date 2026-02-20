# Part 1: DNS & Domain Setup

## Domain Registration

Purchase domain from registrar (GoDaddy, Namecheap, Route53, etc).

## DNS Configuration

Edit your VPS A record to point to your IP:
```
Type: A
Name: example.com
Value: your.vps.ip.address
TTL: 3600
```

Add www subdomain:
```
Type: CNAME
Name: www
Value: example.com
```

## Test DNS Resolution

```bash
# Verify DNS propagation
nslookup example.com
dig example.com

# Should show your VPS IP
```

---

**Next:** [Part 2: Nginx Configuration](../Part-2/Part-2-Nginx-Configuration.md)
