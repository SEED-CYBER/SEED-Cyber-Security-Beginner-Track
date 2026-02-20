# Part 4: Network Scanning & Service Enumeration

## Nmap (Network Mapper)

Discover hosts and services on network.

### Basic Scanning

```bash
# Scan single host
nmap 192.168.1.100

# Scan range
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 target.com

# All ports
nmap -p- target.com

# UDP ports
nmap -sU target.com

# Service version detection
nmap -sV target.com
```

### Scanning Your Own VPS

```bash
# From external machine
nmap -p- your.vps.ip

# Shows:
# Which ports respond
# Which services are running
# Versions (with -sV)

# Compare with: ss -tlnp on VPS
```

### Output Options

```bash
# Save to file
nmap target.com -oN scan.txt

# XML output
nmap target.com -oX scan.xml

# Aggressive scan (careful!)
nmap -A -p- -sV target.com
```

## Interpreting Results

```
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
443/tcp   open     https
3306/tcp  closed   mysql
```

- **open**: Service is running, accepting connections
- **closed**: Port responds but no service
- **filtered**: Firewall blocking response

---

**Next:** [Week 2 Exercises](../Week-2-Exercises.md)
