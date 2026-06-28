# Does a behavioral interface contract prevent integration bugs that a typed contract does not?

**A controlled n=1 experiment in parallel multi-agent software construction.**

Date: 2026-06-26 · Builder model: Claude Opus 4.8 (1M) · Seas / Seasar research

---

## 1. The question

Seasar (our build-order compiler) bets that to let N autonomous agents build one
codebase in parallel, you must pin the *seams* between their modules. The 2026
research frontier supports the general shape of that bet — coordinated parallel
multi-writing now beats isolation on benchmarks **when coordination is a designed
structural mechanism, not a prompt** (DeLM, STORM, TraceFix, all 2026). But those
results validate a *runtime* coordination substrate. Seasar ships a *static,
design-time* substrate, and its distinctive claim is that the contract should pin
**behavior** (ordering, idempotency, units, error model, responsibility
assignment), not just **types**.

A prior cheap test in this project showed that without *any* contract, independent
agents diverge at ~8/9 interface seams (wrong event names, method names, field
names). But that only proves a *typed* interface helps — which nobody disputes.
The open question, and the one thing that justifies Seasar existing rather than
"just use TypeScript," is:

> **Does pinning behavior buy anything beyond pinning types?**

## 2. Method

A two-arm controlled experiment. Six agents per arm build one module each of a
real CLI ("Quietkeep" — a watch/draft/approve/commit git tool), in parallel, each
seeing only the contract, never each other's code.

- **Independent variable (the only difference):** Arm A agents additionally
  received `BEHAVIOR.md`, which assigns, for each cross-module responsibility the
  constitution requires, *which module owns it* and the precise runtime semantics.
  Arm B received the typed contract only.
- **Held constant:** identical `SPEC.md` (product + a 5-point constitution that
  *states the requirements* — both arms know redaction and merge-pause are
  mandatory), identical shared `types.ts` (full data shapes + factory
  signatures), identical environment, identical builder model.
- **Dependent variable:** a fixed runtime probe battery (`probes/harness.ts`),
  identical for both arms, exercising only the public interface. Probes test
  behavior the constitution already requires: P0 happy-path commit, P1
  secret-redaction-before-LLM, P2 offline network-gating, P5 no-commit-during-merge,
  P6 hook-failure error model. Plus an 8-format redaction stress test.

The apparatus (`types.ts`, `BEHAVIOR.md`, `SPEC.md`, the probes) was authored by
the experimenter so the variable is clean. All code is in this directory;
`./run_probes.sh` reproduces the measurement.

## 3. The prediction — and how it was wrong

I predicted Arm B would leave a **gap**: two modules each assuming the *other*
owns a safety responsibility, so *neither* does it (daemon passes a raw diff
assuming the messager redacts; messager assumes it gets a clean diff → leak).

**That prediction was wrong, and the way it was wrong is the most interesting
result.** Arm B agents did the opposite of leaving a gap: uncertain who owned each
safety responsibility, **each defensively grabbed it**. Redaction ended up
*duplicated* (both daemon and messager redact); the merge-pause guard ended up
*triplicated* (daemon, committer, and approval all check). Independent agents,
under a known-but-unassigned safety requirement, **over-guard** rather than
under-guard. So the naive "they'll leave a gap" intuition is the wrong failure
model. The real failure modes were subtler — and the experiment still found two.

## 4. Results

### Probe battery

| Probe | Arm A (typed+behavioral) | Arm B (typed-only) |
|---|---|---|
| P0 happy-path commit | ✅ commits | ❌ **never commits** |
| P1 redaction before LLM | ✅ no leak | ❌ **secret leaked to LLM** |
| P2 offline gating | ✅ | ✅ |
| P5 no-commit-during-merge | ✅ | ✅ (in fact triple-guarded) |
| P6 hook-failure honored | ✅ surfaced | ⚠️ vacuous pass* |
| **Score** | **5/5 genuine** | **2 genuine / 2 hard-fail / 1 masked** |

\*P6 "passes" its assertion (no commit) but for the wrong reason: the staging
break (P0) refuses the commit *before the hook ever runs*, so the hook-failure
path is never actually exercised. One seam break cascades into a false-green on
another probe.

### Redaction stress (8 secret formats, LLM path forced on)

| | classic `sk-` | `sk-proj-` | AKIA | `ghp_` | JWT | 40-char hex | `password=` | `AWS_SECRET=` | **leaked** |
|---|---|---|---|---|---|---|---|---|---|
| **Arm A** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | **0/8** |
| **Arm B** | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | **3/8** |

Arm A is *robustly* better, not lucky on one input.

### Who-owns-what (from the agents' own structured decision records)

| Seam | Arm A | Arm B |
|---|---|---|
| Redaction | messager only; daemon passes raw | **daemon AND messager** (each cites the other as "harmless backstop") |
| Merge-pause guard | committer authoritative (+ 2 defensive) | **daemon ×2 + committer + approval** |
| Staging | committer stages `draft.files` unconditionally | committer stages **only if `autoAdd`** (default false) → no-op |

## 5. Mechanism (directly observed, not inferred)

**Finding 1 — the staging seam break (P0).** The typed contract says
`committer.commit(draft): Promise<CommitResult>`. It does not say *whether the
committer stages `draft.files` itself or only commits what is already staged.*
Arm A's `BEHAVIOR.md` pinned "stages exactly `draft.files`." Arm B's committer
agent, with no pin, chose: `if (config.autoAdd && draft.files.length) git add …`
else stage nothing, then refuse if the index is empty. `autoAdd` defaults to
false; the daemon and approval modules assume "approved ⇒ it commits." Result:
**the assembled Arm B pipeline silently never commits anything.** Every module
compiles; every module is individually defensible; the *integration* is dead.
This is the cleanest demonstration, because `BEHAVIOR.md` here only assigned
*ownership/semantics* — it conveyed no extra capability — and that alone flipped
commit vs. no-commit.

**Finding 2 — the redaction completeness gap (P1, 3/8).** Redaction in Arm B was
*duplicated*. The daemon agent wrote: *"redaction happens HERE … If the messager
redacts again, that is harmless idempotent double-coverage … not a claim of
completeness; it is the 'obvious secrets' floor."* The messager agent wrote:
*"even if a caller also redacts, this is safe."* **Both built a redactor; both
explicitly leaned on the other as a backstop; neither built a complete one.** The
proximate cause is a one-character regex difference plus two missing rules:

```
Arm A messager:  /\bsk-[A-Za-z0-9_\-]{16,}\b/   + JWT rule + 40-char blob catch-all
Arm B (both):    /\bsk-[A-Za-z0-9]{16,}\b/      (hyphen NOT in class; no JWT; no blob rule)
```

`sk-proj-…` (the modern OpenAI key format) contains a hyphen after `proj`, which
ends Arm B's character class after 4 chars (< 16) → no match → the secret reached
the LLM prompt verbatim, under a header literally reading *"secrets already
removed."*

## 6. Interpretation

Two distinct values of a *behavioral* (vs merely typed) contract showed up, and
they are worth separating because they have different strengths of evidence:

- **Ownership assignment prevents integration breaks (strong).** The staging seam
  (Finding 1) is the clean result: types underdetermine *who does the work*; when
  two agents fill that gap with different-but-individually-reasonable assumptions,
  the system compiles and silently does nothing. Naming the owner fixes it. This
  is the same failure Cognition names as *"actions carry implicit decisions, and
  conflicting decisions carry bad results"* — here made concrete and measured.
- **Completeness expectation improves safety implementations (softer).** The
  redaction result (Finding 2) is partly that `BEHAVIOR.md` enumerated the secret
  classes to cover ("…long hex/base64 secrets"), and partly that single ownership
  concentrated investment while duplicated ownership diffused it. A careful
  types-only agent *could* have written Arm A's redactor; the contract made it the
  default rather than the exception.

And the headline structural finding: **the failure mode of unassigned
responsibility is not a gap but diffusion** — agents over-guard safety seams
(defense-in-depth) while their duplicated implementations each under-invest, and
they diverge freely on *non-safety* functional seams (staging) where no
constitutional alarm fires. A type checker sees none of it.

This is consistent with the 2026 frontier (structure beats prompts for parallel
coding) and extends it with a specific, mechanistic claim about *what kind* of
structure: **assigning cross-module responsibility** is load-bearing in a way the
type system cannot express.

## 7. Threats to validity (read before believing this)

1. **n = 1.** One order, one builder model, one run per arm. Directional, not
   conclusive.
2. **Apparatus authored by the experimenter.** `BEHAVIOR.md`'s redaction line
   enumerates secret types — a skeptic can fairly say Arm A was "told the answer"
   on redaction *coverage*. This is why Finding 1 (staging) is weighted as the
   stronger result: there, `BEHAVIOR.md` assigned only ownership, no extra
   capability, yet flipped the outcome.
3. **Strong builder model (Opus 4.8).** Weaker models might gap rather than
   duplicate; stronger ones might converge without a contract. The effect size is
   model-dependent.
4. **Subset of seams probed.** Only redaction, staging, merge-pause, gating,
   hook-failure. Batcher/watcher seams (units, debounce, emission semantics) were
   built but not runtime-probed.
5. **Arm comparison is typed-vs-typed+behavioral, not no-contract.** The
   no-contract condition (already shown to fail on shapes) is a separate, weaker
   baseline.

## 8. How to strengthen toward publishable n>1

- Repeat across ≥5 distinct orders (the 4 other compiled orders on disk: PaperPulse,
  Marginalia ×2, EchoIndex) and ≥3 runs each — report leak-rate and
  commit-success distributions.
- Re-run with a blind third party authoring `BEHAVIOR.md` from the constitution,
  to remove experimenter-coverage bias on the redaction finding.
- Add a third arm: typed + *runtime-mediated* coordination (the DeLM/STORM style)
  to test Seasar's static-vs-runtime gap directly.
- Vary builder model (Sonnet, a weaker OSS model) to map where over-guarding flips
  to gapping.

## 9. Reproduce

```
cd runs/seasar-behavioral-contract-ab
(cd armA && npm i && npm i -D @types/node@20)   # typescript + node types
(cd armB && npm i && npm i -D @types/node@20)
./run_probes.sh                                  # compiles both arms, runs the battery
# redaction stress: (cd armX && ./node_modules/.bin/tsc && node dist/probes/stress.js)
```

Raw data: `results.json`. Apparatus: `fixtures/`. Built modules: `armA/`, `armB/`.
Each agent's structured seam-decision record is in the build workflow output
(task `ws0hs4un7`).

## 10. Bottom line

On this n=1, a behavioral contract bought two things a typed contract did not: it
**prevented a silent whole-pipeline functional break** (the staging seam) and
**reduced secret-leak rate from 3/8 to 0/8**. The mechanism is *responsibility
assignment*, and the surprising structural fact is that unassigned responsibility
fails by **diffusion and divergence, not by gaps**. That is real support for the
distinctive half of Seasar's thesis — pin behavior, not just types — tempered by
the honest caveats above, chiefly that this is one sample and that part of the
redaction win rode on the contract doubling as a more complete spec.
