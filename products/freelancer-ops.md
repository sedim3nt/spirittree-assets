# Freelancer Ops Agent — AI Freelance Operations Package

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Invoice tracking, time logging, client communication templates, project status, and quarterly tax prep reminders. For freelancers and consultants who hate admin work.

---

## What's In This Package

- `ROLE.md` for your freelancer ops agent
- Invoice template generator
- Time tracking system
- Client communication templates (full project lifecycle)
- Tax prep reminder cron (quarterly)
- Revenue tracking setup
- Rate calculator

---

## ROLE.md — Freelancer Ops Agent

Copy to `agents/freelance/ROLE.md`:

```markdown
# ROLE.md — Freelancer Ops Agent

**Role:** Freelancer Operations Manager
**Model:** Claude Sonnet
**Channel:** Sub-agent or direct

## Responsibilities
- Invoice generation and tracking
- Time logging and project summaries
- Client communication (templated, not improvised)
- Tax prep reminders and documentation
- Revenue tracking and projections

## Client Communication Rules
- Professional but human — not robotic
- Always include project or reference number
- Never commit to timelines without checking current capacity
- Flag: disputes, payment delays >14 days, scope creep

## Invoice Rules
- Net 30 default unless different terms agreed
- Late payment: flag at day 7, 14, and 21 after due date
- Never waive late fees without explicit approval
- Track invoice status: Draft / Sent / Viewed / Paid / Overdue

## Constraints
- Never send communications impersonating the operator
- Draft + review workflow for all client-facing communications
- Track all time — even partial hours
```

---

## Invoice Template Generator

### Basic Invoice Prompt

```
Generate an invoice for:

Client: [CLIENT NAME]
Project: [PROJECT NAME]
Invoice #: [INV-YYYY-MM-XXX]
Date: [TODAY'S DATE]
Due Date: [30 days from today or agreed terms]

Line Items:
1. [Service description] — [hours]h × $[rate] = $[amount]
2. [Service description] — [hours]h × $[rate] = $[amount]
3. [Expenses, if any] — $[amount]

Notes: [Any specific notes for this client]

Payment: [Payment method — wire/ACH/PayPal/etc.]
Late fee: 1.5% per month on overdue balances.

Format as a clean invoice I can send as text or paste into a doc.
```

### Invoice Markdown Template

```markdown
---
**INVOICE**

From: [Your Name / Company]
[Your Address]
[City, State ZIP]
[your@email.com]

To: [Client Name]
[Client Company]
[Client Address]

---

**Invoice #:** INV-[YYYY-MM-XXX]
**Invoice Date:** [DATE]
**Due Date:** [DATE — Net 30]
**Project:** [Project Name]

---

| Description | Hours | Rate | Amount |
|-------------|-------|------|--------|
| [Service 1] | [X]h | $[rate]/hr | $[amount] |
| [Service 2] | [X]h | $[rate]/hr | $[amount] |
| [Expense/item] | — | — | $[amount] |

---

**Subtotal:** $[amount]
**Tax (if applicable):** $[amount]
**Total Due:** $[TOTAL]

---

Payment methods: [Wire / ACH / PayPal / Stripe link]
[Payment instructions or link]

Late payments: 1.5% per month on unpaid balances after due date.

Thank you for the opportunity to work together.
```

---

## Time Tracking System

### Daily Time Log

Create `freelance/time/[YYYY-MM].md`:

```markdown
# Time Log — [MONTH YEAR]

## [DATE]
- [Client/Project]: [Description of work] — [X.X hours]
- [Client/Project]: [Description of work] — [X.X hours]
**Day Total:** [X.X hours]

## [DATE]
...

## Monthly Summary
| Client | Project | Hours | Rate | Value |
|--------|---------|-------|------|-------|
| [Client] | [Project] | [X.X]h | $[rate] | $[amount] |
| [Client] | [Project] | [X.X]h | $[rate] | $[amount] |

**Total Hours:** [X.X]
**Total Value:** $[amount]
```

### Time Tracking Prompts

**Log time quickly:**
```
Log time: [CLIENT] — [BRIEF DESCRIPTION] — [X hours/minutes]
Add to freelance/time/[current month].md
```

**Weekly summary:**
```
Summarize this week's time logs from freelance/time/[month].md.

Show:
1. Hours by client
2. Hours by project type (design / dev / calls / admin)
3. Billable vs. non-billable split
4. This week vs. last week trend

Flag: any project over budget or any client taking more time than billed.
```

**Invoice prep:**
```
Pull all time logged for [CLIENT] in [MONTH] from freelance/time/[month].md.

Format as invoice line items:
- Group by project/phase
- Total hours per group
- Apply rate: $[X]/hour
- Include date range

Ready for invoice generation.
```

---

## Client Communication Templates

Create `freelance/templates/`:

### Project Kickoff

```
Subject: [PROJECT NAME] — Kickoff Confirmation

Hi [Name],

Excited to get started on [PROJECT NAME]. Here's what to expect:

**Scope:** [1-2 sentences on what we're building]

**Timeline:**
- Kickoff: [DATE]
- First milestone: [DATE + description]
- Delivery: [DATE]

**What I need from you:**
- [Asset/access/decision 1]
- [Asset/access/decision 2]

**Next step:** I'll [describe first action] by [DATE].

Best,
[Your Name]
```

### Weekly Status Update

```
Subject: [PROJECT NAME] — Week [#] Update

Hi [Name],

Quick update:

**Done this week:**
- [Completed item 1]
- [Completed item 2]

**In progress:**
- [What's underway]

**Next week:**
- [What's coming]

**Questions / needs:**
- [Any blocking issues or decisions needed]

On track for [DELIVERY DATE]. Let me know if anything's shifted on your end.

[Your Name]
```

### Project Completion

```
Subject: [PROJECT NAME] — Complete ✓

Hi [Name],

[PROJECT NAME] is complete. Here's a summary of what was delivered:

**Deliverables:**
- [Item 1 — with link if applicable]
- [Item 2]
- [Item 3]

**Access:**
[Any credentials, repo links, handoff files]

**Next steps for you:**
[What they need to do to launch/use it]

Invoice #[INV-XXX] for $[AMOUNT] is attached. Due [DATE].

It was great working on this. [Personalized 1-liner about the project]

[Your Name]
```

### Follow-up After No Response

```
Subject: Re: [ORIGINAL SUBJECT]

Hi [Name],

Following up on my message from [DATE]. Wanted to make sure this didn't fall through the cracks.

[Restate the key point or ask in 1-2 sentences]

Happy to jump on a quick call if easier. [Calendly link if you have one]

[Your Name]
```

### Scope Creep Response

```
Subject: Re: [PROJECT NAME] — Additional Work

Hi [Name],

Happy to do this. Just want to flag that [REQUESTED CHANGE/ADDITION] is outside the original scope of [PROJECT NAME].

Here's what the additional work would involve:
- [Description]
- Estimated: [X hours] at $[rate] = $[amount]

If you'd like to proceed, I'll add this to the project and update the invoice. Otherwise, I can complete the original scope as planned.

Let me know how you'd like to handle it.

[Your Name]
```

### Late Payment (Day 7)

```
Subject: Invoice #[INV-XXX] — Payment Reminder

Hi [Name],

Just a friendly reminder that Invoice #[INV-XXX] for $[AMOUNT], due [DATE], is now [X] days past due.

[Payment link or instructions]

Please let me know if there's anything on your end I can help resolve.

[Your Name]
```

---

## Tax Prep Reminders

### Quarterly Cron

Schedule: First day of each quarter at 8:00 AM

**Q1 (April 1):** `0 8 1 4 *`
**Q2 (July 1):** `0 8 1 7 *`
**Q3 (October 1):** `0 8 1 10 *`
**Q4 (January 1):** `0 8 1 1 *`

Cron message:
```
Quarterly tax prep reminder. Pull revenue data from freelance/time/*.md.

1. Calculate total revenue for the past quarter
2. Estimate self-employment tax owed (SE tax ≈ 15.3% on 92.35% of net income)
3. Estimate federal income tax owed (varies by bracket)
4. List all deductible expenses tracked this quarter
5. Flag any missing documentation

Quarterly estimated tax payment is due [STANDARD DATES: Apr 15, Jun 15, Sep 15, Jan 15].

Action items:
- [ ] Pay estimated taxes by [DATE]
- [ ] Review and categorize expenses
- [ ] Check for any uncollected invoices

Send summary via Telegram.
```

---

## Revenue Tracking

Create `freelance/revenue.md`:

```markdown
# Revenue Tracker

## [YEAR] Summary

| Month | Client | Project | Amount | Status |
|-------|--------|---------|--------|--------|
| Jan | [Client] | [Project] | $[amount] | ✅ Paid |
| Jan | [Client] | [Project] | $[amount] | ⏳ Outstanding |

## Running Totals
- **Invoiced YTD:** $[amount]
- **Collected YTD:** $[amount]
- **Outstanding:** $[amount]
- **Estimated Q[X] Tax:** $[amount]

## Pipeline
| Client | Potential Project | Estimated Value | Stage |
|--------|------------------|-----------------|-------|
| [Client] | [Project] | $[estimate] | Proposal |
| [Client] | [Project] | $[estimate] | Negotiation |
```

### Revenue Tracking Prompt (monthly)

```
Update freelance/revenue.md with this month's activity.

From time logs (freelance/time/[month].md), pull:
1. All invoices sent this month with amounts
2. Any payments received this month
3. Outstanding invoices and their age

Update the tracker and calculate:
- Invoiced vs. collected gap
- Estimated quarterly tax liability
- Monthly run rate vs. last month
```

---

## Rate Calculator

Prompt to help set or evaluate rates:

```
Help me calculate a fair rate for [SERVICE TYPE].

My inputs:
- Annual income target: $[amount]
- Billable hours/week (realistic): [X hours]
- Weeks working/year (subtract vacation): [X weeks]
- Annual expenses (tools, insurance, taxes): $[amount]

Calculate:
1. Minimum viable rate: (income target + expenses) ÷ billable hours
2. Market rate check: [search for typical rates for this role/service]
3. Recommended rate with 20% buffer for slow months
4. What I'd need to charge at [X] hours/week to hit [Y income target]

Also calculate what happens if utilization drops from 100% to 70%.
```

---

## Troubleshooting

**Invoice not tracking correctly**
- Use consistent invoice numbering: INV-YYYY-MM-XXX
- Log every invoice in revenue.md the day it's sent
- Use Stripe or a simple spreadsheet if you need receipt confirmation

**Client not responding to payment follow-ups**
- Day 7: Friendly reminder (template above)
- Day 14: Firm reminder, reference late fee policy
- Day 21: Final notice, consider collections or dispute
- Always document every communication with dates

**Time logs getting behind**
- Log at end of each work session, not end of day
- Use a 5-second log: "Add [CLIENT] 2h [BRIEF] to today's log"
- Weekly review forces you to fill in gaps before memory fades

**Scope creep not being caught**
- Review original scope at the start of each project week
- Use the scope creep template early — it's easier at "here's a small addition" than "we've already done 3x the work"
- Track all out-of-scope requests even if you do them for free

---

*Built on OpenClaw. No external tools required.*
