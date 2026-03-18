# The Substack Publishing System — API-Driven Publishing & Distribution

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 45-60 minutes

Automate your entire Substack operation. This guide covers SID cookie auth, programmatic draft creation with HTML formatting rules, scheduled publishing, byline configuration, and auto-distribution to X, Facebook, and Bluesky — the exact pipeline that published 13 articles in one month.

---

## What's In This Package

- SID cookie extraction guide (how to auth with Substack API)
- Draft creation API — full endpoint reference with HTML formatting rules
- Publish API — schedule, send to all, or email-only configuration
- Byline and author configuration
- Cross-posting scripts for X/Twitter, Facebook, and Bluesky
- Editorial calendar template (JSON format)
- Cover image generation workflow (automated OpenAI image gen)
- Quality checklist (double-encoding detection, formatting verification)
- Anti-phantom-success rule: verify articles actually published
- Growth tactics: SEO, social proof, subscriber conversion
- Troubleshooting: SID expiry, HTML entity encoding, byline errors

---

# The Substack Publishing System
## API-Driven Drafting, Publishing, and Cross-Platform Distribution

This guide covers the full Substack publishing workflow via API — from authenticating with your session cookie to cross-posting across four platforms automatically.

---

## Why API Over Manual?

Manual Substack publishing requires opening a browser, fighting the editor, and manually cross-posting to every platform. The API approach:
- Creates drafts programmatically (from agent-generated content)
- Publishes on schedule without touching a browser
- Triggers cross-posting automatically
- Works from anywhere — cron job, agent command, or CLI

---

## Step 1: Get Your Substack SID Cookie

Substack doesn't have a traditional OAuth API. Authentication uses a session cookie (`substack.sid`) from your browser session.

### Extract from Chrome/Firefox:

1. Open your browser → navigate to your Substack publication
2. Open DevTools: `F12` or `Cmd+Option+I`
3. Go to **Application** tab → **Cookies** → `https://[your-publication].substack.com`
4. Find the cookie named `substack.sid`
5. Copy the full value (a long string starting with `s%3A...`)

### Extract via curl (verify it works):

```bash
SUBSTACK_SID="s%3Ayour-sid-value-here"
PUBLICATION="your-publication-name"

curl -s "https://$PUBLICATION.substack.com/api/v1/drafts?limit=5" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" | python3 -m json.tool
```

You should see a JSON list of your recent drafts. If you get a 401, the cookie expired — re-extract it.

### Store Securely:

```bash
openclaw config set substack.sid "s%3Ayour-sid-value"
openclaw config set substack.publication "your-publication-name"
```

**Important:** SID cookies expire. They typically last 30–90 days. When publishing fails with 401, re-extract and update the config.

---

## Step 2: Create a Draft

### Basic Draft Creation

```bash
SUBSTACK_SID=$(openclaw config get substack.sid)
PUBLICATION=$(openclaw config get substack.publication)

# Create a draft with title and body
curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{
    "draft_title": "Your Article Title Here",
    "draft_subtitle": "Optional subtitle or deck",
    "draft_body": "<p>Your article content as HTML.</p><p>Paragraphs must be HTML — plain text does not format correctly.</p>",
    "draft_section_id": null,
    "section_chosen": false,
    "type": "newsletter"
  }' | python3 -m json.tool
```

The response includes the draft `id` — save it for publishing.

### Full Draft with Metadata

```bash
curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{
    "draft_title": "Article Title",
    "draft_subtitle": "Subtitle",
    "draft_body": "<h2>Section Header</h2><p>Opening paragraph...</p><p>Second paragraph...</p>",
    "draft_section_id": null,
    "section_chosen": false,
    "type": "newsletter",
    "audience": "everyone",
    "draft_podcast_duration": null,
    "draft_podcast_url": null,
    "draft_podcast_preview_upload_id": null,
    "draft_video_upload_id": null,
    "draft_cover_upload_id": null
  }'
```

### Important: HTML Formatting Rules

Substack renders body content as HTML. Common mistakes:
- ✅ `<p>Paragraph text</p>` — renders as paragraph
- ✅ `<h2>Header</h2>` — renders as section header
- ✅ `<strong>bold</strong>` — renders as bold
- ✅ `<hr/>` — horizontal rule / section divider
- ❌ Plain `\n` newlines — do NOT create paragraph breaks
- ❌ `\n\n` — does NOT create paragraphs

**Always wrap paragraphs in `<p>` tags when creating via API.**

---

## Step 3: Get the Byline ID

If you're publishing under a specific byline (author name), you need the byline ID.

```bash
curl -s "https://$PUBLICATION.substack.com/api/v1/user/publication-users" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" | python3 -m json.tool
```

Look for your user entry and note the `id` field. Use this as the `author_id` when publishing.

---

## Step 4: Publish a Draft

Once you have a `draft_id` from Step 2:

```bash
DRAFT_ID="123456"  # From the create draft response

curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts/$DRAFT_ID/publish" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{
    "send": true,
    "share_automatically": false,
    "audience": "everyone",
    "sent_at": null
  }'
```

**Parameters:**
- `"send": true` — sends to subscribers immediately
- `"send": false` — posts but doesn't email subscribers
- `"audience": "everyone"` — public post
- `"audience": "only_paid"` — paywall post
- `"sent_at": "2026-03-20T09:00:00.000Z"` — schedule for future time

### Schedule a Post

```bash
# Schedule for tomorrow at 9am Mountain Time (UTC-6 in winter, UTC-7 in summer)
SCHEDULED_TIME="2026-03-20T15:00:00.000Z"  # 9am MDT = 3pm UTC

curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts/$DRAFT_ID/publish" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d "{
    \"send\": true,
    \"audience\": \"everyone\",
    \"sent_at\": \"$SCHEDULED_TIME\"
  }"
```

---

## Step 5: Cross-Posting Workflow

After publishing, distribute to all platforms. Run these sequentially.

### Cross-Post to X (Twitter)

```bash
ARTICLE_URL="https://$PUBLICATION.substack.com/p/your-post-slug"
ARTICLE_TITLE="Your Article Title"

# Using xurl CLI
xurl post create \
  --text "$ARTICLE_TITLE

New post on Substack 👇

$ARTICLE_URL" \
  --media-path /tmp/cover-image.jpg  # Optional cover image
```

### Cross-Post to Facebook

Facebook cross-posting via API requires page access. Using an n8n webhook or direct Graph API:

```bash
PAGE_ID="your-page-id"
PAGE_TOKEN="your-page-access-token"

curl -X POST "https://graph.facebook.com/v20.0/$PAGE_ID/feed" \
  -d "access_token=$PAGE_TOKEN" \
  -d "message=New post: $ARTICLE_TITLE" \
  -d "link=$ARTICLE_URL"
```

**Note:** Facebook's Lexical editor (for personal posts) has different requirements. See AGENTS.md Facebook posting instructions for the clipboard injection method.

### Cross-Post to Bluesky

```bash
# Using bsky CLI or direct API
BSKY_HANDLE="yourhandle.bsky.social"
BSKY_PASSWORD="your-app-password"

# Get auth token
AUTH_RESPONSE=$(curl -s -X POST "https://bsky.social/xrpc/com.atproto.server.createSession" \
  -H "Content-Type: application/json" \
  -d "{\"identifier\": \"$BSKY_HANDLE\", \"password\": \"$BSKY_PASSWORD\"}")

ACCESS_JWT=$(echo $AUTH_RESPONSE | python3 -c "import sys,json; print(json.load(sys.stdin)['accessJwt'])")
DID=$(echo $AUTH_RESPONSE | python3 -c "import sys,json; print(json.load(sys.stdin)['did'])")

# Create post with link card
curl -s -X POST "https://bsky.social/xrpc/com.atproto.repo.createRecord" \
  -H "Authorization: Bearer $ACCESS_JWT" \
  -H "Content-Type: application/json" \
  -d "{
    \"repo\": \"$DID\",
    \"collection\": \"app.bsky.feed.post\",
    \"record\": {
      \"\$type\": \"app.bsky.feed.post\",
      \"text\": \"$ARTICLE_TITLE\n\n$ARTICLE_URL\",
      \"createdAt\": \"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"
    }
  }"
```

---

## Step 6: Editorial Calendar Template

Keep this in `memory/editorial-calendar.md`:

```markdown
# Editorial Calendar

## Publishing Cadence
- Newsletter: [weekly/biweekly/monthly]
- Day: [Tuesday]
- Time: [9am MDT]

## Queue

### Ready to Publish
| Title | Scheduled For | Status |
|-------|--------------|--------|
| [Title] | [Date] | Draft created |

### In Progress  
| Title | Draft Date | Target Publish |
|-------|-----------|----------------|
| [Title] | [Date] | [Date] |

### Ideas
- [Topic idea 1]
- [Topic idea 2]

## Published (last 30 days)
| Title | Published | X | FB | Bluesky | Performance |
|-------|-----------|---|----|---------|-----------  |
| [Title] | [Date] | ✅ | ✅ | ✅ | [opens] |
```

---

## Step 7: Cover Image Generation

For consistent cover art, use the OpenAI image generation API:

```bash
# Generate cover image for a post
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "dall-e-3",
    "prompt": "Abstract cover image for article about [topic]. Style: [your brand style]. Colors: [your brand colors]. Aspect ratio: 16:9.",
    "size": "1792x1024",
    "quality": "standard",
    "n": 1
  }' | python3 -c "import sys,json; print(json.load(sys.stdin)['data'][0]['url'])"
```

Download the image:
```bash
IMG_URL="[url from above]"
curl -o /tmp/cover-$(date +%Y%m%d).png "$IMG_URL"
```

Upload to Substack as draft cover:
```bash
# Upload image and get upload_id
curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/image" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -F "image=@/tmp/cover-image.png"
```

Use the returned `id` as `draft_cover_upload_id` when creating the draft.

---

## Quality Checklist (10 Points)

Before publishing any article, verify:

1. **Title is compelling** — Does it make someone want to click? Test: would you click this in your own inbox?

2. **Subtitle/deck adds value** — The subtitle should complement, not repeat the title.

3. **Opening paragraph hooks** — First 2 sentences should earn the reader's attention.

4. **No raw HTML in rendered view** — Draft preview shows no visible `<p>`, `<h2>`, `&lt;` etc.

5. **All headers are formatted** — Section headers use `<h2>` or `<h3>`, not bold text.

6. **Links are functional** — Click every link in the preview.

7. **Cover image is set** — Missing cover image = weaker social preview.

8. **Audience is correct** — Verify "everyone" vs. "only paid" before publishing.

9. **Cross-posting assets ready** — Have tweet text + image ready before hitting publish.

10. **Verify post-publish** — After publishing, use `web_fetch` on the live URL to check for encoding issues or formatting problems.

---

## Growth Tactics

**What actually moves the needle for Substack:**

1. **Referral program** — Substack's built-in referral system is powerful. Turn it on.

2. **Cross-promotion** — Find 3 newsletters in adjacent spaces. Offer to mention each other.

3. **X to Substack funnel** — Post threads about your newsletter topics on X. Link to the Substack in the last tweet.

4. **Consistency beats frequency** — One post every two weeks that's excellent > one post every week that's mediocre. Pick a cadence you can sustain.

5. **Notes** — Substack Notes is their native social feed. Post there 3–5× per week. Short observations from your niche. Direct traffic to newsletter.

6. **Embed the subscribe form** — Put your Substack subscribe link in your email signature, GitHub profile, and social bios.

7. **Guest posts** — Writing for a larger newsletter in your niche is faster subscriber growth than most content tactics.

8. **Reactivation campaign** — Once/quarter, send a personal note to people who subscribed but have low open rates. "Still interested? Here's what you've missed."

---

## Complete Publishing Script

Save this as `scripts/publish-substack.sh`:

```bash
#!/bin/bash
# Usage: ./publish-substack.sh "Article Title" /path/to/content.html

TITLE=$1
CONTENT_FILE=$2
PUBLICATION=$(openclaw config get substack.publication)
SID=$(openclaw config get substack.sid)

# Read HTML content
CONTENT=$(cat "$CONTENT_FILE")

# Create draft
echo "Creating draft..."
RESPONSE=$(curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts" \
  -H "Cookie: substack.sid=$SID" \
  -H "Content-Type: application/json" \
  -d "{\"draft_title\": \"$TITLE\", \"draft_body\": $(echo "$CONTENT" | python3 -c 'import json,sys; print(json.dumps(sys.stdin.read()))'), \"type\": \"newsletter\"}")

DRAFT_ID=$(echo $RESPONSE | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")
echo "Draft created: $DRAFT_ID"

# Publish
echo "Publishing..."
curl -s -X POST "https://$PUBLICATION.substack.com/api/v1/drafts/$DRAFT_ID/publish" \
  -H "Cookie: substack.sid=$SID" \
  -H "Content-Type: application/json" \
  -d '{"send": true, "audience": "everyone"}'

echo "Published. Draft ID: $DRAFT_ID"
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
