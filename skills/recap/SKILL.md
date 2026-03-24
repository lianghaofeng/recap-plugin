---
name: recap
description: "Session recap and daily review tool. Generates structured conversation summaries stored in docs/context/ by date. Supports full session recaps, weekly/monthly reports, and quick notes. Use when the user says 'recap', 'summarize this session', 'what did we do today', 'weekly report', 'monthly report', or wants to log a note. Automatically triggers when a session ends without a recap."
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "1.0.0"
---

# Recap — Session Summary & Daily Review

Distill conversations into structured summaries, archived by date in `docs/context/` for easy review and progress tracking.

## Argument Parsing

| $ARGUMENTS | Action |
|---|---|
| empty | Full session recap |
| `weekly` | Weekly report |
| `monthly` | Monthly report |
| `note <text>` | Append a quick note |
| `status` | Show today's recap status |

Current arguments: $ARGUMENTS

---

## Action 1: Full Session Recap (no arguments)

### Steps

1. **Get current time**: `date '+%Y-%m-%d %H:%M'`
2. **Get file changes**: `git status --short`
3. **Review conversation** — Extract:
   - **Topics** — Subjects and technical points discussed (2-5 items)
   - **Work Done** — Concrete tasks completed (coding, config, debugging)
   - **Files Changed** — From git status, marked M(modified)/A(added)/D(deleted)
   - **Key Decisions** — "Why A instead of B" with rationale
   - **Remaining Issues** — Unfinished work, known bugs, next steps
   
   Omit any section that has no content. Do not write "None".

4. **Check if today's file exists**: `ls docs/context/YYYY-MM-DD.md 2>/dev/null`
5. **Write summary** (see format below)
6. **Update INDEX.md**

### New File Format

```markdown
# YYYY-MM-DD Session Log

## Session 1 (HH:MM)

### Topics
- Topic 1
- Topic 2

### Work Done
- Task 1
- Task 2

### Files Changed
- M path/to/file.cpp
- A path/to/new_file.hpp

### Key Decisions
- Decision — Rationale

### Remaining Issues
- Issue description
```

### Appending to Existing File

Read the existing file to determine the next Session number. Append with `---` separator:

```markdown

---

## Session N (HH:MM)

### Topics
...
```

---

## Action 2: Weekly Report (`weekly`)

### Steps

1. Get date range from Monday to today
2. Read all `docs/context/YYYY-MM-DD.md` files in range
3. Write to `docs/context/weekly/YYYY-WXX.md` (XX = ISO week number)
4. Update INDEX.md

### Format

```markdown
# YYYY Week XX Report (MM-DD ~ MM-DD)

## Week Summary
- One-line summary of main achievements

## Daily Highlights

### Monday (MM-DD)
- Highlight 1
- Highlight 2

### Tuesday (MM-DD)
- Highlight 1

(Skip days with no records)

## Completed This Week
- All completed work items

## Remaining & Next Week
- Unfinished items
- Items to follow up
```

---

## Action 3: Monthly Report (`monthly`)

### Steps

1. Get date range from 1st to today
2. Read all daily records and weekly reports for the month
3. Write to `docs/context/monthly/YYYY-MM.md`
4. Update INDEX.md

### Format

```markdown
# YYYY-MM Monthly Report

## Monthly Overview
- 2-3 sentences on main achievements and direction

## Weekly Review

### Week 1 (MM-01 ~ MM-07)
- Highlights

### Week 2 (MM-08 ~ MM-14)
- Highlights

(List by actual weeks)

## Key Milestones
- Important features, architecture changes

## Tech Debt & Remaining
- Accumulated unresolved issues
```

---

## Action 4: Quick Note (`note <text>`)

Append user's text as a timestamped note to today's record.

### Steps

1. Get current time
2. Extract note content (everything after "note ")
3. Check today's file:
   - **Doesn't exist** → Create with title and Notes section
   - **Exists with `## Notes` section** → Append under it
   - **Exists without `## Notes`** → Add section at end

### Format

```markdown
## Notes
- [HH:MM] User's note text
```

---

## Action 5: Status (`status`)

Quick overview of today's recap state.

### Steps

1. Check if `docs/context/YYYY-MM-DD.md` exists
2. If yes, show: number of sessions, latest session time, note count
3. If no, show: "No recap for today yet"

---

## INDEX.md Maintenance

Update `docs/context/INDEX.md` after every write operation.

### Format

```markdown
# Session Log Index

## 2026-03
- [03-24](2026-03-24.md) - Brief description of main topics
- [03-23](2026-03-23.md) - Brief description
- 📋 [Week 12 Report](weekly/2026-W12.md)
- 📊 [March Report](monthly/2026-03.md)

## 2026-02
- ...
```

### Rules

- Group by month, newest month first
- Within each month, newest date first
- Update description if entry already exists
- Use 📋 for weekly, 📊 for monthly reports
- Create INDEX.md if it doesn't exist

---

## File Writing

Create directories if needed: `mkdir -p docs/context/weekly docs/context/monthly`

Use the **Write** tool for creating/updating files. On WSL2 environments where Write tool files may not be visible to Windows, use Bash heredoc instead:

```bash
cat > docs/context/YYYY-MM-DD.md << 'RECAP_EOF'
content
RECAP_EOF
```

---

## Directory Structure

```
docs/context/
├── INDEX.md
├── 2026-03-24.md
├── 2026-03-23.md
├── weekly/
│   └── 2026-W12.md
└── monthly/
    └── 2026-03.md
```
