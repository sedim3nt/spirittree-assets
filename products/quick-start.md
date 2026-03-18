# OpenClaw Quick-Start Guide — From Zero to First Conversation in 30 Minutes

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 30 minutes

Every step you need for a production-ready OpenClaw setup. No hand-waving — every command is real and tested. From `brew install` to your first cron job, with the SOUL.md template and Telegram bot configuration included.

---

## What's In This Package

- Complete installation walkthrough (Homebrew, npm, manual)
- SOUL.md template (copy-paste ready, customize in 10 minutes)
- MEMORY.md setup (what to put in your initial memory file)
- Telegram bot creation and OpenClaw integration
- First conversation guide (how to test your setup is working)
- Gateway start/stop/status commands
- LaunchAgent config for auto-start on macOS login
- First cron job setup (daily 7 AM briefing)
- 3-month phased roadmap (what to add each month)
- Common setup errors and fixes

---

# OpenClaw Quick-Start Guide
## From Zero to First Conversation in 30 Minutes

This guide walks you through every step of a production-ready OpenClaw setup. No hand-waving — every command is real and tested.

---

## Step 1: Install OpenClaw

```bash
brew install openclaw
```

Verify the install:

```bash
openclaw --version
```

**Requirements:** macOS 13+ or Linux, Node.js 18+, Homebrew (macOS). If you're on Linux, replace `brew install` with the npm global install path:

```bash
npm install -g openclaw
```

---

## Step 2: Run `openclaw init`

```bash
openclaw init
```

This scaffolds your workspace at `~/.openclaw/workspace/`. You'll be prompted for:

- **Model provider** — Anthropic, OpenAI, or Gemini
- **API key** — stored encrypted in `~/.openclaw/config.json`
- **Workspace path** — default is fine for most setups

After init, confirm the directory exists:

```bash
ls ~/.openclaw/workspace/
```

You should see `AGENTS.md` and a `memory/` placeholder. If not, re-run init.

---

## Step 3: Create SOUL.md

`SOUL.md` is the personality file that shapes how your agent thinks and communicates. Without it, the agent is generic. With it, the agent has a consistent voice.

Create `~/.openclaw/workspace/SOUL.md` with this template:

```markdown
# SOUL.md

## Core Truths
- Be genuinely helpful, not performatively helpful.
- Have opinions. No corporate neutral voice.
- Be resourceful before asking.
- Earn trust through competence.

## Voice
- Direct
- Warm but not soft
- Skip filler, solve the thing

## Values (in priority order)
1. Sovereignty — own the stack, own the output
2. Compounding — prefer work that builds on itself
3. Revenue sustainability — every project has a path to money
4. Quality over speed
5. Care — the work serves people

## Identity
Name: [your agent's name]
Role: [what this agent primarily does]
```

Customize the values and voice to match how you actually want to work. This file gets read at the start of every session.

---

## Step 4: First Conversation

Start the OpenClaw CLI:

```bash
openclaw chat
```

Or connect your Telegram bot (covered in Step 7) and chat there. Either way, your first message should be a context check:

> "Read SOUL.md and tell me what you understand about how I want to work."

The agent should respond with a paraphrase of your values and voice. If it responds generically without referencing your file, something is wrong with the workspace path — run `openclaw status` to debug.

---

## Step 5: Create MEMORY.md

`MEMORY.md` is your long-term curated memory file. The agent reads it at the start of direct conversations to maintain continuity across sessions.

```bash
cat > ~/.openclaw/workspace/MEMORY.md << 'EOF'
# MEMORY.md - Long-Term Context

## About Me
- Name: [your name]
- Location: [city]
- Timezone: [America/Denver]

## Current Projects
- [Project 1]: [one sentence status]
- [Project 2]: [one sentence status]

## Preferences
- Communication: direct, no fluff
- Code style: [your preferences]

## Standing Instructions
- Always check this file before starting work
- Update when something important changes
EOF
```

Update this file regularly. It's the difference between an agent that knows you and one that starts from zero every time.

---

## Step 6: Create the memory/ Directory

The `memory/` directory holds daily context logs written by your agent — what happened, what decisions were made, what's next.

```bash
mkdir -p ~/.openclaw/workspace/memory
```

Create today's log stub:

```bash
DATE=$(date +%Y-%m-%d)
cat > ~/.openclaw/workspace/memory/$DATE.md << EOF
# $DATE

## What Happened
-

## Decisions Made
-

## Blockers
-

## Next Steps
-
EOF
```

Your agent will fill this in during sessions. You can also write to it manually. Think of it as a shared logbook — both you and your agents contribute.

---

## Step 7: Connect Telegram

Telegram is the best interface for OpenClaw — persistent, mobile-accessible, and multi-topic via Forum Groups.

### Create a Bot

1. Message `@BotFather` on Telegram
2. Send `/newbot`
3. Name it (e.g., "AgentOrchard Assistant")
4. Copy the token: `1234567890:ABCdef...`

### Configure OpenClaw

Add the bot token to your OpenClaw config:

```bash
openclaw config set telegram.botToken "YOUR_TOKEN_HERE"
openclaw config set telegram.mode "polling"
```

Or edit `~/.openclaw/config.json` directly:

```json
{
  "plugins": {
    "telegram": {
      "botToken": "YOUR_TOKEN_HERE",
      "mode": "polling"
    }
  }
}
```

### Start the Gateway

```bash
openclaw gateway start
```

Verify it's running:

```bash
openclaw gateway status
```

Message your bot on Telegram. You should get a response within a few seconds.

---

## Step 8: Set Your First Cron — Morning Greeting

Crons let your agent do things on a schedule without you initiating. A morning greeting is the simplest and most useful starting point.

Add to `~/.openclaw/workspace/AGENTS.md` (or wherever your crons are configured):

```markdown
## Crons

### Morning Greeting
- Schedule: 0 8 * * *  (8am daily)
- Task: Read MEMORY.md and today's memory log. Send a morning briefing to Telegram with:
  1. Any pending tasks from yesterday
  2. Today's date and day of week
  3. One sentence of encouragement
```

Or use the CLI:

```bash
openclaw cron add \
  --schedule "0 8 * * *" \
  --task "Morning briefing: read MEMORY.md, summarize pending tasks, wish me good morning"
```

Restart the gateway to activate:

```bash
openclaw gateway restart
```

---

## Step 9: Common Mistakes (and How to Avoid Them)

**Mistake 1: Skipping SOUL.md**
The agent without a soul is just a generic chatbot. Even three sentences of personality makes a huge difference. Write SOUL.md before your first real conversation.

**Mistake 2: Never updating MEMORY.md**
If you don't update MEMORY.md, the agent's long-term context goes stale. Make it a habit: when something important changes (new project, new preference, new constraint), update MEMORY.md immediately.

**Mistake 3: Using Opus for everything**
Claude Opus costs 5× more than Sonnet and 15× more than Haiku. For monitoring crons and simple tasks, use Haiku. Reserve Opus for strategy and synthesis. See Step 11 for the full cost breakdown.

**Mistake 4: Hardcoding secrets in workspace files**
Don't put API keys, passwords, or tokens directly in MEMORY.md or AGENTS.md. Use `openclaw config set` for secrets — they get encrypted. Workspace files are plaintext.

**Mistake 5: Running one giant session for everything**
Context rot is real. Long sessions with many context switches drop accuracy from ~90% to ~30%. Start fresh sessions for distinct tasks. Use MEMORY.md to pass context between sessions, not the conversation history.

---

## Step 10: Model Selection Basics

| Model | Best For | API Cost (per 1M tokens) |
|-------|----------|--------------------------|
| Claude Opus 4.6 | Strategy, orchestration, complex synthesis | $15 in / $75 out |
| Claude Sonnet 4.6 | Coding, content, research | $3 in / $15 out |
| Claude Haiku 3.5 | Monitoring, crons, quick tasks | $1 in / $5 out |
| GPT-5.4 | When you need OpenAI compatibility | $2.50 in / $10 out |
| Gemini 2.5 Pro | Long documents, bulk processing | Free via CLI |

**Quick routing rule:**
- Is it a cron or heartbeat? → Haiku
- Is it coding or writing? → Sonnet
- Is it strategic planning or multi-agent orchestration? → Opus
- Is it processing a 200-page PDF? → Gemini

Set your default model:

```bash
openclaw config set model "anthropic/claude-sonnet-4-6"
```

---

## Step 11: Budget Planning

**API pricing reality check (monthly estimates):**

| Usage Level | Models Used | Est. Monthly Cost |
|-------------|-------------|-------------------|
| Light (personal, 1-2 tasks/day) | Mostly Haiku | $5–$15 |
| Medium (solopreneur, 5-10 tasks/day) | Mix of Sonnet + Haiku | $30–$80 |
| Heavy (multi-agent ops, crons, content pipeline) | Mix of Opus + Sonnet + Haiku | $100–$300 |

**Flat-rate alternative:** If you're a heavy API user, Claude Max ($200/mo) covers most personal use. GPT-5.4 Pro ($200/mo) similarly. Run the numbers: if you're spending $150/mo on API, the flat rate is worth it.

**Budget guardrails:**
```bash
openclaw config set budget.monthlyLimitUSD 100
openclaw config set budget.alertAtPercent 80
```

This sends you a Telegram alert when you hit 80% of your monthly budget.

---

## Step 12: What Next — Your Roadmap

Once you have the basics running, here's the recommended progression:

**Week 1–2: Foundation**
- [ ] SOUL.md, MEMORY.md, memory/ directory in place
- [ ] Telegram bot connected and responding
- [ ] Morning briefing cron running
- [ ] Daily memory logs being written

**Month 1: Automation**
- [ ] Add a research cron (weekly briefing on a topic you care about)
- [ ] Set up content drafting workflow
- [ ] Connect n8n or add custom crons for monitoring tasks
- [ ] Review and refine SOUL.md based on actual conversations

**Month 2: Multi-agent**
- [ ] Set up Telegram Forum Group with topics (see Telegram Handbook)
- [ ] Spawn sub-agents for specific tasks (coding, research, content)
- [ ] Build your first skill
- [ ] Add AGENTS.md with role definitions

**Month 3+: Revenue**
- [ ] Deploy a public-facing product built with agent assistance
- [ ] Automate content distribution pipeline
- [ ] Set up client intake and delivery workflows
- [ ] Run your first external consulting session using the agent as a thinking partner

---

## Quick Reference: Essential Commands

```bash
openclaw init              # Initialize workspace
openclaw chat              # Start a chat session
openclaw gateway start     # Start the gateway daemon
openclaw gateway status    # Check gateway health
openclaw gateway restart   # Restart after config changes
openclaw config set <key> <value>   # Set a config value
openclaw cron list         # List all crons
openclaw status            # Full system status
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
