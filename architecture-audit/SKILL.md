---
name: architecture-audit
description: >
  Deep multi-lens self-audit / architecture BLUEPRINT of a large, incrementally-grown source file (or
  subsystem). Fans out independent expert lenses that read the SAME code from DIFFERENT angles in
  parallel, each grounding every claim in file:line evidence; one judge merges + re-verifies into a single
  structural MAP + a risk-rated, tiered refactor/consolidation PLAN. USE BEFORE a big port / rewrite /
  refactor of an accreted file, or when the owner suspects code-block-ordering / feature-partitioning /
  logical-coherence debt from phase-by-phase growth — so the next move is made PURPOSEFULLY in one coherent
  pass instead of piecemeal. Multi-agent, schema'd, ground-truth-anchored. Pairs with rigorous-batch (which
  EXECUTES the resulting plan under a real compiler/test gate). Validated on a large accreted production
  source file (refactor audit → tiered plan → a large multi-hunk refactor under a per-commit compiler gate, 0 blockers).
---

# Architecture Audit / Blueprint

A reusable workflow PATTERN, not a tutorial. It turns a large, incrementally-grown source file into two
artifacts the owner signs off BEFORE any code moves:
1. **The MAP / blueprint** — every subsystem, its inputs/outputs, its state (and lifecycle), its draw/IO
   surface, where it physically lives in the file, and how the subsystems couple. The "design diagram".
2. **The PLAN** — a tiered, risk-rated list of consolidation/refactor/port moves, each with file:line
   evidence, a line-delta estimate, a RISK + the specific hazard, and a behavior-preserving-vs-semantic tag.

**The one idea:** many independent expert lenses read the same code from different angles in parallel, each
grounding every claim in `file:line`; one judge merges and **re-verifies every medium/high-impact claim
against the source** before accepting. A unanimous panel can still be wrong — so the judge reads the cited
lines, and the owner signs off the map before a single line is touched.

---

## 0. When to use / not
USE: before a big PORT / rewrite / refactor of a file that grew phase-by-phase; to blueprint an accreted or
unfamiliar subsystem; to hunt incremental-accretion debt (duplication / dead code / surface-consolidation /
altitude) → a tiered plan the owner can execute over time.
SKIP: small or young files; when you need a single fact (search directly); active hot-feature work where a
reorg would collide (note the debt, defer the churn).

This skill PRODUCES the plan; **`rigorous-batch` EXECUTES it** (verbatim-anchored hunks → ground-truth gate
→ final-review). Keep them separate: map/plan first, sign-off, then execute.

---

## 1. Pipeline
```
Investigate (parallel lenses, file:line-anchored)  →  Judge (merge + RE-VERIFY against source)  →  owner sign-off  →  hand to rigorous-batch
```
Read every lens result before judging. Persist the blueprint + plan to disk (a real .md doc) so the port/
refactor and any rerun are cheap. For a HUGE file, **chunk by subsystem** so each lens reads a TARGETED
slice (offset/limit), never the whole file — context pressure is the enemy of accuracy.

---

## 2. The lenses (pick the set the job needs; run in parallel)

**AUDIT mode — "what's the debt?"** (a solid default audit set):
- **Surface / UX** — overlapping/contradictory options, toggles that gate display AND data, naming drift
  (Show*/Enable*/bare), group/order oddities, properties whose description no longer matches behavior, the
  deprecate-without-breaking-templates shims.
- **Duplication** — "one concept implemented N times": mirrored hi/lo sides, reset twins, the add-a-field-
  touch-N-places tax, positional arg lists. For each: is extraction SAFE (behavior-preserving, no hot-path
  alloc) and worth it, or is explicit better for greppability? Judge honestly.
- **Dead / vestigial** — fields written but never read in a decision; properties that gate nothing; enum
  members never branched on; comment archaeology (changelog narration git already owns) vs load-bearing
  invariants; unused imports; stale counts/claims a later phase obsoleted.
- **Altitude** — the macro shape: is the hot-path body better as named phase-methods or is the linear body
  the RIGHT altitude (decomposition for its own sake is the failure mode)? History-named regions → function-
  named. Class-extraction candidates (does it PAY — e.g. enabled differential testing — or just add
  indirection?). File-split feasibility.

**BLUEPRINT mode — "what IS the system?"** (add these when mapping for a port/rewrite):
- **Subsystem map** — enumerate every subsystem; for each: responsibility, inputs/outputs, the
  function(s)+line-ranges it lives in (flag when ONE feature is scattered across many regions = the
  accretion smell), and its public surface.
- **Data-flow / state-lifecycle** — trace the per-cycle pipeline; classify every piece of state by its
  lifecycle (snapshotted/rolled-back vs rollback-immune vs once-per-load latch); the commit/restore seams.
- **Draw / IO / side-effect surface** — every render path, file/network IO, alert, external call, and what
  gates it; the tag/lifecycle of each draw object (ghosts, cleanup symmetry).
- **Port-relevance** (when blueprinting for a port) — tag each subsystem: must-port-behavior / platform-
  only-telemetry / display / research-target / already-ported-and-current-vs-stale. This is what makes the
  port PURPOSEFUL and one-pass.

Scale to stakes: a quick audit = 4 lenses + 1 judge; "blueprint a 7k-line accreted file for a full port /
be exhaustive" = chunk by subsystem (8-15 region-owners) + the cross-cutting lenses + dual judges that
reconcile. With "unlimited agents", prefer the larger fleet and a second independent judge.

---

## 3. The COMMON preamble (every lens gets it — the accuracy lever)
Always hand each lens: the file path + how to read it (offset/limit; the targeted slice for chunked runs);
the **inviolable-constraints doc** (the WATCHPOINTS / frozen-internals record) to read BEFORE judging; and a
hard ABSOLUTE-CONSTRAINTS list (frozen clean-room internals, rollback rules, template-breaking renames,
output-identity, etc.) where **violating any one disqualifies a proposal**. Require output = concrete
findings with **file:line evidence**, the consolidated design (real code sketch where it matters), a
line-delta estimate, RISK (low/med/high) + the specific hazard, the template/migration need, and a
**mechanical-behavior-preserving vs UX/semantic** tag. No preamble in the final message — just the report.

---

## 4. Schemas (force structured output; validate + retry at the tool layer)
- **Finding**: `{title, kind(consolidation|duplication|dead-code|altitude|subsystem|dataflow|port-reln|naming), evidence(file:line), proposal, lineDelta, risk(low|med|high), hazard, behaviorPreserving(bool), migrationNeeds}`.
- **Blueprint node** (mapping mode): `{subsystem, responsibility, livesAt(line-ranges), inputs, outputs, state(var/immune/latch), drawIO, couplesTo[], portRelevance(must-port|telemetry|display|research|ported-current|ported-stale), notes}`.
- **Plan**: `{tiers:[{tier, items:[{title, action, lineDelta, risk, verification}]}], doNotTouch[], rejected[], headlineDesigns, totals}`.

---

## 5. Harness skeleton

Pseudocode in a generic multi-agent harness: `agent()`, `parallel()`, `phase()`, and `schema:` are fan-out
primitives to map onto your own orchestrator, not a specific product.
```js
export const meta = { name: 'arch-audit', description: '...', phases:[{title:'Lenses'},{title:'Judge'}] }
const SRC = '<file>', WP = '<constraints-doc>'
const COMMON = `Audit ${SRC}. Read it (offset/limit) + ${WP} before judging. ABSOLUTE CONSTRAINTS (violating any disqualifies): ...
Output: file:line evidence, consolidated design, lineDelta, RISK+hazard, behaviorPreserving, migrationNeeds. Final message = report only.`
phase('Lenses')
const lenses = await parallel(LENSES.map(L => () => agent(COMMON + L.task, {label:L.label, phase:'Lenses', schema:FINDING_OR_NODE})))
phase('Judge')                                   // re-verify every med/high claim against the source before accepting
const plan = await agent(COMMON + 'ROLE: JUDGE. Merge + RE-READ the cited lines. Tier1 mechanical / Tier2 dedicated-session / Tier3 rejected-with-reason. Emit the map + the plan.\n' + JSON.stringify(lenses), {schema: PLAN})
return { plan, lenses }
```
For a huge file: make `LENSES` = the per-subsystem region-owners (each with its `offset/limit` slice) PLUS
the cross-cutting lenses; run dual judges and reconcile when accuracy matters.

---

## 6. Principles (carry these even without the full fleet)
- **Ground truth > panel consensus.** The judge RE-READS every cited line for med/high claims. file:line or
  it didn't happen.
- **Distinguish mechanical from semantic.** Behavior-preserving reorg ≠ a UX/behavior change — never blur
  them; the owner approves semantics, not you.
- **Minimal, evidenced moves.** Decomposition/extraction for its own sake is the failure mode. Three similar
  explicit blocks can beat one premature abstraction — say so when true (the Tier-3 "rejected, here's why"
  list stops the owner re-flagging load-bearing code).
- **Map before move.** The blueprint is signed off BEFORE rigorous-batch touches a line.
- **Don't churn during hot feature work.** Note the debt, schedule the reorg with the next real batch.

---

## 7. Provenance
Extracted from a real large-file refactor-audit workflow (4 lenses: surface / duplication / dead-code /
altitude → judge → tiered risk-rated plan; the COMMON preamble pinned the project's frozen clean-room
internals + an architecture-snapshot + template-shim constraints). That plan drove a large multi-hunk refactor under a per-commit compiler gate and a
multi-reviewer final pass, with no blocking findings. Generalized here with the BLUEPRINT
lenses for the case where the owner is mapping an accreted file to PORT/rewrite it purposefully in one pass.
Pairs with `rigorous-batch` (execution); when you have them, also pair with your own domain-map and
platform-rules skills.
