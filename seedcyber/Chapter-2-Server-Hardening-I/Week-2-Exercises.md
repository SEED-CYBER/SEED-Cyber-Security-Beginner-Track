# Week 2 Exercises

## Exercise 1: SSH Hardening

Harden SSH configuration:
1. Disable root login
2. Enable key-only authentication
3. Change settings in sshd_config
4. Test configuration
5. Document before/after

Deliver: sshd_config changes and test results

## Exercise 2: Firewall Configuration

Set up UFW:
1. Enable UFW
2. Deny all incoming by default
3. Allow SSH, HTTP, HTTPS
4. Verify rules: `ufw status`
5. Test with nmap from another machine

Deliver: UFW status output and nmap results

## Exercise 3: Fail2Ban Deployment

Install and configure:
1. Install fail2ban
2. Configure SSH protection
3. Monitor logs
4. Simulate brute-force attempts
5. Verify bans

Deliver: Configuration and ban evidence

## Exercise 4: Network Scanning

Scan your own VPS:
1. Port scan from external machine
2. Service enumeration with -sV
3. Compare with ss -tlnp output
4. Verify only intended services visible
5. Document exposed services

Deliver: Nmap output and analysis

## Exercise 5: Security Report

Document hardening:
1. Explain each hardening step
2. Justify firewall rules
3. Explain Fail2Ban thresholds
4. List open ports and reasoning
5. Minimal attack surface

Deliver: Professional security report

---

**Submission:** Week-2-SUBMISSION.md with all exercises
