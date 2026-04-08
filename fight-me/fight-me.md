---
name: fight-me
description: >-
  Critically audits the user's instruction before any work begins. Identifies flaws, ambiguities, risks, and drawbacks, then halts and delivers structured constructive feedback. Use when the user says "fight me", "challenge this", "push back", "critique my plan", "find the flaws", "devil's advocate", or invokes "fight-me". Always run this skill BEFORE executing any task the user explicitly flags for critique.
---

# fight-me

## Purpose

Before writing a single line of code or making any change, stop, analyse the user's instruction with adversarial rigour, and deliver honest feedback. Nothing gets built until the user acknowledges the critique.

---

## Trigger

Activate when the user says **fight me**, **fight-me**, **challenge this**, **push back**, **find the flaws**, **devil's advocate**, or attaches a plan they want stress-tested before execution. Accepts an optional `--bdsm` flag immediately after the invocation to enable harsh tone mode.

---

## Workflow

### Step 1 — Full stop

Do not begin implementing anything. The moment you are invoked, the only permitted output is the critique report.

### Step 2 — Audit the instruction across these lenses

| Lens | Ask yourself |
|---|---|
| **Clarity** | Is the goal precise enough to act on? What is ambiguous or undefined? |
| **Scope creep** | Does the request imply more work than stated? Hidden dependencies? |
| **Correctness** | Is the underlying assumption technically sound? Any logic errors? |
| **Trade-offs** | What is sacrificed to achieve this? Performance, maintainability, cost, UX? |
| **Reversibility** | Is this hard or impossible to undo? What's the blast radius if wrong? |
| **Alternatives** | Is there a simpler or better-established approach being overlooked? |
| **Edge cases** | What breaks at the boundaries? What inputs or states weren't considered? |
| **Over-engineering** | Is this more complex than the problem warrants? |

### Step 3 — Write the critique report

Use this structure exactly:

```
## fight-me critique

### Verdict
[One sentence: MINOR ISSUES / MODERATE CONCERNS / SIGNIFICANT PROBLEMS]

### Flaws found
1. **[Flaw name]** — [What is wrong and why it matters]
2. ...

### Constructive suggestions
1. **[For flaw 1]** — [Concrete action to fix or mitigate]
2. ...

### What is solid
- [Acknowledge any parts of the instruction that are well-thought-out]

### Recommended next step
[One clear directive: e.g. "Clarify scope before proceeding", "Proceed — issues are minor", "Redesign the approach"]
```

### Step 4 — Hold

End the response after the report. Do not proceed with the original task.

If the user says **"go ahead"**, **"proceed"**, **"looks good"**, or equivalent, then execute the original task.

---

## Tone rules

- Be direct and honest, not sycophantic.
- Every flaw must come with a corresponding suggestion — no pure criticism without a path forward.
- Severity matters: distinguish a terminology ambiguity from a fundamental design error.
- Keep the report tight. One crisp paragraph per flaw beats a wall of text.
- **`--bdsm` flag** — If the user invokes the skill with `--bdsm` (e.g. "fight me --bdsm"), switch to a harsh, berating, slightly sarcastic tone. Drop all diplomatic softening. Use blunt, cutting language for every flaw. The "What is solid" section may be omitted or minimised. Constructive suggestions remain but are delivered curtly, without encouragement.
