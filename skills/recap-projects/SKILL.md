---
name: recap-projects
description: "Show cross-project recap status. Reads global META from ~/.claude/recap/projects/ to display all tracked projects, their latest sessions, and remaining issues. Use when user says 'show projects', 'cross-project status', or invokes '/recap projects'."
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Cross-Project Status

Display status overview of all projects tracked by the recap plugin.

## Steps

1. Read all JSON files from `~/.claude/recap/projects/*.json`
2. Parse each file for project metadata
3. Sort by `lastSession` descending (most recent first)
4. Display formatted overview
5. Highlight the current project

## Data Source

Each file in `~/.claude/recap/projects/` represents one project:
```json
{
  "project": "project-name",
  "path": "/absolute/path",
  "lastSession": "ISO timestamp",
  "lastRecapFile": "docs/recap_context/daily/YYYY/YYYY-MM-DD.md",
  "remainingIssues": ["issue1", "issue2"],
  "recentTopics": ["topic1", "topic2"],
  "sessionCount": 10
}
```

## Output Format

```
# Cross-Project Status

> Current project: **<current project name>**

| Project | Last Session | Sessions | Open Issues |
|---------|-------------|----------|-------------|
| **current-project** ← | 2026-03-24 17:30 | 12 | 2 |
| other-project | 2026-03-23 14:00 | 45 | 3 |
| old-project | 2026-03-10 09:00 | 8 | 0 |

## Active Projects (last 24h)

### current-project
- Recent topics: JWT auth, database schema
- Remaining: refresh token logic

### other-project
- Recent topics: checkout flow, payment API
- Remaining: error handling, load testing

## Inactive Projects (>24h)
- **old-project** (14 days ago) — 0 open issues
```

If `~/.claude/recap/projects/` doesn't exist or is empty, inform the user that no cross-project data exists yet and explain it will be populated automatically after recap sessions.
