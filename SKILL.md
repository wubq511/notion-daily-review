---
name: notion-daily-review
description: "Daily Notion knowledge review. Picks 2-3 knowledge points from user's Notion workspace, generates review summaries and deep-dive discussions. Trigger words: daily review, knowledge review, 每日回顾, 知识回顾, notion review, 今日知识"
---

# Notion Daily Knowledge Review

Intelligently selects 2-3 knowledge points from the user's Notion workspace, generates review summaries and deep-dive discussions for second-round learning and deeper understanding.

---

## Prerequisites

This skill requires a **Notion MCP Server** to be connected. The recommended server is [makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server).

### Notion Integration Token Setup

1. Go to https://www.notion.so/my-integrations
2. Click **"+ New integration"**
3. Name it (e.g. "Cowork"), select your workspace, click **Submit**
4. Copy the **Internal Integration Secret** (starts with `ntn_`)
5. In each Notion page you want this skill to access: click **"..."** → **"Connect to"** → select your integration
6. Provide the token when the skill asks for it during initialization

> **Important**: You must connect the integration to your top-level pages. Child pages inherit the connection automatically.

---

## Configuration

All configuration is stored in the memory system at `C:\Users\h\AppData\Roaming\Claude-3p\local-agent-mode-sessions\41d16b5b-9b59-4a80-9410-c824e988ec92\00000000-0000-4000-8000-000000000001\spaces\3cb39822-49ab-425e-bd2a-14960cb69d97\memory\` as a file named `notion_daily_review_config.md`.

Configuration fields:

```yaml
notion_token: "<PENDING_INIT>"         # ntn_xxx integration token
review_database_id: "<PENDING_INIT>"   # created during init
review_database_title: "📝 Daily Knowledge Review"
knowledge_root_pages: []               # list of {id, name} objects
excluded_pages: []                     # page names to skip
style_preset: "<PENDING_INIT>"         # concise_deep | academic | casual | socratic
language: "<PENDING_INIT>"            # e.g. zh-CN, en
last_reviewed_pages: []                # track recent reviews to avoid repeats
```

---

## First-Run Initialization

**When to run**: Execute if `notion_token` is `<PENDING_INIT>` or the config file does not exist.

### Phase 0a: Check Notion MCP Connection

Check if Notion MCP tools are available by attempting a search:

```
mcp__notion-mcp-server__search(query="", page_size=1)
```

If this tool is not found, inform the user:

> "This skill requires the Notion MCP Server. Let me help you set it up.
> Please install it from: https://github.com/makenotion/notion-mcp-server
> Then provide your Notion Integration Token."

If the tool exists but returns an auth error, ask for the token.

### Phase 0b: Discover Knowledge Pages

**Step 1**: Get workspace pages:

```
mcp__notion-mcp-server__search(query="", page_size=100)
```

**Step 2**: For each candidate root page, inspect children:

```
mcp__notion-mcp-server__retrieve-block-children(block_id="<root_page_id>", page_size=100)
```

**Step 3**: Classify pages into:
- **Knowledge pages**: Notes, articles, book reviews, concepts, research
- **Non-knowledge pages**: Planners, trackers, to-do lists, calendars, dashboards

**Step 4**: Present classification to user and ask for confirmation.

**Step 5**: Record `knowledge_root_pages` and `excluded_pages`.

### Phase 0c: Style Preferences

Present the following options:

> "How would you like your daily reviews? Pick a style:
>
> **A. Concise & Deep** (简练有深度)
> Core summary in 2-3 sentences + extension/new perspectives. No fluff.
>
> **B. Academic** (学术探究)
> Structured analysis with frameworks, theories, cross-disciplinary connections.
>
> **C. Casual** (轻松对话)
> Friendly, conversational. Like chatting with a smart friend.
>
> **D. Socratic** (苏格拉底式)
> Heavy on thought-provoking questions. Maximum thinking stimulation.
>
> Also, what language? (e.g. Chinese/中文, English)"

### Phase 0d: Create Review Database

```
mcp__notion-mcp-server__create-database(
  parent={"type": "page_id", "page_id": "<parent_page_id>"},
  title=[{"type": "text", "text": {"content": "📝 Daily Knowledge Review"}}],
  properties={
    "Name": {"title": {}},
    "Date": {"date": {}},
    "Source Sections": {"multi_select": {"options": []}},
    "Trigger": {
      "select": {
        "options": [
          {"name": "Recent Edit", "color": "blue"},
          {"name": "Conversation", "color": "green"},
          {"name": "Random", "color": "gray"}
        ]
      }
    }
  }
)
```

Record the returned `review_database_id`.

### Phase 0e: Save Configuration

Save the config to memory:

```python
config = {
    "notion_token": "<token>",
    "review_database_id": "<id>",
    "review_database_title": "📝 Daily Knowledge Review",
    "knowledge_root_pages": [{"id": "...", "name": "..."}],
    "excluded_pages": ["..."],
    "style_preset": "concise_deep",
    "language": "zh-CN",
    "last_reviewed_pages": []
}
# Write to memory/notion_daily_review_config.md
```

Inform user:

> "✅ Initialization complete!
> - Monitoring **X** knowledge sections
> - Review style: **[style]**
> - Say '每日回顾' or 'daily review' to trigger, or set up a cron job."

---

## Execution Flow

**Prerequisite**: Config must be initialized (not `<PENDING_INIT>`).

### Phase 1: Collect Context Signals

#### 1a. Get Recently Edited Pages

```
mcp__notion-mcp-server__search(query="", page_size=20)
```

Extract pages edited within the last 3 days (compare `last_edited_time` to today).

#### 1b. Check Memory for Recent Topics

Read the memory index (`MEMORY.md`) to find recent conversation topics. Extract key words as context signals.

---

### Phase 2: Select Knowledge Points

#### Dynamic Page Discovery

For each entry in `knowledge_root_pages`:

```
mcp__notion-mcp-server__retrieve-block-children(block_id="<root_page_id>", page_size=100)
```

Collect all `child_page` and `child_database` blocks. Filter out excluded pages and the review database.

#### Selection Strategy

Pick **2-3 knowledge points**:

1. **Context-related (1-2)**: Search using context signals:

```
mcp__notion-mcp-server__search(query="<keywords>", page_size=10)
```

Exclude non-knowledge pages, review database, and recently reviewed pages.

2. **Random (1)**: Pick one from the knowledge list deterministically:
   - Use `date +%j` (day of year) modulo total knowledge sections count
   - This gives varied but reproducible daily picks

#### Read Content

For each selected page:

```
mcp__notion-mcp-server__retrieve-block-children(block_id="<page_id>", page_size=100)
```

For database-type sections, use:

```
mcp__notion-mcp-server__query-database(database_id="<db_id>", page_size=20)
```

Extract text from all block types.

---

### Phase 3: Generate Review Content

#### Style Templates

**concise_deep**:
- Core summary: 2-3 sentences, no fluff
- Deep-dive: 150-250 words. New angles, cross-domain connections, 1-2 specific thought-provoking questions

**academic**:
- Core summary: Structured with key concepts
- Deep-dive: 200-300 words. Frameworks/theories, cross-disciplinary analysis, further reading

**casual**:
- Core summary: Conversational retelling
- Deep-dive: 150-250 words. Relatable examples, engaging tone

**socratic**:
- Core summary: Brief, 1-2 sentences
- Deep-dive: 200-300 words. Layered questions building to one challenging question

#### Output Format

```markdown
## 🧠 Daily Knowledge Review — YYYY-MM-DD

Today's X knowledge points | [Why these were chosen]

---

### 📌 [Knowledge Point Title]
> Source: [Section] · Trigger: [Recent Edit / Conversation / Random]

**Core Review**
[Summary]

**Deep Dive**
[Extended discussion]

---
```

#### Writing Guidelines
- Use the configured `language`
- Include cross-references to other Notion pages when natural
- Questions should be specific and challenging
- Avoid verbatim repetition — add synthesis value
- Total output should not exceed 800 words

---

### Phase 4: Output

#### 4a. Write to Notion

```
mcp__notion-mcp-server__create-page(
  parent={"type": "database_id", "database_id": "<review_database_id>"},
  properties={
    "Name": {"title": [{"text": {"content": "YYYY-MM-DD Daily Knowledge Review"}}]},
    "Date": {"date": {"start": "YYYY-MM-DD"}},
    "Source Sections": {"multi_select": [{"name": "<section>"}, ...]},
    "Trigger": {"select": {"name": "<primary_trigger>"}}
  },
  children=[
    // heading_2, callout, heading_3, paragraph, divider blocks
  ]
)
```

**Block structure per knowledge point**:

```json
[
  {"object":"block","type":"heading_2","heading_2":{"rich_text":[{"type":"text","text":{"content":"📌 Title"}}]}},
  {"object":"block","type":"callout","callout":{"rich_text":[{"type":"text","text":{"content":"Source: ... · Trigger: ..."}}],"icon":{"emoji":"📍"}}},
  {"object":"block","type":"heading_3","heading_3":{"rich_text":[{"type":"text","text":{"content":"Core Review"}}]}},
  {"object":"block","type":"paragraph","paragraph":{"rich_text":[{"type":"text","text":{"content":"..."}}]}},
  {"object":"block","type":"heading_3","heading_3":{"rich_text":[{"type":"text","text":{"content":"Deep Dive"}}]}},
  {"object":"block","type":"paragraph","paragraph":{"rich_text":[{"type":"text","text":{"content":"..."}}]}},
  {"object":"block","type":"divider","divider":{}}
]
```

#### 4b. Update Review History

After successful write, update `last_reviewed_pages` in config (keep last 7 days, remove older entries).

#### 4c. Send in Chat

Output the formatted review directly in the reply.

---

## Important Notes

1. **Exclude non-knowledge pages**: Skip planners, trackers, to-do lists, daily logs, review database
2. **Avoid repeats**: Check `last_reviewed_pages` in config; skip pages reviewed in last 7 days
3. **Handle empty content**: If a page has < 50 characters, skip and pick another
4. **Database-type sections**: Use `query-database` instead of `retrieve-block-children`
5. **Cross-references**: Connect knowledge points to each other naturally
6. **Error handling**: If an API call fails, retry once after 2 seconds. If still failing, skip that step
7. **Total length**: Keep review output under 800 words across all knowledge points

---

## Trigger Phrases

Any of these activate this skill:
- "每日回顾" / "知识回顾" / "今天回顾什么"
- "帮我回顾 Notion 知识"
- "daily review" / "notion review" / "knowledge review"
- Cron/scheduled task automatic trigger
