---
name: recap-proposal
description: "Create, list, and view design proposals. Extracts solution designs from conversations into structured proposal documents in docs/context/proposals/. Use when user says 'write a proposal', 'design doc', 'save this plan', or invokes '/recap proposal'. Example: '/recap proposal', '/recap proposal list', '/recap proposal 001'."
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# Design Proposals

Create and manage structured design proposals extracted from conversations.

## Arguments

| $ARGUMENTS | Action |
|---|---|
| empty / `proposal` | Create a new proposal from current conversation |
| `list` | List all existing proposals |
| `<number>` (e.g. `001`) | View a specific proposal |

## Create Proposal

1. Run `date '+%Y-%m-%d'`
2. Determine next proposal number:
   ```bash
   ls docs/context/proposals/*.md 2>/dev/null | wc -l
   ```
   Next number = count + 1, zero-padded to 3 digits (001, 002, ...)
3. Review the conversation for design discussions, solution plans, architecture decisions
4. Write to `docs/context/proposals/NNN-<slug>.md`
5. Update `docs/context/META.json` — add filename to `"proposals"` array
6. Update `docs/context/INDEX.md` — add 📝 entry

`mkdir -p docs/context/proposals`

### Proposal Format

```markdown
# Proposal NNN: <Title>

- **Status**: draft
- **Date**: YYYY-MM-DD
- **Author**: <from conversation context or "unknown">

## Problem
Why this change is needed. What problem does it solve.

## Solution
The proposed approach. Be specific and concrete.

## Trade-offs
What alternatives were considered and why they were rejected.

## Implementation
Key steps to implement this proposal.

## References
- Related recap: [YYYY-MM-DD](../YYYY-MM-DD.md)
- Related decisions in DECISIONS.md
```

### Status Values

| Status | Meaning |
|--------|---------|
| `draft` | Initial creation, still being refined |
| `accepted` | Approved for implementation |
| `implemented` | Work completed |
| `rejected` | Decided not to proceed |

## List Proposals

1. Read all files in `docs/context/proposals/`
2. Extract title and status from each
3. Display as table:

```
| # | Title | Status | Date |
|---|-------|--------|------|
| 001 | Cross-Project META | implemented | 2026-03-24 |
| 002 | Progress Tracking | draft | 2026-03-24 |
```

If no proposals exist, say so.

## View Proposal

1. Read `docs/context/proposals/NNN-*.md` (match by number prefix)
2. Display contents
3. If not found, list available proposals

## Rules

- Slug should be lowercase, hyphen-separated, max 5 words (e.g. `cross-project-meta`)
- New proposals always start as `draft`
- Keep Problem section under 3 sentences
- Implementation section should be actionable steps, not vague goals
