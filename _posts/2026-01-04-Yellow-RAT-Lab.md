---
layout: post
title: "Yellow RAT Lab"
author: Alex Maina Weru
date: 2026-01-04
categories: [Lab and CTF challenges, Threat Intel]
tags: [threat-intel, cyberdefender, labs]
image:
  path: assets/rat/rat8.webp
  alt: rat
---

# Yellow RAT Lab – Threat Intelligence Analysis Using VirusTotal & Red Canary

## Category
**Threat Intelligence**

## Tools Used
- **VirusTotal**
- **Red Canary Threat Intelligence**

## Sample Information
- **SHA-256 Hash:**  
30E527E45F50D2BA82865C5679A6FA998EE0A1755361AB01673950810D071C85

  - **Malware Type:** Remote Access Trojan (RAT)
- **File Type:** Malicious DLL

---

## 1. Scenario Overview

During a routine IT security audit at **GlobalTech Industries**, abnormal outbound network traffic was detected from multiple employee workstations. Further investigation revealed that user search queries were being redirected to unfamiliar websites, suggesting the presence of malware.

To understand the scope and nature of the incident, a suspicious file hash was analyzed using **VirusTotal** and **Red Canary threat intelligence** resources. The goal of this investigation was to identify the malware family, associated indicators of compromise (IOCs), dropped artifacts, and command-and-control (C2) infrastructure.

---

## 2. Question 1: Malware Family Identification

### Objective
Identify the malware family responsible for the abnormal network traffic.

### Methodology
1. Upload the provided SHA-256 hash to **VirusTotal**.
2. Navigate to the **Relations** tab.
3. Review the **Graph Summary** and select the **PEDLL** icon.
4. Examine the tree visualization to identify malware family associations.

![](assets/rat/rat1.png)

### Findings
The analyzed DLL sample exhibits behavior consistent with a **Remote Access Trojan (RAT)**. Further analysis links the sample to the malware family:

Yellow Cockatoo RAT


### Conclusion
The abnormal network behavior was caused by **Yellow Cockatoo RAT**, a known malware family capable of remote command execution, traffic redirection, and data exfiltration.

---

## 3. Question 2: Common Filename Associated with the Malware

### Objective
Identify the common filename used by the malware to assist in IOC-based detection across the environment.

### Methodology
1. In VirusTotal, navigate to the **Details** tab.
2. Locate the **Names** section.
3. Review the listed filenames associated with the sample.

![](assets/rat/rat2.png)

### Findings
- **Common Filename:**  
  111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll

### Conclusion
The identified filename can be used as an **indicator of compromise (IOC)** to scan endpoints using EDR or antivirus solutions. Combined with the SHA-256 hash, this significantly improves detection and remediation efforts.

---

## 4. Question 3: Malware Compilation Timestamp

### Objective
Determine when the malware was compiled to understand its development timeline.

### Methodology
1. Navigate to the **Details** tab in VirusTotal.
2. Scroll to the **Portable Executable Info** section.
3. Identify the **Compilation Timestamp** field.

![](assets/rat/rat3.png)

### Findings
- **Compilation Timestamp:**
2020-09-24 18:26:47 UTC

  
### Conclusion
The malware is not newly created, indicating it may be part of a long-running campaign or reused in multiple attacks. This helps correlate the sample with known threat actor activity.

---

## 5. Question 4: First Submission to VirusTotal

### Objective
Identify when the malware was first observed by the cybersecurity community.

### Methodology
1. In VirusTotal, open the **Details** tab.
2. Locate the **History** section.
3. Identify the **First Submission** date.

![](assets/rat/rat4.png)

### Findings
- **First Submission Date:**  
2020-10-15 02:47:37 UTC

### Conclusion
Comparing the first submission date with the compilation timestamp provides insight into how long the malware may have circulated before detection.

---

## 6. Question 5: Dropped `.dat` File in AppData

### Objective
Identify additional components dropped by the malware to ensure full eradication.

### Methodology
1. Review Red Canary’s analysis of the Yellow Cockatoo campaign:
https://redcanary.com/blog/threat-intelligence/yellow-cockatoo/
2. Identify files written to the AppData directory.


![](assets/rat/rat5.png)

### Findings
- **Dropped File:**
solarmarker.dat

- **Location:**
%USERPROFILE%\AppData\Roaming\


### Conclusion
The `solarmarker.dat` file is likely used for configuration storage, persistence, or activity logging. Its presence is a strong IOC and must be removed to prevent reinfection.

---

## 7. Question 6: Command and Control (C2) Server

### Objective
Identify the C2 server used by the malware to enable blocking and containment.

### Methodology
1. Analyze VirusTotal network indicators.
2. Cross-reference with Red Canary threat intelligence findings.

![](assets/rat/rat6.png)

![](assets/rat/rat7.png)



### Findings
- **C2 Domain:**
https://gogohid.com


### Conclusion
This domain functions as a command-and-control server for Yellow Cockatoo RAT. Blocking this domain at the firewall or proxy level is critical to disrupting attacker communications.

---

## 8. Key Indicators of Compromise (IOCs)

| Type | Value |
|----|----|
| SHA-256 | 30E527E45F50D2BA82865C5679A6FA998EE0A1755361AB01673950810D071C85 |
| Malware Family | Yellow Cockatoo RAT |
| Dropped File | solarmarker.dat |
| C2 Domain | gogohid.com |

---

## 9. Lessons Learned

### 1. Threat Intelligence Accelerates Incident Response
Using platforms like VirusTotal and Red Canary allows defenders to quickly classify malware and understand adversary behavior.

### 2. Older Malware Is Still Dangerous
Despite being compiled in 2020, this malware remains effective, highlighting the importance of patching and behavioral detection over signature-only defenses.

### 3. RATs Enable Long-Term Persistence
Yellow Cockatoo RAT provides attackers with remote control capabilities, making early detection essential to prevent widespread compromise.

### 4. Dropped Files Are Critical Cleanup Targets
Artifacts like `solarmarker.dat` can maintain persistence even after the main executable is removed.

### 5. Blocking C2 Infrastructure Disrupts Attacks
Identifying and blocking C2 domains such as `gogohid.com` prevents command execution and data exfiltration.

---

## 10. Final Remarks

This lab demonstrates how effective threat intelligence analysis can uncover malware families, infrastructure, and behaviors with minimal artifacts. By correlating VirusTotal data with Red Canary research, defenders can build strong IOC-driven detections and reduce dwell time during real-world incidents.

