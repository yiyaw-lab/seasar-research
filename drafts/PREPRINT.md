# Attention, not coordination: silent integration failures in parallel multi-agent code generation are an attention-allocation failure a prompt removes

*Working paper / extended abstract (v2, hardened by round 9). See Limitations for n.
Builders: Claude Opus 4.8, Sonnet 4.6, Haiku 4.5. Judges: Opus 4.8 + Sonnet 4.6.
Author: [you].*

## Abstract

When N LLM agents build one codebase in parallel, a class of failures slips past
type checks and review: the system compiles but is silently wrong end-to-end because
a *positive* cross-module requirement was never stated and an agent's reflexive
default violates it. The standard prescription is richer inter-agent *contracts*.
Across nine controlled rounds (~600 agent-built modules) we find contracts are mostly
redundant — what matters is whether the positive requirement is stated — and, more
sharply, that the failure is **an attention-allocation failure, not a capability,
knowledge, or coordination problem**. At baseline, agents apply a convention that
breaks an unstated-but-obvious requirement; yet when *asked to enumerate* failure
modes they name that exact requirement **~100%** of the time (two-judge consensus,
98% agreement). A one-line prompt directing attention to end-to-end failure modes
takes correctness from **~12% to ~75%** on breaking seams for a strong model. Two
bounds: the fix has a **capability floor** (weaker model rescued ~44%) and a
**sticky-convention ceiling** (caching half-closes). A self-vs-external detection
asymmetry seen at n=5 in an earlier round **did not replicate** (both ~ceiling). The
lever is directing builder attention and backing it with executable gates — not
contracts, not "fresh eyes."

## 1. Setup

Small hermetic systems where one or more agents implement a module against shared
types, then the assembly is exercised by a runtime probe checking an end-to-end
property. Each probe is validated before any agent runs (a golden reference passes, a
broken one fails). The under-specified seams follow an *affordance pattern*: the spec
exposes a config knob (e.g. `pageSize`, `cacheTtlMs`, git's `autoAdd`) that invites a
reflexive best-practice which violates an unstated-but-obvious requirement.

## 2. What does not matter (rounds 1–7, summarized)

Behavioral/ownership contracts looked decisive in a pilot but the effect was an
apparatus artifact; de-biased and replicated across domains, types-only agents
*derive* the correct design whenever the stated requirement determines it. One seam
broke more on stronger models (they import a real convention) — a "competence trap" —
but it failed to replicate when the positive requirement was stated. The durable
through-line: silent integration failures come from *unstated positive requirements*.

## 3. The attention finding (rounds 8–9)

On the seams that reliably break (staging, pagination, caching):

**Closure** (correct end-to-end; baseline → integration-primed):

| seam | Opus | Haiku |
|---|---|---|
| pagination | 2/8 → 8/8 | 1/8 → 5/8 |
| caching | 0/8 → 4/8 | 0/8 → 2/8 |
| staging | 1/5 → 5/5 | (no break) |

Pooled breaking-seams: Opus **2/16 → 12/16 (~75%)**, Haiku **1/16 → 7/16 (~44%)**.

**Detection** (named the requirement when asked; two-judge consensus, n=12/cell):
self **24/24 (~100%)**, external **23/24 (~96%)**, inter-judge agreement **98%**.

**The dissociation, correctly stated.** At baseline the agents break (apply the
convention); asked to enumerate, they name the requirement ~100%. The knowledge is
present; the attention is not. Directing attention (a prompt) makes them both surface
it and fix it. This is an attention-allocation failure — not detection-blindness
(an earlier round's framing), not capability, not knowledge.

## 4. Bounds (round 9)

- **Capability floor in the fix:** priming rescues Opus (~75%) far better than Haiku
  (~44%). The nudge degrades with model strength.
- **Sticky-convention ceiling:** caching recovers only 4/8 even primed; pagination
  closes fully. The prompt is not a panacea.
- **Retraction:** the self-vs-external detection asymmetry (53% vs 27%, n=5, one
  judge) did not replicate; both detect at ceiling.
- **Negative:** 2 of 4 new affordance-seams did not break (agents defaulted
  correctly). Reliable-breaking under-specified seams are hard to construct.

## 5. Discussion

The "self-reflection vs external verification" debate for agentic systems is usually
posed as reflection helping or not. Our data give a sharper account: the relevant
deficit is not the agent's ability to *detect* a gap (near-ceiling when asked) but
its allocation of *attention* to integration when generating. A generic attention
nudge closes the dominant failure class cheaply, bounded by model capability and
convention stickiness. For tooling: prevention is a builder-side prompt; the durable
external artifact is an **executable gate** that fires whether or not attention was
directed — the backstop for weak models, sticky conventions, and the runs where no
one thinks to prompt. Inter-agent contracts and "fresh-eyes" review are not where the
value is.

## 6. Limitations

Three reliably-breaking seams; two constructed seams did not break. Closure n=8/cell,
detection n=12/cell; Opus/Sonnet/Haiku (Haiku = weakest model the harness reaches; an
older/OSS floor untested). Single-module isolation for the headline seams. Judges are
LLMs (98% agreement, not human-validated at scale). Extended abstract, not a full
empirical paper.

## 7. What would make it a full paper

More reliably-breaking seams (harder to construct than expected — a finding in
itself); n≥30 with pre-registered analysis; a human-validated judge subset; full-build
(not isolation) replication; an older/smaller model to locate the capability floor
precisely; and an ablation separating "attention prompt" from "self-revision."

## 8. Conclusion

In parallel multi-agent code generation, the dominant silent-integration failure we
could construct is an attention-allocation failure: agents know the requirement but
don't deploy it unprompted. A one-line prompt closes it — strongly for capable models
and loose conventions, partially otherwise. Direct builder attention; back it with
executable gates. Contracts and fresh-eyes review are not the lever.

## Artifacts

Method, raw per-trial data, self-tested harnesses, and built modules:
github.com/yiyaw-lab/seasar-research.
