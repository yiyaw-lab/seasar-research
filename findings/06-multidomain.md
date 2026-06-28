# When does a behavioral/ownership contract actually prevent integration bugs in parallel multi-agent builds?

**Multi-domain replication + a pre-registered convention-divergent test (loop
closed).** Builder model: Claude Opus 4.8 · 2026-06-26 · Seas / Seasar research.
Read with the pilot (`../seasar-behavioral-contract-ab/`) and the 2-model study
(`../seasar-behavioral-contract-study/`).

---

## Bottom line

The 2-model study found one strong effect: assigning *who stages files* flipped a
Quietkeep build from working (6/6) to silently broken (0/6). This phase tested
whether that generalizes — across **4 new domains** (queue, checkout, etl, and a
**deliberately convention-divergent** transaction domain), 2 arms, 3–5 runs, **56
agent-built modules**. **It did not generalize in a single domain.** Typed-only
(Arm B) converged on the correct owner everywhere.

**The complete picture — every behavioral/ownership seam tested in this work:**

| seam | Arm A | Arm B | broke? |
|---|---|---|---|
| **Quietkeep staging** (git "commit only staged" convention) | 6/6 | **0/6** | **YES — the only break** |
| Quietkeep redaction (de-biased) | 19/48 | 19/48 | no (null; pilot was deck-stacking) |
| Quietkeep merge-pause | ✓ | ✓ | no (both over-guard) |
| queue — who acks | 3/3 | 3/3 | no |
| checkout — charge idempotency | 3/3 | 3/3 | no |
| etl — load dedup | 3/3 | 3/3 | no |
| txn — unit-of-work (convention-divergent by design) | 5/5 | 5/5 | no |

**One break across seven rigorously-tested seams. Four deliberate attempts to
manufacture a convention-divergence break all converged.**

## The sharpened principle (what the loop-closer revealed)

I built the txn domain specifically to be convention-divergent: *who owns the
transaction boundary* — the service (unit-of-work: begin → debit → credit → commit
once) vs the repository (DAO: each method self-commits). Both are real, widely
taught conventions, and — verified by self-test — **each wrong choice fails a
different probe**: if nobody commits, all writes are silently lost; if the repo
self-commits, atomicity breaks (debit persists without credit). Over-guarding does
*not* save you here. It looked like the perfect break.

It didn't break. And the Arm B decision records show **why**: the agents *derived*
the owner. Verbatim (Arm B, no contract): *"a self-committing DAO would make the
debit durable before the credit could fail"* → therefore the service must own the
unit of work. **The atomicity invariant logically determines the owner, and
capable agents reason their way to it.**

That is the real mechanism, and it unifies all seven results:

> A stated correctness requirement that is crisp enough to make the silent break
> **detectable** is almost always also crisp enough to **determine the owner** —
> and a capable model derives it. So wherever the requirement determines the owner
> (atomicity, charge-once, one-row-per-id, ack-once), the ownership contract is
> **redundant**: agents converge. The contract is load-bearing only in the gap —
> where the requirement leaves **slack that does not determine the owner**, and a
> divergent prior fills it wrongly.

**Quietkeep staging was that gap.** "Stage the approved files / never commit
without approval" is fully satisfied by "commit only what the user pre-staged"
(autoAdd off) — a silent no-op that violates *no stated invariant*. The spec never
made "the pipeline must actually produce a commit" a crisp, owner-determining
requirement. The ownership contract didn't add magic; it **patched the missing
crispness by naming the owner.**

## What this means for Seasar (re-weighted)

The load-bearing thing is **requirement crispness — an owner-determining,
ideally *testable* invariant** — not ownership prose. And a crisp testable
invariant is better supplied by a **compiler-authored gate** (a check the build
must pass) than by a behavioral-contract sentence, because a gate is verifiable and
a sentence is not. This **re-weights Seasar's own design**: its *gates* half
(testable invariants a non-agent runner enforces) is the part this evidence
supports; its *behavioral-contract prose* half is mostly redundant wherever the
requirement already determines the owner — which, with a capable model, is almost
everywhere.

Honest one-liner for Seasar: **don't sell "behavioral contracts make multi-agent
builds reliably better." The data supports "crisp, testable, owner-determining
requirements (gates) prevent silent integration breaks; ownership prose is cheap
insurance only for the rare under-specified seam — and a weak substitute for a
gate."**

## The cross-phase arc (pilot → study → multi-domain → loop-closer)

1. **Pilot (n=1):** looked decisive — staging break + 3/8→0/8 redaction.
2. **Study (de-biased, 2 models):** redaction win was **deck-stacking** → null
   (19/48 both arms). Staging win **robust** (6/6 vs 0/6, p≈0.002).
3. **Multi-domain (3 domains):** staging-type effect **did not generalize** — Arm
   B over-guarded safely (3/3 everywhere).
4. **Loop-closer (txn, convention-divergent by design):** still null (5/5) —
   agents derived the owner from the atomicity invariant. Revealed the real
   mechanism: **owner-determinacy of the stated requirement**, not the contract.

Each phase tempered the previous one. The early "behavioral contracts are a big
win" reading did not survive contact with replication.

## Threats to validity

1. **All Opus.** The whole convergence story rests on a capable model *deriving*
   the owner. A weaker model might fail to derive it and gap — in which case
   ownership prose would earn more keep. The study already showed a model effect
   (Sonnet 2× redaction leaks); owner-derivation is plausibly model-bound too.
   **This is the most important open question.**
2. **I could not manufacture a convention-divergent break in 4 tries.** Either such
   breaks are genuinely rare with capable models, or my domains kept stating
   owner-determining requirements (the txn decision records suggest the latter is
   the mechanism, not a flaw). The one natural break (staging) arose from a tool
   convention I did not design.
3. **n=3–5 per cell; small domains; single seam each.** The staging break is a
   single, if clean, data point.
4. queue's SPEC leaked the owner (noted in-line) — weak domain; the txn domain is
   the clean convention-divergent test.

## Reproduce

```
cd runs/seasar-multidomain-ownership
( cd toolchain && npm i typescript@5 @types/node@20 )
./run_multidomain.sh && node aggregate_md.js     # all 4 domains
```
Raw: `md_results.jsonl`. Summary: `md_summary.json`. Apparatus + built modules per
domain under `<domain>/fixtures/` and `<domain>/trials/`. Each harness was
self-tested (golden passes / broken fails) before agents ran.

## The next test that would matter most

Re-run the **staging** seam (the one real break) and the **txn** seam (the clean
convention-divergent design) **on a weaker model** (Sonnet, then a small OSS
model). Prediction from this work: as model capability drops, owner-derivation
fails more often, Arm B breaks appear in more seams, and ownership prose / gates
start to earn their keep. That maps the capability frontier at which behavioral
contracts transition from redundant to load-bearing — the actually-useful result
for deciding when to pay for them.
