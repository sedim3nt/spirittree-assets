# The OpenClaw Client Playbook
## A Complete Consulting Delivery System for AI Operations

This playbook covers everything from first contact to post-engagement follow-up. Use it to run professional, repeatable AI consulting engagements.

---

## The Offer Stack

Before diving into delivery, get clear on what you're selling:

| Tier | Price | What's Included |
|------|-------|-----------------|
| Strategy Session | $500 | 60-min call, written roadmap, 14-day bug fix |
| Full Setup | $1,000 | Strategy + hands-on implementation (up to 4 hours) |
| Retainer | $2,000/mo | Ongoing ops: 8 hours/mo, priority support, quarterly reviews |

**Why this pricing:**
- $500 is below the "decision fatigue" threshold for most professionals
- $1,000 represents 1–2 days of your time + real value delivered
- $2,000/mo retainer requires roughly 8–10 hours/month to deliver well — protects your margins

---

## Pre-Call Intake Questionnaire

Send this before the strategy session. Use a Typeform, Notion form, or Google Form.

**Form Title:** "AI Operations Strategy Session — Pre-Call Intake"

1. **What's your primary goal for AI in your work?**
   *(Open text — look for: save time, make money, reduce stress, specific pain point)*

2. **What tools are you currently using for AI?**
   *(Checkboxes: ChatGPT, Claude, Copilot, n8n, Zapier, None, Other)*

3. **How many hours per week do you spend on tasks you wish were automated?**
   *(Dropdown: <2, 2-5, 5-10, 10-20, 20+)*

4. **What's your primary operating environment?**
   *(Mac / Linux / Windows — OpenClaw is not fully supported on Windows)*

5. **Do you have a dedicated machine for AI operations?**
   *(Yes / No / Planning to get one)*

6. **What's your comfort level with command line tools?**
   *(1-5 scale: Never used it → Use it daily)*

7. **Do you have existing API keys (Anthropic, OpenAI)?**
   *(Yes / No / Not sure what this means)*

8. **What communication channel do you primarily use for work?**
   *(Telegram / Discord / Slack / Email / Other)*

9. **What's your approximate monthly budget for AI tools and API costs?**
   *(Dropdown: $0-20, $20-50, $50-100, $100-200, $200+)*

10. **Is there anything specific you want to cover or avoid in our session?**
    *(Open text — look for constraints, past frustrations, specific use cases)*

**Review before the call:** Flag anyone with Windows as primary OS (limited support), budget under $20 (may not be ready), or CLI comfort below 2 (plan for more hand-holding).

---

## 60-Minute Session Facilitation Guide

### Agenda

| Time | Section | Goal |
|------|---------|------|
| 0–5 min | Welcome + context | Warm up, verify intake answers |
| 5–20 min | Current state audit | Understand their stack and workflow |
| 20–35 min | Gap analysis | Identify highest-leverage opportunities |
| 35–50 min | Roadmap sketch | Outline a phased implementation |
| 50–60 min | Next steps + close | Clear action items, set expectations |

### What to Cover (Must-Haves)

**Current state (5–20 min):**
- Walk through their intake answers; ask follow-up questions
- Ask: "Walk me through a typical day — when do you wish you had more help?"
- Map their current tools to the 9-layer stack (see Stack Assessment guide)
- Identify any quick wins (things they could automate today)

**Gap analysis (20–35 min):**
- Which layers are missing entirely?
- Where is manual work substituting for automation?
- What's the highest-value thing they're NOT doing because it takes too long?
- Priority question: "If you could only fix one thing in your workflow this month, what would it be?"

**Roadmap sketch (35–50 min):**
- Phase 1 (first 2 weeks): Foundation — OpenClaw install, memory setup, Telegram
- Phase 2 (month 1): Automation — first crons, key integrations
- Phase 3 (month 2–3): Leverage — multi-agent, content pipeline, revenue ops
- Adapt based on their technical comfort and specific goals

### What to Skip

- Don't go deep on technical implementation during the call — that's what Phase 2 is for
- Don't oversell complexity; keep Phase 1 achievable in 1–2 weekends
- Don't promise specific ROI numbers you can't back up
- Skip features they won't use (e.g., don't pitch multi-agent orchestration to someone who just wants to automate email)

---

## Written Roadmap Template

Deliver within 48 hours of the session. Keep it to 1–2 pages.

---

**[Client Name] — AI Operations Roadmap**
*Prepared by [Your Name] | [Date]*

**Session Summary:**
[2-3 sentences: where they are, what they want, primary constraint]

**Phase 1: Foundation (Week 1–2)**
*Goal: Working OpenClaw installation with memory and Telegram*

- [ ] Install OpenClaw via Homebrew
- [ ] Configure SOUL.md with [their] values and voice
- [ ] Set up MEMORY.md with current project context
- [ ] Connect Telegram bot
- [ ] Set morning briefing cron

*Success criteria: Agent responds on Telegram within 5 seconds. Morning briefing arrives by 9am.*

**Phase 2: Automation (Month 1)**
*Goal: First key automations running*

- [ ] [Specific automation 1 based on their gap]
- [ ] [Specific automation 2]
- [ ] [Integration they specifically need]

*Success criteria: [Specific, measurable outcome for their situation]*

**Phase 3: Leverage (Month 2–3)**
*Goal: [Their specific high-value goal]*

- [ ] [Longer-term capability based on their roadmap]
- [ ] [Revenue or time-saving goal]

**Recommended Models:**
- Default: Claude Sonnet 4.6 (quality/cost balance)
- Monitoring crons: Claude Haiku (cheapest option)
- [Specific use case]: [specific model]

**Estimated Monthly Cost:** $[X]–$[Y] based on your usage pattern

**Resources:**
- Quick-Start Guide: [link to product]
- Security Checklist: [link]
- Your 14-day bug fix window: [expiry date]

---

Questions? Reply to this email or message me on [channel].

---

## Handoff Walkthrough Script

For $1,000 Full Setup clients: a 30-minute walkthrough call after implementation.

**Script:**

"Let's do a quick tour of what I set up. [Screen share or async Loom]

First, here's your workspace at ~/.openclaw/workspace/. The three files that matter most are:
- SOUL.md — this shapes how your agent communicates; I set it to [describe their personality]
- MEMORY.md — this is your agent's long-term memory; I seeded it with your current projects
- AGENTS.md — this defines what your agent does and when

Your Telegram bot is [bot username]. I've set it up to receive your messages here [show topic structure if forum].

The crons I set up are:
1. [Cron 1 description] — runs at [time]
2. [Cron 2 description] — runs at [time]

To test it yourself, send this message: 'What are my active projects?' — you should see it pull from MEMORY.md.

For the next 14 days, if anything breaks, message me and I'll fix it within 24 hours. After that, the resources to maintain this yourself are [links].

Any questions?"

---

## 14-Day Bug Fix Warranty

Include in every engagement:

> **14-Day Bug Fix Guarantee**
> 
> If anything in your OpenClaw setup stops working within 14 days of our session, I'll fix it at no charge. This covers: configuration issues, integration failures, cron problems, and anything else directly related to the work I delivered.
> 
> It does not cover: changes you make to the configuration, external service outages (Telegram, Anthropic, etc.), or new feature requests.
> 
> To invoke: message me directly at [your contact] with a description of the issue.

---

## Monthly Retainer Structure

**What's included at $2,000/month:**

- 8 hours of implementation and operations work
- Priority response (within 4 hours business hours, 24 hours otherwise)
- Monthly 30-minute strategy call
- Quarterly stack assessment and roadmap update
- Access to all new guides and products you release

**What's not included:**
- On-call emergency support (available as add-on)
- Third-party API costs (client pays their own Anthropic/OpenAI bills)
- Major custom development beyond 8 hours

**How to scope work at the start of each month:**
Send a brief at the beginning of each month outlining their top 3 priorities. You deliver against those priorities within the 8-hour budget. Unused hours don't roll over (keeps scope clean).

---

## Calendly Booking Flow

Configure Calendly (or equivalent) with these event types:

**"AI Strategy Session — 60 min" ($500)**
- Requires intake form completion before booking link appears
- Buffer: 30 min before and after (you need prep/debrief time)
- Questions on booking: "Any last-minute context I should know?"
- Confirmation email: Sends intake form link if not already completed

**"Implementation Review — 30 min" (Free, retainer clients only)**
- Requires retainer client code in URL
- Available 2× per month

**"Quick Consult — 15 min" (Free discovery call)**
- No payment required
- Purpose: determine if they're a fit for paid services
- Qualification: ask budget and tech comfort in booking questions

---

## Post-Engagement Follow-Up Sequence

Send these three emails after a paid engagement:

**Email 1: Day 3**
Subject: Quick check-in — [Name]'s OpenClaw setup

> Hey [Name],
> 
> Just checking in — how's everything running? Any issues with the setup so far?
> 
> If you haven't tried the morning briefing cron yet, message your bot "what's on my plate today?" and see what comes back.
> 
> Let me know if anything needs tweaking.
> 
> [Your name]

**Email 2: Day 14 (end of warranty)**
Subject: Your 14-day bug fix period ends today

> Hey [Name],
> 
> Quick note: your 14-day bug fix period ends today. If anything is still acting up, let me know now and I'll handle it.
> 
> Going forward, here are the resources to maintain your setup:
> - [Link to quick-start guide]
> - [Link to security checklist]
> - [Community or forum link]
> 
> If you want ongoing support, I have monthly retainer spots available at $2,000/month — [link to Calendly].
> 
> Thanks for working with me. Would love a quick review if you have 2 minutes: [testimonial link]
> 
> [Your name]

**Email 3: Day 45**
Subject: 6 weeks in — how's your AI setup?

> Hey [Name],
> 
> Six weeks since we set up your OpenClaw operation. Curious:
> 
> - Is the morning briefing still running?
> - Have you added any new crons or integrations?
> - What's working? What's not?
> 
> I'm also releasing some new guides you might find useful:
> - [Relevant guide 1]
> - [Relevant guide 2]
> 
> If you want to level up further, my retainer spots are [open/almost full] this quarter.
> 
> [Your name]

---

## Statement of Work Template

Use for $1,000+ engagements:

---

**Statement of Work**
Between: [Your name/company] ("Consultant") and [Client name] ("Client")
Date: [Date]
Project: AI Operations Setup — OpenClaw Implementation

**Scope of Work:**
1. OpenClaw installation and configuration on Client's machine
2. SOUL.md, MEMORY.md, and AGENTS.md setup
3. Telegram bot connection and [topics/DM] configuration
4. [N] cron automations per intake agreement
5. [Specific integration] connection and testing
6. 30-minute handoff walkthrough call

**Not Included:**
- Third-party API costs
- Changes requested after handoff call
- Support after 14-day warranty period

**Timeline:** Work completed within 5 business days of payment

**Payment:** $[amount] due before work begins. Payment via [Stripe/invoice link].

**14-Day Warranty:** As described in the attached warranty terms.

**Confidentiality:** Consultant will not share Client's configuration, data, or workflow details with third parties.

---

## Client Feedback Form

Send after Email 2 (Day 14):

1. **How easy was the onboarding process?** (1–5 stars)
2. **How useful has the system been in your daily work?** (1–5 stars)
3. **Would you recommend this service?** (Yes / No / Maybe)
4. **What's working well?** (Open text)
5. **What could be better?** (Open text)
6. **Can I use your feedback as a testimonial?** (Yes / No / Yes, anonymously)

*Use responses to improve intake, the session guide, and onboarding materials. Track NPS over time.*

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
