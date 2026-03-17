# Discord Multi-Agent Setup — AI Operations Center

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 60-90 minutes

Set up a Discord server as your AI operations center. Multi-channel agent workflows, automated research pipelines, content approval flows, and project channels with pinned artifacts.

---

## What's In This Package

- Discord bot setup guide
- Multi-channel architecture template
- Channel-per-project layout
- Automated research pipeline
- Content approval workflow
- Competitor monitoring channel
- Daily digest automation
- Permission configuration

---

## Why Discord as an Operations Center

Discord gives you:
- **Channel isolation** — separate contexts per topic
- **Thread-based discussions** — keep conversations organized
- **Bot integration** — automate messages to specific channels
- **Persistent history** — searchable record of all agent outputs
- **Mobile access** — full functionality on phone

This is an alternative to Telegram for teams or multi-project setups.

---

## Discord Bot Setup

### Create Your Bot

1. Go to [discord.com/developers/applications](https://discord.com/developers/applications)
2. "New Application" → name it (e.g., "YourName Ops")
3. Click "Bot" in sidebar → "Add Bot"
4. Under Privileged Gateway Intents, enable:
   - **Server Members Intent**
   - **Message Content Intent**
5. Copy the **Token** — this is your bot token

Store it:
```bash
openclaw secret set DISCORD_BOT_TOKEN "your-bot-token"
openclaw secret set DISCORD_SERVER_ID "your-server-id"
```

### Invite Bot to Server

1. In your app page → OAuth2 → URL Generator
2. Scopes: `bot`, `applications.commands`
3. Bot Permissions:
   - Read Messages/View Channels
   - Send Messages
   - Embed Links
   - Attach Files
   - Read Message History
   - Add Reactions
   - Manage Messages (for pinning)
4. Copy generated URL → open in browser → authorize

### Get Channel IDs

Right-click any channel → "Copy Channel ID" (requires Developer Mode in Discord settings).

---

## Multi-Channel Architecture

### Server Template

Create this channel structure:

```
📁 OPERATIONS
├── #briefings         — daily/weekly automated reports
├── #alerts            — urgent system alerts
└── #log               — agent activity log

📁 RESEARCH
├── #daily-brief       — morning research delivery
├── #youtube-monitor   — new video alerts
├── #competitor-intel  — competitor tracking
└── #ideas             — extracted content ideas

📁 CONTENT
├── #content-queue     — pieces in progress
├── #drafts-review     — content awaiting review
├── #published         — confirmation of published content
└── #calendar          — editorial calendar updates

📁 PROJECTS
├── #project-alpha     — channel per active project
├── #project-beta
└── #archive           — completed projects

📁 FINANCE
├── #revenue           — daily/weekly revenue reports
└── #alerts-finance    — failed payments, anomalies

📁 SYSTEM
├── #health            — service health alerts
└── #deploy            — deployment notifications
```

### Configure Channel IDs

Create `discord/channels.md`:

```markdown
# Discord Channel Configuration

## Server
- Server ID: [YOUR_SERVER_ID]

## Operations
- #briefings: [CHANNEL_ID]
- #alerts: [CHANNEL_ID]
- #log: [CHANNEL_ID]

## Research
- #daily-brief: [CHANNEL_ID]
- #youtube-monitor: [CHANNEL_ID]
- #competitor-intel: [CHANNEL_ID]
- #ideas: [CHANNEL_ID]

## Content
- #content-queue: [CHANNEL_ID]
- #drafts-review: [CHANNEL_ID]
- #published: [CHANNEL_ID]

## Finance
- #revenue: [CHANNEL_ID]
- #alerts-finance: [CHANNEL_ID]

## System
- #health: [CHANNEL_ID]
```

---

## Sending Messages to Discord

### Direct API Call

```bash
# Send to a channel
curl -s -X POST "https://discord.com/api/v10/channels/$CHANNEL_ID/messages" \
  -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"content\": \"Your message here\"}"

# Send with embed (formatted)
curl -s -X POST "https://discord.com/api/v10/channels/$CHANNEL_ID/messages" \
  -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "embeds": [{
      "title": "Daily Brief",
      "description": "Your brief content here",
      "color": 5814783,
      "timestamp": "2024-01-15T07:00:00Z"
    }]
  }'
```

### Agent Discord Send Prompt

```
Send this message to Discord channel #briefings.

Read discord/channels.md to get the channel ID for #briefings.

Message:
[CONTENT]

Format as embed with:
- Title: [TITLE]
- Description: [CONTENT]
- Color: blue (5814783)
- Timestamp: now

Confirm delivery: check if message appears in channel history.
```

---

## Automated Research Pipeline

### The Pipeline Flow

```
Trending content detection
    ↓
Agent research (web + YouTube)
    ↓
Synthesis and framing
    ↓
Post to #daily-brief
    ↓
Notify #ideas with content opportunities
```

### Research Pipeline Prompt (daily cron)

```
Run daily research pipeline. Deliver to Discord.

1. Check research/topics.md for configured topics
2. Search for news from last 24 hours per topic
3. Check youtube-channels.md for new videos

Compile report and send to Discord:
- Full report → #daily-brief channel
- Any YouTube alerts → #youtube-monitor channel
- Content ideas extracted → #ideas channel

Use discord/channels.md for channel IDs.
Send as embeds. Keep #daily-brief under 2000 chars (Discord limit).
Split into multiple messages if needed.
```

---

## Content Approval Workflow

### The Flow

```
Agent drafts content
    ↓
Posts to #drafts-review with 👍 / 👎 reactions
    ↓
You react to approve or reject
    ↓
Agent checks for reactions (via poll or next run)
    ↓
If approved: publishes to platform
    ↓
Confirmation to #published
```

### Draft Review Prompt

```
Post draft content for review in Discord.

Draft:
[CONTENT]

Send to #drafts-review with:
- Type: [Platform — Substack / Tweet / Facebook / Bluesky]
- Title/Hook: [First line]
- Full draft in embed
- Add 👍 and 👎 reactions after posting

Instruction at bottom of message: "React 👍 to approve for publishing"

Save draft ID and Discord message ID to content/pending/[slug].md
```

### Approval Check Prompt

```
Check content approval status in Discord.

Read content/pending/*.md for any pending drafts.

For each pending item:
1. Fetch the Discord message (from stored message ID)
2. Check reactions: any 👍 from authorized user?
3. If approved: proceed with publishing to [platform]
4. If rejected (👎): delete from pending, log reason
5. If no reaction after 48h: send reminder to #drafts-review

Report: X approved (publishing now) / X rejected / X pending
```

---

## Competitor Monitoring Channel

### Competitor Alert Format

When a competitor does something notable:

```
Send competitor alert to Discord #competitor-intel.

Alert data:
- Competitor: [NAME]
- Event type: [New product / Pricing change / YouTube video / Funding / Other]
- Details: [What happened]
- Source: [URL]
- Impact assessment: [HIGH / MEDIUM / LOW — why]

Format as embed:
- Title: "🔍 Competitor Alert: [Company]"
- Color: orange (16744448) for medium, red (16711680) for high
- Description: what happened + why it matters
- Footer: "Monitor source: [URL]"
```

### Competitor Digest Cron

Weekly, Sunday 6 PM → #competitor-intel:

```
Weekly competitive digest for Discord.

Read intel/weekly/[THIS-WEEK].md
Send to #competitor-intel as a formatted embed.

Format:
Title: "🗓️ Competitive Week in Review — [DATE]"
Sections as fields:
- Biggest Move
- YouTube Activity  
- Opportunity Flags

Color: blue
```

---

## Stock Research Channel Setup

For financial monitoring, replicate the pattern from this popular workflow:

### #finance-alerts Setup

```
Send financial alert to Discord #revenue.

Alert:
[TYPE: Revenue Milestone / Failed Payment / MRR Update]
[DETAILS]

Format:
- Revenue milestone → green embed (65280)
- Warning (failed payments) → yellow (16776960)
- Critical → red (16711680)
```

### Daily Revenue Discord Report

```
Post daily revenue report to Discord #revenue channel.

Read finance/stripe/[TODAY].md for revenue data.

Format as embed:
Title: "💰 Revenue — [DATE]"
Fields:
- Yesterday: $[X]
- MTD: $[X] / Goal $[X]
- Trend: [+/-]X% vs last week
- Failed: [X] payments

Color: green if on pace, yellow if behind, red if significantly behind.
```

---

## Permission Configuration

### Role Structure

For a solo operator:
- **@owner** — full bot access (you)
- **@agents** — bot posts to all channels
- Channels: agents can post, only @owner can manage

### Webhook Alternative

For simpler setups, use Discord Webhooks instead of bots:

```bash
# Create webhook: Channel Settings → Integrations → Webhooks
# Copy webhook URL

DISCORD_WEBHOOK="https://discord.com/api/webhooks/[ID]/[TOKEN]"

curl -s -X POST "$DISCORD_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{"content": "Your message", "username": "Agent Name"}'
```

Webhooks are simpler to set up but have fewer features than bots.

---

## n8n Discord Integration

For automated workflows, use the n8n Discord node:

1. Add Discord credential in n8n (bot token or webhook URL)
2. Discord node → "Send Message" action
3. Configure:
   - Guild ID: your server ID
   - Channel ID: target channel
   - Content: dynamic from previous node

Example: n8n workflow that posts Substack publish confirmation to #published

---

## Troubleshooting

**Bot not receiving messages**
- Check Message Content Intent is enabled in Discord Developer Portal
- Verify bot has "Read Messages" permission in the channel
- Test: send bot a DM and check if it responds

**Messages over 2000 character limit**
- Split into multiple messages using `\n` continuation markers
- Use embeds (can be up to 6000 chars total)
- Or: post summary to channel, full report to thread

**Channel IDs wrong**
- Enable Discord Developer Mode: Settings → Advanced → Developer Mode
- Right-click channel → "Copy Channel ID"
- Server ID: right-click server name → "Copy Server ID"

**Bot appears offline**
- Check that bot process is running (if self-hosted)
- Verify token hasn't been regenerated in Developer Portal
- Tokens change when you click "Regenerate" in bot settings

---

*Built on OpenClaw. Requires Discord developer account and server with Admin permissions.*
