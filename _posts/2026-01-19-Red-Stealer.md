---
layout: post
title: "Red Stealer Lab – Threat Intelligence Analysis"
author: Alex Maina Weru
date: 2026-01-19
categories: [Lab and CTF challenges, Threat Intel]
tags: [RedLine Stealer, Trojan, VirusTotal, MalwareBazaar, C2, MITRE ATT&CK]
image: 
     path: /assets/red/red10.webp
     alt: red
---

# Red Stealer Lab – Threat Intelligence Analysis

## Category
**Threat Intelligence**

## Tools Used
- VirusTotal  
- MalwareBazaar  
- ThreatFox  
- WHOIS  
- ANY.RUN  

## MITRE ATT&CK Tactics
- **Execution**
- **Persistence**
- **Privilege Escalation**
- **Defense Evasion**
- **Discovery**
- **Collection**
- **Impact**

---

## Scenario Overview

As part of the SOC Threat Intelligence team, an executable file was discovered on a colleague’s workstation and suspected to be malware communicating with a Command and Control (C2) server. The goal of this investigation was to analyze the malware hash using multiple threat intelligence platforms to extract Indicators of Compromise (IOCs), identify malware classification, C2 infrastructure, MITRE ATT&CK techniques, and privilege escalation mechanisms.

This intelligence is intended to support Incident Response (IR) teams in containment, eradication, and future detection.

---

## Q1: Malware Category (VirusTotal – Microsoft)

### Objective
Determine how Microsoft classifies the malware to understand its behavior and threat level.

### Analysis
Reviewing the **VirusTotal → Details → Category** section under Microsoft’s detection results.

![](assets/red/red1.png)

### Finding
**Malware category:**
Trojan


This classification indicates the malware disguises itself as a legitimate file while performing malicious actions in the background.

---

## Q2: Malware File Name

### Objective
Identify the malware file name for consistent communication within the SOC.

### Analysis
The file name was extracted from the **VirusTotal → Names** section.

![](assets/red/red2.png)

### Finding
**Associated file name:**
WEXTRACT


This name may be used as part of social engineering or masquerading techniques to appear benign.

---

## Q3: First Submission Timestamp

### Objective
Establish when the malware was first observed to assess its novelty and urgency.

### Analysis
The **First Submission** timestamp was reviewed in VirusTotal.

![](assets/red/red3.png)

### Finding
**First submission (UTC):**

2023-10-06 04:41:50 UTC


This indicates the malware is relatively recent and may still be actively evolving.

---

## Q4: MITRE ATT&CK – Data Collection Technique

### Objective
Identify the MITRE ATT&CK technique used for data collection prior to exfiltration.

### Analysis
The malware exhibits behavior consistent with collecting local system data before transmitting it externally.

![](assets/red/red4.png)

### Finding
**MITRE ATT&CK Technique ID:**

T1005 – Data from Local System


This technique is commonly used by information-stealing malware families.

---

## Q5: Social Media Domains Resolved via DNS

### Objective
Identify legitimate domains resolved by the malware to evade detection.

### Analysis
Post-execution network traffic shows DNS queries to well-known social media platforms.

![](assets/red/red5.png)

### Finding
**Resolved domain:**

facebook.com


Using trusted domains allows the malware to blend into normal user traffic and potentially bypass network security controls.

---

## Q6: Command and Control (C2) Infrastructure

### Objective
Identify the malicious IP address and destination port used for C2 communication.

### Analysis
Decoded configuration data revealed hardcoded network indicators.

![](assets/red/red6.png)

### Findings
**C2 Server:**

IP Address: 77.91.124.55
Port: 1971


This infrastructure should be immediately blocked at firewall and IPS levels. Additional fallback URLs were also observed, indicating redundancy in C2 communication.

---

## Q7: MalwareBazaar YARA Rule

### Objective
Identify existing YARA rules to assist in detection and hunting.

### Analysis
Searching **MalwareBazaar** for YARA rules created by analyst *Varp0s*.

![](assets/red/red7.png)

### Finding
**YARA rule name:**

detect_Redline_Stealer


This rule can be leveraged by SOC teams to detect the malware across endpoints and forensic datasets.

---

## Q8: Malware Alias (ThreatFox)

### Objective
Identify alternative malware family names associated with the malicious IP.

### Analysis
ThreatFox was queried for aliases linked to the C2 infrastructure.

![](assets/red/red8.png)

### Finding
**Malware alias:**

RECORDSTEALER


This indicates overlap with other credential-stealing malware families and shared infrastructure.

---

## Q9: DLL Used for Privilege Escalation

### Objective
Identify DLLs leveraged by the malware to perform privilege escalation or sensitive system operations.

### Analysis
Inspection of imported libraries revealed usage of Windows API libraries commonly associated with token manipulation and privilege management.

![](assets/red/red9.png)

### Finding
**DLL used for privilege escalation:**

advapi32.dll


This DLL provides access to advanced Windows security functions and is often abused for credential access and privilege escalation.

---

## Indicators of Compromise (IOCs)

### Network IOCs
- **IP Address:** `77.91.124.55`
- **Port:** `1971`
- **Domain:** `facebook.com` (abused for evasion)

### Host-Based IOCs
- **File Name:** `WEXTRACT`
- **Malware Category:** Trojan
- **YARA Rule:** `detect_Redline_Stealer`

---

## Lessons Learned

### 1. Legitimate Services Can Be Abused
Attackers frequently leverage trusted platforms such as social media domains to hide malicious traffic in plain sight.

### 2. Early Threat Intelligence Saves Time
Knowing the malware category, aliases, and C2 infrastructure enables faster containment and response by SOC and IR teams.

### 3. YARA Rules Are Powerful Detection Tools
Community-driven YARA rules significantly enhance detection coverage and should be integrated into SOC workflows.

### 4. Privilege Escalation Increases Impact
Use of system DLLs like `advapi32.dll` highlights how quickly malware can move from user-level execution to higher privileges.

### 5. Infrastructure Reuse Reveals Malware Families
Shared IPs and aliases across platforms such as ThreatFox help uncover relationships between malware campaigns.

---

## Conclusion

This lab demonstrates how combining **VirusTotal, MalwareBazaar, ThreatFox, and dynamic analysis tools** provides a comprehensive understanding of malware behavior and infrastructure. By extracting actionable intelligence, SOC teams can improve detection, containment, and long-term defensive strategies against information-stealing malware such as RedLine / Red Stealer.

---


