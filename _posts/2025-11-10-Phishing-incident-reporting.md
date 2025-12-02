---
layout: post
title: "🛡️ Incident Report: Phishing Attempt via Inbound Email"
author: "Alex Maina Weru"
date: 2025-11-10 15:04:00 +03:00
categories: ["Cybersecurity", "SOC Analyst"]
tags: ["SOC", "phishing"]
---
<div style="overflow: auto;">
  <div style="float: left; width: 55%; padding-right: 20px; box-sizing: border-box;">
## 1. Alert Triage and Classification

**Incident ID:** 8815  
**Incident Type:** Phishing  
**Severity Level:** Medium  
**Date and Time Detected:** Nov 10th, 2025 at 15:04  

---
</div>
<div style="float: right; width: 45%; box-sizing: border-box;">
    ![Incident Report Snippet](/assets/Screenshot-from-2025-11-03-15-59-36.png)
  </div>
</div>
<div style="clear: both;"></div>
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
| **Malicious URL** | http://bit.ly/3sHkX3da12340 |
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
| **User Education** | ✅ Complete | Notified the recipient not to click links or reply. |
| **Link Investigation** | ⚠️ Pending/L2 | Forwarded the bit.ly link for sandbox detonation
