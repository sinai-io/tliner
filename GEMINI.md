# tliner Extension

Timeline tracking for AI work. Logs decisions, outcomes, and context as markdown files.

## MCP Tools

| Tool | Description |
|------|-------------|
| `timeliner_save_step` | Save current work to timeline |
| `timeliner_get_steps` | Retrieve past work (time/keyword filters) |
| `timeliner_task_list` | List all tasks |
| `timeliner_group_steps` | Consolidate scattered steps |

## Work Folder

Default: `docs/timeline/` relative to project.
Customize: Set `TIMELINER_WORK_FOLDER` env var.

## Task ID

Format: `20251225T211153.467845Z`

When compacting context, preserve the current task ID. Add to summary:
```
Tliner TASK_ID: 20251225T211153.467845Z
```

## Skills

Available in `skills/`: save, load, report, group.
