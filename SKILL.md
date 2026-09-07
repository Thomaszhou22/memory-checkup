---
name: memory-checkup
description: Audit and repair an AI agent's memory corpus for consistency. Use when memory contains stale numbers or superseded facts, when different memory files contradict each other (e.g. old download/star counts), when a metric changed and documents must be synced, when daily memory flushes have gaps or stopped, when memory references files that no longer exist, when the user says "check my memory", "memory audit", "体检记忆", or when answering from memory feels risky because facts may have drifted.
---

# Memory Checkup

Audit an agent's memory corpus (MEMORY.md, USER.md, and files under memory/) and surface consistency problems before they reach the user: stale superseded numbers, contradictions, dangling file references, abandoned daily entries, and orphan files.

## When to run

1. The user asks to check/audit/clean memory ("体检", "check my memory").
2. Before answering questions that depend on facts recorded over many days (awards, counts, project statuses).
3. After a metric changes and the user asks to sync all documents (e.g. download counts in application essays).
4. Periodically, alongside daily memory flushes.

## Procedure

### Step 1: Locate the corpus

The corpus root is the agent workspace: it must contain `MEMORY.md` and/or a `memory/` directory. Ask or infer from context; in OpenClaw this is the workspace directory.

### Step 2: Run the audit script

```bash
python3 scripts/memory_checkup.py --memory-dir <workspace-root> [--json]
```

The script scans and reports:

| Check | Meaning |
|---|---|
| Stale numbers | Same metric (downloads/stars/users) with an old value in some files and a newer value in newer files |
| Dangling references | File paths mentioned in memory that no longer exist |
| Stale entries | Daily-flush files whose date has a gap to the newest entry |
| Orphans | memory/ files never referenced from anywhere else |

### Step 3: Verify before editing

For every stale-number finding, confirm the current value with the user or with live evidence (run the real check: hit the API, count the files). Never assume the newest mention is correct just because it is newest.

### Step 4: Fix

- Update stale mentions, or annotate them as superseded if history should be preserved ("旧口径 4,600+ 已作废").
- Remove or repair dangling references.
- Do NOT delete orphan daily files; list them to the user and let them decide.
- After edits, re-run the audit to confirm zero issues.

### Step 5: Report

Summarize in the user's language: how many files scanned, issues by type, what was fixed, what needs a user decision.

## Notes

- The script is read-only. All fixes are made by the agent, file by file, after verification.
- Number extraction currently covers downloads/stars/users patterns (English and Chinese); extend `DEFAULT_PATTERNS` for other metrics.
- JSON output is machine-readable for piping into other tools.
