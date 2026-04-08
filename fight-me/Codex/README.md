# fight-me — Codex Skill

Last updated: 2026-04-08

Critically audits your instruction before any work begins. Identifies flaws, ambiguities, risks, and trade-offs, then holds until you acknowledge the critique.

## Usage

Invoke by saying **fight me**, **fight-me**, **challenge this**, **push back**, **find the flaws**, or **devil's advocate** before or alongside any plan you want stress-tested.

Add the `--bdsm` flag for a harsher, unfiltered critique.

---

## Installation

### Option A — Global install (recommended)

Copy the `fight-me` folder into your Codex skills directory so it is auto-discovered in every session:

```bash
cp -r ./fight-me "$HOME/.codex/skills/fight-me"
```

If you have a custom `CODEX_HOME` set:

```bash
cp -r ./fight-me "$CODEX_HOME/skills/fight-me"
```

### Option B — Project-local install

Copy the folder into your repo's local skills directory so it is available only in that project:

```bash
cp -r ./fight-me ./.codex/skills/fight-me
```

### Verify

After copying, confirm the layout looks like this:

```
~/.codex/skills/fight-me/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── README.md
```

Restart Codex (or open a new session) — the skill will be discovered automatically. You can confirm it is active by typing `fight me` in a new conversation; Codex should load the skill and audit your next instruction.

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
