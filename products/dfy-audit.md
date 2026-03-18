# Done-For-You Security Audit
## Professional Review of Your OpenClaw Setup

This document explains what the Done-For-You (DFY) Audit service includes, what we check, what you'll receive, and how to book.

---

## What Is the DFY Audit?

The DFY Audit is a professional security and architecture review of your OpenClaw setup. We go through your configuration, memory files, installed skills, network exposure, and operational patterns to find vulnerabilities and inefficiencies you wouldn't catch running the self-service checklist.

The result: a detailed report of what we found, instructions for fixing each issue, and a 30-minute call to walk through the findings.

**Delivered within 48 hours of access being granted.**

---

## What We Check (9 Areas)

### Area 1: File System and Permissions

We review the permissions on all workspace files, memory files, config files, and skill directories.

What we look for:
- World-readable files containing sensitive context
- Config files accessible to other users or processes
- Memory files with incorrect permissions
- Workspace directory structure that creates unnecessary exposure

Tools used: `ls -la`, `find`, permission analysis against security baseline

---

### Area 2: Gateway Configuration

We audit your OpenClaw gateway configuration for network exposure and authentication strength.

What we look for:
- Gateway bound to `0.0.0.0` (exposed to all interfaces)
- Weak or default auth tokens
- Ports accessible from outside your local network
- Missing or misconfigured TLS on public-facing deployments
- Unused gateway services that increase attack surface

---

### Area 3: Installed Skills

We review every installed skill for security anti-patterns.

What we look for:
- Outbound HTTP calls to unexpected or unverified domains
- Credential access beyond the skill's documented scope
- Shell command execution that could be exploited
- Skills not updated in >6 months (may be broken or vulnerable)
- Skills you didn't explicitly install

We use the SkillScan procedure to flag each finding with severity (low/medium/high).

---

### Area 4: Memory and Context Files

We review your MEMORY.md, daily memory logs, and other workspace files for sensitive content that shouldn't be there.

What we look for:
- Hardcoded API keys, tokens, or passwords
- Personal information that creates privacy risk
- Content that could be exploited via prompt injection
- Inconsistent or stale context that creates confusing agent behavior
- Memory hygiene issues (files that compound confusion over time)

---

### Area 5: Cron and Automation Security

We review all configured crons for suspicious patterns and security gaps.

What we look for:
- Crons calling external URLs you don't recognize
- Crons with overly broad access
- Crons that were never meant to persist but are still running
- Missing error handling that could lead to silent failures or loops
- Automation that creates unintended external state changes

---

### Area 6: Network Exposure

We check what your system is exposing to the network.

What we look for:
- Open ports from OpenClaw services
- Other services (n8n, web servers, etc.) that are inadvertently exposed
- Firewall configuration gaps
- Services running on default ports without authentication
- Tailscale or VPN configuration issues (if applicable)

Commands: `lsof -i`, `netstat -an`, firewall rule review

---

### Area 7: Prompt Injection Resistance

We test whether your setup is vulnerable to prompt injection attacks.

We run the 5-point injection test battery:
1. Role override attempt
2. File exfiltration via instruction
3. Credential extraction request
4. Authority spoofing test
5. Instruction persistence test

For each test, we document whether the agent resisted or complied and recommend specific SOUL.md/AGENTS.md additions to close any gaps.

---

### Area 8: Sub-Agent Permissions

We review how sub-agents are scoped in your AGENTS.md.

What we look for:
- Sub-agents with implicit unlimited permissions
- Missing financial operation blocks
- Missing external communication restrictions
- Escalation paths that don't exist (sub-agent told to escalate, but no clear path defined)
- Autonomy assignments that don't match actual task risk

---

### Area 9: Operational Security Hygiene

We review overall security practices — the stuff that doesn't fit neatly in the other eight areas.

What we look for:
- Credential rotation practices (how often API keys are rotated)
- Backup practices (is the workspace backed up?)
- Update practices (is OpenClaw current?)
- Session discipline (long context sessions with sensitive data)
- Documentation of what's running and why

---

## How to Schedule

**Book your audit here:** [Calendly link — link to booking page]

At booking, you'll be asked to:
1. Confirm your operating system (macOS / Linux)
2. Describe your setup at a high level
3. Choose a delivery window (standard: 48 hours, urgent: 24 hours +$100)

---

## What Access We Need

We work asynchronously — you grant us temporary access to review your setup, then revoke it after we deliver the report.

**Required access:**
- SSH or screen share session (30 minutes, scheduled at your convenience)
- OR: a zip export of your workspace directory with sensitive values redacted (we'll guide you through this)

**What we do NOT need:**
- Your API keys (we'll check that they exist, not what they are)
- Your Telegram token (we'll verify config structure, not credentials)
- Your MEMORY.md contents (unless you choose to share for context hygiene review)

All access is governed by the confidentiality terms in the engagement agreement. Nothing leaves our review environment.

---

## Timeline

| Step | Timeline |
|------|----------|
| Booking confirmed | Day 0 |
| Access session (30 min) | Day 1 |
| Audit completed | Day 2 (within 48 hours of access) |
| Report delivered | Day 2 |
| Debrief call (30 min) | Day 3–5 (scheduled at your convenience) |
| Follow-up questions | Within 7 days of report delivery |

For urgent audits, we offer a 24-hour turnaround at additional cost.

---

## What You Get

**1. Vulnerability Report**

A structured document covering all 9 audit areas. For each finding:
- **Severity:** Low / Medium / High / Critical
- **Description:** What we found and why it's an issue
- **Evidence:** The specific file, config, or pattern that triggered the finding
- **Risk:** What could happen if this isn't addressed
- **Fix:** Step-by-step remediation instructions with exact commands

**2. Fix-It Instructions**

Every finding over "Low" severity includes a concrete fix — not just "review this" but the actual commands or file changes needed to resolve it. You can apply most fixes in under an hour.

**3. 30-Minute Debrief Call**

A call where we walk through the top findings, answer your questions, and prioritize what to fix first. Recorded and shared with you for reference.

**4. Architecture Recommendations (if applicable)**

If we identify structural issues beyond security (e.g., memory architecture that will create problems at scale, automation patterns that are fragile), we note them separately with recommendations.

---

## Sample Report Format

```markdown
# OpenClaw Security Audit Report
Client: [Name]
Date: [Date]
Auditor: AgentOrchard Operations

## Executive Summary
[2-3 sentences: overall assessment, top findings, recommended priority]

Overall Risk Level: [Low / Medium / High / Critical]
Findings: [N] total — [N] Critical, [N] High, [N] Medium, [N] Low

## Findings

### [FINDING-001] Gateway bound to 0.0.0.0
Severity: HIGH
Area: Gateway Configuration

Description:
The OpenClaw gateway is bound to 0.0.0.0, making it accessible on all 
network interfaces, including public-facing ones.

Evidence:
$ lsof -i :3737
openclaw  12345  user  IPv4  ...  0.0.0.0:3737

Risk:
Any device on your network (or internet, if port is open) can reach 
your agent gateway without credentials.

Fix:
Update ~/.openclaw/config.json:
  "gateway": { "bind": "127.0.0.1", ... }
Then: openclaw gateway restart

---

### [FINDING-002] Hardcoded token in MEMORY.md
Severity: CRITICAL
Area: Memory and Context Files

[...]
```

---

## Pricing

| Package | Price | Includes |
|---------|-------|---------|
| Standard Audit | $350 | Full 9-area review, written report, 30-min call |
| Urgent (24hr) | $450 | Same as Standard, faster delivery |
| Audit + Fix | $750 | Standard audit + we implement all the fixes for you |

---

## FAQ

**Do I need to be technical to benefit from this?**
No. The report is written for a range of technical levels. Fix-it instructions are step-by-step. The debrief call is where we answer questions in plain language.

**Will you see my private files?**
We only review files directly relevant to the audit. We use a redacted workspace export for most clients. If we need to see specific files during the access session, we ask permission first.

**What if my setup is a mess and I'm embarrassed?**
This is exactly what the audit is for. We've seen everything. A messy setup is not a judgment — it's an opportunity to improve.

**What if you don't find anything?**
That's a good outcome. We'll confirm your setup is solid and provide recommendations for next steps. Full report and call still included.

**Can I use this as a basis for a client-facing security claim?**
The audit report is for your internal use. It documents our findings, not a certification. We don't offer SOC2 or similar certifications.

**What's covered by the 7-day follow-up window?**
Any questions about the report findings or fix instructions. Not: new issues that arise after the audit, implementation troubleshooting, or new feature requests.

**Is this the same as the self-service security checklist?**
The security checklist (products/security-checklist.md) covers the same areas but is designed for self-service. The DFY audit is deeper, includes prompt injection testing, architecture review, and a debrief call — and you get an expert set of eyes rather than running checks yourself.

---

## Ready to Book?

**[Schedule your audit →]** [Calendly link]

Questions before booking? Message at [contact method] and expect a response within 24 hours.

---

*Part of the AgentOrchard professional services offering. See client-playbook.md for the full consulting service menu.*
