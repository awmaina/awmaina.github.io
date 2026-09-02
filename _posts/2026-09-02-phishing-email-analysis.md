---
layout: post
title: " Phishing Email Investigation: Header, URL, and Reputation Analysis"
author: Alex Maina Weru
date: 2026-09-02
categories: [Lab and CTF challenges, SOC]
tags: [HTB, phishing, Email, headers]
image:
    path: /assets/cap/cap10.png
    alt: cap
---


> A practical email-forensics write-up based on a LetsDefend Sherlock scenario involving a fraudulent PayPal-themed message.

## Overview

This investigation analyzes a suspicious email that impersonates PayPal and uses a delivery-confirmation lure. The objective is to determine whether the message is malicious by correlating **email-header evidence**, **link analysis**, and **URL-reputation results**.

The investigation was performed in an isolated lab environment. No credentials were entered into the suspicious message, and the embedded URL was treated as potentially malicious.

| Field | Value |
|---|---|
| Scenario | Phishing Email / Sherlock challenge |
| Target VM | `10.129.118.58` |
| Claimed brand | PayPal |
| Investigation focus | Return path, destination domain, reputation, sender IP, and final classification |
| Final classification | **Phishing email** |

## Lab Connection

The challenge VM was accessed through RDP from the Pwnbox environment. Replace the example address with the target IP assigned to your lab session.

```bash
xfreerdp /v:10.129.118.58 /u:letsdefend /p:'' \
  /cert:ignore /dynamic-resolution
```

The evidence archive was located at:

```text
C:\Users\LetsDefend\Desktop\Files\PhishingChallenge.zip
```

The archive password supplied by the challenge was:

```text
infected
```

## Investigation Methodology

The analysis followed a simple evidence-driven workflow:

1. Extract the challenge archive in the isolated lab VM.
2. Open the email evidence and inspect the full Internet headers.
3. Compare the visible sender and PayPal branding with the `Return-Path` value.
4. Inspect the actual destination of the hyperlink rather than trusting its display text.
5. Check the destination URL against reputation evidence.
6. Review the `Received-SPF` header for the sender IP address.
7. Correlate all indicators before deciding whether the message is phishing.

## Finding 1: Return Path

The `Return-Path` header identifies the address used for handling bounced mail. In this case, the address was:

```text
bounce@rjttznyzjjzydnillquh.designclub.uk.com
```

This domain does not align with PayPal's visible branding or the expected identity of a legitimate PayPal notification. The mismatch is a strong indicator that the message was sent through infrastructure unrelated to the impersonated brand.

**Answer:** `bounce@rjttznyzjjzydnillquh.designclub.uk.com`

## Finding 2: Destination Domain

The visible button text in an email can conceal the real destination. The hyperlink was therefore inspected directly by hovering over the button and reviewing the URL shown by the mail client.

The destination began with:

```text
https://storage.googleapis.com/
```

The randomized path following the domain is part of the complete URL, but the challenge asks for the domain name only.

**Answer:** `storage.googleapis.com`

## Finding 3: Is the Destination Domain Suspicious?

`storage.googleapis.com` is a legitimate cloud-hosting domain, but the legitimacy of a hosting provider does not make every object hosted there trustworthy. Attackers can abuse reputable infrastructure to distribute phishing pages, malware, or credential-harvesting content.

In this scenario, the **full embedded URL** was flagged by multiple security vendors in VirusTotal. The reputation result applies to the specific URL and its hosted content; it should not be interpreted as proof that every resource on the broader Google Cloud platform is malicious.

**Answer:** Yes. The URL used in the email was suspicious and had multiple malicious detections.

## Finding 4: Sender IP in the `Received-SPF` Header

The sender IP address should be extracted from the `.eml` file rather than inferred from the visible sender address. Open the message in a text editor and search for the `Received-SPF` header.

The relevant field follows this pattern:

```text
Received-SPF: ... client-ip=<SENDER_IP> ...
```

The source notes for this write-up did not include the extracted IP value. The correct workflow is to copy the address that appears immediately after `client-ip=` in the original header and record it as the answer.

**Answer:** Extract from the original `.eml` file using the `client-ip=` value in `Received-SPF`.

## Finding 5: Final Classification

The message should be classified using the combined evidence rather than a single indicator. Three findings are especially significant:

| Indicator | Observation | Security significance |
|---|---|---|
| Return path | `designclub.uk.com` infrastructure unrelated to PayPal | Indicates sender-identity misalignment |
| Embedded link | Redirects to `storage.googleapis.com` instead of a PayPal-controlled destination | Suggests abuse of third-party hosting and concealed destination infrastructure |
| URL reputation | The complete URL was flagged by multiple security vendors | Provides independent evidence that the link is malicious or unsafe |

The email also used PayPal branding and a delivery-confirmation lure to create urgency and credibility. When combined with the unrelated return path, the non-PayPal destination, and the malicious reputation results, the evidence supports a clear conclusion.

**Final answer:** Yes, this is a phishing email.

## Lessons Learned

This investigation demonstrates why email analysis should not rely on branding or display text alone. A convincing logo and familiar wording can be copied easily, while headers and destination URLs often reveal the underlying infrastructure.

The most reusable checks are to compare the visible sender with the `Return-Path`, inspect the actual hyperlink destination, review authentication-related headers such as `Received-SPF`, and validate suspicious URLs using reputable threat-intelligence sources. These checks are most effective when performed together and in a controlled environment.

## Conclusion

The message impersonated PayPal but failed several trust checks. Its return path pointed to an unrelated randomized subdomain, its hyperlink used a third-party cloud-hosting destination, and the complete URL received multiple malicious detections. These findings establish that the email was designed to deceive recipients and should be treated as phishing.

## References

[1]: `phishingemail.odt` — Source challenge notes supplied for this portfolio write-up.

[2]: [VirusTotal](https://www.virustotal.com/) — URL and file reputation analysis platform referenced by the challenge workflow.

[3]: [RFC 7208: Sender Policy Framework](https://www.rfc-editor.org/rfc/rfc7208) — Background on SPF and the interpretation of SPF-related email headers.

---


