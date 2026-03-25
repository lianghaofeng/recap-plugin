---
name: recap-progress
description: "Track and update project progress. Shows current focus, completed milestones, blockers, and next steps. Use when user says 'show progress', 'update progress', or invokes '/recap progress'. Example: '/recap progress', '/recap progress update'."
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# Progress Tracking

Maintain a living progress document for the current project.

## Arguments

| $ARGUMENTS | Action |
|---|---|
| empty / `progress` | Display current progress |
| `update` | Update progress from conversation context |

## Display Progress

1. Read `docs/recap_context/PROGRESS.md`
2. If exists → display contents
3. If not → "No progress file yet. Use `/recap progress update` to create one."

## Update Progress

1. Run `date '+%Y-%m-%d %H:%M'`
2. Run `git log --oneline -10` for recent activity
3. Read existing `docs/recap_context/PROGRESS.md` if it exists
4. Read `docs/recap_context/META.json` if it exists
5. Review the current conversation context
6. Write/overwrite `docs/recap_context/PROGRESS.md` (this is always a full rewrite, not append)

## Format

```markdown
# Progress — <project-name>

> Last updated: YYYY-MM-DD HH:MM

## Current Focus
- What is actively being worked on right now

## Completed Milestones
- [YYYY-MM-DD] Milestone description
- [YYYY-MM-DD] Milestone description

## Blockers
- Blocker description and what's needed to unblock

## Next Steps
- Prioritized list of what to do next
```

## Rules

- Always overwrite the entire file (not append) — this is a snapshot of current state
- Keep each section concise (3-7 bullet points max)
- Completed Milestones should be ordered newest first
- If no blockers, omit the Blockers section
- Pull remaining issues from the latest recap to populate Next Steps
