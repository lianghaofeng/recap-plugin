---
name: recap
description: "Session recap and daily review. Generates structured conversation summaries in docs/context/ by date. Use when user says 'recap', 'summarize this session', 'what did we do today', or wants to log a note. Example: '/recap', '/recap note fixed auth bug', '/recap status'. Routes to sub-skills for weekly/monthly reports, search, projects, progress, and proposals."
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
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
| `progress` | Invoke `recap:recap-progress` skill |
| `progress update` | Invoke `recap:recap-progress` skill with "update" |
| `proposal` | Invoke `recap:recap-proposal` skill |
| `proposal list` | Invoke `recap:recap-proposal` skill with "list" |
| `proposal <N>` | Invoke `recap:recap-proposal` skill with number |

Current arguments: $ARGUMENTS

If the argument matches a sub-skill, invoke that skill and stop here.

---

## Full Session Recap (no arguments)

1. Run `date '+%Y-%m-%d %H:%M'` and `git status --short`
2. Review conversation — extract (omit empty sections, never write "None"):
   - **Topics** — Subjects discussed (2-5 items)
   - **Work Done** — Concrete tasks completed
   - **Files Changed** — From git status, marked M/A/D
   - **Key Decisions** — "Why A instead of B" with rationale
   - **Remaining Issues** — Unfinished work, next steps
   - **Delegated Tasks** (optional) — If sub-agents were used, list: `[agent-type] summary of what it did`
3. Check if `docs/context/YYYY-MM-DD.md` exists
4. Write summary (new file or append as next Session N with `---` separator)
5. Post-write sync (see below)

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
6. Post-write sync (see below)

---

## Status (`status`)

1. Check if `docs/context/YYYY-MM-DD.md` exists
2. Yes → show session count, latest session time, note count
3. No → "No recap for today yet"

---

## Post-Write Sync (after every recap/note)

Run all of the following after writing recap content:

### 1. META Sync

**Project META** — `docs/context/META.json`:
```json
{
  "project": "<from git root basename>",
  "totalSessions": "<count>",
  "lastSession": "<ISO timestamp>",
  "recentTopics": ["<latest topics>"],
  "remainingIssues": ["<latest remaining issues>"],
  "recapFiles": ["YYYY-MM-DD.md", ...],
  "proposals": ["001-xxx.md", ...]
}
```

**Global META** — `~/.claude/recap/projects/<project-name>.json`:
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
  "sessionCount": "<total>"
}
```

### 2. DECISIONS.md Auto-Extract

If the recap contains a **Key Decisions** section, append each decision to `docs/context/DECISIONS.md`:

```markdown
# Decision Log

## YYYY-MM-DD
- Decision — Rationale
```

If `DECISIONS.md` doesn't exist, create it with the header. Append under existing date section or add new date section (newest first).

### 3. Git Auto-Commit

Unless `$RECAP_AUTO_COMMIT` is set to `false`:

```bash
git add docs/context/
git commit -m "recap: YYYY-MM-DD session N summary"
```

---

## INDEX.md Maintenance

Update `docs/context/INDEX.md` after every write. Group by month (newest first), newest date first. Use 📋 weekly, 📊 monthly, 📝 proposals. Create if missing.

## File Writing

`mkdir -p docs/context/weekly docs/context/monthly docs/context/proposals`

Use **Write** tool. On WSL2, fallback to Bash heredoc if needed.
