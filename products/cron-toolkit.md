# Cron Toolkit — 15 Complete Automation Templates

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 10-30 minutes per template

Copy-paste cron templates. Each includes the exact schedule expression, the full message prompt, expected behavior, and a LaunchAgent plist for macOS.

---

## How to Use These Templates

**Option A — OpenClaw native crons:**
Configure in your OpenClaw workspace via the `openclaw cron` command or config file.

**Option B — macOS LaunchAgents:**
Each template includes a `.plist` file. Copy, customize, and load:
```bash
launchctl load ~/Library/LaunchAgents/com.openclaw.[name].plist
```

**Option C — crontab:**
```bash
crontab -e
# Add the cron expression + openclaw command
```

---

## Template 1: Morning Brief

**Schedule:** Weekdays at 7:00 AM
**Expression:** `0 7 * * 1-5`

**Message Prompt:**
```
Morning brief. Read yesterday's memory log. Deliver:
1. Top 3 priorities for today
2. Any open loops from yesterday
3. Calendar check: any meetings or deadlines today?
4. One quick win (completable in 30 min)
Keep it under 150 words. Lead with the most important thing.
```

**LaunchAgent:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.openclaw.morning-brief</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Morning brief. Read yesterday's memory log. Top 3 priorities, open loops, calendar, one quick win. Under 150 words.</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict><key>Hour</key><integer>7</integer><key>Minute</key><integer>0</integer></dict>
    <key>StandardOutPath</key><string>/tmp/morning-brief.log</string>
</dict>
</plist>
```

---

## Template 2: End-of-Day Summary

**Schedule:** Weekdays at 5:30 PM
**Expression:** `30 17 * * 1-5`

**Message Prompt:**
```
End of day summary. Log to memory/[TODAY].md:
1. What I accomplished today
2. What's blocked or unfinished
3. Key decisions made
4. Top 3 priorities for tomorrow

Then send me a 3-bullet Telegram: done / blocked / tomorrow.
```

---

## Template 3: Weekly Review

**Schedule:** Sundays at 8:00 PM
**Expression:** `0 20 * * 0`

**Message Prompt:**
```
Weekly review. Read memory files from the past 7 days.

Produce:
1. Wins — what actually got done
2. Stalls — what didn't move and why
3. Patterns — recurring blockers or themes
4. Decisions made this week
5. Next week focus — top 3 priorities
6. One experiment — something to try differently

Write to memory/weekly-[DATE].md. Send Telegram summary.
```

---

## Template 4: Research Brief

**Schedule:** Daily at 6:30 AM
**Expression:** `30 6 * * *`

**Message Prompt:**
```
Daily research brief. Check research/topics.md for configured topics.

For each primary topic: search for news from last 24 hours.
Deliver:
- Headline: most important development
- By topic: 1-2 sentence summary per topic with source links
- Signal/Noise: one thing worth going deeper vs. one thing that's hype

Write to research/daily/[DATE].md. Send Telegram summary under 200 words.
```

---

## Template 5: Email Check

**Schedule:** Every 8 hours at 8am, 1pm, 5pm
**Expressions:** `0 8,13,17 * * *`

**Message Prompt:**
```
Email check. Use himalaya to read inbox (last 20 messages).

Categorize: URGENT (reply today) / IMPORTANT (reply 48h) / FYI (no reply) / SPAM (delete).

For URGENT: draft responses using email/templates.md.
For IMPORTANT: flag with summary.

Send Telegram: X urgent drafted / X important flagged / X archived.
```

---

## Template 6: YouTube Monitor

**Schedule:** Every 6 hours
**Expression:** `0 */6 * * *`

**Message Prompt:**
```
Check YouTube channels in research/youtube-channels.md.

For Daily Check channels: find videos published in last 48 hours.
Check against trigger keywords.

If matches: title, URL, date, 2-sentence summary.
If nothing new: say so — no padding.
```

---

## Template 7: Gateway Heartbeat

**Schedule:** Every 5 minutes
**Expression:** `*/5 * * * *`

**Message Prompt:** *(not an LLM call — use the watchdog script)*

**Script path:** `scripts/gateway-watchdog.sh`

**LaunchAgent:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.openclaw.watchdog</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USER/.openclaw/workspace/scripts/gateway-watchdog.sh</string>
    </array>
    <key>StartInterval</key><integer>300</integer>
    <key>StandardOutPath</key><string>/tmp/watchdog.log</string>
</dict>
</plist>
```

---

## Template 8: Financial Morning Brief

**Schedule:** Weekdays at 6:00 AM
**Expression:** `0 6 * * 1-5`

**Message Prompt:**
```
Financial morning brief. Read finance/watchlist.md and finance/sectors.md.

Pre-market:
1. Macro pulse (2-3 sentences: futures, news, VIX)
2. Watchlist check (price + any news per ticker)
3. Today's key events (earnings, data releases)

Under 300 words. Fast — this is pre-market.
```

---

## Template 9: Weekly Competitive Digest

**Schedule:** Sundays at 7:00 PM
**Expression:** `0 19 * * 0`

**Message Prompt:**
```
Weekly competitive digest. Read all intel files from past 7 days.

Compile:
1. Biggest move (single most notable competitor development)
2. YouTube roundup (view counts included)
3. Social signals (top themes)
4. Product updates (launches, pricing changes)
5. Opportunity flags (based on competitor gaps)
6. Recommended actions (1-3 specific things to do)

Write to intel/weekly/[DATE].md. Send full digest.
```

---

## Template 10: Brain Backup

**Schedule:** Nightly at 11:55 PM
**Expression:** `55 23 * * *`

**Script:** *(runs shell script, not LLM)*

```bash
#!/bin/bash
# daily-brain-backup.sh
cd ~/.openclaw/workspace
git add -A
git commit -m "Brain backup: $(date '+%Y-%m-%d %H:%M')"
git push origin main 2>&1
echo "Backup complete: $(date)"
```

**LaunchAgent:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.openclaw.brain-backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USER/.openclaw/workspace/scripts/daily-brain-backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict><key>Hour</key><integer>23</integer><key>Minute</key><integer>55</integer></dict>
    <key>StandardOutPath</key><string>/tmp/brain-backup.log</string>
</dict>
</plist>
```

---

## Template 11: Self-Improvement Review

**Schedule:** Monthly, 1st day at 9:00 AM
**Expression:** `0 9 1 * *`

**Message Prompt:**
```
Monthly self-improvement review. Read memory files from the past month.

Analyze:
1. What workflows are working well?
2. What's slowing things down?
3. What am I (the agent) getting wrong repeatedly?
4. What should we automate that we're still doing manually?
5. What context would make me more effective?

Propose 3 improvements. Write to memory/monthly/[MONTH].md.
```

---

## Template 12: n8n Health Check

**Schedule:** Daily at 8:00 AM
**Expression:** `0 8 * * *`

**Message Prompt:**
```
n8n health check. Fetch last 50 executions from n8n API (localhost:5678).

Count: total / success / failed / waiting.
If failure rate >20%: list failing workflows and errors.
Check if any workflows haven't run in 24h when they should have.

Report: HEALTHY / DEGRADED / CRITICAL
If degraded/critical: list workflows to investigate.
Send Telegram alert only if DEGRADED or CRITICAL.
```

---

## Template 13: Content Pipeline Check

**Schedule:** Weekdays at 9:00 AM
**Expression:** `0 9 * * 1-5`

**Message Prompt:**
```
Content pipeline check. Read content/calendar.md.

1. What's scheduled for today?
2. What's in progress and where is it?
3. Any content due this week that isn't drafted yet?
4. Suggest 1 quick piece based on recent research (research/daily/)

Flag: anything at risk of missing schedule.
```

---

## Template 14: Quarterly Tax Reminder

**Schedule:** First day of each quarter at 8:00 AM

Add 4 entries to crontab:
```
0 8 1 1 * openclaw cron --message "Q4 estimated tax payment due Jan 15. Pull revenue from freelance/ and calculate what's owed."
0 8 1 4 * openclaw cron --message "Q1 estimated tax payment due Apr 15. Pull revenue from freelance/ and calculate what's owed."
0 8 1 7 * openclaw cron --message "Q2 estimated tax payment due Jun 15. Pull revenue from freelance/ and calculate what's owed."
0 8 1 10 * openclaw cron --message "Q3 estimated tax payment due Sep 15. Pull revenue from freelance/ and calculate what's owed."
```

---

## Template 15: New Week Planning

**Schedule:** Mondays at 7:30 AM
**Expression:** `30 7 * * 1`

**Message Prompt:**
```
New week planning. Read:
- memory/weekly-[LAST WEEK].md (last week's review)
- Any upcoming calendar events this week
- content/calendar.md (what's due)

Build this week's plan:
1. Top 3 goals for the week (not tasks — outcomes)
2. Key tasks by day (Mon-Fri rough allocation)
3. Any blockers to clear Monday morning
4. One thing to move out of the way early

Write to memory/weekly-plan-[DATE].md. Send via Telegram.
```

---

## Cron Expression Quick Reference

```
*  *  *  *  *
┬  ┬  ┬  ┬  ┬
│  │  │  │  └── Day of week (0=Sun, 1=Mon...6=Sat)
│  │  │  └───── Month (1-12)
│  │  └──────── Day of month (1-31)
│  └─────────── Hour (0-23)
└────────────── Minute (0-59)
```

Common patterns:
- `0 7 * * 1-5` — Weekdays at 7am
- `0 */6 * * *` — Every 6 hours
- `*/5 * * * *` — Every 5 minutes
- `0 9 * * 1` — Mondays at 9am
- `0 20 * * 0` — Sundays at 8pm
- `0 8 1 * *` — 1st of every month at 8am
- `55 23 * * *` — Every night at 11:55pm

---

## Troubleshooting

**LaunchAgent not running**
- Validate plist: `plutil -lint ~/Library/LaunchAgents/[name].plist`
- Check it's loaded: `launchctl list | grep openclaw`
- Check error log: `cat /tmp/[name].err`
- Reload: `launchctl unload [plist] && launchctl load [plist]`

**Cron runs but does nothing**
- Test the command manually first
- Check that openclaw is in the PATH in the plist EnvironmentVariables
- Some LaunchAgents need full paths: `/opt/homebrew/bin/openclaw`

**LLM cron sends at wrong time**
- LaunchAgent times are local timezone
- Verify system timezone: `sudo systemsetup -gettimezone`

---

*Built on OpenClaw. macOS LaunchAgents require OpenClaw installed at /usr/local/bin/openclaw or equivalent.*
