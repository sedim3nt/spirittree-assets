# Learning Agent — AI Personal Tutor Package

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 15-20 minutes

Turn any topic into a structured learning path. Your AI breaks complex subjects into digestible modules, creates practice exercises, tracks progress, and adjusts based on your pace.

---

## What's In This Package

- `ROLE.md` for your learning agent
- Topic decomposition framework
- Learning path generator (beginner → advanced)
- Practice exercise templates
- Progress tracking system
- Spaced repetition schedule
- Resource curation workflow

---

## ROLE.md — Learning Agent

Copy to `agents/learning/ROLE.md`:

```markdown
# ROLE.md — Learning Agent

**Role:** Personal Learning Tutor
**Model:** Claude Sonnet
**Channel:** Sub-agent or direct

## Responsibilities
- Decompose complex topics into learnable modules
- Create structured learning paths
- Generate practice exercises and assessments
- Track learning progress and adjust difficulty
- Curate quality resources (books, courses, tools)

## Teaching Principles
1. **Start with why** — connect every concept to real-world application
2. **Concrete before abstract** — example first, then principle
3. **Check understanding** — ask questions, don't just explain
4. **Spaced repetition** — revisit earlier concepts in new contexts
5. **Translate expert** — make it accessible without dumbing it down

## Constraints
- Adjust complexity to stated level (beginner / intermediate / advanced)
- Flag when a prerequisite is missing ("You need X to understand Y")
- Don't skip to advanced material before foundations are solid
- Honest about uncertainty — don't explain concepts you're not sure about
```

---

## Topic Decomposition Framework

### Step 1: Topic Audit

Before building a learning path, understand what you're getting into:

```
Topic audit for: [TOPIC]

I want to learn [TOPIC] at [BEGINNER / INTERMEDIATE / ADVANCED] level.

Decompose this topic:
1. **Core concepts** — the 5-7 foundational ideas I MUST understand
2. **Prerequisite knowledge** — what do I need before starting?
3. **Common misconceptions** — what do most beginners get wrong?
4. **The mental models** — how do experts actually think about this?
5. **Typical learning curve** — where do people usually get stuck?
6. **The minimum viable knowledge** — what's the 20% that gives 80% of the value?

Assessment: What level of mastery am I aiming for?
- Recognition (I know this exists)
- Understanding (I can explain it)
- Application (I can use it)
- Fluency (I can teach it and handle edge cases)
```

### Step 2: Prerequisite Check

```
I want to learn [TOPIC]. Check my prerequisites.

Ask me 5 diagnostic questions to assess my current level.
After I answer, tell me:
1. What I already know well enough
2. What I need to fill in before starting
3. Recommended starting point in the learning path
```

---

## Learning Path Generator

### Full Learning Path Prompt

```
Build a learning path for: [TOPIC]

My level: [BEGINNER / INTERMEDIATE]
My goal: [What I want to be able to do with this knowledge]
Time available: [X hours/week]
Timeframe: [X weeks]

Structure the path as modules:

**Module 1: Foundation**
- Concepts covered
- Time estimate
- Practice exercise
- Success check: what can I do at the end of this module?

**Module 2: Core Skills**
[same structure]

**Module 3: Application**
[same structure]

**Module 4: Advanced / Edge Cases**
[same structure]

For each module:
- Order concepts from most to least foundational
- Include at least one hands-on exercise
- Include a check at the end to confirm I understood before moving on
```

### Module Deep-Dive

When you're ready to study a specific module:

```
Teach me Module [N]: [MODULE TITLE] from my [TOPIC] learning path.

Teaching method:
1. Start with a real-world example (before the explanation)
2. Explain the concept
3. Give me 3 practice exercises (increasing difficulty)
4. Ask me 3 questions to check my understanding
5. Preview: how does this connect to the next module?

If I'm confused, slow down and try a different angle.
If I'm getting it, push me a bit harder.
```

---

## Practice Exercise Templates

### Concept Application Exercise

```
Give me a practice exercise for: [CONCEPT]

Exercise requirements:
- Real-world scenario (not abstract)
- Difficulty: [EASY / MEDIUM / HARD]
- Solvable in 15-30 minutes
- Has a clear right/wrong outcome I can verify

Include:
1. The exercise prompt
2. Hints (hidden — tell me if I ask)
3. The solution with explanation
4. What this exercise is testing
```

### Worked Example Request

```
Walk me through a worked example of [CONCEPT/SKILL].

Format:
1. State the problem
2. How an expert would approach it (thinking out loud)
3. Step-by-step solution
4. Common mistakes people make here
5. How I'd vary this in a real scenario
```

### Self-Assessment Quiz

```
Quiz me on [MODULE/TOPIC].

5 questions, increasing difficulty:
- Questions 1-2: Recognition (do I know the terms?)
- Questions 3-4: Understanding (can I explain/apply?)
- Question 5: Application (can I use this on a new problem?)

After I answer, tell me:
- What I got right and why it's right
- What I got wrong and the correct understanding
- Overall: am I ready to move on or do I need to review?
```

---

## Progress Tracking System

Create `learning/progress.md`:

```markdown
# Learning Progress

## Active Learning Path: [TOPIC]

**Started:** [DATE]
**Target completion:** [DATE]
**Time invested:** [X hours total]

### Module Progress

| Module | Status | Date Completed | Notes |
|--------|--------|----------------|-------|
| 1. Foundation | ✅ Complete | [DATE] | [Any notes] |
| 2. Core Skills | 🔄 In Progress | — | Currently on concept X |
| 3. Application | ⏳ Not Started | — | — |
| 4. Advanced | ⏳ Not Started | — | — |

### Concepts Mastered

- [Concept 1] — can explain ✅
- [Concept 2] — can apply ✅
- [Concept 3] — still shaky ⚠️

### Sticking Points

- [Thing I keep getting confused about]
- [Question I haven't resolved yet]

### Resources Used

- [Book/Course/Article] — [Rating/notes]
- [Book/Course/Article] — [Rating/notes]
```

### Progress Check Prompt

Run weekly:

```
Progress check for [TOPIC] learning.

Read learning/progress.md.

1. What module am I on?
2. What was the last concept I completed?
3. What should I review (anything flagged as shaky)?
4. Based on my pace, will I hit my target completion date?
5. Should I adjust the pace or scope?

Recommend: what to do in my next study session.
```

---

## Spaced Repetition Schedule

Spaced repetition is how long-term memory works. Use this schedule:

| Review | When |
|--------|------|
| First review | 1 day after learning |
| Second review | 3 days after first review |
| Third review | 1 week after second review |
| Fourth review | 2 weeks after third review |
| Fifth review | 1 month after fourth review |

### Spaced Repetition Prompt

```
Set up spaced repetition for concepts I've learned.

I completed Module [N]: [TITLE] on [DATE].

Key concepts to retain:
1. [Concept 1]
2. [Concept 2]
3. [Concept 3]

Create review prompts for each concept at these intervals:
- Tomorrow ([DATE])
- [DATE +3 days]
- [DATE +1 week]
- [DATE +2 weeks]

Format as flashcard-style questions (not "define X" — use application questions).
Add to learning/review-schedule.md.
```

### Review Schedule File

Create `learning/review-schedule.md`:

```markdown
# Review Schedule

## Upcoming Reviews

### [DATE]
- [ ] [Concept] — [Review prompt or question]
- [ ] [Concept] — [Review prompt or question]

### [DATE + 3 days]
- [ ] [Concept from earlier module]

### [DATE + 1 week]
- [ ] [Earlier concept]
```

---

## Resource Curation

### Resource Research Prompt

```
Find the best resources for learning [TOPIC] at [LEVEL].

Research and recommend:

**Books (1-2)**
- Best for foundational understanding
- Avoid: textbooks that are comprehensive but dense

**Courses (1-2)**
- Video/interactive courses for hands-on learners
- Look for: high completion rate, practical exercises

**Free Resources**
- Official documentation or tutorials
- Quality blog posts or video series

**Practice Platforms**
- Where to apply the skills in real exercises

For each resource:
- Why it's good for this level
- Estimated time investment
- What specifically it covers

Skip anything that's more than 2 years old for fast-moving fields.
```

---

## Troubleshooting

**Learning path is too theoretical**
- Tell the agent: "I learn better with hands-on projects — rebuild the path around projects"
- Ask for a project-based path: "Build X using Y concepts"
- Request worked examples before explanations

**Getting stuck on a concept**
- Ask: "Try explaining [concept] 3 different ways"
- Ask for an analogy: "Explain [concept] like I'm a [different background]"
- Ask what the prerequisite knowledge is — maybe you're missing a foundation

**Progress tracker falling behind**
- Run a quick log: "Log 2 hours on [module], [date]" — takes 5 seconds
- Weekly check-in is more sustainable than daily logging
- Use simple status: ✅ done / 🔄 in progress / ⏳ not started — nothing more complex

**Spaced repetition not sticking**
- Make review questions harder — recognition is not retention
- Use output-based questions: "Explain X to me" not "Define X"
- Connect new reviews to current work: "Here's a problem at work — apply [concept]"

**Difficulty calibration off**
- If too easy: "Assume I understand [basics] and skip to application"
- If too hard: "Start over from first principles on this concept"
- Tell the agent your background explicitly — it adjusts

---

*Built on OpenClaw. No external tools required.*
