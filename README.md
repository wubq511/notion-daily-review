# 📝 Notion Daily Knowledge Review

<<<<<<< HEAD
A [Cowork](https://claude.ai) skill that turns your Notion workspace into an intelligent spaced-repetition system. Every day, it picks 2-3 knowledge points from your Notion, generates concise review summaries with deep-dive discussions, and writes them back to a dedicated Notion database.
=======
An agent skill that turns your Notion workspace into an intelligent spaced-repetition system. Every day, it picks 2-3 knowledge points from your Notion, generates concise review summaries with deep-dive discussions, and writes them back to a dedicated Notion database.
>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908

## Features

- **Smart Selection**: Prioritizes knowledge points related to your recent Notion edits and conversation topics, plus random picks for serendipity
- **Deep Dive Generation**: Goes beyond simple recap — extends with new perspectives, cross-domain connections, and thought-provoking questions
- **Auto-Setup**: First run automatically discovers your Notion structure, identifies knowledge sections, and creates a review database
- **Multiple Styles**: Choose from 4 review styles — Concise & Deep, Academic, Casual, or Socratic
- **Dual Output**: Reviews are both saved to Notion (for long-term reference) and sent in chat (for immediate reading)
- **Context-Aware**: Leverages agent memory to connect reviews with your recent conversations and interests
- **Dynamic Discovery**: Automatically picks up new Notion sections — no manual config updates needed

## Prerequisites

<<<<<<< HEAD
- [Claude Desktop](https://claude.ai/download) with Cowork mode enabled
- [Notion MCP Server](https://github.com/makenotion/notion-mcp-server) connected
=======
- An AI agent with MCP tool access
>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908
- A Notion workspace with knowledge content (notes, articles, book reviews, etc.)
- Notion Integration Token ([how to get one](#notion-integration-setup))

## Installation

### Step 1: Install the Notion MCP Server

<<<<<<< HEAD
1. Clone or download the Notion MCP Server:
   ```
   git clone https://github.com/makenotion/notion-mcp-server.git
   ```
=======
```
<agent-core>/skills/
```
>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908

2. Follow the server's README to build and run it. Typically:
   ```
   cd notion-mcp-server
   npm install
   npm run build
   ```

3. In Claude Desktop settings, add the MCP server configuration:
   ```json
   {
     "mcpServers": {
       "notion-mcp-server": {
         "command": "node",
         "args": ["<path-to-notion-mcp-server>/dist/index.js"],
         "env": {
           "NOTION_TOKEN": "<your-internal-integration-secret>"
         }
       }
     }
   }
   ```

### Step 2: Install This Skill

Copy the `notion-daily-review` folder into your Cowork skills directory, or use the skill installation flow in Cowork.

### Step 3: Connect Notion Pages

In each Notion page you want the skill to access:
1. Click **"..."** (three dots menu)
2. Click **"Connect to"**
3. Select your integration (e.g. "Cowork")

> **Important**: Connect your top-level pages. Child pages inherit the connection automatically.

## First Run

On first use, say "每日回顾" or "daily review". The skill runs an interactive initialization:

### 1. Notion Connection Check
Verifies the Notion MCP server is connected and authenticated.

### 2. Knowledge Page Discovery
Scans your Notion workspace and classifies pages into:
- ✅ **Knowledge sections** (notes, articles, concepts) — included in reviews
- ❌ **Non-knowledge sections** (planners, trackers, to-do lists) — excluded

You can adjust the classification.

### 3. Style Selection

| Style | Description |
|-------|-------------|
| **Concise & Deep** | 2-3 sentence core + extension/new perspectives. No fluff. |
| **Academic** | Structured analysis with frameworks and cross-disciplinary references. |
| **Casual** | Friendly, conversational. Like chatting with a smart friend. |
| **Socratic** | Question-heavy. Minimal answers, maximum thinking stimulation. |

### 4. Database Creation
A `📝 Daily Knowledge Review` database is automatically created in your Notion.

## Usage

### Manual Trigger

Say any of these to Claude:

- `每日回顾` / `知识回顾`
- `daily review` / `notion review`
- `帮我回顾 Notion 知识`

### Scheduled (Cron)

<<<<<<< HEAD
Use the Cowork `schedule` skill to set up automatic daily reviews:

```
Schedule a daily task at 9 PM that runs "每日回顾"
=======
Set up a scheduled task in your agent to trigger the skill automatically. Example for daily 9 PM review:

```
schedule: { kind: "cron", expr: "0 21 * * *", tz: "Asia/Shanghai" }
message: "每日回顾"
>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908
```

Or configure via the schedule skill directly.

## How It Works

```
Phase 1: Collect Context Signals
  • Recent Notion edits (last 3 days)
  • Conversation topics from memory

Phase 2: Select Knowledge Points
  • 1-2 context-related (from signals)
  • 1 random (for serendipity)
  • Dynamic page discovery (no hardcoding)

Phase 3: Generate Review Content
  • Core summary (style-dependent)
  • Deep dive (extension + questions)
  • Cross-references to other Notion pages

Phase 4: Output
  • Write to Notion review database
  • Send formatted review in chat
```

<<<<<<< HEAD
=======
## Notion Integration Setup

1. Go to [https://www.notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **"+ New integration"**
3. Name it (e.g., "My Agent"), select your workspace, click **Submit**
4. Copy the **Internal Integration Secret** (starts with `ntn_`)
5. In each Notion page you want the skill to access: click **"..."** → **"Connect to"** → select your integration

> **Important**: You must connect the integration to your top-level pages. Child pages inherit the connection automatically.

>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908
## Configuration

Configuration is stored in the memory system as `notion_daily_review_config.md`. Key settings:

```yaml
notion_token: "ntn_xxx..."
review_database_id: "abc123-..."
knowledge_root_pages:
  - { id: "page-id-1", name: "Knowledge Hub" }
  - { id: "page-id-2", name: "Projects" }
excluded_pages:
  - "2026 Planner"
  - "Habit Tracker"
<<<<<<< HEAD
=======
agent_memory_path: "/home/user/.agent/memory"
>>>>>>> dfccf170cd02bec85f4179ccdd396881131f4908
style_preset: "concise_deep"
language: "zh-CN"
last_reviewed_pages: []
```

To change settings, tell Claude "reconfigure daily review" or edit the config file directly.

## Notion Integration Setup

1. Go to https://www.notion.so/my-integrations
2. Click **"+ New integration"**
3. Name it (e.g. "Cowork"), select your workspace, click **Submit**
4. Copy the **Internal Integration Secret** (starts with `ntn_`)
5. In each Notion page: click **"..."** → **"Connect to"** → select your integration

## License

MIT
