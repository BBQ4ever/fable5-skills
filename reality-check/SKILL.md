---
name: reality-check
description: >
  Audit accumulated CLAIMS / IMPRESSIONS / a-shipped-feature's-described-behavior against GROUND TRUTH = the
  code + the real data, NOT the prior narrative. Independent lenses RE-DERIVE each claim from the source from
  scratch, cite file:line or data for every gap, and classify it (wrong / overstated / unverified /
  subtle-risk / nit); a synthesizer separates genuine errors from overstatements from confirmed-sound and
  gives a calibrated verdict. USE when a long session has piled up assertions you're about to act on, when a
  conclusion "feels" true but was never recomputed, when auditing whether a just-shipped feature actually does
  what you said it does, or whenever the owner asks "is this real or did we drift?". The model's confidence is
  not evidence. Multi-agent, schema'd, skeptic-by-default. Validated on a real project where the model's
  mental model diverged from reality twice (a statistic that read as a constant only because of a tie-handling
  bug in the metric; a "the parameter locks at event start" belief that was false — it actually changed mid-process).
---

# Reality Check — impression vs. ground truth

A working method, not a tutorial. It finds the places where what you (or the model) BELIEVE has quietly
drifted from what the code and data ACTUALLY do — before that belief drives a decision.

**The one idea:** **ground truth = the code + the real data, never the running narrative.** Every load-bearing
claim gets RE-DERIVED from the source independently (not "does this sound right?" but "show me the line / the
number"). Impressions accumulate silently across a long session; a measurement bug or a plausible-but-wrong
mental model survives precisely because nobody recomputed it. This audit recomputes it.

---

## 0. When to use / not
USE: before acting on a belief that was asserted but never re-verified; after a long analysis session that
accreted claims; to check a just-shipped feature's faithfulness to its spec/description; when a result "feels"
settled but rests on one earlier computation; when the owner says "double-check this / is it real?". SKIP:
a single fresh fact you can verify in one read (just verify it); brand-new code with no accumulated narrative.

This audits CLAIMS. To change code use `rigorous-batch`; to map structure use `architecture-audit`; to prove
two implementations equal use `clean-room-rewrite`. Reality-check often PRECEDES those — it tells you whether
the premise is even true.

---

## 1. Pipeline
```
Lenses (parallel: each RE-DERIVES a cluster of claims from code+data, file:line-cited)  →  Synthesize (honest gap report + calibrated verdict)
```
Group the claims into independent clusters and give each lens one cluster + the GROUND-TRUTH preamble. Then a
synthesizer merges, separating genuine errors from overstatements from sound — and re-reads the cited
evidence for anything it upgrades to "must fix".

---

## 2. The GROUND-TRUTH preamble (hand it to every lens)
State plainly: **ground truth = `<the code paths>` + `<the real data>`, NOT the orchestrator's prior claims.**
Name the specific drift episodes that motivated the audit (it calibrates the skeptics). List what is ALREADY
empirically established (with the exact constants / line refs / data splits) so lenses re-verify rather than
re-discover. Require: be a ruthless skeptic; cite **file:line or the data row/number for EVERY gap**; find any
mechanism the narrative described that the code does NOT do, and any path the narrative MISSED. Return strict
schema JSON.

---

## 3. Schemas
- **Finding** (per lens): `{lens, confirmations[] (claims re-derived and CONFIRMED), gaps:[{severity(wrong|overstated|unverified|subtle-risk|nit), claim, reality (what the code/data ACTUALLY does), evidence (file:line or data), impact, fix}], summary}`.
- **Synthesis**: `{verdict(MODEL-SOUND|MINOR-GAPS|REAL-ERRORS), realErrors[] (genuine wrong claims / bugs to fix), overstatements[] (stated more certainly than warranted — recalibrate the framing), confirmedSound[], <featureVerdict> (is the shipped thing faithful?), fullAnswer}`.

The **severity ladder is the point**: separate `wrong` (a real error / bug) from `overstated` (true-ish but
oversold — fix the FRAMING, not the code) from `unverified` (no evidence either way — go measure) from
`subtle-risk` / `nit`. Most "problems" are overstatements; most danger is in the one `wrong`.

---

## 4. Harness skeleton

Pseudocode in a generic multi-agent harness: `agent()`, `parallel()`, and `schema:` are fan-out primitives to
map onto your own orchestrator, not a specific product.
```js
const GROUND = `GROUND TRUTH = ${CODE} + ${DATA}, NOT prior claims. This audit exists because <drift episodes>.
ALREADY ESTABLISHED (re-verify if you doubt): <constants/line-refs/data-splits>.
Be a ruthless skeptic; cite file:line or data for EVERY gap; return strict schema JSON.`
const finds = await parallel(LENSES.map(L => () => agent(GROUND + '\nYOUR LENS: ' + L.focus, {schema: FIND})))
const synth = await agent(GROUND + '\nSYNTHESIZE into an honest gap report; separate genuine errors from overstatements from sound; calibrated verdict.\n' + JSON.stringify(finds.filter(Boolean)), {schema: SYNTH})
return { finds, synth }
```
Diverse lenses beat redundant ones: e.g. (A) re-derive the MECHANISM from code independently; (B) audit a
shipped feature's FAITHFULNESS to what the engine will actually do; (C) audit the session's narrative CLAIMS
one by one. Scale lenses to the claim count; with "unlimited agents", one lens per claim-cluster + a second
independent synthesizer to reconcile.

---

## 5. Category-error guards (the traps this catches)
- **Self-correlation ≠ prediction.** "X predicts the final bucket 90%" can be geometric self-correlation, not
  a genuine predictive signal — name the variables and check they're actually different.
- **Confounded A/B.** A "clean comparison" where the arms differ in a hidden toggle (one factor left on in one
  arm) is not clean — flag it.
- **Frozen-vs-live.** A value the narrative calls "deterministic" may need a future event to materialize
  (an input that hasn't arrived yet, a precondition another process could satisfy first) — is the label honest?
- **A tooling bug upstream.** A "this value is a pure artifact" call can trace to a tie-handling bug in the
  metric — a wrong number, not a wrong world. Re-run the computation, don't re-reason from its output.

---

## 6. Principles
- **Recompute, don't re-reason.** If a claim rests on a number, re-run the number — a buggy result reasoned
  about ten times is still buggy.
- **Confidence is not evidence.** "I'm sure" from the model (or you) carries zero weight; file:line or data.
- **Calibrate, don't catastrophize.** Don't manufacture problems to look thorough; don't paper over the real
  one. The verdict is honest, not alarmist.
- **Separate error from framing.** Fixing an overstatement = changing how you SAY it; fixing a `wrong` =
  changing the code/conclusion. Don't conflate.

---

## 7. Provenance
A real project's model-reality audit + a shipped-feature faithfulness audit: 3 lenses (re-derive the
mechanism from code / audit a shipped feature's faithfulness to what the engine will actually do / audit the
session's claims one by one) → honest synthesis (real-errors vs overstatements vs sound + a
MODEL-SOUND/MINOR-GAPS/REAL-ERRORS verdict). It exists because the model's mental model diverged from reality
twice in one session (a statistic frozen by a tie-handling bug; a "the parameter locks at event start" claim
that was false). The project's spine — "model agents propose and cross-examine; the compiler/data DECIDE" — is
exactly this skill applied continuously. The interpretation-discipline rules live in `reproducible-analysis`.
Pairs with `rigorous-batch` / `architecture-audit` / `clean-room-rewrite` (reality-check verifies the premise
BEFORE they act on it).
