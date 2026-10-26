# Reporting & Documentation

## Penetration Test Report Template

### 1. Executive Summary
```markdown
# EXECUTIVE SUMMARY

## Overview
Penetration testing was conducted on [Date] to evaluate the wireless
network security of [Company Name].

## Key Findings
- **Critical:** [X] findings
- **High:** [X] findings
- **Medium:** [X] findings
- **Low:** [X] findings

## Business Risk
[Summary of business impact]

## Recommendations
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]
```

### 2. Table of Contents
```
1. Executive Summary
2. Scope & Methodology
3. Findings Summary
4. Detailed Findings
   4.1 [Finding Title]
       - Description
       - Impact
       - Evidence
       - Recommendation
   4.2 [Finding Title]
   ...
5. Technical Details
6. Tools & Techniques
7. Conclusion
8. Appendices
   - A. Raw Data
   - B. Tool Output
   - C. CVSS Scoring
```

### 3. Detailed Finding Template
```markdown
## [VULN-ID]: [Vulnerability Title]

**Severity:** Critical/High/Medium/Low/Informational
**CVSS Score:** 9.8 (Critical)
**CWE:** CWE-XXX
**Affected:** [SSID/BSSID/Host]

### Description
[Technical description of the vulnerability]

### Impact
[Impact if exploited]

### Evidence
[Screenshot/log/command output]

### Proof of Concept
[Reproduction steps]

### Recommendation
[Suggested fix]

### References
- [CVE-XXXX-XXXX]
- [Link]
```

---

## CVSS Scoring for Wireless

### Metric Groups

**Base Score:**
| Metric | Values | Weight |
|--------|--------|--------|
| Attack Vector (AV) | Network (N), Adjacent (A), Local (L), Physical (P) | Network=0.85 |
| Attack Complexity (AC) | Low (L), High (H) | Low=0.77 |
| Privileges Required (PR) | None (N), Low (L), High (H) | None=0.85 |
| User Interaction (UI) | None (N), Required (R) | None=0.85 |
| Scope (S) | Unchanged (U), Changed (C) | - |
| Confidentiality (C) | None (N), Low (L), High (H) | High=0.56 |
| Integrity (I) | None (N), Low (L), High (H) | High=0.56 |
| Availability (A) | None (N), Low (L), High (H) | High=0.56 |

### Scoring Examples

**Evil Twin + Credential Harvest:**
```json
{
  "CVE": "N/A",
  "Title": "Evil Twin Attack - Credential Harvesting",
  "CVSS": {
    "AV": "A",
    "AC": "L",
    "PR": "N",
    "UI": "R",
    "S": "C",
    "C": "H",
    "I": "H",
    "A": "N",
    "Score": 8.8,
    "Severity": "High"
  }
}
```

**Deauth DoS:**
```json
{
  "Title": "Deauthentication DoS",
  "CVSS": {
    "AV": "A",
    "AC": "L",
    "PR": "N",
    "UI": "N",
    "S": "U",
    "C": "N",
    "I": "N",
    "A": "H",
    "Score": 6.5,
    "Severity": "Medium"
  }
}
```

---

## Severity Classification

| Severity | CVSS | Description | SLA Response |
|----------|------|-------------|--------------|
| **Critical** | 9.0-10.0 | Immediate exploit, full compromise | 24 hours |
| **High** | 7.0-8.9 | Significant impact, easy exploit | 7 days |
| **Medium** | 4.0-6.9 | Moderate impact | 30 days |
| **Low** | 0.1-3.9 | Limited impact | 90 days |
| **Info** | 0.0 | No direct impact | N/A |

---

## Remediation Matrix

| Finding | Priority | Effort | Impact | Timeline |
|---------|----------|--------|--------|----------|
| WEP enabled | 🔴 Critical | Low | High | Immediate |
| No PMF | 🟡 Medium | Low | Medium | 30 days |
| Rogue AP | 🔴 Critical | Medium | High | 24 hours |
| Weak PSK | 🟠 High | Low | High | 7 days |
| Open management | 🔴 Critical | Low | High | Immediate |

---

## Report Checklist

### Content
- [ ] Executive summary (non-technical)
- [ ] Scope & methodology
- [ ] Findings with severity rating
- [ ] CVSS scoring for each finding
- [ ] Evidence (screenshot, log, command output)
- [ ] Proof of concept
- [ ] Remediation recommendations
- [ ] Risk matrix
- [ ] References

### Formatting
- [ ] Professional layout
- [ ] Consistent branding
- [ ] Table of contents
- [ ] Page numbers
- [ ] Version control
- [ ] Classification marking (Confidential)

### Delivery
- [ ] Encrypted delivery (PGP/S-MIME)
- [ ] Client review meeting
- [ ] Q&A session
- [ ] Executive presentation (optional)
- [ ] Follow-up assessment scheduled

---

## Reporting Tools

### Markdown → PDF
```bash
# Pandoc
pandoc report.md -o report.pdf --template=eisvogel

# With LaTeX
pandoc report.md -o report.pdf \
  -V geometry:margin=1in \
  --highlight-style=tango
```

### Dradis (Collaboration)
```bash
# https://dradis.com/
# Vulnerability management & reporting
```

### PlexTrac (Pentest Reporting)
- https://plextrac.com/
- Professional pentest reporting platform

### Faraday
```bash
# https://github.com/infobyte/faraday
# Collaborative penetration test & vulnerability management
```

---

## References

- **PTES Reporting:** http://www.pentest-standard.org/index.php/Reporting
- **CVSS v3.1:** https://www.first.org/cvss/v3.1/specification-document
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **SANS Report Template:** https://www.sans.org/white-papers/
