
# Passive Network Discovery and Evidence Collection

## Tool 1: Passive Network Discovery (Netdiscover)
Before any hacking, you must first find your targets. On a local network, devices constantly broadcast ARP (Address Resolution Protocol) packets.

**Command:**
```bash
sudo netdiscover -p -i eth0
```
- `-p` (Passive mode) forces the tool to "sniff" only. It will not scan the network.

**Practical Task:**
Look for a vendor named "PCs Systemtechnik GmbH" or "Oracle/VirtualBox"—this is usually your Metasploitable VM.

**Why it’s Ethical:**
Active scanning (ARP requests) can be logged by an IDS (Intrusion Detection System). Passive discovery is invisible.

**Importance:**
You cannot attack what you cannot find. This provides the Target IP.

---

## Tool 2: Protocol Analysis (TShark)
If the Metasploitable VM is running any background services (like a web server or clock sync), it will leak data.

**Command:**
```bash
sudo tshark -i eth0 -f "not host [Your_Kali_IP]"
```
- `-f` is a "Capture Filter." This tells TShark to ignore your own Kali Linux traffic so you only see what others are doing.

**Practical Task:**
Wait for a few minutes. Do you see "HTTP" or "TCP" traffic? If you see Port 80, the target has a web server.

**Why it’s Ethical:**
This is "wiretapping" for information gathering. It helps you map the attack surface without interaction.

---

## Tool 3: OS Fingerprinting (Passive)
You can often tell what Operating System a machine is running just by looking at the TTL (Time to Live) value in its packets.

**Command:**
```bash
sudo tcpdump -ni eth0 -c 100
```

**Practical Task:**
Look at the `ttl` value in the output.
- TTL 64: Usually Linux/Unix (Metasploitable)
- TTL 128: Usually Windows

**Why we use it:**
To decide which exploits to prepare. You don't want to use a Windows exploit on a Linux machine.

**Importance:**
This is called Stack Fingerprinting. Every OS handles networking slightly differently.

---

## Tool 4: Automated Evidence Collection (Bettercap)
Bettercap is a powerful "all-in-one" tool that can map a whole network passively.

**Command:**
```bash
sudo bettercap -iface eth0
```
Inside bettercap, type:
```
net.recon on
set net.show.meta true
net.show
```

**Practical Task:**
Use the `net.show` table to identify the "Meta" column. It will show you if the device is a Virtual Machine or a physical router.

**Importance:**
Bettercap organizes the "noise" into a readable table for your final report.

---

## Additional Recommended Tools

- **Wireshark**: GUI-based packet analysis for deep inspection of network traffic.
- **arpwatch**: Monitors ARP traffic and reports changes, useful for detecting new devices.
- **p0f**: Advanced passive OS fingerprinting tool.
- **EtherApe**: Visualizes network traffic in real time.
- **Snort (in passive mode)**: Can be used to log and analyze suspicious traffic without active scanning.

---

## 📂 Intern Deliverable: The "Invisible" Report
To pass this lab, interns must submit a `.md` file to GitHub with the following data captured only via the tools above:

### Target Evidence Table
| Metric         | Found Value         | Tool Used   | Confidence % |
|---------------|--------------------|------------|--------------|
| Target IP     | 192.168.x.x        | netdiscover | 100%         |
| MAC Address   | 08:00:27:xx        | bettercap   | 100%         |
| Predicted OS  | Linux (TTL 64)     | tcpdump     | 90%          |
| Leaked Services | Port 80 (HTTP)   | tshark      | 80%          |

---

### Final Question for Interns
> In your own words, why is it safer for an Ethical Hacker to use `netdiscover -p` instead of a standard `nmap -sn` ping sweep during the initial phase of an audit?

---

### Notes
- Only use passive tools for this exercise.
- Do not interact with the target except as described above.