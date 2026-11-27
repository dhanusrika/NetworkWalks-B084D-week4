# NETWORKWALKS--DHANUSRIKA R-B084-WK4-Penetration-Testing-Project-Medirozahospital

# Week 4 – Black-Box Penetration Testing Assessment

## Cybersecurity Internship | NETWORKWALKS

This README documents my Week 4 practical project as a Cybersecurity Intern at **NETWORKWALKS**. The exercise covered reconnaissance, DNS enumeration, web application firewall identification, network/service scanning, application-security testing, and review of exposed files.

> **Privacy and responsible-disclosure notice:** This public summary intentionally excludes patient names, dates of birth, patient IDs, employee/shareholder details, recovered passwords, PDF hashes, database contents, exploit payloads, exact vulnerable URLs, and backup paths. Do not upload patient documents, database dumps, raw hashes, credentials, or unredacted screenshots to a public repository. Keep raw evidence in an approved private location and share it only with authorised reviewers. If real patient information was accessed, stop further testing and promptly report the incident through the authorised channel.

## Objectives

- Perform reconnaissance within the approved assessment scope.
- Collect domain registration and DNS information.
- Identify web technologies and WAF information where possible.
- Scan the target host and identify exposed ports and services.
- Assess the assigned patient portal and document confirmed security weaknesses.
- Review the security impact of exposed documents and backup data.
- Recommend practical mitigation strategies.

## Tools used

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration, registrar, name-server, and DNSSEC information. |
| WhatWeb | Web technology fingerprinting. |
| cURL | HTTP response and header inspection. |
| NSLOOKUP | DNS resolution. |
| DNSRecon | Enumeration of DNS records. |
| WAFW00F | Web application firewall fingerprinting. |
| Nmap | Host discovery, TCP port scanning, service/version detection, OS fingerprinting, NSE checks, and traceroute. |
| Networkwalks Hash Calculator | Extracting hashes from the supplied PDF files. |
| Networkwalks Password Cracker | Attempting password recovery from the extracted PDF hashes. |
| Custom `special.txt` wordlist | Testing variable-length special-character combinations against the third PDF hash in the lab exercise. |

## Methodology and observations

### 1. WHOIS reconnaissance

WHOIS identified **NameCheap, Inc.** as the registrar. The domain record showed a creation date of August 2026, an expiry date of August 2027, two Namecheap Hosting name servers, and DNSSEC reported as unsigned. Registrant information was privacy-protected.

**Observation:** WHOIS provides useful domain-registration context. An unsigned DNSSEC status is a hardening consideration, not proof of an exploitable vulnerability.

### 2. WhatWeb

WhatWeb returned **408 Request Timeout** for the HTTP request and identified the target IP address. The request did not provide a reliable technology fingerprint.

**Observation:** A timeout is inconclusive; it does not establish which web technologies are present or absent.

### 3. NSLOOKUP

NSLOOKUP, using Google Public DNS (`8.8.8.8`), resolved the domain to an IPv4 address 199.188.201.16

### 4. DNSRecon

DNSRecon reported SOA, NS, MX, A, TXT, DMARC, and SRV records. It also reported that DNSSEC was not configured. The DMARC TXT record used the policy `p=none`.

**Security consideration:** `p=none` is a monitoring policy; it does not instruct recipient mail systems to quarantine or reject messages that fail DMARC. Review legitimate mail flows and consider moving to an enforcement policy after validation.

### 5. WAFW00F

WAFW00F identified the HTTPS site as being behind **LiteSpeed (LiteSpeed Technologies) WAF**.

**Observation:** Identifying a WAF does not prove that the application is secure or that the WAF prevents every attack.

### 6. Nmap network and service scan

Nmap performed a SYN scan of the resolved host, followed by service/version detection, OS fingerprinting, NSE script checks, and traceroute. It reported one host up, 987 filtered TCP ports in the scanned 1,000-port range, and the following open services:

| Port | Service/version reported | Assessment note |
|---:|---|---|
| 21 | FTP – Pure-FTPd with SSL/TLS | Initial connection prompted for credentials; subsequent attempts failed. No successful authentication was established. |
| 25 | SMTP | Nmap could not establish a connection for command enumeration. |
| 26 | SMTP – Exim 4.100.1 | SMTP commands were returned. |
| 53 | DNS | Generic DNS response; service identification was not fully conclusive. |
| 80 | HTTP proxy – HAProxy signature; server header reported openresty | Review whether public access is required. |
| 110 | POP3 – Dovecot | Mail retrieval service detected. |
| 143 | IMAP – Dovecot | Mail access service detected. |
| 443 | HTTPS proxy – HAProxy signature | Nmap received `403 Forbidden`; TLS certificate was for a hosting-provider wildcard domain. |
| 465 | SMTP over SSL/TLS – Exim 4.100.1 | Mail service detected. |
| 587 | SMTP – Exim 4.100.1 | Mail submission service detected. |
| 993 | IMAPS – Dovecot | Encrypted IMAP service detected. |
| 995 | POP3S – Dovecot | Encrypted POP3 service detected. |

OS detection did not produce an exact match. Nmap identified the device role as a load balancer and reported a network distance of 20 hops.

**Interpretation:** An open port is not automatically a vulnerability. Each service needs review against business requirements, secure configuration, patch levels, access restrictions, and encryption requirements. The FTP connection attempts did not demonstrate a successful login.

### 7. Patient portal – SQL injection and document exposure

During the assigned application assessment, I identified a SQL injection weakness in the patient portal login functionality. The weakness allowed access to protected patient PDF documents.

I used the Networkwalks Hash Calculator to extract hashes from the downloaded PDFs and the Networkwalks Password Cracker to attempt password recovery. For the third PDF, I created a custom wordlist containing variable-length special-character combinations and used it in the password-recovery exercise.

The recovered documents contained sensitive patient information, including identifying and demographic fields.

**Impact:** SQL injection that enables access to patient documents presents a critical confidentiality risk. Exposure of health-related personal information may also trigger legal, regulatory, contractual, and incident-response obligations.

This public README does not include injection strings, exploitation steps, patient details, document contents, hashes, or recovered passwords.

### 8. Legacy backup and database exposure

Review of a PDF's document properties revealed a note indicating that a database backup had been moved to an old directory during a site migration. Reviewing the legacy location revealed a database backup containing staff records, salary information, and shareholder records.

**Impact:** A publicly accessible database backup can expose large volumes of confidential personal and business information. This should be treated as a serious data-exposure incident.

The exact directory and backup filename are intentionally omitted from this public repository.

## Summary of findings

| Finding | Assessment evidence | Preliminary risk |
|---|---|---|
| SQL injection in patient portal login | Assessment resulted in access to protected patient PDF documents. | **Critical** |
| Patient information exposure | PDFs contained identifying and demographic information. | **Critical** |
| Publicly accessible legacy database backup | Backup contained staff salary and shareholder records. | **Critical** |
| DMARC monitoring-only policy | DNSRecon reported `p=none`. | **Medium – review and strengthen** |
| DNSSEC not configured | WHOIS and DNSRecon reported unsigned/not configured. | **Informational / hardening** |
| Multiple externally reachable services | Nmap identified web, mail, DNS, and FTP-related services. | **Requires service-by-service review** |
| FTP service exposed | FTP detected; no successful login was achieved. | **Requires configuration review** |
| WAF detected | WAFW00F identified LiteSpeed WAF. | **Informational; not a vulnerability by itself** |

These are preliminary risk indications based on observed impact. A formal report should apply the organisation's agreed severity methodology and include likelihood, affected assets, and validation evidence.

## Mitigation strategies

### 1. Fix SQL injection

- Use parameterised queries or prepared statements for all database operations.
- Never build SQL statements by concatenating untrusted user input.
- Apply server-side input validation and context-appropriate output encoding.
- Give the application database account only the minimum permissions it needs.
- Review authentication and database-access code, conduct secure code review, and retest after remediation.
- Treat a WAF as an additional defensive layer, not a substitute for fixing vulnerable code.

### 2. Protect patient documents and portal access

- Enforce server-side authorisation checks for every patient-document request.
- Ensure users can access only records they are explicitly authorised to view.
- Store sensitive PDFs outside the public web root and serve them through authenticated, access-controlled application endpoints.
- Log document access and alert on unusual download patterns.
- Review access logs to establish what may have been accessed and follow the organisation's incident-response and privacy-notification procedures.

### 3. Secure database backups

- Remove backups and legacy copies from publicly accessible web directories.
- Store backups in private, access-controlled storage and encrypt them at rest.
- Restrict backup access to authorised administrators and approved backup services.
- Establish backup retention and secure-deletion procedures.
- Search web roots and migration locations for forgotten backup files.
- Rotate credentials and secrets if they may have been included in exposed backups.
- Review logs and treat confirmed unauthorised exposure as a security incident.

### 4. Strengthen password and PDF protection

- Use long, unique passwords for sensitive documents.
- Avoid predictable passwords and repeated password patterns.
- Use approved encryption and document-sharing controls appropriate to the information's sensitivity.
- Do not rely on PDF passwords as the only access-control mechanism for patient records.
- Share passwords through a separate approved secure channel.
- Never publish recovered passwords or PDF hashes.

### 5. Improve email authentication

- Ensure SPF authorises only legitimate sending services.
- Monitor DMARC reports and identify legitimate senders that require alignment.
- After validating legitimate mail flows, consider moving DMARC from `p=none` to `p=quarantine` and eventually `p=reject`.
- Deploy DKIM and maintain SPF, DKIM, and DMARC alignment.

### 6. Improve DNS security

- Assess whether DNSSEC can be enabled and correctly maintained by the DNS provider.
- Monitor DNS records for unauthorised changes.
- Restrict DNS administration to authorised accounts and protect registrar accounts with MFA.

### 7. Reduce and secure exposed services

- Confirm that every externally reachable service is required.
- Disable unused services and close unnecessary ports at the host and network firewall.
- Restrict administration and file-transfer services to trusted management networks or VPN access.
- Prefer secure file-transfer protocols such as SFTP or appropriately configured FTPS; disable FTP if it is not required.
- Enforce TLS for mail services and disable weak protocols and ciphers.
- Keep the operating system, mail services, DNS software, web server, proxy, and hosting components patched.

### 8. Improve monitoring and incident response

- Centralise web, database, authentication, file-access, and network logs.
- Alert on unusual login activity, unexpected document downloads, SQL errors, and access to backup files.
- Preserve relevant logs and evidence securely.
- If real patient or employee information was exposed, escalate promptly to the authorised supervisor and the organisation's incident-response/privacy team.
- Conduct a scoped retest after remediation.

## What I learned

This project gave me practical experience across several stages of a black-box penetration test. I learned how to:

- Use WHOIS, WhatWeb, cURL, NSLOOKUP, DNSRecon, and WAFW00F for reconnaissance and infrastructure discovery.
- Interpret A, NS, SOA, MX, SPF, DMARC, and SRV DNS records.
- Use Nmap to identify open ports, enumerate services and versions, review script output, and perform traceroute.
- Interpret scan results carefully: an open port or detected WAF is not automatically a vulnerability.
- Understand how SQL injection can compromise application data confidentiality when database queries are not handled safely.
- Recognise the risks of storing database backups in publicly accessible locations.
- Use PDF hash extraction and password-recovery tools in the assigned lab, including a custom wordlist for a more challenging password.
- Translate technical observations into risk statements and practical remediation recommendations.
- Appreciate the importance of authorisation, privacy, evidence handling, and responsible disclosure when an assessment involves sensitive information.

One of my biggest takeaways was that weaknesses in an application and its supporting file storage can combine to create serious confidentiality risks. A professional penetration test is not only about identifying weaknesses; it is also about documenting their impact responsibly and recommending effective fixes.


## Conclusion

Week 4 was a valuable and challenging stage of my cybersecurity internship at **NETWORKWALKS**. It allowed me to combine reconnaissance and network scanning with application-security assessment, data-exposure analysis, and remediation planning.

The exercise strengthened my understanding of how separate weaknesses can combine to create serious confidentiality risks. It also reinforced that a professional penetration test is not just about finding weaknesses; it is about documenting their impact responsibly and providing clear, actionable recommendations to reduce risk.

---

**Disclaimer:** This repository is a sanitised educational summary of an authorised training exercise. It intentionally excludes sensitive information and operational exploit details. All testing must remain within the written scope and authorisation provided by the assessment owner.
