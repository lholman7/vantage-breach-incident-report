# vantage-breach-incident-report

# Vantage Breach Incident Report
**Incident ID:** INC-10001  
**Date of Incident:** July 1, 2025  
**Report Date:** March 10, 2026  
**Author:** Holman (Security Analyst)

---

## Executive Summary

On July 1, 2025, at **09:40:07 UTC**, the Vantage website suffered a major breach resulting in the exposure of sensitive user and company data. The attacker gained unauthorized access to the OpenStack cloud environment by brute‑forcing an admin account through the Horizon dashboard. Once inside, they downloaded the OpenStack API configuration, enumerated the environment, and exfiltrated **28 user records** (full names, emails, phone numbers) from Swift object storage. To maintain persistence, the attacker created a new administrative user (`jellibean`) using MITRE ATT&CK technique **T1136.003** (Create Account: Cloud Account).

In accordance with company policy and regulatory requirements, all affected users must be notified within 72 hours. An executive meeting is required to assess business impact, customer trust, and remediation strategies.

---

## Technical Summary

### Phase 1 – Reconnaissance & Subdomain Discovery
- **Time:** Before 09:39 UTC  
- **Tool:** `ffuf@2.1.0` (User‑Agent: “Fuzz Faster U Fool v2.1.0‑dev”)  
- **Action:** The attacker enumerated subdomains of `vantage.tech` and discovered `cloud.vantage.tech`, which hosted the OpenStack Horizon dashboard.  
- **Attacker IP:** `117.200.21.26` (identified via `X‑Forwarded‑For` headers).  
- **Browser Artifacts:** Chrome/Mozilla prefetch headers confirmed manual human interaction.

### Phase 2 – Brute‑Force Login
- **Time:** 09:39:33 UTC (first failed attempt)  
- **Attempts:** 3 total  
- **Successful Login:** At **09:39:xx UTC** (exact time from pcap) the attacker used the credentials of an administrative account.  
- **Method:** POST requests to `/dashboard/auth/login/` with form data.

### Phase 3 – API Access & Configuration Retrieval
- **Time:** 09:40:29 UTC  
- **Action:** The attacker downloaded the OpenStack API remote configuration file.  
- **Keystone Authentication:** The attacker then authenticated to the Keystone identity service and obtained a session token:  
  `req-279aa4af-af73-4f44-9c3f-dc0eaacb791f` (token truncated for security).

### Phase 4 – Post‑Exploitation & Data Exfiltration
- **First API Call to Controller:** 09:41:44 UTC – the attacker enumerated the default project (ID: `9fb84977ff7c4a0baf0d5dbb57e235c7`).  
- **Swift Access:** The attacker listed containers in the Swift object storage service (endpoint `http://134.209.71.220:8080/v1/AUTH_9fb8…`) and discovered three containers:
  - `User Data`
  - `Employee Data`
  - `User/Employee-Details`
- **Data Exfiltration:** At **09:45:23 UTC**, the attacker downloaded a file from one of these containers containing **28 user records** (full names, emails, phone numbers).

### Phase 5 – Persistence
- **Technique:** MITRE ATT&CK **T1136.003** (Create Account: Cloud Account)  
- **Action:** The attacker created a new administrative user named **`jellibean`** with a weak password (`P@$$word` – redacted in this report).  
- **Impact:** This backdoor account allows future re‑entry, bypassing any password changes on the original compromised account.

---

## Recommendations

1. **Enforce Multi‑Factor Authentication (MFA)** – Require MFA for all administrative accounts to prevent brute‑force attacks.
2. **Implement Account Lockout & Rate Limiting** – Block after 3–5 failed attempts.
3. **Restrict Administrative Access** – Limit admin login to trusted networks (corporate VPN or internal IP ranges).
4. **Monitor for Fuzzing Activity** – Deploy detection rules for tools like ffuf (e.g., User‑Agent strings, rapid sequential requests).
5. **Apply Principle of Least Privilege** – Review which accounts have access to sensitive containers; the compromised admin should not have had direct access to user PII.
6. **Enable Comprehensive Logging & Alerting** – Ensure all API interactions, especially user creation and data access, are logged and monitored.
7. **Rotate All Credentials** – Immediately rotate all service account credentials, API keys, and user passwords exposed during the incident.

---

*All timestamps are in UTC.*
