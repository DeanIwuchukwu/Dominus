---
name: audit-requirements
description: Audit the codebase against REQUIREMENTS.md using a subagent grader. Trigger when the user says "audit requirements", "check against spec", "run the grader", "am I done?", or before submission.
---

# audit-requirements

Dispatch a subagent to grade the current codebase against every REQ-N in REQUIREMENTS.md. Returns a PASS / PARTIAL / FAIL verdict per requirement with evidence.

## When to run

- After completing a category (quick check on that REQ)
- Before submission (full audit across all REQs)
- Anytime the user wants a reality check on completeness

## Steps

1. **Read REQUIREMENTS.md** — collect all REQ-N IDs and their acceptance criteria.

2. **Gather artifacts** for the subagent:
   - REQUIREMENTS.md (full text)
   - Relevant source files per REQ
   - Current measurement outputs from `tools/measurements/current/` (if available)
   - Any recent test output

3. **Spawn the grader subagent** with this prompt:

```
You are an independent requirements auditor with no prior context about this project.

## Task
Review the provided artifacts and grade each requirement. Be strict.

## Requirements
[paste REQUIREMENTS.md content]

## Artifacts
[paste relevant files and measurement output]

## Grading format — one block per REQ:

REQ-1: <name>
Evidence: [what you found — specific file:line references or metric values]
Verdict: PASS | PARTIAL | FAIL
Gap: [what's missing or below target — one sentence, or "none" if PASS]
```

4. **Report results** to the user with a summary table:

```
REQ-1: PASS
REQ-2: PARTIAL — type error count reduced 18% but target was 25%
REQ-3: FAIL — no evidence of X implemented
```

5. **For each PARTIAL or FAIL**, propose the specific work needed to close the gap. Reference T-N.M IDs from WORK_BREAKDOWN.md if they exist.

## Principles

- The subagent must evaluate evidence, not intent. "I planned to" is not evidence.
- Measurement outputs are treated as ground truth for quantitative REQs.
- Qualitative REQs require source file evidence with specific line references.
- A PARTIAL verdict means meaningful progress but not meeting the acceptance bar.
