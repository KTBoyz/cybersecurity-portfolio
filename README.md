Advanced Network Reconnaissance and Service Enumeration Using Nmap
Overview

This project demonstrates a structured network reconnaissance workflow using Nmap. The objective was to understand how reconnaissance techniques reveal information about hosts, services, and operating systems within a local network environment.

The assessment was conducted using Kali Linux and Nmap against devices connected to a local home network, including a router and endpoint devices. The project covers host discovery, TCP SYN scanning, service enumeration, service version detection, and operating system fingerprinting while examining how filtering, access controls, and network infrastructure influence scan results.

Objectives
Discover active hosts on a local network.
Identify accessible services.
Determine service versions where possible.
Detect operating systems through fingerprinting.
Analyze filtered and anomalous responses.
Understand the limitations of automated reconnaissance tools.
Lab Environment

Operating System: Kali Linux

Reconnaissance Tool: Nmap 7.99

Target Network: 192.168.1.0/24

Scan Types:

Host Discovery
TCP SYN Scanning
Full Port Enumeration
Service Enumeration
Operating System Fingerprinting
Methodology

Phase 1 – Host Discovery

The first stage focused on identifying active hosts within the target subnet.

Command:

sudo nmap -PR -sn 192.168.1.0/24

Purpose:

Discover active systems
Collect MAC addresses
Create a target inventory

Results:

Active devices were identified within the network, including:

Router (192.168.1.1)
Intel-based device (192.168.1.33)
Mobile device (192.168.1.37)
Additional active host (192.168.1.35)

Observation:

The scan successfully demonstrated that ARP-based discovery is effective for identifying active systems inside a local subnet,
Bypassed standard ICMP Firewall Restrictions, and
Provided live hosts with IP and MAC address.

Phase 2 – TCP SYN Scanning
Objective

After identifying active hosts through ARP-based host discovery, TCP SYN scanning was performed to determine the state of services running on discovered devices. The objective was to identify whether ports were open, closed, or filtered and to understand how different devices responded to reconnaissance attempts.

Target 1: Router Analysis

Command
sudo nmap -sS -p- 192.168.1.1

Results
The router exposed multiple accessible services:

22/tcp     open  ssh
53/tcp     open  domain
80/tcp     open  http
443/tcp    open  https
8000/tcp   open  http-alt
52881/tcp  open  unknown

Analysis

The SYN scan successfully identified several open services running on the router. These services included Secure Shell (SSH), DNS, HTTP, HTTPS, and additional management-related ports. The presence of open ports indicated that the router was actively providing network services and administrative interfaces.

Target 2: Filtered Device Analysis

Command
sudo nmap -sS -sV 192.168.1.33

Results
All 1000 scanned ports are in ignored states.
1000 filtered tcp ports (no-response)

Analysis

The host was successfully discovered during the host discovery phase, confirming that it was active on the network. However, all scanned ports appeared filtered, meaning the probe packets did not receive a usable response. This behavior is commonly associated with host-based firewalls, packet-filtering rules, or endpoint security mechanisms that intentionally suppress reconnaissance traffic.

This demonstrates that a host may be online while simultaneously preventing service enumeration.

Target 3: Closed Port Analysis

Command
sudo nmap -sS -p 80,443,22,53,21 192.168.1.37

Results
21/tcp  closed  ftp
22/tcp  closed  ssh
53/tcp  closed  domain
80/tcp  closed  http
443/tcp closed  https

Analysis

Unlike the filtered host, this device responded to the SYN probes and explicitly reported that the examined services were closed. This indicates that the host was reachable but no services were listening on the tested ports.

Closed ports provide different information from filtered ports. A closed port confirms host availability while indicating that no application is currently accepting connections on that port.

Discussion of TCP SYN Scan Results

The TCP SYN scan revealed three distinct port states across the discovered hosts:

Open Ports: On the router (192.168.1.1), multiple ports were identified as open, including SSH (22/tcp), DNS (53/tcp), HTTP (80/tcp), HTTPS (443/tcp), HTTP-ALT (8000/tcp), and an unknown high-numbered port (52881/tcp). Open ports indicate that a service is actively listening for incoming connections and can potentially be enumerated further.

Closed Ports: On host 192.168.1.37, the scanned ports (FTP, SSH, DNS, HTTP, and HTTPS) were reported as closed. A closed port indicates that the host is reachable and responding to network traffic, but no service is currently listening on that port.

Filtered Ports: On host 192.168.1.33, all scanned ports appeared filtered. Filtered ports indicate that packet-filtering mechanisms such as firewalls, access control lists, or endpoint protection software prevented Nmap from receiving a definitive response. In this situation, the host was active and reachable, but the services could not be enumerated.

These results demonstrate that TCP SYN scanning can reveal significantly different security postures among devices located on the same network. Some devices openly expose services, some respond by indicating services are unavailable, while others actively suppress reconnaissance attempts through packet filtering and access-control mechanisms. This highlights the importance of interpreting scan results within the broader context of device configuration and network security controls.

Phase 3 - Service Version Enumeration & Anomalies

The next phase focused on identifying the application versions behind the discovered services.

Command:

sudo nmap -sS -sV 192.168.1.1
 
Results:

22/tcp open ssh Dropbear sshd 2019.78
53/tcp open domain dnsmasq 2.80

Additional Findings:

23/tcp filtered telnet
139/tcp filtered netbios-ssn
445/tcp filtered microsoft-ds

Analysis:

Service version detection successfully identified the software running behind the SSH and DNS services. The router revealed use of Dropbear SSH and dnsmasq, while other services appeared filtered, suggesting firewall or access-control policies.

A focused analysis was performed against the router's web service.

Command:

sudo nmap -sV -p 80 192.168.1.1

Results:

80/tcp open tcpwrapped
 

Analysis:

The service accepted the initial connection but terminated communication before Nmap could complete banner extraction.

Possible causes include:

Access-control mechanisms
Embedded device security controls
Firewall-assisted service protection
TCP wrapper implementations

Overall Observation:

Nmap successfully identified Dropbear SSH and dnsmasq because these services responded to its version‑detection probes with recognizable banners. In contrast, port 80 returned “tcpwrapped,” indicating that although the TCP handshake succeeded, the service terminated the connection before Nmap could send its probes. This behavior is typical of embedded router interfaces that enforce access‑control policies or restrict HTTP management to specific clients. Filtered ports (23, 139, 445) suggest firewall rules preventing any probe response.

Phase 4 - OS fingerprinting & Probabilistic Scoring and Feedback Loop

OS fingerprinting (-O) yielded advanced troubleshooting insights regarding scanner logic and network edge defences.

Subsection: 
Probabilistic Scoring and Fingerprint Ambiguity

Command:
sudo nmap -O 192.168.1.37

Results:
Too many fingerprints match this host to give specific OS details.

Analysis:

Probabilistic Scoring in Nmap OS Detection
When performing operating system fingerprinting, Nmap analyzes multiple characteristics of a target's TCP/IP stack and network behavior. These characteristics include responses to crafted packets, TCP flags, sequence generation patterns, window sizes, and other protocol-specific traits.

The collected fingerprint is then compared against entries stored in the nmap-os-db database, which contains thousands of operating system signatures. Many consumer devices, routers, embedded systems, and IoT products are built on similar open-source Linux kernels and often exhibit nearly identical TCP/IP stack behavior.

As a result, multiple operating system signatures may closely match the observed fingerprint. Rather than forcing a potentially incorrect identification, Nmap applies probabilistic scoring to evaluate the confidence level of each match.

When several signatures exceed acceptable confidence thresholds and no single signature clearly dominates, Nmap reports multiple possible matches or declines to identify the operating system with certainty. This behavior reduces the likelihood of false positives and highlights an important limitation of passive fingerprint comparison: similarities in kernel implementation and network stack behavior can make precise identification difficult, particularly among embedded Linux-based devices.

Factors affecting identification include:

Mobile operating system behavior
Limited fingerprint confidence

Subsection:
Reconnaissance Feedback Effect

Command:
sudo nmap -O 192.168.1.1

Results:
Linux 3.X / 4.X
ASUS Embedded Device
ASUS RT-AC66U WAP

Analysis:

Fingerprint Distortion by Network Infrastructure
During multiple reconnaissance iterations against the gateway device, several scan responses exhibited characteristics that were inconsistent with direct endpoint fingerprinting. In certain cases, Nmap reported traits that closely resembled the scanning environment itself or produced results that differed across successive scans.

This behavior can occur when intermediary networking components such as NAT devices, firewall engines, embedded router firmware, or packet-normalization mechanisms process reconnaissance traffic before it reaches the target host. Rather than exposing the underlying operating system or hardware characteristics directly, these systems may modify, suppress, filter, or normalize responses according to predefined security policies.

As a result, the fingerprint observed by the scanner may represent a combination of endpoint behavior and network infrastructure behavior rather than the target device alone. This creates a reconnaissance feedback effect in which scan results are influenced by the defensive mechanisms operating between the scanner and the intended target.

Evidence supporting this observation was seen through:

Ambiguous operating system fingerprint matches.
Filtered port responses that prevented service identification.
tcpwrapped service behavior that terminated enumeration attempts.
Variations in fingerprint confidence between different scan targets.

These results demonstrate that reconnaissance output should not be interpreted as a direct representation of the endpoint in isolation. Effective analysis requires consideration of NAT translation, firewall filtering, access-control policies, packet normalization, and other network-layer mechanisms that may influence the responses returned to scanning tools.

Factors affecting identification include:

NAT interaction
Firewall responses
Packet normalization

Key Findings

ARP host discovery successfully identified active systems on the local network.
Full TCP enumeration exposed multiple management and web services.
Service version detection identified Dropbear SSH and dnsmasq.
Filtered hosts significantly reduced reconnaissance visibility.
TCPWrapped services prevented detailed banner extraction.
Operating system fingerprinting accuracy depended heavily on target behavior and service availability.
Reconnaissance tools rely on probability and signature matching rather than certainty.

Limitations

Firewall policies affected scan visibility.
Some services terminated connections before enumeration could complete.
Embedded and mobile devices produced incomplete fingerprint data.
OS detection results were sometimes ambiguous.

Skills Demonstrated

Network Reconnaissance
Nmap Scanning
TCP SYN Scanning
Service Enumeration
Port Enumeration
Host Discovery
OS Fingerprinting
Security Analysis
Technical Documentation

Conclusion

This project demonstrated a complete reconnaissance workflow beginning with host discovery and progressing through TCP SYN scanning, service version enumeration, and operating system fingerprinting.

The results showed that devices within the same network can exhibit significantly different security characteristics. While some hosts exposed identifiable services and operating system signatures, others returned filtered, wrapped, or ambiguous responses due to firewall behavior, access controls, or embedded operating system implementations.

The project further demonstrated that reconnaissance tools should not be treated as sources of absolute truth. Instead, their outputs must be interpreted within the broader context of network infrastructure, filtering mechanisms, and device-specific behavior.

Ultimately, the investigation reinforced the importance of structured reconnaissance methodology and critical analysis when evaluating network assets and interpreting scan results.
