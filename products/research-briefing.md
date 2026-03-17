# Research Briefing System — Daily Intelligence Automation

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Full daily briefing system. Script docs, topic templates, YouTube monitor, and cron configuration. Wake up to relevant intelligence, not noise.

---

## What's In This Package

- Daily briefing workflow (configurable topics)
- Topic templates for 8 domains
- YouTube monitor setup
- Source quality guide
- Briefing format templates
- Cron configuration

---

## System Overview

The briefing system has three components:

1. **Topic Config** — what you want to know about
2. **Briefing Generator** — the agent that runs the research
3. **Delivery** — cron schedule + Telegram/email delivery

---

## Daily Briefing Prompt

This is the core prompt. Customize the topics:

```
Daily briefing. Run research for all topics in research/topics.md.

For each PRIMARY topic:
1. Search web for news from last 24 hours
2. Note the most significant development
3. Summarize in 2-3 sentences with source link

For YOUTUBE CHANNELS:
1. Check each channel in research/youtube-channels.md
2. Flag any videos published in last 48 hours
3. If trigger keywords match, include title + link + 1-sentence summary

BRIEFING FORMAT:
---
## 📰 [DATE] — Daily Brief

### Headline
[Most important thing across all topics today in 1 sentence]

### [Topic 1]
[2-3 sentence summary] — [Source link]

### [Topic 2]
[2-3 sentence summary] — [Source link]

### YouTube Roundup
- [Channel]: "[Video Title]" — [1-sentence summary] [URL]
  (or: No new videos matching keywords)

### Signal vs. Noise
- Worth going deeper: [One thing with a reason why]
- Hype to ignore: [One thing with a reason why]
---

Write full brief to research/daily/[DATE].md
Send compressed Telegram version (under 200 words)
```

---

## Topic Templates

Copy and customize `research/topics.md`:

### Template A: Solo Operator / Creator

```markdown
# Research Topics

## Primary Topics (Daily)
- AI tools and agent frameworks
- Creator economy news
- Solopreneur case studies and tools

## Secondary Topics (Weekly)
- Platform changes (Substack, X, YouTube, LinkedIn)
- Indie hacker revenue reports

## YouTube Channels
Daily Check:
- indie hackers: https://youtube.com/@indiehackers
- [Your favorite creator channels]

Trigger Keywords:
- "AI agent"
- "solopreneur"
- "automated"
- "passive income"

## Sources to Prioritize
- indiehackers.com
- substack.com/discover
- hackernews (news.ycombinator.com)
```

### Template B: Tech / Engineering

```markdown
# Research Topics

## Primary Topics (Daily)
- Software engineering trends
- AI/ML announcements (OpenAI, Anthropic, Google)
- Developer tools and frameworks

## Secondary Topics (Weekly)
- Startup fundraising news (relevant to tech)
- Open source notable releases

## YouTube Channels
Daily Check:
- [Your favorite tech channels]

Trigger Keywords:
- "new release"
- "open source"
- "[languages/frameworks you use]"

## Sources to Prioritize
- github.com/trending
- news.ycombinator.com
- techcrunch.com
```

### Template C: Finance / Investing

```markdown
# Research Topics

## Primary Topics (Daily)
- US equity markets (S&P, Nasdaq)
- Crypto (Bitcoin, Ethereum)
- Fed policy / macro indicators

## Secondary Topics (Weekly)
- Earnings this week
- Emerging market signals

## YouTube Channels
Daily Check:
- [Finance YouTube channels you follow]

Trigger Keywords:
- "rate decision"
- "earnings beat"
- "inflation"
- "recession"

## Sources to Prioritize
- federalreserve.gov
- bloomberg.com (headlines)
- wsj.com
- coindesk.com
```

### Template D: Content Creator / Media

```markdown
# Research Topics

## Primary Topics (Daily)
- Your niche topics (replace with specifics)
- Trending topics on your platforms
- Competitor content performance

## YouTube Channels
Daily Check:
- [Top 5 channels in your niche]

Trigger Keywords:
- "[your niche keywords]"
- "viral"
- "[specific terms your audience cares about]"
```

### Template E: B2B Sales / GTM

```markdown
# Research Topics

## Primary Topics (Daily)
- [Your target industry] news
- Competitor activity
- Buyer persona news (what affects your customers)

## Secondary Topics (Weekly)
- Sales methodology updates
- CRM and GTM tool changes

## Trigger Keywords
- "[Company names to watch]"
- "[Product category]"
- "funding" + [industry vertical]
```

### Template F: Health & Wellness

```markdown
# Research Topics

## Primary Topics (Daily/Weekly)
- Evidence-based nutrition research
- Exercise science developments
- Sleep and recovery research

## Secondary Topics (Monthly)
- Clinical trial results
- Supplement research updates

## Sources to Prioritize
- pubmed.ncbi.nlm.nih.gov
- examine.com
- hubermanlab.com (transcripts/notes)
```

### Template G: Web3 / Crypto

```markdown
# Research Topics

## Primary Topics (Daily)
- Bitcoin and Ethereum price action + news
- DeFi TVL and protocol news
- Governance proposals (your DAOs)

## Secondary Topics (Weekly)
- Layer 2 development updates
- NFT market signals
- Regulatory news

## Sources to Prioritize
- defillama.com
- dune.com
- theblock.co
- cryptoslate.com
```

### Template H: Policy / Civic Tech

```markdown
# Research Topics

## Primary Topics (Weekly)
- [Your city/state] legislation
- Tech policy and AI regulation
- Municipal governance updates

## Secondary Topics (Monthly)
- Related research and reports
- Academic publications in your domain

## Sources to Prioritize
- your-city-government.gov
- congress.gov
- techpolicy.press
```

---

## YouTube Monitor Setup

### Channel Configuration File

Create `research/youtube-channels.md`:

```markdown
# YouTube Channels

## Daily Check
Add channels to check every 12-24 hours:
- [Channel name]: https://youtube.com/@handle
- [Channel name]: https://youtube.com/@handle

## Weekly Check
Channels that post less frequently:
- [Channel name]: https://youtube.com/@handle

## Trigger Keywords
Only alert if title or description contains:
- "[keyword 1]"
- "[keyword 2]"
- "[keyword 3]"

## Archive (Not Currently Monitoring)
- [Channel]: Paused — [reason]
```

### Monitor Prompt

```
YouTube channel check. Read research/youtube-channels.md.

For DAILY CHECK channels:
1. Fetch [channel_url]/videos
2. Find videos published in last 48 hours
3. Check titles/descriptions against trigger keywords

For WEEKLY CHECK channels:
1. Same process but check last 7 days

Report:
- NEW (last 48h): [title] — [channel] — [published date] — [url]
  [1-sentence summary if keyword match]
- TRENDING (>10K views, last 7 days): [title] — [views] — [url]

If nothing new: "No new videos matching keywords" — don't fabricate

Write to research/youtube/[DATE].md
```

---

## Source Quality Guide

Rate your sources to configure research priorities:

### Tier 1 — Primary Sources (highest signal)
These are authoritative and direct:
- Official government data (BLS, FRED, SEC EDGAR)
- Company press releases and IR pages
- Academic preprints (arxiv.org, pubmed)
- Official product changelogs

### Tier 2 — Quality Journalism (good synthesis)
- Reuters, AP, Bloomberg
- WSJ, Financial Times, The Economist
- Niche trade publications in your domain
- Quality newsletters by known authors

### Tier 3 — Aggregators (useful but verify)
- Hacker News (vote-ranked but varies)
- Reddit (sentiment, not facts)
- Twitter/X (breaking news, verify before trusting)
- LinkedIn (professional signal, often promotional)

### Tier 4 — Use Sparingly
- SEO content farms
- Anonymous or unverified sources
- Anything without a publication date
- Content that cites other content (not primary)

**Agent instruction:** When uncertain about source quality, state it. "This comes from [Tier 3 source] — verify before acting on it."

---

## Briefing Format Variants

### Short Form (Telegram, under 200 words)

```
📊 [DATE] Brief

🔑 Headline: [1 sentence]

📌 [Topic 1]: [1-2 sentences]
📌 [Topic 2]: [1-2 sentences]
📌 [Topic 3]: [1-2 sentences]

📺 YouTube: [Video title] — [Channel] (or: nothing new)

⚡ Signal: [One thing worth going deeper]
```

### Long Form (Full report file)

```markdown
# Daily Brief — [DATE]

**Generated:** [TIME]

---

## Executive Summary
[2-3 sentences on the overall state of your topics today]

## [Topic 1]
**Headline:** [Most important development]
**Context:** [Why this matters]
**Source:** [Link + date]

## [Topic 2]
[Same structure]

## YouTube
[List of new/trending videos with summaries]

## Sources Checked
[List of sources searched]

## Confidence Notes
[Any data quality issues, outdated sources, gaps noted]
```

---

## Troubleshooting

**Briefing is too generic**
- Your topics.md needs more specificity — add company names, specific technologies
- Add keyword filters: "only include if mentions [X]"
- Ask for the contrarian view: "what are they getting wrong about [topic]?"

**Too much content, not enough signal**
- Reduce primary topics to 3 max
- Add "only include if it's actually novel — skip recurring non-stories"
- Ask for: signal/noise section explicitly

**YouTube fetching fails**
- YouTube pages are dynamically loaded — sometimes direct fetch fails
- Try the RSS feed URL: `https://www.youtube.com/feeds/videos.xml?channel_id=[ID]`
- Get channel ID: search YouTube → view page source → find `channelId`

**Research is always the same sources**
- Explicitly list different sources to check
- Ask the agent to "try 3 sources you haven't used this week"
- Add: "Do not use [domain] — I've seen that source exhausted"

---

*Built on OpenClaw. Requires OpenClaw installed and configured.*
