# Timeliner

> 🌟 AI's diary: track the work with the markdown log

## TL;DR

1. Your AI agent forgets everything between sessions. Timeliner fixes that *elegantly*.
<br/>**The idea:** `/save` your progress → timestamped markdown gets created. Next session, `/load` brings it back into context. That's it.

2. And honestly? You forget a lot too. AI-assisted coding is fast, but your memories get scattered.
<br/>**The fix:** Log is all `.md` files → `/report last week` for a quick recap, use in Obsidian, grep...

Your agent gets *context*. You get *clarity*.

## Table of Contents
- [What is it?](#what-is-it)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Advanced Usage](#advanced-usage)
- [CLI Commands](#cli-commands)

## What is it?

Timeliner tracks development sessions as timestamped markdown files, giving recall of past decisions, implementations, and context.

Makes **context** control for your Agent much easier. Works with any MCP-enabled agent.

### Feature Highlights

- **Markdown storage** - Human-readable logs
- **Agent-agnostic** - Works with Claude Code, Gemini CLI, Opencode, Cline/Roo, and other MCP-enabled tools
- **[Obsidian](https://obsidian.md/)-friendly** - Auto-creates `.obsidian/` with Minimal theme on install

### Use Cases

**Software Development:**
- Document decisions and rationale during coding sessions
- Track refactoring steps
- Log implementation progress

**Research & Learning:**
- Maintain experiment logs with results and insights
- Learning journal
- Track hypothesis


## Installation

> DISCLAIMER: Though many Agents are supported, Timeliner is mostly tested with Claude Code

### Claude Code
```bash
claude plugin marketplace add sinai-io/ai-plugins
claude plugin install tliner
```

### Gemini CLI
```bash
gemini extensions install https://github.com/sinai-io/tliner
```
### OpenCode + OmO

With [OmO](https://github.com/brycewestheimer/oh-my-opencode-local-dev) compatibility layer, use Claude plugins natively:
```bash
claude plugin marketplace add sinai-io/ai-plugins
claude plugin install tliner
```

### Codex CLI

Run the Codex and use the built-in skill installer (separately for every skill in the Timeliner):
```
$skill-installer install tliner skills from https://github.com/sinai-io/tliner/tree/main/skills/save
$skill-installer install tliner skills from https://github.com/sinai-io/tliner/tree/main/skills/load
$skill-installer install tliner skills from https://github.com/sinai-io/tliner/tree/main/skills/report
$skill-installer install tliner skills from https://github.com/sinai-io/tliner/tree/main/skills/group
```

It will run Codex Internal script:  `.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py`.
MCP server is auto-configured when skills reference it.

### Other agents
See [Manual Installation](#manual-installation).


## Quick Start

### `/save`

1. Run `/tliner:save` (`/save`) in Agent Tool, when you feel you have made significant progress or decisions
2. File will be created/updated in `docs/timeline/` folder (configurable - see `TIMELINER_WORK_FOLDER` below)

### `/load`

Restore previous work into **LLM's context**. Universal retrieval: `/tliner:load [topic | task_id ] [since] [until]`

**Examples:**
- `/load last week` - Load all work from last week
- `/load` - Loads data from last save  (remembers context)
- `/load auth bug since Monday` - Topic + time filter
- `/load 20251204T182649.123456Z` - Load exact task or step by ID


### `/report`

Generate work report for **You** analyzing the Timeline: `/tliner:report [topic] [time_period]`

**Examples:**
- `/report authentication bug last 48 hours` - Find work on authentication in recent files
- `/report yesterday` - Generate summary of all work yesterday


## Configuration

### Environment Variables
- `TIMELINER_WORK_FOLDER`: Storage directory (default: `docs/timeline`)
- add envvar to `.claude/settings.json` or `.claude/settings.local.json` :
```json
{
  "env": { "TIMELINER_WORK_FOLDER": "info/mydocs"  }
}
```

## Advanced Usage

### Obsidian Integration

Open your `docs/timeline/` folder in Obsidian and browse your AI work sessions like a wiki.

**What you get out of the box:**
- `.obsidian/` vault auto-created on install
- [Minimal theme](https://github.com/kepano/obsidian-minimal) pre-configured
- Style Settings plugin for theme customization


### Archiving Tasks

Got too many tasks cluttering your timeline? Just toss old ones into year-month folders like `2025_09`, `2025_10`, etc.

Everything still works—searches, retrieval, CLI commands—but now your root stays clean.


### Manual Installation

#### Generic MCP Setup (Cline, Roo, etc.)

Add to your MCP config (`.mcp.json`, `mcp_settings.json`, etc.):

```json
{
  "mcpServers": {
    "timeliner": {
      "command": "uvx",
      "args": ["tliner@latest", "serve"],
      "env": {"TIMELINER_WORK_FOLDER": "${PWD}/docs/timeline"}
    }
  }
}
```

Copy skills from [skills/](./skills/) to your agent's skills directory.


### CLI Commands

For manual inspection and debugging:

```bash
# List all tasks
TIMELINER_WORK_FOLDER="${PWD}/docs/timeline" uvx tliner@latest task-list

# Show all steps for a task
TIMELINER_WORK_FOLDER="${PWD}/docs/timeline" uvx tliner@latest task-show <task_id>

# Run MCP server manually
TIMELINER_WORK_FOLDER="${PWD}/docs/timeline" uvx tliner@latest serve
```
