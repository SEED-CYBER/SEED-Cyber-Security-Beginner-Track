  GNU nano 7.2                                                                               report.txt

In my learning journey of the following, I used the tryhackme website to learn, text and applythe knowledge acquired.

1. Passive recon
Here we rely on publicly available knowledge and we have no direct contact  with the target.
As there is passive one can guess  there is also an active reconnaissance

2. Active recon
Here,  the attacker has a contact with the target .
Social engineering can be used to do the reconnaissance

PASSIVE RECON TOOLS

A tool that can be used to perform passive recon is the WHOIS lookup.

1. WHOIS :
It is a request response protocol which listens to the TCP port 43 for incoming requests.
this is used to optain the following
.Registrar
..Contact info of the registrant
...Creation ,updatesand expiring date
....Name server

ACTIVE RECON TOOLS
Here , tools such as PING ,TRACEROUTE and NC can be used to gather info about the network, system and services.
1.WED BROUSER (DEVELOPER TOOLS)
here, we make use of the developers tool to optain informations about the current web browser
to enter the developer tool, use Ctrl+Shift+I or the F12 key on the keyboard.
Also,
Foxyproxy and Wappalyser are adds_on for firefox and chrome used to quickly change the proxy server in use and to provide insights on technologies used on the current website.

2.PING:
This is used to check if the remote server is online , if the can be a communication between the attacker and the target.
use "man ping" to have info about the ping command.

NMAP LIVE HOST DISCOVERY
It is an industry standard tool used for
+ Mapping networks
+Identifying live hosts and discovering open ports
the following are the approaches used by nmap to obtain informations
°°ARP scan
°°ICMP scan
°°TCP/UDP ping scan

Subnetwork
This is the logical part of the connection instead of the physical part

given the following ip addresses ,

10.2.0.0/16 this tells us that the subnet is /16 and it can have a subnet mask of 255.255.0.0
10.1.100.0/24 tells us the subnet is /24 and se can have a subnet mask of 255.255.255.0
  
note that 

