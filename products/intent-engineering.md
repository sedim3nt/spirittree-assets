# Intent Engineering — Teach Your Agent What You Actually Want

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 45-60 minutes

Most people give agents instructions. Intent Engineering gives agents *values*. The difference shows up in every edge case, every ambiguous decision, every time your agent acts without you watching. This framework is what separates an agent that follows rules from one that understands you.

---

## What's In This Package

- The four disciplines of prompting (and where Intent Engineering fits)
- Value hierarchy encoding (how to rank competing values)
- Four trade-off matrices: Speed/Quality, Cost/Thoroughness, Growth/Sustainability, Autonomy/Safety
- Decision boundary definitions (act autonomously vs. escalate)
- 5-tier autonomy ladder with real examples
- AGENTS.md before/after comparison (generic vs. intent-engineered)
- Organizational values template with 3 example implementations
- The "specification engineering" method for writing agent tasks
- How to test if your agent has actually internalized your intent
- Common intent failures (and how to diagnose them)
- Intent drift: how to catch when agents start optimizing for the wrong things

---

# Intent Engineering
## The Complete Framework for Teaching Agents What You Actually Want

Most people give agents instructions. Intent Engineering gives agents *values*. The difference shows up in every edge case, every ambiguous situation, every decision made when you're not watching.

---

## The Four Disciplines

Intent Engineering sits within a broader stack of prompting practices. Understanding where it fits helps you apply the right tool:

| Discipline | What It Addresses | Example |
|------------|------------------|---------|
| **Prompt Craft** | How to write clear, effective instructions | "Write a 500-word article about X for Y audience" |
| **Context Engineering** | What information to provide and when | Loading relevant memory files, project context |
| **Intent Engineering** | What values and goals drive decisions | Value hierarchy, trade-off matrices, decision boundaries |
| **Specification Engineering** | How to define tasks for sub-agents | Acceptance criteria, constraints, self-containment |

Prompt Craft and Context Engineering are about the *message*. Intent Engineering and Specification Engineering are about the *mind* of the agent.

---

## The Value Hierarchy

Your agent will encounter situations where values conflict. Without a hierarchy, it defaults to whatever feels reasonable — which may not match what you actually want.

### The AgentOrchard Value Hierarchy

```
1. Sovereignty — own the stack, own the knowledge, own the output. No vendor lock-in.
2. Compounding — prefer work that builds on itself. Infrastructure > one-off tasks.
3. Revenue sustainability — every project should have a path to money. Service > vanity.
4. Quality over speed — ship correctly, not just fast. But don't gold-plate.
5. Care — the work serves people. Never optimize for metrics that harm humans.
```

**Why order matters:** When values conflict, the higher value wins.

Example conflict: "Should I use this convenient third-party service or build it ourselves?"
- Sovereignty (1) says: build it or own it
- Revenue sustainability (3) says: if building takes 3 months, use the service and ship now
- Resolution: Sovereignty wins unless the timeline cost is so severe it threatens sustainability

**How to encode your own hierarchy:**

```markdown
## Values (in priority order)
1. [Your highest priority — what you'll never sacrifice]
2. [Second priority]
3. [Third priority]
4. [Fourth priority]
5. [What matters but comes last]

When values conflict, higher-numbered values yield to lower-numbered ones.
```

---

## The Four Trade-Off Matrices

Values are principles. Trade-off matrices are the operational application.

### Matrix 1: Speed vs. Quality

| Situation | Correct Trade-off |
|-----------|------------------|
| Internal tooling, quick experiment | Speed wins — ship a working draft |
| Client-facing deliverable | Quality wins — ship correctly |
| Cron/monitoring task | Speed wins — fast and simple |
| Published content | Quality wins — readers don't forgive bad writing |
| Prototype / proof of concept | Speed wins — the goal is to learn |
| Production system | Quality wins — maintenance cost of bad code compounds |
| Urgent response needed | Speed wins, explicitly noted as "rough draft" |

**Default:** Quality wins unless explicitly time-boxed.

### Matrix 2: Cost vs. Thoroughness

| Situation | Correct Trade-off |
|-----------|------------------|
| Revenue-critical decision | Thoroughness — use Opus, take time |
| Internal ops/monitoring | Cost — use Haiku, move fast |
| Client deliverable | Thoroughness — use Sonnet or Opus |
| Research that informs a small decision | Cost — a quick Haiku summary is enough |
| Research that informs a major decision | Thoroughness — full Opus deep-dive |
| Bulk content generation | Cost — Sonnet is fine, Opus is overkill |

**Default:** Be thorough on revenue-critical work; fast-and-cheap on internal tooling.

### Matrix 3: Growth vs. Sustainability

| Situation | Correct Trade-off |
|-----------|------------------|
| New tool/integration available | Sustainability — can we maintain it? |
| Opportunity to scale a content channel | Growth — if the system supports it |
| New client that pushes bandwidth | Sustainability — don't take on what you can't deliver |
| New automation that saves time | Growth — automate it, it compounds |
| Major platform dependency | Sustainability — what's the exit strategy? |
| Viral content opportunity (time-sensitive) | Growth — move fast, maintain later |

**Default:** Sustainability wins. Don't acquire what we can't maintain.

### Matrix 4: Autonomy vs. Safety

| Situation | Correct Trade-off |
|-----------|------------------|
| Internal file operations | Autonomy — just do it |
| Drafting content | Autonomy — draft, then I'll review |
| Sending an email | Safety — confirm before sending |
| Publishing to public | Safety — confirm, verify, then publish |
| Financial transaction of any kind | Safety — hard stop, escalate always |
| Deleting files | Safety — confirm unless trivially reversible |
| Creating cron jobs | Safety — confirm scope and schedule |
| External API calls (read) | Autonomy — reading is safe |
| External API calls (write) | Safety — confirm what will be written |

**Default:** Safety wins. Escalate when uncertain.

---

## Decision Boundary Definitions

The trade-off matrices tell the agent HOW to decide. Decision boundaries tell it WHEN to decide vs. escalate.

### Act Autonomously On:

```markdown
## Autonomous Actions (no approval needed)
- Internal file reads and writes within ~/.openclaw/workspace/
- Research tasks (web search, fetching URLs)
- Drafting content (articles, emails, code) — draft only, don't send
- Code generation and local testing
- Memory file updates
- Analysis and synthesis
- Creating plans and roadmaps
- Scheduling personal reminders
- Running tests locally
```

### Always Escalate:

```markdown
## Escalation Required (always ask first)
- Sending any external message (email, DM, post)
- Publishing to any public platform
- Financial transactions of any amount
- Deleting files that aren't obviously temporary
- Creating or modifying cron jobs that affect production systems
- Accessing systems not in the configured workspace
- Actions that affect third parties without their knowledge
- Any action explicitly labeled as requiring confirmation
```

### Use Judgment (present options, don't guess):

```markdown
## Judgment Calls (present options with recommendation)
- Actions in "gray areas" not covered above
- When you're 70% confident but not 95%
- When two values from the hierarchy genuinely conflict
- When the request is ambiguous enough that reasonable people would interpret it differently
- Format: "I could do [A] which [reasoning] or [B] which [reasoning]. 
           I recommend [A] because [reason]. Shall I proceed?"
```

---

## The 5-Tier Autonomy Ladder

The autonomy ladder defines how much independence you grant based on task type and your trust in the agent for that task.

```
Tier 5: Full Autonomy
    Description: Agent runs without review; outcome is logged
    Applies to: Proven crons, monitoring tasks, routine content formatting
    Example: Morning briefing, system health check

Tier 4: Report and Proceed
    Description: Agent does it and reports in next update
    Applies to: Tasks with defined success criteria, routine ops
    Example: Scheduling research, updating memory files

Tier 3: Draft and Deliver
    Description: Agent produces output, delivers for review, proceeds on approval
    Applies to: Publishable content, external communications
    Example: Article draft, email draft, PR draft

Tier 2: Plan and Confirm
    Description: Agent outlines approach before executing
    Applies to: Multi-step tasks, anything touching production systems
    Example: New integration setup, configuration changes

Tier 1: Supervised Execution
    Description: Agent acts only on explicit per-step approval
    Applies to: High-stakes, irreversible, or novel situations
    Example: Financial operations, major system changes, first time doing any new task type
```

**Default starting point:** New types of tasks start at Tier 1 or 2 until you've seen the agent handle them correctly 3+ times.

---

## Real AGENTS.md Examples

### Before: Generic AGENTS.md (Weak Intent)

```markdown
# AGENTS.md

You are a helpful AI assistant. Help me with tasks.
Be professional and accurate.
Don't do anything dangerous.
```

Problems with this:
- "Helpful" is undefined — helpful to whom? By what definition?
- No value hierarchy for conflicting situations
- "Dangerous" is vague — what counts?
- No autonomy boundaries
- No decision guidance

### After: Intent-Encoded AGENTS.md (Strong Intent)

```markdown
# AGENTS.md

## Values (priority order)
1. Sovereignty — always prefer solutions we own and control
2. Compounding — choose work that builds infrastructure for future leverage
3. Revenue sustainability — every project needs a path to money
4. Quality — ship correctly; don't optimize for speed at quality's expense
5. Care — the work ultimately serves people; never harm for metrics

## Decision Boundaries
Act autonomously: file ops, drafts, research, local code, memory updates
Ask first: external sends, publish actions, deletes, financial anything, new cron creation
Present options: any gray area, any ambiguity >30%, any conflict between values 1-3

## Autonomy Ladder
Proven crons → Full autonomy
Routine ops → Report and proceed  
Content for publish → Draft and deliver
System changes → Plan and confirm
Financial / irreversible → Supervised only

## Trade-offs
Speed vs Quality: Quality wins unless I explicitly say "rough draft OK"
Cost vs Thoroughness: Thorough on revenue decisions; cheap on internal tooling
Growth vs Sustainability: Sustainability wins; ask "can we maintain this?" first
Autonomy vs Safety: Safety wins; escalate when uncertain

## Voice
Direct, no corporate filler, warm but not soft. Have opinions.
If you disagree with my approach, say so before executing.
```

---

## Before/After Comparison: The Same Situation, Different Outcomes

**Situation:** You ask your agent to "set up monitoring for the n8n service."

**Before (weak intent):**
Agent installs a monitoring tool, adds it to startup, creates a complex cron that texts you every 5 minutes with status, modifies the LaunchAgent configuration, and reports "done."

Problems: You didn't ask for SMS alerts. Cron every 5 minutes is excessive. Config changes weren't confirmed.

**After (strong intent):**
Agent responds: "For n8n monitoring, I can set up:
- A cron that checks every 6 hours and alerts only on failure (Tier 4: Report and proceed) — recommended
- Or a more comprehensive health check every hour with Telegram delivery to Infra topic (Tier 3: Draft and deliver — I'll show you the cron config first)

I'd recommend the first option: simpler, less noise, will alert you if it actually breaks. Shall I proceed?"

The agent understood: "monitoring" involves system changes (Tier 2+ by default), the trade-off between noise and utility, and the preference for simple compounding solutions.

---

## Encoding Intent in Practice

Intent lives in three places:

**1. SOUL.md** — Values and identity (read every session)
**2. AGENTS.md** — Operational rules and boundaries (read every session)
**3. Task specifications** — Task-specific intent (read per task)

For task specs, use this pattern when delegating to sub-agents:

```markdown
## Task Specification

**Goal:** [What you want accomplished]
**Why:** [The intent behind the request]
**Constraints:** [What NOT to do]
**Success looks like:** [Specific, verifiable outcome]
**Values to prioritize:** [Which values matter most for this task]
**Autonomy tier:** [Which tier from the ladder]
**Escalation trigger:** [What should cause the agent to stop and ask]
```

**Example:**
```markdown
## Task: Set up morning briefing cron

**Goal:** Create a cron that sends me a morning briefing at 8am MDT
**Why:** I want to start each day with context without manually checking anything
**Constraints:** 
  - Don't include more than 5 items (I'll stop reading long briefings)
  - Don't send if it's a weekend unless I specifically ask
  - Don't modify any existing crons
**Success looks like:** Cron appears in `openclaw cron list`, runs at 8am, delivers ≤5 items to Telegram General topic
**Values:** Compounding (once set up, runs forever) > Speed (get it right)
**Autonomy tier:** Tier 3 — show me the cron definition before activating
**Escalation trigger:** Stop if any part requires modifying AGENTS.md or gateway config
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
