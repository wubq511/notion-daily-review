---
name: notion-daily-review
description: "Daily Notion knowledge review. Picks 2-3 knowledge points from user's Notion workspace, generates review summaries and deep-dive discussions. Supports context-aware recommendations based on recent edits and conversation history. Trigger words: daily review, knowledge review, 每日回顾, 知识回顾, notion review, 今日知识"
---

# Notion Daily Knowledge Review

Intelligently selects 2-3 knowledge points from the user's Notion workspace, generates review summaries and deep-dive discussions for second-round learning and deeper understanding.

---

## Configuration (filled during initialization)

```yaml
# ⚠️ These values are populated automatically during first-run initialization.
# Do NOT manually edit unless you know what you're doing.

notion_user_id: "<PENDING_INIT>"
review_database_id: "<PENDING_INIT>"
review_database_name: "📝 Daily Knowledge Review"
knowledge_root_pages: []          # list of {id, name} objects
excluded_pages: []                # page names to exclude from knowledge sources
agent_memory_path: "<PENDING_INIT>"
style_preset: "<PENDING_INIT>"    # one of: concise_deep | academic | casual | socratic
language: "<PENDING_INIT>"        # e.g. zh-CN, en, ja
```

---

## First-Run Initialization

**When to run**: Execute this section if ANY of the config values above are `<PENDING_INIT>`. Once all values are filled, skip directly to the Execution Flow.

### Phase 0a: Check Notion MCP Connection

**Goal**: Ensure Notion MCP tools are available and authenticated.

**Step 1**: Check if Notion tools are accessible:

```
mcp_call({
  action: "search",
  keyword: "notion"
})
```

If Notion tools are found (e.g. `search_notion`, `get_notion_page`, etc.), proceed to Step 2.

If Notion tools are NOT found, inform the user:

> "This skill requires Notion MCP integration. Let me help you set it up."

Then attempt to search for Notion auth tools:

```
mcp_call({
  action: "search",
  keyword: "set_notion"
})
```

**Step 2**: Test if Notion is authenticated by making a simple search call:

```
mcp_call({
  action: "mcp",
  name: "search_notion",
  arguments: {
    user_id: "<user_id>",
    page_size: 1
  }
})
```

- If it succeeds → Notion is connected. Proceed to Phase 0b.
- If it fails with auth error → Guide the user through Notion Integration Token setup:

  1. Ask the user to go to https://www.notion.so/my-integrations
  2. Create a new integration (name it "Accio" or similar)
  3. Copy the Internal Integration Secret (starts with `ntn_`)
  4. Connect the integration to their Notion pages (page menu → Connect to → select integration)
  5. Once the user provides the token, store it:

```
mcp_call({
  action: "mcp",
  name: "set_notion_token",
  arguments: {
    user_id: "<user_id>",
    token: "<user_provided_token>"
  }
})
```

**Determining user_id**: The `user_id` is the Accio account ID. Find it from:
- The agent's skill directory path: look for the numeric ID in paths like `~/.accio/accounts/<USER_ID>/agents/...`
- Or from the system prompt's user profile section

### Phase 0b: Discover Knowledge Pages

**Goal**: Traverse the user's Notion workspace, identify knowledge-containing pages, and let the user confirm/adjust.

**Step 1**: Get the workspace's top-level pages:

```
mcp_call({
  action: "mcp",
  name: "search_notion",
  arguments: {
    user_id: "<user_id>",
    page_size: 100
  }
})
```

**Step 2**: From the results, identify **root-level pages** (pages that are parents of other content). These are typically workspace sections like "Knowledge Base", "Projects", "Notes", etc.

For each candidate root page, call `get_notion_block_children` to inspect its children:

```
mcp_call({
  action: "mcp",
  name: "get_notion_block_children",
  arguments: {
    user_id: "<user_id>",
    block_id: "<root_page_id>",
    page_size: 100
  }
})
```

**Step 3**: Classify pages into two categories:
- **Knowledge pages**: Pages/databases containing notes, articles, book reviews, concepts, research, etc.
- **Non-knowledge pages**: Planners, habit trackers, to-do lists, daily logs, calendars, dashboards, templates

**Step 4**: Present the classification to the user and ask for confirmation:

> "I've scanned your Notion workspace and identified the following knowledge sections:
>
> ✅ **Included** (will be used for daily review):
> 1. [Page Name 1]
> 2. [Page Name 2]
> 3. ...
>
> ❌ **Excluded** (non-knowledge pages):
> 1. [Planner/Tracker/etc.]
> 2. ...
>
> Would you like to add or remove any sections? Just tell me."

**Step 5**: After user confirmation, record:
- `knowledge_root_pages`: list of `{id: "<page_id>", name: "<page_name>"}` for all confirmed root pages to traverse
- `excluded_pages`: list of page names to always skip

### Phase 0c: Style Preferences

**Goal**: Let the user choose their preferred review style.

Present the following options:

> "How would you like your daily reviews to feel? Pick a style:
>
> **A. Concise & Deep** (简练有深度)
> Core summary in 2-3 sentences + extension/new perspectives. No fluff.
>
> **B. Academic** (学术探究)
> Structured analysis with references to frameworks, theories, and cross-disciplinary connections.
>
> **C. Casual** (轻松对话)
> Friendly, conversational tone. Like a smart friend chatting with you about interesting ideas.
>
> **D. Socratic** (苏格拉底式)
> Heavy on thought-provoking questions. Minimal direct answers, maximum thinking stimulation.
>
> Also, what language should the reviews be in? (e.g. Chinese/中文, English, Japanese/日本語)"

Record the user's choice as `style_preset` and `language`.

### Phase 0d: Create Review Database in Notion

**Goal**: Create a dedicated database in Notion to store daily reviews.

Choose a parent page: use the first page in `knowledge_root_pages` or the workspace root.

```
mcp_call({
  action: "mcp",
  name: "create_notion_database",
  arguments: {
    user_id: "<user_id>",
    parent: {
      type: "page_id",
      page_id: "<parent_page_id>"
    },
    title: [{ type: "text", text: { content: "📝 Daily Knowledge Review" } }],
    properties: {
      "Name": { "title": {} },
      "Date": { "date": {} },
      "Source Sections": {
        "multi_select": {
          "options": []   // will be auto-populated on first use
        }
      },
      "Trigger": {
        "select": {
          "options": [
            { "name": "Recent Edit", "color": "blue" },
            { "name": "Conversation", "color": "green" },
            { "name": "Random", "color": "gray" }
          ]
        }
      }
    }
  }
})
```

Record the returned database ID as `review_database_id`.

### Phase 0e: Resolve Agent Memory Path

**Goal**: Find the agent's memory directory so the skill can access conversation history.

The agent memory path follows this pattern:
```
~/.accio/accounts/<USER_ID>/agents/<AGENT_DID>/agent-core
```

To find it:
1. Check the system prompt for `${agent_core}` or `${memory}` path references
2. Or look at the skill's own installation path — it's under `<agent-core>/skills/notion-daily-review/`
3. Or search for the MEMORY.md file using glob: `glob({ pattern: "**/MEMORY.md" })`

Record the resolved path as `agent_memory_path`.

### Phase 0f: Save Configuration

After all values are collected, **update this SKILL.md file** by replacing the `<PENDING_INIT>` placeholders with actual values:

```
edit({
  file_path: "<path_to_this_skill>/SKILL.md",
  old_string: 'notion_user_id: "<PENDING_INIT>"',
  new_string: 'notion_user_id: "<actual_value>"'
})
// ... repeat for each config value
```

Also update `knowledge_root_pages` and `excluded_pages` with the actual arrays.

Inform the user:

> "✅ Initialization complete! Your daily knowledge review is ready to use.
> - Monitoring **X** knowledge sections in your Notion
> - Review style: **[chosen style]**
> - Reviews will be saved to **📝 Daily Knowledge Review** database
>
> Say '每日回顾' or 'daily review' anytime to trigger a review, or set up a cron job for automatic daily reviews."

---

## Execution Flow

**Prerequisite**: All config values must be filled (not `<PENDING_INIT>`). If any are missing, run First-Run Initialization first.

Execute strictly in these 4 phases.

### Phase 1: Collect Context Signals

**Goal**: Gather "recent edits" and "conversation memory" signals for context-aware recommendations.

#### 1a. Get Recently Edited Notion Pages

```
mcp_call({
  action: "mcp",
  name: "search_notion",
  arguments: {
    user_id: "<notion_user_id>",
    page_size: 20
  }
})
```

From the results:
- Extract pages edited within the last 3 days (compare `last_edited_time` to today)
- Record their titles, IDs, and parent sections
- These are the "recent edit" signals

#### 1b. Search Memory for Recent Conversation Topics

Use `memory_search` to find recent conversation topics:

```
memory_search({ query: "recent discussion topics, yesterday's conversation, recent interests" })
```

Also get today's date and try to read yesterday's diary:

```
get_time()
read({ file_path: "<agent_memory_path>/diary/YYYY-MM-DD.md" })  // yesterday's date
```

Extract key topic words as "conversation" signals.

---

### Phase 2: Select Knowledge Points

**Goal**: Pick 2-3 knowledge points, prioritizing context relevance.

#### Dynamic Page Discovery

**Step 1**: For each entry in `knowledge_root_pages`, fetch children:

```
mcp_call({
  action: "mcp",
  name: "get_notion_block_children",
  arguments: {
    user_id: "<notion_user_id>",
    block_id: "<root_page_id>",
    page_size: 100
  }
})
```

**Step 2**: Collect all `child_page` and `child_database` blocks. Filter out pages matching `excluded_pages` names and the review database itself.

**Step 3**: This gives you the full list of knowledge sections available for review.

#### Selection Strategy

Pick **2-3 knowledge points** with this priority:

1. **Context-related (1-2)**: Search for pages related to recent edit / conversation signals:

```
mcp_call({
  action: "mcp",
  name: "search_notion",
  arguments: {
    user_id: "<notion_user_id>",
    query: "<keywords from context signals>",
    page_size: 10
  }
})
```

Exclude non-knowledge pages, the review database, and empty pages. Pick the most relevant 1-2.

2. **Random (1)**: For serendipitous discovery:
   - Randomly select one knowledge section from the discovered list
   - If it's a database → use `query_notion_database` to list entries
   - If it's a page collection → use `get_notion_block_children` to list sub-pages
   - Randomly pick one item (use date-based hash for deterministic but varied selection)

#### Read Knowledge Content

For each selected page, read its content:

```
mcp_call({
  action: "mcp",
  name: "get_notion_block_children",
  arguments: {
    user_id: "<notion_user_id>",
    block_id: "<selected_page_id>",
    page_size: 100
  }
})
```

Extract text from all block types (paragraph, heading, list items, callout, toggle, etc.).

---

### Phase 3: Generate Review Content

**Goal**: Generate review summary + deep-dive for each knowledge point.

#### Style Templates

Apply the style based on `style_preset`:

**concise_deep**:
- Core summary: 2-3 sentences, no fluff
- Deep-dive: 150-250 words. Mix of perspective extension (new angles, cross-domain connections) and thought-provoking questions (1-2 specific, challenging questions)

**academic**:
- Core summary: Structured with key concepts highlighted
- Deep-dive: 200-300 words. Reference frameworks/theories, cross-disciplinary analysis, further reading suggestions

**casual**:
- Core summary: Conversational retelling, "here's the key idea..."
- Deep-dive: 150-250 words. Relatable examples, "have you noticed that..." style engagement

**socratic**:
- Core summary: Brief, 1-2 sentences max
- Deep-dive: 200-300 words. Primarily questions — layered from surface to deep, each building on the last. End with one "uncomfortable question" that challenges assumptions.

#### Output Format (per knowledge point)

```markdown
### 📌 [Knowledge Point Title]
> Source: [Section Name] · Trigger: [Recent Edit / Conversation / Random]

**Core Review**
[Summary following the chosen style]

**Deep Dive**
[Extended discussion following the chosen style]

---
```

#### Overall Header

```markdown
## 🧠 Daily Knowledge Review — YYYY-MM-DD

Today's X knowledge points | [Brief explanation of why these were chosen]

---
```

#### Writing Guidelines
- Use the configured `language`
- If cross-references to other Notion pages are natural, include them in the deep dive
- Questions should be specific and challenging, not generic
- Avoid repeating the original text verbatim — add value through synthesis and new connections

---

### Phase 4: Output

#### 4a. Write to Notion Database

Create a single page in the review database containing all knowledge points:

```
mcp_call({
  action: "mcp",
  name: "create_notion_page",
  arguments: {
    user_id: "<notion_user_id>",
    parent: {
      type: "database_id",
      database_id: "<review_database_id>"
    },
    properties: {
      "Name": {
        "title": [{ "text": { "content": "YYYY-MM-DD Daily Knowledge Review" } }]
      },
      "Date": {
        "date": { "start": "YYYY-MM-DD" }
      },
      "Source Sections": {
        "multi_select": [{ "name": "<section_name>" }, ...]
      },
      "Trigger": {
        "select": { "name": "<primary_trigger_type>" }
      }
    },
    children: [
      // All knowledge points as Notion blocks (heading_2, callout, heading_3, paragraph, divider)
      // Same block structure as the markdown format above, converted to Notion block format
    ]
  }
})
```

**Notion block structure per knowledge point**:

```json
[
  {
    "object": "block", "type": "heading_2",
    "heading_2": { "rich_text": [{ "type": "text", "text": { "content": "📌 Title" } }] }
  },
  {
    "object": "block", "type": "callout",
    "callout": {
      "rich_text": [{ "type": "text", "text": { "content": "Source: ... · Trigger: ..." } }],
      "icon": { "emoji": "📍" }
    }
  },
  {
    "object": "block", "type": "heading_3",
    "heading_3": { "rich_text": [{ "type": "text", "text": { "content": "Core Review" } }] }
  },
  {
    "object": "block", "type": "paragraph",
    "paragraph": { "rich_text": [{ "type": "text", "text": { "content": "..." } }] }
  },
  {
    "object": "block", "type": "heading_3",
    "heading_3": { "rich_text": [{ "type": "text", "text": { "content": "Deep Dive" } }] }
  },
  {
    "object": "block", "type": "paragraph",
    "paragraph": { "rich_text": [{ "type": "text", "text": { "content": "..." } }] }
  },
  { "object": "block", "type": "divider", "divider": {} }
]
```

#### 4b. Send in Chat

- If triggered by cron/scheduled task: use `question` tool to send the formatted review
- If triggered by user message: output directly in the reply

---

## Important Notes

1. **Exclude non-knowledge pages**: Skip planners, to-do lists, habit trackers, daily logs, the review database itself, and any pages in `excluded_pages`
2. **Avoid repeats**: Query the review database for the last 7 days to avoid selecting the same knowledge point twice
3. **Handle empty content**: If a selected page has < 50 characters of content, skip it and pick another
4. **Database-type sections**: Use `query_notion_database` instead of `get_notion_block_children` for database-type knowledge sources
5. **Cross-references**: Naturally connect knowledge points to each other and to other content in the user's Notion
6. **Error handling**: If a Notion API call fails, retry once. If still failing, skip that step and continue

---

## Trigger Phrases

Any of these should activate this skill:
- "每日回顾" / "知识回顾" / "今天回顾什么"
- "帮我回顾 Notion 知识"
- "daily review" / "notion review" / "knowledge review"
- Cron/scheduled task automatic trigger
