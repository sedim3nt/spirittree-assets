# Multi-Agent Communication Architecture — OpenClaw Agent Coordination

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 60-90 minutes

The complete communication architecture for multi-agent OpenClaw systems. Covers how agents delegate to each other, share memory, handle handoffs, and coordinate across a single Telegram interface — from a single CEO agent all the way to a 6-agent swarm.

---

## What's In This Package

- Single CEO + sub-agent delegation pattern (the recommended starting point)
- Session spawning and management (`sessions_spawn`, `sessions_yield`)
- Memory sharing across agents (shared workspace vs. isolated workspaces)
- Handoff protocols (how agents pass context to each other)
- Push-based completion (results auto-announce back — no polling)
- BULLETIN.md cross-topic coordination system
- Agent role definitions for clean delegation (no ambiguous ownership)
- Communication failure recovery patterns
- Multi-channel routing (different agents respond to different Telegram topics)
- Load balancing across agent types
- Trust and permission levels between agents
- Production examples from a 6-agent system

---

# Multi-Agent Communication Architecture for OpenClaw

**Compiled:** 2026-03-14
**Context:** Analysis for Suede/Orchard setup — single CEO agent on Telegram delegating to sub-agents
**Audience:** OpenClaw operators running multi-agent systems via Telegram

---

## The Core Question

Should you run one bot (CEO model) or multiple bots with separate Telegram presences?

**Short answer:** Start with the CEO model using topic threads. Graduate to separate bots only when you hit specific scaling walls.

---

## Part 1: Architecture Patterns

### Pattern A: Single CEO Bot (What Suede Runs Now)

```
┌──────────┐
│   You     │
│  (Human)  │
└────┬──────┘
     │ Telegram DM
     ▼
┌──────────────────┐
│   Orchard (CEO) │  ← Opus 4.6, main session
│   All context     │
│   All decisions   │
└────┬─────┬───────┘
     │     │  spawns sub-agents via OpenClaw
     ▼     ▼
  ┌─────┐ ┌─────┐ ┌─────┐
  │ Dev │ │ Ops │ │ Res │  ← ephemeral sub-agents
  └─────┘ └─────┘ └─────┘    (no Telegram presence)
```

**How it works:**
- You talk to Orchard in one chat
- He spawns sub-agents via `sessions_spawn` or `exec` for specific tasks
- Sub-agents report back to Orchard, who synthesizes and relays to you
- All coordination flows through a single context window

**Strengths:**
- Simplest to operate — one bot, one chat, one relationship
- CEO holds full picture — can make cross-domain decisions
- Sub-agents inherit workspace automatically
- No Telegram bot token juggling
- Natural for how humans think about delegation

**Weaknesses:**
- **Token economics:** Every sub-agent report gets relayed through Opus ($15/M input, $75/M output). A 2K-token DevOps report costs ~$0.15-0.30 just to relay through the CEO when it could go direct for ~$0.01 on Haiku
- **Context window pressure:** At 200K tokens, if Orchard is managing 3 active projects + monitoring + personal tasks, compaction hits faster. Each compaction loses specifics
- **Serial bottleneck:** Orchard processes one turn at a time. If a security alert fires while he's deep in a coding task, the alert waits
- **Blast radius:** If Orchard's session gets corrupted or stuck, everything stops
- **Compaction amnesia:** When context compacts, Orchard may forget what sub-agents were working on unless explicitly written to files

---

### Pattern B: CEO + Topic Threads (What I Recommend)

```
┌──────────────────────────────────────┐
│         Telegram Group (HQ)          │
│                                      │
│  📌 General (topic 1) ← You + CEO   │
│  💻 Dev (topic 2)     ← own session │
│  🔧 Infra (topic 3)   ← own session │
│  📊 Research (topic 4) ← own session │
│  💰 Trading (topic 5)  ← own session │
│  🔒 Security (topic 6) ← own session │
│                                      │
│  ALL topics = same bot, same gateway │
│  Each topic = separate OpenClaw      │
│  session with isolated context       │
└──────────────────────────────────────┘
```

**How it works:**
- Create a Telegram supergroup with Forum Topics enabled
- One bot (Orchard) joins the group
- OpenClaw automatically creates separate sessions per topic
- Each topic gets its own context window, memory, and compaction cycle
- You post in whichever topic is relevant — the bot responds with domain-specific context
- Cross-topic coordination via shared workspace files (BULLETIN.md, memory/, etc.)

**Strengths:**
- **Context isolation** — dev discussion doesn't pollute trading context (and vice versa)
- **Parallel processing** — topics can run concurrently (OpenClaw handles this)
- **Domain-specific memory** — each topic accumulates relevant context over time
- **Still one bot** — you configure one Telegram token, one gateway
- **Human-friendly** — you naturally organize by posting in the right topic
- **Compaction is scoped** — when /dev compacts, /trading keeps its full context
- **Can use different models per topic** — Haiku for monitoring, Opus for strategy

**Weaknesses:**
- **No automatic cross-topic awareness** — if you discuss a decision in /general that affects /dev, you need to manually mention it or the agent needs to check BULLETIN.md
- **More configuration** — need to set up topics in gateway config
- **File coordination discipline** — agents must write to shared files, not just respond in-thread
- **Can't override model per-topic natively** (yet) — but cron jobs can target specific topics with different models

**Setup (OpenClaw config):**
```json5
{
  channels: {
    telegram: {
      groups: {
        '-100XXXXXXXXX': {  // your group ID
          requireMention: false,
          topics: {
            '1': { enabled: true },    // General
            '23': { enabled: true },   // Dev
            '24': { enabled: true },   // Infra
            // ... etc
          }
        }
      }
    }
  }
}
```

---

### Pattern C: Multiple Separate Bots

```
┌──────────┐
│   You     │
└──┬──┬──┬─┘
   │  │  │   separate Telegram chats/groups
   ▼  ▼  ▼
┌────┐┌────┐┌────┐
│CEO ││Dev ││Ops │  ← separate bot tokens
│Opus││Son.││Hai.│  ← separate OpenClaw agents
│DM  ││GH  ││Cron│  ← separate channels
└──┬─┘└──┬─┘└──┬─┘
   │     │     │
   └──┬──┘     │   shared workspace
      └────┬───┘
           ▼
    ┌─────────────┐
    │  Filesystem  │
    │  BULLETIN.md │
    │  memory/     │
    └─────────────┘
```

**How it works:**
- Each agent is a separate OpenClaw agent with its own bot token
- Each has its own Telegram presence (different bot username)
- They share a workspace directory for coordination
- You talk to whichever agent is relevant, or let them coordinate via files

**Strengths:**
- **True parallelism** — agents literally run simultaneously with zero contention
- **Model optimization** — Dev runs on Codex/Sonnet, Ops on Haiku, CEO on Opus
- **Independent lifecycle** — restart one without affecting others
- **Cost control** — monitoring on Haiku costs pennies vs. Opus dollars
- **Fault isolation** — one agent crashing doesn't take down the system

**Weaknesses:**
- **Operational complexity** — multiple bot tokens, multiple configs, multiple sessions to monitor
- **Coordination overhead** — agents can't see each other's conversations, only shared files
- **User context switching** — you need to remember which bot to talk to for what
- **⚠️ Shared workspace race condition** — known OpenClaw issue (#39701): `agents delete` can trash shared workspace. Use separate workspaceDirs or be very careful
- **Duplicate token burn** — each agent loads its own system prompt, AGENTS.md, etc.
- **No native inter-agent messaging** — must coordinate through files or external channels

**When this makes sense:**
- You have genuinely independent workstreams (e.g., a DevOps bot that watches CI/CD independently)
- You want to run different models for cost optimization
- You need agents that respond to external events (webhooks, alerts) without going through a coordinator
- Your CEO agent is hitting token budget limits from relay overhead

---

## Part 2: Decision Framework

### Start With Pattern B (Topics) If:
- ✅ You run 1-4 workstreams
- ✅ You want one bot to talk to
- ✅ You need domain isolation without operational complexity
- ✅ You're the primary (or only) human operator
- ✅ You want to keep costs predictable

### Graduate to Pattern C (Separate Bots) When:
- 🔄 You need truly independent event-driven agents (monitoring that alerts at 3AM)
- 🔄 Token costs from routing everything through Opus become significant (>$50/day)
- 🔄 You have 5+ distinct workstreams that rarely cross-reference
- 🔄 Multiple humans need to interact with different agents independently
- 🔄 You want to assign different risk profiles (one agent gets shell access, another doesn't)

### Stay With Pattern A (Single CEO) If:
- ✅ You're just getting started
- ✅ Your workstreams are tightly coupled (decisions in one affect another)
- ✅ You value simplicity over optimization
- ✅ Total daily token usage is under ~$20

---

## Part 3: The Token Economics

### Relay Cost Analysis

When the CEO relays sub-agent output, you pay twice:
1. Sub-agent generates the report (their model cost)
2. CEO reads and relays it (Opus cost to process + respond)

**Example: Daily DevOps report (2K tokens)**

| Pattern | Model | Cost per relay |
|---------|-------|---------------|
| A (CEO relay) | Opus reads 2K + responds 500 | ~$0.07 |
| B (Topic direct) | Sonnet reads 2K + responds 500 | ~$0.01 |
| C (Separate Haiku bot) | Haiku reads 2K + responds 500 | ~$0.002 |

Over 30 days with 10 daily reports:
- Pattern A: ~$21/month just for relay
- Pattern B: ~$3/month
- Pattern C: ~$0.60/month

**The relay tax is real.** For high-frequency, low-complexity updates (monitoring, status checks, log summaries), routing through Opus is wasteful.

### Where Opus Relay IS Worth It
- Synthesizing across domains ("how does this deploy affect our trading position?")
- Making judgment calls that need full context
- Communicating with you (the human) about complex topics
- Any decision that could have irreversible consequences

---

## Part 4: Cross-Agent Memory & Coordination

Regardless of which pattern you choose, agents need to share state. Here's what works:

### File-Based Coordination (Works Today)

```
workspace/
├── BULLETIN.md          # Cross-agent announcements
│                         # Timestamped, max ~10 entries
│                         # Every agent reads on startup
│
├── memory/
│   ├── YYYY-MM-DD.md    # Shared daily log
│   ├── agent-sediment/  # CEO's private memory
│   ├── agent-dev/       # Dev agent's private memory
│   └── entities/        # Shared entity files
│       ├── projects/
│       └── people/
│
├── shared/
│   ├── handoffs/        # Agent-to-agent task handoffs
│   │   └── dev-to-ops-deploy-v2.md
│   └── status/          # Current status per agent
│       ├── sediment.json
│       ├── dev.json
│       └── ops.json
│
└── TODO.md              # Shared task queue
```

### BULLETIN.md Protocol
```markdown
# BULLETIN.md — Cross-Agent Announcements

## 2026-03-14 16:30 [dev]
Deployed v2.3.1 to staging. All tests passing.
Breaking change: API endpoint /v1/users renamed to /v2/users.
@ops: update monitoring targets. @sediment: update client docs.

## 2026-03-14 14:00 [ops]  
SSL cert for api.example.com expires in 7 days. Auto-renewal configured.
No action needed unless renewal fails.
```

### Status File Protocol
Each agent writes a lightweight status JSON:
```json
{
  "agent": "dev",
  "lastActive": "2026-03-14T16:30:00Z",
  "currentTask": "PR review for feature/auth-v2",
  "blockers": [],
  "recentCompletions": ["deploy v2.3.1", "fix auth bug #234"],
  "pendingHandoffs": ["ops: update monitoring for v2 endpoints"]
}
```
CEO reads all status files to get a dashboard view without burning tokens on direct relay.

---

## Part 5: Practical Setup Guide

### For Suede's Setup (CEO + Topics)

**Step 1: Create a Telegram Supergroup**
- Create group → Settings → Toggle "Topics" on
- Add Orchard bot to the group
- Create topics: General, Dev, Ops, Research (or whatever domains you have)

**Step 2: Configure OpenClaw**
Add the group to your gateway config with topics enabled (see Pattern B config above).

**Step 3: Set Up Cross-Topic Coordination**
- Create BULLETIN.md in workspace root
- Add to AGENTS.md: "Read BULLETIN.md every session. Write significant events to it."
- Create `shared/status/` directory

**Step 4: Configure Memory Flush Per Domain**
Each topic can have a customized flush prompt. In AGENTS.md, add topic-aware instructions:
```
When in a Dev topic: prioritize saving code decisions, PR states, deploy info.
When in Ops topic: prioritize saving infrastructure state, alert history, config changes.
When in General: prioritize saving cross-domain decisions and handoffs.
```

**Step 5: Set Up Cron Jobs for Background Work**
- Health checks → target the Ops topic (use Haiku model)
- Trading sentinel → target Trading topic
- Daily summary → target General topic (synthesize across domains)

---

## Part 6: The Orchard-Specific Recommendation

Given what Suede described:
- Orchard is the CEO on Opus (Claude Max primary)
- Has Gemini Pro for secondary model access
- Has Gmail/Google tools + TOTP self-auth
- Currently single DM chat

**Recommended evolution:**

1. **Now:** Keep DM for personal/quick interactions. Create HQ group with topics for structured work.

2. **Month 1:** Add 3-4 topics (Dev, Ops, Research, General). Let Orchard learn to use them. Write BULLETIN.md discipline into AGENTS.md.

3. **Month 2:** If any topic is burning significant tokens on routine work (monitoring, daily syncs), consider spinning that into a separate Haiku-powered agent.

4. **Month 3+:** Evaluate whether Gemini Pro should power a dedicated research agent (its long context window is wasted on short monitoring tasks but perfect for deep analysis).

**The key insight:** Don't optimize prematurely. The CEO model is fine until you feel the pain. Then graduate the specific domain that's causing the pain, not everything at once.

---

## References

- OpenClaw Multi-Agent Orchestration: https://zenvanriel.com/ai-engineer-blog/openclaw-multi-agent-orchestration-guide/
- OpenClaw Multi-Agent Team Setup: https://www.mejba.me/blog/openclaw-agent-team-configuration
- Multiple Concurrent Agents Discussion: https://www.answeroverflow.com/m/1471453972932984956
- Shared Workspace Race Condition: https://github.com/openclaw/openclaw/issues/39701
- OpenClaw Security Architecture: https://nebius.com/blog/posts/openclaw-security
- Memory Solutions Compendium: memory-solutions-compendium.md (companion doc)
