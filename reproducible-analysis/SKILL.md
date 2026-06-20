---
name: reproducible-analysis
description: >
  Turn raw data exports into a STANDARDIZED, reproducible offline analysis REPORT — one versioned generator that
  computes the SAME sections/metrics every run (so results are comparable across runs and across growing data),
  partitions by the axes that matter, flags small samples, and ships with the INTERPRETATION DISCIPLINE that
  stops the common analysis errors (chasing the max-metric cell, over-reading noise, confusing per-item gains
  with total output, pooling incompatible experiment arms, treating a hindsight/offline number as if it were
  live). USE when you repeatedly analyze refreshed exports of the same kind and need every report to be
  apples-to-apples and honestly caveated; SKIP for a one-off ad-hoc number. The value is the FIXED report + the
  guardrails, not a clever one-time chart. Pairs with reality-check (verify a claim the report rests on) and
  rigorous-batch (change the upstream code that emits the data; measure-before-gate).
---

# Reproducible Analysis Report

A working method, not a tutorial. It replaces ad-hoc "look at the data and eyeball a conclusion" with a single
**versioned report generator**: point it at the latest export, get the SAME calculations every time, and read
them through a fixed set of guardrails so the recommendation is honest and comparable to last week's.

**The one idea:** the report is a PROGRAM, not a vibe. Same inputs → same sections → same metrics, every run.
That makes results comparable as data accumulates, makes the analysis auditable, and lets the hard-won
*interpretation lessons* live IN the tool (as guardrails and required caveats) instead of in someone's head.

---

## 0. When to use / not
USE: you analyze refreshed exports of the same shape repeatedly (a metrics dump, an experiment log, an export
from some upstream system); you need runs to be apples-to-apples over time; a recommendation will be made FROM
the numbers and must be honestly caveated. SKIP: a single fresh number you can read once; exploratory one-offs
where a fixed report would just be overhead — do that ad hoc, then PROMOTE it to a report if the question recurs.

This PRODUCES the standardized report + the reading. To verify a specific claim the report rests on, use
`reality-check`; to change the upstream code that emits the data, use `rigorous-batch`.

---

## 1. Pipeline
```
Define the standard report  →  Generate (run the versioned generator on the export)  →  Interpret (guardrails)  →  Caveat  →  Re-run as data grows
```
The generator is a real, committed script — not a notebook rewritten each time. Persist its output (a dated
report doc) so runs diff cleanly. As data accumulates the calculations DON'T change — only the sample grows;
that invariance is the whole point.

---

## 2. The standard report (every run computes the same thing)
- **Fixed sections.** Enumerate the sections once; each run emits all of them in the same order, so two runs
  diff line-for-line. A new question becomes a new PERMANENT section, not a one-off edit.
- **Partition by the axes that matter; never pool across them.** Split by the dimensions that change the answer
  (segment / cohort / variant / direction / time-bucket). Pooling two arms of an experiment into one number
  voids the comparison — keep them separate by construction.
- **Carry every precondition.** A headline number is meaningless without the conditions it was computed under;
  the report states them inline (the full configuration that produced this cell) so no one can quote it bare.
- **Flag thin cells.** Mark any cell below an n-threshold as noise; never present a small-sample cell as if it
  were a tuned parameter.
- **Long-format output for free pivoting.** Emit tidy long-format rows (one row per cell + its dimensions) so a
  spreadsheet / BI tool can pivot any view without re-running the generator.

---

## 3. Interpretation discipline (the lessons — do NOT skip; these are why the skill exists)
- **The max-metric cell is usually a trap.** Don't recommend whatever maximizes the headline metric if it
  violates a sanity floor (e.g. a reward:risk or precision:recall constraint). Optimize the metric *subject to*
  the floor, not nakedly — the naked max is typically fragile.
- **Small samples are noise — say so loudly.** Below the n-threshold the cell is a coin flip; never hand it back
  as a parameter. State exactly what data you'd need to make it real.
- **Per-item gain ≠ total gain.** A filter that raises per-item quality but cuts volume can LOWER total output.
  Always report BOTH the per-item metric AND the total (n × per-item), and condition the recommendation on the
  use-case — the right answer FLIPS with it.
- **Don't pool incompatible arms.** Two arms of an A/B (or two regimes/conditions) must not share one bucket;
  let the tool enforce the partition so the comparison stays valid by construction.
- **Separable vs entangled factors.** Some factors can be recorded as a SHADOW column in ONE run (a per-item
  toggle that changes nothing upstream) → cheap to compare both sides. Others reshape the whole pipeline
  upstream (a detection / parsing / selection-layer toggle) → you need TWO runs (or a parallel engine) and an
  offline diff. Know which kind you have BEFORE you claim a comparison is clean.
- **Hindsight / offline ceiling.** Numbers from replay, backfill, or a frozen snapshot are an UPPER BOUND vs
  live (latency, slippage, drift, repaint, lookahead). Always caveat; discount before acting; get a real-world
  sample to calibrate the gap.
- **The decision the owner owns stays the owner's.** Present options (and both directions) conditioned on the
  inputs the owner controls; don't silently bake in the judgment call they must make.
- **Beware look-ahead in descriptive labels.** A whole-period label computed with full hindsight is NOT an
  early-period predictor unless you tested it as one — and that test usually fails. Keep "describes a completed
  unit" separate from "predicts at the start"; never sell the first as the second.

---

## 4. Data-quality gate (before you fix any parameter)
Before turning a directional finding into a HARD parameter, require: enough samples ACROSS regimes/conditions
(not all from one), a real-world sample to discount the offline ceiling, and the relevant conditioning table.
Until you have them: **directional conclusions only — never hardcode a single tuned value.** This is the
analysis analogue of `rigorous-batch`'s *measure-before-gate*: ship the finding diagnostics-only, calibrate
from accumulated data, THEN enforce.

---

## 5. Principles
- **The report is a program.** Same inputs → same output; reruns are free; the DIFF between runs is the finding.
- **Comparability over cleverness.** A fixed, boring report you can trust across weeks beats a brilliant one-off
  you can't reproduce.
- **Caveat is part of the result.** An uncaveated number is wrong even when it's right; the ceiling, the
  sample size, and the preconditions ship WITH every headline.
- **Recompute, don't re-reason.** If a recommendation rests on a number, re-run the number on fresh data — a
  buggy or stale figure reasoned about ten times is still buggy. (This is `reality-check` applied to your own
  report.)
- **Promote, don't accrete.** A recurring ad-hoc question becomes a permanent section; the generator grows, the
  methodology stays fixed.

---

## 6. Provenance
Generalized from a domain analysis-report skill: a single offline report generator that turned refreshed data
exports into a standardized report (fixed sections; partitioned by the axes that mattered; small-sample flags;
long-format output for pivoting) plus a set of hard-won interpretation rules (optimize the metric under a
sanity floor; small samples are noise; per-item-vs-total flips with the use-case; never pool incompatible arms;
separable-vs-entangled factors; the hindsight/offline ceiling; look-ahead in descriptive labels; the decision
stays the owner's). The domain specifics and the bundled generator scripts were dropped; the cross-domain
methodology — *the report is a program, and the interpretation discipline ships inside it* — is what remains.
Pairs with `reality-check` (verify a claim the report rests on) + `rigorous-batch` (change the upstream code
that emits the data; measure-before-gate).
