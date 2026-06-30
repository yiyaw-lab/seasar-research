# Directed attention, not coordination: a prompt that points an LLM coder at integration removes most silent integration failures — bounded by a steep capability floor

*Working paper (v3, full-paper grid). Builders: Claude Opus 4.8 / Sonnet 4.6 /
Haiku 4.5. Judges: Opus 4.8 + Sonnet 4.6. Author: [you].*

## Abstract

When LLM agents build modules of one system independently, a class of failures
slips past type checks and review: the assembly compiles but is silently wrong
end-to-end, because an agent applied a reflexive convention that violates a
*positive* requirement no spec stated. The standard prescription is richer
inter-agent contracts. Across ten controlled rounds (~1000 agent-built modules) we
find contracts mostly redundant, and the failure better explained as an
**attention-allocation** problem. In a pre-registered ablation over four breaking
seams (n=15/cell, Wilson CIs), a strong builder is correct **23%** at baseline,
**45%** under a generic "review for bugs" nudge, and **83%** under an
integration-directed nudge ("consider how this could silently fail when assembled")
— integration vs generic CIs do not overlap (72–91% vs 33–58%). So generic
reflection helps, but **directing attention at integration specifically is the
dominant lever**, not more compute. The effect has a **steep capability floor**:
the same prompt lifts Opus 23→83% but only Sonnet 15→27% and Haiku 0→15%. And it is
**convention-dependent**: it fully closes three seams (15/15 each) but a "cache for
performance" habit resists every prompt (5/15 even primed). Detection is not the
bottleneck — asked to enumerate failure modes, agents name the exact requirement at
ceiling (self 24/24, external 23/24; two judges, 98% agreement; manual audit 48/48).
The lever is directing builder attention, backed by executable gates where the
prompt fails: weaker models, sticky conventions, and unprompted runs.

## 1. Setup

Small hermetic systems: one agent implements a module against shared types; the
assembly is exercised by a runtime probe checking an end-to-end property. Every
probe is validated before any agent runs (a golden reference passes, a broken one
fails). Under-specified seams follow an *affordance pattern*: the spec exposes a
config knob (`pageSize`, `cacheTtlMs`, `maxRetries`, `debounceMs`, git's `autoAdd`)
that invites a reflexive best-practice which violates an unstated-but-obvious
requirement. Reliable-breaking seams are hard to construct: of ~10 candidates, five
break (staging, pagination, caching, retry, debounce); five do not (agents default
correctly) — itself a finding about how often the failure actually fires.

## 2. What does not matter (rounds 1–7)

Behavioral/ownership contracts looked decisive in a pilot but the win was an
apparatus artifact; de-biased and replicated across domains, types-only agents
*derive* the correct design whenever the stated requirement determines it. A
"competence trap" (stronger models break more by importing a real convention)
failed to replicate once the positive requirement was stated. Through-line: silent
integration failures come from *unstated positive requirements*.

## 3. The mechanism ablation (the central result)

Four breaking seams (pagination, caching, retry, debounce), Opus, n=15/cell, three
prompt conditions. Correct end-to-end (95% Wilson CI):

| seam | baseline | generic-effort | integration-directed |
|---|---|---|---|
| pagination | 8/15 | 13/15 | 15/15 |
| caching | 3/15 | 4/15 | 5/15 |
| retry | 1/15 | 4/15 | 15/15 |
| debounce | 2/15 | 6/15 | 15/15 |
| **pooled** | **14/60 (14–35%)** | **27/60 (33–58%)** | **50/60 (72–91%)** |

Generic reflection beats baseline (23→45%), but integration-directed attention beats
generic with non-overlapping CIs (45→83%). The deficit is *where attention is
allocated*, not how much reasoning is spent.

## 4. A steep capability floor

Same four seams, baseline → integration-directed, pooled (n=60/cell):

| model | baseline | integration | lift |
|---|---|---|---|
| Opus 4.8 | 23% | **83%** | +60 |
| Sonnet 4.6 | 15% | 27% | +12 |
| Haiku 4.5 | 0% | 15% | +15 |

Directing attention only works if the model is capable enough to *act* on the
direction. Below the frontier the prompt largely fails — the product-relevant
number: that is exactly where an external gate, not a prompt, is required.

## 5. Detection is not the bottleneck (and the dissociation)

On pagination and caching (n=12/cell, two judges, 98% agreement; manual
keyword-and-read audit: target present in 48/48 enumerations): when *asked* to
enumerate failure modes, agents name the requirement at ceiling — **self 24/24,
external 23/24**. So at baseline they break while knowing the requirement; the gap is
attention-allocation, not detection-blindness, and not a self-vs-external asymmetry
(an n=5 earlier result that did not replicate).

## 6. Convention-dependence

Integration-directed attention fully closes pagination, retry, and debounce (15/15
each on Opus) but barely moves caching (5/15). The "cache for performance, forget
invalidation" habit resists every prompt. Some conventions are sticky enough that
attention alone never suffices.

## 7. Discussion

The agentic-systems literature debates whether self-reflection helps. Our data give
a sharper account: the deficit is not detection (near-ceiling when asked) nor raw
reasoning (generic effort helps only modestly) but *attention allocation* to
integration, which a targeted prompt fixes — bounded by a steep capability floor and
by convention stickiness. For tooling: prevention is a builder-side, integration-
directed prompt; the durable external artifact is an **executable gate** that fires
whether or not attention was directed — the backstop for weaker models, sticky
conventions, and runs where no one prompts. Inter-agent contracts and "fresh-eyes"
review are not the lever.

## 8. Limitations

Five breaking seams; single-module isolation for the grid; n=15/cell (pooled n=60);
Opus/Sonnet/Haiku (Haiku = weakest model the harness reaches; an older/OSS floor
untested). Detection at scale ran on two seams (the retry/debounce detection batch
hit an API budget limit and is unrun). Judges are LLMs, corroborated by a manual
audit (48/48) and 98% inter-judge agreement, but not human-validated at scale.
Effects are large and CI-separated where claimed; the capability floor and
convention-stickiness are the most novel and should be replicated on more models
and seams.

## 9. What would finish the paper

Detection across all five seams; n≥30 with a pre-registered analysis plan; a
human-validated judge subset; full-build (not isolation) replication; an older/OSS
model to locate the floor precisely; and a separate decision-theoretic estimate of
when a gate beats a prompt (a function of model strength × convention stickiness ×
how often the seam is hit).

## 10. Conclusion

The dominant silent-integration failure we could construct is an attention-allocation
failure: agents know the requirement but do not deploy it unprompted. An
integration-directed prompt removes most of it on a strong model and beats a generic
effort nudge with separated CIs — but degrades sharply with model capability and
fails on sticky conventions. Direct builder attention; back it with executable gates
where the prompt cannot reach.

## Artifacts

Method, raw per-trial data, self-tested harnesses: github.com/yiyaw-lab/seasar-research.
