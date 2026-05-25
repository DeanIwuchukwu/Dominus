---
name: measure-baseline
description: Capture baseline measurements before work begins. Trigger when the user says "measure baseline", "capture baseline", "establish baseline", or at setup time after generate-measurements.
---

# measure-baseline

Run all measurement scripts and store their output as the baseline. This must happen **before any work on the brief starts** — it's the before-state that all improvement targets are measured against.

## Preconditions

- `tools/measurements/measure-all.sh` exists and is executable
- At least one `tools/measurements/measure-<category>.sh` script exists
- `tools/measurements/targets.json` exists

If any of these are missing, run `/generate-measurements` first.

## Steps

1. **Verify the repo is in the starting state** — confirm with the user that no improvements have been made yet. If work has already started, the baseline will be contaminated.

2. **Run the baseline:**
   ```bash
   cd <project-root>
   bash tools/measurements/measure-all.sh --baseline
   ```

3. **Show the results** — list each `tools/measurements/baselines/<category>.json` file and its key metric value.

4. **Summarize the starting state** in a table:

```
Category        | Metric              | Baseline Value
----------------|---------------------|---------------
type-safety     | totals.total        | 142
bundle          | total_bytes         | 4,821,033
a11y            | pages[*].score avg  | 61.3
```

5. **Log to DECISIONS.md** (brief entry):
   ```
   [YYYY-MM-DD] Baseline captured
   Values: [paste summary table]
   ```

## What to do if a script fails

- Check that the script's dependencies are available (pnpm build done? Dev server running if needed?)
- Fix the script before proceeding — a missing baseline for a category means that category can't be scored
- Re-run just that script: `bash tools/measurements/measure-type-safety.sh > tools/measurements/baselines/type-safety.json`

## After capturing baseline

Tell the user: "Baseline captured. Do not re-run `--baseline` again for this brief — it will overwrite the starting point. Use `/measure-current` to check progress."
