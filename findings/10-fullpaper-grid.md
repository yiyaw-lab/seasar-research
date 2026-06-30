# Round 10 (full-paper grid): mechanism ablation + capability floor

420-build grid: 4 breaking seams (pagination, caching, retry, debounce) x Opus/
Sonnet/Haiku x {baseline, generic-effort, integration-directed} x n=15. Wilson 95% CI.

## Mechanism ablation (Opus, pooled n=60/condition)
| condition | correct | CI |
|---|---|---|
| baseline | 14/60 (23%) | 14-35% |
| generic-effort ("review for bugs") | 27/60 (45%) | 33-58% |
| integration-directed | 50/60 (83%) | 72-91% |

Integration vs generic CIs do NOT overlap. Generic reflection helps (23->45%);
integration-directed attention is the dominant lever (45->83%). Not just compute.

Per-seam (Opus, n=15): pagination 8->13->15, caching 3->4->5, retry 1->4->15,
debounce 2->6->15. Integration fully closes 3/4; caching is the sticky exception.

## Capability floor (pooled n=60, baseline -> integration)
Opus 23%->83% (+60) | Sonnet 15%->27% (+12) | Haiku 0%->15% (+15).
The prompt works only if the model can act on the direction; below the frontier it
largely fails -> that is where an executable gate, not a prompt, is required.

## Detection (pagination+caching, n=12, 2 judges, 98% agreement; manual audit 48/48)
self 24/24, external 23/24 -> at ceiling, no asymmetry. Agents KNOW the requirement
(name it ~100% when asked) but break unprompted: an attention-allocation gap.

## Limitations
5 breaking seams (5 of ~10 candidates broke); single-module isolation; n=15; Haiku =
weakest reachable model. retry/debounce DETECTION batch hit an API budget limit, unrun.
Data: data/round10_grid.jsonl, data/round9_detection.json.
