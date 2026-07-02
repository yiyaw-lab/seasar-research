# Round 13: gate-authoring is apparatus-dependent too, and a convention-specific counter-cue closes the sticky seam

Round 12 left two registered predictions to grade. Round 13 grades them. One is
**confirmed cleanly**; the other is **partially wrong and more useful for it** — the
"elicit-then-gate" equalizer is real, but authoring the gate inherits the very
apparatus-dependence we found in building it.

Method note carried from round 12: a gate "has teeth" if it PASSES a correct
(golden) module and FAILS a silently-broken one. Because some models discriminate
golden from broken but *invert* the pass label (GPT-5 does this), the polarity-robust
score is **discrimination** — does the gate return different verdicts on golden vs
broken — not the raw pass boolean. Every apparatus below is self-tested with a
two-sided negative control before any gate is scored: a reference gate that must
score teeth, and a deliberately toothless gate that must not.

## Prediction #2 — CONFIRMED: a convention-specific counter-cue beats the generic cue

Caching is the sticky seam: a strong builder holds at **14/30 (47%)** even under the
generic integration cue, because it reflexively imports a "cache for performance"
convention and the generic cue doesn't reliably make it self-derive the counter-
requirement. A cue that *states* the specific unstated requirement — "reflect the
current store on every read; never serve a stale cached value" — closes it:

| caching build (Opus 4.8, agentic) | correct |
|---|---|
| generic integration cue | 14/30 (47%) |
| convention-specific counter-cue | **47/48 (98%)** |

Non-overlapping. The mechanism is the point: the counter-cue works by converting an
*attention* problem into a *stated-requirement* problem. That implies the lever is
attention-**routing** — detect the sticky affordance, inject the matched counter-cue —
not a blanket "try harder" prompt, which underperforms precisely where the reflex is
strongest. Data: `data/round13_counter_cue.jsonl`.

## Prediction #1 — REFINED: elicit-then-gate works, but gate-authoring is apparatus-dependent

The prediction was that compiling a builder's own enumeration of failure modes into an
executable end-to-end gate would close what the build-time cue cannot. The gate does
close it — **but only when the gate is authored in the same agentic posture that made
building work.** Ask a model to "write a test that catches this silent failure" in a
single shot and you mostly get a toothless test.

**Single-shot gate-authoring on the sticky caching seam** (raw API, one completion):

| model (single-shot) | naive prompt | integration-directed prompt |
|---|---|---|
| Opus 4.8 | 1/10 teeth | 3/10 teeth |
| GPT-5 | 0/10 teeth (10/10 *discriminate*, inverted label) | 0/9 (8/9 discriminate) |
| Haiku 4.5 | 0/10 | 0/10 |
| Fable 5 | — | 0/16 |

Even directed, Opus authors a caching gate with teeth only 3/10 of the time: the gate
checks the *stated* spec but misses the *unstated* runtime-flip (a stale cache returns
the old value after the store changes), so it passes both golden and broken. Authoring
a test is itself a production task, and it inherits the same attention gap as building.

**Agentic gate-authoring** (each gate authored by a separate Opus agent that reads the
spec with a tool, reasons, and writes the test over multiple steps) — the same posture
as the agentic build harness:

| seam | discrimination (polarity-robust) | Wilson 95% | compiles |
|---|---|---|---|
| caching | **60/60** | (0.94, 1.0) | 60/60 |
| pagination | 29/30 | (0.83, 0.99) | 30/30 |
| retry | **30/30** | (0.89, 1.0) | 30/30 |
| debounce | 24/30 | (0.63, 0.91) | 30/30 |
| **pooled (4 seams)** | **143/150 (95%)** | (0.91, 0.98) | 150/150 |

Same model, opposite apparatus, opposite result: single-shot Opus authors a caching
gate with teeth ~1–3/10; agentic Opus authors one **60/60**, and the effect generalizes
across all four seams (pooled 143/150 = 95% discrimination; the three new seams alone
pool to 83/90 = 92%). Gate-authoring is
apparatus-dependent exactly as building is.

Two honest nuances:

- **Retry is a dual hazard.** All 30 retry gates discriminate, but they split 13/17 on
  *which* correct behavior they encode: 13 treat "retry the transient failure" as
  correct (the spec's stated intent), 17 treat "charge at most once" as correct (the
  assembly hazard — a blind retry double-charges with no idempotency key). Both are
  legitimate; the agents reliably probe *a* retry integration hazard, and the split
  reflects a genuine tension in the seam's own spec, not gate failure. This is why we
  score discrimination, not a fixed polarity.
- **Debounce is the weakest seam** (24/30): 6 gates await the debounce timer before
  reading the count, so they never observe the synchronous under-count a real caller
  would hit. Still far above any single-shot floor.

## What this means for the mechanism

Elicit-then-gate is not a free equalizer that rescues weak models with a one-shot "write
a test" call — that yields toothless gates. It is the durable mechanism **inside the
agentic posture**: a fleet that authors its gates the way it builds (agentic, spec-read,
integration-directed) gets gates with teeth across seams; the same request issued
single-shot does not. For Seasar, the compiler-authored gate must be authored
agentically and with the integration cue baked in — not delegated to a single model
call. Prevention (the cheap builder-side cue) and the backstop (the agentic-authored
gate) share the same apparatus-dependence; that is the consistent thread from round 12.

## Data
`data/round13_counter_cue.jsonl` (P#2), `data/round13_gate_teeth_singleshot.jsonl`
(single-shot naive+directed+fable), `data/round13_gate_teeth_agentic.jsonl` (agentic,
all four seams). Predictions #3 (integration-targeted self-critique) and the agentic
non-Claude harness remain open.
