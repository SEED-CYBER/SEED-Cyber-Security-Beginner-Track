# Week 5 Comprehensive Exercises

## Overview
Week 5 focuses on **monitoring, detection, and incident response** (Chapter 4). This week transitions from defensive configuration to active threat detection and response, building foundation for offensive security understanding.

---

## Exercise W5-1: ELK Stack Deployment & Log Aggregation

**Time: 120 minutes | Difficulty: Intermediate-Advanced**

### Objectives
- Deploy Elasticsearch, Logstash, Kibana stack
- Aggregate logs from multiple sources
- Create dashboards for security monitoring
- Generate alerts

### Setup

```bash
# Option 1: Docker Compose (easiest)
cat > docker-compose.yml << 'EOF'
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000/udp"
      - "5000:5000/tcp"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

volumes:
  es_data:
EOF

docker-compose up -d
```

**Option 2: Native Installation (Ubuntu)**
```bash
# Add Elastic repo
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list

# Install
sudo apt update
sudo apt install elasticsearch kibana logstash

# Start services
sudo systemctl start elasticsearch kibana logstash
sudo systemctl enable elasticsearch kibana logstash
```

### Configuration

```bash
# Step 1: Create Logstash configuration
cat > /etc/logstash/conf.d/syslog.conf << 'EOF'
input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
  file {
    path => "/var/log/auth.log"
    start_position => "beginning"
    tags => ["auth"]
  }
  file {
    path => "/var/log/apache2/access.log"
    start_position => "beginning"
    tags => ["apache"]
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGLINE}" }
    }
  }
  if "auth" in [tags] {
    grok {
      match => { "message" => "%{SYSLOG5424LINE}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
EOF

# Step 2: Configure Rsyslog to forward logs
sudo nano /etc/rsyslog.d/logstash.conf
```

```bash
# Add line:
*.* @@(o)localhost:5000
```

```bash
# Step 3: Restart services
sudo systemctl restart rsyslog logstash
```

### Dashboard Creation

```bash
# Access Kibana: http://localhost:5601

# Step 1: Create index pattern
# Kibana > Stack Management > Index Patterns
# Pattern: logs-*
# Time field: @timestamp

# Step 2: Explore data
# Kibana > Discover
# Search for failed login attempts:
# filter: "Failed password"

# Step 3: Create dashboard
# Kibana > Dashboard > Create
# Add visualizations:
# - Login attempts over time
# - Failed vs successful logins
# - Top source IPs
# - Service scan attempts
```

### Alert Configuration

```bash
# Elasticsearch Watcher for alerts
curl -X PUT "localhost:9200/_watcher/watch/failed_auth_alert" -H 'Content-Type: application/json' -d'
{
  "trigger": {
    "schedule": {
      "interval": "5m"
    }
  },
  "input": {
    "search": {
      "request": {
        "indices": ["logs-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "match": {
                    "message": "Failed password"
                  }
                }
              ],
              "filter": {
                "range": {
                  "@timestamp": {
                    "gte": "now-5m"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total.value": {
        "gte": 10
      }
    }
  },
  "actions": {
    "send_alert": {
      "email": {
        "to": "admin@example.com",
        "subject": "Security Alert: Multiple Failed Logins",
        "body": "Multiple failed login attempts detected: {{ctx.payload.hits.total.value}}"
      }
    }
  }
}'
```

### Testing

```bash
# Generate test logs
for i in {1..20}; do
  ssh baduser@localhost 2>&1 | grep -i denied
done

# Verify in Kibana
# Kibana > Discover > search for failed logins
# Should see spike in failed authentication events

# Windows/WSL: Same ssh commands work
for i in {1..20}; do ssh -p 22 baduser@localhost; done
```

### Deliverables
- [ ] Docker Compose or installation script
- [ ] Logstash configuration files
- [ ] Dashboard screenshots showing:
  - Login activity timeline
  - Failed vs successful attempts
  - Top source IPs
  - Alert system active
- [ ] Sample alert emails
- [ ] Monitoring setup guide (500+ words)

**Success Criteria**
- ✓ ELK stack deployed and running
- ✓ Multiple log sources aggregated
- ✓ Dashboard displaying key metrics
- ✓ Alerts triggered on suspicious activity
- ✓ Documentation complete

---

## Exercise W5-2: IDS Configuration & Network Monitoring

**Time: 90 minutes | Difficulty: Intermediate**

### Objectives
- Install Snort or Suricata IDS
- Configure detection rules
- Test with network traffic
- Monitor alerts

### Snort Installation

```bash
# Install Snort
sudo apt install snort

# Or from source
wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
tar -xzf snort-2.9.20.tar.gz
cd snort-2.9.20
./configure --prefix=/usr/local/snort
make && sudo make install
```

### Configuration

```bash
# Create rules directory
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /var/log/snort

# Create custom rules file
sudo nano /etc/snort/rules/local.rules
```

```bash
# SQL Injection detection:
alert tcp any any -> any any (msg:"SQL Injection Attempt";
  content:"' OR '1'='1"; http_uri;
  sid:1000001; rev:1; priority:1;)

# Port scan detection:
alert tcp any any -> any any (msg:"Port Scan Detected";
  flags:S; threshold:type limit, track by_src, count 20, seconds 60;
  sid:1000002; rev:1; priority:2;)

# SSH brute force:
alert tcp any 22 -> any any (msg:"SSH Brute Force";
  content:"Failed password";
  threshold:type threshold, track by_src, count 5, seconds 60;
  sid:1000003; rev:1; priority:1;)

# Metasploit detection:
alert tcp any any -> any any (msg:"Metasploit Bind Shell";
  content:"bind("; http_client_body;
  sid:1000004; rev:1;)
```

### Snort Configuration File

```bash
sudo nano /etc/snort/snort.conf
```

```bash
# Key settings:
RULE_PATH ../rules
SO_RULE_PATH ../so_rules
PREPROC_RULE_PATH ../preproc_rules

# Network variables
HOME_NET [192.168.0.0/16,10.0.0.0/8]
EXTERNAL_NET !$HOME_NET

# Output plugin:
output alert_fast: /var/log/snort/alert

# Enable preprocessors
preprocessor normalize_ip4
preprocessor defrag_mempool:memcap=65536000,max_frags=65536
preprocessor stream5_global: track_tcp yes
```

### Network IDS Testing

```bash
# Start Snort in IDS mode
sudo snort -c /etc/snort/snort.conf -i eth0 -l /var/log/snort -m0x1b -d 2>/dev/null

# Or as daemon:
sudo snortd -c /etc/snort/snort.conf -i eth0 -l /var/log/snort -D

# From another machine, generate suspicious traffic:
# Port scan:
nmap -T4 -A target-server

# SQL injection attempt:
curl "http://target-server/app?id=1' OR '1'='1"  

# SSH brute force (5+ failed attempts):
for i in {1..10}; do ssh baduser@target-server; done
```

### Alert Analysis

```bash
# View Snort alerts
cat /var/log/snort/alert | head -50

# Parse alerts with unified format
sudo unified2text /var/log/snort/unified2.log.* | head -50

# Monitor alerts in real-time
tail -f /var/log/snort/alert

# Search for specific threat
grep "SQL Injection" /var/log/snort/alert

# Windows/PowerShell: Use Notepad or:
Get-Content C:\snort\log\alert.txt -Wait
```

### Advanced: Suricata Setup

```bash
# Suricata is lightweight, modern alternative
sudo apt install suricata suricata-update

# Update rules
sudo suricata-update

# Configure
sudo nano /etc/suricata/suricata.yaml
# Set: home-net: [192.168.0.0/16,10.0.0.0/8]

# Run
sudo suricata -c /etc/suricata/suricata.yaml -i eth0

# Monitor
tail -f /var/log/suricata/eve.json
```

### Deliverables
- [ ] Snort/Suricata configuration files
- [ ] Custom detection rules (minimum 5)
- [ ] Screenshots of:
  - Rule compilation successful
  - Alerts triggered on suspicious traffic
  - Alert log with multiple entries
- [ ] Proof of concept showing:
  - Port scan detection
  - Brute force detection
  - Pattern-based detection
- [ ] IDS deployment guide (800+ words)

**Success Criteria**
- ✓ IDS installed and running  
- ✓ Custom rules deployed
- ✓ Suspicious traffic detected
- ✓ Alerts properly logged
- ✓ Real-time monitoring working

---

## Exercise W5-3: Incident Response & Response Procedures

**Time: 120 minutes | Difficulty: Advanced**

### Objectives
- Create incident response plan
- Practice response procedures
- Document findings
- Implement lessons learned

### IR Plan Components

Create comprehensive IR plan document:

```markdown
# Incident Response Plan

## 1. Incident Classification
- **Severity Levels:**
  - Critical: System down, data breach, active attack
  - High: Account compromise, malware detected
  - Medium: Configuration change, policy violation
  - Low: Non-security issues

## 2. Response Procedures

### Critical Incident Protocol

**Immediate Actions (0-5 minutes):**
- Isolate affected system
- Notify security team
- Begin logging/documentation
- Start packet capture

**Investigat Phase (5-30 minutes):**
- Preserve evidence
- Identify attack vector
- Determine scope
- Collect logs

**Containment (30+ minutes):**
- Block attacker IP
- Reset compromised credentials
- Patch vulnerabilities
- Enhance monitoring

**Recovery:**
- Restore from clean backup
- Verify system integrity
- Re-enable monitoring
- Document lessons learned

## 3. Communication Plan

- **Security Team**: Slack #security-incidents
- **Management**: Email + meeting within 1 hour
- **Customers**: Prepared statement after initial triage
- **Law Enforcement**: Contact if required by law

## 4. Tools & Resources

- Forensic tools: VolatilityFX, EnCase
- Network tools: Wireshark, tcpdump
- Log analysis: grep, awk, sed, jq
- Contact: Legal team, ISP, CISO

## 5. Post-Incident

- Review system logs  
- Patch identified vulns
- Update detection rules
- Debrief team
- Update IR plan
```

### Tabletop Exercise

Execute simulated incident:

```bash
# Scenario: Database server compromised via SQL injection

# Step 1: Alert Detection (Kibana shows spike in failed queries)
# Action: DECLARE INCIDENT

# Step 2: Investigation
grep -r "sqlmap" /var/log/apache2/access.log
cat /var/log/mysql/error.log | tail -50

# Step 3: Root Cause Analysis  
# Finding: Application vulnerable to SQL injection

# Step 4: Containment
sudo ufw block 192.168.1.100  # Block attacker IP
mysql> UPDATE users SET password=SHA2(UUID(),256);  # Reset passwords
sudo systemctl restart mysql

# Step 5: Eradication
# Patch vulnerable code
# Redeploy application
# Run DB integrity check

# Step 6: Recovery
# Restore database from clean backup (pre-attack)
mysqldump database > backup_post-incident.sql
mysql < backup_clean_2024_02_01.sql

# Step 7: Lessons Learned
# Document: How penetrated, how detected, what to improve
```

### Forensic Analysis

```bash
# Preserve evidence
mkdir -p /cases/case001
sudo tar czf /cases/case001/logs.tar.gz /var/log/

# Timeline creation
stat /var/www/html/*.php | grep Modify

# File integrity checking
find / -type f -mtime -1 2>/dev/null > recent_files.txt

# Memory dump
sudo dd if=/dev/mem of=/cases/case001/memory.bin

# Network forensics
tcpdump -r suspicious.pcap -w analysis.pcap  "dst port 3306"
```

### Deliverables
- [ ] Formal Incident Response Plan (15-20 pages)
- [ ] Incident classification matrix
- [ ] Response procedures flowchart
- [ ] Communication templates
- [ ] Tabletop exercise report:
  - Timeline of discovery
  - Initial response actions
  - Investigation findings
  - Containment measures
  - Recovery confirmation
- [ ] Lessons learned document
- [ ] Updated detection rules/monitoring

**Success Criteria**
- ✓ IR plan documented
- ✓ Response procedures practiced
- ✓ Evidence properly handled
- ✓ Incident successfully contained
- ✓ Recovery verified

---

## Exercise W5-4: Threat Hunting & Anomaly Detection

**Time: 90 minutes | Difficulty: Advanced**

### Objectives
- Hunt for indicators of compromise
- Identify anomalies in data
- Create detection signatures
- Improve alerting

### Threat Hunting Methodology

```bash
# Step 1: Define threat hypothesis
# "Attackers may establish persistent access via cron jobs"

# Step 2: Collect relevant data
find / -path /proc -prune -o -path /sys -prune -o \
  -name "*.cron" -o -name "*crontab*" -ls 2>/dev/null > cron_check.txt

# Step 3: Analyze data  
cat /etc/crontab
for user in $(cut -d: -f1 /etc/passwd); do
  crontab -u $user -l 2>/dev/null | grep -v "^#"
done

# Step 4: Investigate anomalies
# Look for: Unusual schedules, suspicious commands, encoded payloads

# Step 5: Develop detection
# Create rule for malicious cron jobs
```

### Anomaly Detection Techniques

```bash
# Baseline establishment
# Step 1: Collect normal behavior
cp /var/log/auth.log baseline_auth.log
cp /var/log/syslog baseline_syslog.log

# Step 2: Analyze baseline
wc -l baseline_auth.log  # Normal login volume
tail -100 baseline_auth.log | grep "Failed" | wc -l  # Normal failure rate

# Step 3: Compare with current
# Alert trigger: Failed logins > 2x baseline

# Statistical anomaly detection
# Failed logins: Baseline 5/hour -> Anomaly if >20/hour detected
```

### Hunting Queries (Elasticsearch/Kibana)

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "event.action": "authentication_failure"
          }
        }
      ],
      "filter": {
        "range": {
          "@timestamp": {
            "gte": "now-24h"
          }
        }
      }
    }
  },
  "aggs": {
    "by_source": {
      "terms": {
        "field": "source.ip",
        "size": 100
      }
    }
  }
}
```

### Behavioral Analysis

```bash
# Monitor process execution patterns
# Unusual: service accounts running interactive shells
ps auxww | grep -v "/bin/false" | grep service

# Look for: Interactive sessions from automated accounts
grep "service@" /var/log/auth.log | grep "TTY"

# Network connections from servers  
netstat -tulpn | grep -v LISTEN | grep -v ESTABLISHED | head -50

# Outbound connections to suspicious ports
ss -tulpn | grep tcp | grep ESTABLISHED | awk '{print $4,$5}' | \
  sort | uniq -c | sort -rn
```

### Deliverables
- [ ] Threat hunting plan (3-5 scenarios)
- [ ] Baseline metrics for:
  - Login activity
  - Network connections
  - Process execution
  - File access
- [ ] Anomaly detection rules
- [ ] Hunt results report (1000+ words)
- [ ] Elasticsearch queries/dashboards
- [ ] Indicators of compromise list

---

## Exercise W5-5: Security Monitoring Dashboard

**Time: 60 minutes | Difficulty: Intermediate**

### Objectives
- Create comprehensive monitoring dashboard
- Display key security metrics
- Implement visual alerts
- Enable real-time monitoring

### Grafana Setup

```bash
# Install Grafana
sudo apt install -y grafana-server

# Start
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Access: http://localhost:3000
# Default: admin/admin
```

### Dashboard Creation

```json
{
  "dashboard": {
    "title": "Security Operations Center",
    "description": "Real-time security monitoring dashboard",
    "panels": [
      {
        "title": "Failed Login Attempts (Last 24h)",
        "targets": [
          {
            "expr": "count(auth_failed_logins[24h])"
          }
        ]
      },
      {
        "title": "Network Traffic by Port",
        "targets": [
          {
            "expr": "rate(network_bytes_total[5m])"
          }
        ]
      },
      {
        "title": "System CPU Usage",
        "targets": [
          {
            "expr": "node_cpu_seconds_total"
          }
        ]
      },
      {
        "title": "Fail2Ban Ban Count",
        "targets": [
          {
            "expr": "fail2ban_bans_total"
          }
        ]
      },
      {
        "title": "Critical Alerts (24h)",
        "targets": [
          {
            "expr": "count(alerts{severity='critical'})"
          }
        ]
      }
    ]
  }
}
```

### Setup Prometheus for metrics

```bash
# Install Prometheus
sudo apt install prometheus

# Configure Prometheus
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s
  
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
tar -xzf node_exporter-1.4.0.linux-amd64.tar.gz
sudo mv node_exporter-1.4.0.linux-amd64/node_exporter /usr/local/bin/

# Run Node Exporter
nohup /usr/local/bin/node_exporter &
```

### Deliverables
- [ ] Grafana dashboard configuration
- [ ] Prometheus metrics queries
- [ ] Screenshots of operational dashboard
- [ ] Alert rules for critical metrics
- [ ] Monitoring guide (500+ words)

**Success Criteria**
- ✓ Metrics collected and displayed
- ✓ Dashboard showing real-time data
- ✓ Alerts configured for anomalies
- ✓ Professional appearance and organization

---

## Exercise W5-6: Comprehensive Week 5 Assessment

**Time: 180 minutes | Difficulty: Advanced**

### Scenario: "Intrusion Detection & Response"

Deploy complete security monitoring stack and respond to simulated breach:

### Phase 1: Detection Setup (60 min)
- [ ] Deploy ELK stack
- [ ] Configure IDS/Snort
- [ ] Create monitoring dashboard
- [ ] Enable alerting

### Phase 2: Simulate Attack (30 min)
- [ ] SQL injection attempt
- [ ] Port scanning
- [ ] Brute force attack  
- [ ] Data exfiltration

### Phase 3: Detection & Response (60 min)
- [ ] Identify attack in logs
- [ ] Trigger alerts
- [ ] Investigate incident
- [ ] Contain threat
- [ ] Document findings

### Phase 4: Reporting (30 min)
- [ ] Create incident report
- [ ] Timeline of events
- [ ] Recommendations
- [ ] Lessons learned

### Deliverables
- [ ] Complete monitoring solution deployed
- [ ] Incident response executed
- [ ] Professional incident report (15+ pages)
- [ ] All configuration files
- [ ] Presentation (if required)

---

## Week 5 Capstone: SOC Operations Document

Compile comprehensive document for operating security operations center:

### Sections
1. **Monitoring Setup** (5 pages)
2. **Alert Procedures** (5 pages)
3. **Incident Classification** (3 pages)
4. **Response Procedures** (5 pages)
5. **Escalation Matrix** (2 pages)
6. **Tools & Resources** (3 pages)
7. **Training Requirements** (2 pages)

### Submission
- [ ] SOC Operations Manual (25-30 pages)
- [ ] All dashboards/configurations
- [ ] Incident response templates
- [ ] Alert playbooks

---

## Week 5 Summary

By end of Week 5, you should:
✓ Deploy comprehensive monitoring stack  
✓ Detect security threats in real-time  
✓ Execute incident response procedures  
✓ Hunt for advanced threats  
✓ Analyze security events professionally  
✓ Create actionable alerts

**Estimated Effort: 10-12 hours**  
**Difficulty Progression: ⭐⭐⭐⭐ Advanced**
