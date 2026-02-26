# Security Policy: PaperTimes Infrastructure

At PaperTimes, we take the security of our "Global-to-Local" infrastructure and our clients' data isolation seriously. If you believe you have found a security vulnerability, we appreciate your help in disclosing it to us in a responsible manner.

## Supported Versions

We recommend all clients and internal operators stay on the latest production releases.

---

## Reporting a Vulnerability

**Do not open a GitHub Issue for security vulnerabilities.** Publicly disclosing a bug before a fix is available puts our B2B partners at risk.

Please report security bugs via email to: **security@papertimes.ca**

### **What to include in your report:**

1. **Scope:** Is this a Global Admin leak or a Client Isolation (Convex/WorkOS) bypass?
2. **Severity:** Use [CVSS v3.1](https://www.first.org/cvss/calculator/3.1) if possible.
3. **Proof of Concept (PoC):** Step-by-step instructions or scripts to reproduce the issue.
4. **Impact:** What data or environment can be accessed?

---

## Our Disclosure Process

Once a report is received, our Infrastructure Ops team follows this lifecycle:

1. **Acknowledgment:** Within **24–48 hours**, we will confirm receipt of your report.
2. **Triage:** We will validate the vulnerability and determine its impact on our Railway/Convex architecture.
3. **Remediation:** We aim to deploy a fix to the `Global Admin` and migrate all `Client Projects` within **1-14 days** for critical issues.
4. **Disclosure:** Once the fix is verified in Production, we will coordinate a public disclosure if requested.

---

## Out of Scope

The following activities are strictly prohibited and not eligible for recognition:

- **Denial of Service (DoS/DDoS)** attacks against Railway or Convex infrastructure.
- **Social Engineering** or Phishing of PaperTimes employees or clients.
- **Physical Attacks** against data centers or offices.
- **Spamming** through our WorkOS integration or notification systems.

---

### **Responsible Disclosure Guidelines**

- Give us a reasonable amount of time to fix the issue before sharing it publicly.
- Make a good faith effort to avoid privacy violations and data destruction.
- Only interact with accounts you own or have explicit permission to test.
