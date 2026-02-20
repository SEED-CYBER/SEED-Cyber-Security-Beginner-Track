# Module 4: Shared VPS Architecture

## Understanding Your Shared Training Environment

All interns share a single VPS instance to practice security skills in an isolated, controlled manner. Understanding the architecture prevents interference between students and ensures fair, ethical practice.

## What is a Shared VPS?

Instead of each student having their own separate server:

```
Traditional (Expensive):
├── Student 1 VPS (full server)
├── Student 2 VPS (full server)
└── Student 3 VPS (full server)

Shared (Efficient):
└── Single VPS
    ├── Student 1 Workspace (Virtual Host A, Ports 8001-8009)
    ├── Student 2 Workspace (Virtual Host B, Ports 8010-8019)
    └── Student 3 Workspace (Virtual Host C, Ports 8020-8029)
```

## Directory Structure for Virtual Hosts

Each student gets an assigned workspace directory:

```
/var/www/
├── student1/
│   ├── public_html/         # Website files
│   ├── logs/                # Individual access/error logs
│   └── backup/              # Student's backups
├── student2/
│   ├── public_html/
│   ├── logs/
│   └── backup/
└── student3/
    ├── public_html/
    ├── logs/
    └── backup/
```

### Your Assigned Workspace

You will receive:

```
Workspace Location: /var/www/your_username/
Home Directory: /home/your_username/
```

**Important:** You can ONLY modify files in your workspace. Other students' directories are off-limits.

## Port Segmentation

Instead of competing for port 80/443, each student uses dedicated port ranges:

```
Web Service Port Mapping:

Student 1: 8001 (http), 8401 (https)
Student 2: 8002 (http), 8402 (https)
Student 3: 8003 (http), 8403 (https)
```

Benefits:
- No conflicts between students' services
- Each student can run web server on their port
- Firewall rules isolate by port
- Easy to scale to more students

## Isolation Mechanisms

### File Permissions

```
/var/www/student1/ : rwx for student1 user, no access for others
/var/www/student2/ : rwx for student2 user, no access for others

Check permissions:
ls -la /var/www/
```

### User & Group Isolation

Each student gets:
- Personal user account (e.g., `student1`)
- Personal group (e.g., `student1`)
- Home directory accessible only to that user
- Sudo access only within their workspace

```bash
# You can only see/modify YOUR files
cat /var/www/student2/secret.txt  # Permission denied

# You can only run services on YOUR ports
sudo systemctl start nginx-student1
# nginx-student2 is separate
```

### Process Isolation

Services (web servers, databases) run under individual user accounts:

```bash
ps aux | grep student1
# nginx running as student1
# mysql running as student1

ps aux | grep student2
# nginx running as student2
# mysql running as student2
# (These are separate processes, can't interfere)
```

## Firewall Rules for Isolation

The VPS firewall enforces port access:

```
Zone : Student 1
├── Inbound: Ports 8001, 8401 (their http/https)
├── Internal: Can reach shared services (logs, auth)
└── Outbound: Internet access for updates/tools

Zone : Student 2
├── Inbound: Ports 8002, 8402
├── Internal: Can reach shared services
└── Outbound: Internet access
```

## Shared Resources

Some resources ARE shared (monitored for fairness):

```
Shared Resource          | Usage Limit
Bootstrap DNS           | Shared, read-only
Internal Clock          | Synchronized
Log Aggregate Service   | Write quota: 1GB/week
Temp Storage (/tmp)     | 100MB per student
Network Bandwidth       | Fair-share QoS
```

## Suspension Policy

To maintain fair practice environment:

### Prohibited Activities

❌ **Attempting to access other students' workspaces**
❌ **Attacking shared infrastructure (DNS, firewall)**
❌ **Port hijacking (claiming ports outside assignment)**
❌ **DoS attacks on internal network**
❌ **Unauthorized privilege escalation**
❌ **Weaponizing tools against students**

### Consequences

| Violation | Action |
|-----------|--------|
| First warning | Written warning + remedial exercise |
| Second violation | Suspension for 1 week |
| Third violation | Removal from program |

## Your Responsibilities

### DO:

✓ **Practice within your assigned workspace only**
✓ **Use tools ethically against your own services**
✓ **Respect other students' privacy**
✓ **Keep your credentials secure**
✓ **Report issues to supervisor immediately**

### DON'T:

✗ **Scan ports outside your range**
✗ **Read other students' logs**
✗ **Modify shared configuration**
✗ **Run resource-intensive processes 24/7**
✗ **Enable open services accessible to others**

## Accessing Your Workspace

### SSH Connection

```bash
# From local machine
ssh -i ~/.ssh/key.pem student1@shared.vps.address

# Now on the VPS
student1@vps:~$ pwd
/home/student1

# See your web workspace
student1@vps:~$ ls /var/www/student1/
public_html/  logs/  backup/
```

### Workspace Structure You'll Use

```
/home/student1/
├── .ssh/                       # SSH config
├── .bashrc                      # Shell config
├── projects/                    # Your project files
└── notes/                       # Documentation

/var/www/student1/
├── public_html/                 # Website files (Week 3)
├── logs/                        # Nginx access/error logs
└── backup/                      # Your backups
```

## Network Topology

```
                    ┌─────────────────┐
                    │  Shared VPS     │
                    │  (Single Host)  │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────┐
         │                   │               │
    ┌────▼───┐          ┌────▼───┐     ┌───▼────┐
    │Student1│          │Student2│     │Student3│
    │Port8001│          │Port8002│     │Port8003│
    │Dir:/..1│          │Dir:/..2│     │Dir:/..3│
    └────┬───┘          └────┬───┘     └───┬────┘
         │                   │             │
    ┌────▼──────────────────▼────────────▼────┐
    │ Internal Shared Services                 │
    │ - syslog (monitoring)                    │
    │ - auth daemon (access control)           │
    │ - DNS resolver (name resolution)         │
    │ - NTP (time sync)                        │
    └──────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────┐
    │        Internet / External Access │
    └─────────────────────────────────┘
```

## Assignment Checklist

Before Week 1, you should have:

- [ ] Assigned username (e.g., `student1`)
- [ ] SSH access verified (`ssh user@vps` works)
- [ ] Workspace location confirmed (`/var/www/username/`)
- [ ] Port range assigned (e.g., ports 8001, 8401)
- [ ] File permissions view (`ls -la /var/www/username/`)
- [ ] Can see your home directory (`pwd` shows `/home/username/`)

## Getting Help

If you have questions about:
- **Access issues:** Contact supervisor immediately
- **File permission errors:** Check with supervisor
- **Port conflicts:** Report to supervisor right away
- **Your workspace structure:** Refer to this guide

## Next Steps

You're almost ready for Week 1! Next, read [Module 5: Access Control & Permissions](Part-5-Access-Control-Permissions.md) to learn about authentication and access to your workspace.

---

**Next:** [Module 5: Access Control & Permissions](Part-5-Access-Control-Permissions.md)
