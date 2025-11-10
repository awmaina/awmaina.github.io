---
layout: post
title: " 🛡️ Incident Report: Phishing Attempt via Inbound Email"
author: Alex Maina Weru
date: 2025-11-10 
categories: [Cybersecurity, SOC Analyst]
tags: [ SOC, phishing]
---

## 1. Alert Triage and Classification

**Incident ID:** 8815  
**Incident Type:** Phishing  
**Severity Level:** Medium  
**Date and Time Detected:** Nov 10th, 2025 at 15:04  

---

### 1.1 Summary of Alert

The alert was triggered by an inbound email containing a suspicious external link.  
Initial investigation confirmed the email is a **malicious attempt to steal credentials** through **brand impersonation** and **social engineering**.

---

### 1.2 Initial Findings (Alert Details)

| Field | Detail |
|-------|--------|
| **Time of Activity** | 11/10/2025 12:01:42.516 |
| **Data Source** | Email |
| **Recipient (Affected Entity)** | h.harris@thetrydaily.thm |
| **Sender** | urgents@amazon.biz |
| **Subject** | Your Amazon Package Couldn't Be Delivered - Action Required |
| **Malicious URL** | [http://bit.ly/3sHkX3da12340](http://bit.ly/3sHkX3da12340) |
| **Direction** | Inbound |

---

### 1.3 Incident Classification

**Classification:** ✅ True Positive  

**Reason for True Positive:**  
The email uses multiple definitive indicators of a phishing attempt:

- **Brand Impersonation:** The subject and content impersonate a legitimate Amazon delivery notification.  
- **Suspicious Domain:** The sender uses a deceptive domain (`amazon.biz`) instead of the official Amazon domain.  
- **Obscured Malicious Link:** The content directs the user to confirm shipping information via an unverified bit.ly link, concealing the final malicious destination.  
- **Social Engineering:** The message employs urgency and a threat of package return to pressure the recipient into immediate action.

---

## 2. Escalation Rationale and Status

**Escalation Status:** ✅ YES  

**Reason for Escalating the Alert:**

- **Active Threat and High Risk of Compromise:**  
  This active phishing attempt could lead to financial loss or credential theft if the recipient interacts with the link.  

- **Potential for Wide Spread:**  
  Other employees might have received similar emails, necessitating a review of email gateway filters.  

- **Need for Secondary Review:**  
  The shortened URL requires secure detonation and analysis by the L2/L3 security team to identify and block the final malicious destination.

---

## 3. Remediation and Containment Actions

### 3.1 Recommended Remediation Actions (Checklist)

| Action | Status | Rationale/Detail |
|--------|--------|------------------|
| **Block the Sender** | ✅ Complete | Added sender (`urgents@amazon.biz`) and domain (`amazon.biz`) to blocklist. |
| **Containment** | ✅ Complete | Quarantined/deleted the phishing email from the recipient’s mailbox. |
| **User Education** | ✅ Complete | Notified recipient not to click links or reply. |
| **Link Investigation** | ⚠️ Pending/L2 | Forwarded the bit.ly link for sandbox detonation and analysis. |

---

### 3.2 Further Investigation / Next Steps

- Verify sandbox results to determine the **final phishing page** and capture **IOCs**.  
- Perform a **mail log search** to identify if other employees received the same email.  
- If any users **clicked the link**, execute the **Credential Compromise Playbook** (password resets, session invalidation, endpoint scanning).

---

## 4. Closure

**Rationale for Closure:**  
Initial triage and containment steps are complete.  
The malicious email has been **removed from the recipient's inbox**, and the sender is **blocked at the email gateway**.  
The incident is considered **contained**, pending the completion of advanced link analysis and user activity review.

---

**Report Prepared by:** SOC Analyst  
**Date:** Nov 10th, 2025  
**Status:** ⏳ Contained / Awaiting Final Link Analysis

