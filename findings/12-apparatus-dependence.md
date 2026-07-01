# Round 12: the effect is apparatus-dependent, not model-family-specific (a disconfirmation)

Round 12 set out to answer two loose ends for full-conference grade: (1) does the
integration-attention effect transfer across **model families** (GPT-5, Grok-4.3,
Sonnet 5), and (2) does it survive **full-pipeline** replication on the other three
breaking seams. The second replicated. The first disconfirmed a tempting claim and
replaced it with a more important one.

## The trap we almost fell into

Built with non-Claude models via raw API (single completion, spec inline), the
four-seam ablation showed **no integration effect** for any family:

| model (single-shot, raw API) | baseline | integration |
|---|---|---|
| GPT-5 | 11/48 (23%) | 12/48 (25%) |
| Grok-4.3 | 11/48 (23%) | 12/48 (25%) |
| Sonnet 5 | 15/48 (31%) | 12/48 (25%) |

The tempting headline: *the effect is Claude-specific; it doesn't transfer.* We did
not ship it, because a control killed it.

## The control that killed it

We re-ran the **same model that shows the effect** — Opus 4.8, which lifts 21→82%
in our agentic harness — through the **identical raw-API single-shot apparatus**:

| Opus 4.8 | baseline | integration |
|---|---|---|
| agentic harness (grid) | 21% | **82%** |
| single-shot raw API (control) | 24% (11/46) | 21% (10/48) |

Single-shot, Opus is flat — and does not even fall into the pagination trap at
baseline (11/12 correct vs 14/30 agentic). The nudge text is **byte-identical**
across apparatuses; failures are behavioral (modules compile), not compile errors.

So the difference between "shows the effect" and "doesn't" is **not the model
family — it is the apparatus**: agentic (read the spec with a tool, reason, write the
module over multiple steps) vs single-shot (spec inline, one completion). Every model
is flat single-shot; the same Opus is 82% agentic.

## What we actually learned

- **The directed-attention effect is apparatus-dependent.** It requires the agentic
  build posture and vanishes single-shot. That bounds the result to the agentic
  setting — which is the ecologically relevant one for a fleet of coding agents — and
  warns that a single-shot benchmark would miss the effect entirely.
- **Cross-family transfer is untested, not refuted.** Non-Claude models can't run in
  our Claude-only agentic harness, so the raw-API nulls are apparatus-confounded. No
  model-family claim is made. The clean test needs an **agentic non-Claude harness**
  (a raw-API tool-use loop) — the priority next build.

## Full-pipeline replication on all four seams (this DID replicate)

3-module pipelines, each module a separate agent, Opus, n=15/arm:

| pipeline | baseline | integration |
|---|---|---|
| pagination | 0/15 | 15/15 |
| debounce | 0/15 | 13/15 |
| retry | 0/15 | 4/15 |
| caching | 0/15 | 3/15 |
| **pooled** | **0/60** | **35/60 (58%)** |

The break holds at baseline everywhere; the nudge lifts every pipeline, strongly for
pagination/debounce and weakly for caching/retry. Notable: retry closes 93%
single-module but only 4/15 in the pipeline — a compounding effect where the
end-to-end property can fail in any of the three independently-built modules. The
effect survives assembly, but the harder the assembly the more a prompt alone leaves
on the table (motivating the external gate).

## Data
`data/round12_crossfamily.jsonl` (GPT-5/Grok/Sonnet-5 single-shot),
`data/round12_opus_singleshot_control.jsonl` (the decisive control),
`data/round12_pipelines.jsonl` (four-seam full-pipeline, 60/60).
