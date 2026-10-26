# Legal Framework & Ethics

## Legal Framework — Indonesia

### UU ITE (Indonesian Electronic Information and Transactions Law)

| Article | Provision | Penalty |
|---------|-----------|---------|
| **Article 30** | Unauthorized computer access | 8 years imprisonment + IDR 2B fine |
| **Article 31** | Interception / wiretapping | 10 years imprisonment + IDR 5B fine |
| **Article 32** | Altering data without authorization | 8 years imprisonment + IDR 2B fine |
| **Article 33** | Producing hacking tools | 10 years imprisonment + IDR 5B fine |

> **2024 Update:** Amendment to UU ITE (Law No. 1 of 2024) clarifies definitions and enforcement.

### KUHP (Indonesian Criminal Code)
- **Article 378:** Fraud (phishing)
- **Article 406:** Data destruction
- **Article 310-312:** Defamation

---

## Rules of Engagement (RoE)

### RoE Template

```markdown
# RULES OF ENGAGEMENT
## Wireless Penetration Testing

**Project:** [Project Name]
**Client:** [Company Name]
**Date:** [Date]
**Authorization:** [Authorization Provider Name]

## 1. SCOPE
- SSID in-scope: [SSID List]
- BSSID in-scope: [BSSID List]
- Location: [Location]
- Time window: [Start Date] - [End Date]
- Working hours: [Hours]

## 2. AUTHORIZATION
- Written authorization from: [Name, Title]
- Document reference: [Doc ID]
- Emergency contact: [Name, Phone]

## 3. RULES
✅ ALLOWED:
- Passive reconnaissance
- Active scanning
- Exploitation of vulnerabilities
- Credential testing (with care)
- Social engineering (if in scope)

❌ PROHIBITED:
- Denial of Service (unless explicitly approved)
- Accessing out-of-scope systems
- Modifying/deleting data
- Storing sensitive data beyond test
- Testing outside working hours

## 4. DATA HANDLING
- All captured data encrypted (AES-256)
- Credentials hashed in reports
- Data destroyed after [X] days
- No data leaves authorized devices

## 5. INCIDENT RESPONSE
- Stop testing if: [Conditions]
- Report immediately: [Contact]
- Preserve evidence: [Method]
```

---

## Ethical Guidelines

### Authorization Hierarchy
```
1. ✅ Your own network/equipment
2. ✅ Written permission from owner
3. ✅ Contractual engagement (pentest)
4. ⚠️  Bug bounty program (with rules)
5. ❌ No permission = ILLEGAL
```

### Responsible Disclosure
```markdown
# Template: Responsible Disclosure

To: security@company.com
Subject: Security Vulnerability in Wireless Infrastructure

## Summary
[Brief description]

## Impact
[What an attacker could do]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]

## Recommended Fix
[Suggestion]

## Timeline
- [Date] - Vulnerability discovered
- [Date] - Report sent
- [Date] + 90 days - Public disclosure (if no response)
```

---

## Scope Definition

### In-Scope vs Out-of-Scope

| Aspect | In-Scope | Out-of-Scope |
|--------|----------|--------------|
| SSID | Specific SSID listed | Other SSIDs |
| BSSID | Authorized APs | Neighbor APs |
| Clients | Company-owned | Personal devices |
| Wired | Internal network | Cloud, third-party |
| Social Eng | Approved methods | Unapproved targets |

### Emergency Stop Conditions
1. Service disruption to critical systems
2. Data corruption or loss
3. Discovery of child abuse material
4. Discovery of active intrusion
5. Legal authority request
6. Client requests stop

---

## Professional Certifications

| Certification | Organization | Focus |
|---------------|--------------|-------|
| **OSWP** | Offensive Security | Wi-Fi pen testing |
| **CWNA** | CWNP | Wireless fundamentals |
| **CWSP** | CWNP | Wireless security |
| **GPEN** | SANS/GIAC | Pen testing |
| **CEH** | EC-Council | Ethical hacking |

---

## Insurance & Liability

### Professional Liability Insurance
- Covers: Errors, omissions, negligence
- Recommended: $1M+ coverage
- Required: For most enterprise engagements

### Client Agreement Checklist
- [ ] Statement of Work (SOW)
- [ ] Rules of Engagement (RoE)
- [ ] Non-Disclosure Agreement (NDA)
- [ ] Liability waiver/indemnification
- [ ] Data handling agreement
- [ ] Emergency contact list

---

## References

- **UU ITE (Law No. 11 of 2008):** https://peraturan.go.id/uu-no-11-tahun-2008.html
- **UU ITE Amendment (Law No. 1 of 2024):** https://peraturan.go.id/
- **PTES Standard:** http://www.pentest-standard.org/
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **OSWP:** https://www.offsec.com/courses/pen-200/
