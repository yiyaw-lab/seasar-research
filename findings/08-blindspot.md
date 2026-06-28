# Silent multi-agent integration failures are an attention failure — and prevention dissociates from detection

**The final study.** Builders: Claude Opus 4.8 / Sonnet 4.6 / Haiku 4.5. Judge:
Sonnet 4.6. 2026-06-27. Seas / Seasar research.

---

## The question

Eight phases established: silent integration breaks in parallel multi-agent
builds come from *unstated positive requirements*; stating them (a gate) fixes it.
The open, Seasar-make-or-break question: **can the omitted requirement be
*detected* before the build — and is that blind spot *self*-detectable (a cheap
self-critique prompt closes it) or only *externally* detectable (you need an
outside process, i.e. a compiler)?**

Tested on the one confirmed blind-spot seam (Quietkeep *staging*: the
under-specified "who stages so a commit actually results"), committer-only
isolation, across the capability ladder.

## Two measurements

**CLOSURE — does a prompt fix the build?** (commit-success on the happy path)

| condition | Opus | Sonnet | Haiku |
|---|---|---|---|
| build only (baseline) | 1/5 | 1/5 | — |
| build + "enumerate integration failure modes" | **5/5** | **4/5** | **5/5** |
| build + self-critique + revise | **6/6** | **6/6** | — |

Same model, SPEC, and harness — the only change is the prompt framing. Commit-
success jumps from **~20% to ~96%** the moment the builder is asked to consider
end-to-end failure modes.

**DETECTION — does the critique *explicitly name* the gap?** (judge-scored, /5)

| | Opus | Sonnet | Haiku | total |
|---|---|---|---|---|
| **self** (built it, then reflected) | 2/5 | 1/5 | 1/5 | **4/15 (27%)** |
| **external** (fresh eyes, spec only) | 4/5 | 1/5 | 3/5 | **8/15 (53%)** |

## The three findings

1. **It's an attention failure, not a capability or knowledge gap.** The builders
   always *could* stage correctly; in build-only mode they didn't (they reflexively
   gated on `autoAdd`), but a single integration-priming prompt closed it across
   all three models — including Haiku. You do not need a stronger model or a
   contract; you need to direct attention at integration.

2. **Explicit self-detection has a genuine blind spot.** Reflecting on its own
   build, the builder named the actual gap only **27%** of the time. A fresh
   external reviewer named it **~2× more often (53%)** — the "you can't see your
   own blind spot, but fresh eyes can" effect, measured. (Noisy at n=5: external >
   self for Opus and Haiku, a wash for Sonnet.)

3. **The novel part — prevention dissociates from detection.** The priming prompt
   *closed the build* (~96% commit) far more than it *produced explicit awareness*
   (~27% named the gap). Builders staged correctly while their own enumerations
   talked about *other* risks (over-staging, path encoding) and never named the
   staging requirement they had just satisfied. The judge transcripts confirm it:
   "the actual committer stages unconditionally — exactly what the requirement
   demands — but the enumeration never articulates why." **The fix runs through
   implicit carefulness, not through understanding.**

## What it means for Seasar (the product decision)

- **Prevention is cheap and belongs in the builder loop, not a compiler.** A
  "before you finish, consider how this could silently fail when assembled"
  step closes the dominant failure class at ~zero cost. Seasar should ship that as
  a default in any agent it orchestrates. The behavioral-contract apparatus is not
  where the value is.
- **The external compiler's defensible niche is AUDIT + VERIFICATION, not building.**
  Fresh-eyes detection beats self ~2×, and *explicit* detection is where the
  residual misses live (even external missed ~half). So Seasar's durable value is
  as the *outside* process that (a) enumerates the integration requirements a
  builder won't surface about its own work, and (b) emits them as *executable
  gates* that catch the failure regardless of whether anyone reasoned about it —
  because the build dissociates from the awareness. That is a verification role, and
  it is exactly what a compiler that "produces, does not execute" should own.
- **Net:** reposition Seasar from "behavioral-contract compiler" to
  "**integration-attention + external gate generator**": prime the builders, and
  generate the executable end-to-end gates that an inside view provably misses.

## Why this is novel / publishable

- A clean, counterintuitive, *dose-response* result: silent multi-agent
  integration failures are an attention artifact a one-line prompt removes.
- A measured **self vs external blind-spot asymmetry** in agentic code review.
- The **prevention/detection dissociation** — the fix doesn't require the
  awareness — which bears directly on the live "self-reflection vs external
  verification" debate for agentic systems, and on test-/gate-first agentic coding.

## Threats to validity

- **One seam family (staging), committer-only isolation, n=5–6/cell, 3 models, one
  judge.** The closure dose-response is large and consistent (robust); the
  detection asymmetry is directional and noisy (Sonnet is a wash). Not a full
  paper as-is — workshop / NIER grade. Hardening = more seams, larger n,
  pre-registration, multiple judges, human-validated judge.
- **Judge is an LLM (Sonnet).** Target defined crisply; judge transcripts spot-
  checked and coherent, but a human-validated subset would strengthen it.
- **"Priming closes it" may partly include implicit self-revision** during the
  enumerate step. The robust claim is "any integration framing closes it"; the
  build-vs-awareness dissociation holds regardless.

## Reproduce

```
cd runs/seasar-multidomain-ownership/blindspot
sh measure_p0.sh                 # closure: commit-success per condition
# detection rates + judge: see task wqzsb82ib output / blindspot-detection workflow
```
Baseline = staging study Arm B (`../staging_results.jsonl`). Built artifacts under
`blindspot/detect-self/` and `blindspot/revise/`.
