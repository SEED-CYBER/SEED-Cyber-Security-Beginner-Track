# Chapter 0: Platforms, Tools & Environment Setup

**Duration:** 1-2 days (before Week 1 begins)

## Overview

This chapter ensures you have all necessary tools, accounts, and knowledge to succeed throughout the 6-week internship. You'll set up your local development environment, understand the tools you'll use, and configure your access to the shared VPS.

## Prerequisites Check

Before starting Chapter 1, complete the following:

- [ ] Linux terminal familiarity (or willing to learn)
- [ ] Cloud provider account (AWS, Linode, DigitalOcean, or similar)
- [ ] SSH client installed locally
- [ ] Ability to spend 10-15 hours per week on coursework
- [ ] Daily access to internet and computer

## Learning Objectives

By the end of Chapter 0, you will:

✓ Have a cloud VPS ready to deploy  
✓ Understand the tools used throughout the internship  
✓ Know how to access and manage the shared server  
✓ Be comfortable with basic Linux operations  
✓ Have all documentation and permissions in place

## Modules in This Chapter

### Module 0: Introduction & Cloud Providers
**File:** [Part-0-Introduction-and-Cloud-Providers.md](Part-0-Introduction-and-Cloud-Providers.md)

Understand the cloud landscape and choose your VPS provider. Learn about:
- Different cloud providers (AWS, DigitalOcean, Linode, Azure)
- Cost considerations and free tier options
- Deployment regions and server location choice
- Account setup and billing configuration

### Module 1: Development Tools & Environment
**File:** [Part-1-Development-Tools.md](Part-1-Development-Tools.md)

Set up your local development environment:
- SSH client configuration
- Terminal/shell setup (Bash, Zsh, PowerShell on Windows)
- Text editors for configuration files (VS Code, Nano, Vim)
- Package managers on your local machine
- Parallel logging tools (terminal multiplexers like tmux/screen)

### Module 2: Linux Fundamentals Review
**File:** [Part-2-Linux-Fundamentals-Review.md](Part-2-Linux-Fundamentals-Review.md)

Quick review of essential Linux concepts you'll need:
- User types (root, sudo, regular users)
- File paths and directory structure
- File permissions (chmod, chown)
- Basic navigation commands
- Working with packages (apt, yum)

### Module 3: Security Tools Overview
**File:** [Part-3-Security-Tools-Overview.md](Part-3-Security-Tools-Overview.md)

Introduction to the tools you'll use throughout the curriculum:
- **Network tools:** Nmap, Wireshark, netstat
- **System tools:** systemctl, journalctl, auditd
- **Security frameworks:** OpenVAS, Burp Suite overview
- **Monitoring:** Prometheus, ELK stack concepts
- **Exploitation:** Metasploit, exploitation frameworks overview

### Module 4: Shared VPS Architecture
**File:** [Part-4-Shared-VPS-Architecture.md](Part-4-Shared-VPS-Architecture.md)

Understand how the shared training VPS is organized:
- Virtual host directory structure
- Port segmentation strategy
- Isolation between intern workspaces
- Shared resources and coordination
- Suspension policies for rule violations

### Module 5: Access Control & Permissions
**File:** [Part-5-Access-Control-Permissions.md](Part-5-Access-Control-Permissions.md)

Set up proper access to your workspace:
- Receiving VPS credentials
- SSH key generation and deployment
- Workspace directory assignment
- Port allocation
- Backup and recovery procedures

## Exercises

**File:** [Chapter-0-Exercises.md](Chapter-0-Exercises.md)

Complete all exercises in this chapter:
1. Create your cloud account and deploy a test VPS
2. Generate and test SSH keys
3. Connect to the shared training VPS
4. Run diagnostic commands to verify connectivity
5. Document your setup with screenshots
6. Read through all tools documentation

## Quick Reference

Before you begin Week 1, you should be comfortable with:

```bash
ssh -i keyfile user@host          # Connect via SSH
ssh-keygen -t rsa -b 4096         # Generate SSH keys
sudo apt update && sudo apt upgrade # Update system
man command-name                   # Read manual pages
whoami                             # See current user
pwd                                # Print working directory
ls -la                             # List files with permissions
chmod 644 file                     # Change file permissions
```

## Deliverables for Chapter 0

Submit a documentation file containing:

1. **Cloud Provider Choice & Account Setup**
   - Screenshot of your VPS instance details
   - Region chosen and reasoning
   - Instance specifications (CPU, RAM, Disk)

2. **Local Development Environment**
   - Commands showing your terminal setup
   - Confirmation of SSH client working
   - Output of `ssh -V` verification

3. **Shared VPS Access Confirmation**
   - Successful SSH connection output
   - Your assigned workspace location
   - Your assigned port range
   - Output of `whoami` and `pwd` on the shared server

4. **Tools Verification**
   - List of installed tools with version numbers
   - Confirmation that you can run at least 3 tools from Part 3

5. **Questions or Concerns**
   - List any blockers or questions for the supervisor

## Estimated Time Commitment

- Reading & videos: 3-4 hours
- Tool setup & testing: 2-3 hours
- Exercise completion: 2-3 hours
- **Total:** 7-10 hours

## Next Steps

Once you complete all Chapter 0 exercises:
1. Submit your documentation for supervisor review
2. Get approval to proceed to Week 1
3. Begin [Chapter 1: Foundation – Linux & Internet Infrastructure](../Chapter-1-Foundation-Linux-Internet)

---

**Note:** Do not proceed to Week 1 until Chapter 0 is complete. The foundation you build here is critical for success.
