# Week 1 Exercises

**Complete by:** Friday, Week 1  
**Time Estimate:** 8-10 hours  
**Deliverable:** Documentation with screenshots and outputs

## Exercise 1: Internet Protocol Understanding

**Task:** Explain how a request to www.example.com works.

```
Document the following:
1. DNS resolution process (5 steps minimum)
2. TCP three-way handshake diagram
3. HTTP request and response headers
4. The role of ports in communication
5. Difference between TCP and UDP
```

**Bonus:** Use wireshark to capture actual packets.

## Exercise 2: VPS Deployment

**Task:** Deploy your VPS instance completely.

```
Deliver:
1. Screenshot of VPS running
2. IP address and specs
3. Initial SSH connection successful
4. Output of: uname -a, df -h, free -h
5. Hostname set to a meaningful name
```

## Exercise 3: User Account Setup

**Task:** Create accounts and test permissions.

```
Create:
1. Two regular user accounts with sudo access
2. One system service account (no password login)
3. One group for shared access

Document:
1. useradd commands used
2. Output of /etc/passwd entries
3. sudoers configuration
4. Group membership verification
```

## Exercise 4: SSH Key Authentication

**Task:** Set up secure SSH access.

```
Complete:
1. Generate RSA-4096 key locally
2. Deploy public key to VPS
3. Disable password authentication
4. Test key-based login
5. Add SSH config file

Deliver:
1. Screenshot of passwordless SSH login
2. SSH config file content
3. Verification that password auth is disabled (sshd_config screensho)
```

## Exercise 5: Workspace Setup

**Task:** Create isolated workspaces.

```
Create:
1. /var/www/username directory structure
2. Proper permissions and ownership
3. Create README documenting workspace
4. Verify isolation (can't access other users)

Deliver:
1. Directory tree showing structure
2. Permission listing (ls -la output)
3. Proof of isolation attempts
```

## Exercise 6: Network Verification

**Task:** Test connectivity and network tools.

```
Run and document:
1. ping to determine reachability
2. traceroute to see network path
3. nslookup for DNS resolution
4. curl to test HTTP/HTTPS
5. netstat/ss to show listening ports
6. telnet or nc to test port connectivity

Deliver:
1. Output of all commands
2. Explanation of each result
3. Analysis of your VPS's network position
```

## Exercise 7: Documentation & Diagram

**Task:** Create professional documentation.

```
Include:
1. Network topology diagram (your local machine → VPS → Internet)
2. Directory structure diagram
3. User and permission matrix
4. Setup workflow (step-by-step)
5. Troubleshooting guide for Week 1 issues
6. Important URLs/IPs/usernames (sanitized)
```

## Submission Deliverable

Create file: `Week-1-SUBMISSION.md`

```markdown
# Week 1 Submission
**Student:** [Your Name]
**Date:** [Submission Date]

## Exercise 1: Internet Protocols
[Your answers]

## Exercise 2: VPS Deployment
[Screenshots and information]

## Exercise 3: User Accounts
[Commands and outputs]

## Exercise 4: SSH Setup
[Keys and configuration]

## Exercise 5: Workspace
[Structure and permissions]

## Exercise 6: Network Utils
[Command outputs and analysis]

## Exercise 7: Documentation
[Diagrams and guides]

## Questions/Issues
[Any problems and resolutions]
```

---

## Completion Checklist

- [ ] VPS running and accessible
- [ ] SSH key auth working
- [ ] Users created with sudo access
- [ ] Workspace directory exists and clean
- [ ] Network tools tested
- [ ] All documentation complete
- [ ] Ready for Week 2

