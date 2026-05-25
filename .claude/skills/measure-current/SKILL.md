---
name: measure-current
description: Run current measurements and compare against baseline. Trigger when the user says "measure current", "check progress", "how are we doing on metrics", "did we hit the target?", or after completing a feature category.
---

# measure-current

Run all measurement scripts and compare results against the captured baseline. Reports PASS/FAIL per category based on `targets.json`.

## Preconditions

- `tools/measurements/baselines/` contains at least one JSON file (baseline was captured)
- `tools/measurements/measure-all.sh` and `tools/measurements/compare-to-baseline.py` are present

## Steps

1. **Run current measurements:**
   ```bash
   cd <project-root>
   bash tools/measurements/measure-all.sh
   ```

2. **Compare to baseline:**
   ```bash
   python tools/measurements/compare-to-baseline.py
   ```

3. **Report results** — show the comparison output clearly:

```
PASS  type-safety:  31% reduction  (target: 25%)
FAIL  bundle:        8% reduction  (target: 15%)
PASS  a11y:         +14 pts       (target: +10)

OVERALL: FAIL
```

4. **For each FAIL**, diagnose and propose next steps:
   - What's the current value vs. target?
   - What's the most likely lever to close the gap?
   - Which T-N.M task should be picked up next?

## Partial runs (single category)

If only one category was worked on, you can run and compare just that one:
```bash
bash tools/measurements/measure-type-safety.sh > tools/measurements/current/type-safety.json
python tools/measurements/compare-to-baseline.py
```

## Interpreting results

- **PASS** — metric moved enough in the right direction. Don't stop at exactly the target if there's room to push further.
- **FAIL** — metric didn't move enough. Investigate whether the improvement was real or hidden by noise.
- **SKIP** — no baseline or no current file for that category. Fix the script, not the result.

## After a full PASS

Suggest running `/audit-requirements` to verify qualitative requirements before declaring the brief complete.
