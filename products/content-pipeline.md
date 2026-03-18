# Content Pipeline Templates — Publish Across 5 Platforms on Autopilot

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 45-60 minutes

The complete content operations system extracted from a pipeline that published 6 Substack articles, 17 tweets, and 2 Facebook posts in a single day. Queue-based publishing, platform-specific templates, cron automation, and the 1-piece → 5-platform repurposing workflow.

---

## What's In This Package

- Queue-based publishing system with JSON templates (why queues beat posting on impulse)
- Platform templates: Substack, X/Twitter, LinkedIn, Facebook, Instagram
- Cron job configurations for automated posting schedules
- Editorial calendar template (structured JSON format)
- 1-piece → 5-platform content repurposing workflow
- Voice consistency framework (keep your AI writing like you)
- Analytics tracking template (what to measure, where)
- Distribution checklist (mandatory by content type)
- Cover image generation workflow
- Content QA checklist before publishing
- Anti-phantom-success rule: verify content actually landed

---

# Content Pipeline Templates
### The System That Publishes Across 5 Platforms While You Sleep

*By AgentOrchard — extracted from a pipeline that published 6 Substack articles, 17 tweets, and 2 Facebook posts in a single day*

---

## What You're Getting

This isn't a "how to use ChatGPT for content" guide. This is a complete content operations system — the same one running our publication right now.

**Included:**
- Queue-based publishing system (JSON templates)
- Platform-specific prompt templates (Substack, X/Twitter, LinkedIn, Facebook, Instagram)
- Cron job configurations for automated posting
- Editorial calendar template
- Content repurposing workflow (1 piece → 5 platforms)
- Voice consistency framework
- Analytics tracking template

---

## Part 1: The Queue System

### Why Queues Beat "Post When Inspired"

Most people create content and post it immediately. This is wrong for three reasons:
1. You post in bursts, then go silent for days
2. You can't review quality before it goes live
3. You have zero buffer for busy days

**The fix:** Create content in batches, queue it, and let automation post on schedule.

### Tweet Queue Template

```json
{
  "tweets": [
    {
      "text": "your tweet content here (max 280 chars)",
      "posted": false,
      "scheduledFor": "2026-03-10T09:00:00-07:00",
      "category": "insight",
      "thread": false,
      "createdAt": "2026-03-09T14:00:00-07:00"
    },
    {
      "text": "second tweet in queue",
      "posted": false,
      "scheduledFor": "2026-03-10T14:00:00-07:00",
      "category": "promotion",
      "thread": false,
      "createdAt": "2026-03-09T14:05:00-07:00"
    }
  ]
}
```

**Fields:**
- `text`: The post content
- `posted`: Flips to `true` after posting (cron reads this)
- `scheduledFor`: When to post (your cron checks this)
- `category`: Tag for analytics (insight, promotion, engagement, educational)
- `thread`: If true, this is part of a thread (array of texts)
- `createdAt`: When you wrote it (for tracking content velocity)

### Long-Form Queue Template (Substack, Blog)

```json
{
  "articles": [
    {
      "title": "Article Title Here",
      "slug": "article-title-here",
      "body_html": "<h2>Introduction</h2><p>Your content...</p>",
      "cover_image": "https://your-cdn.com/image.png",
      "subtitle": "The subtitle shown in previews",
      "status": "draft",
      "category": "ai-operations",
      "publishDate": "2026-03-11",
      "createdAt": "2026-03-09",
      "platforms": ["substack", "linkedin", "email"]
    }
  ]
}
```

---

## Part 2: The Batch Content Workflow

### The 1→5 Repurposing System

Write one substantial piece. Derive everything else from it.

```
Original Article (1,500-2,500 words)
  ├── Substack post (full article)
  ├── Twitter thread (5-8 tweets extracting key points)
  ├── LinkedIn post (300-500 words, professional angle)
  ├── Facebook post (150-200 words, conversational)
  └── Instagram carousel (5-7 slides with key takeaways)
```

### Repurposing Prompt Template

```
I wrote this article:

---
[paste your article]
---

Create the following from this article:

1. TWITTER THREAD (5-8 tweets):
- First tweet should hook with a surprising stat or bold claim
- Each tweet should stand alone (people see them individually)
- Last tweet should link back to the full article
- No hashtags except 1-2 on the last tweet
- Use line breaks for readability

2. LINKEDIN POST (300-500 words):
- Professional tone, first person
- Lead with a business insight, not a personal story
- Include 2-3 specific numbers or results
- End with a question to drive comments
- No hashtags in the body, 3-5 at the very end

3. FACEBOOK POST (150-200 words):
- Conversational, like talking to a friend
- Ask a question or share a realization
- No links in the post body (put link in comments)
- Use 1-2 emojis max

4. INSTAGRAM CAROUSEL OUTLINE (5-7 slides):
- Slide 1: Bold statement or question (hook)
- Slides 2-5: One key insight per slide, large text
- Slide 6: The "so what" — actionable takeaway
- Slide 7: CTA (follow for more, link in bio)
- Each slide: max 30 words

Voice guidelines: [describe your voice - direct, warm, technical, casual, etc.]
```

---

## Part 3: Platform-Specific Prompt Templates

### Substack Article Prompt

```
Write a Substack article for my publication [name].

Topic: [topic]
Angle: [what's different about your take]
Target reader: [describe them in one sentence]
Desired length: [1,500-2,500 words]

Structure:
1. Hook (first 2 sentences must stop the scroll)
2. The problem (why this matters right now)
3. The insight (your original take — not what everyone else says)
4. Evidence (specific examples, numbers, or experience)
5. The "so what" (what the reader should do differently)
6. CTA (subscribe, share, reply)

Voice: [your voice description]
Avoid: [clichés, phrases, topics to skip]

Important: 
- No listicles unless the topic demands it
- Specific > vague (use real numbers, real examples)
- Every paragraph must earn its place
- The reader should feel smarter after reading, not just entertained
```

### Twitter/X Post Prompt

```
Write a tweet about [topic].

Constraints:
- Max 280 characters
- Must be a complete thought (not clickbait that requires a thread)
- No hashtags unless they add meaning
- Use line breaks for readability
- Should make someone stop scrolling

Format preference: [choose one]
A) Bold claim → supporting evidence
B) Counterintuitive insight → explanation
C) Specific number → what it means
D) "Most people do X. The best do Y."
E) Story in 3 lines

My voice: [description]
```

### LinkedIn Post Prompt

```
Write a LinkedIn post about [topic].

Audience: [your LinkedIn audience]
Goal: [thought leadership / generate leads / drive discussion]
Length: 300-500 words

Structure:
- Hook line (stands alone, makes people click "see more")
- [blank line]
- 3-5 short paragraphs with insight
- [blank line]
- Question or CTA to drive engagement

Rules:
- First person, professional but not corporate
- Include at least one specific metric or result
- No "I'm thrilled to announce" energy
- No more than 5 hashtags, placed at the very end
- Line breaks between every 1-2 sentences
```

---

## Part 4: The Editorial Calendar

### Monthly Planning Template

| Week | Pillar Topic | Substack | Twitter (3x) | LinkedIn (2x) | Other |
|------|-------------|----------|-------------|---------------|-------|
| 1 | [Theme A] | Full article | 3 derived posts | 2 posts | IG carousel |
| 2 | [Theme B] | Full article | 3 derived posts | 2 posts | FB post |
| 3 | [Theme A deep-dive] | Full article | 3 derived posts | 2 posts | IG carousel |
| 4 | [Theme C or timely] | Full article | 3 derived posts | 2 posts | Newsletter recap |

### Content Pillar Framework

Pick 3-4 pillars that your publication covers. Every piece should fit one:

**Example (our pillars):**
1. **AI Operations** — How to run AI agents, practical guides
2. **AI Economy** — Labor markets, economic impact of AI
3. **Builder Dispatches** — What we're actually building, honest updates
4. **Macro/Markets** — Economic analysis (FOMC, liquidity, cycles)

**Your pillars:**
1. ____________________________
2. ____________________________
3. ____________________________
4. ____________________________

### Content Velocity Targets

| Stage | Target | Rationale |
|-------|--------|-----------|
| Starting (month 1-2) | 1 article/week + 3 social posts | Build the habit |
| Growing (month 3-6) | 2 articles/week + 5 social posts | Build the library |
| Cruising (month 6+) | 2-3 articles/week + daily social | Compound returns |

---

## Part 5: Voice Consistency Framework

### Define Your Voice (Fill In)

```
My writing voice is:
  Tone: [e.g., direct, warm but not soft, occasionally irreverent]
  Complexity: [e.g., explain simply but don't dumb down]
  Perspective: [e.g., practitioner, not theorist — I share what I build]
  Signature moves: [e.g., specific numbers, real examples, blunt assessments]
  Never: [e.g., corporate jargon, "I'm excited to share", passive voice]
```

### Voice Calibration Prompt

Use this before any content generation session:

```
Before writing, calibrate to this voice:

Tone: [your tone]
I sound like: [a person you admire + their style]
I never sound like: [what to avoid]

Example of MY writing at its best:
[paste 200-300 words of your actual writing that you're proud of]

Match this voice for everything you write in this session.
```

---

## Part 6: Automated Posting Cron Jobs

### The Posting Schedule

```
# Post first queued tweet at 9am MT
0 9 * * * /path/to/post-tweet.sh

# Post second tweet at 2pm MT  
0 14 * * * /path/to/post-tweet.sh

# Publish scheduled Substack at 7am MT on Tuesdays and Thursdays
0 7 * * 2,4 /path/to/publish-substack.sh

# Post to LinkedIn at 8am MT on Monday, Wednesday, Friday
0 8 * * 1,3,5 /path/to/post-linkedin.sh
```

### Simple Posting Script Template

```bash
#!/bin/bash
# post-tweet.sh — Post next queued tweet

QUEUE_FILE="/path/to/tweet-queue.json"

# Find first unposted tweet
TWEET=$(python3 -c "
import json
with open('$QUEUE_FILE') as f:
    data = json.load(f)
tweets = data.get('tweets', [])
for i, t in enumerate(tweets):
    if not t.get('posted'):
        print(t['text'])
        break
")

if [ -z "$TWEET" ]; then
    echo "No tweets in queue"
    exit 0
fi

# Post via API (replace with your platform's API)
# For X/Twitter via xurl:
# xurl post "$TWEET"

# For X/Twitter via API:
# curl -X POST "https://api.twitter.com/2/tweets" ...

# Mark as posted
python3 -c "
import json
with open('$QUEUE_FILE') as f:
    data = json.load(f)
for t in data.get('tweets', []):
    if not t.get('posted'):
        t['posted'] = True
        break
with open('$QUEUE_FILE', 'w') as f:
    json.dump(data, f, indent=2)
"

echo "Posted: $TWEET"
```

---

## Part 7: Analytics Tracking

### What to Track

| Metric | Platform | Why It Matters | Target |
|--------|----------|---------------|--------|
| Subscribers | Substack | Your owned audience | +10%/month |
| Open rate | Substack email | Content resonance | >40% |
| Impressions | Twitter/X | Reach | Growing trend |
| Engagement rate | All | Content quality signal | >2% |
| Click-through | All | Conversion indicator | >1% |
| Time to create | Internal | Efficiency | Decreasing |

### Weekly Tracking Template

```markdown
## Content Report — Week of [date]

### Published
- Substack: [X] articles, [Y] views, [Z] new subscribers
- Twitter: [X] posts, [Y] impressions, [Z] engagements
- LinkedIn: [X] posts, [Y] impressions, [Z] comments
- Other: [details]

### Queue Status
- Tweets queued: [X]
- Articles drafted: [X]
- Days of buffer: [X]

### Top Performer
- [Title] on [platform]: [metric]
- Why it worked: [analysis]

### Adjustments
- [What to change based on data]
```

---

## The 10-Minute Daily Routine

1. **Morning (5 min):** Review queue — are today's posts ready? Quick quality check.
2. **Evening (5 min):** Add 1-2 ideas to the queue. Not full posts — just the hooks/ideas.
3. **Weekly (60 min):** Batch-write the week's content. Use the repurposing system.
4. **Monthly (30 min):** Review analytics. Update editorial calendar. Adjust pillars if needed.

---

© 2026 AgentOrchard · agentorchard.dev
