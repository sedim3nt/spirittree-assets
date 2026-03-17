# Blog Intel — RSS & Feed Monitoring System

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Monitor any blog, RSS feed, or publication for new posts. Auto-summarize, extract key takeaways, and queue content ideas from what your competitors and thought leaders publish. Built on blogwatcher CLI.

---

## What's In This Package

- blogwatcher CLI setup
- RSS feed configuration templates
- Auto-summary prompts
- Content idea extraction
- Source ranking
- Cron configuration for daily/6h checks
- Telegram delivery config

---

## blogwatcher CLI Setup

### Install

```bash
npm install -g blogwatcher
# or
npx blogwatcher --help
```

### Configure feeds

Create `~/.blogwatcher/config.json`:

```json
{
  "feeds": [
    {
      "name": "Paul Graham",
      "url": "http://paulgraham.com/rss.html",
      "category": "essays",
      "priority": "high"
    },
    {
      "name": "Hacker News Best",
      "url": "https://hnrss.org/best",
      "category": "tech",
      "priority": "high"
    },
    {
      "name": "Your Competitor Blog",
      "url": "https://competitor.com/blog/rss",
      "category": "competitor",
      "priority": "medium"
    }
  ],
  "storage": "~/.blogwatcher/seen.json",
  "maxAge": 7
}
```

### Test a feed

```bash
blogwatcher check --feed "https://paulgraham.com/rss.html"
blogwatcher list  # Show all new posts
```

---

## Feed Configuration Templates

### Template: Tech / Indie Hacker Feeds

```json
{
  "feeds": [
    {"name": "Hacker News Best", "url": "https://hnrss.org/best", "category": "tech"},
    {"name": "Hacker News Ask", "url": "https://hnrss.org/ask", "category": "tech"},
    {"name": "Indie Hackers", "url": "https://feeds.feedburner.com/IndieHackers", "category": "business"},
    {"name": "Paul Graham", "url": "http://paulgraham.com/rss.html", "category": "essays"},
    {"name": "Wait But Why", "url": "https://waitbutwhy.com/feed", "category": "essays"},
    {"name": "Stratechery (free)", "url": "https://stratechery.com/feed/", "category": "tech-strategy"}
  ]
}
```

### Template: AI / ML Feeds

```json
{
  "feeds": [
    {"name": "Anthropic Blog", "url": "https://www.anthropic.com/news/rss.xml", "category": "ai"},
    {"name": "OpenAI Blog", "url": "https://openai.com/blog/rss.xml", "category": "ai"},
    {"name": "Google AI Blog", "url": "https://ai.googleblog.com/feeds/posts/default", "category": "ai"},
    {"name": "The Batch (deeplearning.ai)", "url": "https://www.deeplearning.ai/the-batch/feed/", "category": "ai"},
    {"name": "Simon Willison", "url": "https://simonwillison.net/atom/everything/", "category": "ai"}
  ]
}
```

### Template: Business / Finance

```json
{
  "feeds": [
    {"name": "Seth Godin", "url": "https://seths.blog/feed/", "category": "marketing"},
    {"name": "Axios Business", "url": "https://api.axios.com/feed/", "category": "business"},
    {"name": "The Information (free)", "url": "https://www.theinformation.com/feed", "category": "tech-business"},
    {"name": "CB Insights Newsletter", "url": "https://www.cbinsights.com/research/feed/", "category": "startup"}
  ]
}
```

### Template: Your Niche (customize)

```json
{
  "feeds": [
    {
      "name": "[Blog Name]",
      "url": "[RSS URL]",
      "category": "[category]",
      "priority": "high",
      "keywords": ["keyword1", "keyword2"]
    }
  ]
}
```

**Finding RSS URLs:**
- Most blogs: `[domain.com]/feed` or `[domain.com]/rss`
- Medium: `https://medium.com/feed/@authorhandle`
- Substack: `https://[publication].substack.com/feed`
- YouTube channel: `https://www.youtube.com/feeds/videos.xml?channel_id=[CHANNEL_ID]`
- Reddit: `https://reddit.com/r/[subreddit].rss`

---

## Auto-Summary Prompts

### Daily Feed Summary Prompt

```
Check blogwatcher for new posts. Run: `blogwatcher list`

For each new post since last check:
1. Fetch the article: `web_fetch [URL]`
2. Extract:
   - Core thesis (1 sentence)
   - 3 key points
   - Any data or stats worth noting
   - Quotes worth saving

Format:
---
**[Blog Name]** — [Post Title]
[1-sentence thesis]
Key points:
- [Point 1]
- [Point 2]
- [Point 3]
Link: [URL]
---

Write full report to research/feeds/[DATE].md
Send Telegram summary: X new posts. Most interesting: [title]
```

### Competitor Blog Alert

When a competitor publishes:

```
New post from competitor blog: [URL]

Full analysis:
1. What are they announcing/claiming?
2. What's the argument or angle?
3. What does this signal about their strategy?
4. What do their readers now know that they didn't?
5. How should we respond? (Ignore / Rebut / Build on / Emulate)

This is time-sensitive — competitors get mind-share from content.

Alert: HIGH PRIORITY if competitor is making a direct claim about our space.
```

---

## Content Idea Extraction

When you find an interesting article, extract ideas for your own content:

```
Extract content ideas from: [URL]

Read the article. Then:

1. **Agree / Build** — What could we expand on from our perspective?
   Angle: "We've seen this too, but also [our experience]"

2. **Disagree** — What's the contrarian take?
   Angle: "Everyone's saying X, but actually..."

3. **Apply to our audience** — How does this topic apply specifically to [your niche]?

4. **Deeper dive** — What did they scratch the surface on that deserves full treatment?

5. **Question it raised** — What unanswered question does this article create?

For each idea: assign a content type (essay / how-to / thread / case study) and estimated effort (low/medium/high).

Add to content/ideas.md
```

### Weekly Content Ideas Digest

```
Content ideas from this week's feeds.

Read research/feeds/[THIS WEEK'S DATES].md

Synthesize:
1. What themes came up across multiple publications? (Trending topics)
2. What's being under-covered that we could own?
3. What formats are performing well? (Long essay? Short take? Tutorial?)
4. Top 3 content opportunities this week

Write to content/ideas-[WEEK].md
```

---

## Source Ranking

Categorize your feeds by quality:

**Tier 1 — Primary Signal (check daily)**
- Personal blogs of known experts in your domain
- Official product blogs for tools you use
- High-conviction independent newsletters

**Tier 2 — Good Aggregators (check every 2-3 days)**
- Hacker News best (community-filtered)
- Curated newsletters (Morning Brew, etc.)
- Top subreddits in your domain

**Tier 3 — Volume Sources (weekly digest)**
- Major tech publications (TechCrunch, Wired)
- SEO-heavy content farms (verify before using)
- Social media aggregators

**Tier 4 — Monitor But Skeptical**
- Competitor blogs (high relevance, potential bias)
- PR-driven press releases
- Anything with aggressive sponsored content

---

## Cron Configuration

### 6-Hour Check

For high-priority competitive feeds:

```
0 */6 * * * openclaw cron --message "Check blogwatcher for new posts from competitor feeds. Alert on any new competitor content. Summarize via Telegram."
```

### Daily Morning Check

For your primary reading:

```
0 7 * * * openclaw cron --message "Daily blog feed check. Run blogwatcher list. Summarize new posts from research/feeds/ config. Write to research/feeds/[today].md and send Telegram digest."
```

### Weekly Deep Read

For lower-priority sources:

```
0 9 * * 1 openclaw cron --message "Weekly feed digest. Check all secondary and tier-3 sources from blogwatcher. Extract top 5 most interesting pieces. Identify content ideas. Write to research/feeds/weekly-[date].md"
```

### LaunchAgent (6h Feed Check)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.openclaw.blog-intel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Check blogwatcher for new posts. Summarize and send Telegram alert for any competitor content or high-priority articles.</string>
    </array>
    <key>StartInterval</key><integer>21600</integer>
    <key>StandardOutPath</key><string>/tmp/blog-intel.log</string>
</dict>
</plist>
```

---

## Telegram Delivery Config

### Short Alert Format

For Telegram notifications:

```
📰 [N] new posts — [DATE]

🔥 Top picks:
1. [Blog]: [Post title] — [1-sentence summary]
2. [Blog]: [Post title] — [1-sentence summary]

⚠️ Competitor alert: [if any competitor posted]

Full digest: research/feeds/[DATE].md
```

### Competitor-Only Alert

If a monitored competitor posts, send immediately:

```
🚨 Competitor post: [Blog Name]
"[Post Title]"
[1-sentence summary of what they're saying]
[URL]

Action needed? [Yes — [reason] / No — filing for content ideas]
```

---

## Troubleshooting

**RSS feed URL not working**
- Try common patterns: `/feed`, `/rss`, `/feed.xml`, `/atom.xml`
- Use a feed validator: feedvalidator.org
- Some sites require user-agent headers — try `web_fetch [url]` directly

**blogwatcher not finding new posts**
- Check `~/.blogwatcher/seen.json` — may be marking everything as seen
- Try: `blogwatcher check --since "1 day ago"`
- Reset seen posts for a specific feed: delete its entry from seen.json

**Too many posts, not enough time to read**
- Reduce to 5-10 highest-quality sources
- Add keyword filtering — only alert if post mentions [your keywords]
- Move low-signal sources to weekly digest

**Content ideas not relevant**
- Add more context in your extraction prompt: "I create content for [audience] about [topics]"
- Tell the agent what you've already covered to avoid repeats
- Focus extraction on what's genuinely novel, not comprehensive

---

*Built on OpenClaw and blogwatcher CLI. Requires blogwatcher installed.*
