# Task Plan: Restructure Recap File Storage (Revised)

## Goal
Add YYYY subdirectories for high-volume file types (daily, weekly). Keep low-volume types flat.

## New Directory Structure
```
docs/recap_context/
├── daily/
│   └── YYYY/
│       └── YYYY-MM-DD.md
├── weekly/
│   └── YYYY/
│       └── YYYY-WXX.md
├── monthly/
│   └── YYYY-MM.md              ← unchanged
├── proposals/
│   └── NNN-slug.md             ← unchanged
├── META.json
├── PROGRESS.md
├── DECISIONS.md
├── INDEX.md
└── .agent-activity.jsonl
```

## Key Pattern Changes

### Daily
| Old | New |
|---|---|
| `docs/recap_context/YYYY-MM-DD.md` | `docs/recap_context/daily/YYYY/YYYY-MM-DD.md` |
| `docs/recap_context/$(date +%Y-%m-%d).md` | `docs/recap_context/daily/$(date +%Y)/$(date +%Y-%m-%d).md` |
| `ls -1 docs/recap_context/????-??-??.md` | `ls -1 docs/recap_context/daily/*/*.md 2>/dev/null` |
| `mkdir -p docs/recap_context/weekly ...` | add `mkdir -p docs/recap_context/daily/$(date +%Y)` |

### Weekly
| Old | New |
|---|---|
| `docs/recap_context/weekly/YYYY-WXX.md` | `docs/recap_context/weekly/YYYY/YYYY-WXX.md` |
| `mkdir -p docs/recap_context/weekly` | `mkdir -p docs/recap_context/weekly/$(date +%Y)` |

## Phases

### Phase 1: hooks/hooks.json [pending]
- [ ] Stop hook: daily file path + mkdir
- [ ] UserPromptSubmit hook: ls glob + latest file detection

### Phase 2: Core skills [pending]
- [ ] skills/recap/SKILL.md
- [ ] skills/recap-zh/SKILL.md

### Phase 3: Context loader skills [pending]
- [ ] skills/recap-context/SKILL.md
- [ ] skills/recap-context-zh/SKILL.md

### Phase 4: Aggregation skills [pending]
- [ ] skills/recap-weekly/SKILL.md + weekly/YYYY path
- [ ] skills/recap-weekly-zh/SKILL.md
- [ ] skills/recap-monthly/SKILL.md (daily read path only)
- [ ] skills/recap-monthly-zh/SKILL.md

### Phase 5: Search skills [pending]
- [ ] skills/recap-search/SKILL.md
- [ ] skills/recap-search-zh/SKILL.md

### Phase 6: Project/other skills [pending]
- [ ] skills/recap-projects/SKILL.md — lastRecapFile
- [ ] skills/recap-projects-zh/SKILL.md

### Phase 7: Documentation [pending]
- [ ] README.md
- [ ] ARCHITECTURE.md

### Phase 8: Sync installed plugin [pending]
- [ ] Copy all to ~/.claude/plugins/marketplaces/recap/
