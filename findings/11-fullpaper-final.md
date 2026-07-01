# Round 11 (full-paper finish): n=30 ablation, full-pipeline replication, detection generality

Definitive numbers (supersedes the n=15 figures in 10-fullpaper-grid.md).

## Mechanism ablation (Opus, pooled, n=30/cell, 95% Wilson CI)
| condition | correct | CI |
|---|---|---|
| baseline | 25/120 (21%) | 15-29% |
| generic-effort ("review for bugs") | 53/120 (44%) | 36-53% |
| integration-directed | 88/107 (82%) | 74-88% |

Integration vs generic CIs non-overlapping -> directing attention at integration is
the dominant lever, not generic effort/compute. Per-seam integration: pagination
17/17, retry 28/30, debounce 29/30, caching 14/30 (sticky).

## Capability floor (pooled n~120, baseline -> integration)
Opus 21->82 (+61) | Sonnet 9->27 (+18) | Haiku 0->9 (+9). Steep; the prompt needs a
capable-enough model to act on it. (Claude Fable 5 was unavailable -> no 4th point.)

## Full-pipeline replication (NOT a single-module artifact)
3-module pipeline repo->service->report, each built by a separate agent, pagination
seam embedded. Opus, n=15/arm: **baseline 0/15 (0-20%), integration 15/15 (80-100%)**
-- non-overlapping CIs. Break AND closure both survive multi-agent assembly.

## Detection generality (all 4 seams, n=12/cell, 2 judges ~99% agree, manual audit 48/48)
self 48/48, external 46/47 -> ceiling for both, no asymmetry. Agents NAME the
requirement ~100% when asked but break unprompted: attention-allocation, not
detection-blindness or knowledge gap.

## Data
data/round10_grid_n30.jsonl, data/round11_fullbuild.jsonl,
data/round11_detection_retry_debounce.json, data/round9_detection.json.

## Named limitations (deferred)
- Fable 5 unavailable; older/OSS model floor untested.
- Full-pipeline replication done on 1 of 5 seams.
- Judge is LLM (agent manual audit 48/48; human-validation is future work).
