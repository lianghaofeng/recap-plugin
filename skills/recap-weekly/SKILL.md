---
name: recap-weekly
description: "Generate a weekly report aggregating daily session logs. Use when user says 'weekly report', 'week summary', or invokes '/recap weekly'."
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Weekly Report

Aggregate daily session logs into a weekly summary.

## Steps

1. Run `date '+%Y-%m-%d'` to get today's date
2. Calculate Monday of the current week
3. Read all `docs/recap_context/YYYY-MM-DD.md` files from Monday to today
4. Generate weekly report
5. Write to `docs/recap_context/weekly/YYYY-WXX.md` (XX = ISO week number)
6. Update `docs/recap_context/INDEX.md`

`mkdir -p docs/recap_context/weekly`

## Format

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

## INDEX.md

Add entry with 📋 prefix under the appropriate month section.
