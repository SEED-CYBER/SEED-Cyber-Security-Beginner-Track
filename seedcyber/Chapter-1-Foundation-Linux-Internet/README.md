# Chapter 1: Foundation – Linux & Internet Infrastructure

**Duration:** Week 1 (5 days)  
**Prerequisite:** Chapter 0 must be completed  
**Level:** Foundational

## Overview

This week you'll establish the foundation for the entire internship. You'll understand how the internet works end-to-end, master Linux fundamentals, and deploy your first secure VPS with proper user and permission management.

## Learning Objectives

By the end of Week 1, you will:

✓ Understand TCP/IP, DNS, and HTTP protocols  
✓ Confidently use Linux command line for system administration  
✓ Create and manage users and groups with proper permissions  
✓ Deploy a VPS instance with Ubuntu Linux  
✓ Establish secure SSH key-based access  
✓ Assign and manage workspace isolation on shared VPS  
✓ Document your setup for future reference

## Weekly Project Task

**Title:** Deploy and Document Your VPS Workspace  
**Deliverable:** Complete VPS setup with documentation, screenshots, network diagrams, and access logs

Complete all of the following:
1. Deploy a VPS instance on your cloud provider
2. Create non-root user accounts with sudo access
3. Establish SSH key-based authentication
4. Assign workspace directories
5. Implement initial permission controls
6. Create comprehensive documentation with screenshots
7. Set up network diagrams showing your setup
8. Document all setup commands and logs

**Expected Outcome:**
A fully functional, documented VPS accessible securely via SSH with an assigned workspace and basic network understanding.

## Modules in This Chapter

### Module 1: Internet Fundamentals
**File:** [Part-1-Internet-Fundamentals.md](Part-1/Part-1-Internet-Fundamentals.md)

Understand the foundation of all connectivity:
- OSI Model (7 layers)
- TCP/IP Protocol Suite
- DNS resolution and domain names
- HTTP and HTTPS basics
- Network packets and flow

### Module 2: Linux Command Line Mastery
**File:** [Part-2-Linux-Command-Line.md](Part-2/Part-2-Linux-Command-Line.md)

Become proficient with essential Linux commands:
- Navigation and file management
- Text processing and searching
- Process management
- System information commands
- Piping and redirection
- Shell scripting basics

### Module 3: User, Group & Permission Management
**File:** [Part-3-Users-Groups-Permissions.md](Part-3/Part-3-Users-Groups-Permissions.md)

Master access control in Linux:
- Creating users and groups
- Understanding /etc/passwd and /etc/group
- File ownership and permissions (chmod, chown)
- Sudo configuration
- User account lifecycle management

### Module 4: VPS Deployment & SSH Setup
**File:** [Part-4-VPS-Deployment-SSH.md](Part-4/Part-4-VPS-Deployment-SSH.md)

Deploy your production VPS:
- Initial VPS configuration
- System updates and hardening basics
- SSH daemon configuration
- SSH key-based authentication setup
- Workspace assignment and isolation
- Initial firewall rules

## Exercises

**File:** [Week-1-Exercises.md](Week-1-Exercises.md)

Complete weekly exercises:
1. Explain TCP handshake and DNS resolution
2. Deploy VPS and configure users
3. Set up SSH key authentication
4. Create and manage workspaces
5. Document full setup with diagrams
6. Test network connectivity

## Daily Breakdown

### Day 1: Internet Fundamentals & Linux Basics
- Module 1: Internet Protocols
- Module 2: Linux CLI Part A (navigation, files)
- Exercise: Understand TCP/IP locally

### Day 2: Linux Deep Dive
- Module 2: Linux CLI Part B (processes, pipes)
- Module 3: User & Permission Basics
- Exercise: User creation and permission management

### Day 3: User Management
- Module 3: Advanced User Management
- Begin Module 4: VPS Deployment prep
- Exercise: Set up local test users

### Day 4: VPS Deployment
- Module 4: Actual VPS deployment
- SSH key setup
- Workspace assignment
- Exercise: Connect via SSH

### Day 5: Documentation & Testing
- Final workspace setup
- Complete all exercises
- Create documentation deliverable
- Test all systems

## Key Tools Introduced

- **SSH:** Secure remote access
- **chmod/chown:** Permission management
- **useradd/userdel:** User management
- **systemctl:** Service management
- **journalctl:** Log viewing

## Outcomes & Deliverables

### Week 1 Deliverable Checklist

By Friday, submit:

- [ ] VPS deployment documentation
  - Screenshots of running instance
  - Instance specifications (IP, CPU, RAM)
  
- [ ] User account setup documentation
  - List of created users
  - sudo configuration file (/etc/sudoers)
  - Screenshot of user verification

- [ ] SSH key setup
  - Evidence of key-based login working
  - SSH config file
  - Screenshot of passwordless SSH connection

- [ ] Workspace setup
  - Directory structure diagram
  - Permission settings for workspace
  - Output of `ls -la` showing setup
  - Verification that isolation works

- [ ] Network documentation
  - Simple network diagram showing your setup
  - Explanation of network topology
  - Internet protocol explanation (TCP/IP, DNS)

- [ ] Command log & troubleshooting
  - All commands executed  with outputs
  - Any issues encountered and resolutions
  - Performance test results

## Preparation for Week 2

Once Week 1 is complete, you'll have:
- A running Ubuntu VPS under your control
- Secure SSH access
- Proper user accounts with permissions
- Assigned workspace
- Foundation knowledge of internet protocols

This setup is the basis for hardening in Week 2!

## Additional Resources

- [Linux Man Pages Online](https://man7.org/)
- [TCP/IP Illustrated (Concepts)](https://en.wikipedia.org/wiki/TCP/IP_model)
- [SSH Protocol RFC](https://tools.ietf.org/html/rfc4251)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)

---

**Next:** [Module 1: Internet Fundamentals](Part-1/Part-1-Internet-Fundamentals.md)
