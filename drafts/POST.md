# I tried to prove my AI-orchestration tool was grounded in research. It mostly wasn't. Here's what I found instead.

I run a small agent lab. One of the things I build there, Seasar, takes an idea
and compiles it into a "build order" for a fleet of coding agents working in
parallel: a task graph, typed interface contracts between the modules, and quality
gates. The pitch was that the contracts are what let many agents build one codebase
without silently stepping on each other.

I wanted to know if that pitch was true, or if I was building on vibes. So instead
of guessing, I ran the experiment. Several, actually. About 330 agent-built modules
across eight rounds. The story of how the answer changed is more useful than the
tool.

## The first result looked great. Then it fell apart, twice.

I built the same small program twice with parallel agents: once where each agent
got a behavioral contract telling it who owns what at each seam, once with only the
shared types. The contract version worked; the types-only version shipped a real
bug (a secret leaked to an LLM, and the pipeline silently never committed). Win for
contracts.

Then I got skeptical of my own result and kept testing.

- The secret-leak win turned out to be an artifact: my contract had quietly listed
  which secret formats to redact. Remove that hint and the advantage vanished.
- I tried the same setup on three more domains. The agents, with no contract,
  just figured out the right design on their own every time.
- I found one case that did break (who stages files before a git commit), and it
  broke *more* on the smarter models, because they "knew" git's convention and
  applied it where it didn't fit. I got excited: maybe contracts matter more at the
  frontier, not less. Then I tried to reproduce that on two more seams and it
  refused to replicate. The smarter models complied fine whenever I stated the
  requirement clearly.

Every round tempered the one before it. What survived was boring and robust:
silent integration failures come from a *positive requirement nobody wrote down*
("a commit must actually result"), and the fix is to state it, ideally as a test.
Not contracts. Not a bigger model.

## The punchline study

That left the question my tool's whole existence depends on: can you *detect* the
missing requirement before the build? And specifically, is it a blind spot you can
catch by yourself, or do you need an outside reviewer?

I took the one seam that reliably broke and changed nothing but the prompt.

- Build it normally: it produced a working commit **about 20% of the time**. The
  rest silently did nothing.
- Build it after asking the agent one extra thing, "before you finish, list how
  this could silently fail when assembled," and commit-success jumped to **~96%**,
  on every model including the smallest one.

So the failure was never a capability gap. The agent could always do it right. It
just didn't look until asked. A one-line nudge closed it.

But here is the part I did not expect. I also scored whether the agents actually
*named* the specific gap in their self-review. They mostly didn't. Reviewing its
own work, an agent named the real problem **27% of the time**. A fresh agent that
had never written the code named it **53%** of the time, roughly twice as often.

Read those two results together and you get something genuinely strange: the nudge
*fixed the code* far more than it *produced understanding*. The agents started
staging files correctly while their own write-ups talked about other risks and
never mentioned the thing they had just gotten right. The fix runs through
attention, not comprehension. And the part that does require comprehension, saying
out loud what could go wrong, is exactly where an agent is worst at reviewing
itself and most helped by fresh eyes.

## What I'm doing with it

The honest read deflated the original pitch and pointed at a better one.

Preventing these silent failures is cheap, and it belongs inside the agent, not in
a compiler: just make every builder agent consider end-to-end failure modes before
it finishes. I'm baking that into the orchestration loop as a default.

The thing that actually needs an outside process is the other half. Agents are bad
at auditing their own work for what's missing, and fresh eyes roughly double the
catch rate, and even fresh eyes miss about half. So the durable value of an
external tool is not building, it is auditing: enumerate the integration
requirements an agent won't surface about its own code, and emit them as runnable
tests that catch the failure whether or not anyone reasoned about it, because, as
this study shows, the code being correct and someone understanding why come apart.

That is a much smaller and much truer claim than "behavioral contracts make
multi-agent builds better." It is also one I can stand behind, because I spent
eight rounds trying to break it and this is what was left.

## I ran the hardening (round 9), and where I was wrong

I pre-registered a deliberate attempt to break my own conclusions: more seams, a
weaker model, more runs, and a second judge to grade the grader. Here is what held
and what didn't.

**What held, and got sharper.** The closure effect is real. On the seams that
actually break, the one-line "consider how this could silently fail when assembled"
prompt takes correctness from roughly 12% to 75% on the strong model. And the
strange part resolved into something cleaner: when I *ask* the agents to list failure
modes, they name the exact missing requirement about 100% of the time, with two
judges agreeing 98% of the time, even though, left alone, they break. They already
know the requirement. They just don't look unless told to. So it is not that they
can't see the gap. It is where their attention goes. That is a tighter and truer
claim than I started with: an attention problem, not a blindness problem.

**What I got wrong.** Two things from the lab note did not survive more data.

- "Fresh eyes catch it twice as well as self-review" evaporated. At higher n with
  two judges, self-review and an outside reviewer both name the gap about 100% of
  the time. The 2x gap was a small-sample, single-seam artifact. Retracted.
- "A one-line prompt fixes it" needs two asterisks. It works far better on the
  strong model than the weak one (about 75% versus 44% rescue), and some conventions
  are sticky: caching staleness only half-closes even when primed. Cheap and
  powerful, but bounded by model strength and how reflexive the bad habit is.

And an honest miss: of four new traps I built, two didn't trip at all. The agents
just did the right thing by default. Making a coding agent reliably fall into a
silent integration bug is harder than it sounds, which is itself worth knowing.

**What it means.** Prevent these cheaply by directing the builder's attention with a
prompt, and back it with executable tests for the cases the prompt misses: weaker
models, sticky conventions, and the runs where nobody thinks to prompt. The external
value is not fresh eyes, they are no better. It is a gate that fires whether or not
anyone was paying attention.

---

*Method, raw data, and every built artifact are at github.com/yiyaw-lab/seasar-research.
Run with Claude (Opus, Sonnet, Haiku as builders; Opus and Sonnet as the two judges).
Round 9 used n=8 per closure cell and n=12 per detection cell, with Wilson confidence
intervals and two-judge agreement. It is now a hardened lab note / extended abstract,
not yet a full paper: three reliably-breaking seams, single-module isolation, and an
LLM (not human-validated) judge are the main limits. The next step to a full paper is
more breaking seams, n above 30, a human-checked judge subset, and an older/smaller
model to pin the capability floor.*
