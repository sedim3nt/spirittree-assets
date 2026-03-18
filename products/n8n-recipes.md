# n8n Workflow Recipes — 8 Production Automation Templates

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 15-30 minutes per workflow

8 production n8n workflows for AI agent operations — tweet poster, Substack publisher, Instagram poster, daily digest, content distribution, webhook handlers, health monitoring, and scheduled tasks. Each includes complete setup instructions and copy-paste node configurations.

---

## What's In This Package

- **Workflow 1:** Tweet Poster — schedule and post to X/Twitter with OAuth 1.0
- **Workflow 2:** Substack Publisher — create drafts + publish via API with SID cookie auth
- **Workflow 3:** Instagram Poster — Meta Graph API image publishing with container flow
- **Workflow 4:** Daily Digest — compile multi-source briefings delivered at 7 AM
- **Workflow 5:** Webhook Handler — process Stripe, GitHub, Cal.com, and custom webhooks
- **Workflow 6:** Health Monitor — 5-minute service checks with Telegram alerts
- **Workflow 7:** Content Distribution — auto-distribute Substack articles to all platforms
- **Workflow 8:** Scheduled Tasks — universal template for any recurring operation
- Error handling patterns (alert throttling, retry logic)
- Troubleshooting guide for common failures
- n8n LaunchAgent config for macOS (runs on startup)

---

---

## Prerequisites

- n8n installed and running (local or cloud)
- OpenClaw agent accessible
- Platform API credentials for relevant workflows

### Install n8n (Local)

```bash
npm install -g n8n
n8n start
# Opens at http://localhost:5678
```

### n8n with LaunchAgent (macOS)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.n8n.service</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/n8n</string>
        <string>start</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>N8N_PORT</key><string>5678</string>
        <key>N8N_PROTOCOL</key><string>http</string>
    </dict>
    <key>RunAtLoad</key><true/>
    <key>KeepAlive</key><true/>
    <key>StandardOutPath</key><string>/tmp/n8n.log</string>
</dict>
</plist>
```

---

## Workflow 1: Tweet Poster

**Purpose:** Schedule and post tweets to X/Twitter  
**Trigger:** Manual or scheduled  
**Estimated setup:** 15 minutes

### Setup

1. New workflow in n8n
2. Add **Manual Trigger** (or **Schedule** for recurring)
3. Add **Code node** — paste tweet content
4. Add **HTTP Request node**:
   - Method: POST
   - URL: `https://api.twitter.com/2/tweets`
   - Auth: OAuth 1.0 (your Twitter credentials)
   - Body: `{"text": "{{$json.tweet}}"}`
5. Add **Telegram node** (optional): confirm posted

### Key Configuration

Twitter/X credentials in n8n:
- Credential Type: OAuth1 API
- Consumer Key: Twitter App API Key
- Consumer Secret: Twitter App API Secret
- Access Token: User access token
- Access Secret: User access token secret

### Thread Support

For threads, add a **Loop Over Items** node to post multiple tweets in sequence with a 2-second delay between each.

---

## Workflow 2: Substack Publisher

**Purpose:** Create and publish Substack articles  
**Trigger:** Manual trigger + article input  
**Estimated setup:** 20 minutes

### Setup

1. **Manual Trigger** with input fields:
   - `title` — article title
   - `body_html` — HTML content
   - `byline_id` — your Substack user ID

2. **HTTP Request** — Create Draft:
   - Method: POST
   - URL: `https://[yourpub].substack.com/api/v1/drafts`
   - Headers: `Cookie: substack.sid={{$env.SUBSTACK_SID}}`
   - Body: `{...draft data...}`

3. **Wait node**: 2 seconds (API rate limit safety)

4. **HTTP Request** — Publish:
   - Method: POST
   - URL: `https://[yourpub].substack.com/api/v1/drafts/{{$json.id}}/publish`
   - Headers: same cookie
   - Body: `{"send": true}`

5. **Telegram node**: notify with article URL

### Store SID Safely

```bash
# In n8n: Settings → Environment Variables
# Add: SUBSTACK_SID = your_sid_value
```

---

## Workflow 3: Instagram Poster

**Purpose:** Post images to Instagram Business account  
**Trigger:** Manual or schedule  
**Estimated setup:** 25 minutes (Meta API setup takes time)

### Prerequisites

- Facebook Business account
- Instagram Business/Creator account connected
- Facebook App with Instagram Graph API enabled
- Long-lived Page Access Token

### Workflow Steps

1. **Manual Trigger** with inputs: `image_url`, `caption`

2. **HTTP Request** — Create Container:
   - Method: POST
   - URL: `https://graph.facebook.com/v18.0/{INSTAGRAM_USER_ID}/media`
   - Body: `{"image_url": "{{$json.image_url}}", "caption": "{{$json.caption}}", "access_token": "{{$env.INSTAGRAM_TOKEN}}"}`

3. **Wait node**: 30 seconds (media processing time)

4. **HTTP Request** — Publish Container:
   - Method: POST
   - URL: `https://graph.facebook.com/v18.0/{INSTAGRAM_USER_ID}/media_publish`
   - Body: `{"creation_id": "{{$json.id}}", "access_token": "{{$env.INSTAGRAM_TOKEN}}"}`

5. **Telegram node**: confirmation

### Image URL Requirement

Instagram requires a publicly accessible image URL. Use:
- GitHub Pages URL (upload image to repo)
- S3 or Cloudflare R2
- Any CDN with public access

---

## Workflow 4: Daily Digest

**Purpose:** Compile daily briefing from multiple sources  
**Trigger:** Schedule (7 AM weekdays)  
**Estimated setup:** 25 minutes

### Workflow Steps

1. **Schedule Trigger**: 7 AM Mon-Fri (`0 7 * * 1-5`)

2. **Parallel branches** (use Merge node at end):
   - Branch A: HTTP Request to news API
   - Branch B: RSS Feed read (Hacker News, Feedly)
   - Branch C: Stripe revenue (optional)

3. **Code node** — compile digest:
   ```javascript
   const news = $('HTTP Request').all()[0].json;
   const rss = $('RSS Feed').all()[0].json;
   
   return [{
     json: {
       digest: `📊 Daily Digest\n\n${news.summary}\n\n${rss.items.slice(0,3).map(i => `• ${i.title}`).join('\n')}`
     }
   }];
   ```

4. **Telegram node**: send digest

5. **n8n HTTP Request** (optional): also POST to Discord webhook

---

## Workflow 5: Webhook Handler

**Purpose:** Process incoming webhooks from external services  
**Trigger:** Webhook URL  
**Estimated setup:** 15 minutes

### Setup

1. **Webhook node** (not trigger — this is the "Webhook" node):
   - HTTP Method: POST
   - Path: `/webhook/stripe` (or any path)
   - Authentication: Header Auth
   - Copy the webhook URL for configuration

2. **Switch node** — route by event type:
   - Branch: `{{$json.type}} === "charge.succeeded"`
   - Branch: `{{$json.type}} === "charge.failed"`
   - Branch: default

3. **Per branch**: appropriate handling (log, alert, trigger workflow)

4. **Respond to Webhook node**: return 200 OK

### Use Cases

- Stripe → log payments, alert on failures
- GitHub → trigger CI notifications
- Cal.com → meeting booked notifications
- Any webhook source

---

## Workflow 6: Health Monitor

**Purpose:** Monitor services and alert on failures  
**Trigger:** Schedule (every 5 minutes)  
**Estimated setup:** 20 minutes

### Workflow Steps

1. **Schedule Trigger**: every 5 minutes (`*/5 * * * *`)

2. **HTTP Request nodes** (parallel):
   - n8n: GET `http://localhost:5678/healthz`
   - App 1: GET `http://localhost:[PORT]/health`
   - App 2: GET `http://localhost:[PORT2]/health`

3. **Code node** — check results:
   ```javascript
   const checks = $input.all();
   const failed = checks.filter(c => c.json.status !== 200);
   
   if (failed.length > 0) {
     return [{ json: { status: 'FAILED', services: failed.map(f => f.json.service) } }];
   }
   return [{ json: { status: 'OK' } }];
   ```

4. **IF node**: if status === 'FAILED'

5. On failure branch: **Telegram node** → alert + **HTTP Request** → attempt restart

6. On healthy branch: optional log to file (skip Telegram)

### Throttling Alerts

Add a **Code node** to prevent alert spam:
```javascript
// Check if we already alerted in last 30 min
const lastAlert = $getWorkflowStaticData('global').lastAlert || 0;
const now = Date.now();

if (now - lastAlert < 1800000) {
  return [{ json: { skip: true } }];
}

$getWorkflowStaticData('global').lastAlert = now;
return [{ json: { skip: false } }];
```

---

## Workflow 7: Content Distribution

**Purpose:** Auto-distribute published Substack articles to all platforms  
**Trigger:** Schedule or webhook (on Substack publish)  
**Estimated setup:** 30 minutes

### Workflow Steps

1. **RSS Feed Trigger**: subscribe to your Substack RSS
   - `https://[yourpub].substack.com/feed`
   - Poll interval: every 30 minutes
   - Only trigger on NEW items

2. **Code node** — extract article data:
   ```javascript
   const article = $json;
   return [{
     json: {
       title: article.title,
       url: article.link,
       summary: article.contentSnippet?.slice(0, 200) + '...'
     }
   }];
   ```

3. **Parallel branches**:

   **Branch A — X/Twitter**:
   - HTTP Request → xurl API or Twitter API
   - Content: `New: "${title}" [link]`

   **Branch B — Bluesky**:
   - HTTP Request → Bluesky create session + create post

   **Branch C — Facebook**:
   - HTTP Request → Graph API → post with link preview

4. **Merge node**: collect results

5. **Code node**: compile success/failure report

6. **Telegram node**: "✅ Distributed: [title] — X ✅ / Bluesky ✅ / Facebook ✅"

---

## Workflow 8: Scheduled Tasks

**Purpose:** Template for running any recurring scheduled operation  
**Trigger:** Configurable schedule  
**Estimated setup:** 10-15 minutes per task

### Universal Scheduled Task Template

```json
{
  "name": "Scheduled Task: [NAME]",
  "nodes": [
    {
      "type": "Schedule Trigger",
      "cronExpression": "0 7 * * 1-5"
    },
    {
      "type": "HTTP Request",
      "method": "POST",
      "url": "http://localhost:3000/message",
      "body": {
        "message": "[YOUR OPENCLAW PROMPT HERE]",
        "channel": "telegram"
      }
    },
    {
      "type": "Error Trigger",
      "note": "Catches any failures and alerts"
    },
    {
      "type": "Telegram",
      "text": "⚠️ Scheduled task failed: [NAME]"
    }
  ]
}
```

### Pre-Built Schedule Configurations

**Daily at 7 AM weekdays:** `0 7 * * 1-5`  
**Every 6 hours:** `0 */6 * * *`  
**Weekly Monday 9 AM:** `0 9 * * 1`  
**Monthly 1st at 8 AM:** `0 8 1 * *`  
**Every 5 minutes:** `*/5 * * * *`

---

## Error Handling Patterns

### Add to Every Workflow

1. **Error Trigger node** — catches uncaught errors
2. **Telegram node** — "Workflow [NAME] failed: {{$json.error.message}}"
3. **Set node** — log error to file (optional)

### Rate Limit Handling

Add a **Wait node** (1-5 seconds) between API calls to external services.

For retry logic:
1. HTTP Request → IF error → Wait 30s → HTTP Request (retry once)
2. If still fails → Alert

---

## Troubleshooting

**Workflow doesn't trigger on schedule**
- Check system timezone vs. n8n timezone (Settings → Execution)
- Verify n8n process is running: `curl http://localhost:5678/healthz`
- Check workflow is "Active" (green toggle at top)

**API calls return 401**
- Credentials may have expired — check token expiry
- Re-enter credentials in n8n: Settings → Credentials
- For Substack SID: re-fetch from browser and update env var

**RSS trigger fires for old items**
- n8n RSS trigger tracks seen items — first run may flood
- Set a "published_after" filter in first run
- Or: run once manually, let it mark items as seen, then activate

**Workflow succeeds but content doesn't appear**
- Read the n8n anti-phantom-success rule: workflow "success" ≠ content landed
- Always add a verification step: fetch recent posts from the platform
- If verification fails: alert, don't assume success

---

*Built on n8n. Requires n8n installed and running. Platform API credentials required for respective workflows.*
