---
name: dls-color-scale
description: >
  Generate primitive color tonal scales from a brand hex or RGB input, write them
  to code tokens, and push variable values to Figma.
  Triggers: "generate color scale", "dls-color-scale", "create color palette",
  "create color scale", "brand color tokens", "primitive color tokens",
  "generate palette from hex", "add color to DLS".
---

# dls-color-scale

Generate a full tonal scale (steps 50–950) for one or more color families. Each family accepts one or more anchor values; pinned values lock their exact step, unpinned values are placed at the step whose generated color they match most closely. Writes primitive tokens to code and pushes to the Figma variable collection defined in the bridge.

> Global invariants: [README-agent.md](../../../README-agent.md)
> If `schema.md` does not exist, stop and route to `dls-new`.
> If the Figma bridge for primitive colors is missing, stop and route to `dls-config`.

---

## Route: help

**Before anything else**, check the user's raw message (trim whitespace, lowercase).

- If the result is exactly `help`:
  1. Read `.cursor/skills/dls-color-scale/references/help.md`.
  2. Display its content to the user verbatim.
  3. **Stop here.** Do not load memory, do not run any workflow step, do not read any project files.

---

## When to use

- Generating a new primitive color palette from a brand hex or RGB.
- Replacing an existing primitive palette after a rebrand.
- Adding a new neutral, accent, or semantic color family.

## Stop and escalate

- If the Figma bridge for `Primitive colors` is not set in `schema.md`, stop: "Bridge is missing for Primitive colors. Run `dls-config` to set the collection name and file key, then re-run `dls-color-scale`."
- If `tokens.md` does not have a primitive color path, stop and ask the user to specify the code file path before continuing.

---

## Workflow

### Step 0 — Load memory

If `.cursor/skills/dls-color-scale/MEMORY.md` exists, read it now — both sections:
- **`## Summary`** — condensed learnings from all older runs; treat these as standing guidance for this skill.
- **`## Recent Runs`** — the most recent entries; apply any **Improve next time** notes to your approach for this run.

If the file does not exist, skip and proceed to Step 1.

---

### Step 1 — Gather inputs

Ask:
1. Color family name(s) (e.g. `red`, `brand`, `neutral`)? Used as the token prefix. Accept one or multiple families in a single run.
2. For each family: one or more color values (`#rrggbb` or `rgb(r,g,b)`), each optionally paired with a step number.
   - **Pinned** — value + explicit step (e.g. `#FAFAFA at 50`, `#111111 at 950`): that hex is locked to that step exactly.
   - **Unpinned** — value only (e.g. `#DDDDDD`): the agent places it at the best-matching step after generating the scale (see Step 2).
3. Which steps to generate? (default: 50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950)
4. Should I push to Figma now, or just write code tokens?

> **Scale direction is fixed:** step 50 is always the lightest, step 950 is always the darkest.

**Early placement notice for unpinned values**

After the user answers all four questions, and before generating the scale, for every unpinned value:

1. Convert the hex to HSL.
2. Map its lightness onto the default step curve (approximate reference: step 50 ≈ L 97%, step 500 ≈ L 50%, step 950 ≈ L 5%) to estimate the nearest step.
3. Tell the user immediately:
   > "`{hex}` will be placed at approximately **step {N}** (L≈{L}%) by lightness fit. Pin it to a different step, or proceed with best-fit?"
4. Wait for the user's response before continuing to Step 2. If they specify a step, treat it as a pinned anchor going forward.

This lets the user redirect early — before any scale computation — if the estimated placement is not what they expected.

---

### Step 2 — Generate scale

For each color family:

**2a — Build the initial scale using pinned anchors**

- Sort all pinned values by step (ascending).
- For the range between consecutive pinned steps, interpolate lightness (HSL) so the curve passes exactly through both endpoints.
- For the range below the lowest pinned step, extrapolate lightness toward white (step 50 = lightest).
- For the range above the highest pinned step, extrapolate lightness toward black (step 950 = darkest).
- Preserve hue and saturation throughout; apply slight saturation correction at extremes for perceptual quality.

**2b — Place unpinned values**

For each unpinned value, find the step in the generated scale whose hex is closest by perceptual distance (ΔECMC or simple HSL-lightness proximity). Treat that step as a pinned anchor, regenerate the affected segment of the scale (so the curve still passes through the newly pinned point), and annotate the step with `← best fit` in the preview.

If two unpinned values resolve to the same step, flag the conflict and ask the user which one to keep (or whether to override one to an adjacent step).

**2c — Present and confirm**

Output the full scale as hex values before writing anything:
```
red-50:  #fafafa  ← your input (pinned)
red-100: #f0f0f0
...
red-400: #dddddd  ← best fit (unpinned)
...
red-700: #777777  ← best fit (unpinned)
...
red-950: #111111  ← your input (pinned)
```

Ask: "Does this scale look right? Proceed to write to code (and Figma if you chose that)?"

---

### Step 3 — Write to code

1. Read `tokens.md` for the primitive color code path.
2. Write or update the token file at that path. Format matches the project's token file format (CSS custom properties, JS/TS object, or JSON — read from `schema.md`).
3. Example (CSS):
   ```css
   --color-red-50: #fff5f5;
   --color-red-100: #ffe3e3;
   /* ... */
   --color-red-950: #1c0606;
   ```
4. If the file already has a `red` scale, present the diff and confirm before overwriting.

---

### Step 4 — Write to Figma (if confirmed)

1. Load the `figma-use` skill.
2. Read the bridge from `schema.md`: file key and collection name for `Primitive colors`.
3. For each color step, create or update a variable in the collection:
   - Variable name: `{family}/{step}` (e.g. `red/500`)
   - Type: `COLOR`
   - Value: hex or RGBA
4. If a variable already exists with a different value, list the conflict and confirm before overwriting.
5. Confirm the full batch before executing any Figma write.

---

### Step 5 — Update tokens.md

Add or update the entry for this color family in `tokens.md` §4 (Figma collection mapping) and §3 (code paths) if not already present. Note all anchor steps (pinned and best-fit) and the scale algorithm used in a comment.

---

### Step 6 — Update showcase

After writing to code (and optionally Figma), update the Storybook token gallery to include the new or updated primitive color scale.

1. Load the `dls-showcase` skill (`.cursor/skills/dls-showcase/SKILL.md`).
2. Scope the run to **token gallery only** — specifically the color token category that was just written. Skip the component story audit and visual parity steps.
3. Follow `dls-showcase` Step 5 ("Token showcase") for the color family(ies) added this run:
   - Create or update the relevant story file (e.g. `src/stories/Tokens/Colors.stories.tsx`) to include swatches for every new step (`{family}-50` → `{family}-950`).
   - Import values from the token file written in Step 3 — do not hardcode hex values in the story.
   - Use the token naming convention from `schema.md`.
4. Run `dls-showcase` Step 8 (retro) so that skill's `MEMORY.md` is updated.

> If the Storybook directory does not exist or cannot be determined from `schema.md`, note this as a `Known gap` in the retro and skip this step rather than blocking the overall run.

---

### Step 7 — Verify completion

Before writing the retro, confirm all of the following:

- [ ] Token file contains entries for all requested steps in the correct format
- [ ] Every pinned value in the written file exactly matches the input hex at its declared step
- [ ] Every unpinned value is placed at the step chosen in Step 2b, and confirmed by the user
- [ ] No two adjacent steps share the same hex value (plateau = algorithm failure)
- [ ] No step is pure white (`#ffffff`) or pure black (`#000000`) unless explicitly approved by the user
- [ ] `tokens.md` is updated with the color family code path and Figma mapping
- [ ] (If Figma confirmed in Step 1) All steps exist in the specified collection, named `{family}/{step}`
- [ ] Showcase story for the color family is created or updated (or gap documented)

If any item fails, fix it before proceeding. If it cannot be fixed (e.g. Figma MCP unavailable), record it as a `Known gap` bullet in the retro instead.

---

### Step 8 — Retro

Write to `.cursor/skills/dls-color-scale/MEMORY.md`:

Prepend a new entry under `## Recent Runs`:

```
### {date}
- **Went well:** {what worked smoothly — be specific}
- **Friction:** {what caused delays, errors, or extra back-and-forth}
- **Improve next time:** {one concrete change to make on the next run}
- **Summary:** {date} — Generated scale(s): {family names}. Steps: {count}. Code: written. Figma: {written | skipped}. Showcase: {updated | skipped — reason}. Conflicts: {count}.
```

**Compaction rule:** if `## Recent Runs` exceeds 20 entries, condense the oldest 15 into a prose paragraph under `## Summary` (merge with any existing Summary), and retain only the 5 most recent entries in `## Recent Runs`.
Create `MEMORY.md` with `## Summary` and `## Recent Runs` headers if the file does not exist yet.

---

## Outputs

| Artifact | Action |
|---|---|
| Primitive color token file (code) | Created / updated |
| Figma variable collection entries | Created / updated (if confirmed) |
| `tokens.md` | Updated (code paths + Figma mapping) |
| Storybook token gallery story | Created / updated (color family swatches) |
| `.cursor/skills/dls-color-scale/MEMORY.md` | Updated |
| `.cursor/skills/dls-showcase/MEMORY.md` | Updated |
