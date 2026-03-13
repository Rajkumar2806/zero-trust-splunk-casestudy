# Zero Trust Security Architecture — Splunk Implementation

[![NIST SP 800-207](https://img.shields.io/badge/NIST-SP%20800--207-blue)](https://csrc.nist.gov/publications/detail/sp/800/207/final)
[![CISA ZTMM v2.0](https://img.shields.io/badge/CISA-ZTMM%20v2.0-green)](https://www.cisa.gov/zero-trust-maturity-model)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)](https://attack.mitre.org/)
[![Splunk](https://img.shields.io/badge/Platform-Splunk-orange)](https://www.splunk.com/)
[![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)]()

> **Graduate Case Study** | M.S. Applied Information Technology | Towson University | Spring 2026
> **Author:** Rajkumar Vadthyavath | **Advisor:** Prof. Raza Hasan | **Course:** AIT 730

---

## What This Project Does

This repository contains a **fully functional Zero Trust Security monitoring prototype** built in Splunk, designed to detect and visualize seven critical governance gaps in a mid-size hybrid cloud enterprise.

The prototype ingests **147 synthetic security events** across 8 data source types, runs real-time SPL queries, and renders a 7-panel executive dashboard — demonstrating how a NIST SP 800-207 aligned Zero Trust Architecture improves governance, risk visibility, and access control over a traditional perimeter-based security model.

**Research Question:**
> *How does adopting a NIST SP 800-207 Zero Trust Security Architecture improve enterprise governance, risk management, and system control compared to a traditional perimeter-based model?*

---

## Live Dashboard — 7 Panels

| Panel | Title | CISA ZTMM Pillar | Governance Gap Addressed |
|-------|-------|-----------------|--------------------------|
| 1 | Identity Risk & MFA Detection | Identity | 4 users signing in without MFA — mbrown risk score 85/100 |
| 2 | Device Compliance Status | Devices | 5 non-compliant devices — DV007 flagged CRITICAL (non-compliant + unhealthy EDR) |
| 3 | Network Segmentation — Lateral Movement | Networks | RDP (T1021.001) and SSH (T1021.004) blocked at micro-segmentation boundary |
| 4 | Policy Decision Audit Log | Governance | Every allow/deny with policy_id and reason — full audit trail |
| 5 | Data Access Governance — Unapproved Exports | Data | 6 unapproved exports of PII, FinancialReports, HRRecords, CustomerData |
| 6 | Privileged Access — Unapproved Admin Sessions | Identity (PAM) | 6 privileged sessions with no approved ticket including ProdDB and FinanceDB |
| 7 | Anomaly Monitoring — Impossible Travel | Visibility & Analytics | 1 geo_impossible flag (rajkumar — Dublin), 5 multi_country logins detected |

---

## Cross-Pillar Risk Correlation

The dashboard includes a cross-pillar correlation panel identifying users flagged simultaneously across multiple Zero Trust pillars — the highest risk signal in the entire prototype.

| User | Pillars Flagged | Risk Level | Pillars Hit |
|------|----------------|------------|-------------|
| mbrown | 5 of 5 | CRITICAL | Identity, Device, Network, Data, Privileged Access |
| rjones | 5 of 5 | CRITICAL | Identity, Device, Network, Data, Privileged Access |
| slee | 5 of 5 | CRITICAL | Identity, Device, Network, Data, Privileged Access |
| tdavis | 5 of 5 | CRITICAL | Identity, Device, Network, Data, Privileged Access |
| rajkumar | 4 of 5 | HIGH | Data, Policy, Privileged Access, Anomaly |
| amiller | 4 of 5 | HIGH | Data, Identity, Policy, Privileged Access |

---

## Key Findings

- **4 users** consistently bypassing MFA — mbrown highest risk score 85/100
- **5 non-compliant devices** — DV007 CRITICAL (non-compliant + unhealthy EDR simultaneously)
- **2 lateral movement attempts blocked** — RDP (T1021.001) and SSH (T1021.004)
- **6 unapproved data exports** — PII and FinancialReports (GDPR/HIPAA exposure)
- **6 unapproved privileged sessions** — no change ticket, including ProdDB and FinanceDB
- **1 geo_impossible travel** (rajkumar — Dublin) + **5 multi_country logins**
- **4 users flagged CRITICAL** across all 5 Zero Trust pillars simultaneously
- **Policy engine produced 6 explicit denials** with auditable policy_id and reason codes

---

## Repository Structure

```
zero-trust-splunk-casestudy/
|
|-- README.md
|-- requirements.txt
|
|-- docs/
|   |-- ZTA_Case_Study_Proposal.docx
|   |-- Seven_UseCase_Splunk_Implementation.docx
|   +-- Splunk_Demo_Guide.docx
|
|-- splunk/
|   |-- queries/
|   |   +-- Splunk_Queries_Reference.txt
|   +-- dashboard/
|       +-- zero_trust_dashboard.xml
|
|-- data/
|   |-- csv/
|   |   +-- network_access.csv
|   +-- logs/
|       |-- policy_decision.log
|       |-- privileged_access.log
|       |-- splunk_activity.log
|       |-- splunk_data_access.log
|       |-- splunk_device.log
|       |-- splunk_identity.log
|       |-- splunk_network.log
|       |-- splunk_policy.log
|       |-- splunk_privileged.log
|       +-- user_activity.log
|
+-- src/
    +-- data_generator.py
```

---

## Frameworks Applied

| Framework | How It Is Applied |
|-----------|------------------|
| **NIST SP 800-207** | Core ZTA design — Policy Decision Point, Policy Enforcement Point, continuous per-session validation |
| **CISA ZTMM v2.0** | All 5 pillars mapped to dashboard panels — maturity from Traditional to Optimal |
| **NIST SP 800-53 Rev 5** | Control families AC, IA, SC, SI, AU, CA mapped to each governance gap |
| **MITRE ATT&CK** | T1021.001 (RDP) and T1021.004 (SSH) lateral movement detected in Panel 3 |
| **STRIDE** | Threat and risk analysis phase — attack vector identification |
| **DoD ZT Reference Architecture v2.0** | Supporting reference for enterprise ZT deployment |

---

## How to Run This in Splunk

### Step 1 — Create the Index
```
Settings > Indexes > New Index > Name: zt
```

### Step 2 — Upload All Data Files
```
Settings > Add Data > Upload
Upload all files from: data/logs/ and data/csv/
Set index = zt for every file
```

### Step 3 — Import the Dashboard
```
Dashboards > Create New Dashboard > Import XML
Paste contents of: splunk/dashboard/zero_trust_dashboard.xml
```

### Step 4 — Verify All Data Loaded
```spl
index=zt
| stats count by source
| rename source as "Data Source", count as "Events Indexed"
```
Expected: 147 total events across 8 source types

### Step 5 — Run the Executive Summary Query
```spl
index=zt
| eval gap=case(
    source="identity_signin.csv" AND mfa_used="false",  "Gap 1: No MFA",
    source="device_posture.csv"  AND compliant="false", "Gap 2: Non-Compliant Device",
    source="network_access.csv"  AND outcome="deny",    "Gap 3: Blocked Lateral Movement",
    source="policy_decision.log" AND decision="deny",   "Gap 4: Policy Denial",
    source="data_access.log"     AND export="true"
                                 AND approved="no",     "Gap 5: Unapproved Export",
    source="privileged_access.log" AND approved="no",  "Gap 6: Unapproved Privilege",
    source="user_activity.log"   AND anomaly!="none",   "Gap 7: Anomaly Detected")
| where isnotnull(gap)
| stats count by gap
| sort gap
```

---

## Project Timeline

| Phase | Activity | Status |
|-------|----------|--------|
| 1 | Enterprise context and governance baseline | Complete |
| 2 | Threat and risk analysis (STRIDE + MITRE ATT&CK) | Complete |
| 3 | Zero Trust architecture and policy model design | Complete |
| 4 | Splunk prototype — 7-panel dashboard | Complete |
| 5 | Comparative evaluation (legacy vs ZT model) | In Progress |
| 6 | Final report and documentation | In Progress |
| 7 | Presentation preparation | Upcoming |

---

## Tools and Technologies

`Splunk Enterprise` `SPL` `Python 3` `NIST SP 800-207` `CISA ZTMM v2.0`
`NIST SP 800-53 Rev 5` `MITRE ATT&CK` `STRIDE` `Microsoft Azure / Entra ID`
`Conditional Access` `RBAC` `PAM` `ISO 27001`

---

## References

- NIST. (2020). *Zero Trust Architecture* — SP 800-207
- CISA. (2023). *Zero Trust Maturity Model Version 2.0*
- DoD. (2022). *Zero Trust Reference Architecture Version 2.0*
- Microsoft. (2023). *Zero Trust Deployment Plan for Microsoft 365*
- AWS. (2023). *AWS Well-Architected Framework — Security Pillar*

---

## Author

**Rajkumar Vadthyavath**
M.S. Applied Information Technology | Towson University | GPA: 3.90 / 4.0
[LinkedIn](https://linkedin.com/in/raj-kumar28) | [GitHub](https://github.com/Rajkumar2806)

**Certifications:** AZ-104 · CompTIA Security+ (SY0-701) · Splunk Core Certified Power User · (ISC)² CC

---

> All data in this repository is entirely synthetic. No real organizational data is used.
> This project is actively in progress — updated regularly as the case study develops.
