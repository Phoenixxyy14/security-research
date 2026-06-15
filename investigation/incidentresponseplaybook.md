## PLAYBOOK: External Actor Breach, Internet-Facing Infrastructure

Trigger: Alert indicating exploitation of an internet-facing system, anomalous C2 communication detected, or third-party notification of active breach.

### PHASE 1 — DETECTION AND INITIAL TRIAGE (0–2 Hours)

Objective: Confirm breach and activate response team.

| Step | Action | Owner | Tool/Method |
| --- | --- | --- | --- |
| 1.1 | Isolate alert — confirm whether event is false positive or confirmed incident | SOC Analyst | SIEM correlation |
| 1.2 | Escalate to Incident Commander (CISO or designate) | SOC Lead | — |
| 1.3 | Convene Incident Response Team: CISO, IT Security, Legal, Comms, relevant business owner | CISO | — |
| 1.4 | Preserve logs — snapshot all relevant system logs before any remediation | IT Security | SIEM, server logs |
| 1.5 | Assign incident severity rating: P1 (Critical) / P2 (High) / P3 (Medium) | Incident Commander | Severity matrix |
| 1.6 | Open incident log — timestamp every action from this point | IR Lead | Incident ticketing system |

### PHASE 2 — CONTAINMENT (2–12 Hours)

Objective: Prevent further access and lateral movement without destroying evidence.

| Step | Action | Owner |  |
| --- | --- | --- | --- |
| 2.1 | Isolate affected system(s) from network — do NOT power off (preserves memory artefacts) | IT Security |  |
| 2.2 | Block attacker IP addresses and C2 domains identified in logs at perimeter firewall | Network Security |  |
| 2.3 | Revoke all credentials, tokens, and API keys associated with the compromised system | IT Security |  |
| 2.4 | Identify and document all systems the compromised host had network access to | IT Security |  |
| 2.5 | Notify integration partners (Remita, NIBSS, Cardinal Stone, relevant third parties) of potential compromise | CISO + Legal |  |
| 2.6 | Brief executive leadership: CEO, CFO, Board Risk Committee Chair | CISO |  |

Do NOT: Reimage the affected system before forensic capture. Do not communicate breach details externally until Legal has reviewed messaging.

### PHASE 3 — INVESTIGATION AND SCOPE (12–72 Hours)

Objective: Establish full scope of compromise — what was accessed, what was exfiltrated, how the attacker moved.

| Step | Action | Owner |
| --- | --- | --- |
| 3.1 | Engage external forensic firm if internal capability is insufficient | CISO |
| 3.2 | Forensic image all affected systems | Forensics Lead |
| 3.3 | Reconstruct full attack timeline from log evidence | IR Team |
| 3.4 | Identify all data categories accessed or exfiltrated | IR Team + Data Protection Officer |
| 3.5 | Map affected data subjects — customer accounts, employee records, third-party data | DPO |
| 3.6 | Assess regulatory notification obligations — NDPC, CBN, NIBSS | Legal + DPO |
| 3.7 | Prepare initial incident report for CBN notification | Legal + CISO |

**Regulatory deadline:** NDPA 2023 Section 40 requires notification to NDPC within 72 hours of becoming aware of a breach. This clock runs from the moment internal awareness is confirmed, not from the moment forensics are complete.

### PHASE 4 — ERADICATION AND HARDENING (72 Hours–2 Weeks)

**Objective:** Remove attacker access, close the vulnerability, harden the environment.

| Step | Action | Owner |
| --- | --- | --- |
| 4.1 | Apply patch for exploited CVE across all affected and related systems | IT Security |
| 4.2 | Remove all malicious artefacts: C2 implants, backdoors, malicious scripts | IT Security |
| 4.3 | Rotate all cryptographic keys accessible from compromised environment | CISO + Operations |
| 4.4 | Audit all user accounts for unauthorised creation or privilege escalation | IT Security |
| 4.5 | Implement emergency network segmentation between test and production environments | Network Security |
| 4.6 | Conduct credential sweep across all systems the attacker had logical access to | IT Security |
| 4.7 | Validate eradication via external penetration test of affected environment | CISO |

### PHASE 5 — NOTIFICATION AND COMMUNICATIONS (Concurrent with Phase 3–4)

Objective: Meet legal obligations, communicate appropriately, do not make the reputational situation worse.

| Audience | Timeline | Channel | Key Messages |
| --- | --- | --- | --- |
| NDPC | Within 72 hours of awareness | Formal written notification | Breach scope, data categories, data subjects, steps taken |
| CBN | Within 24 hours for material incidents | CBN incident reporting portal | Nature of breach, regulatory exposure, containment steps |
| NIBSS and integration partners | Immediately upon confirmation | Direct contact | Potential HSM key exposure, integration security |
| Affected customers | As soon as scope is confirmed — no later than 7 days | SMS + Email + App notification | What happened, what data was affected, what to do |
| Media | After regulatory notifications are filed | Prepared statement only | Factual, no speculation, confirm investigation is underway |

On ransom: Under no circumstances should a ransom payment be made without prior notification to the CBN and legal counsel review. Paying a ransom without disclosure may constitute a regulatory violation and potentially a criminal matter. This must be explicit in the IR policy.

### PHASE 6 — RECOVERY AND POST-INCIDENT REVIEW (2–6 Weeks)

| Step | Action | Owner |
| --- | --- | --- |
| 6.1 | Restore affected systems from clean backups after validation | IT Operations |
| 6.2 | Monitor restored systems for 30 days for any signs of persistent access | SOC |
| 6.3 | Conduct post-incident review: what failed, why, what changes to make | CISO + IR Team |
| 6.4 | Update IR playbook based on lessons learned | IR Lead |
| 6.5 | Present findings and remediation plan to Board Risk Committee | CISO + CEO |
| 6.6 | Commission independent review of CBN Framework compliance posture | External auditor |
| 6.7 | File post-incident report with NDPC confirming remediation steps taken | Legal + DPO |

### APPENDIX — REGULATORY REFERENCE SUMMARY

| Regulation | Relevant Provision | Sterling Bank Exposure |
| --- | --- | --- |
| NDPA 2023 | Section 40 — Breach Notification (72-hour NDPC notification, data subject notification) | Confirmed violation — no notification issued |
| NDPA 2023 | Section 48 — Administrative penalties up to 2% of annual gross revenue or ₦10M | Material financial exposure |
| CBN Cybersecurity Framework 2022 | Section 3.2.4 — Patch Management | Unpatched CVSS 10.0 CVE on production network |
| CBN Cybersecurity Framework 2022 | Section 3.3 — Access Management | Privilege escalation undetected; no MFA evidence |
| CBN Cybersecurity Framework 2022 | Section 4.3 — Cryptographic Controls | Encryption keys in plaintext client-side code |
| CBN Cybersecurity Framework 2022 | Section 5.1 — Incident Management | No formal incident response executed |
| Cybercrimes Act 2024 | Section 19 — Critical National Infrastructure | Breach impact on Remita constitutes critical infrastructure incident |
| PCI DSS v4.0 | Requirement 6.3.3 — Security patches | Critical patches must be applied within 1 month |

*This assessment is based on information publicly reported by Techloy, the Web Security Lab / Security Intelligence Substack, The Guardian Nigeria, BusinessDay, and related sources between April 8–28, 2026. It does not represent a commissioned engagement with Sterling Bank and does not rely on privileged or non-public information. All findings are derived from published breach disclosures and mapped to applicable regulatory frameworks for educational and professional development purposes.*
