---
name: clean-room-rewrite
description: >
  Reimplement or REPLACE a component and PROVE it is behavior-equivalent (often bit-exact) to the original via
  an executable differential-equivalence harness: a faithful ORACLE of the original and the new engine(s) run
  through identical edge-stressing inputs, outputs diffed event-by-event — so a rewrite, modernization, or
  hot-path optimization lands with ZERO behavioral regression. Optionally adds clean-room ISOLATION
  (implementers see ONLY a behavioral spec + interface, never the original source) + an expression-similarity
  audit for a documented, defensible provenance. USE when: you must rewrite a risky/legacy/auto-grown component
  without regressions; you want N independent implementations cross-checked against a golden oracle; or you
  need a documented independent reimplementation. Multi-agent (oracle + blind harness + isolated implementers +
  runner fix-loop + judge), schema'd, gated by a REAL compile+run where bit-exact output equality — not any
  model's opinion — decides. Pairs with rigorous-batch (which TRANSPLANTS the winner into production). Its
  cleanest, highest-value use is producing a PROVABLY-INDEPENDENT implementation of a PUBLIC concept/algorithm
  — generating the evidence (isolation log + similarity audit + provenance) that rebuts an
  access-plus-similarity copying presumption (§3). NOT a tool for shedding a license you accepted; not legal
  advice. Validated on a real detection-layer reimplementation (ZERO differential mismatches across an
  edge-stressing suite, structurally independent implementers).
---

# Clean-Room Rewrite / Differential-Equivalence

A working method, not a tutorial. It produces a NEW implementation of a component plus **executable proof**
that it behaves identically to the original — so a rewrite, modernization, or replacement of a derived
component lands with zero behavioral regression and a documented, defensible provenance.

**The one idea:** behavior is described in a neutral SPEC; isolated implementers build from the spec ALONE;
a blind harness runs the new engine(s) and a faithful ORACLE of the original through identical input streams
and **diffs the output event-by-event**. Bit-exact equality across an edge-stressing suite — not consensus —
is the gate. (The same harness technique proves any behavior-preserving change, e.g. a refactor's
"output-identical" claim: run old vs new on real inputs, diff, require 0 mismatch.)

---

## 0. When to use / not
USE: rewrite a fragile/legacy/auto-grown component you can't safely hand-edit; modernize/optimize a hot path
with a proof of equivalence; get N independent designs cross-validated against a golden oracle; or produce a
**provably-independent implementation of a PUBLIC concept/algorithm** — one you can show you designed rather
than half-remembered from someone's specific code (§3 explains why this is the clean, high-value case). SKIP:
trivial components; when a unit test already pins behavior cheaply; when no original exists to be equivalent TO
(then it's a normal rigorous-batch build). NOT for escaping a license you actually accepted on code you
incorporated — a different problem this method does not solve (§4).

This skill PRODUCES the proven engine; **`rigorous-batch` TRANSPLANTS it** into the production file under its
own ground-truth gate. Keep them separate.

---

## 1. Pipeline
```
SpecAudit  →  Build (oracle ‖ blind-harness ‖ isolated implementers)  →  Verify (compile+run, fix-loop)  →  Judge (winner + similarity audit)  →  hand winner to rigorous-batch (transplant)
```

### 1a. SpecAudit — write + leak-check the behavioral spec
Author a SPEC (exhaustive behavior: every edge — warm-up, ties, NaN, inclusive-vs-strict comparisons,
evaluation ORDER, output/event formats, idempotence/rollback) + a CONTRACT (the I/O event grammar) + an
INTERFACE file. Then an **IP-leak auditor** (may read everything) checks the SPEC/CONTRACT for copied
EXPRESSION — distinctive identifier names, copied comment sentences, code fragments from the original.
Behavior is not protectable; expression is. Neutralize every leak (keep behavior identical). Generic
domain vocabulary is fine.

### 1b. Build — three agent roles, run in parallel
- **Oracle adapter** (sees EVERYTHING; its output is test-only, never ships): faithfully ports the ORIGINAL
  logic behind the interface (copying old code is fine here — it's the golden reference). Document the
  state-mapping (original state → snapshot fields).
- **Blind test harness** (ISOLATED — spec+interface only): deterministic synthetic generators (fixed seeds)
  that STRESS the edges — random walks at varied step sizes, strong monotone runs, tight oscillation, flat/constant runs,
  staircases with EXACT ties, boundary `==` cases, near-equal repeats (tolerance), gaps; a run MATRIX over
  params × gate combinations (at least all corner combos + mixed per-step toggles); a **churn/idempotence**
  mode (capture → restore → re-process a not-yet-finalized record N times → only the final cycle counts; compare against
  the same engine fed finalized records only). Driver runs oracle vs each new engine through identical streams and
  writes a per-suite report: PASS/FAIL, mismatch counts, and the FIRST K mismatching event lines WITH the
  triggering inputs. **Reports stay behavioral-only — never echo engine source.** Exit 0 iff all pass.
- **N isolated implementers** (ISOLATED — spec+contract+interface only): each designs the engine THEIR OWN
  way (own state layout, own naming, own algorithm family — nudge implementer B toward a DIFFERENT family,
  e.g. monotonic-deque vs per-element rescan, so independence is structural). Honor every spec edge + the
  allocation/rollback discipline. Each ends with an **honesty section** listing every file read + command run.

### 1c. Verify — compile + run the suites, loop fix rounds
A runner copies the implementers' files into the harness, compiles it, then runs the full suite. Attribute every
compile error to its file; the runner MAY fix harness/oracle files but NEVER the clean-room engine files
(those go back to their isolated implementer with ONLY their own file + the behavioral mismatch report).
Loop (≈5 rounds) until every suite is bit-identical. **This is the keystone: a real compiler + a real diff,
model-independent.**

### 1d. Judge — pick the winner + similarity audit
Among engines that PASS (a passer beats any non-passer), judge on hot-path efficiency / clarity / snapshot
robustness / transplant-friendliness. Then an **expression-similarity auditor** (may read everything)
compares the winner against the original sources for independence of structure/naming/idioms/decomposition
(behavioral equivalence is intended — flag only similarity BEYOND what shared behavior forces: the merger
doctrine), and emits a 3-sentence **provenance statement** for a PROVENANCE doc.

---

## 2. The isolation discipline — and its real limits
Each implementer's ONLY permitted inputs are the SPEC + CONTRACT + INTERFACE (+ its own output + a named
report). **Do NOT** read any other file, the original source, the web, or git; do NOT recall the original from
memory. Where the spec leaves freedom, the implementer makes its OWN choice. The honesty section records every
file read and command run.

Be honest about how strong this is. A real clean-room defense contemplates a **strict information barrier
between separate parties**: one side reads the original and writes the spec, a *different* side implements with
no access. A
single operator who authors the spec/oracle from the original AND also directs the "isolated" implementers is
not that — the wall is only as good as it is in fact, and an honesty section is an audit aid, not proof. So
this discipline is excellent for **zero-regression confidence** (the implementer genuinely built from the
spec); what makes it legally clean, though, is feeding it a PUBLIC concept rather than a protected original —
see §3.

---

## 3. Why this holds up — provable independent creation
The clean, high-value use of this skill is producing an implementation of a PUBLIC concept that you can PROVE
you wrote independently.

**The mechanism it buys.** Independent creation is a complete defense to copyright — copyright restricts
copying, not arriving at something similar on your own. The catch is evidentiary: infringement is inferred from
**access + substantial similarity**, so once you have seen an implementation, any similarity raises a
presumption that you copied and shifts the burden onto you to prove you didn't. Seeing is not copying — but
after seeing, proving it gets hard. This skill manufactures the rebuttal: an isolation log, an
expression-similarity audit, and a provenance statement — affirmative evidence the work was designed from a
public spec, not transcribed.

**What makes the input clean.** Feed the pipeline a PUBLIC, unprotectable concept/algorithm (a documented
definition, a textbook method). The isolated implementer starts from that public spec; no protected original
ever enters, so there is nothing to "escape" — clean by construction. (Rolling-max is a public algorithm, but
a given library's monotonic-deque implementation is recognizable expression: study that code, write your own,
and your version can unconsciously echo it. Here, you — who saw it — write the spec from the PUBLIC algorithm;
an implementer who never read that library builds it; the similarity audit confirms the result stays within the
similarity that shared behavior forces — the merger doctrine. The output is provably your design, not a
half-remembered copy.)

**The one linchpin.** Write the spec from the PUBLIC concept — never from a specific implementation you
admired. If protected expression leaks from your memory into the spec, the isolation is wasted. That is what
§1a's IP-leak audit on the SPEC and the implementer's "don't reproduce the original from memory" rule defend —
doubly important for LLM implementers, which may have trained on the very public implementations in question.

---

## 4. Non-goals — what this is NOT for
Drawing the red line explicitly, so there is no ambiguity about scope:
- **Not a license-stripper.** This skill does NOT shed, strip, or work around a license on code you have
  accessed and accepted. If you have pulled in licensed or copyleft code and want the obligation gone, this is
  the wrong tool — that is a licensing decision, not an engineering one. Stop and consult an IP lawyer.
- **Not a non-infringement certificate.** It produces *evidence* of independent creation; it does not by
  itself prove any specific result is non-infringing. The idea/expression split and the independent-creation
  defense are solid in principle, but whether a given project clears is fact-dependent.
- **Not a way to hide provenance.** Its entire value is *adding* an audit trail (isolation log, similarity
  audit, provenance statement) — never removing one. If you are reaching for it to make something look clean
  rather than to build something clean, it is the wrong tool.

This skill is not legal advice; for anything you intend to rely on commercially, have qualified counsel review
the specific facts first.

---

## 5. Principles
- **Bit-exact diff > consensus.** A unanimous agent panel can be unanimously wrong; the harness can't be.
- **Edge-first generators.** The mismatches live at ties, NaN, warm-up, boundary `==`, and churn — generate
  those deliberately, don't hope a random walk hits them.
- **Independence is structural.** Different state layout + different algorithm family per implementer; the
  similarity audit verifies it, the isolation discipline earns it.
- **The oracle is faithful, not clever.** Port the original exactly (copying is fine) — its only job is to be
  the ground truth, then it's discarded.
- **Provenance always.** Emit the provenance statement and keep the audit trail — the value is the evidence
  you ADD, never anything you remove (§4).

---

## 6. Provenance
A real detection-layer clean-room rewrite: SpecAudit (IP-leak check) → Build (oracle adapter + blind
harness + multiple isolated implementers, divergent algorithm families + a pure-functions implementer) → Verify (compile+run
equivalence suites, fix-loop) → Judge (winner + expression-similarity audit, independent=true). Differential
harness = ZERO mismatches; a structurally independent reimplementation, an expression-similarity audit, and a
provenance doc shipped (any licensing question is separate — see §4). The same
differential-harness technique later produced a
refactor's executable proofs across large generated test suites, all 0 mismatch. Pairs with `rigorous-batch` (transplant) + `architecture-audit`
(map before rewrite).
