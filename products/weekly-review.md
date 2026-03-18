# Weekly Review System — Your AI-Assisted Weekly Reset

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 15-20 minutes

A complete weekly review system for AI-assisted operations. Capture what happened, clear the mental queue, set the week ahead, and let your agent prepare the brief before you sit down. Designed for founders and solopreneurs who keep losing time to reviewing what they should be executing.

---

## What's In This Package

- Weekly review prompt template (copy-paste ready for OpenClaw)
- Automated pre-brief script (agent prepares the review before you read it)
- Memory update protocol (how to capture the week in persistent files)
- Goal tracking template (rolling 90-day view)
- Content queue review (what shipped, what's stuck, what's next)
- Infrastructure health summary (was anything broken this week)
- Revenue snapshot section (Stripe, pipeline, client status)
- Cron job setup for automated Friday 5 PM preparation
- Customization guide: what to add for your specific operation
- Integration with MEMORY.md (what to write down vs. let go)

---

## Why Weekly Reviews Fail

Most weekly review systems fail because they require too much manual effort. You have to collect information from 5 different places, remember what happened, and write it all up yourself.

This system flips it: your agent collects the data before you sit down. You read a prepared brief and add judgment, not administrative work.

---

## The Weekly Review Structure

A good weekly review has four parts:

1. **What happened** — factual, from your files and logs
2. **What matters** — your judgment on what's significant
3. **What's next** — priorities for the coming week
4. **What to remember** — anything that needs to survive to next week

Your agent handles part 1 automatically. You handle parts 2-4.

---

## Setup: The Weekly Review Prompt

Save this to `~/.openclaw/workspace/scripts/weekly-review-prompt.md`:

```markdown
# Weekly Review Prompt

Run this at the start of each weekly review session.

## Agent Instructions

Prepare my weekly review brief. Specifically:

1. **Activity Summary** — Read memory/[LAST_WEEK_DATES].md and summarize:
   - Major tasks completed
   - Decisions made
   - Blockers encountered and resolved
   - Anything still open

2. **Content Status** — Check content queues and summarize:
   - What published this week (count by platform)
   - What's queued and ready
   - What's stuck or overdue
   - Engagement/growth if available

3. **Infrastructure Health** — Check the heartbeat logs and summarize:
   - Any alerts fired this week
   - Services that went down and recovered
   - Anything still degraded

4. **Revenue Snapshot** (if Stripe configured):
   - This week vs. last week
   - Open pipeline (Gumroad/Lemon Squeezy pending)
   - Any payment issues

5. **Open Projects** — Read memory/open-projects.md and list:
   - Active projects with status
   - Blocked projects
   - Projects completed this week

Format as a clean brief I can read in 5 minutes.
After I've read it, ask: "What do you want to carry forward to next week?"
```

---

## The Agent Preparation Cron

Add this to your crontab to have the agent prepare a brief every Friday at 5 PM:

```bash
# Weekly review prep — every Friday at 5 PM
0 17 * * 5 /Users/$(whoami)/.openclaw/workspace/scripts/weekly-review-prep.sh
```

Create the script at `~/.openclaw/workspace/scripts/weekly-review-prep.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

WORKSPACE="$HOME/.openclaw/workspace"
WEEK=$(date +%Y-W%V)
OUTPUT="$WORKSPACE/memory/weekly-brief-$WEEK.md"

echo "# Weekly Brief — Week $WEEK" > "$OUTPUT"
echo "**Prepared:** $(date)" >> "$OUTPUT"
echo "" >> "$OUTPUT"

# Append memory files from this week
MONDAY=$(date -v-mon +%Y-%m-%d 2>/dev/null || date -d "last monday" +%Y-%m-%d)
echo "## Memory Files This Week" >> "$OUTPUT"
for day in $(seq 0 6); do
  DATE=$(date -v+"${day}d" -v"$MONDAY" +%Y-%m-%d 2>/dev/null || date -d "$MONDAY +${day} days" +%Y-%m-%d 2>/dev/null || echo "")
  FILE="$WORKSPACE/memory/$DATE.md"
  if [[ -f "$FILE" ]]; then
    echo "" >> "$OUTPUT"
    echo "### $DATE" >> "$OUTPUT"
    cat "$FILE" >> "$OUTPUT"
  fi
done

echo "Weekly review brief prepared at $OUTPUT"
```

Make it executable:
```bash
chmod +x ~/.openclaw/workspace/scripts/weekly-review-prep.sh
```

---

## The Review Session Flow

### Before You Sit Down

Your agent has already prepared the brief. Open it:

```
cat ~/.openclaw/workspace/memory/weekly-brief-$(date +%Y-W%V).md
```

Or in Telegram:

```
Run the weekly review.
```

### During the Review (15-20 minutes)

**Minutes 1-5: Read the brief**
Don't take notes yet. Just absorb what happened.

**Minutes 5-10: Add judgment**
Tell your agent what matters:
```
The Substack article was the most important thing. The Discord bot can wait until Q2.
The revenue dip is expected — January is always slow.
```

**Minutes 10-15: Set priorities**
```
This week I want to focus on:
1. [TOP PRIORITY]
2. [SECOND PRIORITY]
3. [NICE TO HAVE]
Nothing else unless something breaks.
```

**Minutes 15-20: Write it down**
Ask your agent to update MEMORY.md with this week's summary:

```
Add this week's summary to MEMORY.md:
- Week of [DATE]: [1-2 sentences about what happened and what shifted]
- Current priority: [TOP PRIORITY]
- Open question: [anything unresolved that needs to carry forward]
```

---

## Goal Tracking Template

Keep a rolling 90-day view in `~/.openclaw/workspace/GOALS.md`:

```markdown
# GOALS.md — Rolling 90-Day View

**Updated:** [DATE]

## 90-Day Horizon
What do I want to be true in 90 days?
- [ ] [GOAL 1]
- [ ] [GOAL 2]
- [ ] [GOAL 3]

## This Month
What needs to happen this month?
- [ ] [MILESTONE 1]
- [ ] [MILESTONE 2]

## This Week
What am I actually doing?
- [ ] [TASK 1]
- [ ] [TASK 2]
- [ ] [TASK 3]

## Paused / Deprioritized
Things I've consciously decided to ignore:
- [ITEM] — paused because [REASON] — revisit [DATE]
```

Ask your agent to update GOALS.md at the end of each review:
```
Update GOALS.md based on our review. This week's focus is [X].
Move [Y] to Paused with reason "revisit in April."
```

---

## What to Write in MEMORY.md

Not everything belongs in long-term memory. Use this rule:

**Write it down if:**
- A major decision was made (especially one you'll want to understand later)
- Something changed direction
- A system broke and was fixed in a non-obvious way
- Revenue hit a milestone
- A relationship (client, collaborator) had a significant interaction

**Let it go if:**
- Routine task completed successfully
- Minor bug fixed
- Social media post published

**Format for MEMORY.md entries:**
```markdown
## [DATE] — [1-sentence description]

[2-3 sentences: what happened, why it matters, what changed]

**Tags:** [system/revenue/content/relationships/direction]
```

---

## Integration with Daily Memory

The weekly review feeds from daily memory logs (`memory/YYYY-MM-DD.md`).

Your agent writes daily logs automatically if you have Grove (ops/journal agent) configured. If you don't:

```bash
# Add to crontab — daily log init at 8 AM
0 8 * * * echo "# $(date +%Y-%m-%d)\n\n## Today's Context\n\n" >> ~/.openclaw/workspace/memory/$(date +%Y-%m-%d).md
```

---

## Troubleshooting

**Review feels too long**
Keep it under 20 minutes by ruthlessly cutting "should we revisit [old thing]?" conversations. The review is for carrying forward, not for relitigating past decisions.

**Agent doesn't know what happened this week**
Daily memory logs are probably not being written. Check if Grove is running, or set up the daily log cron above.

**Too many open projects**
Apply the constraint: maximum 3 active projects at a time. Move anything beyond 3 to Paused in GOALS.md. This is the most common productivity failure.

**Review frequency**
Weekly is the minimum. Some operators do Friday/Monday pairs: Friday to close the week, Monday to confirm priorities before execution starts.

---

*Your best week starts with a clear head about what the last one was.*
