# Competitive Intel Agent — AI Competitive Monitoring Package

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Automated competitor monitoring. YouTube trending videos, social activity, product launches, pricing changes, hiring signals. Daily competitive digest delivered to your Telegram. Built from real monitoring workflows used to track 8+ competitor channels.

---

## What's In This Package

- `ROLE.md` for your competitive intel agent
- YouTube competitor monitor setup
- Social media scanning templates
- Competitive digest cron
- Trending content detection framework
- Competitive profile templates

---

## ROLE.md — Competitive Intel Agent

Copy to `agents/intel/ROLE.md`:

```markdown
# ROLE.md — Competitive Intel Agent

**Role:** Competitive Intelligence Agent
**Model:** Claude Sonnet
**Channel:** Cron-driven + sub-agent

## Responsibilities
- Monitor competitor YouTube channels for new content
- Track social media activity and engagement
- Detect product launches, pricing changes, positioning shifts
- Monitor hiring signals on LinkedIn/job boards
- Compile weekly competitive digest

## Research Standards
- Note exact dates on all findings
- Distinguish observation from interpretation
- Flag when data is incomplete or inferred
- Track trends over time (compare to last week)

## Output Format
- Lead with what's NEW this week (not recurring status)
- Quantify when possible (views, engagement rate, price change %)
- Flag: HIGH PRIORITY items requiring immediate attention
- Tone: factual, not alarmist

## Constraints
- Monitor only publicly available information
- No scraping that violates ToS
- No impersonation of competitors or their customers
- Flag intel gaps honestly — better than fabricating
```

---

## Competitor Configuration

Create `intel/competitors.md`:

```markdown
# Competitor Configuration

## Primary Competitors (Daily Monitoring)

### [Competitor 1]
- Website: [URL]
- Twitter/X: [@handle]
- YouTube: [channel URL]
- LinkedIn: [company page URL]
- Product: [brief description]
- Price: [$X/month or pricing page URL]
- Key differentiator: [what they claim makes them unique]

### [Competitor 2]
- Website: [URL]
- Twitter/X: [@handle]
- YouTube: [channel URL]
- LinkedIn: [company page URL]
- Product: [brief description]
- Price: [$X/month or pricing page URL]
- Key differentiator:

## Secondary Competitors (Weekly Monitoring)

### [Competitor 3]
[same structure]

## Adjacent Players to Watch
[Companies not direct competitors but moving toward your space]

## Trigger Alerts
Send immediate alert if any competitor:
- Announces pricing change
- Raises funding
- Launches new product/feature
- Goes viral (>50K views on a single piece)
- Mentions us by name

## Our Positioning vs. Competitors
[Brief notes on how we're different — helps contextualize intel]
```

---

## YouTube Competitor Monitor

### Monitor Prompt

Run every 12 hours or daily:

```
Check YouTube channels for all Primary Competitors in intel/competitors.md.

For each channel:
1. Fetch the channel's /videos page
2. Find videos published in the last 48 hours
3. Note view count on videos from the last 7 days (trending detection)

Report:
- New videos: title, publish date, early view count
- Trending older videos: >10K views, published last 30 days
- Notable topics covered this week

Flag if any competitor:
- Posted more than 2 videos this week (unusual volume)
- Got >50K views on a video (viral detection)
- Changed their content focus/topic area

Write full report to intel/youtube/[DATE].md
Send Telegram summary (under 150 words)
```

### Trending Content Detection

To find what's resonating in your space:

```
Trend detection for [YOUR CATEGORY/NICHE]:

Search YouTube for:
- "[primary keyword]" — sort by this week
- "[competitor brand]" — recent videos
- "[industry topic]" — past month

From results, identify:
1. What formats are getting most views (tutorial / opinion / case study)
2. What angles are trending (cost-cutting? AI? automation?)
3. Any topic that multiple creators are covering simultaneously (emerging trend)

This is market signal — what does your audience want to learn right now?

Write to intel/trends/[DATE].md
```

---

## Social Media Scanning

### Daily Social Scan Prompt

```
Social media competitive scan. Check intel/competitors.md for competitor handles.

For each primary competitor on Twitter/X:
1. Fetch their recent tweets (last 24-48h)
2. Note: what topics are they talking about?
3. Flag: anything with unusually high engagement

Specifically look for:
- Product announcements or teasers
- Customer testimonials they're amplifying
- Complaints they're responding to (signals product gaps)
- Any mentions of pricing, new features, partnerships
- Attacks on competitors (sometimes they name us)

Engagement baseline: flag anything 3x higher than their average.

Write to intel/social/[DATE].md
Send Telegram summary of any flagged items.
```

### Competitive Listening Prompt

Find what customers say about competitors:

```
Customer sentiment scan for [COMPETITOR]:

Search for recent mentions on:
1. Twitter/X: "[competitor name]" filter:replies -from:[competitor_handle]
2. Reddit: site:reddit.com "[competitor name]"
3. Review sites: "[competitor name] review"

Categorize findings:
- What do customers love?
- What do customers hate?
- What are common complaints?
- What do they wish the product did?

This is your product opportunity map. Where competitors fall short, we win.

Write to intel/sentiment/[competitor].md and note the date.
```

---

## Competitive Digest Cron

### Weekly Digest (Sunday 7:00 PM)

Schedule: `0 19 * * 0`

Prompt:
```
Weekly competitive digest. Read all intel files from the past 7 days:
- intel/youtube/*.md
- intel/social/*.md
- intel/sentiment/*.md
- intel/trends/*.md

Compile a weekly digest with:

1. **Biggest Move** — the single most notable competitor development this week

2. **YouTube Roundup** — most significant video content (with view counts)

3. **Social Signals** — top themes in competitor social activity

4. **Product Updates** — any launches, pricing changes, new features

5. **Customer Sentiment Shifts** — any notable shifts in what customers say

6. **Emerging Trends** — topics gaining traction in our space

7. **Opportunity Flags** — based on competitor gaps/complaints

8. **Recommended Actions** — 1-3 specific things we should do in response

Write to intel/weekly/[DATE].md
Send full digest via Telegram
```

### LaunchAgent plist (Weekly Digest)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.competitive-digest</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Run weekly competitive digest per intel/competitors.md — read all intel files from past 7 days</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>0</integer>
        <key>Hour</key>
        <integer>19</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/competitive-digest.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/competitive-digest.err</string>
</dict>
</plist>
```

---

## Competitive Profile Template

Maintain an updated profile for each major competitor:

Create `intel/profiles/[competitor].md`:

```markdown
# Competitive Profile: [COMPETITOR NAME]

**Last Updated:** [DATE]
**Category:** [Direct / Adjacent / Emerging]

## Overview
- **What they do:** [1-2 sentences]
- **Target customer:** [who they serve]
- **Business model:** [how they make money]
- **Founded:** [year]
- **Funding:** [amount / bootstrapped / public]

## Product
- **Core features:** [list]
- **Pricing:** [tiers and prices]
- **Recent changes:** [latest updates]
- **Tech stack:** [if known]

## Market Position
- **Positioning tagline:** [what they claim]
- **Key differentiators:** [their strengths]
- **Known weaknesses:** [from customer feedback]
- **ICP:** [their ideal customer profile]

## Content & Distribution
- **YouTube:** [channel size, posting frequency, content themes]
- **Twitter/X:** [follower count, posting frequency]
- **Newsletter/Blog:** [if applicable]
- **Top performing content:** [links to their best stuff]

## Customer Sentiment
- **Loves:** [what customers praise]
- **Hates:** [common complaints]
- **Switching reasons:** [why customers leave or come to us]

## Our Comparison
- **Where we win:** [our advantages]
- **Where they win:** [be honest]
- **Battlecard response:** [how to respond when prospects mention them]

## Change Log
- [DATE]: [what changed]
- [DATE]: [what changed]
```

---

## Hiring Signal Detection

Competitor hiring often signals strategy:

```
Hiring signal scan for competitors in intel/competitors.md.

For each primary competitor, search:
- LinkedIn jobs: "[company name]"
- Company careers page: [URL from competitor profile]
- Indeed / Glassdoor: "[company name]"

What to look for:
- New roles in areas they didn't hire before (signals new product area)
- Volume of hiring (growth signal)
- Seniority level (are they building or scaling?)
- Location/remote policy changes
- Technical roles (what stack are they investing in?)

Interpret:
- Hiring 5 ML engineers → building AI features
- Hiring sales team → moving upmarket
- Hiring support → dealing with scaling issues

Write to intel/hiring/[DATE].md and flag anything notable.
```

---

## Troubleshooting

**YouTube data not current**
- YouTube pages are public — fetching directly usually works
- Try: `https://youtube.com/@channelhandle/videos`
- If blocked, try the RSS feed: `https://www.youtube.com/feeds/videos.xml?channel_id=[CHANNEL_ID]`

**Social data incomplete**
- Twitter/X public data is rate-limited for unauthenticated access
- Use xurl if you have Twitter API access
- Supplement with manual checks during weekly review

**Too many false positives in trending detection**
- Add view count thresholds to your prompts
- Focus on percentage change vs. absolute numbers
- Compare to competitor's own historical average, not industry average

**Competitive digest is too long**
- Use the tiered priority system — lead with biggest move only
- Rest goes in the full report file, Telegram gets the summary
- Configure max word count in the prompt

**Competitor hired someone notable**
- LinkedIn search for the company, filter by Recently Added
- Watch for senior executives joining (CEO, CTO, VP Sales)
- These are leading indicators of strategic shifts

---

*Built on OpenClaw. Requires OpenClaw installed and configured.*
