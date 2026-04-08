# fight-me — Claude Slash Command

Last updated: 2026-04-08

Critically audits your instruction before any work begins. Identifies flaws, ambiguities, risks, and trade-offs, then holds until you acknowledge the critique.

## Usage

```
/fight-me <your plan or instruction>
```

Add `--bdsm` for a harsher, unfiltered critique:

```
/fight-me --bdsm <your plan or instruction>
```

---

## Installation

Claude slash commands are stored as `.md` files and discovered automatically from two locations.

### Option A — Global install (available in every Claude project)

```bash
mkdir -p ~/.claude/commands
cp ./fight-me.md ~/.claude/commands/fight-me.md
```

### Option B — Project-local install (available only in the current project)

```bash
mkdir -p .claude/commands
cp ./fight-me.md .claude/commands/fight-me.md
```

### Verify

Open a new Claude conversation in your project and type `/fight-me` — Claude should autocomplete the command and audit your next instruction before doing any work.

---

## What the critique report looks like

```
## fight-me critique

### Verdict
MODERATE CONCERNS

### Flaws found
1. **Ambiguous scope** — The request doesn't define success criteria, making it impossible to know when the task is done.

### Constructive suggestions
1. **Ambiguous scope** — Add a one-sentence definition of done before starting.

### What is solid
- The general direction is sound and the technology choice is appropriate.

### Recommended next step
Clarify scope before proceeding.
```
