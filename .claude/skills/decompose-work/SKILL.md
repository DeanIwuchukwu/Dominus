---
name: decompose-work
description: Decompose REQUIREMENTS.md into a task breakdown with T-N.M trace IDs. Trigger when the user says "decompose work", "break down tasks", "create work breakdown", or "write WORK_BREAKDOWN.md".
---

# decompose-work

Produce `docs/WORK_BREAKDOWN.md` — a living task plan where every task has a `T-N.M` ID that Superpowers can reference.

## ID scheme

```
T-{REQ}-{TASK}
```

- `REQ` = the REQ-N number from REQUIREMENTS.md (e.g., `1`, `2`)
- `TASK` = sequential task within that requirement (e.g., `1`, `2`, `3`)
- Example: `T-1.3` = third task under REQ-1

## Inputs

- `docs/REQUIREMENTS.md` — locked, read it fully
- `docs/PROJECT.md` — for context on the brief shape
- Codebase (if inherited) — scan for relevant files to understand scope

## Decomposition rules

- Each task should be completable in one focused session (half-day or less)
- Tasks should be independently testable (a PR, a passing test, a measurement delta)
- If a task depends on another, note it explicitly
- Include setup tasks (e.g., "establish baseline", "scaffold test harness")
- Include verification tasks (e.g., "measure-current for REQ-2", "audit-requirements")

## Output format

```markdown
# Work Breakdown

> Source: docs/REQUIREMENTS.md
> Updated: YYYY-MM-DD

## REQ-1: <Category Name>

| ID | Task | Status | Notes |
|----|------|--------|-------|
| T-1.1 | <Setup / scaffold task> | TODO | |
| T-1.2 | <Core implementation task> | TODO | Depends on T-1.1 |
| T-1.3 | <Verification: measure-current for REQ-1> | TODO | |

## REQ-2: <Category Name>

| ID | Task | Status | Notes |
|----|------|--------|-------|
| T-2.1 | ... | TODO | |

---

## Cross-cutting tasks

| ID | Task | Status | Notes |
|----|------|--------|-------|
| T-0.1 | measure-baseline (all categories) | TODO | Do before any REQ work |
| T-0.2 | audit-requirements (pre-submission) | TODO | Do last |
| T-0.3 | compound-phase (each session end) | TODO | Recurring |
```

## Status values

`TODO` → `IN PROGRESS` → `DONE` | `BLOCKED`

## After writing

Tell the user: "Update task status in WORK_BREAKDOWN.md as you work. Reference T-N.M IDs in commit messages and task plans for traceability."
