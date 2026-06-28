# Round 9 (hardening): the core claim survived; two shakier ones did not

**Pre-registered hardening of the round-8 blind-spot study.** Builders: Opus 4.8 /
Haiku 4.5. Judges: Opus 4.8 + Sonnet 4.6. 2026-06-27.

---

## What round 9 was for

Round 8 (n=5, one seam, one judge) produced three claims: (1) an integration-priming
prompt closes silent breaks; (2) self-review detects the gap ~2× worse than a fresh
external reviewer; (3) prevention dissociates from detection (the fix doesn't need
the awareness). Round 9 tested all three across **more seams, a capability ladder,
larger n, and a second judge** — pre-registered so the result could disappoint.

It did, usefully. **The core claim got stronger and sharper. Two of the three got
falsified or refined.**

## Method

4 new under-specified seams on an *affordance pattern* (a config knob that invites a
reflexive best-practice violating an unstated-but-obvious requirement, generalizing
staging's `autoAdd`): **pagination** (`pageSize` → cap → undercount), **caching**
(`cacheTtlMs` → stale read), **logfilter** (`minLevel` → drop events), **buffering**
(`batchSize` → lose on close). Each self-tested (golden passes / broken fails) before
any agent ran. Closure: baseline vs primed × Opus/Haiku × n=8. Detection: self vs
external × n=12 × Opus, every critique scored by **two** judges. Staging (round 8)
joins as the 5th seam.

## Result 1 — CLOSURE replicates, with two new bounds (CONFIRMED + refined)

Commit/correct end-to-end (Wilson 95% CI):

| seam | Opus base → primed | Haiku base → primed |
|---|---|---|
| pagination | 2/8 → **8/8** | 1/8 → 5/8 |
| caching | 0/8 → 4/8 | 0/8 → 2/8 |
| staging (r8) | 1/5 → 5/5 | (no break) |
| logfilter | 8/8 → 8/8 *(did not break)* | 8/8 → 8/8 |
| buffering | 8/8 → 8/8 *(did not break)* | 8/8 → 8/8 |

Pooled over the breaking seams (pagination + caching): **Opus 2/16 → 12/16 (~75%),
Haiku 1/16 → 7/16 (~44%)**. The priming effect is real and replicates. Two bounds
the hardening surfaced:

- **A capability floor in the *fix*.** Priming rescues Opus far better than Haiku
  (~75% vs ~44%). The one-line nudge degrades with model capability — below some
  point it helps but no longer closes.
- **Priming is not a panacea.** Caching is a *sticky* convention: even primed Opus
  recovers only 4/8. Pagination closes cleanly (8/8); caching half-closes.
- **Honest negative:** 2 of 4 new affordance-seams didn't break at all (agents
  recorded-all / flushed-on-close by default). Reliable-breaking under-specified
  seams remain hard to manufacture (a limitation seen every round). Breaking seams
  to date: staging, pagination, caching.

## Result 2 — the DETECTION ASYMMETRY did NOT replicate (RETRACTED)

Did the critique explicitly name the emergent requirement? (consensus = both judges)

| seam / mode | consensus (95% CI) |
|---|---|
| pagination · self | 12/12 (76–100%) |
| pagination · external | 12/12 (76–100%) |
| caching · self | 12/12 (76–100%) |
| caching · external | 11/12 (65–99%) |
| **pooled self** | **24/24 (~100%)** |
| **pooled external** | **23/24 (~96%)** |

Inter-judge agreement **47/48 (98%)**. **Self ≈ external ≈ ceiling.** Round 8's
"fresh eyes detect ~2× better" (53% vs 27%) was a staging-specific, n=5, single-judge
artifact. **Retracted as a general claim.**

## Result 3 — the dissociation, sharpened into the real finding

Round 8 framed it as "the fix doesn't need the awareness." Round 9 shows the cleaner
truth. On pagination/caching, at **baseline** the agents **break** (apply the
convention: Opus 2/16 correct), yet when **asked to enumerate** failure modes they
**name the exact requirement ~100% of the time**. So:

> **The knowledge is present; the attention is not.** Agents already know the
> requirement (they state it ~100% on request) but do not *act* on it unless
> attention is directed. It is not a detection blind spot (round 8's framing) and not
> a capability or knowledge gap — it is an **attention-allocation failure**. Directing
> attention (a one-line prompt, or an enumeration request) makes them both surface it
> and fix it.

This is the strongest, best-supported version of the whole project's thesis, and
round 9 *strengthens* it: ceiling detection proves the knowledge is there; the
baseline break proves it isn't deployed unprompted; priming proves direction closes
it.

## Net: what is now safe to claim

- **Silent multi-agent integration failures are an attention-allocation failure.**
  (Strongly supported; sharpened by round 9.)
- **A one-line integration-priming prompt closes them** — strongly for capable models
  on loose conventions; partially for weaker models and sticky conventions. (Bounded.)
- **There is a capability floor and a "sticky convention" ceiling on the fix.** (New.)
- **No general self-vs-external detection asymmetry.** Both detect at ceiling when
  asked. (Round-8 claim retracted.)

## Implication for Seasar (updated)

The external tool's value is **not** "fresh eyes see what you can't" (refuted —
self-detection is fine when asked). It is the **backstop for where the cheap prompt
under-performs**: weaker builder models (the floor), sticky conventions (caching),
and the runs where nobody thinks to prompt. Concretely: **prime every builder for
integration by default, and emit executable gates as the safety net** for the
residual the prompt misses. The gate — not the contract, not "fresh eyes" — is the
durable external artifact, because it fires whether or not attention was directed.

## Threats / scope

- Breaking seams n=3 (staging, pagination, caching); 2 affordance-seams didn't break.
- Closure n=8/cell, detection n=12/seam/mode, Opus+Haiku (Haiku = weakest model the
  harness can reach; an older/OSS floor is untested). Single-module isolation.
- Detection ran on the 2 new breaking seams (where the gap is real); judges are LLMs
  (98% agreement; not human-validated at scale).

## Reproduce
```
cd runs/seasar-multidomain-ownership/round9
sh run_closure.sh && node aggregate_closure.js
# detection: detection_results.json (self/external x 2 judges); see this file's tables
```
