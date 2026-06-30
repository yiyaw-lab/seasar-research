# Seasar Research

Empirical studies on autonomous multi-agent software construction, from
[Seasar](https://yiya.dev) — an agent lab. Public companion to the write-ups on
[yiya.dev](https://yiya.dev).

## Current study: from coordination contracts to integration attention

We asked whether the standard prescription for parallel multi-agent coding —
richer inter-agent *contracts* — actually prevents the failures it targets. Across
**nine controlled rounds (~600 agent-built modules)** the answer kept getting
narrower, ending somewhere more useful, and survived a deliberate attempt to break
it (round 9).

**Headline finding** (420-build pre-registered ablation, Wilson CIs). The dominant
silent-integration failure we could construct is an **attention-allocation failure,
not a coordination problem**. Agents *know* the requirement — asked to enumerate
failure modes they name it at ceiling (self 24/24, external 23/24; two judges, 98%
agreement; manual audit 48/48) — but break unprompted. The fix is **directing
attention**, and it is *specifically* integration-direction, not generic effort:
pooled over four breaking seams a strong builder is correct **23%** at baseline,
**45%** under a generic "review for bugs" nudge, and **83%** under "consider how this
could silently fail when assembled" (integration vs generic CIs do not overlap). Two
hard bounds: a **steep capability floor** (the same prompt lifts Opus 23→83% but only
Sonnet 15→27%, Haiku 0→15%) and **sticky conventions** (a "cache for performance"
habit resists every prompt: 5/15 even primed).

This reframes "multi-agent coordination tooling": **prevention is a cheap
builder-side prompt; the durable external artifact is an executable gate** that fires
whether or not attention was directed — the backstop for weak models, sticky
conventions, and the runs where no one thinks to prompt. Contracts and "fresh-eyes"
review are not the lever.

### The arc (each round tempered the last)
1. **Pilot** — contracts looked decisive. [findings/01-pilot.md](findings/01-pilot.md)
2. **De-biased, 2 models** — the redaction win was an apparatus artifact; removed, it vanished. [findings/02-debiased-study.md](findings/02-debiased-study.md)
3–6. **Multi-domain** — types-only agents converged on correct designs with no contract. [findings/06-multidomain.md](findings/06-multidomain.md)
7. **Competence trap** — one seam broke *more* on stronger models, but it failed to replicate: a symptom of an *unstated positive requirement*, not a capability law. [findings/07-competence-trap.md](findings/07-competence-trap.md)
8. **Blind-spot study** — the attention result, a self-vs-external detection asymmetry, and a prevention/detection dissociation (n=5). [findings/08-blindspot.md](findings/08-blindspot.md)
9. **Hardening** — the core held and sharpened (attention-allocation); two round-8 sub-claims did **not** survive. [findings/09-round9.md](findings/09-round9.md)
10. **Full-paper grid** — the mechanism ablation (integration-direction ≫ generic effort) + a steep capability floor, 5 breaking seams, CIs. [findings/10-fullpaper-grid.md](findings/10-fullpaper-grid.md)

### Read
- Narrative: [drafts/POST.md](drafts/POST.md)
- Working paper / extended abstract (v2): [drafts/PREPRINT.md](drafts/PREPRINT.md)
- Where this is headed + venues: [drafts/SUBMISSION_NOTES.md](drafts/SUBMISSION_NOTES.md)
- Raw per-trial data: [data/](data/)

## Status & honesty

A **hardened lab note / extended abstract**, not yet a full paper.

### Round 9 (done): what held and what didn't
- **Held + sharpened:** closure replicates across breaking seams (pagination,
  caching, staging) with Wilson CIs; the failure is an *attention-allocation* gap —
  agents name the requirement ~100% when asked but break unprompted.
- **New bounds:** a **capability floor** in the fix (Opus ~75% vs Haiku ~44% rescue)
  and a **sticky-convention ceiling** (caching half-closes).
- **Retracted:** the self-vs-external detection asymmetry (round 8's 53% vs 27%, n=5,
  one judge) **did not replicate** — at n=12 with two judges both detect at ~ceiling.
- **Honest miss:** 2 of 4 new affordance-seams didn't break at all; reliable-breaking
  under-specified seams are hard to construct.

### To reach a full paper
More reliably-breaking seams; n above 30 with a pre-registered analysis; a
human-validated judge subset; full-build (not isolation) replication; an older/smaller
model to pin the capability floor; an attention-prompt-vs-self-revision ablation.

## Reproduce

Apparatus (TypeScript modules + probe harnesses, each self-tested with a
golden/broken reference before any agent ran) and per-trial data are in this repo.
Aggregated data in [data/](data/). Builders: Claude Opus 4.8 / Sonnet 4.6 / Haiku
4.5; judges: Opus 4.8 + Sonnet 4.6.

## License
[MIT](LICENSE). Please cite the preprint if you build on this.
