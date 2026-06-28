# Does assigning cross-module responsibility (beyond types) prevent integration bugs in parallel multi-agent builds?

**A replicated, de-biased experiment across two models.** Builder models: Claude
Opus 4.8 and Claude Sonnet 4.6 · 2026-06-26 · Seas / Seasar research.

---

## Bottom line

Across **12 independent builds** (2 models × 3 runs × 2 arms, 6 parallel agents
each, 72 module builds total), giving the agents a contract that **assigns
cross-module ownership** — and *nothing else types can't already express* —
produced a **deterministic, model-robust** effect on one safety seam and **no
effect** on another:

- **Staging ownership — STRONG, replicated:** Arm A (ownership assigned) produced
  a working pipeline in **6/6** trials; Arm B (types only) produced a pipeline
  that **silently never commits** in **6/6** trials. Same mechanism every time.
  Fisher exact p ≈ **0.002**.
- **Redaction completeness — NULL once de-biased:** both arms leaked secrets at an
  **identical 19/48** rate in a stress test. The pilot's dramatic redaction win
  was an artifact of the contract enumerating secret formats; remove that and the
  effect vanishes. Reported as a refuted sub-hypothesis.

The honest headline: **a behavioral contract's value is responsibility
*assignment*, not capability.** It decides *who does a job* (and that prevents
silent integration breaks); it does not make the job *done better* — that tracks
model strength, not the contract.

## Why this study (vs the pilot)

A prior n=1 pilot (`../seasar-behavioral-contract-ab/`) found Arm A beat Arm B on
both a staging break and secret-leak rate (3/8 → 0/8). But the pilot's
`BEHAVIOR.md` **enumerated which secret formats to redact** — a reviewer can
fairly say Arm A was handed the answer on redaction. This study fixes three
threats:

1. **De-bias:** `BEHAVIOR.md` v2 pins *only ownership + wiring semantics* (who
   redacts, who stages, who guards, the error-model, the outcome mapping). It
   gives **no capability or coverage hints** — no list of secret types, git
   states, etc. The constitution (identical to both arms) still states the
   *requirements*; only the *owner* differs.
2. **Replication:** 3 runs per cell → rates, not points.
3. **Second model:** Sonnet alongside Opus → tests the "strong-model" threat and
   whether a weaker model *gaps* where Opus *over-guards*.

Apparatus (`types.ts`, `SPEC.md`, probes) is reused unchanged from the pilot; the
**only** changed input is `BEHAVIOR.md` → de-biased v2.

## Pre-registered predictions (logged before running the measurement)

1. Staging finding **survives strongly**.
2. Redaction finding **weakens but persists**.
3. Sonnet **diverges more** — possibly compile failures or *gapping* rather than
   over-guarding.

## Results

| cell | trials | compile | P0 commit | P1 leak | P5 merge-guard | stress leak (/24) |
|---|---|---|---|---|---|---|
| opus / armA   | 3 | 3/3 | **3/3** | 1/3 | 3/3 | 6  (25%) |
| opus / armB   | 3 | 3/3 | **0/3** | 2/3 | 3/3 | 7  (29%) |
| sonnet / armA | 3 | 3/3 | **3/3** | 2/3 | 3/3 | 13 (54%) |
| sonnet / armB | 3 | 3/3 | **0/3** | 3/3 | 3/3 | 12 (50%) |

Aggregates:
- **Commit success (P0):** Arm A **6/6**, Arm B **0/6**. Uniform mechanism: all
  six Arm B committers gate staging on `config.autoAdd` (default false) → stage
  nothing → refuse "nothing staged"; the daemon/approval assume "approved ⇒
  commits." Every module compiles; the *integration* is dead. Arm A's committers
  all stage `draft.files` unconditionally (the one thing `BEHAVIOR.md` pinned).
- **Redaction stress:** by arm, **19/48 vs 19/48** — *exactly equal*. By model,
  **opus 13/48 (27%) vs sonnet 25/48 (52%)**. Redaction completeness is
  **model-bound, not contract-bound.**
- **Merge-pause (P5):** 3/3 in every cell. Both arms over-guard (the constitution
  alarms loudly; agents each grab the guard). Not a differentiator.
- **Compile:** 12/12. No compile-failure confound in either model.

## Predictions scorecard (2 of 3 wrong — reported straight)

1. Staging survives strongly — ✅ **correct** (6/6 vs 0/6, both models).
2. Redaction weakens but persists — ❌ **wrong**: it **vanished** (19 vs 19). The
   pilot's redaction effect was the enumeration (deck-stacking), not ownership.
3. Sonnet gaps / fails to compile — ❌ **wrong**: Sonnet compiled 3/3, over-guarded
   like Opus (P5 3/3), and did **not** gap. It *did* diverge more, but on
   **capability** (2× the secret-leak rate), not coordination.

Being wrong on 2/3 is the point of pre-registering: the surviving claim is the one
the data actually supports.

## Interpretation

- **What a behavioral contract buys (supported): responsibility assignment.**
  Types underdetermine *who does the work* at a seam. When two agents fill that
  gap with different-but-individually-reasonable choices, the system compiles and
  silently malfunctions. Naming the owner fixes it — here, deterministically,
  across 12 builds and two models, with the contract adding *zero capability*.
  This is the clean, model-robust evidence for the distinctive half of Seasar's
  thesis (pin behavior/ownership, not just types) — and a concrete instance of
  Cognition's "actions carry implicit decisions, and conflicting decisions carry
  bad results."
- **What it does NOT buy (refuted): capability.** A contract that says "the
  messager redacts" does not make the messager's redaction *complete*. Coverage
  improvises either way and tracks model strength (Opus 2× better than Sonnet,
  regardless of arm). To get completeness you need either an explicit spec of what
  to cover (which is then the *spec* doing the work, as in the pilot) or a stronger
  model — not an ownership assignment.
- **Structural fact, replicated: unassigned safety responsibility fails by
  over-guarding, not gaps.** Both arms guarded the merge-pause in 12/12 trials.
  Even the weaker model over-guarded rather than gapped. The naive "they'll leave
  a hole" failure model is wrong for these models; the real risks are (a) divergent
  *functional* seams with no constitutional alarm (staging) and (b) capability
  shortfalls (redaction) — neither of which a type checker sees.

## Threats to validity (updated)

1. **Single task / domain.** One CLI ("Quietkeep"). The staging result is one
   instance of the ownership mechanism; generalization across domains is untested.
2. **n = 3 per cell.** The staging effect is total separation (6/6 vs 0/6,
   p≈0.002) so it is robust at this n; the redaction null is clear (19 vs 19) but a
   small *positive* effect could hide below this power.
3. **Two models, both frontier-ish.** A much weaker model might gap; untested.
4. **The ownership mechanism is shown at the staging seam specifically.** Other
   ownership seams (redaction owner, guard owner) did not produce a measurable
   *outcome* difference here because both arms converged on safe ownership for
   those (single-owner redaction, over-guarded merge-pause). Staging was the seam
   where ownership genuinely diverged.
5. The comparison is typed vs typed+ownership. No-contract (shown in prior work to
   fail on shapes) and runtime-mediated coordination are separate conditions.

## Future work (to reach a full paper)

- **More domains/orders:** repeat on the 4 other compiled orders (PaperPulse,
  Marginalia ×2, EchoIndex) — each needs a bespoke probe harness — to show the
  ownership effect is not Quietkeep-specific.
- **More runs + inferential stats** (≥10/cell) to bound the redaction null and any
  small effects.
- **Weaker models** (a small OSS model) to find where over-guarding flips to
  gapping.
- **Third arm — runtime-mediated coordination** (DeLM/STORM-style) to test
  Seasar's *static-vs-runtime* gap directly (the distinct next hypothesis).
- **Blind apparatus authorship** to fully retire experimenter bias.

## Reproduce

```
cd runs/seasar-behavioral-contract-study
( cd toolchain && npm i typescript@5 @types/node@20 )   # shared, symlinked
./run_study.sh            # compile + probe + 8-format stress over all 12 trials
node aggregate.js         # per-cell rates -> study_summary.json
```

Raw per-trial data: `study_results.jsonl`. Per-cell summary: `study_summary.json`.
De-biased contract: `fixtures/BEHAVIOR.md`. Built code: `trials/<model>-<run>-<arm>/`.
Pilot (enumerated contract, n=1): `../seasar-behavioral-contract-ab/`.

## One-paragraph abstract (for posting/publication)

We tested whether a contract that assigns *cross-module responsibility* — beyond
what a type system expresses — prevents integration bugs when N coding agents
build one codebase in parallel. Across 12 controlled builds (2 models × 3 runs ×
2 arms; 72 agent-built modules), a de-biased ownership contract made the
difference between a working pipeline (6/6) and one that compiles yet silently
never commits (0/6), deterministically and on both models (Fisher p≈0.002). The
same contract had *no* effect on a capability-bound property (secret-redaction
completeness: 19/48 leaks in both arms; completeness tracked model strength
instead, Opus 2× better than Sonnet). Unassigned safety responsibility failed by
*over-guarding*, never by gaps. Conclusion: in parallel multi-agent construction,
the load-bearing thing a contract adds over types is **who-does-what**, not
**how-well** — assignment prevents silent integration failure; capability is the
model's job.
