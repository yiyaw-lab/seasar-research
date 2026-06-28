# Where to publish, in what order, and what to fix first

## Honest grade
The **closure result** (an integration-priming prompt takes silent-failure rate
from ~80% to ~4%) is clean and robust. But "self-critique helps code" is
well-trodden (Self-Refine, Reflexion). The **defensible novelty** is the
combination, in a *code-integration* setting:
1. the eight-round arc showing inter-agent **contracts are mostly redundant**; what
   matters is whether a *positive requirement* is stated;
2. the **prevention/detection dissociation** (the prompt fixes the artifact without
   the agent being able to name the gap);
3. the **self vs external detection asymmetry** for integration gaps (~27% vs ~53%).
Frame the paper around (2) and (3), grounded in the self-correction-limits
literature — **not** around "reflection helps," which reviewers will call known.

At current n (5–6/cell, one headline seam) this is **workshop / NIER / extended-
abstract grade**, not a main-conference paper. That is fine for the sequence below.

## Related work to cite (and position against)
- Huang et al., "Large Language Models Cannot Self-Correct Reasoning Yet" (2023) —
  our dissociation is a nuanced data point: the *artifact* self-corrects under a
  generic nudge even though *explicit self-detection* is poor.
- Self-Refine (Madaan 2023), Reflexion (Shinn 2023) — position: we show the
  improvement can be attention, not understanding, and is weaker for self than
  external review.
- Anthropic "Building Effective Agents" (2024); Cognition "Don't Build Multi-Agents"
  (2025) — the coordination-contract framing we test.
- Multi-agent failure taxonomies (MAST, 2025) — "inter-agent misalignment" as the
  failure class we operationalize.
(Verify each citation before submitting — do not ship a bibliography from memory.)

## Publishing sequence (your plan, made concrete)
1. **Personal site (yiya.dev)** — `POST.md`, in your voice. Run a disclosure +
   secrets scan first (the /publish gate). Ship now; it is lab-note grade and
   honest.
2. **Seasar research page** — the same narrative + a link to `BLINDSPOT_FINDING.md`
   and the repo. Add the explicit product-repositioning paragraph (prime builders;
   external tool = audit + executable gates). This doubles as product comms.
3. **arXiv preprint** — `PREPRINT.md`, **after** the hardening pass below. Categories:
   primary **cs.SE**, cross-list **cs.AI** (and cs.LG if you lean the LM-behavior
   framing). arXiv has no bar but reviewers will later judge the n — harden first.
4. **Workshop / NIER submission** — pick by framing:
   - SE-for-AI framing (recommended fit): **AIware** (AI-powered software conf) or
     **LLM4Code** (ICSE-co-located workshop). Both value focused empirical agent
     results.
   - "promising preliminary": **ICSE NIER** or **FSE Ideas/Vision**.
   - ML-agent framing (self-reflection vs verification): an **LLM-Agents workshop at
     NeurIPS / ICLR / ICML**, or a **COLM** workshop.
   Submit the hardened version; lead with the dissociation.

## Hardening to clear a real bar ("round 9", ~1 day of compute)
Do these before arXiv/workshop, in priority order:
1. **More seams** (≥6 under-specified, varied imported conventions and requirement
   phrasings) — generalizes the closure + dissociation beyond staging.
2. **Larger n** (≥20/cell) with reported confidence intervals; the detection
   asymmetry is currently too noisy (Sonnet was a wash) to assert without it.
3. **Human-validated judge** on a sampled subset (+ a second LLM judge for
   agreement) — removes the single-LLM-judge threat.
4. **Priming-vs-revision ablation** — separate "knowing a critique is coming"
   (careful build) from "actually revising," to state the mechanism cleanly.
5. **A weaker/older model** — find where the attention nudge stops working (maps
   the capability floor).
6. **Full-build (not committer-isolation) replication** of the closure result.

## Pre-publish checklist
- Disclosure + secrets scan on every artifact you post (no keys, no internal paths
  that matter).
- Re-verify all citations against the real papers (titles, authors, years).
- Decide the Seasar framing deliberately: "we tried to break our own tool and
  repositioned it" is credible and good for the brand, but it is a public
  down-weighting of the original pitch — own it as a strength.
- Keep the claim narrow. The reviewers who will respect this are the ones who
  notice you tested yourself eight times; do not let the post overclaim past the n.
