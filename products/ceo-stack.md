# CEO Operations Stack

The complete autonomous AI operations configuration used to run SpiritTree — a real zero-employee company generating revenue. Production-tested over 3+ months.

## What's Inside

### SOUL.md — AI Persona & Decision Framework
Your AI agent's personality, communication style, and operating principles. This isn't a generic prompt — it's a production-tested identity system that determines how your agent thinks, communicates, and makes decisions. Includes boundary definitions, autonomy levels, and the SpiritTree operating principles.

### AGENTS.md — Multi-Agent Orchestration Config
The complete agent roster and orchestration playbook. 8 named agent roles (Orchestrator, Research, Coding, Content, Ops, Image Gen, Google Ecosystem, Blockchain) with specific model assignments, responsibilities, and the Blueprint coding pattern (PRE-FETCH → LLM LOOP → DETERMINISTIC → repeat → SUBMIT → SMOKE TEST). Includes specification engineering templates, trade-off matrices, and decision boundaries.

### IDENTITY.md — Brand Identity System
How your AI identifies itself across channels. Codename, avatar, emoji, mantras. Small file, big impact — this is what makes your agent feel like a team member instead of a chatbot.

### HEARTBEAT.md — Monitoring & Health Check System
Automated heartbeat configuration for 24/7 system monitoring. Defines check-in intervals, infrastructure health checks (n8n, dashboard, LaunchAgents), known issue suppression, publish verification (anti-phantom-success), and critical blocker override rules. This is what lets you sleep while your AI keeps working.

### USER.md — Human Context File
Template for encoding your background, skills, preferences, communication style, and goals so your AI agent has persistent context about who it's working for. Includes sections for technical skills, timezone, and operating doctrine.

## How to Use

1. Copy these files into your OpenClaw workspace (`~/.openclaw/workspace/`)
2. Customize each file for your business (the templates have comments showing what to change)
3. Your agent reads these on every session startup
4. Iterate — these files are living documents that improve as your agent learns your preferences

## What Makes This Different

These aren't theoretical templates. This is the exact configuration running a real business:
- 6 Substack articles published autonomously
- Multi-platform content distribution (X, Facebook, Substack)
- Automated email handling and responses
- Infrastructure monitoring with self-healing
- Revenue operations including Stripe integration

Every file has been refined through months of daily production use. You're getting the result of hundreds of iterations, not a v1 draft.

## Requirements

- OpenClaw installed and running
- Claude Code subscription ($100-200/mo)
- 30-60 minutes to customize for your business
