# The Research Analyst — AI Research Agent Package

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Your AI researcher that never sleeps. Monitors sources, synthesizes web research, tracks YouTube channels, and delivers briefings on the topics you care about. Built on the Rowan research agent architecture.

---

## What's In This Package

- `ROLE.md` for your research agent
- Daily briefing cron template
- YouTube channel monitor setup
- Web research workflow
- Source synthesis framework
- Topic configuration guide

---

## ROLE.md — Research Agent

Copy to your OpenClaw workspace as `agents/research/ROLE.md`:

```markdown
# ROLE.md — Research Agent

**Role:** Research Agent
**Model:** Claude Sonnet (recommended)
**Channel:** Sub-agent or direct

## Responsibilities
- Web search and source analysis
- Knowledge extraction and synthesis
- Competitive intelligence
- Market research
- Technology evaluation
- Daily briefings on configured topics

## Research Process
1. Define research scope and key questions
2. Search multiple sources (web_search, web_fetch, docs)
3. Cross-reference and validate findings
4. Synthesize into actionable brief
5. Write results to workspace files

## Output Format
- Always write findings to files (not just chat)
- Include sources and confidence levels
- Separate facts from analysis
- Highlight actionable insights
- Flag conflicting information

## Constraints
- Never fabricate sources or citations
- Mark speculation clearly with [ANALYSIS] tag
- Prefer primary sources over summaries
- Note publication date for time-sensitive info
- When uncertain, say so

## Daily Briefing Format
1. **Top Stories** — 3-5 news items with 1-sentence summaries
2. **Key Insights** — what this means for [configured domain]
3. **Source Links** — for deeper reading
4. **Action Items** — anything requiring a response or decision
```

---

## Topic Configuration

Create `research/topics.md` in your workspace:

```markdown
# Research Topics

## Primary Topics
- [Your main industry/domain]
- [Technology area you follow]
- [Competitor names or products]

## Secondary Topics
- [Adjacent areas of interest]
- [General tech/business trends]

## YouTube Channels to Monitor
- [Channel 1 name and URL]
- [Channel 2 name and URL]
- [Channel 3 name and URL]

## Keywords to Track
- "[Product or company name]"
- "[Industry term]"
- "[Competitor name]"

## Sources to Prioritize
- [Domain: e.g., techcrunch.com, reuters.com]
- [Newsletter or blog URL]
- [Specific author or handle]
```

---

## Daily Briefing Cron

Schedule: Daily at 6:30 AM (`30 6 * * *`)

### Cron Message Prompt

```
Morning research brief. Read research/topics.md for configured topics.

For each primary topic:
1. Search for news from the last 24 hours
2. Check for relevant YouTube videos published this week
3. Identify the 1-3 most significant developments

Deliver a briefing with:
- **Headline** — most important development across all topics
- **By Topic** — 1-2 sentence summary per primary topic with source links
- **YouTube Roundup** — any notable videos from monitored channels
- **Signal/Noise** — one thing worth going deeper on vs. one thing that's hype

Write full report to research/daily/[DATE].md
Send Telegram summary (under 200 words)
```

### LaunchAgent plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.research-brief</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Run daily research briefing per research/topics.md</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>6</integer>
        <key>Minute</key>
        <integer>30</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/research-brief.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/research-brief.err</string>
</dict>
</plist>
```

---

## YouTube Monitor Setup

### Configure channels to monitor

Add to `research/youtube-channels.md`:

```markdown
# YouTube Channels

## Daily Check
- [Channel 1]: https://youtube.com/@channel1
- [Channel 2]: https://youtube.com/@channel2

## Weekly Check
- [Channel 3]: https://youtube.com/@channel3
- [Channel 4]: https://youtube.com/@channel4

## Trigger Keywords
Only alert if video title/description contains:
- "[keyword 1]"
- "[keyword 2]"
- "[topic area]"
```

### YouTube Monitor Prompt

```
Check YouTube channels in research/youtube-channels.md.

For each channel in the Daily Check list:
1. Fetch the channel's recent videos page
2. Find videos published in the last 48 hours
3. Check if any match the trigger keywords

If matches found:
- Extract video title, URL, and publish date
- Fetch transcript if available
- Summarize in 2-3 sentences

Output: list of relevant videos with summaries. If nothing new, say so.
```

### YouTube Monitor Cron (6h interval)

Use `crontab -e` for every 6 hours:
```
0 */6 * * * openclaw message "Check YouTube channels per research/youtube-channels.md and alert on new relevant videos"
```

---

## Web Research Workflow

### Deep Research Prompt Template

Use this when you need thorough research on a specific topic:

```
Deep research request: [TOPIC]

Research questions to answer:
1. [Question 1]
2. [Question 2]
3. [Question 3]

Research constraints:
- Focus on: [specific angle or timeframe]
- Ignore: [what's not relevant]
- Prioritize sources: [academic / news / practitioner / all]

Output format:
1. Executive summary (100 words)
2. Key findings (with sources)
3. Conflicting information (if any)
4. Confidence level: HIGH / MEDIUM / LOW with rationale
5. Recommended next steps

Write to research/[slug].md
```

### Competitive Research Prompt

```
Research [COMPANY/PRODUCT]:

1. **Overview** — what they do, who they serve, revenue model
2. **Recent news** — last 30 days (product launches, funding, hires)
3. **Product analysis** — features, pricing, positioning
4. **Strengths** — what they do well
5. **Weaknesses** — where they're vulnerable
6. **Customer sentiment** — what users say (Reddit, reviews, Twitter)
7. **Comparison to us** — where we overlap, where we differ

Sources to check: company website, Crunchbase, LinkedIn, G2/Capterra, Reddit, Twitter/X, recent press.

Write to research/competitive/[company].md
```

---

## Source Synthesis Framework

When you have multiple sources on the same topic, use this prompt:

```
Synthesize these sources on [TOPIC]:

[SOURCE 1 URL or content]
[SOURCE 2 URL or content]
[SOURCE 3 URL or content]

Synthesis process:
1. Find the core claim each source makes
2. Note where they agree
3. Note where they contradict — and why (different timeframes? different data?)
4. Identify what's missing from all sources
5. Form a synthesized view with confidence level

Label each claim:
- ✅ CONSENSUS — multiple sources agree
- ⚠️ DISPUTED — sources conflict
- 🔍 UNVERIFIED — single source, needs confirmation
- 💡 ANALYSIS — your interpretation, not direct sourcing
```

---

## Research File Structure

Organize your research output:

```
workspace/
├── research/
│   ├── topics.md              # Your configured topics
│   ├── youtube-channels.md    # Channels to monitor
│   ├── daily/
│   │   ├── 2024-01-15.md     # Daily briefings
│   │   └── 2024-01-16.md
│   ├── competitive/
│   │   ├── competitor-a.md    # Competitive profiles
│   │   └── competitor-b.md
│   ├── deep/
│   │   └── topic-name.md      # Deep research on specific topics
│   └── sources.md             # Curated source list with quality ratings
```

---

## Troubleshooting

**Daily brief is too generic**
- Add more specific topics and keywords to `topics.md`
- Include competitor names and specific product names
- Add trigger keywords that narrow down what's relevant

**YouTube monitor missing videos**
- Check channel URLs are correct (use @handle format)
- Some channels require direct page fetching: `https://youtube.com/@channelname/videos`
- Try fetching manually first: ask agent to "fetch https://youtube.com/@channelname/videos"

**Research quality is shallow**
- Break complex topics into specific questions
- Ask for primary sources, not just summaries
- Use the deep research prompt for important topics
- Tell the agent what you already know to avoid retreading covered ground

**Sources are outdated**
- Add "from the last [7 days / 30 days]" to prompts
- Ask the agent to check publication dates and flag old content
- Use `freshness` parameter in web_search calls

**Too much noise, not enough signal**
- Reduce number of topics
- Add explicit "ignore" rules to topics.md
- Ask for the single most important thing instead of comprehensive coverage

---

*Built on OpenClaw. Requires OpenClaw installed and configured.*
