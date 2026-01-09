---
layout: post
title: "Lespion Lab"
author: Alex Maina Weru
date: 2026-01-09
categories: [Lab and CTF challenges, Threat Intel]
tags: [threat-intel, cyberdefender, labs]
image:
  path: assets/les/Les10.webp
  alt: lespion
---


# Lespion Lab – Insider Threat Investigation Using OSINT

## Category
**Threat Intelligence**

## Tools Used
- **GitHub**
- **Sherlock**
- **Google Search**
- **Google Image Search**
- **Google Maps**

---

## 1. Scenario Overview

A client’s corporate network was compromised and taken offline. Initial forensic findings indicate that the attack originated from a **single internal user account**, strongly suggesting an **insider threat**.

Incident responders preserved digital evidence and identified online artifacts linked to the suspected insider. The objective of this investigation is to:
- Identify exposed credentials
- Correlate online accounts
- Trace personal activities through OSINT
- Attribute malicious activity to a specific individual

---

## 2. Question 1: Exposed API Key on GitHub

### Objective
Identify the API key exposed by the insider in a public GitHub repository.

### Methodology
1. Open the file `Github.txt`.
2. Navigate to the GitHub repository: Project-Build---Custom-Login-Page
3. Inspect the file: Login Page.js
4. Search for hardcoded secrets.

![](assets/les/les1.png)

### Findings
- **Exposed API Key:**
aJFRaLHjMXvYZgLPwiJkroYLGRkNBW


### Conclusion
The insider hardcoded a sensitive API key into a public repository, creating a serious security exposure.

---

## 3. Question 2: Plaintext Password Disclosure

### Objective
Identify the plaintext password exposed in the GitHub repository.

### Methodology
1. Review strings within `Login Page.js`.
2. Identify a Base64-encoded value:
3. Decode the string using **CyberChef**.

![](assets/les/les2.png)

### Findings
- **Decoded Password:**
PicassoBaguette99


### Conclusion
The insider exposed a plaintext password in a public repository, significantly increasing the risk of credential reuse attacks.

---

## 4. Question 3: Cryptocurrency Mining Tool Used

### Objective
Determine the cryptocurrency mining tool used by the insider.

### Methodology
1. Review all **14 GitHub repositories** associated with the insider.
2. Analyze repository descriptions mentioning cryptocurrency or mining activity.

![](assets/les/les3.png)

### Findings
- **Mining Tool Identified:**
XMRig


### Conclusion
The insider used **XMRig**, a common Monero mining tool, indicating misuse of computing resources for unauthorized crypto-mining.

---

## 5. Question 4: Gaming Website Account

### Objective
Identify the gaming platform where the insider holds an account.

### Methodology
1. Locate the insider’s Instagram profile.
2. Identify and scan a QR code displayed on the profile.
3. Follow the extracted URL.

![](assets/les/les4.png)

### Findings
- **Gaming Platform:**
Steam


### Conclusion
The insider maintains an account on the Steam gaming platform, providing additional correlation points for identity attribution.

---

## 6. Question 5: Instagram Profile Link

### Objective
Identify the insider’s Instagram account.

### Methodology
1. Perform a Google search using:
site:instagram.com EMarseille99


![](assets/les/les5.png)

### Findings
- **Instagram Profile:**
https://www.instagram.com/emarseille99


### Conclusion
This Instagram account served as a central pivot for OSINT correlation.

---

## 7. Question 6: Country Visited on Holiday

### Objective
Determine the country visited by the insider during a holiday.

### Methodology
1. Review Instagram image: https://www.instagram.com/emarseille99/p/CAjDd_dlHds/
2. 2. Identify a recognizable landmark.
3. Use **Google Image Search** for landmark identification.

![](assets/les/les6.png)


### Findings
- **Country Identified:**
Singapore


### Conclusion
The insider posted travel images from **Singapore**, revealing location information through social media.

---

## 8. Question 7: City Where the Insider’s Family Lives

### Objective
Identify the city where the insider’s family resides.

### Methodology
1. Review uploaded images on the Instagram profile.
2. Identify multiple posts tagged or recognizable as being from Dubai.

![](assets/les/les7.png)


### Findings
- **City Identified:**
Dubai


### Conclusion
The presence of multiple images from Dubai suggests that the insider’s family resides there.

---

## 9. Question 8: Company Office Location

### Objective
Determine the city where the company’s office is located.

### Methodology
1. Analyze the provided image: office.jpg
2. Identify architectural and environmental clues.
3. Cross-reference using **Google Maps**.

![](assets/les/les8.png)


### Findings
- **Company Office City:**
Birmingham



### Conclusion
The company’s office is located in **Birmingham**, assisting physical and organizational attribution.

---

## 10. Question 9: IP Camera Location

### Objective
Identify the U.S. state where the IP camera is located.

### Methodology
1. Analyze the image: Webcam.png
2. Identify environmental and infrastructural indicators.
3. Cross-reference with geolocation clues.

![](assets/les/les9.png)


### Findings
- **Camera Location (State):**
Indiana


### Conclusion
The IP camera capturing the target was located in **Indiana**, confirming cross-border movement.

---

## 11. Investigation Summary

The insider:
- Exposed API keys and plaintext passwords on GitHub
- Used company resources for cryptocurrency mining
- Maintained multiple online identities
- Revealed travel and family locations via social media
- Enabled attribution through poor operational security (OPSEC)

---

## 12. Lessons Learned

### 1. Public Code Repositories Are High-Risk
Hardcoding secrets in public repositories enables rapid compromise and lateral abuse.

### 2. Insider Threats Combine Technical and Human Factors
Technical misuse paired with careless personal online activity made attribution possible.

### 3. OSINT Can Unmask Anonymous Actors
Cross-platform correlation (GitHub, Instagram, gaming platforms) quickly reveals identity.

### 4. Social Media Is a Goldmine for Intelligence
Travel photos, QR codes, and geotagged images expose sensitive personal information.

### 5. Crypto-Mining Is a Common Insider Abuse
Unauthorized mining tools like XMRig indicate misuse of corporate resources.

---

## 13. Final Remarks

This lab demonstrates how effective OSINT techniques, combined with technical analysis, can identify and attribute insider threats. Poor OPSEC, exposed credentials, and oversharing on social media enabled investigators to reconstruct the insider’s activities and identity with high confidence.


