# Dominus

A Claude Code plugin providing a repeatable workflow for Dominus AI.

## Install

```
/plugin install DeanIwuchukwu/Dominus
```

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `/extract-requirements` | New brief PDF | Brief PDF → `docs/REQUIREMENTS.md` with REQ-N IDs |
| `/draft-project` | After requirements | REQUIREMENTS.md → `docs/PROJECT.md` (what + why) |
| `/decompose-work` | After project draft | REQUIREMENTS.md → `docs/WORK_BREAKDOWN.md` with T-N.M task IDs |
| `/generate-measurements` | After decompose | REQUIREMENTS.md → `targets.json` + `measure-*.sh` scripts + full harness |
| `/measure-baseline` | Before any work | Capture starting state — run once, never again |
| `/measure-current` | After each category | Run scripts + compare to baseline, report PASS/FAIL |
| `/audit-requirements` | Before submission | Subagent grades each REQ-N with evidence |
| `/compound-phase` | Every session end | Propose entries for `DECISIONS.md` and `KNOWLEDGE.md` |

## Workflow

```
Planning
  1. Drop brief → docs/briefs/week-N-name.pdf
  2. /extract-requirements
  3. /draft-project
  4. /decompose-work
  5. /generate-measurements
  6. /measure-baseline   ← do this before any code changes

Facilitate Execution 
  - Pick T-N.M tasks from WORK_BREAKDOWN.md
  - Implement → /measure-current to check progress
  - /compound-phase at the end of each session

Verify
  - /audit-requirements
```

## What `/generate-measurements` creates

On first run in a new project it bootstraps the full measurement harness:

- `tools/measurements/measure-all.sh` — runs all measure-*.sh scripts
- `tools/measurements/compare-to-baseline.py` — compares current vs baseline
- `tools/measurements/baselines/` — baseline snapshots
- `tools/measurements/current/` — current run output
- `tools/measurements/targets.json` — brief-specific metric targets
- `tools/measurements/measure-<category>.sh` — one per measurable REQ

## Memory files

| File | Purpose |
|------|---------|
| `docs/PROJECT.md` | What this project is — stable |
| `docs/REQUIREMENTS.md` | Acceptance criteria — locked once written |
| `docs/WORK_BREAKDOWN.md` | T-N.M task list — updated as work progresses |
| `docs/DECISIONS.md` | Architectural choices + rationale — append-only |
| `docs/KNOWLEDGE.md` | Patterns, anti-patterns, gotchas — grows each week |
