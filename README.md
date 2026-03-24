# Recap — Session Summary & Daily Review

A Claude Code plugin that automatically records conversation summaries by date, helping you track daily work and review progress over time.

## Features

- **Auto Recap** — Automatically generates a session summary when a conversation ends
- **Session Recovery** — Shows previous session's remaining issues when starting a new conversation
- **Weekly Report** — Aggregates daily logs into a weekly summary
- **Monthly Report** — Aggregates weekly/daily logs into a monthly overview
- **Quick Notes** — Append timestamped notes anytime during a session
- **Index** — Maintains a monthly-grouped index for easy navigation
- **Bilingual** — Full support for English and Chinese (中文)

## Installation

```
/plugin install recap
```

After installation, reload plugins:

```
/reload-plugins
```

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

### Status

```
/status             # English
/status-zh          # 中文
```

## Output Structure

```
docs/context/
├── INDEX.md              # Monthly-grouped index
├── 2026-03-24.md         # Daily session logs
├── 2026-03-23.md
├── weekly/
│   └── 2026-W12.md       # Weekly reports
└── monthly/
    └── 2026-03.md        # Monthly reports
```

### Daily Log Format

```markdown
# 2026-03-24 Session Log

## Session 1 (14:30)

### Topics
- Implemented user authentication
- Discussed database schema changes

### Work Done
- Added JWT middleware to API routes
- Created migration for users table

### Files Changed
- M src/auth/middleware.go
- A src/db/migrations/003_users.sql

### Key Decisions
- Chose JWT over session cookies — stateless, better for microservices

### Remaining Issues
- Need to add refresh token logic
```

## Hooks

The plugin includes two automatic hooks:

| Hook | Trigger | Behavior |
|------|---------|----------|
| **Stop** | Conversation ends | Auto-generates recap if none was created in the last 3 minutes |
| **UserPromptSubmit** | New conversation starts | Shows remaining issues from the most recent session (once per day) |

## WSL2 Note

On WSL2 environments where the Write tool creates files invisible to Windows, the plugin will use Bash heredoc (`cat >`) for file writing.

## License

MIT
