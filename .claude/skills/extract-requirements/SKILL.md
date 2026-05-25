---
name: extract-requirements
description: Extract and structure requirements from a Gauntlet brief PDF into REQUIREMENTS.md. Trigger when the user says "extract requirements", "process the brief", hands you a new PDF, or is starting a new weekly brief.
---

# extract-requirements

Turn a Gauntlet brief into a locked contract in `docs/REQUIREMENTS.md`.

## Steps

1. **Locate the brief** — find the PDF in `docs/briefs/`. If none exists, ask the user to drop it there first.

2. **Read thoroughly** — scan for:
   - Explicit acceptance criteria (often in a rubric, checklist, or grading section)
   - Measurable improvement targets (percentages, thresholds, counts)
   - Qualitative requirements (UX, architecture, documentation)
   - Constraints (must preserve, must not break, performance budgets)
   - Ambiguities that need a call before work starts

3. **Assign REQ-N IDs** — one ID per distinct, independently testable requirement. Group related sub-points under a single REQ if they will be verified together.

4. **Draft REQUIREMENTS.md** using the format below. Present it to the user for review and edits before treating it as locked.

## Output format

```markdown
# Requirements

> Source: docs/briefs/<filename>.pdf
> Status: DRAFT — edit before locking

## REQ-1: <Category Name>

**What:** One-sentence description of what must be true.
**Acceptance criteria:** Bulleted list of verifiable conditions.
**Measurable target:** Specific number, percentage, or threshold (e.g., "reduce from X to Y", "≥ N passing").
**Notes:** Ambiguities, assumptions, or open questions.

## REQ-2: ...
```

## Principles

- One REQ per independently verifiable requirement.
- Every REQ must have a measurable target — if the brief is vague, propose a reasonable interpretation and flag it as an assumption.
- Flag anything that looks untestable; propose how to make it testable.
- Do not invent requirements not in the brief.

## After writing

Tell the user: "Review REQUIREMENTS.md. Edit any ambiguities or add missing criteria, then lock it before running `/decompose-work` or `/generate-measurements`."
