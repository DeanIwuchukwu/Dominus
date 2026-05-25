---
name: compound-phase
description: Session-end ritual to capture architectural decisions and reusable knowledge. Trigger at session end, when the user says "compound", "wrap up", "end of session", or "what did we learn?".
---

# compound-phase

Compound the session's work into the two persistent memory files: `docs/DECISIONS.md` and `docs/KNOWLEDGE.md`.

These files outlive the brief. DECISIONS.md is append-only. KNOWLEDGE.md grows with every session.

## Steps

1. **Review what happened this session** — scan recent changes (git diff, files edited, tasks completed in WORK_BREAKDOWN.md).

2. **Propose DECISIONS.md entries** for architectural choices made this session:
   - What was decided
   - Why (the constraint, trade-off, or evidence that drove it)
   - Alternatives considered and rejected

   Only decisions with lasting implications. Don't log every implementation detail.

3. **Propose KNOWLEDGE.md entries** for patterns worth carrying forward:
   - Patterns that worked
   - Anti-patterns discovered (with the failure mode)
   - Gotchas in the codebase or tooling
   - Measurement hacks or workarounds

4. **Present drafts** to the user. Wait for edits or approval before appending.

5. **Append to the files** once approved:

```markdown
<!-- DECISIONS.md append format -->

## [YYYY-MM-DD] <Decision Title>

**Decision:** What was decided.
**Why:** The constraint or evidence.
**Alternatives:** What was considered and why it lost.
---
```

```markdown
<!-- KNOWLEDGE.md append format -->

## <Pattern / Anti-Pattern Title>

**Context:** When this applies.
**Finding:** What we learned.
**Example:** File:line or brief name where this showed up.
---
```

## What qualifies

**DECISIONS.md — yes:**
- Chose X over Y for performance/correctness/deadline reasons
- Decided to skip Z because the brief doesn't require it
- Architectural constraints discovered (e.g., can't change schema because of migration risk)

**DECISIONS.md — no:**
- Implementation details that are obvious from the code
- Decisions that are trivially reversible

**KNOWLEDGE.md — yes:**
- "tsc --noEmit misses errors in test files unless `include` is set"
- "measure-type-safety.sh must be run from project root, not tools/"
- "this codebase's bundle size is dominated by the editor dependency — don't optimize elsewhere first"

**KNOWLEDGE.md — no:**
- Things already in CLAUDE.md or the architecture docs
- Ephemeral context that won't matter next week

## After appending

Confirm to the user what was added and to which file. Suggest running a quick `git add docs/DECISIONS.md docs/KNOWLEDGE.md && git commit` to preserve the session's learnings.
