# Social Automation — Cross-Platform Posting System

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 45-60 minutes

Complete social media automation. Bluesky, X/Twitter, Facebook, and Substack. Includes posting scripts, cross-posting workflow, API configurations, and n8n workflow templates.

---

## What's In This Package

- Bluesky posting via API
- X/Twitter via xurl CLI
- Facebook page posting
- Substack API publishing
- Cross-posting workflow (write once, publish everywhere)
- n8n workflow templates

---

## Bluesky Setup

### App Password Setup

1. Go to bsky.app → Settings → App Passwords
2. Create a new app password
3. Copy it (you won't see it again)

Store credentials:
```bash
openclaw secret set BLUESKY_HANDLE "yourhandle.bsky.social"
openclaw secret set BLUESKY_APP_PASSWORD "xxxx-xxxx-xxxx-xxxx"
```

### Get an Auth Token

```bash
# Get session token
curl -X POST "https://bsky.social/xrpc/com.atproto.server.createSession" \
  -H "Content-Type: application/json" \
  -d "{\"identifier\": \"$BLUESKY_HANDLE\", \"password\": \"$BLUESKY_APP_PASSWORD\"}" \
  | jq -r '.accessJwt'
```

Store as `BLUESKY_TOKEN`.

### Post to Bluesky

```bash
BLUESKY_TOKEN="your-jwt-token"
BLUESKY_HANDLE="yourhandle.bsky.social"

curl -X POST "https://bsky.social/xrpc/com.atproto.repo.createRecord" \
  -H "Authorization: Bearer $BLUESKY_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"repo\": \"$BLUESKY_HANDLE\",
    \"collection\": \"app.bsky.feed.post\",
    \"record\": {
      \"\$type\": \"app.bsky.feed.post\",
      \"text\": \"Your post content here\",
      \"createdAt\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"
    }
  }"
```

### Bluesky Agent Prompt

```
Post to Bluesky: [CONTENT]

Steps:
1. Get auth token using stored BLUESKY credentials
2. Post the content
3. Verify: fetch my recent posts to confirm it appeared

If the token has expired, re-authenticate and retry once.
Max 300 characters. Add link card if URL present.
```

---

## X/Twitter via xurl

### Install xurl

```bash
npm install -g xurl
```

### Configure

```bash
xurl auth
# Follow OAuth2 flow — needs Twitter Developer app credentials
```

Or set env vars:
```bash
export XURL_API_KEY="your-api-key"
export XURL_API_SECRET="your-api-secret"
export XURL_ACCESS_TOKEN="your-access-token"
export XURL_ACCESS_SECRET="your-access-secret"
```

### Post Commands

```bash
# Single tweet
xurl post "Your tweet content here"

# Tweet with media
xurl post "Caption" --media /path/to/image.jpg

# Thread
xurl thread \
  "Tweet 1: The hook" \
  "Tweet 2: The argument" \
  "Tweet 3: The link/CTA"

# Verify it posted
xurl get --user me --count 5
```

### X Agent Prompt

```
Post this to X/Twitter: [CONTENT]

1. Use xurl to post
2. If it's a thread (multiple tweets), use xurl thread
3. Confirm by fetching my last 3 tweets

Content rules:
- Under 280 characters per tweet
- No hashtag spam (max 2 if relevant)
- If linking to an article, put the link in the last tweet of a thread
```

---

## Facebook Page Posting

### Setup

You need a Facebook Page and a Page Access Token.

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Create an app → Business type
3. Add Facebook Login + Pages API
4. Generate a Page Access Token with `pages_manage_posts` permission
5. Exchange for a long-lived token (60 days):

```bash
curl "https://graph.facebook.com/oauth/access_token?grant_type=fb_exchange_token&client_id=APP_ID&client_secret=APP_SECRET&fb_exchange_token=SHORT_LIVED_TOKEN"
```

Store:
```bash
openclaw secret set FACEBOOK_PAGE_ID "your-page-id"
openclaw secret set FACEBOOK_PAGE_TOKEN "your-long-lived-token"
```

### Post to Facebook

```bash
PAGE_ID="your-page-id"
PAGE_TOKEN="your-page-token"

curl -X POST "https://graph.facebook.com/v18.0/$PAGE_ID/feed" \
  -d "message=Your post content here" \
  -d "access_token=$PAGE_TOKEN"
```

### Post with Link

```bash
curl -X POST "https://graph.facebook.com/v18.0/$PAGE_ID/feed" \
  -d "message=Your post caption" \
  -d "link=https://your-article-url.com" \
  -d "access_token=$PAGE_TOKEN"
```

### Facebook Formatting Warning

Facebook's Lexical editor strips plain text newlines. For posts with paragraphs:
- Use `\n\n` (double newlines) for paragraph breaks
- Verify the post rendered correctly after publishing

### Facebook Agent Prompt

```
Post to Facebook Page: [CONTENT]

1. Use Facebook Graph API with stored PAGE credentials
2. Format paragraphs with double newlines
3. Verify post was created: fetch the page feed to confirm

Max 500 words. Conversational tone for Facebook.
If linking an article: caption first, then link at the end.
```

---

## Substack Publishing

### Authentication

Substack uses cookie-based auth. Get your SID:
1. Log into Substack in browser
2. Open DevTools → Application → Cookies
3. Copy `substack.sid` value

```bash
openclaw secret set SUBSTACK_SID "your-sid-value"
openclaw secret set SUBSTACK_PUBLICATION "yourpublication.substack.com"
```

Get your user (byline) ID:
```bash
curl -s "https://substack.com/api/v1/user/profile/self" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" | jq '.id'
```

### Create Draft

```bash
BYLINE_ID=123456789  # Your user ID from above

curl -X POST "https://$SUBSTACK_PUBLICATION/api/v1/drafts" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d "{
    \"draft_title\": \"Your Title\",
    \"draft_body\": \"<p>Article content in HTML</p>\",
    \"draft_byline_id\": $BYLINE_ID,
    \"type\": \"newsletter\"
  }"
```

### Publish Draft

```bash
DRAFT_ID="your-draft-id"  # From the create draft response

curl -X POST "https://$SUBSTACK_PUBLICATION/api/v1/drafts/$DRAFT_ID/publish" \
  -H "Cookie: substack.sid=$SUBSTACK_SID" \
  -H "Content-Type: application/json" \
  -d '{"send": true}'
```

---

## Cross-Posting Workflow

### Write Once, Post Everywhere

This is the master workflow prompt:

```
Cross-post this content to all platforms.

ARTICLE:
Title: [TITLE]
URL: [SUBSTACK OR BLOG URL]
Summary: [2-3 sentence summary]

Platform versions:

1. BLUESKY (250 chars max)
Format: Hook sentence. Key insight. Link.
Post via Bluesky API.

2. X/TWITTER (thread)
Tweet 1: Hook (most compelling line from article)
Tweet 2: Core insight or data point
Tweet 3: Link + CTA
Post via xurl thread.

3. FACEBOOK (150-200 words)
Conversational framing. Paragraph breaks. End with link.
Post via Facebook Graph API.

Execution order: Bluesky → X → Facebook
Verify each before moving to next.
Report: which platforms confirmed ✅ / which failed ❌

Log to content/distribution/[DATE].md
```

### Distribution Verification Prompt

Run after any publish:

```
Verify distribution for: [TITLE]

Check each platform was successful:
1. Bluesky: fetch my recent posts — look for [title/hook]
2. X: xurl get --user me --count 3 — look for tweet
3. Facebook: fetch page feed — look for post
4. Substack: fetch article at [URL] — check it's live

Report within 60 seconds of publish. If any platform failed:
- Note the error
- Retry once
- If still failing: alert me immediately

Distribution complete only when all 4 ✅
```

---

## n8n Workflow Templates

### Workflow 1: Tweet Poster

Manual trigger → post to X.

```json
{
  "name": "Tweet Poster",
  "nodes": [
    {
      "type": "n8n-nodes-base.manualTrigger",
      "name": "Manual Trigger"
    },
    {
      "type": "n8n-nodes-base.set",
      "name": "Set Tweet Content",
      "parameters": {
        "values": {
          "string": [
            {"name": "tweet", "value": "={{$json.tweet}}"}
          ]
        }
      }
    },
    {
      "type": "n8n-nodes-base.httpRequest",
      "name": "Post to Twitter",
      "parameters": {
        "method": "POST",
        "url": "https://api.twitter.com/2/tweets",
        "authentication": "oAuth1",
        "body": {
          "text": "={{$json.tweet}}"
        }
      }
    }
  ]
}
```

### Workflow 2: Daily Content Distribution

Webhook → auto-distribute to all platforms.

Trigger: HTTP webhook with `{title, url, summary, bluesky_text, tweet_thread, facebook_text}`

Steps:
1. Receive webhook payload
2. Post to Bluesky
3. Post thread to X
4. Post to Facebook
5. Verify each (optional: fetch recent posts)
6. Send Telegram success/failure summary

### Workflow 3: Substack to Social (Auto-Cross-Post)

1. Check Substack feed every hour via RSS
2. Detect new articles
3. Generate platform variants via OpenAI/Claude
4. Post to Bluesky, X, Facebook
5. Log to Airtable/Notion/file

RSS feed: `https://[yourpub].substack.com/feed`

---

## Content Queue System

Batch-create social content and queue it:

Create `social/queue.md`:

```markdown
# Social Media Queue

## To Post (Bluesky)
- [ ] [Date/time] — [Content]
- [ ] [Date/time] — [Content]

## To Post (X/Twitter)
- [ ] [Date/time] — [Tweet or thread marker]

## To Post (Facebook)
- [ ] [Date/time] — [Content]

## Posted
- [Date]: [Content summary] — ✅ Bluesky / ✅ X / ✅ Facebook
```

### Queue Posting Prompt (daily)

```
Post social queue for today. Read social/queue.md.

For each item scheduled for today (or past-due):
1. Post to the specified platform
2. Mark as done in the queue file
3. Verify it posted

Skip anything that feels time-sensitive (breaking news tied to date) if it's more than 24h overdue.

Report: X posted / X skipped / X failed
```

---

## Troubleshooting

**Bluesky token expired**
- Tokens expire after 2 hours
- Re-authenticate with app password before each session
- Store token in env var or regenerate in your script

**xurl auth fails**
- Regenerate access tokens at developer.twitter.com
- Check that your app has correct permissions (Read + Write)
- Some accounts need Elevated access for tweet API

**Facebook token expires**
- Page tokens last 60 days after long-lived exchange
- Set a calendar reminder to refresh before expiry
- Automate: n8n workflow to refresh token monthly

**Cross-posting hits rate limits**
- Add 5-10 second delays between platform posts
- Twitter: 300 tweets per 3 hours for most accounts
- Bluesky: more generous limits currently
- Space posts 30-60 seconds apart

**Substack SID expires**
- Re-fetch from browser cookies after each login session
- Usually valid for 2-4 weeks
- Store in openclaw secrets and update when needed

---

*Built on OpenClaw. Requires platform API credentials for each network.*
