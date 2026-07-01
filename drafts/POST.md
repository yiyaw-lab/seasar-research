# Attention, Not Intelligence

*My AI agents reached for the wrong answer while knowing the right one, until I told
them where to look. What a swarm of them taught me about attention, and about the blind
spot they inherited from us.*

We are about to hand a great deal of work to autonomous software. Not one assistant
answering one question, but fleets of AI agents working in parallel, each taking a
slice of a job and building it at the same time as the others. It is the only way the
arithmetic works: ten agents finish in roughly a tenth of the time. The obvious fear
is that they will trip over each other, like too many hands in one engine. That fear
is mostly misplaced. The real failure is stranger and more common, and once you can
see it, more fixable.

My training is in cognitive science, not software engineering. I now spend my time
building and testing systems of AI agents, and one of the things I have built is a tool
I call Seasar. It takes an idea for a piece of software and turns it into a build order
for a fleet of coding agents: which agent builds which part, how the parts are meant to
fit together, and what has to be checked before the result can be trusted. It is closer
to a general contractor for a crew of builders than to a single assistant. The point of
it is to let many agents build one system without breaking each other's work, so that
the autonomy is safe and not only fast.

Before trusting the tool with more, I wanted to know whether its approach rested on
anything. What does the evidence say about coordinating a fleet of parallel agents?

The work I found split into two camps. One holds that a model improves its own output
when asked to reflect on it and revise; Reflexion (Shinn and colleagues, 2023) and
Self-Refine (Madaan and colleagues, 2023) are the well-known versions. The other is
skeptical, with results like Huang and colleagues at DeepMind arguing that models
cannot reliably self-correct their reasoning without an outside signal. A third line,
multi-agent debate (Du and colleagues, 2023), has several models argue toward a
consensus answer. All of it was interesting and little of it fit my case. Most studied
one model reasoning through one task, or a few models debating a single answer, rather
than many agents constructing different parts of one system that then has to work as a
whole. A fair amount of it was written against a generation of models the frontier has
already passed. I did not want to lean on a citation aimed at a different question and
half a generation old.

Cognitive science, though, had a name for what the engineering literature was missing,
which is a large part of why I went into these experiments looking for an attention
problem rather than a coordination one. So I ran them, and kept the whole
record in a public repository as I went.

## The setup

Each experiment is a small, self-contained software system whose parts are built by
independent agents. An agent sees only the shared interfaces, the shapes of the parts,
and never the other agents' code. That is the parallel situation in miniature:
everyone building to the same blueprint, nobody watching the joins. The parts are then
assembled and a probe exercises the finished system from end to end, to test the only
question that counts, which is whether it behaves correctly and not merely whether it
compiles. About two thousand agent-built modules across twelve rounds. I held to one
rule the entire time: a result counted only if it survived my trying to break it.
Anything I was pleased with stayed a suspect until I had tried and failed to make it go
away.

## What I expected, and what actually broke

I expected the answer to be better agreements between the agents. The tool leaned on
what I call agent behavioral contracts: explicit statements, attached to each agent's
task, of who is responsible for what at each boundary, and what each part must
guarantee the others. The evidence retired that assumption quickly. When the
requirement is already implied by the shared interfaces, a capable agent derives the
right design on its own, and the behavioral contract adds nothing. It was mostly
redundant.

What broke things was narrower, and it had a shape I learned to recognize. The
specification would offer a control: how many results to return per call, how long to
hold a cached value, how many times to retry a payment. The agent would reach for a
standard best practice that violated a requirement no one had written down because it
seemed too obvious to state. Hand a coder a page-size setting and they paginate, even
when the one caller needed every record at once, so the final count comes out too low.
Hand them a cache lifetime and they cache, even when the value has to be current, so a
change made elsewhere goes unseen for a minute. Every one of these is something a
competent engineer does on an ordinary day. Each was right in the small and wrong in
the large, at the exact boundary where one agent's output became another's input.

Then came the result that changed how I understood the problem. I asked the agents, one
at a time, to list the ways their own module could break the finished system. They
named the exact requirement almost every time, a rate near a hundred percent, with two
independent AI judges agreeing and a manual read to confirm. They knew. Set loose to
build, they violated it anyway.

This is where the cognitive science earned its place, because the pattern already has a
name. In developmental psychology it is called a production deficiency: someone
possesses a capability or a strategy and fails to deploy it on their own, while a small
cue is enough to bring it out. It is not blindness. In the classic inattentional
blindness studies, where a viewer misses an obvious event because attention is loaded
elsewhere, the person genuinely does not see it. Here the agents were not blind at all;
they could describe the gap in detail the moment they were asked to look. The knowledge
was present and sitting idle. What was missing was the cue that would send attention to
the seam. That reading told me the fix would not be more capability or more
communication between agents. It would be a prompt that points.

There is something worth pausing on there. Production deficiency and inattentional
blindness were found in people, decades before any of this. The models are trained on
us, and they appear to have inherited more than our knowledge; they inherited the habit
of holding a fact and never consulting it. I had reached for cognitive science to
explain the agents, and the agents were handing the explanation back in a form my own
field almost never gets: a blind spot I could open and close on command, across
hundreds of trials, with one line of text as the only thing that changed. You cannot
run that experiment cleanly on a person. You can run it on a swarm of agents all
afternoon, and what it shows you is as much about the people who supervise them as
about the software.

## The fix, and its price

If the failure is a missing cue, the remedy is to supply one. I added a single line to
each agent's instructions: before you finish, consider how this could fail once it is
assembled into the full system, and make sure it does not. On a strong model,
correctness rose from about 21 percent to about 82.

The specificity did the work. A generic instruction to review the code carefully for
bugs helped far less, reaching about 44 percent. Telling an agent to try harder is not
the same as telling it where to look, and only the second closed the gap. When I
rebuilt the test as a real multi-agent pipeline, three separate agents building one
system that was then assembled and tested, the pattern survived: broken every time
without the cue, mostly repaired with it.

None of this is a spell, and the places it fails are the reason the story does not end
here. The cue only helps a model strong enough to act on it; the sentence that lifts a
frontier model from 21 to 82 barely moves a small one. Some habits resist every
phrasing, and the reflex to cache for speed and forget that the value goes stale stayed
broken about half the time even when I aimed straight at it.

The third limit is the one I nearly got wrong, and it is the most important, so I will
be plain about it. I ran the test across other model families and the effect
disappeared. The clean headline, that the effect was specific to one model maker, was a
keystroke from going out under my name. Before writing it, I ran one more control. I
took the same model that showed the effect and asked it to produce the code in a single
response, instead of as an agent reading, reasoning, and building across several steps.
The effect vanished there too. So the real variable was never the make of the model. It
was the setting. The cue works when an agent is building step by step toward an
assembly, and stops working when the code is demanded in one shot. My tidy cross-family
claim had compared two different machines and mistaken the difference for a fact about
models. I dropped it before it went anywhere.

That last limit is why the cue cannot be the entire method. If it depends on the
setting, weakens with model strength, and fails on the stubborn cases, an instruction
is not something you can ship and then call the result safe. You have to ship the
check. The lasting part of the method is an executable gate: an automatic test,
generated with the build, that catches the failure whether or not anyone remembered to
prompt for it.

## What the research made the method

The experiments changed what the tool is for. I started out believing that agent
behavioral contracts were what kept a fleet from colliding. The evidence handed me
something cheaper and more durable. As Seasar works now, the method has three parts. It
keeps the agents in the step-by-step building setting where the effect is real, rather
than asking for finished code in one pass. It attaches one integration-directed cue to
each agent's task, aimed at the boundary where its work meets the rest. And it writes
executable gates into the build that verify end-to-end behavior on their own, to cover
the weak models, the stubborn conventions, and the runs where no one thinks to prompt.
It is a lighter tool than the one I set out to build, and it has the advantage of
resting on experiments I published and tried to break.

## What it comes down to

The surprise in all of this was never about intelligence. The agents had plenty of it.
They could state the right answer on request and then reach straight past it, which is
about the most human way there is to fail. What decided the outcome was attention:
where it landed, and whether anything sent it to the right place. Attention is also the
part you cannot trust, in a machine or in a person. It wanders. It settles on the room
and not the seam. That is why the lasting half of the method is not a better
instruction but a check, an executable gate that looks whether or not anyone remembered
to.

It is the same blind spot that undoes any group of people who each know their own part
and assume the parts will add up. As we route more work to systems like these, what
runs short is not model intelligence but the willingness to aim attention at where
things go wrong and to verify the result rather than the account of it. That holds for
the agents and for the humans meant to watch them. It is why I ran this in the open and
spent as long trying to sink my own tool as to defend it. If the work cannot prove
itself, it does not count as done.

---

*Method, raw per-trial data, and every built artifact are open at
github.com/yiyaw-lab/seasar-research. The experiments used several models as agentic
builders (Claude Opus, Sonnet, Haiku) with two of them as independent judges, plus a
single-response control across model families (Opus, Sonnet 5, GPT-5, Grok). The
central experiment is n=30 per cell with Wilson confidence intervals, and the
multi-agent pipeline replication spans four failure seams. This is a hardened lab note
and working paper rather than a peer-reviewed result: the judge is an AI corroborated
by a manual audit rather than validated by humans at scale, and the fix is shown inside
an agentic harness, which leaves transfer across model families untested. That is the
next experiment.*

*Referenced work: Reflexion (Shinn et al., 2023); Self-Refine (Madaan et al., 2023);
"Large Language Models Cannot Self-Correct Reasoning Yet" (Huang et al., DeepMind,
2023); multi-agent debate (Du et al., 2023); inattentional blindness (Simons & Chabris,
1999; Mack & Rock, 1998); production deficiency in strategy use (Flavell and the
developmental-psychology tradition).*
