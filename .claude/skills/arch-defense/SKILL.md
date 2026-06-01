---
name: arch-defense
description: Own the architecture defense for a brief end-to-end — write ArchDefense.md (design narrative + high-level diagram + key flows) when it doesn't exist yet, then build ArchDefense.pptx from it. Standalone and first: needs only the brief and (for a continuation project) the codebase. Trigger when the user says "arch defense", "architecture defense", "build the defense deck", "make the defense slides", "draft the defense", or is preparing the Monday architecture defense.
---

# arch-defense

Produce the architecture defense for a brief. This is usually the **first**
artifact of the week (the Monday defense) and is **standalone** — it does not
depend on `extract-requirements`, `draft-project`, `decompose-work`, or
`generate-measurements`.

It produces two things:

1. **`ArchDefense.md`** — the written defense (design narrative, high-level
   Mermaid diagram, key sequence diagrams, SOLID rationale, failure modes).
   Created **only if it doesn't already exist**.
2. **`ArchDefense.pptx`** — a speaker-ready deck built from that narrative.

The deck mirrors the defense's own thesis: **depth over breadth, proof over
promises.** Every slide makes one claim backed by a concrete artifact — a file
path, a measurable target, a code shape, or a diagram.

## Inputs

- **An existing `ArchDefense.md`** (project root) — **if present, use it AS-IS.
  Never modify or overwrite it.** Read it and go straight to building the deck.
- **The brief** — `docs/briefs/*.pdf` (or whatever the user points at). The
  source of truth for *what* is being built. Required only when `ArchDefense.md`
  must be drafted from scratch.
- **The codebase** — when this brief is a *continuation of an existing project*,
  inspect the real repo (routes, services, key files) so the defense cites
  concrete artifacts. For a greenfield brief with little code yet, stay at the
  design level and lean on the brief.
- `docs/REQUIREMENTS.md` (optional) — if present, prefer its measurable targets
  verbatim. If absent, use the figures written in `ArchDefense.md` / the brief.

## Steps

1. **Check for `ArchDefense.md`.**
   - **If it exists:** read it, do **not** write to it, and skip to step 3.
   - **If it does not exist:** go to step 2 to draft it.

2. **Draft `ArchDefense.md`** (only when missing). Pull the architecture from the
   brief; if this is a continuation project, ground every claim in the actual
   codebase (real route paths, service names, file locations). Follow the
   structure in "ArchDefense.md structure" below. **Present it to the user for
   review and edits, and only save it to `ArchDefense.md` once they're happy.**
   Do not invent architecture the brief/codebase doesn't support — flag gaps
   instead.

3. **Render the diagrams.** Extract each ```mermaid block from `ArchDefense.md`
   into its own `.mmd` under `arch-defense-build/diagrams/`, then render to PNG
   with the bundled dark theme:

   ```bash
   mkdir -p arch-defense-build/diagrams
   npx -y @mermaid-js/mermaid-cli -i arch-defense-build/diagrams/highlevel.mmd \
       -o arch-defense-build/diagrams/highlevel.png \
       -c <skill_dir>/mermaid.config.json -b transparent -s 3
   ```

   At minimum render the **high-level flowchart** (it earns its own slide). The
   OAuth/webhook sequence diagrams are optional extra `image` slides. If `mmdc`
   can't be installed, fall back to a `pipeline` slide describing the same flow
   as connected boxes, and tell the user the diagram was approximated.

4. **Write `deck.json`.** A list of slide objects (or `{ "brand": "...",
   "slides": [...] }`) following the schema below. `deck.example.json` in this
   skill folder is a complete, runnable model — copy its shape.

5. **Build the deck.**

   ```bash
   pip install python-pptx        # once
   python <skill_dir>/build_deck.py deck.json ArchDefense.pptx
   ```

6. **Report.** Tell the user where `ArchDefense.pptx` landed and how to make a
   PDF if they want one (PowerPoint → Export → PDF, or
   `soffice --headless --convert-to pdf ArchDefense.pptx`).

## ArchDefense.md structure

Adapt these sections to the brief — not every defense has every section, but the
high-level diagram and an opening thesis are mandatory.

1. **Opening Position** — the one core architectural decision, in one paragraph.
2. **High-Level Diagram** — a ```mermaid flowchart of the whole system.
3. **Key Boundary / Design Choice** — the most important structural line and why.
4. **Subsystem sections** — one per major subsystem (auth, data, delivery, SDK…),
   each: what it is, why this design, what it costs, how it fails.
5. **Sequence Diagrams** — ```mermaid sequenceDiagram for the critical flows.
6. **Design Rationale (SOLID / patterns)** — anchored to real file paths.
7. **Failure Modes** — what breaks and how the system degrades.
8. **End-to-End Trace** — "Path: X → Y" naming every interface boundary crossed.
9. **Closing** — the thesis restated as a commitment.

## Slide schema

Top level is a JSON array of slide objects, or `{ "brand": "PLUGFORGE",
"slides": [ ... ] }` (brand shows in the footer). Common optional keys on every
slide: `kicker`, `title`, `subtitle`. Per-type keys:

| `type` | Purpose | Keys |
|--------|---------|------|
| `title` | Opening slide (dark) | `terminal`, `thesis`, `footer` |
| `cards` | 3–4 feature cards + dark callout | `cards: [{title, body}]`, `callout: {title, body}` |
| `pipeline` | Left-to-right boxes + two panels (dark) | `steps: ["A\nB", ...]`, `highlight: <int>`, `panels: [{title, accent:"teal"\|"orange", bullets:[]}]` |
| `two_column` | Two cards w/ numbered steps + "Why" | `columns: [{heading, subhead, accent:"teal"\|"red", steps:[], why}]` |
| `metrics` | Terminal block + big-number chips (dark on light) | `terminal`, `metrics: [{value, label}]`, `callout` |
| `timeline` | Numbered rows (E1…E7) | `items: [{badge, title, body}]` |
| `summary` | Icon rows + thesis footer (dark) | `rows: [{title, body}]`, `footer` |
| `image` | Centered picture (e.g. rendered diagram) | `image: "<png path>"`, `caption`, `dark: true\|false` |

Notes:
- In `steps`/`pipeline` box labels, `\n` forces a line break inside the box.
- A `→` prefix or a `✓` anywhere in a `terminal`/`metrics` line is auto-colored teal.
- `highlight` tints one pipeline box teal (use it for the "payoff" step).

## Suggested deck mapping (from a typical defense)

1. `title` — project + one-line thesis + a verified-outcome terminal line.
2. `cards` — the disciplines / pillars of the design.
3. `image` — **the high-level diagram** (rendered Mermaid flowchart).
4. `pipeline` — request/data lifecycle + boundary panels.
5. `two_column` — the key either/or design choice, each side with a "Why".
6. `cards` or `two_column` — the contract: interfaces, error shape, invariants.
7. `image` — a critical flow (rendered sequence diagram), or a `pipeline` slide.
8. `metrics` — the measurable targets (verbatim from REQUIREMENTS.md / brief).
9. `two_column` (accent `red`/`teal`) — before/after of the headline change.
10. `cards` — design rationale anchored to real file paths.
11. `timeline` — execution plan / epics.
12. `summary` — what you're committing to ship + the thesis as a footer.

## Principles

- **Use an existing `ArchDefense.md` untouched.** Only ever create it when absent,
  and only after the user reviews the draft.
- **Ground claims in artifacts** — real file paths, routes, code shapes — when a
  codebase exists. Prefer "ScopeRegistry — platform/scopes/registry.ts" over
  "well-designed".
- **Numbers must be sourced** — verbatim from REQUIREMENTS.md or the brief/doc.
  If a target isn't written down anywhere, don't put a number on the slide.
- **One claim per slide.** If a slide needs a paragraph, it's two slides.
- **Speaker-ready.** Every slide must be defensible out loud in the room.
- **Don't invent architecture.** The defense reflects the brief + codebase; flag
  gaps to the user rather than fabricating.

## After building

Tell the user: "ArchDefense.pptx is ready — open it in PowerPoint to tweak
wording or reorder slides. Re-run `python build_deck.py deck.json
ArchDefense.pptx` after editing `deck.json` to regenerate. Your ArchDefense.md
was left as-is / saved for review."
