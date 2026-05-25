---
name: generate-measurements
description: Generate measurement scripts and targets.json from REQUIREMENTS.md. Trigger when the user says "generate measurements", "create measurement scripts", "set up metrics", or after requirements are finalized for a new brief.
---

# generate-measurements

Produce the full measurement infrastructure for a project:

**Generic harness (bootstrapped once):**
- `tools/measurements/measure-all.sh` — orchestrator that runs all measure-*.sh scripts
- `tools/measurements/compare-to-baseline.py` — compares current/ vs baselines/ using targets.json
- `tools/measurements/baselines/` — directory for baseline snapshots
- `tools/measurements/current/` — directory for current measurement runs

**Brief-specific (generated each week):**
- `tools/measurements/targets.json` — per-category metric targets
- `tools/measurements/measure-<category>.sh` — one script per measurable REQ

## Inputs

- `docs/REQUIREMENTS.md` — source of truth for what to measure
- The codebase — to understand what's actually measurable (run counts, grep patterns, build output, etc.)

## Step 0: Bootstrap the generic harness

Check whether `tools/measurements/measure-all.sh` exists. If it does not, create the full harness now. If it already exists, skip this step.

Create `tools/measurements/` and write these four items:

**`tools/measurements/measure-all.sh`:**
```bash
#!/usr/bin/env bash
# Generic measurement orchestrator.
# Usage: bash tools/measurements/measure-all.sh [--baseline]
#
# Without --baseline: writes to tools/measurements/current/
# With --baseline:    writes to tools/measurements/baselines/

set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
SCRIPTS_DIR="$SCRIPT_DIR"

BASELINE_MODE=false
if [[ "${1:-}" == "--baseline" ]]; then
  BASELINE_MODE=true
fi

if $BASELINE_MODE; then
  OUTPUT_DIR="$SCRIPT_DIR/baselines"
  echo "==> Capturing BASELINE measurements"
else
  OUTPUT_DIR="$SCRIPT_DIR/current"
  echo "==> Capturing CURRENT measurements"
fi

mkdir -p "$OUTPUT_DIR"

PASS=0
FAIL=0

for script in "$SCRIPTS_DIR"/measure-*.sh; do
  [[ -f "$script" ]] || continue
  name=$(basename "$script" .sh | sed 's/^measure-//')
  output_file="$OUTPUT_DIR/${name}.json"
  printf "  %-20s " "$name"
  if bash "$script" > "$output_file" 2>/tmp/measure-err; then
    echo "OK -> $output_file"
    PASS=$((PASS + 1))
  else
    echo "FAILED (exit $?)"
    cat /tmp/measure-err | sed 's/^/    /'
    FAIL=$((FAIL + 1))
  fi
done

echo ""
echo "Done: $PASS ok, $FAIL failed"
if [[ $FAIL -gt 0 ]]; then
  exit 1
fi
```

**`tools/measurements/compare-to-baseline.py`:**
```python
#!/usr/bin/env python3
"""Compare measurements in baselines/ vs current/ using targets.json (all under tools/measurements/)."""

import json
import sys
from pathlib import Path


def load_json(path):
    with open(path) as f:
        return json.load(f)


def extract(data, metric_path):
    """Extract a value using a dot-path that supports [*] to collect list items."""
    parts = metric_path.split(".")
    current = data
    for part in parts:
        if "[*]" in part:
            key = part.replace("[*]", "")
            if key:
                current = current[key]
            if not isinstance(current, list):
                raise ValueError(f"Expected list at '{key}', got {type(current)}")
            return current
        else:
            current = current[part]
    return current


def scalar(val):
    if isinstance(val, list):
        return sum(val) / len(val) if val else 0
    return float(val)


def main():
    root = Path(__file__).parent
    targets_file = root / "targets.json"
    baselines_dir = root / "baselines"
    current_dir = root / "current"

    if not targets_file.exists():
        print("ERROR: tools/measurements/targets.json not found")
        sys.exit(1)

    targets = load_json(targets_file)
    all_pass = True

    col = 22
    for category, target in targets.items():
        baseline_file = baselines_dir / f"{category}.json"
        current_file = current_dir / f"{category}.json"

        label = f"{category}:"

        if not baseline_file.exists():
            print(f"  SKIP  {label:<{col}} no baseline captured")
            continue
        if not current_file.exists():
            print(f"  SKIP  {label:<{col}} no current measurement")
            continue

        try:
            b_raw = extract(load_json(baseline_file), target["metric"])
            c_raw = extract(load_json(current_file), target["metric"])
        except (KeyError, TypeError, ValueError) as e:
            print(f"  ERROR {label:<{col}} could not extract '{target['metric']}': {e}")
            continue

        b = scalar(b_raw)
        c = scalar(c_raw)

        if "reduce_pct" in target:
            pct_target = target["reduce_pct"]
            actual = (1 - c / b) * 100 if b else 0
            passed = actual >= pct_target
            detail = f"{actual:.1f}% reduction  (target: ≥{pct_target}%,  {b:.0f} → {c:.0f})"

        elif "raise_pts" in target:
            pts_target = target["raise_pts"]
            actual = c - b
            passed = actual >= pts_target
            detail = f"+{actual:.1f} pts  (target: ≥+{pts_target},  {b:.1f} → {c:.1f})"

        elif "fix_count" in target:
            count_target = target["fix_count"]
            actual = b - c
            passed = actual >= count_target
            detail = f"fixed {actual:.0f}  (target: ≥{count_target},  {b:.0f} → {c:.0f})"

        else:
            print(f"  ERROR {label:<{col}} unknown target type in targets.json")
            continue

        status = "PASS" if passed else "FAIL"
        if not passed:
            all_pass = False

        print(f"  {status}  {label:<{col}} {detail}")

    print()
    print("OVERALL:", "PASS" if all_pass else "FAIL")
    return 0 if all_pass else 1


if __name__ == "__main__":
    sys.exit(main())
```

Also create empty `tools/measurements/baselines/.gitkeep` and `tools/measurements/current/.gitkeep` so git tracks the directories.

Confirm to the user: "Harness bootstrapped — measure-all.sh, compare-to-baseline.py, baselines/, current/ are ready."

## Step 1: Identify measurable categories

For each REQ-N, determine:
1. **What** is being measured (TypeScript errors, bundle size, query count, test coverage, etc.)
2. **How** it can be measured mechanically (tsc output, webpack stats, SQL EXPLAIN, lighthouse, etc.)
3. **Target direction**: reduce (lower is better), raise (higher is better), fix (count of issues to eliminate)

## Step 2: Write targets.json

```json
{
  "<category>": {
    "metric": "<dot-path into the JSON the script outputs>",
    "reduce_pct": 25
  }
}
```

Target types (pick one per category):
- `"reduce_pct": N` — reduce the metric by N% from baseline
- `"raise_pts": N` — raise the metric by N points from baseline
- `"fix_count": N` — reduce the raw count by exactly N

## Step 3: Write measure-<category>.sh

Each script must:
1. Run the measurement command(s)
2. Output **valid JSON to stdout** matching the metric path in targets.json
3. Exit 0 on success, non-zero on hard failure (not on bad metric — that's for compare to decide)

### Script template

```bash
#!/usr/bin/env bash
# Measure <category> — generated for REQ-N
set -euo pipefail

# Run the measurement tool
# Parse/transform output
# Emit JSON

echo '{
  "total": 42,
  "details": [...]
}'
```

### Common measurement patterns

**TypeScript errors:**
```bash
count=$(pnpm tsc --noEmit 2>&1 | grep -c "error TS" || true)
echo "{\"total\": $count}"
```

**Bundle size:**
```bash
pnpm build --json 2>/dev/null | jq '{total_bytes: .assets | map(.size) | add}'
```

**Test coverage:**
```bash
pnpm test --coverage --reporter=json 2>/dev/null | jq '{lines_pct: .total.lines.pct}'
```

**Query count (via EXPLAIN):**
```bash
# Run the app scenario and capture pg logs, or use a test harness
# Emit: {"flows": [{"name": "flow-name", "query_count": N}]}
```

**Accessibility (Lighthouse):**
```bash
# Run lighthouse programmatically
# Emit: {"pages": [{"url": "...", "score": N}]}
```

## After writing

Tell the user: "Review `targets.json` — adjust percentages if the brief specifies different targets. Then run `/measure-baseline` to capture starting state before any work begins."
