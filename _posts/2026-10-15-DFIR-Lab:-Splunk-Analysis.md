---
layout: post
title: "DFIR Splunk Analysis Case Study"
author: Alex Maina Weru
date: 2026-10-06
categories: [DFIR Fundamentals, Splunk Analysis]
tags: [DFIR, Threat, splunk]
image:
  path: assets/pcap/s16.png
  alt: threat
---


**DFIR case study · Blue-team DFIR · Splunk Enterprise**  
**Environment:** Training dataset / simulated Windows incident  
**Prepared by:** Alex Maina Weru

## Summary

I analyzed a Splunk CSV export containing Windows event and Sysmon telemetry for a simulated suspicious-download incident. I pivoted from the available log sources to a typosquatted download domain, traced the `update.exe.hta` file to its execution through Windows Script Host components, statically decoded the embedded PowerShell, and correlated related events with a Sysmon ProcessGuid.

The findings in my supplied export point to a download from **`w1ndowsupdate.com`** and follow-on communication to **`3.140.33.120:9001`**. The decoded script behaves like a loader: it creates a web client, retrieves a second-stage response, transforms the response bytes, and passes the result to PowerShell for execution. The GUID-based search returned **267 related events**, including Sysmon network-connection events to the same destination on port 9001.

These findings support a high-confidence assessment that the workstation was exposed to a malicious HTA-based loader and established outbound command-and-control-like communication. The notes supplied for this report do **not** independently establish what the second stage did, whether data was successfully exfiltrated, or whether persistence was installed. I keep those conclusions separate from the evidence actually reviewed.

## Case scope and evidence

The training scenario describes an alert involving employee **Alice** and Windows host **CLIENT2** (`192.168.0.104`) on **30 August 2024**. The workshop states that the investigation should focus on events from **22:50:27 UTC** onward, with the initial suspicious download reported at approximately **22:56 UTC**. [2]

For my Splunk review, I uploaded the provided `Splunk_logs_export.csv` file. The searches in my notes use `source="Splunk_logs_export.csv"`, `sourcetype="csv"`, and `host="parrot"`. In this search context, `parrot` is the host value attached to the imported CSV. It should not be confused with the simulated affected endpoint, `CLIENT2`; I would confirm the export’s host-field semantics before using it as an endpoint identifier.

The exported events include log names such as **System** and **Windows-Sysmon**. The workshop also identifies PowerShell and Security logs as useful sources. Relevant event types include Sysmon **1** (process creation), **3** (network connection), **11** (file creation), and **22** (DNS query); PowerShell **4104** (script-block logging); System **7045** (service installation); and Task Scheduler **106** (task registration). Sysmon **15** can record file-stream creation, including Mark-of-the-Web-related evidence when present. [1] [2]

## Splunk workflow

Splunk provided a central place to search the imported records and correlate events. In a typical deployment, forwarders send event data to indexers for indexing and storage, while a search head provides the interface for searches, reports, and alerts. The investigation follows a practical triage pattern: inventory the data, search for the initial indicator, pivot to the downloaded file, reconstruct execution, and correlate related network activity.

### 1. Inventory the imported events

I first counted events by Windows log name to understand which sources were available:

```spl
source="Splunk_logs_export.csv" host="parrot" sourcetype="csv"
| stats count by LogName
```
![](/assets/pcap/s0.png)

![](/assets/pcap/s1.png)

The export contained roughly **4,000 events**. This inventory step matters because the available evidence limits what can be concluded. For example, process ancestry is much easier to reconstruct when Sysmon process-creation events are present, and PowerShell behavior is more visible when script-block logging is enabled.

### 2. Search the suspicious domain

I searched for the domain spelling present in my exported findings and sorted the results chronologically:

```spl
source="Splunk_logs_export.csv" host="parrot" sourcetype="csv" *w1ndowsupdate.com*
| sort 0 _time
```
![](/assets/pcap/s2.png)

My notes record **five matching events** among approximately 4,000 events. The records show Alice using **Firefox** to query or connect to `w1ndowsupdate.com`. The DNS-related result associates that domain with **`3.140.33.120`**.
![](/assets/pcap/s3.png)

![](/assets/pcap/s4.png)

![](/assets/pcap/s5.png)
This search establishes a useful pivot, but a domain match alone does not prove that a downloaded file executed. I therefore searched for the filename and correlated the related process and file events.

### 3. Trace the downloaded HTA

```spl
source="Splunk_logs_export.csv" host="parrot" sourcetype="csv" *update.exe.hta*
| sort 0 _time
```

The search returned **six related events**, according to my notes. A Sysmon event **15** records file-stream information for `update.exe.hta`. The associated Zone.Identifier details include:

- **Referrer URL:** `http://w1ndowsupdate.com:8000/`
- **Host URL:** `http://w1ndowsupdate.com:8000/update.exe.hta`

![](/assets/pcap/s6.png)

![](/assets/pcap/s7.png)

This ties the file to the suspicious download source and gives a second artifact-based pivot in addition to the DNS record. The notes report that Alice launched the file through Windows Explorer about a minute after the file-stream event. A Sysmon process-creation event then shows **`mshta.exe`** involved in opening the HTA, followed by a child **`powershell.exe`** process.

![](/assets/pcap/s8.png)

![](/assets/pcap/s9.png)

![](/assets/pcap/s10.png)

### 4. Inspect the HTA and decode the embedded command

I downloaded `update.exe.hta` for static inspection and opened it in a text editor rather than executing it. The file had an HTML structure and used a shell object to run a Base64-encoded command. I decoded the embedded data with CyberChef and removed the null bytes to make the PowerShell readable.

The decoded script creates a .NET `WebClient`, sets a browser-like User-Agent, configures proxy handling, and requests data from a remote endpoint. The server address is represented in Base64 within the script; decoding it produces **`http://3.140.33.120:9001`**. The script appends the path **`/login/process.php`** and calls `DownloadData` to retrieve bytes. It treats the first four bytes as an initialization value, transforms the remaining bytes with a custom RC4-like stream routine, and pipes the resulting content into **`IEX`** (PowerShell’s expression evaluator).

That behavior is consistent with a staged loader: the HTA launches PowerShell, PowerShell retrieves an additional payload, decodes or decrypts it, and executes it in memory. The supplied notes do not include the retrieved second-stage payload itself, so its full behavior remains unknown.
![](/assets/pcap/s11.png)

![](/assets/pcap/s12.png)

![](/assets/pcap/s13.png)

![](/assets/pcap/s14.png)

### 5. Correlate the activity with ProcessGuid

I used the Sysmon ProcessGuid and ParentProcessGuid to collect events linked to the process and its parent/child relationships:

```spl
source="Splunk_logs_export.csv" host="parrot" sourcetype="csv"
(ProcessGuid="{4cd32793-4cc8-66d2-7701-000000001900}"
 OR ParentProcessGuid="{4cd32793-4cc8-66d2-7701-000000001900}")
| sort 0 _time
```
![](/assets/pcap/s15.png)

Reports **267 matching events** for this GUID-centered search. The results include Sysmon event **3** network-connection records from the client to **`3.140.33.120:9001`**. Repeated connections make this a high-value pivot for defining the activity window and correlating process behavior with outbound communications.

The ProcessGuid is particularly useful here because it can link a process-creation record to related network or file events even when multiple processes have similar names. The OR condition also returns events where the GUID appears as the parent, allowing review of nearby process relationships.

## Findings

| Finding | Evidence in the supplied notes | Assessment |
|---|---|---|
| Suspicious domain access | Firefox-related DNS/search events for `w1ndowsupdate.com`; five matches reported | Strong initial indicator; verify exact event fields and DNS response in the original export. |
| File download | Sysmon event 15 for `update.exe.hta`; Zone.Identifier-style referrer and host URLs point to the domain on port 8000 | Strong evidence connecting the HTA to the suspicious source. |
| User execution | The file search shows Alice launching the HTA via Explorer; process creation includes `mshta.exe` and child `powershell.exe` | Supports execution of the downloaded HTA and a script-driven follow-on. |
| Loader behavior | Static decoding reveals a WebClient request to `3.140.33.120:9001/login/process.php`, byte transformation, and `IEX` | Consistent with retrieval and in-memory execution of a second-stage payload. |
| Related network activity | The ProcessGuid/ParentProcessGuid search returns 267 matches, including connections to `3.140.33.120:9001` | Strongly supports repeated communication associated with the suspicious process chain. |
| Follow-on impact | The second-stage payload and later endpoint activity were not included in the supplied findings | Persistence, privilege escalation, lateral movement, and successful data theft are not confirmed by this analysis alone. |

## Conclusion

The supplied Splunk findings reconstruct a coherent initial compromise sequence: Alice’s browser accesses a suspicious domain, `update.exe.hta` is downloaded from that domain, the HTA is launched through `mshta.exe`, and PowerShell executes an encoded loader. Static analysis of the script identifies a request to `3.140.33.120:9001/login/process.php`; the GUID-centered search then links the suspicious process context to repeated network-connection records to that destination.

Based on these records, I assess the case as a **high-confidence suspicious HTA / PowerShell loader incident with repeated outbound communications**. The strongest next steps would be to preserve the original CSV and HTA, collect the full Sysmon process tree and PowerShell 4104 records, review the PCAP for the destination and port, and examine the affected host for the retrieved payload and any persistence or staging artifacts. Those steps would be needed to establish the final payload’s purpose and determine whether data was transferred.

## Evidence boundaries and discrepancies

The hostname and IP in the supplied CSV findings differ from the values in the video-based notes prepared earlier. My export findings and decoded script point to **`w1ndowsupdate.com` → `3.140.33.120`**. The workshop case-introduction slide also prints `w1ndowsupdate.com`, but the video walkthrough has been described with **`windowsupdate.com` → `217.160.33.120`**. These values should not be silently merged. They may reflect a different dataset or course-material version, or a transcription difference. For the analysis above, I use the values in my own export and decoded HTA notes as the primary evidence. I would validate the final IOC set against the original CSV, the HTA bytes, and the PCAP before publishing it as a definitive indicator list. 


The domain and IP are kept in code formatting so they do not become clickable links, while remaining exact for reproducibility. This is a training-lab case study, not evidence of a live incident.

## References

[2]: https://bluecapesecurity.com/wp-content/uploads/2024/09/Workshop_slides_v4.pdf "Blue Cape Security — Incident Response and Data Collection Techniques workshop slides"
