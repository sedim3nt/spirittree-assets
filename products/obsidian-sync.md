# Obsidian Knowledge Sync — AI Agent ↔ Second Brain

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Bridge your Obsidian vault and your AI agent. Your agent reads notes, writes meeting notes, creates daily journals, and maintains your knowledge graph. Your second brain becomes AI-accessible.

---

## What's In This Package

- Obsidian skill configuration for OpenClaw
- Vault path setup
- Daily journal template
- Meeting notes automation
- Knowledge graph navigation
- Tag-based retrieval
- Link creation patterns
- Weekly review automation

---

## Setup

### Install obsidian-cli

```bash
npm install -g obsidian-cli
# or
npx obsidian-cli --help
```

Verify:
```bash
obsidian-cli list-vaults
```

### Configure OpenClaw Obsidian Skill

In your OpenClaw skill config, add:

```json
{
  "skills": {
    "obsidian": {
      "vaultPath": "/Users/YOUR_USERNAME/path/to/your/vault",
      "defaultFolder": "",
      "dailyNotesFolder": "Daily",
      "templateFolder": "Templates"
    }
  }
}
```

Or set environment:
```bash
export OBSIDIAN_VAULT_PATH="/Users/yourname/Documents/MyVault"
```

### Find Your Vault Path

- macOS: usually `~/Documents/[VaultName]` or `~/[VaultName]`
- Open Obsidian → Settings → About → Your vault is listed at the bottom
- Or: `ls ~/Documents | grep -i vault` or `ls ~/` for vault folders

---

## Basic Operations

### Reading Notes

Agent prompt to read an Obsidian note:

```
Read my Obsidian note: [NOTE TITLE or PATH]

The vault is at $OBSIDIAN_VAULT_PATH.
The note file is at [VAULT PATH]/[folder/][NOTE].md

Read it and summarize the key points.
```

Using obsidian-cli:

```bash
# Read a note
obsidian-cli read "Note Title"

# Search for notes
obsidian-cli search "keyword"

# List notes in a folder
obsidian-cli list "FolderName/"
```

### Writing Notes

```bash
# Create a new note
obsidian-cli create "Note Title" "Note content here"

# Append to existing note
obsidian-cli append "Note Title" "Additional content"

# Update a note
obsidian-cli update "Note Title" "New content"
```

---

## Daily Journal Template

### Template File

Create `Templates/Daily Note.md` in your vault:

```markdown
---
date: {{date}}
tags: daily-note
---

# {{date}} — Daily Note

## Morning

### Priorities Today
1. 
2. 
3. 

### Intentions
*How do I want to show up today?*

---

## During the Day

### Work Log
- 

### Ideas
- 

### Connections Made
*Links to other notes, concepts*
- 

---

## Evening

### What Happened
*3-5 bullet summary*

### Wins
- 

### What I'll Do Differently
- 

### Tomorrow's Top Priority
- 

---

## Links
[[Relevant note 1]] | [[Relevant note 2]]
```

### Daily Journal Agent Prompt

Configure as morning cron or run on demand:

```
Create today's Obsidian daily note.

Date: [TODAY]
Template: read Templates/Daily Note.md from vault

Pre-fill based on context:
- Priorities: pull from memory/[TODAY].md if it exists
- Any carryover action items from yesterday's daily note

Create at: Daily/[YYYY-MM-DD].md

After creating: link to any existing notes about today's priorities or projects.
Return the file path.
```

---

## Meeting Notes Automation

### Meeting Notes Template

Create `Templates/Meeting Notes.md` in your vault:

```markdown
---
date: {{date}}
tags: meeting
attendees: 
project: 
---

# Meeting: {{title}}

**Date:** {{date}}
**Attendees:** 
**Duration:** 

## Agenda
- 

## Discussion Notes

### [Topic 1]


### [Topic 2]


## Decisions Made
- 

## Action Items
- [ ] [Action] — [Owner] — [Due Date]
- [ ] [Action] — [Owner] — [Due Date]

## Follow-Up
- 

## Links
[[Related Project]] | [[Previous Meeting Notes]]
```

### Meeting Notes Agent Prompt

```
Create meeting notes in Obsidian.

Meeting: [MEETING NAME]
Date: [TODAY]
Attendees: [LIST]
Duration: [LENGTH]

Notes from meeting:
[DESCRIBE OR PASTE YOUR NOTES]

Create at: Meetings/[YYYY-MM-DD] [Meeting Name].md

After creating:
1. Link to any existing project or person notes
2. Add action items to the attendees' People notes if they exist
3. Link this meeting in the related project note

Return the file path.
```

---

## Knowledge Graph Navigation

### Find Connections

```
Explore connections in my Obsidian vault for the concept: [TOPIC]

Search for:
1. Notes with title containing "[TOPIC]"
2. Notes with tags: [relevant tags]
3. Notes that link to a central [TOPIC] note
4. Notes in the [relevant folder]

Map what I have:
- Core note on this topic: [path if exists]
- Related notes: [list]
- Notes that reference this: [list from backlinks if available]
- Gaps: what am I missing in this topic cluster?

Don't create notes yet — just show me what's here.
```

### Create Linked Notes

```
Create a note on [TOPIC] in Obsidian.

Before creating:
1. Search for existing related notes
2. Note what exists to avoid duplication

Create at: [Appropriate folder]/[TOPIC].md

Content:
- Overview section
- Key concepts/sub-topics (as links to create or existing notes)
- Related: [[link to related note 1]], [[link to related note 2]]
- Sources: [any sources to cite]

After creating: link this note FROM any existing notes that should reference it.
```

---

## Tag-Based Retrieval

### Tag Organization

Recommended tag structure for AI-accessible vaults:

```
#status/active     — actively working on
#status/reference  — reference material
#status/archive    — no longer active

#type/meeting      — meeting notes
#type/project      — project notes
#type/person       — people/contacts
#type/idea         — ideas and concepts
#type/learning     — learning notes

#domain/[your domains] — content-specific tags
```

### Tag Search Prompt

```
Search my Obsidian vault for notes tagged: [TAG]

List all notes with this tag and give a 1-sentence summary of each.
Then: what patterns do I see across these notes?
```

---

## Weekly Review Automation

### Weekly Review Prompt

```
Obsidian weekly review. Vault path: $OBSIDIAN_VAULT_PATH

Read:
1. This week's daily notes: Daily/[MON].md through Daily/[SUN].md
2. Any new notes created this week (list by modification date)

Weekly review output:
1. **What I captured** — notable notes, ideas, connections made
2. **Action items** — extract all unchecked action items from this week's notes
3. **Ideas to develop** — any #type/idea notes created this week
4. **Connections to make** — notes that should be linked but aren't
5. **Archive candidates** — notes that are done/stale

Write summary to Daily/[DATE]-weekly-review.md
```

### Stale Note Detection

```
Find notes in my Obsidian vault that may be stale.

Criteria:
- Notes in "Active Projects" folder with no modification in 30+ days
- Notes tagged #status/active but content seems complete
- Meeting notes without resolved action items

For each stale note:
1. Note path and last modified date
2. Recommend: ARCHIVE / UPDATE / DELETE / KEEP

Present as a list for my review — don't make changes without confirmation.
```

---

## Link Creation Patterns

### Backlink Audit

```
Orphan note audit in Obsidian.

An orphan note has no inbound links from other notes.

List all notes in the vault with zero backlinks.

For each orphan:
- Note title
- Folder location
- Suggested notes to link FROM (based on content similarity)

Present the top 10 most valuable connection opportunities.
```

### Auto-Link Creation Prompt

```
After reviewing the orphan list, create these links:

For note [NOTE A], add a link in [NOTE B]:
- Find the relevant section in NOTE B
- Add [[NOTE A]] where it naturally fits
- Don't force it — only if it genuinely belongs

Do this for the top 5 connections I approved.
Report what was linked where.
```

---

## Troubleshooting

**obsidian-cli can't find vault**
- Verify the path: `ls "$OBSIDIAN_VAULT_PATH"`
- Check permissions: `ls -la "$OBSIDIAN_VAULT_PATH"`
- Ensure no spaces in path (or quote the variable)

**Notes not showing links in Obsidian after agent creates them**
- Links use `[[Note Title]]` format — exact match matters
- Case-sensitive: `[[My Note]]` ≠ `[[my note]]`
- Obsidian may need a vault reload after changes: Cmd+P → "Reload vault"

**Daily notes going to wrong folder**
- Check your Daily notes folder setting in Obsidian: Settings → Core Plugins → Daily Notes
- Ensure agent is writing to the same folder
- Convention: match exactly what Obsidian expects

**Can't find notes by search**
- Obsidian search is case-insensitive but obsidian-cli may differ
- Try file system search: `find "$OBSIDIAN_VAULT_PATH" -name "*keyword*"`
- Or: `grep -r "keyword" "$OBSIDIAN_VAULT_PATH" --include="*.md" -l`

---

*Built on OpenClaw and obsidian-cli. Requires Obsidian vault installed locally.*
