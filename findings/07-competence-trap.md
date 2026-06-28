# The competence trap is real but rare — it failed to replicate, and that re-centers everything on requirement crispness

**Capability ladder + a hardening attempt that came back negative.** Builders:
Claude Opus 4.8 → Sonnet 4.6 → Haiku 4.5. 2026-06-27. Seas / Seasar research.

---

## What happened (the honest arc of this phase)

1. **The staging seam produced a striking result.** At the under-specified
   Quietkeep staging seam, types-only builds broke on the *stronger* models
   (Opus/Sonnet Arm B committers gated on `autoAdd` — git's "commit only what's
   staged" convention — and silently no-op'd) and *worked* on Haiku (which took the
   spec literally: `git add <file>`). `autoAdd` references: Opus 32, Sonnet 14,
   Haiku 0. That looked like a "competence trap": capability imports a convention
   that breaks an ambiguous seam. (Detail below.)

2. **I tried to harden it on two new under-specified seams** with different
   reflexive conventions — `pagination` ("bound result sets / default limit") and
   `eventcount` ("dedup / idempotency", the very convention that was *correct* in
   the checkout domain). 60 builds across the full ladder.

3. **It did not replicate. At all.** Zero models imported the convention —
   including Opus. Every Arm B agent returned all records / appended every event,
   explicitly citing the stated requirement.

| seam | Opus B | Sonnet B | Haiku B | convention imported? |
|---|---|---|---|---|
| pagination | 5/5 ✓ | 5/5 ✓ | 4/5 (1 compile) | **never** (all "returns-all") |
| eventcount | 5/5 ✓ | 5/5 ✓ | 5/5 ✓ | **never** (all "appends-every") |

## Why it didn't replicate — the real mechanism

The difference is **whether the positive requirement was stated**.

- In `pagination` and `eventcount` I wrote a crisp *positive* end-requirement into
  the shared constitution — "the count MUST equal the total, **no records
  omitted**" / "every occurrence **must be counted**." That phrasing *logically
  precludes* the divergent convention (a cap/dedup obviously omits/coalesces). So
  every model — Opus included — **derived the correct behavior and did not apply
  the convention.** Capability was irrelevant.
- In `staging`, the shared constitution stated only a *negative* constraint ("never
  commit without approval"). The *positive* requirement ("a commit must actually
  result / the committer stages the approved files") was **never stated** — it
  lived only in Arm A's behavioral contract and in the probe. So a capable Arm B
  committer, with no stated positive requirement, fell back to its sophisticated
  default (the git convention) — which silently failed the unstated need. The weak
  model's naive default happened to match it.

**So the "competence trap" is not an independent law that "stronger models break
ambiguous seams." It is a *symptom of an unstated positive requirement.*** When
the positive requirement is stated crisply, all models comply regardless of
capability; when it is unstated, behavior is driven by each model's default, and a
sophisticated default can diverge where a literal one accidentally fits. The trap
needs a requirement that is *testable but unstated* — a narrow, hard-to-hit
condition. Two deliberate attempts to manufacture it failed precisely because, to
test fairly, I stated the requirement.

## What is robust (re-confirmed, now across ~9 seams / 8 phases)

The one lever that has survived every phase — de-biasing, multi-domain, the
capability ladder, and this failed hardening — is **stating the positive
requirement crisply and testably (a gate).** Everything else reduces to it:

- Owner-determining / crisply-stated seams (txn, checkout, pagination, eventcount):
  contract redundant at **every** capability tier — models derive the behavior.
- The single under-specified seam where the contract mattered (staging) mattered
  **only because the constitution under-stated the positive requirement.** The
  contract's entire contribution was to supply that missing requirement — which a
  line in the spec, or better a testable gate, supplies more reliably and
  verifiably.

## Honest status of the "competence trap"

- **Real as an instance:** the staging data is genuine and code-confirmed
  (Opus/Sonnet import the git convention and break; Haiku doesn't). Capability
  *can* hurt at an unstated-requirement seam.
- **Not general:** 0/2 replication on new conventions. It is rare and depends on
  the testable-but-unstated condition, not on capability per se.
- **Best understood as:** a vivid special case of *under-specification* — the
  project's through-line — where the under-specified thing is a positive
  requirement and the filler is a model's capability-dependent default.

I am **retracting the earlier framing** ("contracts matter MORE at the frontier")
as overgeneralized from a single seam. The supported claim is narrower and
sharper: **state your positive requirements as testable gates; that immunizes
builds against divergent model defaults at every capability tier. A behavioral
contract helps only as an (unverifiable) stand-in for a requirement the spec
omitted.**

## Implication for Seasar

Seasar's value is not "behavioral contracts" and not "contracts for strong
models." It is **surfacing the omitted *positive* requirements at each seam and
emitting them as testable, compiler-authored gates.** That is the part every phase
of this study supports. The behavioral-contract prose is a weaker substitute for
it, and the competence-trap "frontier" upside does not hold up to replication.

## Threats / scope

- 2 replication seams, n=5/cell, 3 models. A negative result on 2 conventions does
  not prove the trap never generalizes — but combined with the mechanism (stated
  vs unstated positive requirement) it strongly suggests the trap is a
  specification artifact, not a capability law.
- The staging instance is committer-only isolation (corroborated by the study's
  full-build Opus result). The new seams are single-module isolations by design.

## Reproduce

```
cd runs/seasar-multidomain-ownership
sh staging/../run_staging.sh && node aggregate_staging.js            # the one trap instance
cd competence-trap && sh run_ct.sh && node aggregate_ct.js           # the failed replication
```
Raw: `staging_results.jsonl`, `competence-trap/ct_results.jsonl`. Built modules
under `staging/trials/...` and `competence-trap/<seam>/trials/...`.
