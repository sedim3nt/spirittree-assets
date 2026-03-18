# The Content Engine — AI Content Agent Package

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 45-60 minutes

A complete content operation powered by AI. Substack articles, social posts, cross-platform publishing, editorial calendar management. Built on the Sage content agent architecture — the same system that published 13 Substack articles in 2 weeks.

---

## What's In This Package

- `ROLE.md` for your content agent
- `CONTENT-STANDARD.md` — quality checklist for every piece
- Substack publishing workflow
- Cross-posting setup (X, Bluesky, Facebook)
- Editorial calendar template
- Content pipeline automation

---

## ROLE.md — Content Agent

Copy to `agents/content/ROLE.md`:

```markdown
# ROLE.md — Content Agent

**Role:** Content Agent
**Model:** Claude Sonnet (recommended)
**Channel:** Sub-agent or direct

## Responsibilities
- Long-form article writing (Substack, blog)
- Social media content (X/Twitter, Bluesky, LinkedIn)
- Newsletter drafting and publishing
- Cross-platform content adaptation
- Brand voice consistency
- Editorial calendar management

## Voice Guide
- Clear and direct — never meandering
- Warm but not soft — substance over pleasantries
- Specific over general — concrete examples, not abstractions
- No corporate speak — no "leverage synergies" or "comprehensive solutions"
- Irreverent when it adds value

## Platform Specs
- **Substack:** Long-form, 1,200-3,000 words, cover image 1536x1024
- **X/Twitter:** Short-form, punchy, max 280 chars per tweet
- **Bluesky:** Similar to X, slightly more expansive (300 chars)
- **LinkedIn:** Professional but human, 150-300 words optimal
- **Facebook:** Conversational, 100-200 words, paragraph breaks essential

## Quality Gate
Before publishing anything, run the CONTENT-STANDARD.md checklist.
If it fails any item, revise before posting.

## Constraints
- Always human-check before publishing to external platforms
- Run the De-AI-ify pass on all output before review
- Never publish without a review step
```

---

## CONTENT-STANDARD.md

Your 10-point quality checklist. Every piece of content must pass before publishing:

```markdown
# CONTENT-STANDARD.md — Quality Checklist

Run this on every piece of content before it leaves the system.

## 10-Point Quality Check

### 1. The Sniff Test
Would you be embarrassed to show this to someone you respect?
If yes → revise. If no → continue.

### 2. Does It Say Something?
What is the one idea this piece communicates?
If you can't state it in one sentence → it needs a clearer thesis.

### 3. No AI Tells
Remove these words immediately:
- delve, comprehensive, leverage, utilize, facilitate
- furthermore, moreover, in conclusion, it's important to note
- as an AI, I cannot, I'd be happy to
- game-changing, revolutionary, groundbreaking
- ensure, foster, navigate

### 4. Specific Over General
Replace every abstraction with a concrete example.
Bad: "AI can improve your productivity"
Good: "AI cut my email response time from 45 minutes to 8 minutes"

### 5. Earned Claims Only
Every strong claim needs evidence, example, or reasoning.
If it's an opinion, own it: "I think..." not "Studies show..."

### 6. Voice Check
Read it out loud. Does it sound like you — or like a press release?
If it sounds like no one in particular → add personality.

### 7. Platform Fit
Is this the right length and format for the platform?
- Substack: has structure, can breathe
- X/Twitter: no fluff, every word earns its place
- LinkedIn: professional but not stiff

### 8. The Opener
Does the first line make someone want to read the second line?
If the article starts with context-setting → delete and start later.

### 9. The Close
Does it end with something to think about or act on?
A good ending doesn't trail off — it lands.

### 10. De-AI-ify Pass
Run a final scan. Remove any phrases that sound generated.
Target: sounds like a sharp human wrote this.
```

---

## Substack Publishing Workflow

### API Setup

Substack uses cookie-based authentication. Get your SID:

1. Open Substack in browser, log in
2. Open Developer Tools → Application → Cookies
3. Find the `substack.sid` cookie
4. Copy the value (it's long — copy all of it)

Add to OpenClaw secrets:
```bash
openclaw secret set SUBSTACK_SID "your-sid-value"
```

### Get Your User ID

```bash
curl -s "https://substack.com/api/v1/user/profile/self" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" | jq '.id'
```

Note your user ID — you'll need it as `byline_id` in all posts.

### Create a Draft

```bash
curl -s -X POST "https://[yourpublication].substack.com/api/v1/drafts" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{
    "draft_title": "Your Article Title",
    "draft_body": "<p>Article content in HTML</p>",
    "draft_byline_id": YOUR_USER_ID,
    "section_chosen": false,
    "type": "newsletter"
  }'
```

### Publish a Draft

```bash
curl -s -X POST "https://[yourpublication].substack.com/api/v1/drafts/DRAFT_ID/publish" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{
    "send": true,
    "share_automatically": true
  }'
```

### Agent Publishing Prompt

Ask your content agent:

```
Write a Substack article on [TOPIC].

Requirements:
- Title: [working title or ask agent to suggest 3]
- Length: 1,500-2,500 words
- Structure: Hook → 3-4 sections → Close with one clear takeaway
- Audience: [describe your readers]
- Tone: [your voice — direct / technical / narrative]
- Don't include: [any topics/styles to avoid]

After writing:
1. Run the CONTENT-STANDARD.md checklist
2. Write the HTML version (preserve paragraph structure)
3. Create a 3-tweet thread for X from the article
4. Write a 150-word Facebook post version
5. Write a 250-character Bluesky post

Do NOT publish yet — deliver for review.
```

---

## Cross-Posting Setup

### X/Twitter via xurl

Install xurl and configure with your API keys:

```bash
npm install -g xurl
xurl auth
```

Post a tweet:
```bash
xurl post "Your tweet content here"
```

Thread post:
```bash
xurl thread "Tweet 1" "Tweet 2" "Tweet 3"
```

Agent prompt for cross-posting:
```
Post this thread to X: [paste thread content]
Then confirm it posted by fetching my recent tweets.
```

### Bluesky

Configure in OpenClaw or post via API:

```bash
# Set up Bluesky credentials
openclaw secret set BLUESKY_HANDLE "yourhandle.bsky.social"
openclaw secret set BLUESKY_APP_PASSWORD "your-app-password"
```

Post via API:
```bash
curl -X POST "https://bsky.social/xrpc/com.atproto.repo.createRecord" \
  -H "Authorization: Bearer $BLUESKY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "repo": "yourhandle.bsky.social",
    "collection": "app.bsky.feed.post",
    "record": {
      "text": "Your post content",
      "$type": "app.bsky.feed.post",
      "createdAt": "2024-01-15T12:00:00Z"
    }
  }'
```

### Full Cross-Post Agent Prompt

After publishing to Substack:

```
Cross-post this article:

Title: [Article Title]
Substack URL: [URL]
Summary: [2-3 sentence summary]

Post to:
1. X — 3-tweet thread (tweet 1: hook, tweet 2: insight, tweet 3: link)
2. Bluesky — 250-char version with link
3. Facebook — 150-word conversational version

Verify each post landed. Report back with confirmation links.
```

---

## Editorial Calendar Template

Create `content/calendar.md`:

```markdown
# Editorial Calendar

## Content Pillars
1. [Pillar 1 — e.g., "How-to tutorials"]
2. [Pillar 2 — e.g., "Industry analysis"]
3. [Pillar 3 — e.g., "Personal stories / case studies"]

## Weekly Cadence
- Monday: [Content type]
- Wednesday: [Content type]
- Friday: [Content type]
- Social: [Daily or 3x/week]

## Content Pipeline

### Ideas (backlog)
- [ ] [Idea 1]
- [ ] [Idea 2]
- [ ] [Idea 3]

### In Progress
- [ ] [Article title] — drafted, needs edit
- [ ] [Article title] — outline stage

### Scheduled
- [Date]: [Article title]
- [Date]: [Article title]

### Published
- [Date]: [Article title] — [link]
- [Date]: [Article title] — [link]

## Monthly Goals
- Substack: [X] articles
- X/Twitter: [X] posts
- Bluesky: [X] posts
```

### Editorial Calendar Maintenance Prompt

Run weekly:
```
Review content/calendar.md.

1. Move any "In Progress" items that are done to "Published"
2. Pull the top 2 ideas from backlog to In Progress
3. Suggest 3 new content ideas based on recent research (check research/daily/ for inspiration)
4. Flag if we're behind on any pillar

Update the calendar file with changes.
```

---

## Content Generation Prompts

### Article Outline Generator

```
Create an article outline for: [TOPIC]

Audience: [Who reads your content]
Goal: [What should the reader walk away knowing/doing?]
Tone: [Direct / Technical / Narrative / Conversational]
Length target: [1,500 / 2,500 / 3,500 words]

Outline format:
1. Hook (first 2 sentences — make them count)
2. Problem/Context (set up why this matters)
3. Section 1: [Topic] — key points to cover
4. Section 2: [Topic] — key points to cover
5. Section 3: [Topic] — key points to cover
6. Close (takeaway + call to action or thought to leave with)

For each section, suggest 1-2 specific examples or data points to include.
```

### Social Media Batch Generator

```
Generate a week of social content on the theme: [THEME]

For each platform:
- X: 5 standalone tweets (punchy, no filler)
- Bluesky: 5 posts (slightly longer, more context)
- LinkedIn: 2 posts (professional angle)

Rules:
- No hashtag overuse (max 2 per post)
- No "Happy Monday!" filler
- Each post should work as a standalone piece
- Mix formats: insight, question, statement, thread-starter
```

---

## Troubleshooting

**Substack API returns 401**
- SID cookie has expired — re-fetch from browser
- Make sure you're using the right subdomain (yourpublication.substack.com)
- Check that you're using the SID from the right account

**Content sounds robotic**
- Run the De-AI-ify pass explicitly
- Give the agent 3 examples of your best writing to emulate
- Ask for "conversational" and "opinionated" explicitly

**Cross-posting fails on Facebook**
- Facebook requires paragraph breaks via HTML — plain text loses formatting
- Use `<p>` tags, not newlines
- Check your page access token hasn't expired

**Posts going live without review**
- Remove auto-publish from your workflow
- Always end prompts with "deliver for review, do NOT publish"
- Set up a staging review step in your workflow

**Editorial calendar getting stale**
- Run the maintenance prompt every Monday
- Ask agent to check the last 7 days of research for content ideas
- Block 30 minutes weekly for calendar review

---

*Built on OpenClaw. Requires OpenClaw installed and configured.*
