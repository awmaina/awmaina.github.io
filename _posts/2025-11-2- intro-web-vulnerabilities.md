---
layout: post
title: "Understanding Web Vulnerabilities: A Brief Introduction"
author: Alex Maina Weru
date: 2025-11-02 
categories: [Cybersecurity, Web Security]
tags: [vulnerability, OWASP, injection, security]
image:
      path:assets/Screenshot at 2025-12-07 17-06-30.png
      alt:web vuln
    
---

## 🛡️ The Digital Weak Spots: Introduction to Web Vulnerabilities

![web security example](/assets/web.webp){: width="600"}


In today's digital landscape, web applications are the primary gateway for businesses and users. But just like a physical building, they can have weak spots. These "weak spots" are what we call **web vulnerabilities**. Understanding them is the first step toward building a robust defense.

A web vulnerability is essentially a **flaw or weakness in a web application, its operating system, or its management system** that an attacker can exploit to gain unauthorized access, steal data, or cause disruption.

---

### Why Do Vulnerabilities Exist?

Vulnerabilities often arise from simple mistakes during development:

* **Human Error:** Developers might unintentionally leave holes through poor coding practices (e.g., trusting user input).
* **Misconfiguration:** Web servers, databases, or application firewalls might be set up incorrectly, leaving default settings open.
* **Outdated Software:** Using old versions of frameworks or libraries that contain known, unpatched flaws.

---

### 💥 The Big Three: Common Vulnerability Types

The **OWASP Top 10** is the definitive list of the most critical web application security risks. While the list changes, these three are perennial threats:

1.  **Injection Flaws (SQL Injection, etc.)**
    * **What it is:** Occurs when an attacker sends hostile data to an interpreter (like a database query) that convinces the interpreter to execute unintended commands.
    * **Analogy:** Sneaking an extra instruction onto a recipe that the chef (the interpreter) blindly follows.

2.  **Cross-Site Scripting (XSS)**
    * **What it is:** This flaw allows an attacker to inject malicious client-side scripts into a website viewed by other users.
    * **Impact:** Attackers can hijack user sessions, deface websites, or redirect users to malicious sites.

3.  **Broken Access Control**
    * **What it is:** When users are allowed to access resources or perform actions they are **not** supposed to be able to do.
    * **Example:** A standard user gaining administrative privileges simply by changing a number in the URL (e.g., changing `user_id=123` to `admin_id=1`).

---

### 💻 A Defender’s Mindset

As a cybersecurity professional, my role—whether as a Junior SOC Analyst or a Pentester—is to find and eliminate these flaws. Defensive work focuses on **monitoring** for active exploitation, while offensive work involves **simulating attacks** to proactively discover these weaknesses before the bad guys do.

Security is a continuous cycle, not a one-time fix!
