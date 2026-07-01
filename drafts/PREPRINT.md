# Directed attention, not coordination: a prompt that points an LLM coder at integration removes most silent integration failures — bounded by a steep capability floor

*Working paper (v5). Builders: Claude Opus 4.8 / Sonnet 4.6 / Haiku 4.5 (agentic
harness); raw-API apparatus control adds Opus 4.8 / Sonnet 5 / GPT-5 / Grok-4.3.
Judges: Opus 4.8 + Sonnet 4.6. Author: [you].*

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
performance" habit resists every prompt (47% even primed). It **replicates in full
multi-agent pipelines** across four seams — three modules each built by a separate
agent — where the break holds at baseline (0/60 pooled) and the integration nudge
closes it strongly for pagination and debounce (15/15, 13/15) and weakly for caching
and retry (3/15, 4/15): attenuated in the harder multi-module setting, not a
single-module-isolation artifact. Crucially the effect is **apparatus-dependent**:
an identical nudge delivered *single-shot* (one API call, spec inline) produces no
effect even for the model that shows it — Opus 4.8 is 24→21% single-shot vs 21→82%
agentic — so the effect requires the agentic build posture, and cross-family
transfer (untestable in a Claude-only agentic harness) remains open. And **detection
is not the bottleneck**: asked to
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

## 5. Full-pipeline replication across four seams (not an isolation artifact)

3-module pipelines (module → module → module), each module built by a separate
agent in parallel, one breaking seam embedded per pipeline. Opus, n=15/arm.
End-to-end correct, baseline → integration:

| pipeline | baseline | integration | CI (integration) |
|---|---|---|---|
| pagination | 0/15 | **15/15** | 80–100% |
| debounce | 0/15 | **13/15** | 62–96% |
| retry | 0/15 | 4/15 | 11–52% |
| caching | 0/15 | 3/15 | 7–45% |
| **pooled** | **0/60 (0–6%)** | **35/60 (46–69%)** | — |

The break holds at baseline in every pipeline (0/60) and the integration nudge lifts
every one — non-overlapping pooled CIs. Closure is **attenuated and seam-dependent**
in the multi-module setting: strong where the convention is one obvious slip
(pagination, debounce) and weak for caching (the sticky §7 convention, also weak
single-module) and retry — whose 93% single-module closure does *not* survive the
three-module assembly, a compounding effect where the property can fail in any of the
independently-built modules. So the effect survives assembly of an independently-built
multi-agent system, but the harder the assembly, the more a prompt alone leaves on the
table — motivating the external gate (§8).

### 5.1 The effect is apparatus-dependent — cross-family transfer is untested

Everything above was built in an *agentic* harness: each module is written by a coding
agent that reads the spec with a file tool, reasons, and writes the module with a file
tool over multiple steps. To separate the model from the harness, we re-ran the
four-seam ablation in a *single-shot* apparatus — the identical spec, types, and
integration nudge delivered inline in one API call, the model emitting the module in
one completion (n=12/cell). The nudge text is byte-identical to the agentic one.

Single-shot, the effect disappears — for **every** model, including the one that shows
it agentically:

| model (single-shot, raw API) | baseline | integration |
|---|---|---|
| Opus 4.8 | 24% (11/46) | 21% (10/48) |
| Sonnet 5 | 31% (15/48) | 25% (12/48) |
| GPT-5 | 23% (11/48) | 25% (12/48) |
| Grok-4.3 | 23% (11/48) | 25% (12/48) |

Opus 4.8 lifts 21→82% *agentically* but is flat 24→21% *single-shot* — and does not
even fall into the pagination trap at baseline (11/12 correct) that it falls into
agentically (14/30). Failures are behavioral (the modules compile), not compile
errors. Two consequences. **(1) The effect is apparatus-dependent**: it requires the
agentic build posture and vanishes when the same model emits the same module
single-shot under the same instruction. That bounds the result to the agentic setting
— which is the ecologically relevant one for a fleet of coding agents — and warns that
a single-shot benchmark would miss the effect entirely. **(2) Cross-family transfer is
untested**: non-Claude models cannot run in our agentic harness, so the raw-API
GPT-5/Grok/Sonnet-5 nulls are confounded by apparatus (the Opus control flattens
identically). We make **no** model-family claim. The clean test needs an agentic
non-Claude harness (a raw-API tool-use loop); it is the priority extension (§10).

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

Five breaking seams; the ablation grid is single-module, with full-pipeline
replication now on four seams (§5). n=30/cell (ablation), n=15/arm (pipeline),
n=12/cell (single-shot control and detection). The agentic capability ladder is
Opus 4.8 / Sonnet 4.6 / Haiku 4.5 (Fable 5 unavailable). The single-shot apparatus
control (§5.1) shows the effect is agentic-specific and leaves **cross-family
transfer untested** — the raw-API GPT-5/Grok/Sonnet-5 nulls are apparatus-confounded
(the same-model Opus control flattens identically), not a model-family result, so we
claim none. Judges are LLMs, corroborated by a manual audit (48/48) and ~99%
inter-judge agreement; a single-human-judge validation sheet (24 stratified
enumerations, pagination + caching) is prepared but not yet scored, so judge
validation is not human-confirmed at scale. Effects are large and CI-separated where
claimed; apparatus-dependence and cross-family transfer are now the central open
items.

## 10. What would extend it

An **agentic non-Claude harness** (a raw-API tool-use loop giving GPT-5/Grok/others
the same read-spec/write-module posture) to test cross-family transfer cleanly — the
top priority, since §5.1 leaves it the central open question and single-shot cannot
answer it. Scoring the prepared human-judge sheet. More reliably-breaking seams
(harder to construct than expected). An older/OSS model to locate the agentic
capability floor below Haiku. And a decision-theoretic estimate of when a gate beats a
prompt (model strength × convention stickiness × how often the seam is hit ×
apparatus).

## 11. Conclusion

The dominant silent-integration failure we could construct is an
attention-allocation failure: agents know the requirement but do not deploy it
unprompted. An integration-directed prompt removes most of it on a strong model and
beats a generic effort nudge with separated CIs; it survives assembly into full
multi-agent pipelines (attenuated, seam-dependent); but it degrades sharply with
model capability, fails on sticky conventions, and — importantly — is
apparatus-dependent: it lives in the agentic build posture and vanishes when the same
model is prompted single-shot, which leaves cross-family transfer an open question.
Direct builder attention inside the agentic loop; back it with executable gates where
the prompt cannot reach.

## Artifacts

Method, raw per-trial data, self-tested harnesses: github.com/yiyaw-lab/seasar-research.
