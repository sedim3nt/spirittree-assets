# OpenClaw Model Guide — Choose the Right AI for Every Task

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 10 minutes (reading time)

Stop burning money on the wrong models. This guide gives you the full comparison matrix, task routing rules, and a monthly cost calculator — so you know exactly which model to use for every job and what it'll cost.

---

## What's In This Package

- Full model comparison matrix (Claude Opus, Sonnet, Haiku, GPT-5.4, Gemini 2.5 Pro)
- Pricing breakdown: API per-token vs. flat subscription rate analysis
- Task routing rules (which tasks belong to which tier)
- Monthly cost calculator template
- Real production examples from a live multi-agent setup
- Cost optimization playbook (common waste patterns + fixes)
- Model selection decision tree
- How to switch models per agent role in OpenClaw config

---

# OpenClaw Model Guide
## Choosing the Right AI Model for Every Task

Picking the wrong model costs money and produces worse results. This guide gives you the full comparison matrix, task routing rules, and a cost calculator you can use for budget planning.

---

## The Model Comparison Matrix

| Model | Provider | API Input | API Output | Flat Rate | Context Window | Best For |
|-------|----------|-----------|------------|-----------|----------------|----------|
| Claude Opus 4.6 | Anthropic | $15/M tokens | $75/M tokens | $200/mo (Max) | 200K | Strategy, orchestration, complex synthesis |
| Claude Sonnet 4.6 | Anthropic | $3/M tokens | $15/M tokens | $100/mo | 200K | Coding, content, research, daily ops |
| Claude Haiku 3.5 | Anthropic | $1/M tokens | $5/M tokens | $20/mo | 200K | Monitoring, crons, quick lookups |
| GPT-5.4 | OpenAI | $2.50/M tokens | $10/M tokens | $200/mo (Pro) | 128K | OpenAI compatibility, specific tool use |
| Gemini 2.5 Pro | Google | Free via CLI | Free via CLI | Free | 1M+ | Long-document research, bulk processing |

*Prices as of March 2026. Check provider pricing pages for current rates.*

---

## Understanding the Pricing Model

### API Pricing: Pay Per Token

Token pricing means you pay exactly for what you use. One token ≈ 0.75 words. A typical message exchange (your input + agent response) runs 500–2,000 tokens.

**Quick math:**
- 1,000 tokens ≈ 750 words ≈ one typical conversation exchange
- Claude Sonnet 4.6: $3 per 1M input tokens = $0.003 per 1,000 input tokens
- A medium-complexity cron task (500 input + 500 output): ~$0.01 with Sonnet

**Monthly API cost estimator:**

| Daily Pattern | Monthly Token Est. | Sonnet Cost | Opus Cost | Haiku Cost |
|---------------|-------------------|-------------|-----------|------------|
| 10 light tasks | ~3M tokens | ~$18 | ~$90 | ~$6 |
| 30 medium tasks | ~15M tokens | ~$54 | ~$270 | ~$18 |
| 50 heavy tasks + crons | ~50M tokens | ~$180 | ~$900 | ~$60 |

### Flat-Rate Plans: When They Win

Flat-rate plans (Claude Max, GPT Pro) make sense when you're a consistent heavy user who wants predictability. The break-even points:

- **Claude Max ($200/mo):** Breaks even vs. API at ~50M Sonnet-equivalent tokens or ~13M Opus tokens per month
- **Claude at $100/mo:** Breaks even vs. API at ~25M Sonnet tokens per month
- **Haiku at $20/mo:** Almost always cheaper than API unless you're barely using it

**Rule of thumb:** If you're running active multi-agent operations with crons, content pipelines, and research workflows, flat rate is probably cheaper. If you're a light user, API is better.

---

## Task Routing Guide

The goal is to use the cheapest model that produces acceptable quality for each task type. Over-specifying (Opus for everything) burns money. Under-specifying (Haiku for complex reasoning) produces bad results.

### Use Claude Opus 4.6 For:

**Strategic planning and orchestration**
- Multi-agent workflow design
- Architecture decisions ("how should we structure this system?")
- Complex trade-off analysis
- Synthesizing research across many sources
- Writing AGENTS.md, SOUL.md, and other high-stakes configuration

**Why Opus here:**
These tasks require holding many competing considerations in context simultaneously and producing nuanced judgments. Sonnet handles many of them, but Opus produces noticeably better output on genuinely complex synthesis.

**Example production use:**
```
Task: Design a client delivery system for AI consulting
Model: Opus
Why: Requires understanding multiple stakeholder perspectives, 
     anticipating edge cases, and producing a coherent system 
     that works across diverse client situations.
```

---

### Use Claude Sonnet 4.6 For:

**Coding tasks**
- Writing, reviewing, and debugging code
- Generating tests
- Refactoring
- API integrations

**Content production**
- Writing articles, guides, social posts
- Editing and improving drafts
- Research synthesis (moderate volume)
- Email and communication drafting

**Daily operations**
- Answering questions in Telegram
- Research tasks under ~50 pages of source material
- Generating reports from structured data
- Most interactive conversation

**Why Sonnet here:**
Sonnet hits the sweet spot of quality and cost for the majority of real work. It's 5× cheaper than Opus with 80–90% of the output quality for most tasks. For coding and content, Sonnet is the default.

**Example production use:**
```
Task: Write a 2,000-word guide on n8n automation patterns
Model: Sonnet
Why: Content production is Sonnet's wheelhouse. 
     The quality-to-cost ratio is optimal here.
```

---

### Use Claude Haiku 3.5 For:

**Monitoring and heartbeats**
- System health checks
- "Is everything running?" crons
- Alert generation from structured data
- Log summarization

**Quick lookups and classification**
- "What category does this belong to?"
- Simple extraction tasks
- Single-turn Q&A with short answers

**High-volume, low-stakes tasks**
- Processing many items in a loop
- Batch classification
- Simple reformatting tasks

**Why Haiku here:**
Haiku is fast and cheap. For tasks that run on a schedule or process high volumes, the cost difference matters. A cron that runs 50 times a day costs 15× less with Haiku than Opus.

**Example production use:**
```
Task: Morning health check cron — check if n8n, gateway, and Telegram 
      bot are running; send status to Telegram
Model: Haiku
Why: Simple structured task, runs daily, pure Opus overkill 
     at 15× the cost.
```

---

### Use GPT-5.4 For:

**OpenAI-specific tool use**
- DALL-E image generation integration
- Whisper transcription pipelines
- Cases where OpenAI's function calling format is required

**Compatibility requirements**
- When working with systems built on the OpenAI API format
- When a client's existing workflow uses GPT

**When Anthropic is unavailable**
- API outages (rare but happens)
- Rate limit fallback

**Pricing context:**
GPT-5.4 at $2.50/$10 per million tokens is price-competitive with Sonnet. Quality is roughly comparable for most tasks. It's not the default choice in an Anthropic-first setup, but it's a solid second choice.

---

### Use Gemini 2.5 Pro For:

**Long-document research**
- Processing entire books or research papers
- Analyzing large codebases
- Cross-referencing many documents simultaneously
- Anything that needs >200K context window

**Bulk processing**
- Analyzing hundreds of items where per-item cost matters
- YouTube transcript extraction and summarization
- Large-scale data extraction tasks

**Why Gemini here:**
The Gemini CLI is free for personal use, and Gemini 2.5 Pro has a ~1M token context window. For tasks involving large amounts of text, this is unbeatable. The output quality is competitive for research tasks.

**Limitations:**
- Less capable than Opus/Sonnet on nuanced reasoning
- Output quality varies more than Claude on complex tasks
- Free tier has rate limits that can slow batch operations

---

## Model-Per-Task Routing Config

Add this pattern to your AGENTS.md to give agents routing guidance:

```markdown
## Model Routing

| Task Type | Preferred Model | Fallback |
|-----------|----------------|---------|
| Crons and monitoring | claude-haiku-3-5 | — |
| Coding tasks | claude-sonnet-4-6 | claude-opus-4-6 |
| Content writing | claude-sonnet-4-6 | claude-haiku-3-5 |
| Research synthesis | claude-sonnet-4-6 | gemini (for large docs) |
| Strategy and orchestration | claude-opus-4-6 | claude-sonnet-4-6 |
| Image generation | gpt-5.4 (DALL-E) | — |
| Bulk processing | gemini-2.5-pro | claude-haiku-3-5 |

Default model: claude-sonnet-4-6
Override explicitly when task warrants it.
```

---

## The Cost Calculator Template

Use this spreadsheet-style template to estimate your monthly costs before committing to a plan:

```
DAILY TASK INVENTORY
====================
Task                          | Model  | ~Tokens/Run | Runs/Day | Daily Cost
------------------------------|--------|-------------|----------|----------
Morning briefing cron         | Haiku  | 800         | 1        | $0.004
Research task (medium)        | Sonnet | 5,000       | 3        | $0.054
Code review                   | Sonnet | 8,000       | 2        | $0.072
Content draft                 | Sonnet | 6,000       | 2        | $0.054
Strategy session              | Opus   | 15,000      | 0.5      | $0.675
Monitoring crons (5×)         | Haiku  | 300         | 5        | $0.008
Interactive chat              | Sonnet | 2,000       | 10       | $0.060
                              |        |             |          |
DAILY TOTAL                   |        |             |          | $0.927
MONTHLY ESTIMATE (×30)        |        |             |          | $27.81
```

Fill in your actual tasks. If the monthly total approaches $100, consider flat-rate plans.

---

## Real Production Examples

**Example 1: Solo Content Operation**
- 3 articles/week written with Sonnet (~$0.30/article)
- Daily social posts drafted with Haiku (~$0.02/post)
- Weekly strategy review with Opus (~$0.90/session)
- n8n workflow monitoring with Haiku (5 crons/day, ~$0.04/day)
- **Total: ~$25–40/month via API**
- **Verdict:** API beats flat rate at this volume

**Example 2: Multi-Agent Research Operation**
- Daily deep research tasks with Sonnet (5 tasks × 10K tokens = ~$0.75/day)
- Competitive intel with Gemini (long documents, free)
- Content pipeline with Sonnet (10 pieces/week, ~$0.40/week)
- Orchestration with Opus (1 session/day, ~$0.68/day)
- **Total: ~$50–80/month via API**
- **Verdict:** Flat rate ($100/mo) starts making sense here

**Example 3: Client Consulting + Content**
- Heavy Opus use for client strategy work (~$2/session × 15 sessions = $30)
- High-volume Sonnet content production (~$40/month)
- Haiku for all monitoring and crons (~$5/month)
- **Total: ~$75/month via API**
- **Verdict:** Claude Max ($200/mo) is too expensive; API or $100/mo plan is right

---

## Model Capability Comparison: Common Tasks

| Task | Haiku | Sonnet | Opus |
|------|-------|--------|------|
| Write a tweet | ✅ Good | ✅ Great | ✅ Overkill |
| Debug a 200-line function | ⚠️ OK | ✅ Good | ✅ Great |
| Design a multi-agent system | ❌ Poor | ⚠️ OK | ✅ Great |
| Summarize a research paper | ⚠️ OK | ✅ Good | ✅ Great |
| Write a Bash cron task | ✅ Good | ✅ Great | ✅ Overkill |
| Complex strategic planning | ❌ Poor | ⚠️ OK | ✅ Great |
| Extract data from JSON | ✅ Great | ✅ Great | ✅ Overkill |
| Long-form content (2K+ words) | ⚠️ OK | ✅ Good | ✅ Great |
| Multi-step reasoning chain | ❌ Poor | ⚠️ OK | ✅ Great |

---

## Quick Decision Tree

```
Is this a cron or scheduled monitoring task?
└── YES → Haiku

Is this a task involving documents >200K tokens?
└── YES → Gemini

Is this coding, content writing, or research?
└── YES → Sonnet (unless complexity is extreme)

Does this require strategic planning, complex synthesis, 
or orchestrating multiple agents?
└── YES → Opus

Is this an OpenAI-specific integration?
└── YES → GPT-5.4

Default: Sonnet
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
