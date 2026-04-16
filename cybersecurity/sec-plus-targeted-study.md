---
layout: default
title: "Security+ (SY0-701) Targeted Study Guide"
date: 2026-04-16
category: cybersecurity
tags: [security+, comptia, certification, study-guide, GRC, security-architecture]
---

# Security+ (SY0-701) Targeted Study Guide

**Date:** 2026-04-16
**Baseline:** 79% on first practice exam (pass = 750/900 = ~83%)
**Gap:** ~21% — need to close ~4-5 percentage points
**Method:** Target the highest-weighted domains that are hardest for self-taught practitioners

---

## SY0-701 Domain Weights

| Domain | Weight | Self-taught Difficulty | Priority |
|--------|--------|----------------------|----------|
| 1. General Security Concepts | 12% | Low — most of this is intuitive | 4th |
| 2. Threats, Attacks & Vulnerabilities | 22% | Medium — you know the tech, exam frames it differently | 3rd |
| 3. Security Architecture | 18% | **High** — formal models, frameworks, encryption schemes | 2nd |
| 4. Security Operations | 28% | Medium — incident response process is standardized | 5th |
| 5. Governance, Risk & Compliance | 15% | **High** — policy/risk frameworks are NOT intuitive | 1st |

**Strategy:** GRC (15%) + Security Architecture (18%) = 33% of the exam and the two domains most likely to account for the gap. A self-taught practitioner who knows how attacks work often loses points on "which framework applies to this scenario" questions.

---

## Domain 5: Governance, Risk & Compliance (15%)

### 5 Key Concepts

1. **Risk treatment options** — Accept, Mitigate, Transfer, Avoid. The exam loves scenario questions: "A company purchases cybersecurity insurance. Which risk treatment?" → Transfer. "A company decides not to enter a new market due to security concerns." → Avoid.

2. **Quantitative risk analysis** — ALE = SLE × ARO. SLE = AV × EF. Know the formula cold. "Asset value $100K, exposure factor 40%, annualized rate of occurrence 0.3. What's the ALE?" → ($100K × 0.4) × 0.3 = $12,000.

3. **Control types** — Preventive, Detective, Corrective, Deterrent, Compensating. The exam tests your ability to CLASSIFY controls, not just name them. "Security cameras are what type?" → Detective. "Security guards with guns?" → Deterrent (not preventive — they don't stop the attack, they discourage it). "A backup generator when the UPS fails?" → Compensating.

4. **Compliance frameworks** — SOC 2 Type I vs Type II, PCI DSS (4 levels based on transaction volume), HIPAA (PHI, covered entities, business associates), SOX (financial controls for public companies), FISMA (federal systems), GDPR (EU personal data, right to be forgotten, DPO requirement). Know which framework applies to which industry scenario.

5. **Security policies hierarchy** — Policies → Standards → Procedures → Guidelines. Policies are mandatory and broad. Standards are mandatory and specific. Procedures are step-by-step mandatory. Guidelines are optional recommendations.

### 3 Practice Scenarios

**Q1:** A hospital's IT team implements encryption on all hard drives containing patient records. Under HIPAA, what type of safeguard is this?
- A) Administrative
- B) Physical
- C) Technical
- D) Organizational

**Answer:** C) Technical. HIPAA defines three safeguard types. Encryption is a technical safeguard. Administrative = policies, training, risk assessments. Physical = locks, cameras, badge access.

**Q2:** A company values a server at $50,000. A flood would destroy 60% of its value. Floods occur in the area about once every 5 years. What is the ALE?
- A) $6,000
- B) $10,000
- C) $30,000
- D) $12,000

**Answer:** A) $6,000. SLE = $50,000 × 0.6 = $30,000. ARO = 1/5 = 0.2. ALE = $30,000 × 0.2 = $6,000.

**Q3:** An organization subject to PCI DSS processes 50,000 credit card transactions per year via a single payment application. Which PCI DSS level applies?
- A) Level 1 (>6M transactions)
- B) Level 2 (1M–6M)
- C) Level 3 (20K–1M)
- D) Level 4 (<20K)

**Answer:** C) Level 3. 50,000 falls in the 20,000–1,000,000 range.

### Gotcha

**The "compensating control" trap.** The exam loves to ask about compensating controls, and self-taught people often confuse them with preventive controls. A compensating control doesn't prevent the threat — it provides an alternative way to achieve the same security objective when the primary control isn't feasible. Example: "The organization can't implement MFA due to legacy systems. They instead require longer passwords + account lockout after 3 failed attempts." That's compensating, not preventive. If the question says "due to [constraint], they instead use [alternative]", it's almost always compensating.

---

## Domain 3: Security Architecture (18%)

### 5 Key Concepts

1. **Zero Trust architecture** — "Never trust, always verify." Key principles: verify explicitly, use least-privilege access, assume breach. Every request is authenticated and authorized regardless of source network. No implicit trust based on network location.

2. **Defense in depth** — Layered security with multiple control types. If one layer fails, others still protect. The exam tests which layer a control belongs to: perimeter (firewall, WAF), network (IDS/IPS, segmentation), host (antivirus, HIDS), application (input validation, session management), data (encryption, DLP).

3. **Encryption schemes** — Symmetric (AES, DES, 3DES) vs Asymmetric (RSA, ECC, Diffie-Hellman). Know the use cases: symmetric for bulk data (speed), asymmetric for key exchange and digital signatures. Hybrid encryption: asymmetric to exchange a symmetric session key, then symmetric for the data.

4. **PKI components** — CA (Certificate Authority), RA (Registration Authority), CRL (Certificate Revocation List), OCSP (Online Certificate Status Protocol). Know the chain: Root CA → Intermediate CA → End-entity certificate. OCSP is the real-time alternative to CRL (which is a published list). The exam asks "how do you check if a certificate is still valid without downloading a CRL?" → OCSP.

5. **Authentication protocols** — RADIUS (UDP, combines auth+auth, used for network access), TACACS+ (TCP, separates auth/auth/accounting, used for admin access), LDAP (directory-based auth, used in Active Directory environments), Kerberos (ticket-based, SSO, used in Windows domains). Know which protocol uses which transport and which use case.

### 3 Practice Scenarios

**Q1:** A network architect designs a system where internal users must authenticate to access any resource, even within the corporate network. No resource is accessible based solely on network location. Which security model is this?
- A) Defense in depth
- B) Zero Trust
- C) Least privilege
- D) Separation of duties

**Answer:** B) Zero Trust. The key phrase is "no resource is accessible based solely on network location." Defense in depth is about layers, not about removing implicit trust.

**Q2:** A web server needs to verify that a client certificate has not been revoked, but downloading the full CRL is too slow. Which protocol should it use instead?
- A) LDAP
- B) OCSP
- C) Kerberos
- D) SCP

**Answer:** B) OCSP. OCSP provides real-time certificate status checking without downloading the full CRL.

**Q3:** An organization uses AES-256 to encrypt database records and RSA to encrypt the AES key for transmission. What is this approach called?
- A) Symmetric encryption
- B) Asymmetric encryption
- C) Hybrid encryption
- D) Hashing

**Answer:** C) Hybrid encryption. Using asymmetric encryption to protect a symmetric key, then using the symmetric key for bulk data.

### Gotcha

**The "Kerberos vs RADIUS" trap.** Both handle authentication, but the exam tests the transport and use case difference. Kerberos = TCP, ticket-based, Windows domain SSO. RADIUS = UDP, combines authentication and authorization, network device access (VPN, WiFi 802.1X). The trap: "Which protocol should a network admin use to authenticate VPN users?" → RADIUS, not Kerberos. "Which protocol provides SSO in an Active Directory environment?" → Kerberos, not RADIUS. The transport matters because the exam sometimes asks about reliability — UDP (RADIUS) can drop packets; TCP (Kerberos) is reliable.

---

## Domain 2: Threats, Attacks & Vulnerabilities (22%)

### 5 Key Concepts

1. **Threat actor categories** — Nation-state (APT, well-funded, targeted), Organized crime (financial motive, sophisticated), Hacktivist (ideological motive, varying skill), Insider threat (privileged access, knowledge of systems), Script kiddie (low skill, uses existing tools). The exam tests MOTIVE and CAPABILITY classification.

2. **Attack vectors** — Supply chain (SolarWinds example), Social engineering (phishing, pretexting, baiting), Third-party (vendor compromise), Advanced persistent threat (long-term access, lateral movement). Know which vector each scenario describes.

3. **Vulnerability types** — Zero-day (no patch exists), Known (patch available but not applied), Misconfiguration (default settings, open ports), Logic flaw (design error, not implementation error). The exam tests which type is described by a scenario.

4. **Threat modeling frameworks** — STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) — Microsoft's model. PASTA (Process for Attack Simulation and Threat Analysis) — 7-step risk-centric model. Know which is which and when to use each.

5. **Attack surface reduction** — Closing unused ports, removing default credentials, disabling unnecessary services, implementing network segmentation. The exam tests "which of these reduces the attack surface" vs "which of these detects attacks" (IDS) vs "which responds to attacks" (IPS).

### 3 Practice Scenarios

**Q1:** A security team discovers that an attacker has been maintaining persistent access to their network for 6 months, moving laterally and exfiltrating data slowly. What type of threat is this?
- A) Ransomware
- B) APT
- C) DDoS
- D) Supply chain attack

**Answer:** B) APT (Advanced Persistent Threat). Key indicators: persistent access (6 months), lateral movement, slow exfiltration. This is the definition of an APT.

**Q2:** Using the STRIDE model, which threat category applies when an attacker modifies a database record without authorization?
- A) Spoofing
- B) Tampering
- C) Repudiation
- D) Information disclosure

**Answer:** B) Tampering. Unauthorized modification of data is tampering. Spoofing = pretending to be someone else. Repudiation = denying an action. Information disclosure = reading data you shouldn't.

**Q3:** Which of the following is a misconfiguration vulnerability rather than a known vulnerability?
- A) A buffer overflow in OpenSSL that has a CVE assigned
- B) An admin console with default credentials still set
- C) A zero-day in Windows that Microsoft hasn't patched yet
- D) A SQL injection in a custom web application

**Answer:** B) Default credentials is a misconfiguration. A = known vulnerability (CVE). C = zero-day. D = logic/implementation flaw.

### Gotcha

**The "zero-day vs known" trap.** A zero-day is NOT just any unpatched vulnerability. It specifically means **no patch exists yet** (the vendor doesn't know about it or hasn't released a fix). If a CVE exists and a patch is available but the organization hasn't applied it, that's a KNOWN vulnerability, not a zero-day. The exam will phrase it as: "A vulnerability was disclosed last month and a patch was released two weeks ago. The organization has not applied the patch." → Known vulnerability. "A researcher discovered a vulnerability yesterday and the vendor has not yet released a patch." → Zero-day.

---

## Study Priority Summary

| This week | Next week | Before exam |
|-----------|-----------|-------------|
| GRC: risk formulas, control types, compliance frameworks | Security Architecture: Zero Trust, PKI, encryption | Full practice exam #3 |
| GRC: policy hierarchy, SOC 2 vs PCI vs HIPAA scenarios | Threats: STRIDE, PASTA, threat actor classification | Review all wrong answers from exams 1-3 |
| Security Architecture: hybrid encryption, Kerberos vs RADIUS | Threats: attack vector identification | |

**Target:** Close 4-5 percentage points by mastering GRC scenarios and Security Architecture classification questions. These are the "exam language" questions where self-taught practitioners lose points — you know the concepts but the exam frames them in unfamiliar policy/framework language.