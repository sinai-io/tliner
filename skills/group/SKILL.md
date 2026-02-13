---
name: tliner:group
description: "Consolidate scattered steps into coherent tasks. Usage: [task_id | topic] [since] [until]"
allowed_tools: ["Read", "mcp__timeliner__get_steps", "mcp__timeliner__group_steps", "mcp__timeliner__task_list"]
---

## Task

Consolidate scattered Timeline steps into coherent task files. LLM decides groupings semantically, MCP executes moves.

$ARGUMENTS

## Flow: Load Summaries → Analyze → Group → Execute

### Phase 1. Load Summaries via `/load` (`load.md`)

- Read `load.md` file and execute it with `summaries $ARGUMENTS`
- This passes `summaries_only=True` to get lightweight step data (summary + metadata, no full outcomes)
- Summaries carry enough signal for grouping decisions while saving context

**CRITICAL - Pagination**:
- Do NOT stop at "enough" data. Fetch ALL pages (page=1, 2, ... until page == total_pages)
- "Time-first" rule requires complete data - stopping early = wrong target selection
- If total_pages > 1, you MUST continue fetching

### Phase 2. Analyze Steps Semantically

**Goal**: Identify which steps belong together.

**LLM decides WHAT to group** (semantic):
- Group by **coherent narrative**, not by keyword match
- Ask: "Would future-me read these steps together as ONE guide?"
- Same topic can yield MULTIPLE groups:
  - Implementation vs. testing tools vs. deployment scripts
  - Investigation phase vs. fix phase
  - Core work vs. supporting artifacts
- Do NOT create mega-tasks (dumping all keyword matches into one pile)
- Do NOT regroup already coherent tasks
- Consider: same feature? same bug? same thread? same artifact type? etc.

**Rules decide WHERE it goes** (mechanical - no judgment):
- "Time-first becomes king" = target is task with **smallest timestamp** (min `task_id`)
- Do NOT pick target based on "most comprehensive" or "best fit" - use earliest timestamp only
- If MULTIPLE groups identified, each group has its own earliest target
- If NO existing task fits a group → create a new task (use `create_new_tasks` param)

**Skip protection** (check explicitly, don't assume):
- Parse each step's `metadata.groupable` field - if `false`, skip
- Parse each task's `groupable` field - if `false`, skip ALL its steps
- List skipped items in Phase 3 presentation

**Output**: Two mappings:
- `move_to_tasks`: `{existing_task_id: [step_ids]}` for groups targeting existing tasks
- `create_new_tasks`: `{title: [step_ids]}` for groups that need a new task

### Phase 3. Present Grouping Plan

Before executing, show the user:

```markdown
## Proposed Groupings

### Group 1: [narrative_name]
**Target**: [earliest_task_title] (`task_id`) — existing

Moving:
- [step_title] (`step_id`) from [source_task]
- [step_title] (`step_id`) from [source_task]

### Group 2: [narrative_name]
**Target**: "[new task title]" — NEW task

Moving:
- [step_title] (`step_id`) from [source_task]

### No Action (already coherent or no group fit)
- [task_title] (`task_id`) - [reason]

### Skipped (groupable: false)
- [step_title] (`step_id`) - protected

### Summary
- X steps will move across Y groups
- N new tasks will be created
- Z source tasks will be deleted (emptied)
```

**Ask user to confirm** before proceeding.

### Phase 4. Execute Grouping

Call `mcp__timeliner__group_steps` with two parameters:

```python
mcp__timeliner__group_steps(
    move_to_tasks={
        "20251225T100000.000000Z": ["step_id_1", "step_id_2"],
    },
    create_new_tasks={
        "Database migration": ["step_id_3", "step_id_4"],
        "Auth refactor": ["step_id_5"],
    }
)
```

- `move_to_tasks`: existing task_id → list of step_ids to move there
- `create_new_tasks`: new task title → list of step_ids (creates task, then moves steps)
- Both params are optional — use whichever applies
- WARNING: JSON silently drops duplicate keys. Use unique titles in `create_new_tasks`.

### Phase 5. Report Results

```markdown
Grouping Complete:

**Moved**: X steps
**Created**: N new tasks
**Skipped**: Y steps (groupable: false)
**Deleted tasks**: Z empty task files
```

## Writing Style

- Concise summaries when presenting plan
- Technical precision in step/task IDs
- Ask for confirmation before destructive operations
