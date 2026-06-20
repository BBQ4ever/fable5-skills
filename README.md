# Agentic Workflow Skills

Five cross-model, cross-project **methodology skills** for high-accuracy AI-agent work on critical code and
analysis. Each is a reusable workflow PATTERN — multi-agent, schema'd, and anchored to a **model-independent
ground truth** (a compiler, a diff, the data) — because a unanimous panel of agents can still be unanimously
wrong.

## Origin

These patterns were extracted from the workflows our own project actually ran during the roughly **72-hour
window the `fable 5` model was available**. In that window we executed many multi-agent batches against a large
production codebase, and these five are the patterns that earned their keep: each was used in anger, gated
against a real compiler / diff / dataset, and **verified to be highly efficient and high-accuracy in practice**
— not theory. They were then generalized (all project-, domain-, and platform-specific fingerprints removed) so
they carry over to any project and any model. That portability is the whole point: models get swapped out, the
ground-truth gate does not.

## The skills

| Skill | What it does | Reach for it when |
|---|---|---|
| **rigorous-batch** | High-accuracy multi-agent pipeline for changing critical code (or producing high-stakes specs): verbatim-anchored minimal diffs → adversarial multi-lens verification → a REAL compiler/test gate → final-review fleet → atomic commit. | A change is large, load-bearing, or hard to reverse — especially when the running model is not the strongest. |
| **architecture-audit** | Deep multi-lens self-audit / architecture blueprint of a large, accreted source file → a structural MAP + a risk-rated, tiered refactor/port PLAN, signed off before any code moves. | Before a big port / rewrite / refactor; to map an unfamiliar subsystem; to hunt incremental-accretion debt. |
| **clean-room-rewrite** | Reimplement or replace a component and PROVE behavior-equivalence via an executable differential-equivalence harness; optional legal clean-room isolation + an expression-similarity audit. | Rewriting a risky/legacy/derived component with zero regression, or producing a provably-independent implementation of a public concept. |
| **reality-check** | Audit accumulated claims / a shipped feature's described behavior against GROUND TRUTH = the code + the data, not the narrative; re-derive each claim and classify every gap by severity. | A long session piled up assertions you are about to act on; a conclusion "feels" true but was never recomputed. |
| **reproducible-analysis** | Turn raw data exports into a standardized, reproducible offline report (same calculations every run) + the interpretation discipline that stops the common analysis errors. | You repeatedly analyze refreshed exports and need every report apples-to-apples and honestly caveated. |

They compose: **reality-check** verifies a premise → **architecture-audit** maps before you move →
**rigorous-batch** executes under a ground-truth gate → **clean-room-rewrite** proves an equivalent
replacement → **reproducible-analysis** reads the resulting data honestly.

## Operational notes — these are heavyweight by design

They trade tokens for accuracy on purpose; the redundancy and the ground-truth gate are what let a weaker model
land critical changes correctly.

- **`architecture-audit` is the most expensive.** An exhaustive run fans out **dozens of agents concurrently in
  the background** (one per subsystem / lens, plus one or more judges) and typically consumes **2,000,000+
  tokens in a single run**. Reserve it for genuinely large or high-stakes files, and scale the lens count to
  the stakes.
- Every skill documents a "scale to the stakes" knob: a quick pass is cheap (a few agents + a single verify);
  "be exhaustive / max rigor" spins up the full fleet (dual critics, large final-review fleet, mandatory
  ground-truth gate). Pick the tier deliberately — the full pipeline is overkill for trivial edits.
- **Harness-agnostic.** The code skeletons inside the skills use generic multi-agent primitives (`agent()`,
  `pipeline()`, `parallel()`, `schema:`) purely as illustration — map them onto whatever orchestrator you run
  agents with. The patterns themselves don't depend on any specific tool.

## Install

Each skill is a single self-contained `SKILL.md` — no scripts, no dependencies. Copy the skill folders into
your agent's skills directory. On Windows that is typically:

```
%USERPROFILE%\.claude\skills\
```

If you also run agents under WSL/Linux, copy them into that environment's skills directory as well
(e.g. `~/.claude/skills/`) — the two environments do not share a path.

## Disclaimer

This is a personal, unofficial collection of workflow patterns. It is **not affiliated with, endorsed by, or
supported by Anthropic**. "Claude", "Fable", and "Mythos" are trademarks of Anthropic, referenced here only to
describe the context in which these patterns were developed. The skills are methodology documents authored by
the repository owner from their own authorized use of the model; they contain no Anthropic proprietary
materials and are provided as-is, without warranty.
