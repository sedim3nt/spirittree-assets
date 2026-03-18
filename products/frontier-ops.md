# Frontier Operations — Stay Current at the Edge of Human-Agent Work

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 30 minutes (reading + templates)

The boundary between what humans do and what agents do shifts every month. Frontier Ops is the practice of tracking that boundary and adjusting your operations before the rest of the market catches up. This is compounding leverage.

---

## What's In This Package

- Boundary Sensing: monthly capability checklist (what changed this month)
- Seam Design: human/agent handoff template for clean transitions
- Failure Modeling: FAILURE_MODEL.md template with 6 failure categories
- Capability Forecasting: quarterly assessment worksheet
- Leverage Calibration: delegation decision matrix (what to delegate NOW)
- Monthly review protocol with time estimates
- Real examples: 3+ months of production boundary shifts documented
- The "good enough" calibration framework (when AI quality crosses the delegation threshold)
- Tools for tracking frontier developments (RSS, communities, benchmarks)
- Case studies: tasks that moved from human to agent over 6 months

---

# Frontier Operations
## Staying Current at the Edge of Human-Agent Work

The boundary between what humans do and what agents do shifts every month. Frontier Ops is the practice of actively tracking that boundary and adjusting your operations accordingly. Miss this and you're leaving leverage on the table. Get it right and you compound faster than anyone who's just reacting.

---

## The Five Skills of Frontier Operations

1. **Boundary Sensing** — tracking what's changed
2. **Seam Design** — structuring handoffs between human and agent work
3. **Failure Modeling** — knowing where agents fail
4. **Capability Forecasting** — anticipating what comes next
5. **Leverage Calibration** — delegating the right things at the right time

---

## Skill 1: Boundary Sensing

Boundary Sensing is the monthly practice of auditing what agents can now do that they couldn't last month.

### The Monthly Boundary Sensing Checklist

Run this at the start of each month. Document findings in `memory/boundary-sensing-[YYYY-MM].md`.

**Model capability changes:**
- [ ] Any new models released? Check: anthropic.com/news, openai.com/blog, deepmind.google
- [ ] Any significant context window increases?
- [ ] Any new tool/function calling capabilities?
- [ ] Any new modalities (image, audio, video)?
- [ ] Have benchmarks shifted meaningfully on coding/reasoning/analysis tasks?

**OpenClaw ecosystem:**
- [ ] New skills released? Check the skill marketplace.
- [ ] OpenClaw version updates? `openclaw --version` vs. latest on GitHub
- [ ] New integrations available?
- [ ] Any deprecations or breaking changes?

**Adjacent tooling:**
- [ ] Any new agentic frameworks worth evaluating? (CrewAI, AutoGen, etc.)
- [ ] Any new CLI tools that would improve your integrations?
- [ ] Any platforms that added official API support?
- [ ] Any platforms that broke API support?

**Your own operation:**
- [ ] What did you hand-hold agents through last month that they should be able to do alone?
- [ ] What agent-generated output did you have to rewrite vs. ship as-is?
- [ ] Where did agents surprise you (positive or negative)?

**Template:**
```markdown
# Boundary Sensing — [Month YYYY]

## What Changed
- [Model/tool]: [What changed] — Impact: [high/med/low]

## New Capabilities to Test
- [ ] [Capability 1]: Test by [specific task]
- [ ] [Capability 2]: Test by [specific task]

## Retired Manual Tasks
- [Task that I used to do manually, now delegated]: [Date delegated]

## Still Manual (reviewing)
- [Task still manual]: [Why / when might this change]

## Next Month's Watch List
- [What to track]
```

### Real Examples from Production (3+ months of operation)

**Month 1 finding:** Claude Sonnet could draft articles but needed heavy editing. Strategy: have it draft, set editing crons with specific critique prompts.

**Month 2 finding:** Cron tasks were writing to wrong topics when context was ambiguous. Seam fix: added explicit topic IDs to every cron definition.

**Month 3 finding:** Agent could now reliably generate and execute multi-step research workflows without hand-holding. Retired: manual research briefing sessions. Delegated: weekly competitive intel cron.

---

## Skill 2: Seam Design

A seam is the point where work passes between human and agent. Badly designed seams create friction, mistakes, and trust erosion. Well-designed seams are invisible.

### The Human/Agent Handoff Templates

**Template 1: Human initiates → Agent executes → Human reviews**

Use for: content production, code generation, research

```markdown
## Seam: Content Draft

HUMAN INITIATES:
- Provides: topic, target audience, key points, word count, tone
- Provides via: Telegram message in Content topic
- Does NOT provide: sources (agent finds), structure (agent decides), draft (agent writes)

AGENT EXECUTES:
- Researches topic using web_search
- Drafts article in requested format
- Self-reviews against quality checklist
- Delivers draft + notes to Content topic

HUMAN REVIEWS:
- Reads draft (not the sources)
- Makes edits in 15 min or less
- Posts "approve" or provides specific feedback

SEAM BREAKS WHEN:
- Agent asks clarifying questions mid-task (insufficient initial spec)
- Draft requires >30 min of editing (agent quality issue — improve prompts)
- Human provides sources and detailed outline (should trust agent more)
```

**Template 2: Agent initiates → Human decides → Agent executes**

Use for: monitoring alerts, threshold-based decisions, anomaly detection

```markdown
## Seam: Alert → Decision → Action

AGENT INITIATES:
- Detects: [condition]
- Delivers: alert with context + recommended action + confidence level
- Delivers via: Telegram Infra topic (urgent) or morning briefing (non-urgent)
- Does NOT: take action without approval

HUMAN DECIDES:
- Reviews alert + context (< 2 min for routine alerts)
- Replies: "yes", "no", or "wait until [time]"
- Does NOT need to know how to fix it, just whether to fix it

AGENT EXECUTES:
- Takes the approved action
- Reports result back to same topic
- Logs decision in daily memory

SEAM BREAKS WHEN:
- Human has to debug the problem before deciding (agent provided insufficient context)
- Human ignores the alert (alert threshold too sensitive)
- Agent acts without approval (boundaries violation)
```

**Template 3: Fully autonomous with exception reporting**

Use for: crons, monitoring, routine content distribution

```markdown
## Seam: Autonomous with Exceptions

AGENT RUNS:
- Scheduled task executes on cron schedule
- Handles expected variations within defined bounds
- Completes task and logs result

EXCEPTIONS ESCALATE:
- Any unexpected error → Telegram alert immediately
- Any output below quality threshold → flag for human review
- Any action outside defined bounds → stop and ask

HUMAN REVIEWS:
- Daily summary in morning briefing
- Exception alerts when triggered
- Monthly audit of autonomous task logs

SEAM BREAKS WHEN:
- Agent escalates too often (bounds too tight — expand them)
- Agent doesn't escalate when it should (bounds too loose — tighten or add checks)
- Human never reads the daily summary (redesign the summary format)
```

---

## Skill 3: Failure Modeling

Failure modeling is the practice of documenting where and how agents fail so you can design around it.

### The FAILURE_MODEL.md Template

Create this file at `~/.openclaw/workspace/memory/FAILURE_MODEL.md`. Update monthly.

```markdown
# FAILURE_MODEL.md
Last updated: [Date]

## Failure Categories

### Category 1: Context Amnesia
**Description:** Agent forgets relevant context that was established earlier in the conversation or in a previous session.
**Frequency:** Common (weekly)
**Trigger conditions:** Long sessions, context window approaching limits, session restarts
**Impact:** Medium — agent repeats work, asks questions already answered
**Mitigation:** Keep MEMORY.md updated, start new sessions with context summary, keep sessions focused
**Status:** Managed (not eliminated)

### Category 2: Tool Hallucination
**Description:** Agent generates confident-sounding code or commands that don't work, without flagging uncertainty.
**Frequency:** Occasional (monthly)
**Trigger conditions:** Unfamiliar APIs, recently changed docs, obscure edge cases
**Impact:** High — wasted time debugging non-existent functionality
**Mitigation:** Always test agent-generated code before using; require agent to note uncertainty
**Status:** Managed via test protocol

### Category 3: Scope Creep
**Description:** Agent expands the scope of a task beyond what was requested.
**Frequency:** Occasional
**Trigger conditions:** Vague specifications, "fix this" without clear boundaries
**Impact:** Medium — extra work produced that requires review, sometimes breaks other things
**Mitigation:** Be specific in task definitions; use "fix X only, don't touch Y"
**Status:** Mostly managed with specification discipline

### Category 4: Overconfident Factual Claims
**Description:** Agent states facts confidently that turn out to be incorrect or outdated.
**Frequency:** Occasional
**Trigger conditions:** Fast-moving domains (AI tooling, crypto, current events), anything after training cutoff
**Impact:** High for decisions made on bad info
**Mitigation:** Web search required for any time-sensitive claim; verify before acting
**Status:** Managed via verification protocol

### Category 5: Paralysis on Ambiguity
**Description:** Agent asks too many clarifying questions rather than making reasonable assumptions.
**Frequency:** Common
**Trigger conditions:** New types of tasks, conflicting instructions in context
**Impact:** Low — just slow; friction in workflow
**Mitigation:** Add "if uncertain, state your assumptions and proceed" to SOUL.md
**Status:** Mostly resolved

### Category 6: Multi-step Task Degradation
**Description:** Quality degrades across a long multi-step task; later steps lose coherence with earlier ones.
**Frequency:** Occasional
**Trigger conditions:** Tasks with >5 steps, complex dependency chains
**Impact:** Medium — late-stage output needs rewriting
**Mitigation:** Break tasks into independently verifiable chunks; use sub-agents for parallel work
**Status:** Managed via task decomposition

## Resolved Failures (moved from above)
| Failure | How Resolved | Date |
|---------|-------------|------|
| [Failure type] | [Resolution] | [Date] |

## Emerging Patterns (watch list)
- [Suspected failure mode being monitored]
```

---

## Skill 4: Capability Forecasting

Quarterly assessment of where model capabilities are likely to be in 3–6 months.

### Quarterly Capability Forecast Worksheet

```markdown
# Capability Forecast — Q[N] [Year]
Assessment date: [Date]

## Current Capability Baseline

| Task Category | Current Quality | Confidence |
|---------------|----------------|------------|
| Code generation (simple) | Excellent | High |
| Code generation (complex) | Good | High |
| Long-form writing | Good | High |
| Multi-step reasoning | Good | Medium |
| Autonomous task completion | Developing | Medium |
| Real-time information | Poor (requires tools) | High |
| Creative work | Good | High |

## Forecast: What's likely to improve in 3–6 months?

Based on: current model release cadence, research papers, announced roadmaps

1. **[Capability]:** Likely improvement from [current] to [forecast level]
   - Evidence: [why you think this]
   - How I'll use it: [specific workflow change]
   - Trigger point: [what specific capability unlocks the use case]

## Current "Not Yet Delegatable" Task List

Tasks I'm still doing manually because agent quality isn't there:

| Task | Current Gap | Estimated Timeline to Delegate |
|------|------------|-------------------------------|
| [Task] | [Specific quality problem] | [Estimate] |

## Bets to Track

| Bet | Metric to Watch | Checking In |
|-----|----------------|------------|
| Fully autonomous codebase review | PR review quality | Monthly |
| Voice-to-memory logging | Accuracy on technical terms | Quarterly |
```

---

## Skill 5: Leverage Calibration

Leverage calibration is the ongoing practice of matching tasks to the right execution path: fully autonomous, supervised automation, or human-first.

### The Delegation Decision Matrix

For any recurring task, score it on two axes:

**Axis 1: Agent quality (1–5)**
1. Agent consistently fails, output unusable
2. Agent needs heavy editing (>50% rewrite)
3. Agent produces acceptable first draft (20–30% editing)
4. Agent produces good output (light editing only)
5. Agent produces excellent output (ship as-is)

**Axis 2: Stakes (1–5)**
1. Low stakes: internal only, reversible, no consequences if wrong
2. Mild stakes: affects your work quality but correctable
3. Medium stakes: client-visible or published, mistakes are embarrassing
4. High stakes: financial, legal, or reputation impact
5. Critical stakes: irreversible, high-consequence errors

**Decision grid:**

| | Stakes 1–2 | Stakes 3 | Stakes 4–5 |
|---|---|---|---|
| **Quality 4–5** | Full autonomy | Async review | Human final check |
| **Quality 3** | Autonomous + audit | Supervised | Human-first |
| **Quality 1–2** | Don't delegate | Don't delegate | Human-first |

**Full autonomy:** Agent runs, logs result, you review periodically
**Async review:** Agent runs, delivers to your Telegram, you confirm before shipping
**Supervised:** You define spec, agent drafts, you review before anything happens
**Human-first:** You do the work, agent assists (research, formatting, etc.)
**Don't delegate:** The quality isn't there yet. Do it yourself or don't do it.

### Monthly Leverage Review

Add to AGENTS.md:
```markdown
## Monthly Leverage Review Protocol
Date: [First Monday of each month]
Trigger: Review delegation matrix for all recurring tasks

1. Which supervised tasks can move to autonomous? (quality improved)
2. Which autonomous tasks should move back to supervised? (quality degraded or stakes changed)
3. Which human-first tasks can move to supervised? (new capability available)
4. Any new tasks that should be delegated for the first time?
```

---

## Monthly Review Protocol

The full Frontier Ops review takes 60–90 minutes. Schedule it monthly.

**Part 1: Boundary Sensing (20 min)**
- Run the monthly checklist above
- Document new capabilities discovered
- Update the watch list for next month

**Part 2: Seam Audit (15 min)**
- Review each defined seam: is it still working as designed?
- Any new friction points?
- Any seams that need redesign?

**Part 3: Failure Model Update (15 min)**
- What new failures occurred this month?
- Any existing failures resolved?
- Update FAILURE_MODEL.md

**Part 4: Leverage Calibration (20 min)**
- Run the monthly leverage review
- Adjust delegation assignments
- Update AGENTS.md with any changes

**Part 5: Capability Forecast Update (15 min)**
- Review forecasts from last quarter — what came true?
- Update forecasts based on new information
- Adjust "not yet delegatable" list

**Deliverable:** Write a 1-page monthly operations review in `memory/ops-review-[YYYY-MM].md`. Share key insights with yourself in the morning briefing.

---

## Putting It All Together: Sample Operations Rhythm

```
Day 1 of each month:
  - Run Boundary Sensing checklist
  - Monthly leverage review
  
Week 1:
  - Seam audit review
  - FAILURE_MODEL.md update

Week 2-4:
  - Normal operations
  - Log unusual agent behaviors in daily memory

End of each quarter:
  - Capability Forecast worksheet
  - 90-minute ops review
  - Update AGENTS.md with any structural changes
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
