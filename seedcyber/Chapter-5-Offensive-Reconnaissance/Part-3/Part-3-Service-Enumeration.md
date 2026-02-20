# Part 3: Service Enumeration

## Banner Grabbing

```bash
# HTTP
curl -I https://target.com

# SSH
ssh -v target.com

# SMTP
telnet target.com 25

# FTP
ftp target.com
```

## DNS Enumeration

```bash
# Zone transfer attempt
dig @ns1.target.com target.com axfr

# Subdomain enumeration
for sub in www mail smtp ftp admin blog; do
    dig ${sub}.target.com +short
done
```

---

**Next:** [Part 4: Exploitation Basics](../Part-4/Part-4-Exploitation-Basics.md)
