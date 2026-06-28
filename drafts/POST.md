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

## What I'm testing next (and what would prove me wrong)

This is a lab note, not a paper. The closure result (the prompt taking failure from
~80% to ~4%) is solid. The two claims I most want to stress before I'd defend them
in public are the ones with small samples: that fresh eyes detect the gap about
twice as well as self-review, and the strange dissociation where the fix doesn't
require the awareness.

So the next experiment is a deliberate attempt to break those, pre-registered here
so I can't move the goalposts:

- **More seams, different traps.** Six or more under-specified seams, each with a
  different "best practice" an agent might wrongly import, not just the one git
  convention. If the dissociation only shows up for staging, it isn't real.
- **Enough runs to trust the gap.** Twenty-plus runs per condition with confidence
  intervals. Right now the self-versus-external detection numbers are too noisy to
  bet on; one model was a wash.
- **A scorer I can check.** A human-validated subset plus a second independent
  judge, so the detection rates aren't one model grading another unaudited.
- **Find the floor.** A weaker, older model than the three here, to locate the
  point where a one-line attention prompt stops being enough. That floor is the
  real product-relevant number: below it you need the external gates, above it you
  may not.

My prediction: the closure effect holds everywhere, the dissociation holds across
most seams, and there is a capability floor below which priming alone fails. If the
dissociation evaporates once I widen it, I'll say so here too. The whole point of
doing it this way is that the result is allowed to disappoint me.

---

*Method, raw data, and every built artifact are at github.com/yiyaw-lab/seasar-research The whole thing was
run with Claude (Opus, Sonnet, Haiku as builders, Sonnet as judge), n=5 to 6 per
condition. It is a lab note, not a paper yet. The next version widens it to more
seams, more runs, and a human-checked scorer.*
