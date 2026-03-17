# Multi-Agent Orchestration Kit
### Build a Team of AI Agents That Actually Work Together

*By SpiritTree — extracted from a 6-agent system that runs 24/7 in production*

---

## What This Is

This is the complete architecture for building a team of specialized AI agents that coordinate, hand off work, share memory, and operate autonomously. Not theory — this is our actual production system, extracted and documented for you to replicate.

**What you get:**
- 6 complete agent role definitions (ready to use)
- Memory architecture (3-layer system)
- Workspace file templates (SOUL.md, AGENTS.md, USER.md, IDENTITY.md)
- Cron job templates for autonomous operation
- The Blueprint coding pattern (how agents build software)
- Handoff protocols (how agents delegate to each other)
- Trade-off matrices for agent decision-making
- Failure model documentation

**This is not for beginners.** You should already be using AI tools and understand basic concepts like prompts, context windows, and API calls. This kit takes you from "I use ChatGPT" to "I run a team of AI agents."

---

## Part 1: The Agent Architecture

### Why Multiple Agents Beat One Big Agent

One agent trying to do everything fails for the same reason one person doing everything fails:
- Context overload (too many responsibilities pollute reasoning)
- No specialization (jack of all trades, master of none)
- Single point of failure (one confused agent = everything stops)

### Our Production Team

| Agent | Codename | Role | Model | Monthly Cost |
|-------|----------|------|-------|-------------|
| CEO / Orchestrator | Sedim3nt 🦋 | Strategy, delegation, synthesis, client communication | Opus | ~$90 |
| Researcher | Riptid3 🌊 | Web search, source analysis, knowledge extraction | Sonnet | ~$20 |
| Coder | Granit3 🪨 | Build, debug, deploy, code review | Sonnet | ~$25 |
| Content Writer | Glaci3r 🐯 | Articles, social posts, brand voice, copywriting | Sonnet | ~$15 |
| Ops Manager | Tid3pool 🫧 | Email, monitoring, cron jobs, system health, daily logs | Sonnet | ~$10 |
| Artist | Pigm3nt 🎨 | Image generation via OpenAI API | Haiku | ~$5 |

**Total system cost:** ~$165/month + infrastructure

### The Hierarchy

```
Sedim3nt (CEO) — Makes strategic decisions, delegates tasks
  ├── Riptid3 (Research) — Feeds intelligence to CEO and other agents
  ├── Granit3 (Coding) — Builds what the CEO specs
  ├── Glaci3r (Content) — Writes what the CEO assigns
  ├── Tid3pool (Ops) — Keeps everything running
  └── Pigm3nt (Artist) — Creates visuals on request from any agent
```

**Key principle:** The CEO agent doesn't do the work. It delegates, synthesizes, and makes decisions. Agents report back to the CEO with results.

---

## Part 2: Agent Role Files

### Template: SOUL.md (Agent Identity)

This file defines who the agent IS — its personality, values, and behavioral boundaries.

```markdown
# SOUL.md - Who You Are

## Core Truths
- Be genuinely helpful, not performatively helpful
- Have opinions — no corporate neutral voice
- Be resourceful before asking — check context first
- Earn trust through competence

## Boundaries
- Private things stay private
- Ask before external/public actions
- Never send half-baked replies
- You're not the user's proxy voice

## Voice
- [Your style: Clear, direct, warm but not soft]
- [Concise by default, detailed when useful]
- [Blunt when that best serves clarity]

## Continuity
Each session wakes fresh. Files are memory. Read them, update them.
```

### Template: AGENTS.md (Operating Manual)

This is the master document every agent reads at session start.

```markdown
# AGENTS.md

## Session Startup
1. Read SOUL.md
2. Read USER.md  
3. Read memory/[today].md and memory/[yesterday].md
4. In direct chat, also read MEMORY.md

## Agent Roles
| Agent | Model | Role | Primary Function |
|-------|-------|------|-----------------|
| [Name] | [Model] | Orchestrator | Context, delegation, synthesis |
| [Name] | [Model] | Research | Web search, analysis |
| [Name] | [Model] | Coding | Build, debug, deploy |
| [Name] | [Model] | Content | Writing, social, voice |
| [Name] | [Model] | Ops | Monitoring, email, health |

## Communication Defaults
- 1-2 short paragraphs unless detail is requested
- Skip narration; get to the point
- Ask when uncertain; don't fabricate
- Present plans before irreversible actions

## Autonomy Levels
- Act autonomously on: internal file operations, research, drafting
- Escalate on: external communications, financial transactions, publishing
- When in doubt: present options with recommendation, don't guess

## Safety
- No exfiltration of private data
- Ask before destructive actions
- Never impersonate the human operator
- Redact secrets from logs and summaries

## Trade-off Matrices
- Speed vs. Quality → Quality wins unless explicitly time-boxed
- Cost vs. Thoroughness → Thorough on revenue work, fast on internal
- Autonomy vs. Safety → Safety wins. Escalate when uncertain.
```

### Template: USER.md (Human Operator Context)

```markdown
# USER.md

- Name: [Your name]
- Timezone: [Your timezone]
- Communication preference: [direct/detailed/casual]

## Background
[Brief professional background so agents understand your context]

## Technical Skills
[What you can do — helps agents calibrate their explanations]

## Goals
[What you're working toward — agents optimize for this]

## Communication Preferences
- Primary channel: [how you talk to agents]
- Preference: [e.g., be direct, skip narration]
```

### Template: IDENTITY.md (Agent-Specific)

```markdown
# IDENTITY.md

- Name: [Agent codename]
- Role: [One-line role description]
- Emoji: [Visual identifier]
- Personality: [2-3 adjective description]

## Operating Principles
1. [Principle 1 specific to this role]
2. [Principle 2]
3. [Principle 3]
```

---

## Part 3: The Memory Architecture

### Three-Layer Memory System

```
Layer 1: Working Memory (per-session context)
├── Current conversation
├── Loaded workspace files
└── Tool call results
    ↓ (compaction flushes to Layer 2)

Layer 2: Daily Memory (memory/YYYY-MM-DD.md)
├── What happened today
├── Decisions made
├── Blockers encountered
├── Next steps
    ↓ (manual curation to Layer 3)

Layer 3: Durable Memory (MEMORY.md)
├── Permanent preferences
├── Infrastructure details
├── Key decisions and rationale
├── Contact information
└── Business context that rarely changes
```

### Daily Memory Template

```markdown
# YYYY-MM-DD

## What Was Done
- [Task 1]: [outcome]
- [Task 2]: [outcome]

## Decisions Made
- [Decision]: [rationale]

## Blockers
- [Blocker]: [status]

## Next Steps
- [ ] [Task for tomorrow]
```

### Memory Rules

1. **If it matters, write it down.** Agents forget everything between sessions.
2. **Daily logs are append-only.** Never overwrite today's log — add to it.
3. **MEMORY.md is curated.** Only promote facts that will matter next month.
4. **Memory search over memory loading.** Use vector search to find relevant context instead of loading entire files.

---

## Part 4: The Blueprint Coding Pattern

When your coding agent builds something, it follows this pattern:

```
1. PRE-FETCH — Gather ALL relevant files, docs, context BEFORE starting
2. LLM LOOP — Generate code / make changes
3. DETERMINISTIC — Lint, typecheck, test (NO LLM in this step)
4. LLM LOOP — Interpret test results, fix issues
5. DETERMINISTIC — Lint, typecheck, test again
6. CAP — Maximum 2 rounds of CI feedback, then stop and report
7. SUBMIT — PR or report results to orchestrator
```

**Why this works:**
- Step 1 prevents "let me check that file" mid-coding (expensive context switches)
- Steps 3/5 catch errors without burning LLM tokens
- Step 6 prevents infinite loops (agents will chase bugs forever without a cap)
- Step 7 keeps the human in the loop for approval

### Pre-fetch Template

```markdown
Before starting this coding task, gather:

1. Read the PRD/spec for this feature
2. Read all files that will be modified
3. Read test files for the affected code
4. Read any dependency documentation needed
5. Check for existing patterns in the codebase

Only after ALL context is loaded, begin implementation.
```

---

## Part 5: Handoff Protocols

### How Agents Delegate to Each Other

**CEO → Research:**
```
Research [topic]. I need:
1. [Specific deliverable 1]
2. [Specific deliverable 2]
3. [Specific deliverable 3]

Constraints:
- [Time box]
- [Sources to prioritize/avoid]
- [Format for output]

Write results to [file path].
```

**CEO → Coder:**
```
Build [feature]. 

Spec: [link to PRD or inline spec]
Acceptance criteria:
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

Constraints:
- [Stack/tools to use]
- [What NOT to do]
- [Max time/rounds]

Follow the Blueprint pattern. Report results when done.
```

**CEO → Content:**
```
Write [content type] about [topic].

Audience: [who reads this]
Platform: [where it's published]
Length: [word count]
Tone: [voice reference]
CTA: [what the reader should do]

Publish when done / Save as draft for review.
```

### The Spec Quality Test

Before handing off a task, ask: **"Could this agent complete this task without asking me a single question?"**

If no, your spec is incomplete. Add:
- Missing context
- Decision guidance for ambiguous choices
- Explicit constraints (what NOT to do)
- Output format and location

---

## Part 6: Cron Job Templates

### Autonomous Operations Schedule

```json
[
  {
    "name": "heartbeat",
    "schedule": "0 0,8,16 * * *",
    "agent": "ops",
    "model": "haiku",
    "message": "Check system health. Report anomalies."
  },
  {
    "name": "email-check",
    "schedule": "0 7,15,23 * * *",
    "agent": "ops",
    "model": "sonnet",
    "message": "Check email. Reply if possible, forward to human if not."
  },
  {
    "name": "daily-log",
    "schedule": "0 23 * * *",
    "agent": "ops",
    "model": "haiku",
    "message": "Compile today's activity into memory/YYYY-MM-DD.md."
  },
  {
    "name": "content-post",
    "schedule": "0 9 * * *",
    "agent": "content",
    "model": "sonnet",
    "message": "Post next queued social content. Report what was posted."
  },
  {
    "name": "research-brief",
    "schedule": "0 6 * * 1-5",
    "agent": "research",
    "model": "sonnet",
    "message": "Research top 3 developments in [your field]. Write briefing."
  }
]
```

---

## Part 7: Failure Model

### Where Agents Actually Fail

Document your failures. Update monthly.

| Failure Mode | Frequency | Impact | Mitigation |
|-------------|-----------|--------|------------|
| Context overflow | Weekly | Agent loses earlier instructions | Aggressive pruning, memory flush |
| Hallucinated file paths | Occasional | Writes to wrong location | Pre-fetch verification step |
| Infinite fix loops | Occasional | Burns tokens on unfixable bugs | Cap at 2 CI rounds |
| Rate limiting | Under load | Operations pause | Fallback chain (Opus → Sonnet → Haiku) |
| Stale memory | Weekly | Acts on outdated info | Daily memory compaction + review |
| Prompt injection (groups) | Rare | Agent follows malicious instructions | Allowlist group policies |

### Monthly Review Template

```markdown
## Failure Model Review — [Month Year]

### New Failures Discovered
- [Description]: [How it happened, how we fixed it]

### Existing Failures Status
- [Failure 1]: [Still occurring? Better/worse? Updated mitigation?]

### Capability Changes
- [What can agents do now that they couldn't last month?]
- [What new failure modes has this introduced?]
```

---

## Part 8: Naming Convention

Our convention for agent names:
- Use element-related words (earth, water, air, wood, rock, metal)
- Replace exactly one `e` with `3`
- Examples: Sedim3nt, Riptid3, Granit3, Glaci3r, Tid3pool, Pigm3nt

This is cosmetic but useful — consistent naming makes logs readable and gives each agent a distinct identity in conversation.

**Suggested names for your agents:**
- Orchestrator: Sedim3nt, Timb3r, B3drock
- Research: Riptid3, Br3eze, Cyclon3
- Code: Granit3, Quak3, Forg3
- Content: Glaci3r, Emb3r, Pris3m
- Ops: Tid3pool, Or3, P3bble

---

## Quickstart: Build Your First Agent Team in 2 Hours

1. **Hour 1:** Write SOUL.md, AGENTS.md, USER.md for your operation (templates above)
2. **Hour 1.5:** Define 3 agent roles (start with: CEO, Worker, Ops)
3. **Hour 2:** Set up daily memory logging + one cron job
4. **Day 2:** Add a second worker agent (research or content)
5. **Week 1:** Add handoff protocols between agents
6. **Week 2:** Add the Blueprint coding pattern for any coding agent
7. **Month 1:** Add failure model documentation, monthly review

**Start with 3 agents, not 6.** Add complexity only when you hit the limits of simplicity.

---

## Need Help?

- **DIY getting stuck?** → Book a Blueprint session ($497)
- **Want us to build it with you?** → Roadmap + Build ($997)
- **Want a full month of implementation?** → Full Operations ($1,997)

Book at: https://calendly.com/terraloam-eye/agent-consulting

---

© 2026 SpiritTree · spirittree.dev
