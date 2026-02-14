---
name: tliner:load
description: "Restore previous work into context (most relevant first). Use when resuming previous work, searching for past investigations, retrieving specific task history, or filtering by time/keywords. Usage: [task_id | keywords] [since] [until]"
allowed_tools: ["Read", "Write", "Glob", "mcp__timeliner__get_steps"]
---

## Task

Universal data retrieval from the Timeline. Accepts composable filters (AND logic):
- **IDs**: Task or step ID (e.g., `20251217T084803.537626Z`)
- **Time phrases**: Natural language (e.g., "last week", "since Monday", "3 days ago")
- **Keywords**: Fuzzy search in title/outcomes/tags (multiple keywords use OR logic)
- **Summaries mode**: `summaries` keyword → lightweight response (summary + metadata, no outcomes)

$ARGUMENTS

## Flow: Parse → Convert → Retrieve → Understand → Acknowledge

### 1. **Parse Input**

Analyze `$ARGUMENTS` to extract:

**IDs** - Match pattern `20YYMMDDTHHMMSS.ffffffZ` or timestamp-like strings:
- If task_id(s) provided explicitly (`$ARGUMENTS`) -> MANDATORY use them and ONLY them
- Else if there is MEMORIZED taskId(s) from the last `/save` command -> MANDATORY use them and ONLY them
- Else -> no ID filter; default time to "today" if no time phrases in $ARGUMENTS either
- Multiple IDs allowed

**Time phrases** - Common patterns:
- "last week", "this week", "last month"
- "since Monday", "since December 1"
- "2 days ago", "yesterday", "today"
- "from X to Y", "between X and Y"

**Summaries** - Lightweight mode:
- If `$ARGUMENTS` contains the word `summaries` → set `summaries_only=True`
- Remove `summaries` from arguments before keyword extraction
- Returns step metadata + summary only, no full outcomes (saves context)

**Keywords** - Everything else:
- Remaining words after extracting IDs and time phrases
- Passed to `query` param for fuzzy search (partial matches work)
- Multiple keywords use OR logic (matches ANY keyword)

**Additional behavior**:
- Parse any additional instructions from load command

### 2. **Convert Time → ISO UTC**

**Parse time period:**
- Convert natural language inputs to ISO 8601 format (`YYYY-MM-DDTHH:MM:SSZ`) using UTC midnight boundaries
- "last week" = previous calendar week (Monday 00:00:00Z, until next Monday 00:00:00Z exclusive)
- Calculate dates relative to current **UTC date**
- **IMPORTANT**: `until` is EXCLUSIVE (steps < until, not <=)

**Examples:** If today is 2025-10-30 (Thursday):
- "2 days ago" → since: "2025-10-28T00:00:00Z" (includes 2025-10-28 and later)
- "last week" → since: "2025-10-20T00:00:00Z", until: "2025-10-27T00:00:00Z" (Mon Oct 20 to Sun Oct 26, inclusive)
- "yesterday" → since: "2025-10-29T00:00:00Z", until: "2025-10-30T00:00:00Z" (Oct 29 only)
- "all time" → omit both parameters (empty strings)
- "on 2025-10-28" → since: "2025-10-28T00:00:00Z", until: "2025-10-29T00:00:00Z" (single day)
- "this week" → since: "2025-10-27T00:00:00Z", until: "2025-11-03T00:00:00Z" (Mon Oct 27 to Sun Nov 2, inclusive)


### 3. **Retrieve**

Call `mcp__timeliner__get_steps` with parsed filters:

```
mcp__timeliner__get_steps(
  since="...",        # ISO timestamp or "" if no time filter
  until="...",        # ISO timestamp or "" if no time filter
  ids=[...],          # Array of IDs or None if no ID filter
  query=["kw1", "kw2"],  # Array of keywords or None (OR logic - matches ANY)
  page=1,
  summaries_only=true/false  # true when "summaries" mode detected
)
```

**WARNING**: Do NOT override `max_tokens` unless you have a strong reason. The default (10000) is tuned to stay within safe output limits. Increasing it risks "result exceeds maximum allowed tokens" errors.

**MANDATORY**:
- If no/empty `pagination.warnings` → read ALL pages sequentially (page=1, page=2, ... until page == total_pages)
- If `pagination.warnings`  is non-empty → READ warnings FIRST, then decide strategy:
  - Warnings signal large result sets that may blow context window
  - Follow advice in warnings (narrow filters, use summaries_only, delegate to subagent)
  - Do NOT blindly read all pages when warnings are present

### 4. **Understand**

- Do NOT jump to solutions - understand problem and context first
- Consider timestamps:
  - Some problems already solved
  - Some solutions deprecated/overrided
- Assess whole context, not just last step:
  - Re-evaluate based on all available data
  - Don't blindly follow previous assumptions

### 5. **Acknowledge**

- Briefly confirm understanding of context
- State readiness to continue

---

## Example Usage

```
/load 20251204T182649.123456Z              # Exact ID (task or step)
/load last week                             # Time filter only
/load MkDocs navigation                     # Keyword search (fuzzy match)
/load MkDocs bugs since Monday              # Keywords + time
/load authentication flow, login problems   # Multiple keywords (OR logic)
/load 20251204T182649.123456Z auth          # ID + keyword filter
/load                                       # Use MEMORIZED taskId or default to today
/load summaries last week                   # Summaries only (no outcomes) + time filter
/load summaries                             # Summaries for today
```

## Working Style

1. Be concise (~75 tokens per answer)
2. Use technical language and abbreviations
3. Ask questions frequently
4. Do NOT do what you are not asked for
5. Work is grounded in codebase and documents, not general assumptions
