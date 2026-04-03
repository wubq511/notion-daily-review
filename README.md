# 📝 Notion Daily Knowledge Review

An agent skill that turns your Notion workspace into an intelligent spaced-repetition system. Every day, it picks 2-3 knowledge points from your Notion, generates concise review summaries with deep-dive discussions, and writes them back to a dedicated Notion database.

## Features

- **Smart Selection**: Prioritizes knowledge points related to your recent Notion edits and conversation topics, plus random picks for serendipity
- **Deep Dive Generation**: Goes beyond simple recap — extends with new perspectives, cross-domain connections, and thought-provoking questions
- **Auto-Setup**: First run automatically discovers your Notion structure, identifies knowledge sections, and creates a review database
- **Multiple Styles**: Choose from 4 review styles — Concise & Deep, Academic, Casual, or Socratic
- **Dual Output**: Reviews are both saved to Notion (for long-term reference) and sent in chat (for immediate reading)
- **Context-Aware**: Leverages agent memory to connect reviews with your recent conversations and interests
- **Dynamic Discovery**: Automatically picks up new Notion sections — no manual config updates needed

## Prerequisites

- An AI agent with MCP tool access
- A Notion workspace with knowledge content (notes, articles, book reviews, etc.)
- Notion Integration Token ([how to get one](#notion-integration-setup))

## Installation

Copy the `notion-daily-review` folder into your agent's skill directory:

```
<agent-core>/skills/
```

The skill will be automatically detected on the next conversation.

## First Run

On first use, the skill runs an interactive initialization:

### 1. Notion Connection Check
The skill checks if Notion MCP tools are available and authenticated. If not, it guides you through setting up a Notion Integration Token.

### 2. Knowledge Page Discovery
The skill scans your Notion workspace and classifies pages into:
- ✅ **Knowledge sections** (notes, articles, concepts, etc.) — included in reviews
- ❌ **Non-knowledge sections** (planners, trackers, to-do lists) — excluded

You can add or remove sections from either list.

### 3. Style Selection
Choose your preferred review style:

| Style | Description |
|-------|-------------|
| **Concise & Deep** | 2-3 sentence core + extension/new perspectives. No fluff. |
| **Academic** | Structured analysis with frameworks and cross-disciplinary references. |
| **Casual** | Friendly, conversational. Like chatting with a smart friend. |
| **Socratic** | Question-heavy. Minimal answers, maximum thinking stimulation. |

### 4. Database Creation
A `📝 Daily Knowledge Review` database is automatically created in your Notion with fields for date, source sections, and trigger type.

### 5. Configuration Saved
All settings are written directly into the skill file — no separate config files needed.

## Usage

### Manual Trigger

Say any of these to your agent:

- `每日回顾` / `知识回顾`
- `daily review` / `notion review`
- `帮我回顾 Notion 知识`

### Scheduled (Cron)

Set up a scheduled task in your agent to trigger the skill automatically. Example for daily 9 PM review:

```
schedule: { kind: "cron", expr: "0 21 * * *", tz: "Asia/Shanghai" }
message: "每日回顾"
```

## How It Works

```
┌─────────────────────────────────────────────┐
│  Phase 1: Collect Context Signals           │
│  • Recent Notion edits (last 3 days)        │
│  • Yesterday's conversation topics (memory) │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  Phase 2: Select Knowledge Points           │
│  • 1-2 context-related (from signals)       │
│  • 1 random (for serendipity)               │
│  • Dynamic page discovery (no hardcoding)   │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  Phase 3: Generate Review Content           │
│  • Core summary (style-dependent)           │
│  • Deep dive (extension + questions)        │
│  • Cross-references to other Notion pages   │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  Phase 4: Output                            │
│  • Write to Notion review database          │
│  • Send formatted review in chat            │
└─────────────────────────────────────────────┘
```

## Notion Integration Setup

1. Go to [https://www.notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **"+ New integration"**
3. Name it (e.g., "My Agent"), select your workspace, click **Submit**
4. Copy the **Internal Integration Secret** (starts with `ntn_`)
5. In each Notion page you want the skill to access: click **"..."** → **"Connect to"** → select your integration

> **Important**: You must connect the integration to your top-level pages. Child pages inherit the connection automatically.

## Configuration

All configuration is stored in the `SKILL.md` file's YAML block. After initialization, it looks like:

```yaml
notion_user_id: "1234567890"
review_database_id: "abc123-def456-..."
review_database_name: "📝 Daily Knowledge Review"
knowledge_root_pages:
  - { id: "page-id-1", name: "Knowledge Hub" }
  - { id: "page-id-2", name: "Projects" }
excluded_pages:
  - "2026 Planner"
  - "Habit Tracker"
agent_memory_path: "/home/user/.agent/memory"
style_preset: "concise_deep"
language: "zh-CN"
```

To change settings, either edit the YAML directly or tell your agent "reconfigure daily review".

## License

MIT
