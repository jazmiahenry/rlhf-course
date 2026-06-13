# Module 1A: The Hidden Problem (State vs Action Space)

> **What you'll get out of this:** the one distinction that explains why two
> equally smart AI assistants give you opposite answers, and why the usual
> "just write a better prompt" fix doesn't touch it.

## A quick gut check before we start

Ask two genuinely capable AI assistants the same thing: *"Help me research
vaccine effectiveness for a public health talk."*

Assistant A hands you a fast web summary with so-so sources in thirty seconds.
Assistant B pulls peer-reviewed studies but takes ten minutes and blows your
deadline.

Both understood you perfectly. Both had the same tools. Both would swear they're
"helpful and accurate." And they made opposite calls, without either one asking
which trade-off you actually wanted.

So here's the question this whole module answers: **why?** Not "why is AI
unreliable" in the hand-wavy sense. Specifically: what broke, and why won't a
better prompt fix it?

## What you'll be able to do

- Separate what modern AI does well (understanding the situation) from what it
  does inconsistently (deciding what to do about it).
- State the state-space versus action-space distinction precisely, and use it on
  a real interaction.
- Say, accurately, what post-training (RLHF, DPO, GRPO, RLVR) does and does not
  buy you for value-aligned decisions.
- Read the notation `π(tool | context, values)` and explain every piece.

## The part everyone gets wrong about 2026

There's an easy version of this story, and it's out of date. A few years ago you
could say AI picked tools "at random," that the act of using a tool was itself
flaky. That's no longer true, and a course that pretended otherwise would be
teaching you to fight a war that's already over.

In 2026, *using* a tool is solved enough. Models emit typed function calls
checked against a schema. The Model Context Protocol gives a standard way to
expose tools. Labs train tool use with reinforcement learning, so models have
genuinely learned that "compute the 47th Fibonacci number" calls for code, not a
guess. Ask a current model to do arithmetic and it reaches for the calculator.
Reliably.

So where's the problem now? It moved. The open question isn't *whether* the model
can call a tool. It's **which tool it picks when several would "work," and
whether that pick reflects your values and your stakes.**

Look back at Assistant A and Assistant B. Both used legitimate research tools
competently. Neither one matched the choice to *your* speed-versus-rigor
trade-off for *this* high-stakes task. That decision, in almost every deployed
system today, gets made three ways at once: **implicitly** (a side effect of
training and prompting), **inconsistently** (the same request resolves
differently across sessions), and **invisibly** (nobody is scoring whether the
choice matched what you wanted).

> **The shift to internalize:** the gap is no longer "nobody does this." It's
> "everybody does this implicitly, and nobody measures it." Closing that gap,
> making value-conditioned action selection explicit, consistent, and
> measurable, is the entire course.

## Two terms that make this sharp

**State space** is everything the system understands about the situation: the
ask, the domain, the stakes, the time pressure, your stated preferences. Modern
models are genuinely strong here. They parse intent, catch urgency, know a
medical question wants credible sources.

**Action space** is the set of things the system can *do*, which tool, in what
order, at what cost, and the rule it uses to choose. This is where consistency
falls apart, because nothing in the system reliably maps *understood values*
onto *tool choice*.

Here's the claim, and it's narrow on purpose: **most alignment failures you
actually feel are action-space failures, not state-space failures.** The model
knew what you wanted. It just had no systematic, value-conditioned way to decide
what to do about it.

(Module 1B puts hard, cited numbers on what this costs, from enterprise failure
rates to lawyers getting sanctioned for AI-invented citations, so we won't repeat
them here. This lesson is about the mechanism.)

### One interaction, up close

Take a medical query: *"What are the side effects of the new arthritis
medication?"*

The understanding is rock solid and stable: medical domain, factual request,
high stakes, probably a patient or caregiver, credible sources required. A
current model nails all of that, every single time.

The *behavior* is where the variance lives. One session leans on general web
results (fast, uneven). Another on a medical database (slow, rigorous). Another
on recent regulatory news (timely, narrow). Each is a defensible choice. None is
"random" or dumb. But the system has no explicit rule that says *for a high-stakes
medical query from a non-expert, weight source credibility way above speed*, and
apply that rule the same way every time. So the trade-off gets resolved
implicitly, which means differently, depending on things you can't see and the
system can't report.

## "But doesn't post-training already fix this?"

Good objection. It deserves a precise answer, not a dodge.

Post-training has done an enormous amount. RLHF aligned models to broad human
preferences for being helpful and harmless. DPO made preference tuning simpler
and more stable. GRPO and RLVR pushed models to strong performance on tasks with
checkable answers. These methods are *why* tool use works at all, and you'll
learn every one of them properly in Modules 3 and 4.

Here's the precise limit, though. **Post-training optimizes a model against
*aggregate* preferences and *verifiable* tasks. Not against *your* users'
*specific* value trade-offs at the moment of decision.** The reward signal that
shaped the weights was "what do labelers, on average, prefer?" or "did the test
pass?" It was never "for this deployment, in this context, does speed or rigor
win?" A post-trained model carries a strong, general instinct for good behavior.
It does not carry *your* deployment's value weights, and it has no mechanism to
apply them consistently and tell you it did.

That leftover, the distance between a good general instinct and an explicit,
per-deployment, measurable value policy, is exactly what this course is about.
Post-training hands you a capable agent. Making it *reliably serve a particular
set of values* is a design problem you layer on top.

## What the fix looks like

Instead of hoping a good general instinct produces the right trade-off every
time, we make the rule explicit. In notation:

$$\pi(\text{tool} \mid \text{context}, \text{values})$$

Read it slowly, because the whole idea is in the symbols:

- $\pi$ is the **policy**, the rule that outputs which tool to use (or a
  distribution over tools).
- $\text{tool}$ is the action you're picking from the available set.
- $\text{context}$ is the observable situation: query, domain, stakes, time
  pressure, budget.
- $\text{values}$ is an explicit handle on what the user actually cares about,
  how much they weight accuracy versus speed versus cost *right now*.

The thing that separates this from the implicit status quo is that
$\text{values}$ is a **named input**, not an accident of training. When values
are an explicit argument to the policy, the choice becomes **predictable** (same
context and values, same choice), **inspectable** (you can ask *why this tool*),
and **measurable** (you can score whether it matched). Modules 2 through 4 build
that policy three ways, as a hand-designed scoring function (Module 2), as a
learned RL policy (Module 3), and through the post-training algorithms that shape
the underlying model (Module 4). Same goal every time: drag value-conditioned
action selection out of "implicit" and into "explicit."

## Our working example: a 12-tool research agent

One example runs through the whole course, so the abstractions always have
something concrete to grab. It's a research agent with a realistic kit:

academic search, web search, news aggregation, fact-checking, sentiment
analysis, citation analysis, summarization, cross-referencing, bias detection,
confidence assessment, human consultation, and synthesis.

The job is the one this lesson named: pick the right combination of these for
each query, *conditioned on* the user's values (accuracy vs speed vs cost), the
context (stakes, domain, urgency), the resource limits (budget, rate limits,
time), and the quality bar (does this need peer review, or is a solid general
answer fine?). And do it consistently and measurably, not as a lucky draw from a
good instinct.

## The takeaways

- The gap between two equally capable assistants is an **action-space** gap, not
  a **state-space** gap. They understood you identically and chose differently.
- In 2026, **tool invocation is reliable** (function calling, MCP, RL-trained tool
  use). The unsolved part is **value-conditioned tool *selection***, which stays
  implicit, inconsistent, and unmeasured almost everywhere.
- **Post-training gives you a strong general instinct and verifiable-task skill.
  It does not encode a specific deployment's value trade-offs at decision time.**
  That leftover is what this course closes.
- The fix makes values an **explicit input** to the policy
  $\pi(\text{tool} \mid \text{context}, \text{values})$, turning action selection
  from invisible and unmeasurable into predictable, inspectable, and measurable.

## Next

Module 1B puts real, cited numbers on what leaving this gap open actually costs,
plus the talent shortage of people who can close it. Then Module 1C gives you the
mathematical framework that turns the implicit decisions you just met into ones
you can write down, defend, and measure.
