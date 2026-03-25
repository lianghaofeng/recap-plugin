# Findings

## Files Requiring Changes

### High Impact (path logic)
1. **hooks/hooks.json** — 3 hooks, 2 need path updates (Stop, UserPromptSubmit)
2. **skills/recap/SKILL.md** — main EN skill, ~10 path references
3. **skills/recap-zh/SKILL.md** — main ZH skill, ~10 path references

### Medium Impact (read paths)
4. **skills/recap-context/SKILL.md** — ls glob for daily files
5. **skills/recap-context-zh/SKILL.md** — same
6. **skills/recap-weekly/SKILL.md** — reads daily files for aggregation
7. **skills/recap-weekly-zh/SKILL.md** — same
8. **skills/recap-monthly/SKILL.md** — reads daily files
9. **skills/recap-monthly-zh/SKILL.md** — same

### Low Impact (example strings only)
10. **skills/recap-projects/SKILL.md** — lastRecapFile example
11. **skills/recap-projects-zh/SKILL.md** — same
12. **skills/recap-search/SKILL.md** — grep -r (recursive, still works but path could be more specific)
13. **skills/recap-search-zh/SKILL.md** — same

### Documentation Only
14. **README.md** — directory structure diagram
15. **ARCHITECTURE.md** — path references in diagrams

### No Changes Needed
- skills/recap-progress/SKILL.md — only reads PROGRESS.md, META.json
- skills/recap-progress-zh/SKILL.md — same
- skills/recap-proposal/SKILL.md — already uses proposals/ subdirectory
- skills/recap-proposal-zh/SKILL.md — same

## Key Decisions
- Use `daily/YYYY/MM/` not just `YYYY/MM/` to be explicit and consistent with weekly/monthly/proposals naming
- Keep `find ... -name` instead of deep glob for listing daily files — more robust across shells
- grep -r still works recursively from `docs/recap_context/` — no narrowing needed
- `.agent-activity.jsonl` stays in root recap_context/ — it's not a daily file
