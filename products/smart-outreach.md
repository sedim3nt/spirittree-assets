# Smart Outreach — Research-First Cold Email System

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Research-first cold outreach that actually gets replies. AI researches each prospect, writes personalized emails, manages follow-ups, and tracks your pipeline. Not mass spam — surgical outreach.

---

## What's In This Package

- Prospect research template
- Personalization framework
- Email writing system (3 angles per prospect)
- Follow-up sequence (3-touch)
- Reply detection and categorization
- Pipeline tracker
- Do-not-contact management

---

## Core Philosophy

Generic outreach fails because it's about you. Research-first outreach works because it's about them.

Every email you send should answer: "Why are you emailing ME specifically, about THIS, RIGHT NOW?"

If you can't answer that in one sentence → don't send it yet.

---

## Prospect Research Template

Before writing a single word, research the person:

```
Research prospect: [FULL NAME] at [COMPANY]

Find:
1. **Their specific role** — what do they actually do day-to-day? (not just job title)
2. **Recent activity** — anything they've published, spoken about, or shared recently
3. **Company context** — what's the company focused on right now? Any recent news?
4. **Pain signals** — any public evidence of the problem I solve? (job postings, blog posts, complaints)
5. **Trigger event** — why NOW? (funding, new role, product launch, hiring surge, etc.)

Sources:
- LinkedIn (their profile + recent posts)
- Company website (blog, press, team page)
- Twitter/X (recent activity)
- Google News: "[name] [company]"
- Job postings (signals strategic priorities)

Output:
---
**[NAME] — [TITLE] at [COMPANY]**

Current focus: [What they're working on]
Recent signal: [Most relevant recent activity]
Pain indicator: [Evidence they have the problem you solve]
Trigger: [Why now is the right time]
Personalization hook: [The one specific thing to open with]
---
```

---

## Personalization Framework

### The Three Angles

For every prospect, generate 3 different email angles and choose the best one:

```
Generate 3 email angles for reaching [NAME] at [COMPANY].

Context:
- Their focus: [what you found in research]
- My offering: [what you do]
- Connection: [why they specifically would care]

Angle 1: PROBLEM FIRST
Open with evidence of a problem you know they have.
"I saw [specific signal]. That usually means [problem]. We help with that."

Angle 2: INSIGHT FIRST
Lead with a relevant insight or data point.
"Most [role] at [company type] are dealing with [problem]. Here's what we've seen work."

Angle 3: TRIGGER FIRST
Reference the specific trigger event.
"Congrats on [news item / new role / funding]. As you scale [specific thing], [specific challenge] often becomes urgent."

For each angle: write the first 2-3 sentences only. I'll pick the best.
```

---

## Email Writing System

### Initial Email Template

```
Write a cold outreach email to [NAME] at [COMPANY].

Research summary:
[PASTE RESEARCH OUTPUT]

My offering: [WHAT YOU DO IN 1 SENTENCE]
My credibility: [1 relevant proof point — result, client, or experience]

Chosen angle: [ANGLE FROM ABOVE]

Email constraints:
- Under 150 words total
- No more than 2 sentences about me
- End with a specific, low-commitment ask (not "do you have time for a call?")
- No subject line tricks or fake re: patterns

Good asks: "Would it be worth 15 minutes?" / "Can I send you one thing?" / "Is this on your radar?"

Write the email. Then evaluate: does this answer "why me, specifically, right now?" If not, revise.
```

### Subject Line Generator

```
Generate 5 subject lines for outreach to [NAME] about [TOPIC].

Rules:
- Under 8 words
- No question marks (too salesy)
- Reference something specific from their context
- Avoid: "Quick question" / "Following up" / "Partnership opportunity"

Good examples:
- "Re: [their recent blog post title]"
- "[Specific result] for [their company type]"
- "[Mutual connection] suggested I reach out"
```

---

## Follow-Up Sequence

### 3-Touch Follow-Up

If no reply to initial email, follow up twice more. Spacing: 3 days after email 1, 7 days after email 2.

**Touch 2 — Day 3 (value add):**

```
Write Touch 2 follow-up for [NAME].

Initial email: [PASTE]
Days since sent: 3
No reply yet.

Touch 2 rules:
- Don't reference that you emailed before (or do it once, briefly)
- Add something new — a resource, insight, or relevant news
- Still short: under 100 words
- Different angle from Touch 1

Approach: "Thought this might be useful given [their situation]" + [resource/insight]
```

**Touch 3 — Day 10 (close the loop):**

```
Write Touch 3 for [NAME].

This is the last email in the sequence. After this, stop.

Rules:
- Short: under 75 words
- Close the loop respectfully
- Leave the door open without being clingy

Standard pattern: "I'll assume the timing's off. If [problem] becomes urgent, happy to reconnect. [One sentence on what you do]."
```

---

## Reply Detection and Categorization

### Categorize Replies Prompt

```
Categorize this reply from [NAME] at [COMPANY]:

"[PASTE REPLY]"

Categories:
- 🟢 INTERESTED — positive, wants to continue
- 🟡 NOT NOW — interested but timing is wrong (save for future)
- 🔵 WRONG PERSON — referred me to someone else
- 🔴 NOT INTERESTED — clear no
- ❓ UNCLEAR — needs clarification

For each category, recommend the right response:
- INTERESTED → suggest next step (call, demo, send info)
- NOT NOW → suggest timing to follow up, note what they said
- WRONG PERSON → how to approach the referral
- NOT INTERESTED → graceful acknowledgment
- UNCLEAR → what to ask to clarify

Draft the appropriate response.
```

---

## Pipeline Tracker

Create `outreach/pipeline.md`:

```markdown
# Outreach Pipeline

## Active Sequences

| Name | Company | Title | Sent | Status | Last Touch | Next Action |
|------|---------|-------|------|--------|------------|-------------|
| [Name] | [Co] | [Title] | [Date] | Touch 1 | [Date] | Touch 2 [Date] |
| [Name] | [Co] | [Title] | [Date] | Touch 2 | [Date] | Touch 3 [Date] |

## Interested (In Conversations)

| Name | Company | Stage | Last Contact | Next Step |
|------|---------|-------|-------------|-----------|
| [Name] | [Co] | Call scheduled | [Date] | [Action] |

## Not Now (Follow Up Later)

| Name | Company | Reason | Follow Up Date |
|------|---------|--------|----------------|
| [Name] | [Co] | Budget Q2 | [Date] |

## Closed (Not Interested)

| Name | Company | Date | Notes |
|------|---------|------|-------|

## Weekly Metrics

| Week | Sent | Opens (est) | Replies | Positive | Meetings |
|------|------|------------|---------|----------|---------|
```

### Pipeline Review Prompt

Run weekly:

```
Outreach pipeline review. Read outreach/pipeline.md.

1. Who needs a follow-up this week? (Touch 2 or Touch 3 due)
2. Any "Not Now" prospects whose timing may have changed?
3. Active conversations needing next steps?
4. Metrics: emails sent / replies / positive rate

For due follow-ups: draft the appropriate touch
For "Not Now": check if trigger events have occurred for any

Update pipeline.md with any status changes.
```

---

## Do-Not-Contact Management

Create `outreach/dnc.md`:

```markdown
# Do-Not-Contact List

Anyone on this list should never receive outreach again.

| Name | Email/Company | Reason | Date Added |
|------|---------------|--------|------------|
| [Name] | [Email or Company domain] | Opted out | [Date] |
| [Name] | [Email] | Existing client | [Date] |
| [Company domain] | @competitor.com | Competitor | [Date] |
```

### DNC Check Prompt

Before sending any outreach:

```
Before sending to [NAME] at [EMAIL]:

Check outreach/dnc.md for:
1. Their email address
2. Their company domain
3. Their company name

If any match: DO NOT SEND. Report that this contact is on the DNC list and why.
If clear: confirm "not on DNC list — ok to send"
```

---

## A/B Testing Framework

```
A/B test tracking for outreach campaign: [CAMPAIGN NAME]

Variant A: [Subject line or angle]
Variant B: [Different subject line or angle]

After 20+ sends per variant:
- Variant A: sent [X] / replies [X] / positive [X]
- Variant B: sent [X] / replies [X] / positive [X]

Winner: [A/B] with [X%] reply rate vs [X%]

Insight: [Why did the winner work better?]

Update: switch all new sends to the winning variant.
```

---

## Troubleshooting

**Emails not getting replies**
- Is the research actually specific? "I saw your LinkedIn" is not specific
- Is the ask too big? First email should not ask for a 30-minute call
- Is the email too long? If >150 words, cut it
- Send from a personal email, not marketing@ or noreply@

**Research not finding enough info**
- Some prospects are private — that's fine, use company research instead
- Look for recent job postings as a proxy for priorities
- Check if they've been quoted in any press as an industry source

**Follow-ups feel annoying**
- Space them out more (try 5 days and 14 days instead of 3 and 7)
- Each touch needs to add new value — if you're just bumping, stop
- 3 touches max. After that, you're spam.

**Pipeline getting messy**
- Run weekly reviews to clear stale entries
- Move anything >60 days with no response to "Closed" (you can always reopen)
- Keep active sequences to <20 people — quality attention over volume

---

*Built on OpenClaw and himalaya CLI for email sending.*
