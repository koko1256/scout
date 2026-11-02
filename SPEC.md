# Scout Protocol — Specification v0.1

**fork → work → distill → return**

Scout is a context-management protocol for LLM agents. It treats the parent
agent's context as a *distilled log*: bounded, append-only, and containing only
decisions and structured reports. All raw work happens inside delegated
subagent contexts ("scouts") whose transcripts never enter the parent context.

## 1. Roles

- **Parent (orchestrator)** — owns the user-facing task and the distilled log.
- **Scout (worker subagent)** — executes one bounded subtask in isolation.

## 2. Parent context invariant

The parent context MUST contain only:

1. the task goal and constraints,
2. decisions already made,
3. returned docs from scouts.

Raw tool output, file contents, and bulk observations MUST NOT accumulate in
the parent context — that work is always delegated. Because the log is small
by construction, injecting it wholesale into a scout prompt is cheap. This
invariant is what makes lossless dispatch safe (cf. "clean-slate delegation"
advice, which assumes the parent transcript is polluted; a distilled log is
not).

## 3. Dispatch (fork)

For each subtask the parent spawns a scout whose prompt contains:

1. **Distilled log snapshot** — the parent context verbatim (lossless outbound).
2. **Brief** — what to do, what to solve, constraints, done-when; SHOULD cite
   relevant `pitfall` entries from prior docs (pitfall transfer).
3. **Persistence requirement** — where to write working notes/evidence so the
   return doc can carry rehydrate pointers instead of raw content.

## 4. Work

The scout works with its own full context budget and tools. Its raw tool
output stays in its own context. It SHOULD persist intermediate artifacts and
evidence to files and cite paths.

## 5. Distill (mandatory final step)

Before finishing, the scout performs a dedicated distillation pass over its
full working state and returns exactly this schema (not its last tool output):

```text
## 结果 / Result
- outcome:: done | partial | failed   # one line
- answer::  <actual deliverable/answer>

## 证据 / Evidence
- evidence:: <file paths, line numbers, transcript locations>

## 坑 / Pitfalls   (mandatory; write "无"/"none" if empty)
- pitfall:: <dead ends, misleading sources, env quirks; one per line>

## 复水入口 / Rehydrate
- rehydrate:: <where the parent can get more detail if needed>
```

## 6. Return

- The parent appends ONLY the doc to its distilled log.
- For more detail the parent follows `rehydrate` pointers (file reads) or
  resumes/queries the same scout — it does not redo the work in-context.

## 7. When NOT to use

Do not fragment context when subtasks require continuous shared state with the
main thread — a single context with compaction is better (cf. Cognition,
"Don't Build Multi-Agents", 2025). Delegation pays when subtasks are
boundable and raw material is heavy.

## 8. Asymmetry thesis

Standard subagent dispatch compresses on the outbound leg (a lossy brief the
parent writes before knowing what the child will need). Scout compresses only
on the return leg, where the scout — holding complete information — decides
what matters. Lossless dispatch is viable because the injected payload is a
curated log, not a raw transcript.

## 9. Accounting (eval targets)

Two ledgers are tracked separately:

- **parent-context peak tokens** — the "context saturation" metric;
- **total tokens spent** — the cost metric (scout re-reads the injected log;
  prefix caching may offset this).

Quality metrics: task accuracy; **pitfall-transfer rate** (whether later
subtasks avoid recorded traps); schema-compliance rate.

## 10. Related work

- Context-Folding (arXiv:2510.11967), AgentFold (arXiv:2510.24699) — folding
  learned via RL/SFT inside one model. Scout is the same shape as an
  untrained protocol: model-agnostic, framework-level.
- Anthropic, "Effective context engineering" (2025); Manus context-engineering
  notes; Drew Breunig's context failure taxonomy.
- LangChain Deep Agents `mode:"fork"` — context injection exists as a feature;
  scout adds the distilled-log invariant + schema'd return + eval targets.
