# Module 1A: The Hidden Problem — State Space vs Action Space

## Learning Objectives

By the end of this lesson you will be able to:

- Distinguish what modern AI systems do well (understanding the situation) from
  what they do inconsistently (choosing what to do under a user's values).
- State the state-space / action-space framing precisely and apply it to a real
  interaction.
- Explain accurately what post-training (RLHF → DPO → GRPO/RLVR) does and does
  not give you for value-conditioned decisions.
- Read the policy notation $\pi(\text{tool} \mid \text{context}, \text{values})$
  and say what each term means.

## Introduction: The Tale of Two Assistants

Ask two equally capable AI assistants the same thing: *"Help me research vaccine
effectiveness for a public health presentation."*

**Assistant A** returns a quick web summary with mixed-quality sources in thirty
seconds. **Assistant B** pulls peer-reviewed studies but takes ten minutes and
blows past your deadline. Both parsed your request correctly. Both had the same
tools available. Both would call themselves "helpful" and "accurate." Yet they
made opposite trade-offs — and neither *asked* which one you needed.

That gap is the subject of this entire course. It is not a gap in
*understanding*; both assistants understood the query. It is a gap in *deciding*
— specifically, in deciding under the user's values. This lesson names the gap
precisely and corrects a common misconception about whether modern AI has already
solved it.

## What Has Changed Since This Problem Was First Posed

It is worth being honest about the state of the art, because the easy version of
this story is now wrong. A few years ago you could claim that AI systems chose
tools "randomly" — that tool *invocation* itself was unreliable. That is no longer
true, and a course that pretended otherwise would be teaching you to fight a
battle that's already won.

In 2026, the mechanical act of using a tool is **structured and reliable**.
Models emit typed function calls validated against JSON schemas; the Model
Context Protocol (MCP) gives a standard way to expose tools; and frontier labs
explicitly *train* tool use with reinforcement learning, so models have genuinely
learned when a calculator, a search, or a code execution is appropriate for a
*task*. If you ask a current model to "compute the 47th Fibonacci number," it
will reliably reach for code execution rather than guess. Tool invocation is a
solved-enough problem.

So where is the gap? It moved. The remaining problem is not *whether* the model
can call a tool, but **which tool it selects when several would "work," and
whether that selection reflects your specific values and stakes.** Assistant A and
Assistant B both used legitimate research tools competently. What neither did was
condition the choice on *your* trade-off between speed and rigor for *this*
high-stakes task. That selection is, in almost every deployed system today, made
**implicitly** (as a side effect of training and prompting), **inconsistently**
(the same request can resolve differently across sessions), and **unmeasurably**
(no one is scoring whether the choice matched the user's values). The gap is no
longer "nobody does this." It is "everyone does this implicitly, and nobody
measures it." Closing that gap — making value-conditioned action selection
explicit, consistent, and measurable — is what this course teaches.

## The Framing: State Space vs Action Space

Two technical terms make the distinction sharp.

The **state space** is everything the system understands about the situation:
what the user asked, the domain, the stakes, the time pressure, the user's stated
preferences. Modern models are genuinely strong here. They parse intent, detect
urgency cues, recognize that a medical question demands credible sources, and pick
up explicit preferences when you state them.

The **action space** is the set of things the system can *do* about it — which
tool to use, in what sequence, at what cost — and the rule it uses to choose. This
is where consistency breaks down. The same well-understood query can yield
different action choices across sessions, because nothing in the system
systematically maps *understood values* onto *tool selection*.

The course's claim is narrow and, we think, correct: **most alignment failures
that users actually experience are action-space failures, not state-space
failures.** The model knew what you wanted. It just didn't have a systematic,
value-conditioned way to decide what to do about it. (Module 1B documents the
real, cited costs of this — from enterprise project-failure rates to the legal
profession's repeated, sanctioned encounters with AI-fabricated citations — so we
won't repeat those numbers here; the point of this lesson is the mechanism, not
the price tag.)

### One interaction, made concrete

Take a medical query: *"What are the side effects of the new arthritis
medication?"*

The state-space understanding is excellent and stable: medical domain, factual
request, high stakes, user likely a patient or caregiver, credible sources
required. A current model gets all of this right, every time.

The action-space behavior is where variance lives. One session leans on general
web results (fast, uneven quality); another on a medical database (slow, rigorous);
another on recent news about a regulatory decision (timely, narrow). Each is a
*defensible* choice — none is "random" or incompetent. But the system has no
explicit rule that says *for a high-stakes medical query from a non-expert,
weight source credibility far above speed* — and apply that rule the same way
every time. The trade-off gets resolved implicitly, and so it gets resolved
differently depending on factors the user can't see and the system can't report.

## Doesn't Post-Training Already Solve This?

This is the sharp objection, and it deserves a precise answer. Hasn't RLHF —
and its successors — already taught models to make good choices?

Post-training has accomplished an enormous amount. Reinforcement Learning from
Human Feedback (RLHF) aligned models to general human preferences for
helpfulness and harmlessness. Direct Preference Optimization (DPO) made
preference-tuning simpler and more stable. Group Relative Policy Optimization
(GRPO) and Reinforcement Learning from Verifiable Rewards (RLVR) pushed models to
strong performance on tasks with checkable answers — math, code, structured
reasoning. These methods are why tool use is reliable at all, and this course
teaches every one of them properly in Modules 3 and 4.

But here is the precise limit: **post-training optimizes a model against
*aggregate* preferences and *verifiable* tasks, not against *your* users'
*specific* value trade-offs at decision time.** The reward signal that shaped the
weights was "what do labelers, on average, prefer?" or "did the test pass?" —
not "for this deployment's users, in this context, does speed or rigor win?" A
post-trained model carries a strong, general prior about good behavior. It does
not carry *your* deployment's value weights, and it has no mechanism to apply them
consistently and report on having done so. That residual — the difference between
a good general prior and an explicit, per-deployment, measurable value policy — is
exactly the territory of this course. Post-training gives you a capable agent;
making it *consistently serve a particular set of values* is a design problem
layered on top.

## The Shape of the Solution

Instead of hoping a good general prior produces the right value trade-off every
time, we make the selection rule explicit. Notationally, we want a policy:

$$
\pi(\text{tool} \mid \text{context}, \text{values})
$$

Read this carefully, because the terms are the whole idea:

- $\pi$ is the **policy** — the rule that outputs a choice (or a distribution over
  choices) of which tool to use.
- $\text{tool}$ is the action being selected from the available set.
- $\text{context}$ is the observable situation — query, domain, stakes, time
  pressure, resource budget.
- $\text{values}$ is an explicit representation of the user's priorities — how
  much they weight accuracy vs. speed vs. cost in *this* context.

The thing that distinguishes this from the implicit status quo is that
$\text{values}$ is a **named input**, not an emergent side effect. When values are
an explicit argument to the policy, the choice becomes **predictable** (same
context and values → same choice), **inspectable** (you can ask *why* this tool),
and **measurable** (you can score whether the choice matched the values). Modules
2 through 4 build this policy three ways — as a hand-designed scoring function
(Module 2), as a learned RL policy (Module 3), and via the post-training
algorithms that shape the underlying model (Module 4) — but the goal is constant:
move value-conditioned action selection from implicit to explicit.

## The Working Example: A Multi-Tool Research Agent

Throughout the course we build one running example — a **research agent** with a
realistic toolset, so the abstractions always have something concrete to bind to.
The agent has access to twelve tools:

- Academic database search
- Web search
- News aggregation
- Fact-checking services
- Sentiment analysis
- Citation analysis
- Summarization
- Cross-referencing
- Bias detection
- Confidence assessment
- Human consultation
- Synthesis and integration

The challenge is exactly the one this lesson named: choose the right combination
of these tools for each query, *conditioned on* the user's values (accuracy vs.
speed vs. cost), the context (stakes, domain, urgency), the resource limits
(budget, rate limits, time), and the quality bar (does this need peer-reviewed
sources, or is a general answer fine?) — and do it consistently and measurably,
not as a lucky draw from a good prior.

## Key Takeaways

- The gap between two equally capable assistants is an **action-space** gap, not
  a **state-space** gap: they understood the request identically and chose
  differently.
- In 2026, **tool invocation is reliable** (function calling, MCP, RL-trained tool
  use). The unsolved part is **value-conditioned tool *selection***, which remains
  implicit, inconsistent, and unmeasured in almost all deployed systems.
- **Post-training (RLHF → DPO → GRPO/RLVR)** gives a strong *general* prior and
  *verifiable-task* competence. It does **not** encode a particular deployment's
  value trade-offs at decision time — that residual is what this course closes.
- The solution makes values an **explicit input** to the policy
  $\pi(\text{tool} \mid \text{context}, \text{values})$, turning action selection
  from implicit and unmeasurable into predictable, inspectable, and measurable.

## Looking Ahead

Module 1B documents the real, cited costs of leaving this gap unaddressed — and
the historic shortage of people who can close it. Module 1C then introduces the
explicit mathematical framework — tool characterization, value weights, and
utility maximization — that turns the implicit decisions described here into
decisions you can write down, justify, and measure.
