---
name: create-color-scale
description: >
  Generate a tonal color scale (steps 50–950) from a hex or RGB input and push
  it as COLOR variables to a Figma file. No code tokens, no showcase, no external
  config files needed — just color input and a Figma destination.
  Triggers: "create color scale", "generate color scale", "add color to Figma",
  "create palette in Figma", "push color variables to Figma", "create color variables".
---

# create-color-scale

Generate a full tonal scale (steps 50–950) for one or more color families and write them as `COLOR` variables directly into a Figma file. Self-contained — no schema files, no code output, no retro.

---

## Input parsing

Before asking any questions, scan the user's opening message for:

- A `figma.com` URL → extract the `fileKey` from it now.
- A variable collection name (e.g. `"Primitives"`, `"color/base"`) → note it.
- A group prefix (e.g. `"color"`) → note it.
- Color family names and hex/RGB values → note them.

Any fields already present skip their corresponding ask below.

---

## Step 1 — Color input

Ask the user (for any fields not already provided):

1. **Color family name(s)** — e.g. `red`, `brand`, `neutral`. Multiple families can be submitted in one run.
2. **Color value(s) per family** — one or more `#rrggbb` or `rgb(r,g,b)` values, each optionally paired with a step:
   - **Pinned** — `#FAFAFA at 50`: that exact hex is locked to that step.
   - **Unpinned** — `#DDDDDD` only: the agent places it at the best-fit step by lightness.
3. **Steps to generate** — default: `50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950`. Step 50 is always lightest, step 950 always darkest.

**Early placement notice for unpinned values**

After the user provides color input, and before generating the scale, for every unpinned value:

1. Convert the hex to HSL.
2. Estimate the nearest step from lightness (reference curve: step 50 ≈ L 97%, step 500 ≈ L 50%, step 950 ≈ L 5%).
3. Tell the user:
   > "`{hex}` will be placed at approximately **step {N}** (L≈{L}%) by lightness fit. Pin it to a different step, or proceed with best-fit?"
4. Wait for confirmation. If the user pins it, treat it as a pinned anchor going forward.

---

## Step 2 — Figma destination (if not already provided)

If the Figma URL or collection name were not in the opening message, ask now — before generating the scale so any destination issues are caught early:

- **Figma file URL** — paste any `figma.com/design/...` or `figma.com/file/...` link. The `fileKey` is the segment after `/design/` or `/file/`.
- **Variable collection name** — the existing (or new) collection to write into, e.g. `Primitives`.
- **Group prefix (optional)** — a folder path prepended to each variable name. Example: `color` → variables are named `color/red/500`. Default: none → variables are named `red/500`.

---

## Step 3 — Generate scale

For each color family:

**3a — Build the initial scale using pinned anchors**

- Sort all pinned values by step (ascending).
- Interpolate HSL lightness between consecutive pinned steps so the curve passes exactly through both endpoints.
- Extrapolate lightness toward white (step 50 = lightest) below the lowest pinned step.
- Extrapolate lightness toward black (step 950 = darkest) above the highest pinned step.
- Preserve hue and saturation throughout; apply slight saturation correction at extremes for perceptual quality.

**3b — Place unpinned values**

For each unpinned value, find the step in the generated scale whose hex is closest by lightness proximity. Treat that step as a pinned anchor, regenerate the affected segment, and annotate it with `← best fit` in the preview.

If two unpinned values resolve to the same step, flag the conflict and ask the user which one to keep (or whether to shift one to an adjacent step).

---

## Step 4 — Preview and confirm

Output the full scale as a hex table before writing anything:

```
red-50:  #fafafa  ← pinned
red-100: #f0f0f0
...
red-400: #dddddd  ← best fit
...
red-700: #777777  ← best fit
...
red-950: #111111  ← pinned
```

Ask: "Does this scale look right? Proceed to push to Figma?"

Do not continue until the user confirms.

---

## Step 5 — Push to Figma

1. Load the `figma-use` skill at `/Users/jiawei.wu/.codex/skills/figma-use/SKILL.md`.
2. Parse the `fileKey` from the provided Figma URL.
3. For each color step, create or update a `COLOR` variable in the named collection:
   - **Variable name:** `{group}/{family}/{step}` if a group prefix was given, else `{family}/{step}` (e.g. `red/500` or `color/red/500`).
   - **Type:** `COLOR`
   - **Value:** the hex converted to RGBA (r, g, b values in 0–1 range, alpha = 1).
4. Before executing any writes: list all variables that already exist with a different value and ask the user to confirm overwriting.
5. Execute the batch write via `use_figma`.
6. Confirm to the user which variables were created and which were updated.

---

## Verification checklist

Before finishing, confirm:

- [ ] All requested steps have a hex value in the confirmed scale.
- [ ] Every pinned value matches the input hex exactly at its declared step.
- [ ] Every unpinned value is placed at the step confirmed in Step 1.
- [ ] No two adjacent steps share the same hex value (plateau = algorithm failure — regenerate).
- [ ] No step is pure white (`#ffffff`) or pure black (`#000000`) unless explicitly approved.
- [ ] All variables exist in Figma in the correct collection under the correct name pattern.

If any item fails, fix it before finishing. If a fix is not possible (e.g. Figma MCP unavailable), report it clearly to the user.
