# Nmap Vulnerability Scanning

## Commands

```bash
nmap -sV -vv --script vuln TARGET_IP
```

or:

```bash
nmap --script vuln TARGET_IP
```

---

## 1. `nmap`

Starts the **Nmap** network scanning tool.

Nmap is used to discover:

* Open ports
* Running services
* Service versions
* Possible vulnerabilities

---

## 2. `-sV`

### Service Version Detection

```bash
-sV
```

Tells Nmap to identify the **service and its version** running on open ports.

Example:

```text
80/tcp   open   http    Apache httpd 2.4.49
22/tcp   open   ssh     OpenSSH 8.2
```

This is useful because knowing the service version can help determine which vulnerabilities may be relevant.

---

## 3. `-vv`

### Very Verbose Output

```bash
-vv
```

`-v` = verbose

`-vv` = **more verbose information**

It makes Nmap display more information about what it is discovering while the scan is running.

Think of it as:

```text
Normal:
Nmap gives you the result

-v:
Nmap gives you more information

-vv:
Nmap gives you even more information
```

---

## 4. `--script vuln`

This tells Nmap to use the **Nmap Scripting Engine (NSE)** and run scripts from the `vuln` category.

```bash
--script vuln
```

`vuln` = vulnerability detection

These scripts perform various checks for known vulnerabilities.

### Important

You **do not need to remember every script name**.

Instead of manually specifying:

```bash
--script=smb-vuln-ms17-010
```

you can use:

```bash
--script vuln
```

to run vulnerability-related NSE scripts selected by the `vuln` category.

---

## 5. `TARGET_IP`

This is the IP address of the system you are authorized to scan.

Example:

```bash
nmap -sV -vv --script vuln 192.168.1.10
```

---

# Understanding the Full Command

```bash
nmap -sV -vv --script vuln TARGET_IP
```

Read it from left to right:

```text
nmap
  ↓
Start Nmap

-sV
  ↓
Identify services and versions

-vv
  ↓
Show detailed output

--script vuln
  ↓
Run vulnerability-detection NSE scripts

TARGET_IP
  ↓
Scan this target
```

---

# Why Use `-sV` + `--script vuln`?

The basic idea is:

```text
Target
  ↓
Find open ports
  ↓
Identify services
  ↓
Identify versions
  ↓
Run vulnerability checks
  ↓
Analyze the results
```

For example:

```text
Port 445
   ↓
SMB service
   ↓
Version/service information
   ↓
Run relevant vulnerability checks
   ↓
Possible vulnerability detected
```

---

# Difference Between the Two Commands

### Command 1

```bash
nmap -sV -vv --script vuln TARGET_IP
```

Does:

* Service/version detection
* Verbose output
* Vulnerability-script checks

### Command 2

```bash
nmap --script vuln TARGET_IP
```

Does:

* Vulnerability-script checks

It does **not explicitly request `-sV` service-version detection** or `-vv` verbose output.

---

# Key Concept: NSE Scripts

NSE = **Nmap Scripting Engine**

NSE scripts extend Nmap's normal functionality.

They can be used for tasks such as:

* Vulnerability detection
* Service enumeration
* Information gathering
* Authentication-related checks
* SMB checks
* HTTP checks

You can see available scripts on Kali/Linux with:

```bash
ls /usr/share/nmap/scripts/
```

Search for scripts related to a service:

```bash
ls /usr/share/nmap/scripts/ | grep smb
```

Get information about a specific script:

```bash
nmap --script-help SCRIPT_NAME
```

Example:

```bash
nmap --script-help smb-vuln-ms17-010
```

---

# Important Learning Methodology

Do **not** try to memorize hundreds of NSE script names.

Instead:

```text
1. Find an open port
       ↓
2. Identify the service
       ↓
3. Think about what you want to check
       ↓
4. Search available NSE scripts
       ↓
5. Read the script description
       ↓
6. Select the appropriate script/category
       ↓
7. Analyze the result
```

Example:

```text
445 open
   ↓
SMB
   ↓
Need to check SMB vulnerabilities
   ↓
Search NSE SMB scripts
   ↓
Find relevant script
   ↓
Read its help
   ↓
Run the check
```

> **Main idea:** Nmap is not just about memorizing commands. Learn to understand **what information you need and how to make Nmap obtain that information**.

## Quick Cheat Sheet

| Option          | Meaning                                             |
| --------------- | --------------------------------------------------- |
| `nmap`          | Start Nmap                                          |
| `-sV`           | Service/version detection                           |
| `-v`            | Verbose output                                      |
| `-vv`           | More verbose output                                 |
| `--script`      | Run an NSE script                                   |
| `--script vuln` | Run scripts in the vulnerability-detection category |
| `TARGET_IP`     | Target IP address                                   |

## Example

```bash
nmap -sV -vv --script vuln 192.168.1.10
```

**Meaning:**

> Scan the target, identify its services and versions, show detailed output, and perform vulnerability checks using Nmap's NSE vulnerability scripts.

**Use vulnerability scanning only on systems you own or have explicit permission to test.**
