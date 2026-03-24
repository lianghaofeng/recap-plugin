---
name: recap-monthly
description: "Generate a monthly report aggregating daily and weekly logs. Use when user says 'monthly report', 'month summary', or invokes '/recap monthly'."
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Monthly Report

Aggregate all daily and weekly records into a monthly overview.

## Steps

1. Run `date '+%Y-%m-%d'` to get current date
2. Determine date range from 1st of the month to today
3. Read all daily records and weekly reports for the month
4. Generate monthly report
5. Write to `docs/context/monthly/YYYY-MM.md`
6. Update `docs/context/INDEX.md`

`mkdir -p docs/context/monthly`

## Format

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

## INDEX.md

Add entry with 📊 prefix under the appropriate month section.
