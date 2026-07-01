# The problem with a fleet of AI agents isn't that they collide

*Each one quietly does something reasonable and wrong — and knew better the whole
time. I couldn't find research that told me how to stop it, so I ran the experiments
in the open. Here is what survived, and what it proved about building software with
autonomous agents.*

We are about to hand a great deal of work to autonomous software. Not one assistant
answering one question, but *fleets* of AI agents working in parallel — each taking a
slice of a job and building it at the same time as the others. It is the only way the
arithmetic works: ten agents finish in roughly a tenth of the time. The obvious fear
is that they will trip over each other, like too many hands in one engine. That fear
turns out to be mostly misplaced, and the real failure is stranger, more common, and
much more fixable.

I run a small lab. One of the things I build there is a tool — I call it Seasar — that
takes an idea for a piece of software and turns it into a *build order* for a fleet of
coding agents: which agent builds which part, how the parts are meant to fit together,
and what has to be checked before the whole thing is trusted. Think of it as the
general contractor for a crew of AI builders. Its entire reason to exist is to let
many agents build one system without silently breaking each other's work — so that
the autonomy is *safe*, not just fast.

Before putting more weight on that tool, I wanted an honest answer to a plain
question: is any of this grounded in evidence? What does the research actually say is
the best way to coordinate a fleet of parallel agents?

I went looking, and the looking was itself the first result. There is a real and
growing literature on getting multiple AI models to work together, and most of it is
organized around *coordination*: give the agents richer contracts, define protocols,
let them review one another. But two things were true at once. The field turns over in
months, and much of what I found was written against an earlier generation of models,
on a frontier that has since moved. And the guidance that was current was mostly
advice-shaped — "use contracts," "add a reviewer" — without the receipts that would
tell me whether it works, when, and by how much. I did not want to build my tool on a
citation that was already stale, or on my own taste dressed up as a principle.

So I ran the experiments myself, and I put the whole thing in a public repository as I
went.

## The setup

The apparatus is deliberately small and hermetic. Each experiment is a little software
system whose pieces are built by independent agents that see only the shared
*interfaces* — the shapes of the parts — and never each other's code. That is exactly
the parallel situation: everyone building to the same blueprint, nobody watching the
seams. Then the pieces are assembled and a probe runs the whole thing end to end, to
ask the only question that matters: does it actually *behave* correctly — not merely
compile? About two thousand agent-built modules, across twelve rounds. I held myself to
one rule: every result had to survive my trying to kill it. A finding I liked stayed a
suspect until I had tried, and failed, to make it disappear.

## What I expected, and what actually broke

I expected the answer to be *better contracts* — that the elaborate agreements between
agents were what prevented collisions. The evidence retired that belief almost
immediately. When the requirement is implied by the interfaces, capable agents simply
derive the right design without being told. The contract was mostly redundant.

What actually broke things was quieter, and it had a shape I came to recognize. The
specification would offer a knob — how many results to return at a time, how long to
cache a value, how many times to retry a payment — and the agent would reflexively
reach for a reasonable best practice that happened to violate something obvious but
unstated. Give a coder a "page size" setting and they will paginate, even when the one
place that uses it needed *everything at once*, so the final report silently
undercounts. Give them a cache timer and they will cache, even when the value has to be
fresh, so a flag flipped elsewhere is silently ignored for a minute. Each agent did
something a competent engineer does every day. Each was locally right and globally
wrong, precisely at the seam where its work met everyone else's.

Then came the finding that reframed the whole problem. I asked the agents, separately,
to list the ways their own module could break the finished system. They named the exact
requirement almost every time — near a hundred percent, with two independent AI judges
agreeing and a manual read to confirm. They *knew*. Left alone to build, they broke it
anyway. So this was never a knowledge problem, and never a communication problem. It
was a matter of where attention went. They did not look at the seam, because their
attention was on their room.

## The fix, and its price

If the failure is inattention, the remedy is to direct attention. One line, added to
each agent's instructions — *before you finish, consider how this could silently fail
once it is assembled into the whole system, and make sure it doesn't* — took a strong
model from correct about **21%** of the time to about **82%**.

And it mattered that the instruction was *specific*. A generic "review your work
carefully for bugs" helped far less — to about **44%**. Telling an agent to try harder
is not the same as telling it where to look. Pointing it at the seam is the lever;
general diligence is not. When I rebuilt the test as a true multi-agent pipeline —
three separate agents, one system, assembled and tested end to end — the pattern held:
broken every time without the nudge, and the nudge closed most of it.

This is not magic, and the places it fails are the whole reason the story has a second
half. Three limits, each a reason a prompt can never be the entire answer:

- **A capability floor.** The direction only helps a model strong enough to act on it.
  The same sentence that lifts a frontier model from 21 to 82 barely moves a small one.
- **Sticky conventions.** Some habits resist every phrasing. "Cache for speed, forget
  that it goes stale" stayed broken about half the time even when I pointed straight
  at it.
- **The one I almost got wrong** — which is the most important, so I will tell it
  plainly. I re-ran the test across other model families, and the effect vanished. The
  tidy headline *this is specific to one model maker* was one keystroke from being
  published. Before shipping it, I ran a single further control: the very same model
  that showed the effect, but asked to write the code in one shot instead of as an
  agent that reads, reasons, and builds over several steps. The effect vanished there
  too. So the difference was never the brand of model. It was the *setting*. The fix
  lives in the agentic posture — an agent working step by step toward an assembly — and
  disappears when you ask for the answer in a single breath. My clean claim had been an
  artifact of comparing two different machines. I retracted it before it left the
  building.

That last limit is exactly why a prompt cannot be the product. If the effect depends on
the setting, degrades with model strength, and fails on the stubborn cases, then you
cannot ship an instruction and call the result safe. You have to ship the *check*. The
durable artifact is an **executable gate** — an automatic test, generated alongside the
build, that catches the silent failure whether or not anyone remembered to point an
agent's attention at the seam.

## What the research made the method

The experiments changed what my tool is *for*. I began believing that elaborate
contracts between agents were what kept them from colliding. The evidence handed me
something cheaper and sturdier. The proven method for Seasar, as of now, is three
moves:

1. **Keep the agents in the agentic posture the effect actually lives in** — reading
   toward an assembly, building in steps, not generating a whole answer in one shot.
2. **Inject one integration-directed instruction into every agent's task**, pointing
   its attention at the seam where its work meets the rest.
3. **Emit executable gates into the build** that verify the end-to-end behavior
   automatically — the backstop for weak models, sticky conventions, and the runs where
   nobody thinks to prompt.

It is lighter than the tool I set out to build. And, unlike the tool I set out to
build, it rests on experiments I published and actively tried to break.

## The thesis under the specifics

There is a general lesson here, and it is not really about software. As we delegate
more to autonomous systems, the scarce ingredient is not intelligence. These agents
usually *know* the right answer — the core of my result is that they can name it and
then fail to use it. What is scarce is **verification**: pointing attention at the
place things quietly go wrong, and then checking the work instead of trusting the
description of it. That holds for a fleet of coding agents, and it holds for the humans
meant to supervise them. It is also why I ran this in the open and spent as much effort
trying to disprove my own tool as to defend it — the same principle, turned on myself.
Make the work prove itself.

Safe autonomy will not come from smarter agents alone, or from more elaborate protocols
between them. It will come from making their work verifiable at the seams. Verification
is the product — for the code, and for the people relying on it.

---

*Method, raw per-trial data, and every built artifact are open at
[github.com/yiyaw-lab/seasar-research](https://github.com/yiyaw-lab/seasar-research).
Run with several models as agentic builders (Claude Opus, Sonnet, Haiku) and two of
them as independent judges, plus a single-shot control across model families (Opus,
Sonnet 5, GPT-5, Grok). The central experiment is n=30 per cell with Wilson confidence
intervals; the multi-agent pipeline replication spans four failure seams. This is a
hardened lab note and working paper, not yet a peer-reviewed paper: the judge is an AI
corroborated by a manual audit rather than human-validated at scale, and — the honest
frontier — the fix is demonstrated inside an agentic harness, which leaves transfer
across model families genuinely untested. That is the next experiment.*
