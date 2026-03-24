---
name: recap
description: "Session recap and daily review. Generates structured conversation summaries in docs/context/ by date. Use when user says 'recap', 'summarize this session', 'what did we do today', or wants to log a note. Example: '/recap', '/recap note fixed auth bug', '/recap status'. For weekly/monthly reports or history search, routes to sub-skills."
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Recap — Session Summary & Daily Review

Distill conversations into structured summaries, archived by date in `docs/context/`.

## Argument Routing

| $ARGUMENTS | Action |
|---|---|
| empty | Full session recap (below) |
| `weekly` | Invoke `recap:recap-weekly` skill |
| `monthly` | Invoke `recap:recap-monthly` skill |
| `note <text>` | Append a quick note (below) |
| `status` | Show today's status (below) |
| `search <query>` | Invoke `recap:recap-search` skill |
| `projects` | Invoke `recap:recap-projects` skill |

Current arguments: $ARGUMENTS

If the argument matches a sub-skill (weekly, monthly, search, projects), invoke that skill and stop here.

---

## Full Session Recap (no arguments)

1. Run `date '+%Y-%m-%d %H:%M'` and `git status --short`
2. Review conversation — extract (omit empty sections, never write "None"):
   - **Topics** — Subjects discussed (2-5 items)
   - **Work Done** — Concrete tasks completed
   - **Files Changed** — From git status, marked M/A/D
   - **Key Decisions** — "Why A instead of B" with rationale
   - **Remaining Issues** — Unfinished work, next steps
3. Check if `docs/context/YYYY-MM-DD.md` exists
4. Write summary (new file or append as next Session N with `---` separator)
5. Update INDEX.md and META

### File Format

New file: `# YYYY-MM-DD Session Log` → `## Session 1 (HH:MM)` → sections.
Append: read existing file for next Session number, append with `---` separator.

---

## Quick Note (`note <text>`)

1. Run `date '+%Y-%m-%d %H:%M'`
2. If today's file doesn't exist → create with title + `## Notes` section
3. If exists with `## Notes` → append under it
4. If exists without `## Notes` → add section at end
5. Format: `- [HH:MM] User's note text`

---

## Status (`status`)

1. Check if `docs/context/YYYY-MM-DD.md` exists
2. Yes → show session count, latest session time, note count
3. No → "No recap for today yet"

---

## META Sync (after every write)

After writing any recap/note, also update these two files:

### Project META: `docs/context/META.json`
```json
{
  "project": "<from git root basename>",
  "totalSessions": <count>,
  "lastSession": "<ISO timestamp>",
  "recentTopics": ["<latest topics>"],
  "remainingIssues": ["<latest remaining issues>"],
  "recapFiles": ["YYYY-MM-DD.md", ...]
}
```

### Global META: `~/.claude/recap/projects/<project-name>.json`
```bash
mkdir -p ~/.claude/recap/projects
```
```json
{
  "project": "<name>",
  "path": "<git root absolute path>",
  "lastSession": "<ISO timestamp>",
  "lastRecapFile": "docs/context/YYYY-MM-DD.md",
  "remainingIssues": ["<from latest session>"],
  "recentTopics": ["<from latest session>"],
  "sessionCount": <total>
}
```

---

## INDEX.md Maintenance

Update `docs/context/INDEX.md` after every write. Group by month (newest first), newest date first. Use 📋 for weekly, 📊 for monthly. Create if missing.

## File Writing

`mkdir -p docs/context/weekly docs/context/monthly`

Use **Write** tool. On WSL2, fallback to Bash heredoc if needed.
