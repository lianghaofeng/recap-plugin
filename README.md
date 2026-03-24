# Recap — Session Summary & Cross-Project Review

A global Claude Code plugin that automatically records conversation summaries by date, with cross-project awareness, design proposal management, progress tracking, and decision logging.

## Features

- **Auto Recap** — Automatically generates a session summary when a conversation ends
- **Session Recovery** — Shows previous session's remaining issues when starting a new conversation
- **Error Recovery** — Detects missed recaps and prompts recovery on next session start
- **Cross-Project Awareness** — Tracks all projects globally, shows active projects' status on session start
- **Weekly Report** — Aggregates daily logs into a weekly summary
- **Monthly Report** — Aggregates weekly/daily logs into a monthly overview
- **History Search** — Search through historical recaps by keyword
- **Progress Tracking** — Living progress document with current focus, milestones, blockers
- **Design Proposals** — Structured proposal documents with status lifecycle (draft → accepted → implemented)
- **Decision Log** — Auto-extracts key decisions from recaps into DECISIONS.md
- **Git Auto-Commit** — Automatically commits recap files after writing (configurable)
- **Quick Notes** — Append timestamped notes anytime during a session
- **Multi-Agent Records** — Tracks delegated sub-agent tasks in session recaps
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

### Progress Tracking

```
/recap progress             # View current progress
/recap progress update      # Update from conversation context
```

### Design Proposals

```
/proposal                   # Create proposal from current conversation
/proposal list              # List all proposals
/proposal 001               # View specific proposal
/recap proposal             # Alternative: through recap command
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

## Configuration

### Git Auto-Commit

By default, recap automatically commits `docs/context/` changes after writing. To disable:

```bash
export RECAP_AUTO_COMMIT=false
```

## Architecture

### Two-Level META System

**Global Level** (`~/.claude/recap/`):
```
~/.claude/recap/
├── projects/
│   ├── recap-plugin.json      # One file per project
│   ├── my-webapp.json
│   └── data-pipeline.json
└── pending.log                # Error recovery: tracks incomplete recaps
```

**Project Level** (`docs/context/` in each project):
```json
{
  "project": "my-webapp",
  "totalSessions": 45,
  "lastSession": "2026-03-24T17:30:00",
  "recentTopics": ["checkout flow"],
  "remainingIssues": ["fix auth timeout"],
  "recapFiles": ["2026-03-24.md", "2026-03-23.md"],
  "proposals": ["001-cross-project-meta.md"]
}
```

### Concurrent Safety

Each project writes only its own JSON file — no file locking needed. Cross-project reads aggregate all files at query time.

### Progressive Disclosure (Skills)

Skills are split into focused modules, loaded only when needed:

| Skill | Trigger | Context Cost |
|-------|---------|-------------|
| `recap` | Daily recap, notes, status | ~120 lines |
| `recap-weekly` | `/recap weekly` | ~50 lines |
| `recap-monthly` | `/recap monthly` | ~50 lines |
| `recap-search` | `/recap search <query>` | ~40 lines |
| `recap-projects` | `/projects` | ~60 lines |
| `recap-progress` | `/recap progress` | ~50 lines |
| `recap-proposal` | `/proposal` | ~70 lines |

Each has a Chinese (`-zh`) variant. Total: 14 skills.

### Error Recovery

The Stop hook writes a pending marker to `~/.claude/recap/pending.log`. On next session start, the UserPromptSubmit hook checks for unresolved markers. If a recap file is missing for the pending date, it prompts a recovery recap based on git history.

## Output Structure

```
docs/context/                          # Per-project
├── META.json                          # Structured project metadata
├── PROGRESS.md                        # Living progress document
├── DECISIONS.md                       # Auto-extracted decision log
├── INDEX.md                           # Monthly-grouped index
├── 2026-03-24.md                      # Daily session logs
├── 2026-03-23.md
├── weekly/
│   └── 2026-W12.md                    # Weekly reports
├── monthly/
│   └── 2026-03.md                     # Monthly reports
└── proposals/
    ├── 001-cross-project-meta.md      # Design proposals
    └── 002-progress-tracking.md

~/.claude/recap/                       # Global (cross-project)
├── projects/
│   ├── project-a.json
│   └── project-b.json
└── pending.log                        # Error recovery tracking
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

### Delegated Tasks
- [test-runner] Ran 42 tests, 3 failures in auth module
```

### Proposal Format

```markdown
# Proposal 001: Cross-Project META

- **Status**: implemented
- **Date**: 2026-03-24
- **Author**: lance

## Problem
Need cross-project awareness for recap data.

## Solution
Two-level META architecture with per-project JSON files.

## Trade-offs
Considered single global file vs per-project files.

## Implementation
1. Global META at ~/.claude/recap/projects/
2. Project META at docs/context/META.json
3. Read-time aggregation for cross-project queries

## References
- Related recap: [2026-03-24](../2026-03-24.md)
```

## Hooks

| Hook | Trigger | Behavior |
|------|---------|----------|
| **Stop** | Conversation ends | Auto-generates recap + syncs META + git commit + writes pending marker |
| **UserPromptSubmit** | New conversation starts | Recovery check + current project issues + cross-project issues (once per day) |

## WSL2 Note

On WSL2 environments where the Write tool creates files invisible to Windows, the plugin will use Bash heredoc (`cat >`) for file writing.

## License

MIT
