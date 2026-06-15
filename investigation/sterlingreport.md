# Sterling Bank Investigation Post-Incident Analysis

**Classification:** Confidential — For Internal Use

**Assessment Date:** June 2026

**Framework References:** NIST CSF 2.0 · ISO/IEC 27001:2022 · CBN Cybersecurity Framework 2022 · NDPA 2023

**Prepared by:** Joy Usman (aka Phoenixx), Internet Crimes Security Research

## Executive Summary
Sterling Bank experienced a material cybersecurity breach between approximately March 18 and March 27, 2026, perpetrated by the threat actor identified as ByteToBreach. The actor exploited a publicly known, maximum-severity vulnerability (CVE-2025-55182, CVSS 10.0) on an internet-facing test server to gain initial access, subsequently maintaining undetected persistence for nine days before exfiltrating sensitive employee and customer data.

The breach is notable not for its technical sophistication, none was required but for the complete absence of the fundamental controls mandated by the CBN Cybersecurity Framework 2022, to which Sterling Bank is legally obligated as a licensed deposit money bank.

## Findings at a glance
**Critical findings at a glance:**

| Finding | Severity |
| --- | --- |
| Maximum-severity CVE unpatched on public-facing server | Critical |
| Nine days attacker dwell time with zero detection | Critical |
| Encryption keys stored in plaintext client-side code | Critical |
| Lateral movement into Remita (national payment infrastructure) | Critical |
| No breach notification issued to 900,000+ affected customers | High |
| Ransom negotiation conducted without regulatory disclosure | High |
| No public incident statement as of assessment date | High |

**Regulatory exposure:** Sterling Bank faces potential action under the CBN Cybersecurity Framework, the Nigeria Data Protection Act 2023, and the Cybercrimes (Prohibition, Prevention, Etc.) Amendment Act 2024. The failure to notify affected data subjects of a breach involving their personal and financial data is a direct violation of Section 40 of the NDPA 2023.

This assessment provides a structured risk analysis and a remediation roadmap prioritised by regulatory urgency and business impact.

## 2. THE INCIDENT NARRATIVE

### 2.1 Background

Sterling Bank operates a digital banking platform serving approximately 900,000 retail and corporate customers in Nigeria. As part of its technology infrastructure, the bank maintained internet-facing testing environments — servers used by development teams to test application features before promoting code to production.

One such server, identified in post-incident analysis as *enf-pilot.sterling.ng*, was exposed to the public internet with a critical unpatched vulnerability. This server was running a web application affected by CVE-2025-55182, a vulnerability assigned the maximum possible severity score of 10.0 under the Common Vulnerability Scoring System.

## Timeline of Events

| Date | Event |
| --- | --- |
| March 18, 2026, 17:49 | Initial exploitation of CVE-2025-55182 via Metasploit. Attacker gains shell access to enf-pilot.sterling.ng |
| March 22, 2026 | Sliver C2 framework deployed. Attacker establishes persistent encrypted communication channel. First confirmed check-in. |
| March 22–26, 2026 | Network reconnaissance within Sterling Bank's Kubernetes cluster. 168 internal services enumerated. |
| March 23–26, 2026 | Attacker queries Temenos T24 core banking system. Employee database exfiltrated. Customer account data and credit bureau records accessed. |
| March 24–26, 2026 | Encryption keys discovered in plaintext JavaScript client-side code. Key material obtained. |
| March 26–27, 2026 | Lateral movement into Remita infrastructure via trusted internal network connection. |
| March 27, 2026 | Exfiltrated data published to cybercrime forum spear.cx. Ransom demand of €250,000 communicated to Sterling Bank. |
| March 27–April 2026 | Sterling Bank engages in ransom negotiation. Negotiation stalls. No public disclosure made. |
| April 8, 2026 | Security researcher David Odes publishes initial findings. Breach enters public domain. |
| April 15, 2026 | Direct interview with ByteToBreach published. Attacker confirms no data modification; purpose was exfiltration and one-time sale. |
| April 20, 2026 | National media coverage. NDPC opens investigation. |
| Assessment date | Sterling Bank has issued no public statement. No customer notification confirmed. |

## Scope of Data Compromised

| Data Category | Volume | Sensitivity |
| --- | --- | --- |
| Employee records (names, emails, phone numbers, staff IDs, supervisor chains, access permissions) | 3,009 records | High |
| Customer account data (account numbers, transaction histories, loan portfolios) | Up to 900,000 records | Critical |
| Consumer credit profiles via Credit Risk Central (queried by BVN) | Unknown volume | Critical |
| Cardinal Stone Partners investor database (accessed via Sterling integration) | Unknown volume | High |
| Encryption key material (plaintext, obtained from JavaScript files) | Obtained | Critical |

## 3. TECHNICAL ANALYSIS

### 3.1 Initial Access — CVE-2025-55182

CVE-2025-55182 is a remote code execution vulnerability affecting the web application framework deployed on Sterling Bank's pilot server. With a CVSS base score of 10.0, the vulnerability is rated critical across all three impact categories — confidentiality, integrity, and availability — and requires no authentication and no user interaction to exploit.

The vulnerability had been publicly disclosed and a patch made available prior to the incident date. The bank's failure to apply the patch to an internet-facing server constitutes a failure of the Vulnerability Management control domain under both ISO 27001 (Annex A, Control 8.8) and the CBN Cybersecurity Framework (Section 3.2.4).

Exploitation was conducted using Metasploit Framework — an open-source penetration testing tool freely available to any individual. No proprietary attack tooling was required. The attack required no insider knowledge of the bank's environment.

**Assessment of exploitability:** The vulnerability required an internet connection and approximately five minutes of effort from an attacker with basic technical proficiency. This is the lowest possible barrier to initial access for an organisation of this size.

### 3.2 Persistence and Command-and-Control

Following initial access, the attacker deployed Sliver — an open-source adversary simulation framework — to establish a persistent, encrypted command-and-control channel. Sliver communicates over standard encrypted protocols (HTTPS/mTLS), making it difficult to distinguish from legitimate traffic without deep packet inspection or behavioural analysis.

The Silver implant was first checked in on March 22, four days after initial access, confirming the attacker was operating deliberately and patiently rather than opportunistically rushing exfiltration.

**Control failure:** The absence of any alert during a four-day window of malicious activity indicates that Sterling Bank lacked either a functioning endpoint detection and response (EDR) solution on the affected infrastructure, or a network traffic analysis capability capable of identifying C2 communication patterns.

### 3.3 Discovery and Lateral Movement

From inside the compromised server, the attacker used standard network scanning tools to enumerate 168 open internal services within Sterling Bank's Kubernetes cluster — the container orchestration platform managing the bank's application workloads.

The discovery phase revealed a critical misconfiguration: internal services were accessible from the compromised test server without network segmentation controls separating the test environment from production systems.

The attacker then queried Sterling Bank's Temenos T24 core banking platform directly, pulling customer account data, transaction histories, and credit bureau records via the bank's own Credit Risk Central integration.

### 3.4 Cryptographic Key Exposure

During the discovery phase, the attacker searched the compromised environment for credential material using standard keywords: *password, secret, encrypt*. Encryption keys were found embedded in JavaScript files in plaintext — client-side code that is, by design, accessible to any process with access to the application environment.

**Industry standard:** Cryptographic keys must never be stored in client-side code, committed to version control, or embedded in application files without encryption at rest. The practice of hardcoding secrets in JavaScript files is explicitly prohibited by the OWASP Secure Coding Practices Guide, the CBN Application Security Guidelines, and basic software development security standards.

The exposure of these keys creates the theoretical capability to decrypt any data that was encrypted using them — including customer financial data in transit and at rest.

### 3.5 Lateral Movement to Remita

The breach did not stop at Sterling Bank. The attacker identified a trusted network connection between Sterling Bank's infrastructure and Remita — the payment platform processing salaries for all federal civil servants and managing government financial transactions across Nigeria's public sector.

This connection existed because Sterling Bank is an integrated partner institution within Remita's network — a legitimate business relationship. The attacker exploited that trust relationship to pivot from a compromised test server at a single bank into the national payment infrastructure.

**The attack did not involve any exploitation of Remita's own perimeter defences.** The attacker arrived as a trusted connection from Sterling Bank. Remita's security controls — whatever their quality — were rendered irrelevant by the upstream compromise.

## ATTACK PATH (MITRE ATT&CK MAPPING)

| Stage | ATT&CK Tactic | Technique | Description |
| --- | --- | --- | --- |
| Initial Access | Initial Access | T1190 — Exploit Public-Facing Application | CVE-2025-55182 exploitation via Metasploit on enf-pilot.sterling.ng |
| Execution | Execution | T1059 — Command and Scripting Interpreter | Shell execution via web application exploit |
| Persistence | Persistence | T1543 — Create or Modify System Process | Sliver C2 implant deployed for persistent access |
| Command & Control | Command and Control | T1071.001 — Web Protocols | Sliver C2 over HTTPS/mTLS |
| Discovery | Discovery | T1046 — Network Service Discovery | Enumeration of 168 internal Kubernetes services |
| Discovery | Discovery | T1083 — File and Directory Discovery | Search for credential material (password, secret, encrypt) |
| Credential Access | Credential Access | T1552.001 — Credentials in Files | Encryption keys obtained from plaintext JavaScript files |
| Collection | Collection | T1213 — Data from Information Repositories | Temenos T24 core banking queries; employee database access |
| Exfiltration | Exfiltration | T1041 — Exfiltration Over C2 Channel | Data transmitted via established Sliver channel |
| Lateral Movement | Lateral Movement | T1199 — Trusted Relationship | Pivot to Remita via Sterling Bank's trusted integration |

## Financial Impact

| Category | Estimated Impact | Basis |
| --- | --- | --- |
| Regulatory fines (NDPC, CBN) | ₦50M–₦500M+ | NDPA 2023 penalty provisions; CBN enforcement precedent |
| Litigation exposure (class action) | Unquantified | 900,000 affected customers; precedent from international breach litigation |
| Ransom demand | €250,000 (~₦450M) | Confirmed by ByteToBreach interview |
| Reputational impact on deposits | Unquantified | Customer trust erosion; competitor acquisition of displaced customers |
| Incident response and remediation costs | ₦200M–₦500M estimated | Forensics, legal, system rebuild, key rotation |
| **Estimated minimum direct exposure** | **₦700M–₦1.5B** |  |

### 5.2 Regulatory Impact

**Nigeria Data Protection Act 2023 (NDPA):**
Section 40 of the NDPA requires data controllers to notify the NDPC and affected data subjects of a breach likely to result in high risk to individuals within 72 hours of becoming aware. Sterling Bank's failure to issue any customer notification — despite confirmed awareness through the ransom communication — constitutes a direct, provable violation. NDPA maximum penalties apply to the processing turnover of the organisation.

**CBN Cybersecurity Framework 2022:**
The framework mandates minimum baseline controls for all licensed deposit money banks covering vulnerability management, patch management, security monitoring, and incident response. The findings of this assessment indicate material non-compliance across multiple domains.

**Cybercrimes Act 2024:**
While primarily relevant to the attacker, the bank's conduct — specifically the decision to negotiate a ransom privately without regulatory disclosure — may attract scrutiny under provisions relating to the handling of cyber incidents by financial institutions.

### 5.3 Operational Impact

The Remita pivot extends the impact beyond Sterling Bank's own customers and operations. The compromise of a CBN payment channel integration point potentially affects:

- Federal civil servant salary disbursements
- Government-to-business payment flows
- Inter-institutional trust in the NIBSS settlement infrastructure
- Continental payment integrity via the PAPSS integration

The HSM key exposure — if the keys are authentic and have not been rotated — represents an ongoing systemic risk to Nigerian inter-bank settlement rather than a contained, historical event.

## 6. RISK ASSESSMENT

**Risk Rating Methodology:**

Likelihood (1–5) × Impact (1–5) = Risk Score

1–5 Low · 6–12 Medium · 13–19 High · 20–25 Critical

| Risk ID | Risk Description | Likelihood | Impact | Score | Level |
| --- | --- | --- | --- | --- | --- |
| R-01 | Unpatched critical CVE on internet-facing infrastructure exploited | 5 | 5 | 25 | Critical |
| R-02 | Absence of detection capability allowing 9-day dwell time | 5 | 5 | 25 | Critical |
| R-03 | Cryptographic key material stored in plaintext application code | 4 | 5 | 20 | Critical |
| R-04 | Insufficient network segmentation between test and production environments | 5 | 4 | 20 | Critical |
| R-05 | Lateral movement to third-party infrastructure via trusted connection | 4 | 5 | 20 | Critical |
| R-06 | Non-disclosure of breach to affected customers and regulators | 5 | 4 | 20 | Critical |
| R-07 | Potential ongoing validity of exposed HSM key material | 3 | 5 | 15 | High |
| R-08 | Customer impersonation and identity fraud using exfiltrated data | 5 | 4 | 20 | Critical |
| R-09 | Insider threat enabled by full employee record exposure | 3 | 4 | 12 | Medium |
| R-10 | Third-party vendor risk (Cardinal Stone integration) | 3 | 4 | 12 | Medium |
| R-11 | Reputational damage leading to deposit outflow | 4 | 4 | 16 | High |
| R-12 | Regulatory sanction and licence risk | 4 | 5 | 20 | Critical |


##  Immediate Actions (0–30 Days)

These controls address active, unresolved risk from the incident.

| Priority | Control | NIST Function | CBN Framework Reference | Owner |
| --- | --- | --- | --- | --- |
| P1 | Rotate all HSM keys exposed or potentially exposed during the breach. Coordinate with NIBSS and affected member banks. | RESPOND | Section 4.3 — Key Management | CISO + Operations |
| P1 | Issue customer breach notification in compliance with NDPA 2023 Section 40. Notify NDPC formally. | RESPOND | Section 5.1 — Incident Reporting | Legal + CISO |
| P1 | Conduct full audit of all internet-facing servers for unpatched critical CVEs. Patch or decommission within 72 hours. | PROTECT | Section 3.2.4 — Patch Management | IT Security |
| P1 | Revoke all credentials, tokens, and API keys that were accessible from the compromised environment. | RESPOND | Section 3.3 — Access Management | IT Security |
| P2 | Engage external forensic firm to confirm full scope of breach and validate containment. | RESPOND | Section 5.2 — Forensic Investigation | CISO |
| P2 | Decommission or fully isolate the pilot/test server environment pending architecture review. | PROTECT | Section 3.2 — Network Security | IT Operations |

## Short-Term Controls (30–90 Days)

| Control | NIST Function | ISO 27001 Reference | Description |
| --- | --- | --- | --- |
| Deploy EDR on all servers — production and non-production | DETECT | A.8.7 | Endpoint detection capable of identifying C2 communication patterns and anomalous process execution |
| Implement network segmentation between test, staging, and production environments | PROTECT | A.8.22 | Zero trust network architecture — test servers must not have routing access to production Kubernetes clusters |
| Remove all secrets from source code and client-side files | PROTECT | A.8.24 | Implement secrets management (HashiCorp Vault or equivalent). Automated scanning of all repositories for hardcoded credentials. |
| Implement vulnerability management programme with SLA enforcement | IDENTIFY | A.8.8 | Critical CVEs (CVSS 9.0+): patch within 24 hours. High (CVSS 7.0–8.9): 7 days. Medium: 30 days. Automated scanning cadence minimum weekly. |
| Deploy SIEM with behavioural baseline alerting | DETECT | A.8.16 | Alert thresholds: unusual login times, bulk data queries, new outbound connections to unlisted IPs, administrative account creation |
| Review and restrict all third-party integration permissions | PROTECT | A.5.19 | Audit all trusted network connections (Remita, Cardinal Stone, NIBSS). Apply principle of least privilege to each integration. |

## Medium-Term Controls (90–180 Days)

| Control | NIST Function | ISO 27001 Reference | Description |
| --- | --- | --- | --- |
| Establish formal Vulnerability Management Policy | IDENTIFY | A.5.1 | Board-approved policy with defined ownership, SLAs, exception process, and reporting to risk committee |
| Implement Data Loss Prevention (DLP) | PROTECT | A.8.12 | Automated monitoring for bulk data movement from core banking systems. Alert on queries exceeding normal parameters. |
| Conduct red team exercise on internet-facing infrastructure | IDENTIFY | A.8.8 | Authorised adversary simulation to validate effectiveness of new controls before certification |
| Develop and test Incident Response Plan | RESPOND | A.5.26 | Documented plan including regulatory notification timelines, communication protocols, and ransom policy (no payment without regulatory disclosure) |
| CBN Cybersecurity Framework gap assessment | IDENTIFY | A.5.35 | Formal self-assessment against all 23 domains of the CBN framework. Commission external review prior to next regulatory examination. |

*This assessment is based on information publicly reported by Techloy, the Web Security Lab / Security Intelligence Substack, The Guardian Nigeria, BusinessDay, and related sources between April 8–28, 2026. It does not represent a commissioned engagement with Sterling Bank and does not rely on privileged or non-public information. All findings are derived from published breach disclosures and mapped to applicable regulatory frameworks for educational and professional development purposes.*
