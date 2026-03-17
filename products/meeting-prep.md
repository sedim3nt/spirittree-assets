# Meeting Prep Agent — AI Meeting Research Package

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 15-20 minutes

Feed it an agenda and attendee list. It researches every person and their company, and builds you a 1-page brief with talking points and smart questions. 5 minutes instead of 45.

---

## What's In This Package

- `ROLE.md` for your meeting prep agent
- Person research template
- Company research template
- Meeting brief output format
- Talking points generator
- Questions-to-ask framework
- Pre-meeting checklist

---

## ROLE.md — Meeting Prep Agent

Copy to `agents/meetings/ROLE.md`:

```markdown
# ROLE.md — Meeting Prep Agent

**Role:** Meeting Preparation Specialist
**Model:** Claude Sonnet
**Channel:** Sub-agent or direct

## Responsibilities
- Research meeting attendees (professional background, recent activity)
- Research companies (size, stage, recent news, products)
- Build 1-page meeting briefs
- Generate targeted talking points and smart questions
- Identify mutual connections and common ground

## Research Process
1. Search for person by name + company/role
2. Check LinkedIn, Twitter/X, company blog, recent press
3. Note recent activity (published anything? new role? company news?)
4. Research company independently
5. Synthesize into brief

## Output Format
- Always produce a 1-page brief
- Lead with most useful insight, not most obvious
- Talking points must be specific to this person — not generic
- Questions must be open-ended and show research

## Constraints
- Public information only
- Don't fabricate specifics — flag gaps as [NOT FOUND]
- Note the date of each piece of information
- Never make up quotes or positions
```

---

## How to Use This Package

### The Core Prompt

Send this before any meeting:

```
Meeting prep for: [MEETING TITLE]

Date/Time: [WHEN]
Duration: [HOW LONG]
Meeting goal: [What you want to accomplish in this meeting]

Attendees:
1. [Name] — [Title] at [Company]
2. [Name] — [Title] at [Company]

Research each person and company. Build me a 1-page brief.
```

---

## Person Research Template

### Research Prompt (per person)

```
Research [FULL NAME], [TITLE] at [COMPANY].

Find:
1. **Professional background** — career path, previous companies, education
2. **Current role** — how long, what they're responsible for
3. **Recent public activity** — articles written, talks given, social media, interviews
4. **Areas of expertise** — what do they publicly know a lot about?
5. **Mutual connections** — any overlap with [your network or company]?
6. **Recent wins** — any company news, product launches, or announcements under their tenure

What to check:
- LinkedIn (fetch their profile page)
- Twitter/X (recent tweets if public)
- Company website (team/bio page)
- Google News: "[Name]" "[Company]"
- Any personal blog or Substack

Output format:
---
## [NAME] — [TITLE]

**In one sentence:** [Who they are and what they do]

**Background:** [2-3 sentences on career path]

**Currently focused on:** [What matters to them right now]

**Recent activity:** [Last notable thing they published/said/did]

**Smart opener:** [One specific thing to reference that shows you did your research]

**Potential common ground:** [Shared interests, mutual connections, overlapping experience]

**Note gaps:** Any important info not found.
---
```

---

## Company Research Template

### Research Prompt (per company)

```
Research [COMPANY NAME].

Find:
1. **What they do** — product/service and target customer
2. **Stage and scale** — founding year, funding, headcount estimate, revenue signals
3. **Recent news** — last 30-60 days (funding, launches, hires, press)
4. **Business model** — how do they make money?
5. **Competitive landscape** — who do they compete with?
6. **Pain points** — what challenges do companies like this face?
7. **Strategic priorities** — where are they investing? (signals from job postings, blog posts, announcements)

Sources to check:
- Company website (About, Blog, Careers)
- Crunchbase (funding data)
- LinkedIn (headcount, recent hires)
- Google News (recent press)
- Twitter/X (official account)
- G2 or Capterra (customer reviews if B2B SaaS)

Output format:
---
## [COMPANY NAME]

**One-liner:** [What they do in plain English]

**Stage:** [Seed / Series A-C / Late Stage / Public / Bootstrapped] — [founding year]

**Scale:** ~[X] employees | [Funding amount or "bootstrapped"]

**What they're working on:** [Key product or initiative right now]

**Recent news:** [1-2 most notable items from last 60 days]

**Strategic priority:** [Where they're clearly investing based on signals]

**Their challenges:** [What companies at this stage typically struggle with]

**Why this meeting matters to them:** [Speculate based on what you know about why they agreed to meet]
---
```

---

## Meeting Brief Format

The final output. Should fit on one page:

```markdown
# Meeting Brief: [MEETING TITLE]

**Date:** [DATE] at [TIME]
**Duration:** [LENGTH]
**Your Goal:** [What you want from this meeting]

---

## Who You're Meeting

### [Person 1] — [Title]
[2-sentence background. One specific recent thing to reference.]
**Smart opener:** "[Specific reference to something they've done recently]"

### [Person 2] — [Title]
[2-sentence background. One specific recent thing to reference.]
**Smart opener:** "[Specific reference]"

---

## Company Context

[3-4 sentences on the company. Stage, what they're focused on, recent news.]

**Relevant for you:** [Why their situation is relevant to your agenda]

---

## Talking Points

1. [Point 1 — specific to their situation, not generic]
2. [Point 2 — connects your offering to their current priority]
3. [Point 3 — addresses a likely concern or objection]

---

## Questions to Ask

1. [Open-ended question that shows you've done your research]
2. [Question about their strategic priority that you've identified]
3. [Question that reveals their decision process or timeline]
4. [Question that surfaces a pain point you can potentially address]

---

## What to Listen For

- [Signal 1 — if they say this, it means X]
- [Signal 2 — this tells you about their readiness]
- [Red flag — if they mention this, be cautious]

---

## Your Prep Checklist

- [ ] Know their name pronunciation (if unusual)
- [ ] Check LinkedIn one more time the morning of
- [ ] Know the specific company news from last 30 days
- [ ] Have your 1-sentence intro ready
- [ ] Prepare your "what brought you here" story
```

---

## Talking Points Generator

Use this to go deeper on talking points:

```
Generate talking points for my meeting with [PERSON] at [COMPANY].

My goal: [What I'm trying to accomplish]
My offering: [What I do / sell / provide]
Their situation: [What I know about where they are]

Create 3-5 talking points that:
1. Connect my offering to THEIR specific situation (not generic benefits)
2. Reference something specific about THEM (shows homework)
3. Are framed as their win, not my pitch

Format each as:
- **Point:** [The talking point]
- **Their perspective:** [Why this matters to them]
- **Evidence/example:** [Specific thing to mention]
```

---

## Questions-to-Ask Framework

The best meeting questions show research and create dialogue:

### Discovery Questions (understand their situation)
```
Ask me 5 discovery questions for a meeting with [ROLE] at a [COMPANY TYPE/STAGE] company.

Goal: understand their [pain point / buying process / priorities / decision criteria]

Questions should:
- Be open-ended (start with "How", "What", "Tell me about")
- Show that I know their context (not beginner questions)
- Give them room to reveal what matters most to them
- Not be answerable with yes/no
```

### Strategic Questions (show high-level thinking)

These go deeper:
- "What would need to be true for this to be a priority for you this quarter?"
- "How are you thinking about [specific industry trend] affecting your team?"
- "What's the biggest constraint you're working against right now?"
- "If this meeting goes perfectly, what does the next step look like for you?"

### Close Questions (advance the meeting)
- "What would you need to see to move forward?"
- "Who else needs to be in this conversation?"
- "What's your timeline for making a decision?"
- "What would make this a win for you?"

---

## Pre-Meeting Checklist

Run this 30 minutes before any meeting:

```
Pre-meeting check for [MEETING TITLE] in 30 minutes.

1. Read the meeting brief I prepared
2. Google "[Person Name]" + "[Company]" for anything from last 24h
3. Check Twitter/X for any recent posts from attendees
4. Confirm my 3 talking points are still relevant
5. Prepare my intro: name + what I do in 15 seconds
6. What's the single most important thing I want them to walk away knowing?
7. What's the specific next step I want to propose?

Brief me in 30 seconds on anything I might have missed.
```

---

## Troubleshooting

**Can't find information on the person**
- Try name + company + city in Google
- Search LinkedIn directly (some profiles are private but title visible)
- Check if they've spoken at conferences: "[Name] speaker" site:youtube.com
- Mark gaps as [NOT FOUND] — don't guess

**Company information is outdated**
- Startups change fast — prioritize news from last 90 days
- Check recent job postings for strategic signals
- Look for the company's blog or changelog for product updates

**Meeting brief is too generic**
- Add more specific context in your prompt ("I'm trying to [specific goal]")
- Explicitly ask for company-specific talking points, not industry platitudes
- Give the agent your own background too — helps tailor the positioning

**Too much information for the brief**
- Constrain the output: "Keep the full brief under 400 words"
- Ask for the single most useful insight per person, not a full profile
- Prioritize what's useful in the room, not everything you could know

---

*Built on OpenClaw. No external tools required beyond web search capability.*
