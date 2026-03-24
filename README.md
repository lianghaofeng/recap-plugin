# Recap — Session Summary & Cross-Project Review

A global Claude Code plugin that automatically records conversation summaries by date, with cross-project awareness, helping you track daily work and review progress across all your projects.

## Features

- **Auto Recap** — Automatically generates a session summary when a conversation ends
- **Session Recovery** — Shows previous session's remaining issues when starting a new conversation
- **Cross-Project Awareness** — Tracks all projects globally, shows active projects' status on session start
- **Weekly Report** — Aggregates daily logs into a weekly summary
- **Monthly Report** — Aggregates weekly/daily logs into a monthly overview
- **History Search** — Search through historical recaps by keyword
- **Quick Notes** — Append timestamped notes anytime during a session
- **Index** — Maintains a monthly-grouped index for easy navigation
- **META Sync** — Structured JSON metadata at both project and global level
- **Bilingual** — Full support for English and Chinese (中文)

## Installation (Global)

```
claude plugin install recap
```

Or clone directly:

```
git clone https://github.com/lianghaofeng/recap-plugin ~/.claude/plugins/recap
```

After installation, reload plugins:

```
/reload-plugins
```

The plugin works globally — all projects will automatically have recap capabilities.

## Usage

### Session Recap

At the end of a conversation, the plugin auto-triggers. You can also invoke manually:

```
/recap              # English
/recap-zh           # 中文
```

Or just type naturally: "recap", "summarize this session", "总结一下", "复盘"

### Weekly / Monthly Reports

```
/recap weekly       # or: /recap-zh weekly
/recap monthly      # or: /recap-zh monthly
```

### Quick Notes

```
/recap note Fixed the auth bug in middleware
/recap-zh note 修了中间件的认证 bug
```

### History Search

```
/recap search authentication
/recap-zh search 数据库迁移
```

### Cross-Project Status

```
/projects           # English
/projects-zh        # 中文
```

### Status

```
/status             # English
/status-zh          # 中文
```

## Architecture

### Two-Level META System

The plugin maintains metadata at two levels:

**Global Level** (`~/.claude/recap/projects/`):
```
~/.claude/recap/
└── projects/
    ├── recap-plugin.json      # One file per project
    ├── my-webapp.json
    └── data-pipeline.json
```

Each project file contains:
```json
{
  "project": "my-webapp",
  "path": "/home/user/my-webapp",
  "lastSession": "2026-03-24T17:30:00",
  "lastRecapFile": "docs/context/2026-03-24.md",
  "remainingIssues": ["fix auth timeout"],
  "recentTopics": ["checkout flow", "payment API"],
  "sessionCount": 45
}
```

**Project Level** (`docs/context/META.json` in each project):
```json
{
  "project": "my-webapp",
  "totalSessions": 45,
  "lastSession": "2026-03-24T17:30:00",
  "recentTopics": ["checkout flow"],
  "remainingIssues": ["fix auth timeout"],
  "recapFiles": ["2026-03-24.md", "2026-03-23.md"]
}
```

### Concurrent Safety

Each project writes only its own JSON file — no file locking needed. Cross-project reads aggregate all files at query time.

### Progressive Disclosure (Skills)

Skills are split into focused modules, loaded only when needed:

| Skill | Trigger | Context Cost |
|-------|---------|-------------|
| `recap` | Daily recap, notes, status | ~100 lines |
| `recap-weekly` | `/recap weekly` | ~50 lines |
| `recap-monthly` | `/recap monthly` | ~50 lines |
| `recap-search` | `/recap search <query>` | ~40 lines |
| `recap-projects` | `/projects` | ~60 lines |

Each has a Chinese (`-zh`) variant.

## Output Structure

```
docs/context/                          # Per-project
├── META.json                          # Structured project metadata
├── INDEX.md                           # Monthly-grouped index
├── 2026-03-24.md                      # Daily session logs
├── 2026-03-23.md
├── weekly/
│   └── 2026-W12.md                    # Weekly reports
└── monthly/
    └── 2026-03.md                     # Monthly reports

~/.claude/recap/                       # Global (cross-project)
└── projects/
    ├── project-a.json
    └── project-b.json
```

### Daily Log Format

```markdown
# 2026-03-24 Session Log

## Session 1 (14:30)

### Topics
- Implemented user authentication

### Work Done
- Added JWT middleware to API routes

### Files Changed
- M src/auth/middleware.go
- A src/db/migrations/003_users.sql

### Key Decisions
- Chose JWT over session cookies — stateless, better for microservices

### Remaining Issues
- Need to add refresh token logic
```

## Hooks

| Hook | Trigger | Behavior |
|------|---------|----------|
| **Stop** | Conversation ends | Auto-generates recap + syncs global META |
| **UserPromptSubmit** | New conversation starts | Shows current project issues + other active projects (once per day) |

## WSL2 Note

On WSL2 environments where the Write tool creates files invisible to Windows, the plugin will use Bash heredoc (`cat >`) for file writing.

## License

MIT
