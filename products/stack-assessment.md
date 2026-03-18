# The 9-Layer Stack Assessment — Audit Any AI Operations Setup

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 2-3 hours (for a full assessment)

Map the complete surface area of your AI operation. This framework identifies gaps, prioritizes improvements, and builds a remediation roadmap across 9 layers — from hardware substrate to community output. Used in $497-$1,997 consulting engagements.

---

## What's In This Package

- 9-layer framework (L0 Substrate through L8 Commons) with full definitions
- Key diagnostic question for each layer
- "What most people have" vs. "what they're missing" for each layer
- Scoring rubric per layer (0–4 coverage score)
- Priority ranking guide (which layers to fix first)
- Gap analysis template (copy-paste into any doc)
- 104 primitives reference list (every tool/concept that belongs in each layer)
- Remediation roadmap template (90-day improvement plan)
- Sample assessment from a real client engagement (anonymized)
- Self-assessment vs. third-party assessment guide
- Quarterly re-assessment protocol

---

# The 9-Layer Stack Assessment
## A Framework for Auditing and Improving Any AI Operations Setup

This framework maps the complete surface area of an autonomous AI operation. Use it to identify gaps, prioritize improvements, and build a remediation roadmap.

---

## Overview: The 9 Layers

| Layer | Name | Function |
|-------|------|----------|
| L0 | Substrate | The machine and runtime environment |
| L1 | Model Access | Which models you can use and how |
| L2 | Orchestration | How agents are directed and coordinated |
| L3 | Memory | How context persists across sessions |
| L4 | Interface | How you interact with the system |
| L5 | Automation | Scheduled and triggered work |
| L6 | Integration | Connections to external services |
| L7 | Output | What the system produces |
| L8 | Commons | Knowledge base and shared resources |

---

## Layer Assessments

### L0 — Substrate

**What it is:** The physical or virtual machine running your agent, plus the runtime environment (OS, Node.js, Python, etc.).

**Key question:** Is this machine dedicated, properly configured, and reliable?

**What most clients have:**
- Their main laptop running OpenClaw between personal tasks
- No dedicated hardware
- Shared development environment with other tools

**What they're missing:**
- Dedicated machine (Mac Mini, Raspberry Pi, VPS) for always-on operations
- Proper process management (LaunchAgents, systemd) for automatic restart
- Regular backups of workspace and config
- Isolation from personal/client work

**Primitives at this layer:**
- Hardware: Mac Mini, Raspberry Pi 4/5, VPS (DigitalOcean, Hetzner, Linode)
- OS management: macOS, Ubuntu Server, NixOS
- Process management: launchctl (macOS), systemd (Linux)
- Backup: Time Machine, rsync, borgbackup
- Monitoring: htop, healthcheck crons

**Coverage scoring:**
- 0: No dedicated setup, runs manually
- 1: Dedicated machine OR proper process management, not both
- 2: Dedicated machine + process management
- 3: Dedicated machine + process management + backups
- 4: Full: dedicated + process management + backups + monitoring + isolation

---

### L1 — Model Access

**What it is:** Which AI models you have access to, via what mechanism (API vs. flat-rate vs. local), and how you manage costs.

**Key question:** Can you route different tasks to different models based on cost and capability?

**What most clients have:**
- One API key from one provider
- No cost monitoring
- Default model for everything

**What they're missing:**
- Multi-provider setup (Anthropic + OpenAI + Gemini)
- Per-task model routing (Haiku for crons, Sonnet for ops, Opus for strategy)
- Budget alerts and monthly cost tracking
- Fallback provider when primary is down

**Primitives at this layer:**
- Providers: Anthropic, OpenAI, Google Gemini, Mistral, local Ollama
- Routing: openclaw config model routing
- Cost tracking: API dashboard alerts + openclaw budget config
- Flat-rate plans: Claude Max, GPT Pro

**Coverage scoring:**
- 0: One model, no cost awareness
- 1: Multiple models available, no routing
- 2: Active model routing by task type
- 3: Routing + cost monitoring + budget alerts
- 4: Full: routing + monitoring + fallback + flat-rate/API optimization

---

### L2 — Orchestration

**What it is:** How you direct and coordinate agent work. The rules, roles, and protocols that govern what agents do.

**Key question:** Do your agents have clear roles, consistent behavior, and can they coordinate with each other?

**What most clients have:**
- AGENTS.md if they've done any setup
- Single agent doing everything
- No sub-agent delegation

**What they're missing:**
- Clear agent roles with defined scope (like the Orchard/Rowan/Forrest structure)
- Delegation protocols (when to spawn a sub-agent vs. handle directly)
- Error recovery procedures (what happens when a sub-agent fails)
- Cross-agent coordination (BULLETIN.md or equivalent)

**Primitives at this layer:**
- AGENTS.md role definitions
- Sub-agent spawning protocols
- BULLETIN.md coordination file
- FAILURE_MODEL.md documentation
- Escalation rules (what requires human approval)

**Coverage scoring:**
- 0: No orchestration, all ad-hoc
- 1: AGENTS.md exists, basic role definitions
- 2: Multiple agent roles + delegation protocols
- 3: Delegation + coordination + failure handling
- 4: Full: multi-agent + coordination + failure model + escalation rules

---

### L3 — Memory

**What it is:** How context, history, and knowledge persist across sessions so the agent doesn't start from zero.

**Key question:** Can your agent recall what happened last week and build on it?

**What most clients have:**
- No memory files
- Relying on conversation history (lost when context window clears)
- Recreating context at the start of every session

**What they're missing:**
- MEMORY.md (curated long-term memory)
- Daily memory logs (memory/YYYY-MM-DD.md)
- Project-specific memory files
- Memory update protocols (when and what to write)

**Primitives at this layer:**
- MEMORY.md (long-term)
- memory/YYYY-MM-DD.md (daily logs)
- memory/open-projects.md (active work tracker)
- memory/FAILURE_MODEL.md (what goes wrong)
- SOUL.md (persistent identity/values)

**Coverage scoring:**
- 0: No memory files
- 1: MEMORY.md exists and is read
- 2: MEMORY.md + daily logs
- 3: Daily logs + project tracking + consistent update habit
- 4: Full: multi-tier memory + regular curation + audit protocol

---

### L4 — Interface

**What it is:** How you interact with your agent — the channels and UIs you use to send instructions and receive outputs.

**Key question:** Can you reach your agent from anywhere, and does it deliver outputs where you actually see them?

**What most clients have:**
- CLI only
- No mobile access
- Checking back manually to see if tasks are done

**What they're missing:**
- Telegram integration (mobile-first, persistent, organized)
- Forum topics for routing different work types
- Push notifications for completed tasks
- Multiple input methods (voice, quick commands, file drops)

**Primitives at this layer:**
- Telegram bot (polling or webhook)
- Telegram Forum Topics (organized workspace)
- CLI (openclaw chat)
- Web interface (if using gateway)
- Voice input (if configured)

**Coverage scoring:**
- 0: CLI only, no mobile
- 1: Telegram DM connected
- 2: Telegram Forum Topics set up with organization
- 3: Topics + cron delivery to specific topics
- 4: Full: multi-channel + organized + mobile + push notifications

---

### L5 — Automation

**What it is:** Work that happens on a schedule without you initiating it.

**Key question:** Is your operation running 24/7 or only when you actively start a session?

**What most clients have:**
- No automation (everything manual)
- At most: one morning cron

**What they're missing:**
- Morning briefing cron
- System health monitoring crons
- Content delivery crons
- Research crons (weekly briefings)
- Alert crons (notify when something breaks)

**Primitives at this layer:**
- openclaw cron system
- LaunchAgents/systemd for system-level scheduling
- n8n for visual workflow automation
- Webhook receivers for event-triggered automation
- Gateway watchdog (restart on failure)

**Coverage scoring:**
- 0: No automation
- 1: One working cron (typically morning briefing)
- 2: Multiple crons covering key functions
- 3: Crons + monitoring + alerts
- 4: Full: crons + monitoring + n8n workflows + event triggers

---

### L6 — Integration

**What it is:** Connections between your agent and external services.

**Key question:** Can your agent act on the world — posting, reading, writing — without manual handoffs?

**What most clients have:**
- Web search capability
- Maybe GitHub connection
- No write access to external services

**What they're missing:**
- Email integration (reading + sending)
- GitHub operations (PRs, issues, code review)
- Content platform connections (Substack, X, Bluesky)
- Calendar integration
- Communication tool connections (Slack, Discord)

**Primitives at this layer:**
- himalaya (email)
- gh CLI (GitHub)
- xurl (X/Twitter)
- Substack API (newsletter)
- Bluesky AT Protocol
- Facebook Graph API
- Google Calendar API
- Slack/Discord bots

**Coverage scoring:**
- 0: No integrations
- 1: 1–2 integrations (typically GitHub + web search)
- 2: Primary workflow integrations (3–5)
- 3: Full content distribution pipeline + email
- 4: Complete stack: all relevant platforms + bidirectional read/write

---

### L7 — Output

**What it is:** What the system actually produces — the tangible deliverables.

**Key question:** Is the operation generating consistent, quality output, or is it just answering questions?

**What most clients have:**
- Reactive responses to questions
- No consistent content output
- No distribution pipeline

**What they're missing:**
- Regular content production (articles, posts, threads)
- Automated distribution
- Product creation (guides, tools, templates)
- Revenue-generating outputs

**Primitives at this layer:**
- Content pipeline (draft → review → publish)
- Distribution workflow (one platform → all platforms)
- Product creation templates
- Report generation
- Digest creation

**Coverage scoring:**
- 0: Only reactive, no production
- 1: Occasional output on request
- 2: Regular content output with manual publishing
- 3: Regular content + automated distribution
- 4: Full pipeline: production + distribution + monetization path

---

### L8 — Commons

**What it is:** The shared knowledge base and resources that make the system smarter over time.

**Key question:** Is the operation getting smarter? Are you building intellectual capital or starting from scratch each time?

**What most clients have:**
- Nothing — every task starts cold
- Maybe some notes in a personal knowledge management tool

**What they're missing:**
- Research archive (saved briefings, summaries)
- Prompt templates and patterns
- Skill library (custom skills for recurring tasks)
- Documented workflows
- Reference documents (SOPs, checklists)

**Primitives at this layer:**
- Obsidian vault
- SKILL.md libraries
- Workflow templates
- Research archive (memory files)
- Prompt library
- SOPs and process docs

**Coverage scoring:**
- 0: No knowledge base
- 1: Some memory files
- 2: Memory + documented workflows
- 3: Memory + workflows + custom skills
- 4: Full: rich knowledge base + skills + templates + growing library

---

## Gap Analysis Template

Copy and fill in for each client (or yourself):

```markdown
# Stack Assessment — [Name/Organization]
Date: [Date]
Assessed by: [Your name]

## Scores

| Layer | Score (0-4) | Current State | Missing |
|-------|-------------|---------------|---------|
| L0 Substrate | | | |
| L1 Model Access | | | |
| L2 Orchestration | | | |
| L3 Memory | | | |
| L4 Interface | | | |
| L5 Automation | | | |
| L6 Integration | | | |
| L7 Output | | | |
| L8 Commons | | | |
| **Total** | **/36** | | |

## Critical Gaps (score 0–1)
1. [Layer]: [What's missing]
2. [Layer]: [What's missing]

## High-Value Additions (score 2, would move to 3–4)
1. [Layer]: [Specific addition]
2. [Layer]: [Specific addition]

## Priority Ranking
1. [Highest impact gap to close]
2. [Second priority]
3. [Third priority]
```

---

## Coverage Scoring Rubric

| Total Score | Assessment | What It Means |
|-------------|-----------|---------------|
| 0–9 | Embryonic | Just getting started; major infrastructure gaps |
| 10–18 | Developing | Some foundation but significant missing layers |
| 19–27 | Operational | Core system working; gaps in optimization layers |
| 28–33 | Advanced | Well-integrated system with minor gaps |
| 34–36 | Full Coverage | Comprehensive, all layers operational |

---

## Priority Ranking Guide

When you have gaps across multiple layers, fix in this order:

1. **L3 Memory first** — Without memory, everything else is more expensive (you recreate context constantly)
2. **L4 Interface second** — If you can't easily interact with the system, you won't use it
3. **L5 Automation third** — Automation multiplies everything; without it, it's just a smarter ChatGPT
4. **L0 Substrate fourth** — If reliability is failing (crashing, no restarts), fix the foundation
5. **L2 Orchestration fifth** — Once the basics work, add structure and delegation
6. **L1 Model Access sixth** — Cost optimization matters once you're actually using the system regularly
7. **L6 Integration seventh** — Connections to external services unlock real leverage
8. **L7 Output eighth** — Build the production pipeline when you have stable infrastructure
9. **L8 Commons last** — Knowledge compounds over time; start building once other layers are stable

---

## Remediation Roadmap Template

```markdown
# Remediation Roadmap — [Name]
Based on assessment: [Date]
Target score: [X]/36

## Sprint 1 (Week 1–2): Critical Foundation
Goal: Address all L3 and L4 gaps

- [ ] Create MEMORY.md with current context
- [ ] Set up daily memory log structure
- [ ] Connect Telegram interface
- [ ] Test basic interaction end-to-end

Success: Agent responds on Telegram with awareness of memory context.

## Sprint 2 (Month 1): Automation Layer
Goal: First working automations

- [ ] Morning briefing cron
- [ ] System health monitoring cron
- [ ] [Client-specific automation #1]

Success: System runs overnight and delivers useful output.

## Sprint 3 (Month 2): Integration + Output
Goal: External connections and consistent output

- [ ] [Priority integration: GitHub / Email / Content platform]
- [ ] Weekly content production rhythm
- [ ] Distribution pipeline for at least one platform

Success: System produces and distributes content with minimal manual intervention.

## Sprint 4 (Month 3): Optimization
Goal: Close remaining gaps, compound the gains

- [ ] Custom skill development for recurring tasks
- [ ] Knowledge base / Commons buildout
- [ ] Cost optimization (model routing)
- [ ] Security audit (full 10-point checklist)

Success: [Client-specific metric — time saved, content produced, revenue generated]
```

---

## The 104 Primitives Reference

A comprehensive list of tools, files, and patterns that map to each layer:

**L0:** Mac Mini, Raspberry Pi, DigitalOcean VPS, Hetzner, launchctl, systemd, borgbackup, rsync, Time Machine, htop, cron, Tailscale, UFW firewall

**L1:** anthropic, openai, google-generativeai, mistral, ollama, openclaw config, budget alerts, flat-rate plans, API key rotation, model benchmarking

**L2:** AGENTS.md, SOUL.md, sub-agent spawning, BULLETIN.md, escalation rules, failure recovery, Blueprint pattern, role definitions, permission boundaries

**L3:** MEMORY.md, memory/YYYY-MM-DD.md, open-projects.md, FAILURE_MODEL.md, research archive, session summaries, project-specific context files

**L4:** Telegram bot, Forum Topics, CLI (openclaw chat), web gateway, Telegram cron delivery, voice input, file upload interface, webhook input

**L5:** openclaw cron, LaunchAgents, n8n workflows, webhook receivers, event triggers, gateway watchdog, monitoring crons, alert crons, scheduled publishing

**L6:** himalaya (email), gh CLI (GitHub), xurl (Twitter/X), Substack API, Bluesky API, Facebook Graph API, Google Calendar, Slack bot, Discord bot, Notion API, Airtable API

**L7:** newsletter drafts, social posts, code reviews, research briefings, weekly digests, product documentation, automated reports, content calendars, PR generation

**L8:** Obsidian vault, SKILL.md library, prompt templates, workflow SOPs, research notes, checklists, reference documents, decision logs, retrospectives

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
