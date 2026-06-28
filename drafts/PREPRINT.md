# Attention, not coordination: silent integration failures in parallel multi-agent code generation are removed by a prompt, and prevention dissociates from detection

*Working paper / extended abstract. n is small; see Limitations. Author: [you].
Models: Claude Opus 4.8, Sonnet 4.6, Haiku 4.5 (builders); Sonnet 4.6 (judge).*

## Abstract

When N LLM agents build one codebase in parallel, a class of failures slips past
type checks and code review: the system compiles but is silently wrong end-to-end,
because a *positive* cross-module requirement was never stated and two agents
filled the gap incompatibly. A natural prescription is richer inter-agent
*contracts*. Across eight controlled rounds (~330 agent-built modules) we find
contracts are mostly redundant: when the positive requirement is stated, builders
of every capability comply; when it is unstated, behavior is driven by model
defaults. We then isolate the one reliably-failing seam and show three things.
(1) The failure is an **attention artifact, not a capability gap**: a single
prompt asking the builder to "consider how this could silently fail when
assembled" raises end-to-end success from ~20% to ~96% across Opus, Sonnet, and
Haiku. (2) **Explicit self-detection of the missing requirement is poor** (27%) and
roughly half that of a fresh external reviewer (53%) — a measured self-blind-spot
in agentic self-review. (3) **Prevention dissociates from detection**: the prompt
fixes the artifact far more than it produces awareness; builders write correct code
while their own critiques never name the requirement they satisfied. We argue this
reframes "multi-agent coordination tooling" from contract enforcement toward
(a) cheap builder-side integration priming and (b) an external audit/gate-generation
role that targets exactly the inside view's blind spot.

## 1. Introduction

Parallel multi-agent code generation is moving from research to product (isolated
worktrees, fleet orchestration). The headline risk is not merge conflicts (solved
by isolation) but *silent semantic divergence*: independently-built modules that
compile together yet produce wrong end-to-end behavior. The dominant proposed
mitigation is structured inter-agent coordination — interface/behavioral contracts.
We ask whether such contracts are load-bearing, and if not, what is.

## 2. Setup

We build small, hermetic systems (a watch/approve/commit CLI; a money-transfer
service; a queue worker; a report counter; an event logger) where two or more
agents each implement one module against a shared, typed interface, then the
assembled system is exercised by a fixed runtime probe that checks an end-to-end
property the spec requires. Each probe is validated before any agent runs: a
"golden" reference passes and a "broken" reference fails. The independent variable
is what each builder receives (types only; types + behavioral contract; types +
various prompts). All builds are isolated; the only differences are the controlled
inputs and the builder model.

## 3. What does not work (rounds 1–7, summarized)

- A behavioral/ownership contract appeared to prevent two bugs (round 1), but the
  redaction win was an apparatus artifact (the contract enumerated secret formats);
  removed, the effect vanished (round 2).
- Across four additional seams (transactions, charge idempotency, queue acks, load
  dedup), types-only agents converged on correct designs with no contract; they
  *derive* the owner whenever the stated requirement determines it (round 3–4).
- One seam (who stages files before commit) did break under types-only, and broke
  *more* on stronger models, which import a real-world convention (git's
  "commit only what's staged") that fits the general case but not this pipeline
  (round 5–6, a "competence trap"). It failed to replicate on two further seams
  (round 7): with the positive requirement stated crisply, even Opus complied. The
  trap is a symptom of an *unstated positive requirement*, not a capability law.

The robust through-line: silent integration failures are caused by unstated
positive requirements; stating them (ideally as a test) immunizes builds at every
capability tier.

## 4. The blind-spot study (round 8)

On the one confirmed blind-spot seam (staging), committer-only isolation, we run
two measurements across the capability ladder (n=5–6/cell).

**Closure** (commit-success on the happy path):

| condition | Opus | Sonnet | Haiku |
|---|---|---|---|
| build only (baseline) | 1/5 | 1/5 | — |
| build + "enumerate integration failure modes" | 5/5 | 4/5 | 5/5 |
| build + self-critique + revise | 6/6 | 6/6 | — |

**Detection** (judge-scored: did the critique explicitly name the gap?):

| | Opus | Sonnet | Haiku | total |
|---|---|---|---|---|
| self | 2/5 | 1/5 | 1/5 | 4/15 (27%) |
| external | 4/5 | 1/5 | 3/5 | 8/15 (53%) |

**4.1 The failure is attention, not capability.** Build-only commit-success is
~20%; a one-line integration-priming prompt raises it to ~96% on all three models,
including the smallest. No capability increase or contract is involved.

**4.2 Self-review has a blind spot.** Reflecting on its own build, a builder names
the actual gap 27% of the time; a fresh external reviewer 53% — ~2× (directional;
n=5; external>self for Opus/Haiku, tied for Sonnet).

**4.3 Prevention dissociates from detection.** The prompt closes the build (~96%)
far more than it yields explicit awareness (27% self). Builders stage correctly
while their enumerations name *other* risks; judge transcripts confirm correct code
co-occurring with critiques that never articulate the satisfied requirement. The
fix runs through implicit carefulness, not understanding — and the part that needs
understanding is where self-review is weakest.

## 5. Discussion

The "self-reflection vs external verification" debate for agentic systems usually
treats reflection as either helping or not. Our data refine it: a generic attention
nudge *prevents* the failure without the agent being able to *explain* it, while
*detection* (needed for audit, documentation, gate generation) exhibits a genuine
self-blind-spot that an external view mitigates. For tooling, this separates two
jobs: prevention is cheap and belongs in the builder loop (prime integration
attention by default); detection/verification is where an external process earns
its keep, by enumerating requirements the inside view misses and emitting them as
executable gates that hold regardless of whether anyone reasoned about them.

## 6. Limitations

Single under-specified seam family in the headline study; committer-only isolation;
n=5–6/cell; three models from one family; a single LLM judge (target crisply
defined, transcripts spot-checked, not human-validated at scale). The closure
dose-response is large and consistent; the detection asymmetry is directional and
noisy. This is an extended abstract / lab note, not a full empirical paper.

## 7. What would make it a paper

More seams (≥8, varied imported conventions and requirement phrasings); larger n
(≥20/cell) with reported CIs and a pre-registered analysis; multiple judges plus a
human-validated subset; full-build (not isolation) replication; a weaker/older
model to probe where the attention nudge stops working; and an ablation isolating
"priming" from "self-revision."

## 8. Conclusion

In parallel multi-agent code generation, the dominant silent-integration failure we
could construct is an attention artifact removed by a prompt, not a coordination
problem solved by contracts; and the fix dissociates from the agent's ability to
detect or explain the gap, which an external reviewer catches ~2× better. Prime
builders for integration; put the external effort into audit and executable gates.

## Artifacts

Method, raw per-trial data, validated probe harnesses, and all built modules are
released in the project repository.
