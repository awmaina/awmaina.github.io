---
layout: post
title: "The Crime Lab"
author: Alex Maina Weru
date: 2026-01-15
categories: [Lab and CTF challenges, Endpoint Security]
tags: [endpoint, security, labs]
image:
  path: assets/crime/crime10.webp
  alt: crime
---

# THE CRIME LAB – Android Endpoint Forensics Investigation

## Category
Endpoint Forensics

## Tools Used
- **ALEAPP (Android Logs Events And Protobuf Parser)**
- **DB Browser for SQLite**

---

## Scenario Overview

A murder investigation is currently underway, and the victim’s **Android mobile device** was recovered as a critical piece of digital evidence. Witness statements, interviews with close contacts, and preliminary findings suggested that the victim was involved in **financial trading**, had accumulated **significant debt**, and may have been under pressure shortly before his disappearance.

The objective of this investigation was to perform a **forensic analysis of the Android device**, reconstruct the victim’s:
- Financial activity
- Communications
- Movements
- Planned future actions

By correlating digital artifacts with witness testimony, we aimed to piece together the events leading up to the incident.

---

## Q1 – Trading Application Identification

### Question
What is the **SHA-256 hash** of the trading application primarily used by the victim?

### Methodology
- ALEAPP was used to enumerate **installed applications** from the Android device.
- The investigation focused on applications related to **trading and investments**.
- Each installed app artifact includes metadata such as:
  - Package name (bundle ID)
  - Cryptographic hashes (SHA-256)
 
![](assets/crime/crime1.png)


![](assets/crime/crime2.png)

### Findings
- **Application Name:** Olymp Trade
- **Package ID:** `com.ticno.olymptrade`
- **SHA-256 Hash:**
4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c


This confirmed the trading platform used by the victim and supported witness statements regarding risky financial activity.

---

## Q2 – Debt Amount Determination

### Question
How much money does the victim owe the individual who repeatedly contacted him?

### Methodology
- ALEAPP was used to extract:
  - SMS messages
  - Call logs
- Messages demanding repayment were identified and reviewed.
- Financial values mentioned in conversations were cross-referenced.

![](assets/crime/crime3.png)

### Findings
The extracted SMS conversations revealed repeated requests for repayment.

**Total debt owed:**
250,000 EGP


This aligns with testimony stating the victim invested all his money and accumulated debt.

---

## Q3 – Identity of the Lender

### Question
What is the name of the person to whom the victim owes money?

### Methodology
- The phone number identified from SMS messages and call logs was correlated with:
  - The contacts database extracted via ALEAPP

![](assets/crime/crime4.png)


### Findings
- **Name:** Shady Wahab

This confirms the identity of the individual pressuring the victim for repayment.

---

## Q4 – Victim’s Location on September 20, 2023

### Question
Where was the victim located when he left his residence without informing anyone?

### Methodology
- GPS and location artifacts were analyzed using ALEAPP.
- Android stores location data across:
  - Google Maps activity
  - App snapshots
  - System logs
- Google Maps usage at **23:50:29** on September 20, 2023, provided a critical clue.

![](assets/crime/crime5.png)

![](assets/crime/crime6.png)


### Findings
The last known location of the victim was:
The Nile Ritz-Carlton, Cairo


This location became a pivotal point in the investigation timeline.

---

## Q5 – Intended Travel Destination

### Question
Where did the victim intend to travel after his hotel stay?

### Methodology
- Investigated stored images and documents on the device.
- Looked for:
  - Flight tickets
  - Travel confirmations
  - Reservation screenshots
- Travel-related artifacts were identified within the phone’s media directories.

![](assets/crime/crime7.png)

![](assets/crime/crime8.png)

### Findings
The device contained evidence indicating the victim had plans to travel after his 10-day hotel stay.  
The victim intended to travel to Las Vegas.

---

## Q6 – Discord Meeting Location

### Question
Where was the victim supposed to meet a friend according to Discord conversations?

### Methodology
- ALEAPP was used to extract **Discord chat logs**.
- Conversations were reviewed for:
  - Location mentions
  - Meeting arrangements

![](assets/crime/crime9.png)

### Findings
The victim arranged to meet a friend at:
The Mob Museum


This location may be directly relevant to the events surrounding the incident.

---

## Timeline Reconstruction (Summary)

- Victim heavily involved in **trading (Olymp Trade)**
- Accumulated **250,000 EGP debt**
- Pressured by **Shady Wahab**
- Last known location: **The Nile Ritz-Carlton, Cairo**
- Planned travel shortly after hotel stay
- Arranged a meeting at **The Mob Museum**

---

## Lessons Learned

### 📱 1. Mobile Devices Are Digital Goldmines
Smartphones store rich forensic artifacts including:
- Financial activity
- Communications
- Location history
- Travel plans

### 💬 2. Messaging Apps Reveal Motive and Pressure
SMS and chat logs can expose:
- Financial disputes
- Emotional stress
- Threats or coercion

### 🗺️ 3. Location Data Is Crucial for Timeline Reconstruction
GPS artifacts from apps like Google Maps can accurately pinpoint:
- Last known locations
- Movement patterns
- Time-based activity

### 🔗 4. Correlation Is Key in Forensics
Digital evidence becomes powerful when correlated with:
- Witness testimony
- Call logs
- Contact lists

### 🧠 5. Endpoint Forensics Supports Real-World Investigations
This lab demonstrates how endpoint forensics directly supports:
- Criminal investigations
- Suspect identification
- Event reconstruction

---

## Conclusion

The Crime Lab investigation highlights the importance of **Android endpoint forensics** in reconstructing a victim’s final days. By leveraging ALEAPP and SQLite analysis, it was possible to uncover financial distress, personal relationships, movement history, and future plans — all critical elements in understanding the circumstances surrounding the incident.

This lab reinforces how **digital footprints tell stories that people sometimes cannot**.

---

🕵️ *Every tap, message, and movement leaves evidence.*


