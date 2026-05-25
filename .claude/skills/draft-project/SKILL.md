---
name: draft-project
description: Draft PROJECT.md from a Gauntlet brief and its extracted requirements. Trigger when the user says "draft project", "write PROJECT.md", or after requirements are extracted for a new brief.
---

# draft-project

Produce `docs/PROJECT.md` — the stable identity document for this week's brief. It should answer "what are we building and why?" in a way that stays true across the whole week.

## Inputs

- `docs/REQUIREMENTS.md` (must exist and be reviewed)
- The brief PDF in `docs/briefs/`
- Any prior `docs/KNOWLEDGE.md` entries that are relevant

## Brief shapes and what to emphasize

Different Gauntlet briefs call for different emphasis in PROJECT.md. Identify the shape first:

| Shape | Signal | Emphasis |
|-------|--------|----------|
| **Audit / improvement** | "reduce X by Y%", inherit a codebase | Baseline state, what's broken, improvement strategy |
| **Greenfield** | "build from scratch", new system | Architecture decisions, user stories, tech choices |
| **ML / data** | models, datasets, evals | Data sources, model choices, eval strategy, success metrics |
| **Integration** | connect systems, APIs | External contracts, data flow, failure modes |

## Output format

```markdown
# Project: <Brief Name>

> Week N | Gauntlet AI Program
> Brief: docs/briefs/<filename>.pdf

## What We're Building

One paragraph. What is this? Who uses it? What problem does it solve?

## Brief Shape

[Audit / Greenfield / ML / Integration] — one sentence on why this classification matters for how we approach the work.

## Success Criteria

Restate the REQ-N targets in plain language. Not the full REQUIREMENTS.md — just the headline numbers and qualitative bars.

## Starting Point

[For audit/improvement briefs]: What is the inherited codebase? What's its current state? What's broken or below target?
[For greenfield]: What are we starting from (any scaffolding, constraints)?
[For ML]: What data do we have? What's the target metric and baseline?
[For integration]: What systems are involved? What contracts must we respect?

## Key Constraints

- Things we must not break
- Performance or compatibility requirements
- Submission deadline and format

## Approach

2–4 sentences on the overall strategy. Not a task list — that's WORK_BREAKDOWN.md.
```

## After writing

Present to the user for review. PROJECT.md is stable; update it only if the brief understanding changes meaningfully.
