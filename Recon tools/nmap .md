# Nmap Tutorial 

## Table of Contents
1. [Basic Scanning Techniques](#basic-scanning-techniques)
2. [Scan Types](#scan-types)
3. [Port Specification](#port-specification)
4. [Service & Version Detection](#service--version-detection)
5. [Output Formats](#output-formats)
6. [NSE Scripts](#nse-scripts)
7. [Firewall and IDS/IPS Evasion](#firewall-and-idsips-evasion)
8. [Common Scanning Strategies](#common-scanning-strategies)
9. [Useful Command Reference](#useful-command-reference)

---

## Basic Scanning Techniques

### TCP Connect Scan (-sT)

**Command Syntax:**
```
nmap -sT -p <port_number> <target_ip>
```

**Description:** 
Specifies a TCP connect scan. Nmap uses the TCP connect method to check the status of ports by establishing a complete three-way handshake.

**Output Interpretation:**
- **STATE Column:** Indicates the status of the port
  - `open` - Port is open and accepting connections
  - `closed` - Port is closed but accessible
  - `filtered` - Port is blocked by firewall/filter
  - `unfiltered` - Port is accessible but status unknown
  - `open|filtered` - Cannot determine if port is open or filtered

- **SERVICE Column:** Suggests what kind of service might be running on that port (based on standard port assignments)

**Example:**
```
nmap -sT -p 22,80,443 192.168.1.1
```

---

## Scan Types

### TCP Scan (-sT)
Establishes complete TCP connection. Slow but reliable and does not require root privileges.

### TCP SYN Scan (-sS)
Half-open scan (SYN/ACK). Faster than TCP connect, requires root privileges. Stealthier than full connection.

**Example:**
```
sudo nmap -sS <target_ip>
```

### UDP Scan (-sU)
Scans UDP ports instead of TCP. Useful for discovering services like DNS, SNMP, NTP.

**Example:**
```
sudo nmap -sU <target_ip>
```

### ACK Scan (-sA)
Used to determine firewall rule set. Doesn't determine port status but shows which ports are filtered by firewall.

### Ping Scan (-sP)
Determines which hosts are online without port scanning.

**Example:**
```
nmap -sP 192.168.1.0/24
```

---

## Port Specification

### Common Port Scanning Approaches

#### Scan Most Common 1000 Ports (Default)
```
nmap <target_ip>
```

**Localhost Definition:** Localhost refers to the current device you are working on, represented by the IP address `127.0.0.1`. Scanning the most common 1000 ports is a quick way to get an overview of services running on your local machine.

**Example:**
```
nmap localhost
nmap 127.0.0.1
```

#### Scan All 65535 Ports (-p-)

```
nmap -p- <target_ip>
```

**Note:** In the TCP/IP protocol, there are a total of 65535 ports available. Scanning all of them gives a complete picture of services running on the target but takes significantly more time.

**Example:**
```
nmap -p- localhost
```

#### Scan Specific Ports (-p)

```
nmap -p <port1>,<port2>,<port3> <target_ip>
```

**Example:**
```
nmap -p 22,80,443 192.168.1.1
```

#### Scan Port Range (-p)

```
nmap -p 1-1000 <target_ip>
nmap -p 1000-2000 <target_ip>
```

#### Scan Services by Name (-p)

```
nmap -p http,https,ssh <target_ip>
```

---

## Service & Version Detection

### Version Detection (-sV)

**Command Syntax:**
```
nmap -sV <target_ip>
```

**Description:** 
The `-sV` option tells Nmap to probe open ports to determine service/version information. Nmap will attempt to identify what specific software and version is running on the open port.

**How It Works:**
1. Identifies open ports
2. Sends service-specific probes
3. Analyzes responses to determine software and version
4. Reports findings with confidence level

**Example:**
```
nmap -sV localhost
nmap -sV -p 22,80,110,139,143,445,31337 <target_ip>
```

### Aggressive Scanning (-A)

Enables OS detection, version detection, script scanning, and traceroute. Comprehensive but slower.

**Example:**
```
sudo nmap <target_ip> -p 80 -A
```

---

## Output Formats

Nmap supports multiple output formats for different analysis needs:

### Normal Output (-oN)
Human-readable format, default Nmap output sent to file.

```
nmap -oN normal_output.txt localhost
```

### XML Output (-oX)
Machine-readable format, ideal for parsing and integration with other tools.

```
nmap -oX xml_output.xml localhost
```

### Grepable Output (-oG)
Legacy format, grep-friendly for command-line processing.

```
nmap -oG grepable_output.txt localhost
```

### Output All Formats (-oA)
Saves results in all three formats simultaneously.

```
nmap -oA scan_results localhost
```

**Output Files Generated:**
- `scan_results.nmap` (normal)
- `scan_results.xml` (XML)
- `scan_results.gnmap` (grepable)

---

## NSE Scripts

### Nmap Scripting Engine (NSE)

NSE allows you to write and use scripts to automate network scanning and testing tasks.

### Running Default Scripts (-sC)

```
nmap -sC <target_ip>
```

**Description:** 
The `-sC` option runs a curated list of scripts that the Nmap authors consider useful, safe, and quick. These are default-safe scripts.

### Running Vulnerability Scripts (--script vuln)

```
nmap -sV -p <port> --script vuln <target_ip>
```

**Description:** 
Runs vulnerability detection scripts against open ports. Useful for identifying known vulnerabilities.

**Practical Example from Experience:**
```
sudo nmap <target_ip> -p 80 -sV --script vuln
```

**Output Interpretation:**
- Look for CVE references
- Note service-specific vulnerabilities
- Pay attention to references (e.g., robots.txt, config files)

### Common NSE Script Categories

| Category | Purpose | Example |
|----------|---------|---------|
| `vuln` | Vulnerability detection | `--script vuln` |
| `default` | Default safe scripts | `--script default` |
| `discovery` | Service/host discovery | `--script discovery` |
| `auth` | Authentication bypass | `--script auth` |
| `brute` | Brute force attacks | `--script brute` |
| `exploit` | Exploit verification | `--script exploit` |

### Running Specific Scripts

```
nmap --script <script_name> <target_ip>
```

**Example:**
```
nmap --script ssl-cert <target_ip>
nmap --script http-title <target_ip>
```

---

## Firewall and IDS/IPS Evasion

### Disable Ping (-Pn)

Skips ping sweep and assumes all hosts are online. Useful when target blocks ICMP.

```
nmap -Pn <target_ip>
```

### Disable ARP Ping (--disable-arp-ping)

Disables ARP ping detection, useful against certain firewall configurations.

```
sudo nmap -sV -Pn --disable-arp-ping <target_ip>
```

### Custom Source Port (--source-port)

Uses a specific source port for scanning, often port 53 (DNS) or 80 (HTTP) to bypass filtering.

**Command Syntax:**
```
ncat -nv --source-port <port> <target_ip> <target_port>
```

**Example:**
```
sudo ncat -nv --source-port 53 <target_ip> 50000
```

**Note:** Some systems restrict source port binding; use `sudo` if permission denied.

### Fragment Packets (-f)

Fragments packets into smaller pieces to evade simple packet filters.

```
nmap -f <target_ip>
```

### Decoy Scanning (-D)

Makes it appear that multiple hosts are scanning the target.

```
nmap -D 192.168.1.1,192.168.1.2,ME <target_ip>
```

### Idle Scan (-sI)

Uses a "zombie" host to scan the target, hiding your IP address.

```
nmap -sI <zombie_ip> <target_ip>
```

---

## Common Scanning Strategies

### Strategy 1: Quick Host Discovery

```
nmap -sP 192.168.1.0/24
```

**Goal:** Determine which hosts are alive without port scanning.

### Strategy 2: Common Ports on Multiple Hosts

```
nmap -p 22,80,443 192.168.1.0/24
```

**Goal:** Check critical ports across a network range quickly.

### Strategy 3: Comprehensive Service Discovery

```
sudo nmap -sV -sC -p- <target_ip> -oA comprehensive_scan
```

**Goal:** Detailed service identification on all ports with NSE scripts.

### Strategy 4: Vulnerability Assessment

```
sudo nmap -sV -p 1-10000 --script vuln <target_ip> -oX vulns.xml
```

**Goal:** Identify potential vulnerabilities in services.

### Strategy 5: Stealth Scan (Evasion Focused)

```
sudo nmap -sS -Pn --disable-arp-ping -p- --source-port 53 <target_ip> -oA stealth_scan
```

**Goal:** Minimize detection by IDS/IPS systems.

---

## Useful Command Reference

### Essential Flags Summary

| Flag | Description | Example |
|------|-------------|---------|
| `-sT` | TCP connect scan | `nmap -sT <target>` |
| `-sS` | TCP SYN scan | `sudo nmap -sS <target>` |
| `-sU` | UDP scan | `sudo nmap -sU <target>` |
| `-sV` | Service version detection | `nmap -sV <target>` |
| `-sC` | Default NSE scripts | `nmap -sC <target>` |
| `-A` | Aggressive scan (OS, version, scripts, traceroute) | `sudo nmap -A <target>` |
| `-p` | Specify ports | `nmap -p 22,80,443 <target>` |
| `-p-` | Scan all ports | `nmap -p- <target>` |
| `-Pn` | Skip ping (assume online) | `nmap -Pn <target>` |
| `-oN` | Normal output to file | `nmap -oN out.txt <target>` |
| `-oX` | XML output to file | `nmap -oX out.xml <target>` |
| `-oA` | All formats output | `nmap -oA results <target>` |
| `--script` | Run specific NSE scripts | `nmap --script vuln <target>` |
| `--source-port` | Specify source port | `nmap --source-port 53 <target>` |
| `--disable-arp-ping` | Disable ARP ping | `nmap --disable-arp-ping <target>` |

### Quick Reference Commands

**Basic scan:**
```
nmap <target_ip>
```

**Scan specific ports:**
```
nmap -p 22,80,443 <target_ip>
```

**Detect services and versions:**
```
nmap -sV <target_ip>
```

**Comprehensive scan:**
```
sudo nmap -sV -sC -p- <target_ip>
```

**Output to multiple formats:**
```
nmap -oA results <target_ip>
```

**Scan with firewall evasion:**
```
sudo nmap -sS -Pn --source-port 53 <target_ip>
```

---

## Tips for Ethical Penetration Testing

1. **Always get written permission** before scanning any system
2. **Start with limited scope** (-p 22,80,443) before running comprehensive scans
3. **Use appropriate timing** - slow scans are less likely to trigger alerts
4. **Document all scans** - maintain records of what was scanned and when
5. **Respect network resources** - full port scans can impact network performance
6. **Combine tools** - Nmap works best alongside other tools like netcat, Metasploit, Burp Suite

---

## Common Vulnerability Assessment Workflow

1. **Host Discovery:** Identify live hosts
   ```
   nmap -sP <network_range>
   ```

2. **Port Scanning:** Identify open ports
   ```
   nmap -sS <target_ip>
   ```

3. **Service Detection:** Identify running services
   ```
   nmap -sV <target_ip>
   ```

4. **Vulnerability Scanning:** Find known vulnerabilities
   ```
   nmap --script vuln <target_ip>
   ```

5. **Detailed Analysis:** Deep dive on interesting ports
   ```
   sudo nmap -sV -sC -p <ports> --script discovery <target_ip>
   ```

---

## Learning Resources

- **Official Nmap Project:** https://nmap.org
- **NSE Documentation:** https://nmap.org/nsedoc/
- **Practice Platforms:** HackTheBox, TryHackMe, DVWA
- **Books:** "Nmap Network Scanning" by Gordon Lyon (Nmap Creator)

---
