# Directed attention, not coordination: a prompt that points an LLM coder at integration removes most silent integration failures — bounded by a steep capability floor

*Working paper (v4). Builders: Claude Opus 4.8 / Sonnet 4.6 / Haiku 4.5. Judges:
Opus 4.8 + Sonnet 4.6. Author: [you].*

## Abstract

When LLM agents build modules of one system independently, a class of failures
slips past type checks and review: the assembly compiles but is silently wrong
end-to-end, because an agent applied a reflexive convention that violates a
*positive* requirement no spec stated. The standard prescription is richer
inter-agent contracts. Across ten controlled rounds (~1,700 agent-built modules) we
find contracts mostly redundant, and the failure better explained as an
**attention-allocation** problem. In a pre-registered ablation over four breaking
seams (n=30/cell, Wilson CIs), a strong builder is correct **21%** at baseline,
**44%** under a generic "review for bugs" nudge, and **82%** under an
integration-directed nudge ("consider how this could silently fail when assembled")
— integration vs generic CIs do not overlap (74–88% vs 36–53%). So generic
reflection helps, but **directing attention at integration specifically is the
dominant lever**, not more compute. The effect has a **steep capability floor**:
the same prompt lifts Opus 21→82% but only Sonnet 9→27% and Haiku 0→9%. It is
**convention-dependent**: it fully closes three seams (≥93% each) but a "cache for
performance" habit resists every prompt (47% even primed). It **replicates in a
full multi-agent pipeline**: with three modules each built by a separate agent, the
break holds at baseline (0/15) and the integration nudge closes it (15/15) — not a
single-module-isolation artifact. And **detection is not the bottleneck**: asked to
enumerate failure modes, agents name the exact requirement at ceiling across all
four seams (self 48/48, external 46/47; two judges, ~99% agreement; manual audit
48/48). The lever is directing builder attention, backed by executable gates where
the prompt fails: weaker models, sticky conventions, and unprompted runs.

## 1. Setup

Small hermetic systems: one or more agents implement a module against shared types;
the assembly is exercised by a runtime probe checking an end-to-end property. Every
probe is validated before any agent runs (a golden reference passes, a broken one
fails). Under-specified seams follow an *affordance pattern*: the spec exposes a
config knob (`pageSize`, `cacheTtlMs`, `maxRetries`, `debounceMs`, git's `autoAdd`)
that invites a reflexive best-practice which violates an unstated-but-obvious
requirement. Reliable-breaking seams are hard to construct: of ~13 candidates, five
break (staging, pagination, caching, retry, debounce); the rest do not (agents
default correctly) — itself a finding about how often the failure fires.

## 2. What does not matter (rounds 1–7)

Behavioral/ownership contracts looked decisive in a pilot but the win was an
apparatus artifact; de-biased and replicated across domains, types-only agents
*derive* the correct design whenever the stated requirement determines it. A
"competence trap" (stronger models break more by importing a real convention)
failed to replicate once the positive requirement was stated. Through-line: silent
integration failures come from *unstated positive requirements*.

## 3. The mechanism ablation (central result, n=30)

Four breaking seams (pagination, caching, retry, debounce), Opus, n=30/cell, three
prompt conditions. Correct end-to-end, pooled (95% Wilson CI):

| condition | correct | CI |
|---|---|---|
| baseline | 25/120 (21%) | 15–29% |
| generic-effort | 53/120 (44%) | 36–53% |
| **integration-directed** | **88/107 (82%)** | **74–88%** |

Integration vs generic CIs are cleanly non-overlapping. Per-seam integration:
pagination 17/17, retry 28/30, debounce 29/30, caching 14/30 (the sticky exception).
The deficit is *where attention is allocated*, not how much reasoning is spent.

## 4. A steep capability floor

Same four seams, baseline → integration, pooled (n≈120/cell):

| model | baseline | integration | lift |
|---|---|---|---|
| Opus 4.8 | 21% | **82%** | +61 |
| Sonnet 4.6 | 9% | 27% | +18 |
| Haiku 4.5 | 0% | 9% | +9 |

Directing attention only works if the model is capable enough to *act* on the
direction. Below the frontier the prompt largely fails — that is where an external
gate, not a prompt, is required.

## 5. Full-pipeline replication (not an isolation artifact)

A 3-module pipeline (repo → service → report), each module built by a separate
agent in parallel, the pagination seam embedded. Opus, n=15/arm. End-to-end correct:
**baseline 0/15 (0–20%), integration 15/15 (80–100%)** — non-overlapping CIs. Both
the break and the closure survive assembly of an independently-built multi-agent
system; the single-module isolation used elsewhere is not driving the effect.

## 6. Detection is not the bottleneck

Across all four breaking seams (n=12/cell/mode, two judges, ~99% agreement; manual
keyword-and-read audit: target present in 48/48 enumerations): when *asked* to
enumerate failure modes, agents name the requirement at ceiling — **self 48/48,
external 46/47**. So at baseline they break while knowing the requirement; the gap
is attention-allocation, not detection-blindness, and there is no self-vs-external
asymmetry (an n=5 earlier result that did not replicate at n=24/mode).

## 7. Convention-dependence

Integration-directed attention fully closes pagination, retry, and debounce (≥93%
on Opus) but barely moves caching (47%). The "cache for performance, forget
invalidation" habit resists every prompt. Some conventions are sticky enough that
attention alone never suffices.

## 8. Discussion

The agentic-systems literature debates whether self-reflection helps. Our data give
a sharper account: the deficit is not detection (near-ceiling when asked) nor raw
reasoning (generic effort helps only modestly) but *attention allocation* to
integration, which a targeted prompt fixes — bounded by a steep capability floor and
by convention stickiness. For tooling: prevention is a builder-side,
integration-directed prompt; the durable external artifact is an **executable gate**
that fires whether or not attention was directed — the backstop for weaker models,
sticky conventions, and runs where no one prompts. Inter-agent contracts and
"fresh-eyes" review are not the lever.

## 9. Limitations

Five breaking seams; single-module isolation for the ablation grid (full-pipeline
replication on one seam); n=30/cell (ablation), n=15/arm (pipeline), n=12
(detection). Model ladder is Opus/Sonnet/Haiku — Claude Fable 5 was unavailable, and
an older/non-Claude floor is untested. Judges are LLMs, corroborated by a manual
audit (48/48) and ~99% inter-judge agreement, but not human-validated at scale.
Effects are large and CI-separated where claimed; the capability floor and
convention-stickiness are the most novel and warrant more models and seams.

## 10. What would extend it

More reliably-breaking seams (harder to construct than expected); full-pipeline
replication on the other four seams; a human-validated judge subset; an older/OSS
model to locate the floor; and a decision-theoretic estimate of when a gate beats a
prompt (model strength × convention stickiness × how often the seam is hit).

## 11. Conclusion

The dominant silent-integration failure we could construct is an
attention-allocation failure: agents know the requirement but do not deploy it
unprompted. An integration-directed prompt removes most of it on a strong model and
beats a generic effort nudge with separated CIs; it survives assembly into a full
multi-agent pipeline; but it degrades sharply with model capability and fails on
sticky conventions. Direct builder attention; back it with executable gates where
the prompt cannot reach.

## Artifacts

Method, raw per-trial data, self-tested harnesses: github.com/yiyaw-lab/seasar-research.
