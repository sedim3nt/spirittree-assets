# Brand Voice Agent — AI Writing Consistency Package

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Make AI write in YOUR voice, not AI voice. Feed it 10 samples, get a voice profile that stays consistent across every platform. Includes the De-AI-ify pass, CONTENT-STANDARD.md, and platform-specific adaptation guides.

---

## What's In This Package

- `ROLE.md` for your brand voice agent
- Voice Profile Builder (analyze writing samples → voice DNA)
- `CONTENT-STANDARD.md` quality checklist
- De-AI-ify pass — strip robotic patterns
- Platform voice adaptation guide
- Tone calibration matrix
- Before/after examples

---

## ROLE.md — Brand Voice Agent

Copy to `agents/brand/ROLE.md`:

```markdown
# ROLE.md — Brand Voice Agent

**Role:** Brand Voice Agent
**Model:** Claude Sonnet
**Channel:** Sub-agent or direct

## Core Mission
Write in the operator's voice. Not in AI voice. Not in "professional content" voice.
The operator's actual voice — the way they think and talk.

## Process
1. Load voice-profile.md before any writing task
2. Apply CONTENT-STANDARD.md checklist before delivery
3. Run De-AI-ify pass on every output
4. Adapt for platform while maintaining core voice

## Voice Constants (override with voice-profile.md)
- Clear and direct > comprehensive and thorough
- Specific examples > abstract claims
- Own opinions > hedged observations
- Punchy > verbose
- Human > professional

## Constraints
- Never use words on the De-AI-ify list
- Never start with "Certainly!" or "Great question!"
- Never end with "In conclusion" or "To summarize"
- Don't explain what you're about to say — just say it
```

---

## Voice Profile Builder

### Step 1: Gather Your Writing Samples

Collect 8-10 pieces of YOUR writing. Can be:
- Email threads (informal is great)
- Social media posts
- Blog articles
- Slack/Discord messages
- Meeting notes or memos
- Text messages (show how you talk)

The more varied the better. Don't curate for quality — include average stuff too.

### Step 2: Run the Voice Analysis Prompt

```
I'm going to share 10 samples of my writing. Analyze them to build my voice profile.

For each sample, note:
1. Sentence length patterns
2. Vocabulary choices (technical / casual / mix)
3. How I handle complexity (simplify vs. embrace)
4. Tone (serious / irreverent / warm / direct)
5. What I emphasize in arguments
6. Metaphors or analogies I use
7. Things I never say (patterns in the negative)

After all samples, synthesize into a Voice DNA:
1. Primary voice trait (the single most defining characteristic)
2. Secondary traits (3-5)
3. What makes this voice distinct from generic AI writing
4. Words/phrases I use frequently (voice fingerprint)
5. Words/phrases I never use
6. Platform-specific notes if any emerge

[PASTE YOUR WRITING SAMPLES]
```

### Step 3: Save Your Voice Profile

After the analysis, ask the agent to write `voice-profile.md`:

```
Based on your voice analysis, create my voice-profile.md.

Format:
---
# Voice Profile

## Voice DNA
[Primary trait in 1 sentence]

## Core Traits
- [Trait 1 with example]
- [Trait 2 with example]
- [Trait 3 with example]

## Sentence Structure
[Typical pattern — short? varied? how complex?]

## Vocabulary
[Technical level / register / any domain-specific patterns]

## What I Say
Common phrases and patterns: [list]

## What I Never Say
Avoid these: [list specific to this person's voice]

## Platform Notes
- Written/Substack: [any shifts]
- Twitter/X: [how voice adapts]
- LinkedIn: [how voice adapts]
---
```

---

## CONTENT-STANDARD.md

Your 10-point quality checklist. Every piece of content must pass before publishing:

```markdown
# CONTENT-STANDARD.md

Run this before every piece of content leaves your system.

## 10-Point Quality Check

### 1. The Sniff Test
Would you be embarrassed to show this to someone you respect?
Yes → revise. No → continue.

### 2. Does It Say Something?
What is the one idea this piece communicates?
If you can't state it in one sentence → needs a clearer thesis.

### 3. No AI Tells
Remove immediately:
- delve, comprehensive, leverage, utilize, facilitate
- furthermore, moreover, in conclusion, it's important to note
- as an AI, I cannot, I'd be happy to
- game-changing, revolutionary, groundbreaking
- ensure, foster, navigate, embark
- robust, seamless, cutting-edge, transformative

### 4. Specific Over General
Replace every abstraction with a concrete example.
Bad: "AI can improve your workflow"
Good: "This cut my writing time from 3 hours to 40 minutes"

### 5. Earned Claims Only
Every strong claim needs evidence, example, or reasoning.
Opinion: own it. Say "I think" not "research shows."

### 6. Voice Check
Read it out loud. Does it sound like the person — or like a press release?
Generic → add personality.

### 7. Platform Fit
Right length and format for this platform?

### 8. The Opener
Does line 1 earn line 2?
Context-setting opening → delete and start later.

### 9. The Close
Does it end with something to think about or act on?
Trailing off → find a real landing.

### 10. De-AI-ify Pass
Final scan. Remove generated patterns. Target: sounds like a sharp human.
```

---

## De-AI-ify Pass

### The Complete Kill List

Words and phrases to eliminate from any AI output:

**Structural clichés:**
- "In today's fast-paced world..."
- "It's important to note that..."
- "Let's dive in" / "Let's explore"
- "In conclusion" / "To summarize" / "In summary"
- "With that said" / "That being said"
- "At the end of the day"
- "It goes without saying"
- "Without further ado"

**Praise phrases (never use at start of response):**
- "Certainly!" / "Absolutely!" / "Of course!"
- "Great question!" / "Excellent point!"
- "I'd be happy to help"
- "That's a great observation"

**Corporate filler:**
- leverage, utilize, facilitate, implement
- robust, seamless, cutting-edge, state-of-the-art
- innovative, revolutionary, transformative, groundbreaking
- comprehensive, holistic, synergistic
- ensure, foster, empower, navigate
- embark (on a journey)

**Hollow qualifiers:**
- "It's worth noting that..."
- "It's important to consider..."
- "One must keep in mind..."
- "Needless to say"

**AI tells:**
- "As an AI language model..."
- "I don't have personal opinions, but..."
- "I'm unable to have feelings, however..."
- "My training data..."

### De-AI-ify Prompt

Run this on any output before delivery:

```
De-AI-ify this text. Remove all words and phrases from this list:
[PASTE KILL LIST above]

Also:
- Replace passive voice with active where it reads better
- Cut any sentence that exists only to set up the next sentence
- If a paragraph starts with context, cut the context
- If a sentence ends weakly, end it earlier

Do not change: facts, structure, core ideas, the author's actual voice.
Show me the before/after word count.

TEXT TO DE-AI-IFY:
[paste content]
```

---

## Platform Voice Adaptation Guide

Same voice, different register. Here's how to adapt:

### Substack / Long-form Blog
- **Structure:** Can breathe — use sections, subheadings, white space
- **Length:** 1,200-3,000 words is the sweet spot
- **Tone:** Conversational but substantive. This is where you go deep.
- **What works:** Personal stories, contrarian takes, research synthesis
- **What doesn't:** Bullet-point-only structure, corporate framing

### X / Twitter
- **Every tweet must stand alone.** No context tweets.
- **First word matters most.** Don't start with "I think" or "Just a reminder"
- **Cut until it hurts.** Then cut more.
- **Thread format:** Tweet 1 is the hook. Tweet 2-4 are the argument. Last tweet = the turn.
- **Voice at max compression:** your voice, every ounce of filler gone.

### Bluesky
- **Similar to X** but 300 chars gives you slightly more room
- Community is tech/indie-web heavy — rewards specificity and honesty
- Less algorithmic pressure — you can be more obscure

### LinkedIn
- **Professional register, but still human.** Corporate is death.
- **Story-first.** The best LinkedIn posts start with a specific moment, not a claim.
- **Length:** 150-300 words optimal. Paragraphs, not bullets.
- **End with a question** — drives comments, and LinkedIn rewards this.
- **Never** use the word "synergy" on LinkedIn. Just never.

---

## Tone Calibration Matrix

Adjust along these axes based on context:

```
FORMAL ←————————————————→ CASUAL
Technical docs           Text messages
Academic writing         Social media
Legal communications     Internal memos

SERIOUS ←————————————————→ IRREVERENT
Crisis communications    Normal operations
Investor materials       Community building
Sensitive topics         Product launches

EXPANSIVE ←————————————→ COMPRESSED
Long-form articles       Social posts
Tutorials                Executive summaries
Research reports         Push notifications
```

Configure in your voice-profile.md:
```markdown
## Default Calibration
- Formal/Casual: 40% formal, 60% casual
- Serious/Irreverent: situational — see platform notes
- Expansive/Compressed: compressed-first, expand on request
```

---

## Before/After Examples

### Before (generic AI output)
> "It's important to note that leveraging AI tools can significantly enhance your productivity. In today's fast-paced world, many professionals are finding that comprehensive AI solutions enable them to navigate complex workflows more efficiently. As an AI, I'd be happy to help you explore these transformative technologies."

### After (De-AI-ified + voice applied)
> "AI cut my research time by 70%. That's not a motivational poster — it's Tuesday. Here's what actually happened when I stopped treating it like a search engine."

---

### Before (passive, hedged)
> "There may be some considerations worth noting regarding the implementation of such strategies, as they could potentially have varying degrees of effectiveness depending on the specific context."

### After
> "This works — with one caveat. If your audience is enterprise, cut step 3. If they're solo operators, step 3 is the whole thing."

---

## Troubleshooting

**Voice sounds inconsistent across pieces**
- Always load voice-profile.md at the start of content sessions
- Add to your prompt: "Read voice-profile.md before writing anything"
- If it drifts mid-session, remind: "Re-read the voice profile. Something went wrong."

**De-AI-ify pass misses things**
- Run it twice — different patterns surface on second pass
- Ask: "Read this as a skeptical reader. What sounds AI-generated?"
- Use CONTENT-STANDARD.md as a second check

**Voice profile feels generic**
- Add more samples — especially informal writing
- Be explicit about what you hate: "I never say X. I never structure things like Y."
- Give negative examples: "This is NOT my voice: [paste example]"

**Platform adaptation loses the voice**
- Keep adapting voice profile with notes per platform
- Test: take a Substack paragraph, compress it to a tweet — does it still sound like you?

---

*Built on OpenClaw. No external tools required.*
