# Round 14 (DRAFT — framing pending): the attention effect decomposes — execution-availability, scaffold-reasoning-depth, and a counter-cue that generalizes everywhere

> **DRAFT.** Data final (one cell pending: 14c persona isolation); public framing not yet
> settled. This round *refines* round 12's headline: the closures are real but
> apparatus-bounded, and "directed attention" turns out to be at least three distinct
> mechanisms. Nothing here retracts the round-12 data; it bounds what the numbers mean.

## Question
Round 12 left cross-family transfer confounded: Opus lifts 21→82% in our agentic Claude
Code harness but is flat single-shot, and non-Claude models had only been tested
single-shot. Round 14 builds the missing apparatus — a raw-API **agentic tool-use loop**
(read the spec via tool → reason → submit via tool) that any model family can run — plus
an **execute-and-iterate** variant (a `run_ts` tool that compiles and runs agent-written
code; the grading probe is never exposed). Opus runs through both as the positive control:
if the same model that closes in our workflow doesn't close in the new apparatus, the
apparatus — not the family — is the variable. n=12/cell, same seams, specs, byte-identical
nudge, same self-tested probes as round 12.

## Result 1 — the minimal agentic loop reproduces only pagination (cross-family)

| 14a minimal loop, baseline → integration | opus (control) | gpt5 | grok |
|---|---|---|---|
| pagination | 1/12 → **12/12** | 9/12 → **12/12** | 4/12 → 8/12 |
| caching / retry / debounce (pooled) | 0/36 → 1/36 | 0/36 → 0/36 | 0/36 → 1/36 |

On the seam where the control validates, **all three families lift** — the attention
effect transfers cross-family on pagination, with family-specific baselines (which
convention a model reflexively imports differs: Opus falls into the page-size trap,
GPT-5 mostly doesn't). On the harder seams the Opus control itself is flat: the
workflow's retry 28/30 / debounce 29/30 do **not** reproduce in a bare tool loop.

## Result 2 — give the agent execution and the nudge stops mattering

| 14b + `run_ts`, pooled nudge lift | opus | gpt5 | grok |
|---|---|---|---|
| 14a (no execution) | +25pp | +6pp | +10pp |
| 14b (execute-and-iterate) | **+2pp** | **0pp** | +4pp |

The mechanism: `run_ts` raises the **baseline** (Opus pagination baseline 1/12 → 11/12 —
the agent runs its module, sees 50 ≠ 150, and fixes it unprompted). Much of what looked
like "directed attention" on pagination was **execution-availability**: the bug was
invisible without running the code. retry/debounce/caching stay ~0 for every model —
self-testing doesn't surface a hazard the agent never derives (all its self-written
tests encode the same wrong model of the seam).

## Result 3 — reconciliation: the workflow closures are real, and scaffold-bounded

Same probe (byte-identical), same model, same nudge. The difference is what the module
*says*: workflow-Opus modules reason explicitly about the at-most-once hazard ("a thrown
charge may already have landed; retrying double-charges") in **29/30** integration
trials (28/30 pass); raw-API-Opus mentions it in **2/12** (minimal) and **0/12**
(execute-iterate) — and writes a plain retry loop (0/12 pass). The full Claude Code
agent scaffold evokes a depth of hazard reasoning that a bare tool loop does not,
even with execution available. Round 12's numbers are bounded to that posture — which
is the ecologically real one for coding-agent fleets, but a benchmark using a bare
tool loop would never see the effect.

## Result 4 — Fable: the scaffold effect is capability-graded *within* the Claude family

Fable 5 cannot run the raw-API loop at all (`stop_reason: refusal` on any tool-result
continuation — five probe variants, two SDK versions; an apparatus incompatibility we
document rather than work around; the round-12 grid's fable rows are likewise invalid —
the arm ran while the model was unavailable and wrote no modules). Through the **full
workflow apparatus** (one agent per module, same protocol as the grid):

| fable, baseline → integration | | opus same apparatus |
|---|---|---|
| pagination | 11/12 → 12/12 (trap-resistant at baseline) | 14/30 → 17/17 |
| debounce | 1/12 → 5/12 | → 29/30 |
| retry | 0/12 → 0/12 | → 28/30 |
| caching | 0/12 → 0/12 | → 14/30 |
| **pooled** | **25% → ~34%** (+10pp; effort-check confirms, 16/48 vs 17/48) | **21% → 82%** (+61pp) |

Same scaffold, same nudge: Fable shows a modest lift, Opus a large one. The
scaffold-reasoning effect is **not a Claude-family trait — it is model-specific even
within the family**, consistent with round 10's capability floor (Sonnet +18, Haiku +9).

## Result 5 — the counter-cue generalizes across family AND apparatus

Round 13 confirmed P#2 on agentic-Opus caching (47% → 98%). Round 14 extends it to the
seam and families where the generic cue is most useless — retry, single-shot:

| retry, single-shot raw API | generic integration cue | at-most-once counter-cue |
|---|---|---|
| GPT-5 | 0/12 | **12/12** |
| Grok-4.3 | 0/12 | **11/12** |

Stating the unstated requirement closes the seam for every family, in the weakest
apparatus, with zero compile failures. Combined with rounds 12–13: the **generic**
attention nudge is fragile (it works for one model, in one posture); the **specific
counter-cue** is robust everywhere, because it converts an attention problem into a
stated-requirement problem — and every model follows stated requirements (rounds 1–7).

## What this means (draft synthesis, framing TBD)
- "Directed attention" was one label over ≥3 mechanisms: **execution-availability**
  (pagination), **scaffold-evoked hazard reasoning** (retry/debounce — Opus-only),
  **sticky conventions** (caching — nothing closes it but a counter-cue).
- The durable, portable prompt-side mechanism is **attention-routing to specific
  counter-cues** (detect the affordance, inject the matched stated requirement) — robust
  across model families and apparatuses, unlike generic nudges.
- Benchmarks and tooling built on bare tool loops will systematically miss
  scaffold-dependent effects — for better and worse.
- Pending: 14c persona-isolation (does a Claude-Code-style system prompt alone restore
  the at-most-once reasoning in the raw loop?).

## Data
`data/round14_agentic_minimal.jsonl` (288), `data/round14_agentic_iterate.jsonl` (288),
`data/round14_fable_workflow.jsonl` (144), `data/round14_countercue_retry.jsonl` (24).
Every scorer self-tested two-sided (golden passes / broken fails) before scoring.
