# Part 4: IDS & Network Monitoring

## Snort (IDS - Intrusion Detection System)

```bash
# Install Snort
sudo apt install snort

# Start IDS
sudo systemctl start snort

# Monitor alerts
sudo tail -f /var/log/snort/alert
```

## Tcpdump (Packet Capture)

```bash
# Capture all traffic
sudo tcpdump -i eth0 -w capture.pcap

# Capture specific port
sudo tcpdump -i eth0 port 80 -w http.pcap

# Analyze capture
sudo tcpdump -r capture.pcap | head -50

# Extract HTTP requests
sudo tcpdump -r capture.pcap -A | grep "GET\|POST"
```

## Wireshark (GUI Analysis - Windows/macOS/Linux)

```bash
# Windows/macOS: Install from https://wireshark.org
# Linux/WSL:
sudo apt install wireshark

# Launch GUI
wireshark &

# Analyze pcap files:
# File → Open → Select capture.pcap
```

## NetFlow Monitoring

```bash
# View network connections
netstat -an | grep ESTABLISHED

# ss command (modern)
ss -tpan | grep LISTEN
ss -tpan | grep ESTABLISHED

# Monitor in real-time
watch 'ss -tpan | grep ESTABLISHED | wc -l'
```

---

**Next:** [Week 4 Exercises](../Week-4-Exercises.md)
