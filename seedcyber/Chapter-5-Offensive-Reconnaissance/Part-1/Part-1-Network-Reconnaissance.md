# Part 1: Network Reconnaissance

## Nmap Advanced Techniques

```bash
# Service fingerprint
nmap -sV -sC target.com

# Aggressive scan
nmap -A -p- target.com

# Stealth scan
nmap -sS target.com

# UDP scan
nmap -sU target.com

# Output formats
nmap target.com -oN scan.txt
nmap target.com -oX scan.xml
nmap target.com -oA scan       # All formats
```

## Traceroute & Path Analysis

```bash
# See network path
traceroute target.com
mtr target.com

# Windows (WSL)
# Use same commands in WSL or GUI tools
```

---

**Next:** [Part 2: Vulnerability Assessment](../Part-2/Part-2-Vulnerability-Assessment.md)
