# OpenClaw Telegram Handbook — Multi-Topic Agent Operations

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 30-45 minutes

Telegram is the best interface for running an AI agent operation. This handbook covers everything from creating your first bot to organizing a fully structured multi-topic workspace — the same setup running AgentOrchard's six-channel agent command center.

---

## What's In This Package

- BotFather setup walkthrough (create bot, get token)
- OpenClaw JSON config for Telegram integration
- Forum Group setup with 6 dedicated topic channels
- Bot admin permissions configuration
- Topic organization strategy (what goes in each channel)
- Cron delivery routing (route different outputs to different topics)
- BULLETIN.md cross-topic coordination pattern
- Migration guide: single DM → organized forum setup
- Model-per-topic configuration (use cheaper models for monitoring)
- Privacy and security settings for production use
- Troubleshooting: webhook failures, message delivery issues, permission errors

---

# OpenClaw Telegram Handbook
## Complete Setup Guide for Multi-Topic Agent Operations

Telegram is the best interface for OpenClaw. This handbook covers everything from creating your first bot to running a fully organized multi-topic agent workspace with six dedicated channels.

---

## Why Telegram?

- **Persistent history** — conversations don't vanish like some terminal sessions
- **Mobile-first** — manage your agent operation from anywhere
- **Forum Topics** — organize by project/function without multiple bots
- **Bot API** — stable, well-documented, free
- **Rich formatting** — code blocks, bold, links render properly

---

## Phase 1: Create Your Telegram Bot

### Step 1: Talk to BotFather

Open Telegram and search for `@BotFather` (verified, blue checkmark).

```
/newbot
```

BotFather will ask:
1. **Name** — the display name: `AgentOrchard Assistant`
2. **Username** — the @handle, must end in "bot": `spirittree_ops_bot`

BotFather returns your token:
```
7234567890:AAHdef123...longstring...xyz
```

Save this. You'll need it for OpenClaw config.

### Step 2: Configure Bot Settings

Set bot description (shows before first message):
```
/setdescription
```
> "My personal AI operations agent. Handles research, coding, content, and daily ops."

Set bot picture:
```
/setuserpic
```
Upload a square image representing your brand.

Enable inline mode (optional, for quick queries):
```
/setinline
```

---

## Phase 2: Create a Telegram Supergroup with Forum Topics

A supergroup with Forum Topics gives you organized, persistent channels for different functions — all served by one bot.

### Step 1: Create the Supergroup

1. Tap the pencil icon → **New Group**
2. Name it: `[YourName] Operations` (e.g., `AgentOrchard Ops`)
3. Add at least one other account (required to create a group — you can remove them later)
4. Tap **Create**

### Step 2: Enable Forum Topics

1. Open the group → tap the group name at the top
2. Tap **Edit** (pencil icon)
3. Scroll down to **Topics** → toggle **ON**
4. Save

The group chat now has a Topics view. Each topic is its own independent thread.

### Step 3: Add Your Bot as Admin

1. Open group settings → **Administrators** → **Add Administrator**
2. Search for your bot username (e.g., `@spirittree_ops_bot`)
3. Grant permissions:
   - ✅ Post messages
   - ✅ Edit messages
   - ✅ Delete messages
   - ✅ Pin messages
   - ✅ Manage topics
   - ❌ Add new admins (don't give bots this)

### Step 4: Get the Group ID

Send a message in the group, then use the Telegram API to get the group ID:

```bash
curl "https://api.telegram.org/bot[YOUR_TOKEN]/getUpdates" | python3 -m json.tool | grep '"id"' | head -5
```

The group ID will be a large negative number like `-1002381931352`.

**Alternative:** Forward a message from the group to `@userinfobot` — it returns the chat ID.

---

## Phase 3: Configure OpenClaw

### Base Configuration

Edit `~/.openclaw/config.json`:

```json
{
  "plugins": {
    "telegram": {
      "botToken": "7234567890:AAHdef123...xyz",
      "mode": "polling",
      "groups": [
        {
          "id": "-1002381931352",
          "name": "AgentOrchard Ops",
          "type": "forum",
          "topics": {
            "general": 1,
            "dev": 23,
            "research": 47,
            "content": 89,
            "revenue": 124,
            "infra": 156
          }
        }
      ]
    }
  }
}
```

**Getting topic IDs:** After creating topics (next step), send a message in each topic. The `getUpdates` API call will include the `message_thread_id` for each topic. Record these and add to your config.

### Restart the Gateway

```bash
openclaw gateway restart
openclaw gateway status
```

---

## Phase 4: Create the 6 Core Topics

Create these topics in your forum group. The names and purposes are deliberately generic — adapt to your operation.

### Topic 1: General

**Purpose:** Direct conversation, questions that don't belong elsewhere, status updates
**Color:** Blue (default)

**Pin this message:**
```
📌 General Operations

This is the primary topic for direct agent interaction.

Use other topics for:
• /dev — code, technical work, GitHub
• /research — research tasks, analysis, briefings  
• /content — writing, social media, publishing
• /revenue — business, clients, financial ops
• /infra — system health, crons, monitoring

Type /help for available commands.
```

### Topic 2: Dev

**Purpose:** Coding tasks, GitHub operations, technical debugging, deployments
**Color:** Green

**Pin this message:**
```
📌 Development

For coding tasks, debugging, and technical work.

Examples:
• "Review this function for bugs"
• "Write a GitHub Actions workflow for X"
• "Debug why this script is failing"
• "Build [feature] in [repo]"

Model: Claude Sonnet (default) | Opus on request
```

### Topic 3: Research

**Purpose:** Research tasks, analysis, competitive intelligence, deep dives
**Color:** Yellow/Orange

**Pin this message:**
```
📌 Research

For research tasks, analysis, and intelligence gathering.

Examples:
• "Research [topic] and give me a briefing"
• "Analyze this paper and extract key insights"
• "Find the top 5 tools for [use case]"
• "Compare [A] vs [B] across these criteria"

Model: Sonnet default, Gemini for large documents
```

### Topic 4: Content

**Purpose:** Writing, editing, social posts, Substack drafts, content strategy
**Color:** Purple

**Pin this message:**
```
📌 Content

For writing, editing, and content production.

Examples:
• "Draft a Substack post about [topic]"
• "Write 5 tweets about [theme]"
• "Edit this draft for clarity and concision"
• "Create a content calendar for [week/month]"

Model: Claude Sonnet
```

### Topic 5: Revenue

**Purpose:** Business operations, client work, financial tracking, pricing
**Color:** Gold

**Pin this message:**
```
📌 Revenue & Business

For business operations and financial context.

Examples:
• "Draft a proposal for [client]"
• "Review this SOW for gaps"
• "Calculate margins on [project]"
• "Outline a retainer structure for [service type]"

Sensitive: No hardcoded financial data. Reference MEMORY.md for context.
```

### Topic 6: Infra

**Purpose:** System health, monitoring, crons, gateway status, technical ops
**Color:** Red/Orange

**Pin this message:**
```
📌 Infrastructure

For system monitoring and operational health.

Daily status checks post here automatically.
Manual commands:
• "Gateway status"
• "List all crons"
• "Check n8n health"
• "Review today's memory log"

Model: Haiku for all monitoring tasks
```

---

## Phase 5: Configure Cron Delivery Per Topic

Direct cron outputs to the appropriate topic instead of General. Add to your crons config in AGENTS.md:

```markdown
## Cron Delivery Channels

### Morning Briefing → General (topic 1)
Schedule: 0 8 * * *
Delivery: General topic
Content: Read MEMORY.md + today's memory log, summarize pending items

### System Health Check → Infra (topic 156)
Schedule: 0 */6 * * *  (every 6 hours)
Delivery: Infra topic
Content: Check gateway, n8n, Telegram bot status; report anomalies only

### Weekly Research Briefing → Research (topic 47)
Schedule: 0 9 * * 1  (Monday 9am)
Delivery: Research topic
Content: Weekly digest on [your tracked topics]

### Content Calendar → Content (topic 89)
Schedule: 0 8 * * 1  (Monday 8am)
Delivery: Content topic
Content: This week's content plan based on editorial calendar
```

---

## Phase 6: BULLETIN.md Cross-Topic Coordination

`BULLETIN.md` is a coordination file that lives in your workspace. Agents use it to leave notes for other agents across topics — a lightweight message bus.

Create `~/.openclaw/workspace/BULLETIN.md`:

```markdown
# BULLETIN.md — Cross-Agent Coordination

## Active Items
<!-- Agents write here when they need another agent to follow up -->

## Format
[TOPIC] [PRIORITY] [FROM] → [TO]: [Message]

## Example
[CONTENT] [HIGH] [research] → [content]: Found 3 great sources for the AI governance piece — see memory/2026-03-17.md

## Cleared Items (last 7 days)
<!-- Completed items move here -->
```

Add a standing instruction to AGENTS.md:
```markdown
## BULLETIN Protocol
- Before starting work in a topic, read BULLETIN.md for pending items addressed to that topic
- After completing a task that affects another topic, write a note to BULLETIN.md
- Clear completed items weekly
```

---

## Phase 7: Migration from Single DM

If you've been using a direct message conversation with your bot, here's how to migrate:

### 1. Export Important Context

Before migrating, ask your agent in the DM:
> "Summarize the last month of our work together — key decisions, active projects, and anything I should carry forward."

Save that summary to MEMORY.md.

### 2. Update Config

Change your default channel from DM to the forum group:
```json
{
  "plugins": {
    "telegram": {
      "defaultChatId": "-1002381931352",
      "defaultTopicId": 1
    }
  }
}
```

### 3. Test Each Topic

Send a test message in each topic:
```
[Dev] Test message — can you see this?
[Research] Test — responding from Research topic?
```

Verify the bot responds in the correct topic thread.

### 4. Update Cron Targets

Update any crons that were sending to your DM to now target the appropriate forum topic.

---

## Phase 8: Model-Per-Topic Strategy

Optimize costs by assigning different models to different topics:

| Topic | Default Model | Rationale |
|-------|--------------|-----------|
| General | Sonnet | Balanced, most interactions |
| Dev | Sonnet | Coding quality matters |
| Research | Sonnet (Gemini for large docs) | Quality synthesis |
| Content | Sonnet | Writing quality matters |
| Revenue | Sonnet | Business decisions need quality |
| Infra | Haiku | Monitoring is simple, runs often |

Configure in AGENTS.md:
```markdown
## Topic Model Routing
- general → claude-sonnet-4-6
- dev → claude-sonnet-4-6
- research → claude-sonnet-4-6 (gemini-2.5-pro for large documents)
- content → claude-sonnet-4-6
- revenue → claude-sonnet-4-6
- infra → claude-haiku-3-5
```

---

## Troubleshooting

**Bot not responding in topics:**
- Verify bot has "Manage topics" admin permission
- Check that `message_thread_id` in config matches actual topic ID
- Run `openclaw gateway status` to verify the gateway is running

**Messages going to wrong topic:**
- The topic IDs in config must exactly match the Telegram topic IDs
- Use `getUpdates` to verify: send a message in the topic, check the `message_thread_id` value

**Bot responds in DM but not in group:**
- Open BotFather → `/mybots` → [your bot] → Bot Settings → Group Privacy → Turn OFF
- With privacy mode ON, bots only see messages that mention them by username in groups

**Forum Topics option missing:**
- Forum Topics requires the group to be a supergroup (not a basic group)
- If you can't enable it: create a new group and make sure it's set to "supergroup" during creation

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
