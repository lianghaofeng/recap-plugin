---
name: recap-context
description: "Load project context from recap history. Restores META, progress, decisions, and recent session summaries for continuity across sessions. Use: '/recap context', '/recap context 7', '/recap context full'."
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "1.0.0"
---

# Context Loader — Project Context Recovery

Restores project context from recap history so the agent can resume work with full awareness.

## Arguments

| $ARGUMENTS | Action |
|---|---|
| empty or `3` | Load META + PROGRESS summary + last 3 days recap highlights |
| `<N>` (number) | Load META + PROGRESS summary + last N days recap highlights |
| `full` | Load META + PROGRESS + DECISIONS + last 3 days full recap |

Current arguments: $ARGUMENTS

---

## Execution

### Step 1: Project META

Read `docs/recap_context/META.json` if it exists. Output as:

```
📌 Project: <name>
   Sessions: <total> | Last: <date>
   Recent topics: <topics>
   Open issues: <issues>
```

If META.json doesn't exist, say "No project context found. Run /recap first to initialize."

### Step 2: Progress Snapshot

Read `docs/recap_context/PROGRESS.md` if it exists. Extract only:
- **Current Focus** section (first 10 lines)
- **Next Steps** section (first 10 lines)

Skip if file doesn't exist.

### Step 3: Recent Sessions

Determine N from arguments (default 3).

```bash
ls -1 docs/recap_context/daily/*/*.md 2>/dev/null | sort -r | head -N
```

For each file (newest first):

- **Normal mode** (default): Extract only `### Topics` / `### 讨论内容` and `### Remaining Issues` / `### 遗留问题` sections
- **Full mode** (`full` argument): Read entire file content

### Step 4: Decisions (full mode only)

If argument is `full`, read `docs/recap_context/DECISIONS.md` and show the last 10 decisions.

### Step 5: Agent Activity

If `docs/recap_context/.agent-activity.jsonl` exists, read and display today's entries:

```
🤖 Recent agent activity:
   - [test-runner] 17:31 — completed
   - [Explore] 17:35 — completed
```

---

## Output Format

Present all loaded context in a structured, concise format. Use the information to inform your subsequent responses in this session — do NOT just dump raw file contents.

After loading, confirm: "Context loaded. Ready to continue."
