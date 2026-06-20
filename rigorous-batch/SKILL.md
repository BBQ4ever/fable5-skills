---
name: rigorous-batch
description: >
  High-accuracy multi-agent pipeline for changing CRITICAL code (or producing high-stakes specs/analyses)
  where a mistake is expensive and the change is large or load-bearing. Activate when the user asks to
  design+implement a non-trivial change to important code, run a multi-round spec→implementation batch,
  audit/investigate a subtle defect, or explicitly wants "more rigor / more agents / max ultracode / a
  stronger team", AND especially when the working model is NOT the top-tier one (compensate for lower
  accuracy with redundancy + a model-independent ground-truth gate). Validated on large, load-bearing
  behavior changes to a production source file (clean first compiles against a locked baseline fingerprint).
  Encodes: verbatim-anchored minimal diffs, adversarial multi-lens verification, orchestrator rulings, and
  the keystone — a REAL compiler/test gate that no model opinion can override.
---

# Rigorous Batch — validated high-accuracy change pipeline

A working method, not a tutorial. It exists to make large changes to **critical code** land correctly on the
first compile, by replacing "the model is confident" with **independent adversarial verification + a
model-independent ground-truth gate**. Abstracted from a real project's high-stakes batches, which landed
large, load-bearing behavior changes on a production file with clean first compiles against a locked baseline
fingerprint. Use it whenever a mistake is expensive.

**The one idea**: model agents *propose and cross-examine*; the **compiler/test suite decides**. Every stage
below feeds a final gate that does not depend on any model being right.

---

## 0. When to use / when not

USE when: the change is load-bearing, large, or hard to reverse; the file is long and edit-anchoring is risky;
the output is a spec/contract others build against; you are auditing a subtle "is this a bug or by-design?"
question; or the user asked for more rigor / more agents. **Escalate rigor when the running model is not the
strongest available** — add redundancy (dual independent critics) and lean harder on the ground-truth gate.

SKIP for trivial/mechanical edits, conversational turns, or throwaway code — the overhead isn't worth it.

Scale to the ask: "find any bug" → a few finders + single-vote verify. "Be exhaustive / critical code / max
ultracode" → full pipeline, dual critics, large final-review fleet, mandatory ground-truth gate.

---

## 1. The pipeline (stages — run only the ones the task needs)

```
Investigate → Design → Spec → Implement-draft → Reconcile → GROUND-TRUTH GATE → Final-review → Commit
   (audit)    (panel)  (rounds)  (hunks)         (rulings)    (REAL compiler)    (on real diff)
```

Each stage is one or more harness runs (a fan-out, then a gather). **Read every workflow result before deciding the next stage** —
you stay in the loop; each run is one well-scoped fan-out. Persist artifacts to disk (proposal docs, hunk
manifests) so later stages and reruns are cheap.

### 1a. Investigate (only if the problem isn't yet understood)
Fan out independent auditors on different angles → **adversarially verify each finding** (skeptics prompted to
REFUTE; default to "by-design / not-real" unless proven). Classify findings: `by-design | latent-defect |
active-defect | uncertain`. Output a written conclusion the user signs off before any code is designed.
Example: 4 auditors (merge logic / state machine / dedup / cross-platform parity) → per-finding verifier.

### 1b. Design (only for open design/architecture questions)
N **independent perspectives** (e.g. domain-governance / practitioner / systems-engineer-who-owns-this-code) →
synthesize. Diversity of lens beats redundancy. Surface anti-patterns ("what an expert would NOT do") and how
to calibrate any thresholds from data, not intuition.

### 1c. Spec (for non-trivial implementations — produce a signed spec BEFORE code)
- **Slice owners** draft contracts + schemas + verbatim hook anchors for their region.
- **Multi-lens adversarial verify** each slice: rollback/state-discipline · minimal-machinery/house-pattern ·
  spec-conformance. For data-facing output add a **cold-read red-team** (can a fresh reader answer every
  required question from the artifact alone? find where the schema *claims* coverage but the derivation
  breaks). For completeness add a **critic** (what does NO slice cover? cross-slice identifier conflicts?).
- **Orchestrator rulings** (you) resolve every conflict with a recorded reason, in a numbered series
  (R1/R2…, V1/V2…, D1/D2…). You MAY override your own procedural defaults when engineering substance demands
  it — record why.
- **Redraft** until conflict-free. When two deliverables must agree (e.g. a spec and its schema doc), end with
  a **byte-level drift check** that machine-compares the shared strings (headers, signatures), not eyeballs.
- Output: a signed proposal doc on disk + open decisions the user must make.

### 1d. Implement-draft (turn the signed spec into hunks)
Region owners produce **verbatim-anchored minimal-diff hunks** (see §2). Verify each with 3 lenses:
**anchor-guard** (oldCode occurs exactly once, byte-faithful, no overlap) · **correctness** (compiles +
behaves; flag identifiers no hunk defines) · **spec-conformance** (every hunk traces to a spec ruling; nothing
dropped; defaults unchanged). Then the **critic** for cross-hunk integration.

### 1e. Reconcile (resolve what the verifiers found)
Orchestrator rulings again. Merge or order hunks that share an anchor (§2). **If the model is lower-accuracy,
run TWO independent critics and reconcile them** — disagreement between independent critics is itself signal.
Produce a final apply manifest: every hunk, in a sound apply order, with supersessions noted.

### 1f. GROUND-TRUTH GATE — the keystone (never skip for code)
Apply the manifest to the real file(s), then run the **real compiler / test suite / linter** — the thing that
cannot be wrong. Before applying, **lock a baseline fingerprint** (e.g. "0 errors, the exact baseline warning count, these exact
diagnostic codes, none in the target file") so post-change you can require a **byte-exact / count-exact match**. The
compiler's errors are precise — fix and recompile until green. This gate is what makes the method work with a
weaker model: it converts "I think it's right" into "it provably builds and matches the fingerprint."
For non-compiled artifacts the analogue is: run the parser/validator, execute the worked example, diff against
a golden output.

### 1g. Final-review fleet (review the REAL diff, post-compile)
The compiler proved it *builds*; this proves it's *correct*. Fan out reviewers across dimensions
(correctness · spec-conformance · rollback/state · gating-leakage · contract/byte-match · behavior-regression),
each adversarial, on the **applied diff** — higher value than reviewing proposals because it's real code. Fix
findings, recompile, re-review the delta.

### 1h. Commit
Atomic commit (the whole batch in one commit so partial application can't break the build), descriptive
message with provenance (pipeline + agent count + gate result), push only if asked. Then: update memory,
append a project WATCHPOINTS / human-acceptance checklist section listing exactly what the human must verify at runtime
(the things the model **cannot** observe — UI, live data, interaction), and record a **single-command
rollback** (`git checkout <hash>~1 -- <files>`).

---

## 2. Verbatim-anchored minimal-diff hunks (the edit discipline)

Each hunk = `{ oldCode, newCode }`:
- **oldCode** quoted VERBATIM from the current file (exact whitespace/tabs), with enough surrounding lines to
  be **unique in the whole file** — verify uniqueness yourself (search, confirm exactly one occurrence).
  Anchors are applied later with an exact-string Edit, so a wrong/ambiguous anchor is a failed hunk.
- **newCode** in the file's existing style and comment density. New code carries **load-bearing comments**:
  state the invariant and the WHY (cite the spec section/ruling), not what the next line does.
- **Minimal diff**: change only what the design requires. No drive-by refactors, no comment rewrites of
  untouched code. Three similar lines beat one premature abstraction.
- **Shared-anchor hazard (critical)**: two Edits CANNOT share the same oldCode — after the first rewrites it,
  the second fails to match. When two hunks target the same line: **merge them into one hunk**, or re-anchor
  one onto the other's newCode, or pin a strict apply order where the anchors stay disjoint. An integration
  simulator that computes the total apply order catches this before it bites.
- **Track hunks by (draft, id)** — independent drafters reuse ids (C1, C1…); the pair is the key, not the id.

---

## 2a. Worked micro-example (generic — illustration only)

A deliberately trivial, domain-neutral pass of ONE hunk through the loop. The real worked examples belong in
your own project's proposal docs (that is where domain detail lives); this just shows the SHAPE.

**Ruling** — `R1`: clamp negative quantities to 0 before summing (spec §2.1); behavior-preserving for valid input.

**Hunk** `C1` — `oldCode` quoted verbatim and unique in the file:
```
// oldCode
function cartTotal(items) {
  return items.reduce((s, i) => s + i.price * i.qty, 0)
}
// newCode
function cartTotal(items) {
  // R1: a negative qty must not subtract from the total (spec §2.1)
  return items.reduce((s, i) => s + i.price * Math.max(0, i.qty), 0)
}
```

**Ground-truth gate**: lock the baseline (`0 errors, N warnings`, exact diagnostic codes) → apply the hunk →
recompile → require an exact match. **Final-review** on the applied diff: confirm no caller relied on the old
negative-qty behavior. One ruling, one anchored hunk, one gate, one review — the whole loop in miniature.

---

## 3. Principles that make it accurate (carry these even without the full pipeline)

- **Adversarial by default.** Verifiers are prompted to find problems and prefer false-positives to misses.
  A verifier that "looks fine to me" did nothing.
- **Ground truth > model consensus.** A unanimous agent panel can be unanimously wrong. End on the compiler /
  tests / a real run. Schema-validate structured agent output so retries happen at the tool layer.
- **Orchestrator as single authority.** One place resolves conflicts, each ruling has a recorded reason and a
  number. Rulings can overturn earlier rulings or the model's own procedural defaults — record the why.
- **Independence over redundancy of the same lens.** Three identical reviewers ≈ one. Diverse lenses (or, for
  critics, genuinely independent agents told not to assume each other's conclusions) find more.
- **Measure before gate.** For any new control/threshold, ship it diagnostics-only first, calibrate from
  logged data, then turn on enforcement. Don't gate on day-one intuition.
- **Provenance + rollback always.** Every batch records its pipeline and a one-command undo. Behavior that
  deliberately diverges from a prior baseline gets a dated note saying so.
- **Name the human's blind-spot list.** The model can't see runtime UI, live data, or interaction. End every
  code batch with an explicit human-acceptance checklist for exactly those.

---

## 4. Compensating for a weaker model (the reason this skill exists)

When the running model is not the strongest available, do not lower ambition — raise rigor:
1. **Dual independent critics** at every critic stage; reconcile their disagreement explicitly.
2. **More finders/verifiers**, looped until two consecutive rounds surface nothing new (loop-until-dry).
3. **Always end on the ground-truth gate.** The compiler/test suite has the same accuracy regardless of which
   model drove the edits — it is the great equalizer. Make it mandatory and make its pass condition exact
   (fingerprint match, not "looks like it built").
4. **Prefer structural fixes over discipline.** E.g. merge two order-sensitive hunks into one rather than
   relying on remembering the apply order — eliminate the hazard class, don't manage it.

---

## 5. Encoding it in a multi-agent harness

The skeleton names used here — `agent()`, `pipeline()`, `parallel()`, `schema:`, `.filter(Boolean)` — are
generic fan-out primitives, not a specific product: a sequential pipeline, a parallel map over agents, a
per-agent output schema, and a way to discard failed agents. Map them onto whatever orchestration harness you
use to run agents.

- **`pipeline()` by default**; `parallel()` only when a stage genuinely needs ALL prior results (dedup, early-
  exit, cross-item comparison). Filter `.filter(Boolean)` — failed/cancelled agents return null.
- **`schema:`** on every agent that returns data — validation + retry at the tool layer, no parsing.
- **Persist** the proposal doc and hunk manifest to disk (`Write`); pass paths to later agents; this makes
  reruns and the apply stage cheap and lets you recover when an agent dies mid-run (re-extract from the task
  output JSON, relaunch only the missing agents).
- **Agents can die on transient errors.** Check which completed (null verdicts), and relaunch ONLY the gaps —
  don't redo the whole round.
- **Adversarial-verify stage**: `pipeline(findings, finder, review => parallel(review.map(skeptic)))`.
- Scale agent count to the stakes; with "max ultracode / unconditional agents" granted, prefer the larger
  fleet and the dual-critic / large-final-review variants.

---

## 6. Provenance

Validated on a large production source file across multiple high-stakes batches. The pipeline ran end to end:
investigate (multi-agent audit) → design panel (independent perspectives) → spec (multiple rounds, a numbered
ruling series, a byte-level drift check between a spec and its schema doc) → implement-draft (verbatim-anchored
hunks) → reconcile (dual independent critics + an integration simulator, held together even across a mid-run
model swap) → a full-tree compiler gate (lock a baseline fingerprint, require an exact match) → final-review
fleet on the applied diff → atomic commit + a human-acceptance checklist + a one-command rollback. First
compiles landed clean against the locked baseline. Keep each batch's proposal docs (spec / numbered rulings /
acceptance-checklist) as worked examples for reruns.
