# Notion Integration Kit — AI Agent ↔ Notion Bridge

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 30 minutes

Connect your AI agent to Notion. Create pages, update databases, manage projects, sync meeting notes. Your agent reads and writes to your Notion workspace.

---

## What's In This Package

- Notion API setup guide
- Database CRUD operations
- Page creation templates
- Project tracker sync
- Meeting notes automation
- Reading list manager
- Daily journal template

---

## Setup

### Get Your API Key

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click "New Integration"
3. Give it a name (e.g., "OpenClaw Agent")
4. Select your workspace
5. Copy the Internal Integration Token

```bash
openclaw secret set NOTION_TOKEN "secret_your_token_here"
```

### Connect Integration to Pages

Notion requires you to explicitly connect your integration to each database/page:

1. Open a Notion database or page
2. Click "..." in top right → "Connections"
3. Find and add your integration

### Get Database IDs

From any Notion database URL:
`https://notion.so/workspace/[DATABASE-ID]?v=...`

The DATABASE-ID is the UUID before the `?`. Store the ones you'll use:

```bash
openclaw secret set NOTION_PROJECTS_DB "your-database-id"
openclaw secret set NOTION_TASKS_DB "your-tasks-database-id"
openclaw secret set NOTION_NOTES_DB "your-notes-database-id"
```

---

## API Basics

### Test Your Connection

```bash
NOTION_TOKEN="secret_your_token"

curl -s "https://api.notion.com/v1/users/me" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" | jq '.name'
```

### Common Headers

Every request needs:
```
Authorization: Bearer secret_your_token
Notion-Version: 2022-06-28
Content-Type: application/json
```

---

## Database CRUD Operations

### Query a Database (Read)

```bash
# Get all items in a database
curl -s -X POST "https://api.notion.com/v1/databases/$NOTION_DB_ID/query" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 50}'

# Filter: get items where Status = "In Progress"
curl -s -X POST "https://api.notion.com/v1/databases/$NOTION_DB_ID/query" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "property": "Status",
      "select": {
        "equals": "In Progress"
      }
    }
  }'
```

### Create a Database Item (Write)

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_DB_ID"},
    "properties": {
      "Name": {
        "title": [{"text": {"content": "New Task"}}]
      },
      "Status": {
        "select": {"name": "To Do"}
      },
      "Due Date": {
        "date": {"start": "2024-01-20"}
      }
    }
  }'
```

### Update a Database Item

```bash
PAGE_ID="page-id-to-update"

curl -s -X PATCH "https://api.notion.com/v1/pages/$PAGE_ID" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Status": {
        "select": {"name": "Done"}
      }
    }
  }'
```

---

## Agent Prompts for Notion

### Query Prompt

```
Read from Notion database: $NOTION_PROJECTS_DB

Fetch all items where Status = "In Progress".

For each item, get:
- Name
- Status
- Due Date
- Owner (if that property exists)

Present as a project status table. Flag any items with past due dates.
```

### Create Item Prompt

```
Create a new item in my Notion tasks database: $NOTION_TASKS_DB

Item:
- Name: [TASK NAME]
- Status: To Do
- Due Date: [DATE]
- Priority: [High/Medium/Low]
- Notes: [Any context]

Confirm the item was created and give me the Notion page URL.
```

### Update Prompt

```
Update Notion item: [ITEM NAME or PAGE-ID]

Change:
- Status: Done
- Completion Date: today

Find the item by searching for "[ITEM NAME]" in $NOTION_TASKS_DB first if I don't have the ID.
```

---

## Page Creation Templates

### Meeting Notes Page

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "NOTES_DB_ID"},
    "properties": {
      "Title": {
        "title": [{"text": {"content": "Meeting: [Meeting Name] — [DATE]"}}]
      },
      "Type": {
        "select": {"name": "Meeting"}
      },
      "Date": {
        "date": {"start": "2024-01-15"}
      }
    },
    "children": [
      {
        "object": "block",
        "type": "heading_2",
        "heading_2": {"rich_text": [{"text": {"content": "Attendees"}}]}
      },
      {
        "object": "block",
        "type": "bulleted_list_item",
        "bulleted_list_item": {"rich_text": [{"text": {"content": "[Name 1]"}}]}
      },
      {
        "object": "block",
        "type": "heading_2",
        "heading_2": {"rich_text": [{"text": {"content": "Notes"}}]}
      },
      {
        "object": "block",
        "type": "paragraph",
        "paragraph": {"rich_text": [{"text": {"content": "[Meeting notes here]"}}]}
      },
      {
        "object": "block",
        "type": "heading_2",
        "heading_2": {"rich_text": [{"text": {"content": "Action Items"}}]}
      }
    ]
  }'
```

### Meeting Notes Agent Prompt

```
After our meeting, create a Notion meeting notes page.

Meeting: [MEETING NAME]
Date: [DATE]
Attendees: [LIST]

Notes from the meeting:
[PASTE YOUR NOTES OR DESCRIBE WHAT HAPPENED]

Create the page in $NOTION_NOTES_DB with:
- Proper title: "Meeting: [Name] — [Date]"
- Attendees list
- Organized notes (use headings)
- Action items (with owner and due date for each)
- Follow-up section

Return the Notion page URL.
```

---

## Project Tracker Sync

### Sync Agent Workflow Prompt

```
Sync project status to Notion.

Current project status:
[DESCRIBE OR PASTE CURRENT STATUS]

Or: read from memory/[TODAY].md and extract project updates.

For each active project in $NOTION_PROJECTS_DB:
1. Query current status
2. Update if something changed today
3. Add a progress note to the project page

Report: which projects were updated.
```

### Weekly Project Review Prompt

```
Weekly project review in Notion.

Query $NOTION_PROJECTS_DB for all projects where Status != "Done".

For each project:
1. Fetch the project page
2. Read the last update date
3. Flag projects with no update in >7 days (stale)

Report:
- Active projects count
- Stale projects (need attention)
- Projects completed this week

Ask me for updates on stale projects before making any changes.
```

---

## Reading List Manager

### Add to Reading List

```
Add to my Notion reading list: $NOTION_READING_DB

Book/Article: [TITLE]
Author: [AUTHOR]
URL: [URL if article]
Type: [Book / Article / Paper]
Priority: [High / Medium]
Notes: [Why I want to read this]

Create the entry and return the Notion URL.
```

### Reading Tracker Prompt

```
Weekly reading digest from Notion.

Query $NOTION_READING_DB:
- Items with Status = "Reading" (currently reading)
- Items added this week (new)
- Items recently finished (Status = "Done", this week)

Report:
- What I'm currently reading
- What I finished this week
- Top 3 next reads (by priority)

Suggest: should I add any new items based on this week's research briefings?
```

---

## Daily Journal Template

### Create Daily Page

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d "{
    \"parent\": {\"database_id\": \"JOURNAL_DB_ID\"},
    \"properties\": {
      \"Title\": {
        \"title\": [{\"text\": {\"content\": \"$(date '+%Y-%m-%d') — Daily Journal\"}}]
      },
      \"Date\": {
        \"date\": {\"start\": \"$(date '+%Y-%m-%d')\"}
      }
    },
    \"children\": [
      {
        \"object\": \"block\",
        \"type\": \"callout\",
        \"callout\": {
          \"rich_text\": [{\"text\": {\"content\": \"Top 3 priorities for today\"}}],
          \"icon\": {\"emoji\": \"🎯\"}
        }
      }
    ]
  }"
```

### Daily Journal Agent Prompt

```
Create today's Notion journal entry.

Date: [TODAY]

Sections to include:
1. Top 3 Priorities
2. Morning Note [OPTIONAL — from voice notes or reflection]
3. Work Log
4. End of Day Reflection

Pre-fill with:
- Top priorities from this morning's brief (memory/[TODAY].md)
- Any action items from yesterday that carried over

Create page in $NOTION_JOURNAL_DB and return the URL.
I'll fill in the rest as the day goes.
```

---

## Troubleshooting

**401 Unauthorized**
- Check your integration token is correct
- Ensure the integration is connected to the specific database/page
- Tokens start with `secret_` — check for accidental truncation

**404 Not Found**
- Database ID is wrong — re-copy from the URL
- Page may have been deleted or moved
- Ensure hyphens are removed from UUID if using the raw ID

**Can't find my database properties**
- Fetch schema: `GET /v1/databases/[DB_ID]`
- Property names are case-sensitive (exact match required)
- Select values must match exactly what's in the Notion UI

**Writing fails but reading works**
- Your integration needs "Can edit content" permission (not just "Can read content")
- Update in Notion: Connections → your integration → Edit access

---

*Built on OpenClaw. Requires Notion account and integration setup.*
