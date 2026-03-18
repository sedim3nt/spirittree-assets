# AI Cost Optimizer
### Cut Your AI Agent Costs by 80-97% Without Losing Quality

*By AgentOrchard — from an operation that runs 6 AI agents 24/7 on a single Mac Mini*

---

## Who This Is For

You're running AI agents (OpenClaw, LangChain, AutoGPT, CrewAI, or raw API calls) and your API bill is climbing. Maybe it's $50/day, maybe $200. You know you're overspending but you don't know where.

This guide gives you the exact playbook we used to cut our operation from ~$120/day to ~$8/day while maintaining the same output quality. Every config shown is from our production system.

---

## Part 1: The Token Audit (Find Where You're Bleeding)

Before optimizing anything, you need to know where the money goes.

### Step 1: Calculate Your Current Cost Per Task

Most people have no idea what individual tasks cost. Here's how to find out:

| Task Type | Typical Token Usage | Cost at GPT-4/Opus | Cost at Sonnet | Cost at Haiku |
|-----------|-------------------|-------------------|----------------|---------------|
| Simple Q&A | 500-1,000 tokens | $0.03-0.06 | $0.005-0.01 | $0.0005-0.001 |
| Research synthesis | 3,000-8,000 tokens | $0.18-0.48 | $0.025-0.065 | $0.003-0.008 |
| Code generation | 2,000-10,000 tokens | $0.12-0.60 | $0.016-0.08 | $0.002-0.01 |
| Long-form writing | 5,000-15,000 tokens | $0.30-0.90 | $0.04-0.12 | $0.005-0.015 |
| System heartbeat | 200-500 tokens | $0.01-0.03 | $0.002-0.004 | $0.0002-0.0005 |
| Context loading | 10,000-50,000 tokens | $0.15-0.75 | $0.03-0.15 | $0.0025-0.0125 |

**The insight:** Most operations run everything on their most expensive model. A heartbeat check that costs $0.03 on Opus costs $0.0003 on Haiku — 100x difference for identical output.

### Step 2: Audit Your Token Sources

Run this mental checklist on your setup:

**Context bloat (usually 40-60% of your bill):**
- [ ] How large is your system prompt? (>2,000 tokens = problem)
- [ ] Are you loading entire files into context when you only need a section?
- [ ] Do your agents carry conversation history beyond what's needed?
- [ ] Are you pre-loading workspace files that rarely change?

**Redundant calls (usually 15-25% of your bill):**
- [ ] Do heartbeats/monitoring run on your primary model?
- [ ] Are cron jobs triggering full agent sessions for simple checks?
- [ ] Do your agents re-read files they already processed?

**Wrong model for the task (usually 15-25% of your bill):**
- [ ] Are you using Opus/GPT-4 for tasks that Haiku/GPT-4-mini handles perfectly?
- [ ] Do all agents run on the same model regardless of task complexity?

### Step 3: Your Cost Baseline

Fill this in before making changes:

```
Current daily API cost: $________
Number of agents/sessions: ________
Primary model: ________
Daily token usage (estimate): ________
Heartbeat/monitoring frequency: ________
```

---

## Part 2: The Model Routing Matrix (The Biggest Win)

This is where 60-80% of your savings come from.

### The Principle
**Match model capability to task complexity.** You don't use a chainsaw to cut butter.

### Our Production Routing

| Task | Model | Why | Monthly Cost |
|------|-------|-----|-------------|
| Strategic planning, PRD writing, complex synthesis | Opus (Claude 4.6) | Needs deep reasoning, creativity | ~$90 |
| Code generation, research, content writing | Sonnet (Claude 4.6) | Workhorse — 90% of Opus quality at 15% of the cost | ~$45 |
| Heartbeats, monitoring, simple routing, email checks | Haiku (Claude 4.5) | Perfect for yes/no decisions, status checks | ~$3 |
| Image generation | gpt-image-1 | Only OpenAI option that matters | ~$5 |
| Embeddings / memory search | text-embedding-3-small | Cheapest embedding that's still good | ~$0.10 |

### The Fallback Chain (Critical)

Don't just pick one model. Set up a chain so you never hit a wall:

```json
{
  "model": {
    "primary": "anthropic/claude-opus-4-6",
    "fallbacks": [
      "anthropic/claude-sonnet-4-6",
      "anthropic/claude-haiku-4-5"
    ]
  }
}
```

**What this does:** When Opus rate-limits (and it will), your system automatically falls back to Sonnet, then Haiku. Zero downtime. Your agents never stop.

**Real scenario from our operation:** On March 9, 2026, Sonnet rate-limited during a heavy research session. Without the fallback chain, everything would have stopped. Instead, Haiku picked up the work seamlessly.

### Agent-Level Model Assignment

Don't give every agent the same model:

| Agent Role | Recommended Model | Reasoning |
|------------|------------------|-----------|
| Orchestrator / CEO | Opus | Needs best judgment for delegation decisions |
| Researcher | Sonnet | Synthesis quality is important, but not Opus-level |
| Coder | Sonnet | Code quality is measurable; Sonnet passes the same tests |
| Content Writer | Sonnet | Voice/quality matters but Sonnet handles it |
| Ops / Monitoring | Haiku | Checking crons, reading email, heartbeats — Haiku is perfect |
| Image Generation | N/A (API call) | Script-based, no LLM reasoning needed |

```json
{
  "agents": {
    "list": [
      { "id": "main", "model": "anthropic/claude-opus-4-6" },
      { "id": "research", "model": "anthropic/claude-sonnet-4-6" },
      { "id": "coding", "model": "anthropic/claude-sonnet-4-6" },
      { "id": "content", "model": "anthropic/claude-sonnet-4-6" },
      { "id": "ops", "model": "anthropic/claude-sonnet-4-6" },
      { "id": "artist", "model": "anthropic/claude-haiku-4-5" }
    ]
  }
}
```

---

## Part 3: Context Pruning (Stop Paying to Read the Same Files)

### The Problem
Every token your agent reads costs money. If your system prompt is 5,000 tokens and your agent wakes up 50 times a day, that's 250,000 tokens just on system prompts — about $3.75/day on Opus for doing literally nothing.

### Fix 1: Lean System Prompts
**Before (bloated):**
```
You are an AI assistant created by [company]. Your role is to help with 
various tasks including but not limited to research, writing, coding, 
analysis, and general assistance. You should be helpful, harmless, and 
honest. You have access to the following tools: [list of 20 tools with 
full descriptions]. Your knowledge cutoff is [date]. Please follow these 
guidelines: [500 words of guidelines]...
```

**After (lean):**
```
You are Rowan, research agent for AgentOrchard. 
Role: web search, source analysis, knowledge extraction.
Read AGENTS.md for full context. Only load files when needed for the current task.
```

**Savings:** 4,000 tokens × 50 sessions/day × $0.015/1K = **$3/day saved**

### Fix 2: Lazy File Loading
Don't load your entire knowledge base at session start. Load files only when the task requires them.

**Bad:** Load MEMORY.md (5,000 tokens) + USER.md (2,000 tokens) + SOUL.md (1,500 tokens) + today's log (3,000 tokens) every session

**Good:** Load a 200-token summary, then `memory_search` for specific facts when needed

**Our approach:** We use vector memory search (SQLite + embeddings) so agents query semantically instead of loading entire files. Cost: ~$0.10/month for embeddings vs. ~$90/month loading files into context.

### Fix 3: Context TTL (Time-to-Live)
Set aggressive context pruning so old conversation turns get dropped:

```json
{
  "contextPruning": {
    "mode": "cache-ttl",
    "ttl": "1h"
  }
}
```

Conversation turns older than 1 hour get pruned. Your agent stays focused on the current task instead of carrying dead context.

### Fix 4: Compaction Strategy
When context gets too large, compact it into a summary instead of keeping the full history:

```json
{
  "compaction": {
    "mode": "safeguard",
    "memoryFlush": {
      "enabled": true,
      "softThresholdTokens": 4000
    }
  }
}
```

This triggers a memory flush before compaction — the agent writes important facts to disk, then the bloated context gets replaced with a lean summary.

---

## Part 4: Cron Optimization (The Silent Budget Killer)

### The Problem
Every cron job triggers a full agent session. If you have 7 crons running on Opus, that's 7 sessions/day × ~2,000 tokens each × $0.015/1K = $0.21/day minimum. Sounds small, but over a month that's $6.30 of pure waste if those tasks could run on Haiku ($0.63/month instead).

### Our Cron Budget

| Cron Job | Schedule | Model | Monthly Cost |
|----------|----------|-------|-------------|
| Heartbeat | Every 8h (3×/day) | Haiku | $0.03 |
| Email check | Every 8h (3×/day) | Sonnet (needs judgment) | $0.15 |
| Daily log compaction | Daily at midnight | Haiku | $0.01 |
| Self-improvement | Daily at 1am | Haiku | $0.01 |
| Research briefing | Daily at 6am | Sonnet | $0.05 |
| Social media posting | Daily at 7am | Sonnet | $0.10 |
| Social monitoring | Daily at 11am | Sonnet | $0.05 |

**Total cron cost: ~$0.40/day = $12/month**

### The Rule
Any cron job that's a **status check, health ping, or simple routing decision** should run on Haiku. Only crons that require **judgment, writing, or synthesis** deserve Sonnet.

---

## Part 5: The Before/After Calculator

### Calculate Your Savings

**Current Setup (estimate):**
```
Primary model cost per 1K tokens (input): $________
Daily input tokens: ________
Daily output tokens: ________
Number of cron jobs: ________
Number of agents: ________

Current daily cost = (input_tokens × input_price + output_tokens × output_price) / 1000
Current daily cost: $________
```

**After Optimization:**
```
Opus tasks (% of total): ________% 
Sonnet tasks (% of total): ________%
Haiku tasks (% of total): ________%

Optimized daily cost = 
  (opus_tokens × $0.015 + sonnet_tokens × $0.003 + haiku_tokens × $0.00025) / 1000
  + cron costs (from table above)

Optimized daily cost: $________
Savings: $________ / day = $________ / month
```

### Our Real Numbers

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Daily cost | ~$120 | ~$8 | -93% |
| Primary model | Opus for everything | Opus 10%, Sonnet 70%, Haiku 20% | — |
| Context size (avg) | 45,000 tokens | 8,000 tokens | -82% |
| Cron model | Opus | Haiku | -99% per cron |
| Heartbeat cost/month | $2.70 | $0.03 | -99% |
| Quality impact | Baseline | No measurable difference | None |

---

## Part 6: Advanced Moves

### Move 1: Local Models for Zero-Cost Ops
Install Ollama and run small models locally for heartbeats and simple checks:
- Llama 3.2 3B: Free, runs on any laptop, perfect for health checks
- Qwen 2.5 7B: Free, good for simple content review

**When to use local:** Monitoring, health checks, simple classification, email triage
**When NOT to use local:** Research synthesis, creative writing, code generation, anything customer-facing

### Move 2: Embedding Provider Selection
Memory search embeddings are a hidden cost:
- OpenAI `text-embedding-3-small`: $0.02/1M tokens (cheapest good option)
- Gemini `embedding-001`: Free tier available, but rate-limited
- Local via Ollama: $0 but slower

We switched from Gemini (hit quota) to OpenAI embeddings. Cost: negligible.

### Move 3: Batch vs. Real-Time
If your agents process large batches (e.g., 50 emails, 200 data entries), use the OpenAI Batch API for 50% cost reduction on non-time-sensitive work.

---

## Quick Reference Card

**The 5 Rules:**
1. **Route by complexity** — Opus for strategy, Sonnet for work, Haiku for ops
2. **Set up fallback chains** — Never hit a wall, always degrade gracefully
3. **Prune context aggressively** — Every token costs money
4. **Run crons on the cheapest model** — Heartbeats don't need Opus
5. **Measure before and after** — You can't optimize what you don't measure

**Copy-paste configs included above. Adjust model names for your provider (OpenAI, Anthropic, Google).**

---

© 2026 AgentOrchard · agentorchard.dev
