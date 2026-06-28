# Seasar Research

Empirical studies on autonomous multi-agent software construction, from
[Seasar](https://yiya.dev) — an agent lab. Public companion to the write-ups on
[yiya.dev](https://yiya.dev).

## Current study: from coordination contracts to integration attention

We asked whether the standard prescription for parallel multi-agent coding —
richer inter-agent *contracts* — actually prevents the failures it targets. Across
**eight controlled rounds (~330 agent-built modules)** the answer kept getting
narrower, ending somewhere more useful than where it started.

**Headline finding.** The dominant silent-integration failure we could construct is
an **attention artifact, not a coordination problem**: a one-line prompt asking the
builder to "consider how this could silently fail when assembled" takes end-to-end
success from **~20% to ~96%** across Opus, Sonnet, and Haiku. And **prevention
dissociates from detection** — the prompt fixes the code far more than it produces
awareness (builders write correct code while their own critiques never name the
requirement they satisfied), while *explicit* detection of the gap is ~2× better
from a fresh external reviewer than from self-review.

This reframes "multi-agent coordination tooling": **prevention is cheap and belongs
in the builder loop; the external value is audit + executable gates**, aimed
exactly at the inside view's blind spot.

### The arc (each round tempered the last)
1. **Pilot** — contracts looked decisive (prevented a leak + a silent no-commit). [findings/01-pilot.md](findings/01-pilot.md)
2. **De-biased, 2 models** — the redaction win was an apparatus artifact; removed, it vanished. The staging win held. [findings/02-debiased-study.md](findings/02-debiased-study.md)
3–6. **Multi-domain** — across new domains, types-only agents converged on correct designs with no contract. [findings/06-multidomain.md](findings/06-multidomain.md)
7. **Competence trap** — one seam broke *more* on stronger models (imported a real convention), but it failed to replicate: it's a symptom of an *unstated positive requirement*, not a capability law. [findings/07-competence-trap.md](findings/07-competence-trap.md)
8. **Blind-spot study** — the attention result, the self-vs-external detection asymmetry, and the prevention/detection dissociation. [findings/08-blindspot.md](findings/08-blindspot.md)

### Read
- Narrative: [drafts/POST.md](drafts/POST.md)
- Working paper / extended abstract: [drafts/PREPRINT.md](drafts/PREPRINT.md)
- Where this is headed + venues: [drafts/SUBMISSION_NOTES.md](drafts/SUBMISSION_NOTES.md)
- Raw per-trial data: [data/](data/)

## Status & honesty

This is a **lab note**, not a finished paper. The closure result (the prompt
effect) is large and consistent; the detection asymmetry is directional and noisy
(n=5–6/cell, one headline seam family, three models from one family, a single LLM
judge). The **round-9 hardening** below is what would make it citable.

### Round 9 (next): the hardening experiment
Pre-registered so the result can't be moved after the fact:
- **≥6 under-specified seams**, each a different wrongly-imported "best practice"
  (not just the one git convention) — tests whether the dissociation generalizes.
- **n ≥ 20/cell with confidence intervals** — the detection asymmetry is currently
  too noisy to assert.
- **Human-validated judge + a second independent judge** — removes the
  single-LLM-grader threat.
- **A priming-vs-revision ablation** — separates "building carefully because a
  critique is coming" from "actually revising."
- **A weaker/older model** — locates the capability floor below which a one-line
  attention prompt stops being enough (the product-relevant number).

Prediction: closure holds everywhere; the dissociation holds across most seams;
there is a capability floor. If the dissociation evaporates when widened, that will
be reported here too.

## Reproduce

The apparatus (TypeScript modules + probe harnesses, each self-tested with a
golden/broken reference before any agent ran) and the per-trial built modules are
packaged with the round-9 release. Aggregated data and the analysis scripts are in
[data/](data/). Builders: Claude Opus 4.8 / Sonnet 4.6 / Haiku 4.5; judge: Sonnet
4.6.

## License
[MIT](LICENSE). Please cite the preprint if you build on this.
