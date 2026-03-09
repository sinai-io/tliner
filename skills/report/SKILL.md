---
name: tliner:report
description: "Your progress summary. Use when generating work summaries, creating status reports, or reviewing progress on specific topics. Usage: [task_id | topic] [since] [until]"
allowed-tools: Skill
---

# Task

Generate a formatted summary of work done on a specific topic by analyzing Task Data in the Timeline.

> **Note**: This command uses the same time parsing and keyword filtering logic as `/load`.
> Execute `/load` then add formatting and grouping on top per the next rules.


### 1. Execute `/load`

- Execute `/load` skill with $ARGUMENTS

### 2. Analyze and Group Steps

Group by **thematic similarity** (not temporal proximity). Chronological order within each theme.
Each group = one topic section (aim for 2-5).

Extract per topic:
- Key findings and results (from outcomes)
- Implementation details (files, functions, algorithms)
- Status (completed, in progress, blocked)
- Links (PR/MR references, commits, issues)

### 3. Generate Report

#### Writing Rules

0. **MANDATORY GOLDEN RULE**: Write like you're explaining to a colleague in Slack.
   - Short sentences. Simple words. Lazy and direct.
   - Casual BUT keep full technical precision.

1. **State findings, not process.** Lead with what you found, not what you did.
   - ✅ "3 callsites cause soft lockups: BPF JIT, icache flush, jump label patching"
   - ❌ "Pulled kernel logs from 5 containers. Found 3 distinct callsites causing soft lockups..."

2. **One fact per bullet.** If a bullet has two facts — split it.
   - ✅ "vcpu doubles boot time (125s vs 63s)"
   - ✅ "half the penalty is cp -rp sysfs overlay (34.3s vs 3.3s)"
   - ❌ "vcpu config doubles boot time (125s vs 63s). The cp -rp sysfs overlay is 10x slower with vcpu (34.3s vs 3.3s) — accounts for half the penalty."

3. **Max ~20 words per bullet.** Forces conciseness. If longer — split.

4. **Front-load key numbers and results.** Setup context goes second.
   - ✅ "boot reliability: base 100%, vcpu 80%, standalone 60%"
   - ❌ "Ran automated 15-CVD boot test (5 per config). Results: base 5/5 (100%)..."

5. **Skip the narrative.** Don't tell the story of how you got there.

6. **Bad examples to avoid:**
   - ❌ "An investigation was conducted regarding the API endpoint configuration, which revealed that the routing mechanism was not properly configured."
   - ❌ "Launched 2 containers side-by-side for clean comparison: vcpu config doubles boot time..."

#### Format

```markdown
# Report: [Topic Name]
**Period:** [parsed time range or "All time"]  |  **Generated:** [current UTC timestamp]

## TL;DR
[1-3 sentences. The headline. What happened, what's the bottom line.]

---

## [Topic/Theme Name]

- [finding/result — one fact, ~20 words max]
- [finding/result]
- [finding/result]
- 🔗 PR #123, #456

**Status:** [Completed/In progress/Blocked]

---

## [Another Topic]

...

---

## Blockers / Remaining
- [what's stuck or left to do — omit section if nothing]
```

### 4. Output Report

Print the fully formatted markdown report to terminal for user review.


## Edge Cases and Notes

- **No matches**: Print "No work found for [topic] in period" and exit
- **Single step**: Create one topic section, still include TL;DR
- **Missing metadata**: Omit links line if not present
- **Ambiguous time format**: Ask user to clarify
- **Calendar week**: "last week" always refers to previous Monday-Sunday period
- **Exclusive end date**: `until` parameter excludes the boundary (steps < until, not <=)
  - To include a full day: set `until` to next day's midnight
  - Example: "on Oct 28" = since: "2025-10-28T00:00:00Z", until: "2025-10-29T00:00:00Z"
- Keywords for searching must be case-insensitive
- Single date parameter allowed (backend filters with open conditionals)
