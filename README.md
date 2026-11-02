# scout

**fork → work → distill → return** — a prompt-level protocol that keeps raw
material out of an agent's main context. Heavy work is delegated to a *scout*
subagent that inherits a small **distilled context log**, works in isolation,
and returns only a structured report (outcome / evidence / pitfalls /
rehydrate pointer).

Like a field scout: it goes out, surveys the terrain, and brings back a map —
not the mountain.

## Why

Context windows fill up because the main agent's context is a verbatim
transcript — every tool output stays. Scout moves the transcription into
isolated subagent contexts and lets only distilled docs cross the boundary.

- **Lossless dispatch**: the parent injects its whole distilled log into the
  scout prompt (cheap — the log is small by construction). No lossy briefs.
- **Lossy return**: the scout runs a mandatory distillation pass and returns
  only a schema'd doc. Compression happens on the information-complete side.
- **Pitfall transfer**: every report carries a `pitfall` field so dead ends
  and traps propagate to later subtasks instead of being re-hit.
- **Rehydrate**: docs carry pointers back to the scout's on-disk evidence —
  just-in-time detail, not preemptive bulk.

Contrast with standard subagent practice (brief out, free-text back) and with
learned context-folding (Context-Folding, AgentFold): scout needs **no
training** — it is a protocol enforced by a skill + a subagent profile.

## Layout

```
skills/scout/SKILL.md   # parent-side protocol skill (Devin / Claude Code compatible)
agents/scout.md         # worker profile: model pin + built-in report schema
SPEC.md                 # the protocol specification
.claude-plugin/         # Claude Code plugin manifest
```

## Install

**Devin CLI**

```bash
cp -r skills/scout ~/.config/devin/skills/
cp agents/scout.md ~/.config/devin/agents/
```

**Claude Code** — as a plugin (add this repo as a marketplace), or manually:

```bash
cp -r skills/scout ~/.claude/skills/
cp agents/scout.md ~/.claude/agents/
```

## Usage

Long-horizon task → the skill fires, or say "use a scout for X". The parent
composes: `[distilled log snapshot] + [subtask brief] + [persistence
requirement]` and spawns the `scout` profile. The scout replies with the doc
schema; only that doc enters the parent context.

See `SPEC.md` for the full contract and the "when not to use it" clause.

## Status

v0.1 — dogfooding. Eval harness (dispatch-mode ablations, pitfall-transfer
metric, dual token accounting) planned under `harness/`.
