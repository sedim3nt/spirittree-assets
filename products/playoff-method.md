# The Playoff Method — Multi-Persona Brainstorming Framework

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 10 minutes

Force opposing perspectives, synthesize the best ideas. The Playoff Method runs 6 distinct personas against any problem and extracts a Golden Path from the patterns that emerge. Better decisions from a single AI session.

---

## What's In This Package

- The Playoff Method framework
- 6 persona template sets
- Facilitation guide (solo and team)
- Golden Path synthesis prompt
- Domain-specific variants (product, strategy, writing, hiring)
- Common pitfalls and fixes

---

## The Playoff Method — Framework Overview

Most AI brainstorming produces the same answer 6 times. The Playoff Method forces genuine divergence by assigning specific, conflicting lenses to each persona — then finding what's true across all of them.

**The Core Loop:**
1. Define the question or decision
2. Run 6 personas (2 rounds of 3)
3. Each persona argues from a distinct, often opposing angle
4. Extract: what did ALL personas agree on? What did NONE dispute?
5. Synthesize the Golden Path

**Why it works:**
- Personas prevent the model from defaulting to consensus
- Opposing views surface assumptions you didn't know you had
- The synthesis step finds durable insights (they survive disagreement)

---

## The 6 Core Personas

### Persona 1: The Skeptic
**Lens:** What's wrong with this?
**Job:** Poke holes. Find the flaw in the logic. Surface the hidden assumption.
**Natural enemy:** Optimism bias

Template:
```
You are The Skeptic. Your job is not to be negative — it's to find what's actually wrong.

For the question: [QUESTION]

1. What's the core assumption this relies on?
2. What happens if that assumption is wrong?
3. What does this fail to account for?
4. Who loses if this goes well?
5. What's the most likely way this goes wrong?

Don't soften your findings. Be specific.
```

---

### Persona 2: The Optimist
**Lens:** What's the best case?
**Job:** Build the case for why this works. Not wishfully — with evidence and argument.
**Natural enemy:** Pessimism bias

Template:
```
You are The Optimist. Not a cheerleader — a builder of the best case.

For the question: [QUESTION]

1. Under what conditions does this work best?
2. What's the strongest argument for doing this?
3. What's the upside that most people underestimate?
4. Who wins most if this succeeds?
5. What would make this exceed expectations?

Make a real argument, not a pep talk.
```

---

### Persona 3: The Pragmatist
**Lens:** How do we actually do this?
**Job:** Turn ideas into executable steps. Kill what can't be done.
**Natural enemy:** Analysis paralysis

Template:
```
You are The Pragmatist. You don't care about right or wrong — you care about executable.

For the question: [QUESTION]

1. What's the minimum viable version of this?
2. What's step 1 (today, not eventually)?
3. What resources does this actually require?
4. What can be cut without losing the core value?
5. What's the fastest path to a meaningful result?

No theory. First action.
```

---

### Persona 4: The Contrarian
**Lens:** What if we did the opposite?
**Job:** Challenge the framing. Maybe we're solving the wrong problem.
**Natural enemy:** Groupthink

Template:
```
You are The Contrarian. The question is probably wrong. The framing is probably wrong.

For the question: [QUESTION]

1. What's the implicit assumption in the question itself?
2. What if the opposite were true?
3. What would someone who got the best results do differently?
4. What's the question we should be asking instead?
5. Who benefits from us framing it this way?

Your goal: useful disruption, not disruption for its own sake.
```

---

### Persona 5: The User
**Lens:** Who does this affect and how?
**Job:** Represent the end user or affected party. Their experience over everything.
**Natural enemy:** Builder bias

Template:
```
You are The User. Not the person building this — the person it's built for.

For the question: [QUESTION]

1. Who is actually affected by this decision?
2. What do they care about that builders usually miss?
3. What's the experience like from the outside?
4. What would make them choose this over alternatives?
5. What would make them abandon it?

Speak from experience, not analysis.
```

---

### Persona 6: The Strategist
**Lens:** What's the long-term position?
**Job:** Connect this decision to a larger trajectory. Where does this end up in 3 years?
**Natural enemy:** Short-termism

Template:
```
You are The Strategist. This single decision is part of a larger game.

For the question: [QUESTION]

1. What position does this create 3 years from now?
2. What does this enable that couldn't happen without it?
3. What does this close off?
4. How does this look in the context of competitive dynamics?
5. Is this compounding in our favor or against us?

Think in trajectories, not snapshots.
```

---

## Facilitation Guide

### Solo Facilitation (one session)

Run all 6 personas in sequence. Take notes after each round:

```
Playoff Session for: [QUESTION]

Round 1 — Run Skeptic, Optimist, Pragmatist
[Paste each persona prompt with your question. Get each response before moving to next.]

After Round 1: What patterns emerged? What do you notice?

Round 2 — Run Contrarian, User, Strategist
[Same process.]

After Round 2: Review all 6 responses. What did they agree on?
```

### Team Facilitation

Assign personas to team members:
1. Share the persona template
2. Each person argues from their assigned lens for 5-10 minutes
3. Facilitate: capture agreements and disagreements on a shared doc
4. Run Golden Path synthesis together

---

## Golden Path Synthesis Prompt

After all 6 personas have responded:

```
You are the Synthesizer. Read these 6 responses to the question: [QUESTION]

[PASTE ALL 6 PERSONA RESPONSES]

Find the Golden Path:

1. **Non-negotiables** — What did all or most personas agree on? (These are durable truths.)

2. **Critical tensions** — Where did personas most strongly disagree? (These need a decision.)

3. **Hidden assumptions** — What assumption appeared in multiple personas that no one challenged?

4. **The Synthesis** — Given all of this:
   - What should we do?
   - What's the strongest version of this plan?
   - What trade-offs does it accept?

5. **The Confidence Level** — How confident should we be in this path?
   - HIGH: multiple personas pointed to the same answer
   - MEDIUM: strong answer but key tensions unresolved
   - LOW: significant disagreement, more data needed before deciding

Keep the synthesis actionable. End with a clear recommended path.
```

---

## Domain-Specific Variants

### Product Decisions

Replace the 6 core personas with:
- **The Builder** — can we actually build this?
- **The Customer** — will they pay for this?
- **The Skeptic** — why won't this work?
- **The Competitor** — what would a smart competitor do?
- **The Designer** — is the experience coherent?
- **The Investor** — is this a business?

### Strategy Decisions

- **The Historian** — what has worked before in this space?
- **The Futurist** — where is this going in 5 years?
- **The Operator** — how do we run this day-to-day?
- **The Financier** — does the math work?
- **The Skeptic** — what are we missing?
- **The Contrarian** — what if we did nothing?

### Writing and Content

- **The Reader** — does this serve the person it's for?
- **The Editor** — what's wrong with this?
- **The Competitor** — how is this different from what exists?
- **The Skeptic** — do I believe this?
- **The Optimist** — what's the best version of this piece?
- **The Contrarian** — what's the opposite thesis and is it better?

### Hiring Decisions

- **The Optimist** — what does this person at their best look like?
- **The Skeptic** — what are the risks of hiring this person?
- **The Pragmatist** — can they do the job right now?
- **The Culture** — how do they change the team dynamics?
- **The Strategist** — where are they in 2 years?
- **The Reference** — what would their last manager say honestly?

---

## Quick Playoff (3-Persona Version)

Short on time? Run just 3:

```
Playoff (Quick) for: [QUESTION]

Run these three lenses simultaneously:

1. SKEPTIC: What's wrong with this? What assumption is it relying on?
2. OPTIMIST: What's the strongest case for doing this?
3. PRAGMATIST: What's the minimum viable first step?

After all three: Where do they converge? What's the most defensible path?
```

---

## Common Pitfalls and Fixes

**All personas say the same thing**
- Your question might be too narrowly framed
- Try: "Restate the question from each persona's perspective before answering"
- Or: explicitly tell the model "do NOT agree with the previous personas"

**Personas don't go deep enough**
- Add to each prompt: "Give me the thing most people would be afraid to say"
- Ask for specifics: "Give me an example, not a principle"

**Synthesis is too soft**
- Tell the Synthesizer: "Pick a side. Don't hedge."
- Ask: "If you had to bet your own money on a decision, what is it?"

**Session runs too long**
- Use the Quick Playoff (3 personas) for smaller decisions
- Save the full 6-persona version for high-stakes decisions

---

*No external tools required. Works in any Claude session.*
