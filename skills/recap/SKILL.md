---
name: recap
description: "Session recap and daily review. Generates structured conversation summaries in docs/recap_context/ by date. Use when user says 'recap', 'summarize this session', 'what did we do today', or wants to log a note. Example: '/recap', '/recap note fixed auth bug', '/recap status'. Routes to sub-skills for weekly/monthly reports, search, projects, progress, and proposals."
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# Recap — Session Summary & Daily Review

Distill conversations into structured summaries, archived by date in `docs/recap_context/`.

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
| `context` | Invoke `recap:recap-context` skill |
| `context <N>` | Invoke `recap:recap-context` skill with number of days |
| `context full` | Invoke `recap:recap-context` skill with "full" |
| `silent` or `silent on` | Enable silent mode (below) |
| `silent off` | Disable silent mode (below) |
| `silent status` | Show current silent mode setting |

Current arguments: $ARGUMENTS

If the argument matches a sub-skill, invoke that skill and stop here.

---

## Silent Mode Toggle (`silent [on|off|status]`)

Controls whether auto-recap (Stop hook) runs silently or shows output.

1. Determine action from arguments:
   - `silent` or `silent on` → enable
   - `silent off` → disable
   - `silent status` → show current value only

2. For enable/disable: write the setting to `~/.claude/recap/config.json`:
   ```bash
   mkdir -p ~/.claude/recap
   ```
   Read existing `~/.claude/recap/config.json` if it exists, update the `"silent"` field, write back:
   ```json
   {
     "silent": true
   }
   ```
   Merge with existing fields if the file already exists (don't overwrite other settings).

3. Confirm to user:
   - Enable: "Silent mode enabled. Auto-recap will run without visible output."
   - Disable: "Silent mode disabled. Auto-recap will show the generation process."
   - Status: "Silent mode is currently **enabled/disabled**."

---

## Full Session Recap (no arguments)

1. Run `date '+%Y-%m-%d %H:%M'` and `git status --short`
2. Review conversation — extract (omit empty sections, never write "None"):
   - **Topics** — Subjects discussed (2-5 items)
   - **Work Done** — Concrete tasks completed
   - **Files Changed** — From git status, marked M/A/D
   - **Key Decisions** — "Why A instead of B" with rationale
   - **Remaining Issues** — Unfinished work, next steps
   - **Delegated Tasks** (optional) — If sub-agents were used, list: `[agent-type] summary of what it did`. Also check `docs/recap_context/.agent-activity.jsonl` for auto-logged agent completions from this session and include them.
3. Check if `docs/recap_context/daily/YYYY/YYYY-MM-DD.md` exists
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

1. Check if `docs/recap_context/daily/YYYY/YYYY-MM-DD.md` exists
2. Yes → show session count, latest session time, note count
3. No → "No recap for today yet"

---

## Post-Write Sync (after every recap/note)

Run all of the following after writing recap content:

### 1. META Sync

**Project META** — `docs/recap_context/META.json`:
```json
{
  "project": "<from git root basename>",
  "totalSessions": "<count>",
  "lastSession": "<ISO timestamp>",
  "recentTopics": ["<latest topics>"],
  "remainingIssues": ["<latest remaining issues>"],
  "recapFiles": ["daily/YYYY/YYYY-MM-DD.md", ...],
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
  "lastRecapFile": "docs/recap_context/daily/YYYY/YYYY-MM-DD.md",
  "remainingIssues": ["<from latest session>"],
  "recentTopics": ["<from latest session>"],
  "sessionCount": "<total>"
}
```

### 2. DECISIONS.md Auto-Extract

If the recap contains a **Key Decisions** section, append each decision to `docs/recap_context/DECISIONS.md`:

```markdown
# Decision Log

## YYYY-MM-DD
- Decision — Rationale
```

If `DECISIONS.md` doesn't exist, create it with the header. Append under existing date section or add new date section (newest first).

### 3. Git Auto-Commit

Unless `$RECAP_AUTO_COMMIT` is set to `false`:

```bash
# Clean up processed agent activity log
rm -f docs/recap_context/.agent-activity.jsonl
git add docs/recap_context/
git commit -m "recap: YYYY-MM-DD session N summary"
```

---

## INDEX.md Maintenance

Update `docs/recap_context/INDEX.md` after every write. Group by month (newest first), newest date first. Use 📋 weekly, 📊 monthly, 📝 proposals. Create if missing.

## File Writing

`mkdir -p docs/recap_context/daily/$(date +%Y) docs/recap_context/weekly/$(date +%Y) docs/recap_context/monthly docs/recap_context/proposals`

Use **Write** tool. On WSL2, fallback to Bash heredoc if needed.
