# Wireshark Tutorial: Network Penetration Testing & CTF Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Installation & Setup](#installation--setup)
3. [Wireshark Interface Overview](#wireshark-interface-overview)
4. [Capture Filters vs Display Filters](#capture-filters-vs-display-filters)
5. [Fundamental Packet Analysis](#fundamental-packet-analysis)
6. [Protocol Analysis for Penetration Testing](#protocol-analysis-for-penetration-testing)
7. [Network Reconnaissance](#network-reconnaissance)
8. [Credential Extraction](#credential-extraction)
9. [Vulnerability Detection](#vulnerability-detection)
10. [CTF-Specific Techniques](#ctf-specific-techniques)
11. [Real-World Penetration Testing Scenarios](#real-world-penetration-testing-scenarios)
12. [Advanced Wireshark Features](#advanced-wireshark-features)
13. [Best Practices](#best-practices)

---

## Introduction

**Wireshark** is a free, open-source network protocol analyzer that captures and displays real-time network packet data. For penetration testers, it is essential for:

- **Network reconnaissance**: Identifying services, protocols, and active hosts
- **Credential extraction**: Recovering plaintext credentials transmitted over unencrypted protocols
- **Vulnerability analysis**: Detecting misconfigurations and protocol weaknesses
- **Incident response**: Identifying suspicious traffic patterns and anomalies
- **CTF challenges**: Analyzing traffic captures to extract flags and solve networking problems

**Key advantage**: Wireshark operates at Layer 2-7 of the OSI model, allowing comprehensive analysis of protocols from Ethernet to application layer.

---

## Installation & Setup

### Linux Installation

```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install wireshark tshark

# Add current user to wireshark group (capture without sudo)
sudo usermod -aG wireshark $USER
newgrp wireshark

# Verify installation
wireshark --version
tshark --version
```

### macOS Installation

```bash
# Using Homebrew
brew install wireshark

# Grant permissions
sudo chmod +rw /dev/bpf*
```

### Windows Installation

Download installer from [wireshark.org](https://www.wireshark.org/) and run the executable. During installation, allow installation of Npcap for packet capture support.

### Verify Capture Capability

```bash
# Check available interfaces
wireshark -D

# List interfaces with tcpdump
sudo tcpdump -D
```

---

## Wireshark Interface Overview

### Main Window Components

| Component | Purpose |
|-----------|---------|
| **Menu Bar** | File operations, capture settings, analysis tools |
| **Toolbar** | Quick access to capture, stop, restart, search functions |
| **Capture Filter Bar** | Define filters before packet capture begins |
| **Interface List** | Select network interface(s) for capture |
| **Packet List Pane** | Displays captured packets chronologically |
| **Packet Details Pane** | Shows packet structure (headers, payloads, protocols) |
| **Packet Bytes Pane** | Displays raw packet data in hexadecimal and ASCII |
| **Status Bar** | Shows packet count, display filter status, capture statistics |

### Color Coding

- **Light Blue**: TCP packets
- **Light Green**: UDP packets
- **Dark Blue**: TCP SYN packets
- **Red**: Error or suspicious packets
- **Black**: Marked/selected packets

---

## Capture Filters vs Display Filters

### Capture Filters

**Definition**: Filters applied BEFORE packet capture. Only matching packets are captured. Reduces file size and overhead.

**Syntax**: BPF (Berkeley Packet Filter) syntax

**Common Capture Filters for Penetration Testing**

| Filter | Purpose |
|--------|---------|
| `host 192.168.1.100` | Capture traffic to/from specific IP |
| `src host 192.168.1.100` | Capture traffic from source IP |
| `dst host 192.168.1.100` | Capture traffic to destination IP |
| `net 192.168.1.0/24` | Capture entire subnet |
| `port 80` | Capture all traffic on port 80 |
| `tcp port 22` | Capture only TCP traffic on port 22 |
| `udp port 53` | Capture only UDP traffic on port 53 |
| `not broadcast and not multicast` | Exclude broadcast/multicast |
| `host 192.168.1.100 and port 443` | Specific host and port combination |
| `tcp[tcpflags] & tcp-syn != 0` | Capture only SYN packets (network scanning) |

**Use Case Example**: Monitoring FTP traffic
```
tcp port 21
```

### Display Filters

**Definition**: Filters applied AFTER capture. All packets captured, but only matching packets displayed. No performance impact on capture.

**Syntax**: Wireshark filter syntax (different from BPF)

**Common Display Filters for Penetration Testing**

| Filter | Purpose |
|--------|---------|
| `ip.addr == 192.168.1.100` | Show traffic involving specific IP |
| `ip.src == 192.168.1.100` | Show packets from source IP |
| `ip.dst == 192.168.1.100` | Show packets to destination IP |
| `tcp.port == 80` | Show TCP traffic on port 80 |
| `tcp.flags.syn == 1` | Show SYN packets |
| `tcp.flags.reset == 1` | Show RST packets (connection resets) |
| `http.request.method == "GET"` | Show HTTP GET requests |
| `http.response.code == 404` | Show HTTP 404 responses |
| `dns.qry.name contains "evil.com"` | Show DNS queries for specific domain |
| `ftp.request.command == "USER"` | Show FTP USER commands |
| `ftp.request.arg` | Show FTP credentials |
| `smtp.auth.username` | Extract SMTP authentication username |
| `smtp.auth.password` | Extract SMTP authentication password |
| `tcp.stream eq 0` | Show only first TCP stream |
| `tcp.payload != ""` | Show packets with payload data |
| `frame.len > 1000` | Show packets larger than 1000 bytes |

---

## Fundamental Packet Analysis

### Understanding TCP Three-Way Handshake

**Scenario**: Analyzing initial connection from attacker (192.168.1.50) to target web server (192.168.1.10:80)

**Packet 1 - SYN**
```
Source: 192.168.1.50:12345 → Destination: 192.168.1.10:80
TCP Flags: SYN (S)
Sequence Number: 1000
```

**Packet 2 - SYN-ACK**
```
Source: 192.168.1.10:80 → Destination: 192.168.1.50:12345
TCP Flags: SYN, ACK (SA)
Sequence Number: 2000
Acknowledgment Number: 1001
```

**Packet 3 - ACK**
```
Source: 192.168.1.50:12345 → Destination: 192.168.1.10:80
TCP Flags: ACK (A)
Sequence Number: 1001
Acknowledgment Number: 2001
```

**Display Filter**: `tcp.flags.syn == 1 || tcp.flags.ack == 1`

### Analyzing TCP Connection Termination (Four-Way Handshake)

| Packet | Direction | Flag | Purpose |
|--------|-----------|------|---------|
| 1 | Client → Server | FIN | Request connection close |
| 2 | Server → Client | ACK | Acknowledge FIN |
| 3 | Server → Client | FIN | Server closes connection |
| 4 | Client → Server | ACK | Acknowledge FIN |

**Display Filter**: `tcp.flags.fin == 1`

### Detecting Connection Resets

Connection resets (RST flag) indicate abrupt connection termination. Often indicates:
- Firewall blocking traffic
- Service crash
- Attacker force-closing connection

**Display Filter**: `tcp.flags.reset == 1`

---

## Protocol Analysis for Penetration Testing

### 1. HTTP/HTTPS Traffic Analysis

#### Unencrypted HTTP Analysis

**Scenario**: Capturing web traffic on internal network

**Display Filter**: `http`

**Useful Sub-Filters**

| Filter | Extracts |
|--------|----------|
| `http.request.method == "POST"` | POST requests (form submissions) |
| `http.request.uri` | Requested URIs |
| `http.host` | Host header information |
| `http.referer` | HTTP referrer (source page) |
| `http.user_agent` | Client user agent |
| `http.request.full_uri` | Complete request URL |
| `http.cookie` | Session cookies |
| `http.response.code == 200` | Successful responses |
| `http.content_type == "text/html"` | HTML responses |

**Practical Example: Extract All Cookies**

Right-click packet → Follow → HTTP Stream

Look for Set-Cookie header:
```
Set-Cookie: PHPSESSID=a1b2c3d4e5f6g7h8; Path=/; HttpOnly
```

**Extract Form Data**

Filter: `http.request.method == "POST"`

In Packet Details pane, expand HTTP section → POST parameters

---

### 2. DNS Analysis

**Scenario**: Identifying command-and-control (C2) server communications or information gathering

**Display Filter**: `dns`

**Key DNS Filters**

| Filter | Purpose |
|--------|---------|
| `dns.qry.name contains "example.com"` | Query for specific domain |
| `dns.qry.type == 1` | Type A records (IPv4) |
| `dns.qry.type == 28` | Type AAAA records (IPv6) |
| `dns.qry.type == 5` | CNAME records |
| `dns.qry.type == 15` | MX records |
| `dns.resp.z == 0` | Successful DNS responses |
| `dns.flags.rcode == 3` | NXDOMAIN responses (domain doesn't exist) |
| `dns.a == 192.168.1.0/24` | DNS responses to internal network |

**Real-World Example: Detecting DNS Tunneling**

DNS tunneling encodes data in DNS queries (exfiltration technique):

```
dns.qry.name contains "a1b2c3d4e5f6.tunnel.example.com"
```

Extremely long or unusual DNS queries indicate tunneling attempts.

---

### 3. FTP Analysis (Plaintext Credentials)

**Scenario**: Penetration tester capturing credentials transmitted over FTP

**Display Filter**: `ftp`

**Extract FTP Commands**

| Filter | Captures |
|--------|----------|
| `ftp.request.command == "USER"` | Username transmissions |
| `ftp.request.command == "PASS"` | Password transmissions |
| `ftp.request.arg` | Command arguments (includes credentials) |
| `ftp.response.code == 230` | Successful login |
| `ftp.response.code == 530` | Login failure |

**Practical Example: Extracting FTP Credentials**

Filter: `ftp.request.command == "USER" || ftp.request.command == "PASS"`

In Packet Details → FTP → Request Argument:
```
USER admin
PASS P@ssw0rd123
```

---

### 4. SMTP Analysis (Email Protocol)

**Scenario**: Extracting email addresses, usernames, and authentication data

**Display Filter**: `smtp`

**Key SMTP Filters**

| Filter | Extracts |
|--------|----------|
| `smtp.data` | Email body content |
| `smtp.auth.username` | SMTP authentication username |
| `smtp.auth.password` | SMTP authentication password |
| `smtp.req.command == "MAIL"` | Sender information |
| `smtp.req.command == "RCPT"` | Recipient information |

**Practical Example: Extract Email Recipients**

Filter: `smtp.req.command == "RCPT"`

Packet Details → SMTP → Request Command Argument shows recipient email.

---

### 5. Telnet Analysis

**Scenario**: Capturing plaintext authentication and commands

**Display Filter**: `telnet`

Telnet transmits all data in plaintext, including credentials and commands.

**Extract Telnet Session**: Right-click Telnet packet → Follow → TCP Stream

Entire terminal session displayed in readable format, including credentials and executed commands.

---

### 6. POP3/IMAP Analysis (Email Retrieval)

**Scenario**: Extracting email credentials and content

**Display Filter**: `pop || imap`

**Key Filters**

| Filter | Purpose |
|--------|---------|
| `pop.request.command == "USER"` | POP3 username |
| `pop.request.command == "PASS"` | POP3 password |
| `imap.request.command == "LOGIN"` | IMAP authentication |
| `pop.response` | POP3 server responses |

---

### 7. DHCP Analysis

**Scenario**: Identifying DHCP servers, IP ranges, and client information

**Display Filter**: `dhcp`

**DHCP Message Types**

| Message | Purpose |
|---------|---------|
| DHCPDISCOVER | Client requests IP configuration |
| DHCPOFFER | Server offers IP address |
| DHCPREQUEST | Client requests offered IP |
| DHCPACK | Server confirms IP assignment |
| DHCPNAK | Server denies IP request |
| DHCPRELEASE | Client releases IP address |

**Extract Assigned IPs**

Filter: `dhcp.option.dhcp_message_type == 5` (DHCPACK)

In Packet Details → Option Name: "IP Address Lease Time" shows assigned IP.

---

### 8. ARP Analysis (Network Reconnaissance)

**Scenario**: Detecting ARP spoofing attacks or mapping network topology

**Display Filter**: `arp`

**ARP Operation Types**

| Type | Description |
|------|-------------|
| 1 | Request (Who has this IP?) |
| 2 | Reply (This is my MAC address) |

**Detect ARP Spoofing**

```
arp.duplicate-address-detected || 
arp.gratuitous || 
arp.suspicious_opcode
```

Multiple MAC addresses claiming same IP address = ARP spoofing.

---

## Network Reconnaissance

### 1. Identifying Active Hosts

**Scenario**: Enumeration phase of penetration test

**Method 1: ICMP Echo Analysis**

```
Filter: icmp.type == 8
```

Shows all ping requests. Responses indicate active hosts.

**Method 2: ARP Analysis**

```
Filter: arp.opcode == 1
```

ARP requests indicate host is attempting to communicate on network.

**Method 3: TCP SYN Analysis**

```
Filter: tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Hosts responding with SYN-ACK have open ports.

### 2. Port Scanning Detection

**Scenario**: Identifying network scanning activity

**Nmap SYN Scan Signature**

```
Filter: tcp.flags.syn == 1 && tcp.window_size < 1024
```

Rapid SYN packets to different ports = port scan.

**Nmap Stealth Scan Characteristics**

- Rapid sequence of different destination ports
- Low TTL values
- Unusual TCP window sizes

### 3. Service Enumeration

**Banner Grabbing via Wireshark**

```
Filter: tcp.flags.syn_ack == 1
```

Follow TCP stream to see service banners:
- SMTP: "220 mail.example.com ESMTP Postfix"
- FTP: "220 ftp.example.com FTP server ready"
- HTTP: "HTTP/1.1 200 OK"

---

## Credential Extraction

### 1. FTP Credential Extraction

**Scenario**: Auditing internal network for insecure protocols

**Step-by-Step Process**

1. Filter: `ftp.request.command == "USER" || ftp.request.command == "PASS"`
2. Examine each packet in Packet Details pane
3. FTP → Request → Argument shows credentials
4. Alternative: Right-click FTP packet → Follow → TCP Stream

**Example Extraction**

```
→ USER admin
← 331 Password required
→ PASS SecurePass123
← 230 Login successful
```

### 2. HTTP Form Data Extraction

**Scenario**: Extracting submitted form data (login forms, search queries)

**Method 1: Filter POST Requests**

```
Filter: http.request.method == "POST"
```

**Method 2: Analyze Request Payload**

1. Right-click POST request packet
2. Select "Follow" → "HTTP Stream"
3. Scroll to bottom to see form data:

```
username=admin&password=P%40ssw0rd&submit=Login
```

**Method 3: Automated Extraction**

In Packet Details → HTTP → HTML Form Urlencoded:
- Shows parsed key-value pairs
- Automatically URL-decodes values

### 3. SMTP Credential Extraction

**Scenario**: Extracting email authentication credentials

```
Filter: smtp.auth
```

Packet Details → SMTP → Authentication shows base64-encoded credentials.

**Decode Base64**

Linux terminal:
```bash
echo "YWRtaW46UGFzc3dvcmQxMjM=" | base64 -d
# Output: admin:Password123
```

### 4. SNMP Community Strings

**Scenario**: Extracting SNMP credentials (management access)

```
Filter: snmp
```

Community strings transmitted in plaintext for SNMP v1/v2c.

Packet Details → SNMPv2-PDU → Community shows the string.

---

## Vulnerability Detection

### 1. Unencrypted Sensitive Protocols

**Definition**: Using plaintext protocols for authentication/sensitive data transmission.

**Detection Filters**

| Protocol | Filter | Risk Level |
|----------|--------|-----------|
| Telnet | `telnet` | CRITICAL |
| FTP | `ftp` | CRITICAL |
| HTTP (sensitive) | `http.request.method == "POST"` | HIGH |
| SMTP | `smtp` | HIGH |
| POP3 | `pop` | HIGH |
| IMAP | `imap` | HIGH |

**Remediation**: Use encrypted alternatives (SSH, SFTP, HTTPS, SMTPS, POP3S, IMAPS)

### 2. Weak SSL/TLS Configuration

**Self-Signed Certificates**

```
Filter: ssl.certificate.subject
```

Look for unusual certificate issuers or expired certificates.

**SSL Version Detection**

```
Filter: ssl.record.version
```

SSL 2.0 or SSL 3.0 = vulnerable (use TLS 1.2+)

### 3. NULL or Default Credentials

**Scenario**: Services accepting empty/default credentials

**HTTP Basic Auth Detection**

```
Filter: http.authorization
```

Basic auth in plaintext. Decode base64 to see credentials.

**Detection Example**

Packet shows header:
```
Authorization: Basic YWRtaW46YWRtaW4=
```

Decodes to: `admin:admin` (default credentials)

### 4. DNS Spoofing/Cache Poisoning

**Detection Indicators**

```
Filter: dns.response_in && dns.qry.name
```

Monitor for:
- Unexpected IP responses for known domains
- Multiple responses for single query
- Responses from non-authoritative servers

### 5. ARP Spoofing Detection

```
Filter: arp
```

**Indicators**

- Same IP with multiple MAC addresses
- Gratuitous ARP replies
- ARP requests from unexpected sources

### 6. SYN Flood Indicators

```
Filter: tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Detection Pattern**

- High volume of SYN packets
- Rapid increase in connections
- Source addresses varying or fixed
- Destination port constant

---

## CTF-Specific Techniques

### 1. Flag Extraction from Packet Payloads

**Scenario**: Capture file contains flag in packet data

**Search Methods**

**Method 1: Keyword Search**

Menu → Edit → Find Packet

Search for keywords: "flag", "FLAG", "ctf{", "HTB{", "picoCTF{"

**Method 2: Display Filter for Text**

```
frame contains "flag"
```

**Method 3: Export All Objects**

File → Export Objects → HTTP

Extracts images, HTML, scripts. Flag often embedded in HTML comments or image metadata.

### 2. Extracting Data from TCP Streams

**Scenario**: Flag hidden in application-layer data

**Step-by-Step Process**

1. Identify relevant TCP connection (filter by IP/port)
2. Right-click packet → Follow → TCP Stream
3. Change stream dropdown to view different directions
4. Toggle "Show data as" for different formats:
   - ASCII: Readable text
   - Raw: Binary data
   - Hex Dump: Hexadecimal view
5. Copy entire stream for external analysis

### 3. Analyzing DNS Exfiltration

**Scenario**: Flag exfiltrated via DNS protocol

**Example**: Flag encoded in DNS subdomain

```
Packet DNS Query: 81626320666c6167.ctf.local
```

**Extraction Process**

1. Filter: `dns.qry.name`
2. Extract all DNS query names
3. Decode from hex:
   ```bash
   echo "81626320666c6167" | xxd -r -p
   # Outputs: [decoded text]
   ```

### 4. Steganography in Network Packets

**Scenario**: Flag hidden in packet headers or unused fields

**Unusual Field Analysis**

- IP ID field (often used for steganography)
- TCP sequence numbers
- ICMP payload
- DNS TTL field

**Extraction**

1. Create display filter for specific protocol
2. Export packet data using tshark:
   ```bash
   tshark -r capture.pcap -T fields -e frame.hex > output.hex
   ```
3. Analyze hex data for patterns

### 5. Multiple Protocol Layers

**Scenario**: Flag requires analysis across multiple protocols

**Multi-Step Analysis**

Example: HTTP over TCP over IP over Ethernet

1. Filter lowest layer: `eth.src == 00:11:22:33:44:55`
2. Progressively add filters for each layer
3. Combine filters: `ip.src == 192.168.1.100 && tcp.port == 80 && http.request.method == "GET"`

### 6. Timing-Based CTF Challenges

**Scenario**: Flag hidden in packet timing or arrival patterns

**Analyze Packet Timing**

Packet List pane shows time delta (time between packets).

**Extract Timing Data**

1. Export to CSV: File → Export Packet Dissections → As CSV
2. Analyze time deltas in spreadsheet
3. Convert timing to ASCII:
   ```
   Short delay (< 0.1s) = 0
   Long delay (> 0.1s) = 1
   ```
   Creates binary sequence = ASCII characters

### 7. Fragmented Packet Reassembly

**Scenario**: Flag split across fragmented IP packets

**Display Filter**: `ipv4.flags.mf == 1 || ipv4.fragment_offset > 0`

Wireshark automatically reassembles fragments. View reassembled data in Packet Details → Internet Protocol Version 4 → Reassembled IPv4 Packets

---

## Real-World Penetration Testing Scenarios

### Scenario 1: Internal Credential Harvesting

**Objective**: Extract plaintext credentials from internal network traffic

**Network Setup**
- Target: 192.168.1.10 (web server)
- Attacker: 192.168.1.50 (internal compromised machine)
- Monitoring: Wireshark on 192.168.1.100 (network monitoring position)

**Execution**

1. **Start Capture**
   ```
   Capture Filter: host 192.168.1.10 or host 192.168.1.50
   ```

2. **Identify Protocols**
   - Look for unencrypted protocols: HTTP, FTP, Telnet, SMTP

3. **Extract Credentials**
   ```
   Display Filter: ftp || telnet || http.request.method == "POST"
   ```

4. **Analysis**
   - FTP login: User credentials in plaintext
   - HTTP POST: Form data with passwords
   - Telnet: All terminal commands and credentials

**Findings Documentation**

| Protocol | Credentials Found | Risk Level |
|----------|-------------------|-----------|
| FTP | user:password | CRITICAL |
| HTTP | Form submission with email/password | HIGH |
| Telnet | SSH access to router | CRITICAL |

**Remediation**
- Disable FTP, use SFTP
- Enforce HTTPS for all web applications
- Disable Telnet, use SSH

---

### Scenario 2: Detecting Network Reconnaissance (Port Scanning)

**Objective**: Identify attacker performing port enumeration

**Network Setup**
- Target: 192.168.1.0/24 (internal network)
- Attacker IP: 203.0.113.50 (external)

**Detection**

1. **Initial Analysis**
   ```
   Display Filter: tcp.flags.syn == 1 && tcp.flags.ack == 0
   ```

2. **Pattern Recognition**
   - Rapid SYN packets
   - Destination ports incrementing: 21, 22, 23, 25, 53, 80, 110, etc.
   - Same source IP: 203.0.113.50

3. **Detailed Investigation**
   ```
   Display Filter: ip.src == 203.0.113.50
   ```

4. **Extract Timeline**
   - Packet 1: SYN to port 21 (FTP) - RST response
   - Packet 2: SYN to port 22 (SSH) - SYN-ACK response (open)
   - Packet 3: SYN to port 80 (HTTP) - SYN-ACK response (open)

**Analysis Results**

Open ports discovered by attacker:
- Port 22 (SSH)
- Port 80 (HTTP)

Attacker would next attempt credentials or exploits against these services.

**Incident Response**
- Block attacker IP at firewall: `iptables -I INPUT -s 203.0.113.50 -j DROP`
- Monitor SSH for brute force attempts
- Check HTTP for exploitation attempts

---

### Scenario 3: Identifying DNS Tunneling (Data Exfiltration)

**Objective**: Detect compromised host exfiltrating data via DNS

**Network Setup**
- Compromised host: 192.168.1.25
- Attacker C2: 203.0.113.100
- DNS queries observed tunneling

**Detection Process**

1. **Initial Filter**
   ```
   Display Filter: dns && ip.src == 192.168.1.25
   ```

2. **Analyze Query Names**
   - Normal DNS: `mail.example.com`, `www.example.com`
   - Suspicious pattern: 
     ```
     a1b2c3d4e5f6g7h8.attacker.com
     i9j0k1l2m3n4o5p6.attacker.com
     q7r8s9t0u1v2w3x4.attacker.com
     ```

3. **Quantify Anomaly**
   - Count DNS queries by domain
   - Statistics tab shows query volume
   - Attacker's domain: 1000+ queries (exfiltration)
   - Legitimate domains: <10 queries

4. **Extract Exfiltrated Data**
   ```bash
   tshark -r capture.pcap -Y "dns && ip.src == 192.168.1.25" -T fields -e dns.qry.name | cut -d '.' -f1
   ```

   Subdomains form encoded data:
   ```
   a1b2c3d4e5f6g7h8 → hex to ASCII
   i9j0k1l2m3n4o5p6 → hex to ASCII
   ```

5. **Decode Data**
   ```bash
   echo "a1b2c3d4e5f6g7h8" | xxd -r -p
   # Outputs exfiltrated information
   ```

**Indicators of Compromise (IOC)**
- Source: 192.168.1.25
- Destination DNS: 203.0.113.100
- Query pattern: High frequency of suspicious subdomains

**Remediation**
- Block DNS to attacker IP
- Quarantine compromised host
- Check for malware/persistence mechanisms

---

### Scenario 4: Analyzing Web Application Exploitation

**Objective**: Detect SQL injection or parameter tampering attack

**Network Setup**
- Target web server: 192.168.1.10:80
- Attacker machine: 192.168.1.50

**Exploitation Detection**

1. **Capture HTTP Requests**
   ```
   Display Filter: http.request.method == "GET" && ip.src == 192.168.1.50
   ```

2. **Examine Request URLs**
   - Normal: `http://target/product.php?id=5`
   - Malicious: `http://target/product.php?id=5 OR 1=1`
   - SQL injection attempt with quote bypassing: `id=5' OR '1'='1`

3. **Extract Full Requests**
   Right-click packet → Follow → HTTP Stream

   ```
   GET /product.php?id=5' UNION SELECT username,password FROM users-- HTTP/1.1
   Host: 192.168.1.10
   ```

4. **Analyze Server Response**
   - Abnormal database errors exposed
   - Unexpected data returned
   - HTTP 500 errors
   - Database table names in error messages

**Attack Timeline**

| Packet # | Request | Response |
|----------|---------|----------|
| 50 | Normal query (id=1) | 200 OK |
| 51 | Testing (id=1') | 500 Server Error |
| 52 | UNION (id=1 UNION...) | 200 with admin data |
| 53 | Exfiltration payload | Sensitive data returned |

**Forensic Findings**
- SQL injection vulnerability confirmed
- Attacker extracted user database
- Exposure: usernames, password hashes, email addresses

**Remediation**
- Implement prepared statements/parameterized queries
- Input validation and sanitization
- Web application firewall rules
- Database access controls

---

## Advanced Wireshark Features

### 1. Using tshark for Automated Analysis

**Definition**: tshark is command-line version of Wireshark for scripting and automation.

**Extract Specific Field**

```bash
# Extract all HTTP Host headers
tshark -r capture.pcap -Y http -T fields -e http.host

# Extract all DNS queries
tshark -r capture.pcap -Y dns -T fields -e dns.qry.name

# Extract IP addresses with data transferred
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e frame.len
```

**Count Protocol Distribution**

```bash
# Count packets by protocol
tshark -r capture.pcap -T fields -e frame.protocol | sort | uniq -c | sort -rn
```

**Extract Credentials**

```bash
# Extract FTP credentials
tshark -r capture.pcap -Y 'ftp.request.arg' -T fields -e ftp.request.arg

# Extract HTTP Basic Auth
tshark -r capture.pcap -Y http.authorization -T fields -e http.authorization
```

### 2. Creating Custom Columns

**Scenario**: Quickly identify relevant information

**Method**
1. Right-click packet column header
2. Select "Column Preferences"
3. Add custom columns:

| Column Name | Field | Purpose |
|------------|-------|---------|
| HTTPS Port | tcp.dstport | Identify encrypted traffic |
| HTTP Method | http.request.method | See request type |
| DNS Query | dns.qry.name | Track DNS lookups |

### 3. Traffic Statistics

**Navigating Statistics Menu**

Menu → Statistics →

| Option | Use Case |
|--------|----------|
| Packet Lengths | Identify large packet transfers (exfiltration) |
| Protocol Hierarchy | Overview of protocols in capture |
| IPv4 Statistics | Top talkers, data volume per IP |
| TCP Stream Graph | Visualize TCP connection timeline |
| Flow Graph | Analyze communication patterns |

**Identify Top Talkers**

Statistics → IPv4 Statistics → All Addresses

Shows:
- IP address
- Packets sent/received
- Data transmitted (MB/GB)
- Indicates heavy communication

### 4. Exporting Data

**Export Objects**

File → Export Objects → HTTP

Extracts all files transmitted via HTTP:
- Images
- JavaScript
- HTML
- Downloads

**Export Packet Dissections**

File → Export Packet Dissections → As CSV/JSON/XML

Exports structured packet data for analysis in spreadsheet or database.

**Export Raw Data**

```bash
# Extract TCP payload only
tshark -r capture.pcap -Y "tcp.payload" -T json > output.json

# Export as hex dump
tshark -r capture.pcap -T json > packets.json
```

### 5. Packet Comments and Annotation

**Scenario**: Documenting findings during investigation

**Add Comments**

1. Right-click packet
2. Select "Packet Comments"
3. Add analyst notes:
   ```
   [CRITICAL] FTP credentials found: admin:P@ssw0rd
   [ATTACK] SQL injection attempt detected
   [IOC] C2 communication to 203.0.113.100:443
   ```

**Filter by Comments**

```
frame.comment contains "CRITICAL"
```

### 6. Decryption of Encrypted Traffic

**HTTPS Decryption** (requires private key)

Edit → Preferences → Protocols → TLS

Add:
- IP address of server
- Port (443)
- Protocol (http)
- Key file (private key)

**WPA/WiFi Decryption**

Edit → Preferences → Protocols → IEEE 802.11

Add WPA PSK:
- SSID name
- Passphrase

Wireshark decrypts wireless traffic if keys provided.

---

## Best Practices

### 1. Capture Best Practices

**Minimize Noise**

- Use capture filters to reduce captured data
- Monitor specific IPs/ports relevant to test
- Avoid capturing all traffic on busy network

**Capture File Management**

- Set file rotation size: Capture → Options → Output tab
- Enables analysis of large captures without overwhelming system
- Organize by date/incident: `2024-01-15-network-scan.pcapng`

**Documentation**

- Record capture start/end times
- Note network conditions
- Document objectives before starting
- Capture metadata:
  ```
  Interface: eth0
  Capture Filter: host 192.168.1.10
  Duration: 14:30 - 15:45 (75 minutes)
  Objective: Monitor target server
  ```

### 2. Analysis Best Practices

**Layer-by-Layer Approach**

1. **Physical Layer**: Frame errors, CRC failures
2. **Data Link Layer**: MAC addresses, ARP issues
3. **Network Layer**: IP routes, TTL values, fragmentation
4. **Transport Layer**: TCP/UDP flows, port usage
5. **Application Layer**: Protocol-specific data, payloads

**Timeline Analysis**

- Note packet timestamps
- Identify attack sequence
- Calculate timing between events
- Correlate with log files

**Pattern Recognition**

- Unusual port usage
- Unexpected protocols
- Data volume anomalies
- Timing irregularities
- Repeated failed connections

### 3. Performance Optimization

**For Large Captures (>1GB)**

```bash
# Index capture file
editcap -r large_capture.pcapng indexed_capture.pcapng

# Split large file
editcap -c 100000 large.pcapng split.pcapng

# Filter before loading
tshark -r capture.pcapng -Y "tcp.port == 80" -w filtered.pcapng
```

**Memory Management**

- Close unrelated captures
- Use display filters instead of opening large files
- Export relevant packets to smaller file
- Use tshark for bulk processing

### 4. Legal and Ethical Considerations

**Authorization**

- Obtain written approval before capturing traffic
- Specify scope: network segment, timeframe, objectives
- Define data handling procedures
- Establish retention/deletion policies

**Data Protection**

- Encrypt capture files
- Limit access to authorized personnel
- Secure storage (encrypted drive)
- Sanitize data before sharing

**Compliance**

- Comply with data protection regulations (GDPR, HIPAA, etc.)
- Document chain of custody
- Maintain audit trail of analysis
- Preserve data integrity for legal proceedings

---

## Quick Reference Filters

### Reconnaissance Filters

```
tcp.flags.syn == 1 && tcp.flags.ack == 0          # Port scan detection
icmp.type == 8                                     # Ping requests
arp.opcode == 1                                    # ARP requests
dns.qry.name                                       # DNS enumeration
```

### Credential Extraction Filters

```
ftp.request.arg                                    # FTP credentials
http.authorization                                 # HTTP Basic Auth
smtp.auth.username || smtp.auth.password          # SMTP credentials
telnet                                             # All telnet traffic
pop.request.command == "USER" || pop.request.command == "PASS"  # POP3 creds
```

### Vulnerability Detection Filters

```
tcp.flags.reset == 1                               # Connection resets
dns.flags.rcode == 3                               # NXDOMAIN responses
ssl.certificate.subject                            # SSL certificate analysis
tcp.window_size == 0                               # Connection problems
```

### Exfiltration Detection Filters

```
dns contains "tunnel"                              # DNS tunneling
ip.len > 10000                                     # Large packets
tcp.payload != ""                                  # Packets with data
dns.qry.name contains "."                          # Suspicious DNS queries
```

---

## Conclusion

Wireshark is an indispensable tool for network penetration testing and CTF challenges. Mastering packet analysis enables:

- **Reconnaissance**: Identify services, versions, and protocols
- **Exploitation**: Understand attack surface and vulnerabilities
- **Post-Exploitation**: Detect data exfiltration and C2 communication
- **Defense**: Identify and block malicious traffic

**Key Takeaways**

1. Combine capture and display filters for efficient analysis
2. Understand OSI layers for systematic troubleshooting
3. Use tshark for automated and scripted analysis
4. Document findings with detailed annotations
5. Follow ethical and legal guidelines
6. Practice with CTF challenges to build expertise

**Continuous Learning**

- Practice with public PCAP files (Wireshark sample captures)
- Participate in CTF challenges
- Analyze real traffic in lab environment
- Read Wireshark documentation for advanced features

---

## Resources

- **Official Wireshark Documentation**: https://www.wireshark.org/docs/
- **Wireshark User Guide**: https://www.wireshark.org/download/docs/
- **Sample PCAP Files**: https://wiki.wireshark.org/SampleCaptures
- **Wireshark Display Filter Reference**: https://www.wireshark.org/docs/dfref/
- **CTF Practice**: HackTheBox, TryHackMe, PicoCTF (networking challenges)

---

- **Last Updated**: 2024
- **Author**: Security Professional
- **Audience**: Junior security analysts, penetration testers, CTF participants
