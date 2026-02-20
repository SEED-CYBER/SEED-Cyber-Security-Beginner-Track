# Server Security & Offensive Security Curriculum

**Date:** February 2026  
**Duration:** 6 weeks  
**Level:** Intermediate to Advanced  
**Supervisor:** Mazwewoh John Brindi

## Overview

This 6-week intensive cybersecurity internship equips you with practical knowledge and real-world skills in server security, system hardening, and ethical hacking. You'll progress from foundational Linux and networking concepts through advanced offensive security techniques.

## Learning Outcomes

By the end of this curriculum, you will be able to:

✓ Understand how the internet works end-to-end (TCP/IP, DNS, HTTP)  
✓ Deploy and secure production-ready Linux VPS instances  
✓ Harden systems against real attackers using industry-standard tools  
✓ Monitor systems using professional security monitoring tools  
✓ Perform ethical reconnaissance and exploitation exercises  
✓ Identify and document vulnerabilities in web applications  
✓ Implement defense mechanisms and respond to security incidents

## Curriculum Structure

### Week 1: Foundation – Linux & Internet Infrastructure
Learn the foundational concepts of internet connectivity and Linux server management. You'll deploy your first VPS and establish secure access.

- **Topics:** Internet fundamentals, Linux CLI, user/permission management, VPS deployment, SSH security
- **Outcome:** A documented, secure VPS instance with SSH access

### Week 2: Server Hardening I – Access & Network Controls
Close unauthorized access vectors and implement network-level defenses through firewall configuration and intrusion prevention.

- **Topics:** SSH hardening, firewall configuration, Fail2Ban, Nmap scanning
- **Outcome:** Access-controlled VPS with documented firewall policies

### Week 3: Server Hardening II – System & Service Security
Secure the services running on your server, focusing on web servers and encrypted communications.

- **Topics:** DNS setup, Nginx configuration, TLS/HTTPS, service auditing, logging
- **Outcome:** Hardened web stack with encrypted endpoints

### Week 4: Monitoring & Detection
Establish visibility into system behavior and network events through comprehensive logging and alerting.

- **Topics:** Log analysis, centralized logging, alert configuration, IDS fundamentals
- **Outcome:** Working monitoring system with automated alerts

### Week 5: Offensive Security – Reconnaissance & Exploitation
Understand how attackers probe systems through ethical reconnaissance and controlled exploitation in isolated environments.

- **Topics:** Network scanning, vulnerability assessment, service fingerprinting, packet analysis
- **Outcome:** Professional reconnaissance report with vulnerability documentation

### Week 6: Offensive Security – Web & Advanced Exploits
Test web application security, perform privilege escalation, and participate in red team/blue team exercises.

- **Topics:** Web security testing, proxy tools, privilege escalation, CTF exercises
- **Outcome:** Comprehensive offensive and defensive security documentation

## Key Tools Covered

- **Access & Hardening:** SSH, UFW, iptables, Fail2Ban
- **Web Services:** Nginx, Let's Encrypt, TLS certificates
- **Monitoring:** Syslog, centralized log collection, alerting systems
- **Reconnaissance:** Nmap, OpenVAS, Wireshark
- **Web Testing:** Burp Suite, SQLi/XSS testing tools
- **Linux:** bash, systemctl, file permissions, user management

## Prerequisites

- Basic Linux command-line familiarity
- Understanding of networking concepts (TCP/IP basics)
- Access to a cloud provider or virtualization platform
- Ability to rent/access a VPS instance
- Ethical commitment to only attack authorized systems

## How to Use This Curriculum

1. **Follow sequentially:** Each week builds on previous knowledge
2. **Complete all exercises:** Hands-on work is essential
3. **Document everything:** Take screenshots and notes for your portfolio
4. **Share responsibly:** Work within the shared VPS environment respectfully
5. **Review tools:** Familiarize yourself with each week's tools before starting

## Shared VPS Setup

All interns will share a single VPS instance with the following structure:

- **Virtual Hosts & Port Segmentation:** Each intern/team gets a dedicated virtual host directory
- **Service Ports:** Services are segmented by port ranges (e.g., 8001-8009 for intern workspaces)
- **Isolated Environments:** Practice attack/defense without affecting others
- **Supervisor-Deployed Application:** A vulnerable web app for the final week's CTF exercises

## Assessment & Completion

**Weekly Project Tasks:** Each week includes a hands-on project that must be completed and documented.

**Final Deliverable:** A comprehensive portfolio including:
- Week 1-5: Technical documentation, security reports, and configuration guides
- Week 6: Offensive & defensive security reports from the CTF exercise

## Resources

- [Quick Reference Guide](QUICK-REFERENCE.md)
- [Setup Checklist](SETUP-CHECKLIST.md)
- [FAQ](FAQ.md)
- [Tips for Success](TIPS_FOR_SUCCESS.md)

## Getting Started

Navigate to [Chapter 0 - Platforms & Tools](Chapter-0-Platforms-Tools) to begin setting up your environment, or jump directly to [Chapter 1 - Week 1](Chapter-1-Foundation-Linux-Internet) to start the curriculum.

---

**Happy hacking! Remember: Security is a journey, not a destination.**
