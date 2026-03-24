---
name: recap-search
description: "Search through historical session recaps by keyword. Use when user says 'search recaps', 'find in history', or invokes '/recap search <query>'. Example: '/recap search authentication', '/recap search database migration'."
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Recap Search

Search through historical session recaps for specific topics or keywords.

## Arguments

The search query is everything after "search " in $ARGUMENTS.

## Steps

1. Extract the search query from arguments
2. Search across all recap files:
   ```bash
   grep -r -i -n "<query>" docs/context/ --include="*.md" | head -30
   ```
3. Also search META.json for topic matches:
   ```bash
   grep -i "<query>" docs/context/META.json 2>/dev/null
   ```
4. Present results grouped by date, with surrounding context
5. Summarize: which sessions discussed the topic, key points found

## Output Format

```
### Search Results for "<query>"

**Found in N files:**

#### YYYY-MM-DD
- [Session 2] Discussed <topic> in context of...
- Relevant excerpt: "..."

#### YYYY-MM-DD
- [Session 1] Related to <topic> when...

**Summary:** The topic was discussed across N sessions, mainly around...
```

If no results found, say so and suggest alternative search terms.
