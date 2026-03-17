# Voice Journal System — Talk to Your Agent, Let It Organize

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Send voice messages to your AI agent. It transcribes, extracts action items, logs to daily memory, and files ideas by topic. Think out loud — let the agent handle the rest.

---

## What's In This Package

- `voice-journal.sh` script docs
- OpenClaw transcription config
- Action item extraction prompt
- Daily memory logging
- Topic classification system
- Idea filing workflow
- Weekly voice journal digest

---

## How It Works

1. Record a voice note (Telegram voice message, iPhone Voice Memos, etc.)
2. Your agent transcribes it (via OpenClaw's audio processing or Whisper API)
3. Agent extracts: action items, ideas, decisions, questions
4. Files to the right place in your workspace
5. Daily digest surfaces the week's voice notes in organized form

---

## OpenClaw Transcription Config

Enable audio transcription in your OpenClaw config:

```json
{
  "audio": {
    "transcription": {
      "enabled": true,
      "echoTranscript": true,
      "provider": "whisper",
      "model": "whisper-1"
    }
  }
}
```

With `echoTranscript: true`, your agent will automatically process voice messages you send via Telegram.

### Whisper API Setup

If using OpenAI Whisper directly:

```bash
openclaw secret set OPENAI_API_KEY "your-openai-key"
```

Or use the OpenAI Whisper API directly:

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F file="@/path/to/audio.mp3" \
  -F model="whisper-1"
```

---

## voice-journal.sh

The core script that processes incoming audio files:

```bash
#!/bin/bash
# voice-journal.sh — process voice notes and file to workspace

set -e

AUDIO_FILE="$1"
WORKSPACE="${HOME}/.openclaw/workspace"
DATE=$(date '+%Y-%m-%d')
TIME=$(date '+%H:%M')

if [ -z "$AUDIO_FILE" ]; then
  echo "Usage: voice-journal.sh <audio-file>"
  exit 1
fi

echo "Processing voice note: $AUDIO_FILE"

# Step 1: Transcribe
TRANSCRIPT=$(curl -s https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F "file=@$AUDIO_FILE" \
  -F "model=whisper-1" | jq -r '.text')

if [ -z "$TRANSCRIPT" ]; then
  echo "Transcription failed"
  exit 1
fi

echo "Transcript: $TRANSCRIPT"

# Step 2: Save raw transcript
TRANSCRIPTS_DIR="$WORKSPACE/voice/transcripts"
mkdir -p "$TRANSCRIPTS_DIR"
echo "[$DATE $TIME] $TRANSCRIPT" >> "$TRANSCRIPTS_DIR/$DATE.md"

echo "Filed transcript to $TRANSCRIPTS_DIR/$DATE.md"

# Step 3: Process with agent
# The agent prompt below handles the intelligent filing
echo "Transcript saved. Now run processing prompt in OpenClaw."
```

Make executable:
```bash
chmod +x ~/.openclaw/workspace/scripts/voice-journal.sh
```

---

## Processing Prompt

After transcription, run this to extract and file:

```
Process this voice note transcript:

"[TRANSCRIPT]"

Extract and categorize:

1. **Action Items** — things to do (format: "[ ] [action]")
2. **Ideas** — concepts to explore, potential projects
3. **Decisions** — anything you decided during the note
4. **Questions** — things you need to figure out
5. **Connections** — links to other projects, people, or concepts

Then file to the right places:
- Action items → append to memory/[TODAY].md under "### Action Items"
- Ideas → append to ideas.md with date
- Decisions → append to memory/[TODAY].md under "### Decisions"
- Questions → append to questions.md with date

Summary: what was in this voice note? (2-3 sentences for the daily log)
```

---

## Action Item Extraction

For a fast action item pull:

```
Extract action items only from this transcript:

"[TRANSCRIPT]"

Rules:
- Only include explicit commitments or things I said I'd do
- Infer implied actions ("I need to think about X" → "[ ] Think through [X]")
- Include context: who, by when (if mentioned)
- Priority: flag anything with a deadline as URGENT

Output as a simple checkbox list:
- [ ] [Action] (context: [who/when if mentioned])
```

---

## Daily Memory Logging

Format for voice notes in daily memory files:

Create `memory/YYYY-MM-DD.md` (or append to existing):

```markdown
## Voice Notes — [DATE]

### [TIME] — Voice Note
**Transcript:** [Full transcript or summary]

**Action Items:**
- [ ] [Action 1]
- [ ] [Action 2]

**Ideas Filed:**
- [Idea title] → ideas.md

**Decisions:**
- [Decision made]

**Questions:**
- [Question to explore]
```

### Auto-Logging Agent Prompt

Configure your agent to handle this automatically when a voice message comes in via Telegram:

```
Voice message received. Auto-process per voice journal protocol:

1. Transcription complete (provided above)
2. Extract action items, ideas, decisions, questions
3. Append to memory/[TODAY].md
4. File ideas to ideas.md
5. Confirm: "[N] action items captured, [N] ideas filed"
```

---

## Topic Classification System

Create `voice/topics.md` to configure how ideas get filed:

```markdown
# Voice Note Topics

## Active Projects
- project-alpha → voice/projects/alpha.md
- product-ideas → voice/product-ideas.md
- [project name] → [file path]

## Ongoing Themes
- [topic] → [file path]
- personal → voice/personal.md
- research → research/voice-notes.md

## Keywords for Auto-Classification
If transcript contains these words, route to that file:
- "client name" → clients/[name].md
- "product" OR "feature" → voice/product-ideas.md
- "book" OR "read" → reading-list.md
```

### Classification Prompt

```
Classify this voice note and file it:

Transcript: "[TRANSCRIPT]"

Check voice/topics.md for routing rules.

1. Identify the primary topic
2. Route to the configured file
3. If no match: file to voice/unclassified/[DATE].md

Confirm where it was filed.
```

---

## Idea Filing Workflow

Create `ideas.md` in your workspace root:

```markdown
# Ideas Log

## [DATE]
### [Idea Title]
**Source:** Voice note, [TIME]
**Context:** [Where the idea came from]
**Description:** [The idea in full]
**Status:** 💡 New | 🔄 Exploring | 📌 Committed | ❌ Shelved

---
```

### Idea Extraction Prompt

```
Extract ideas from this voice transcript:

"[TRANSCRIPT]"

For each idea:
1. Give it a title (descriptive, 3-7 words)
2. Capture the full idea as described
3. Note any context or trigger mentioned
4. Assign initial status: 💡 New

Append to ideas.md in standard format.

Also check: does this connect to any existing idea in ideas.md? Note the connection.
```

---

## Weekly Voice Journal Digest

Run every Monday at 8 AM:

```
Weekly voice journal digest. Read voice/transcripts/*.md from last 7 days.

Also read memory files from last 7 days for voice note sections.

Compile:
1. **Action items captured** — total, completed, outstanding
2. **Ideas logged** — new ideas from this week
3. **Decisions made** — via voice (often more casual than written)
4. **Themes** — what topics came up repeatedly?
5. **One best idea** — the single most interesting thing from this week's voice notes

This is your voice-sourced weekly review. What did your off-the-cuff thinking produce?

Write to voice/weekly-[DATE].md
Send Telegram summary
```

---

## Advanced: Telegram Voice Note Auto-Processing

Configure OpenClaw to handle Telegram voice messages automatically:

In `AGENTS.md`, add:
```markdown
## Voice Note Protocol

When a voice/audio message arrives via Telegram:
1. Transcribe immediately using Whisper
2. Echo transcript back: "🎙️ Transcript: [text]"
3. Run extraction: action items, ideas, decisions, questions
4. File to today's memory log
5. Confirm: "✅ Captured [N] action items + [N] ideas"

Do this automatically — no confirmation needed.
```

---

## Recording Tips

**For best transcription quality:**
- Speak clearly, at normal pace (not too fast)
- Minimize background noise
- Say "action item" or "idea" explicitly — makes extraction more accurate
- Start each thought as a new sentence, not mid-ramble

**Good patterns:**
- "Action item: email John about the proposal"
- "Idea: what if we built a dashboard for X"
- "I decided to go with option B"
- "Question I need to answer: how does Y work?"

**Bad patterns:**
- Mumbling / very low volume
- Multiple overlapping thoughts without pausing
- Recording in noisy environments (meetings, streets)

---

## Troubleshooting

**Transcription quality is poor**
- Check audio quality first (listen back yourself)
- Try a different Whisper model: `whisper-1` vs. upgraded models
- Add language hint: `-F "language=en"`

**Agent isn't auto-processing voice notes**
- Check that echoTranscript is `true` in OpenClaw config
- Verify the voice note protocol is in your AGENTS.md
- Test by sending a voice message and checking for auto-response

**Ideas going to wrong files**
- Check voice/topics.md keyword mappings
- Make keywords more specific to avoid false matches
- Add explicit routing: "for topic X, always use Y"

**Action items not being completed**
- Review outstanding items weekly via the digest
- Connect to your task management: "Review voice/transcripts/*.md action items"
- The voice capture is only half — the other half is your review habit

---

*Built on OpenClaw. Requires OpenClaw with audio enabled and optional OpenAI Whisper API key.*
