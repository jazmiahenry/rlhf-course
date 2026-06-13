# Module 1D: From Theory to Practice

> **What you'll get out of this:** the roadmap for the rest of the course, the
> 12-tool research agent we'll build on, and a worked example where the
> "obvious" tool choice turns out to be wrong.

## The bridge

You've got the problem (1A-1B) and the framework (1C). What you don't have yet is
a path: how do you go from "inconsistent AI" to "a system that actually works in
production"? Here it is, in three stages, easy to hard.

**Stage 1: Static mathematical scoring (Module 2).** Scoring functions that pick
optimal tools. Implementable in days to weeks. Instant consistency, transparency,
and optimality. No ML required.

**Stage 2: Dynamic learning systems (Modules 3A-3D).** ML with alignment
constraints. Weeks to months. Adapts, learns from experience, handles messy
context.

**Stage 3: Curriculum learning (Modules 3E-3F).** Progressive training for robust
alignment. Months. Survives adversarial conditions and complex multi-step
reasoning.

## Why start with the boring one

Here's the thing people miss: **you do not need machine learning to solve the
action-space problem.** Plain mathematical optimization gets you most of the way,
immediately. Start there.

What Stage 1 buys you on day one:

- **Consistency.** Same query plus same values equals the same choice, every time.
- **Optimality.** Provably the best tool given the preferences and constraints.
- **Transparency.** A clear, mathematical reason for every pick.
- **Tunability.** Change behavior by changing weights. No retraining.

Picture the COVID-vaccine query from 1A. Today it's a slot machine: web articles
one session, peer-reviewed studies the next, fact-check reports the third. With
static scoring and `{accuracy: 0.9, speed: 0.5, safety: 0.95}` in a medical
context, it resolves to academic search every single time. Same user, same
quality, no luck involved.

## Our working example: the 12-tool research agent

This is the system we build and rebuild across the whole course. Not a toy. A
thing you could actually ship. It has twelve tools:

**Gathering:** academic search (accurate, slow, pricey), web search (fast, broad,
uneven), news search (recent, some bias), fact-check (accurate on specific
claims, narrow).

**Analysis:** sentiment analysis, citation analysis, bias detection, confidence
assessment.

**Synthesis:** summarization, cross-reference, synthesis, human consultation
(most reliable, slowest, most expensive).

The whole challenge in one sentence: **given a query, a context, and a user's
values, which combination of these twelve do you use?** Today's answer is "hope
the model picks well." Ours is "optimize it."

### Same query, three different right answers

The point of value weights is that there's no single best plan. There's a best
plan *for this user, in this context*.

**A grad student researching a thesis.** Values: accuracy 0.95, speed 0.4, cost
0.6. Best path: academic search → citation analysis → cross-reference →
synthesis.

**A journalist on deadline.** Values: accuracy 0.7, speed 0.95, cost 0.8. Best
path: news search → fact-check → quick synthesis.

**A public health official mid-crisis.** Values: accuracy 0.9, speed 0.8, safety
0.95. Best path: academic search → fact-check → human consultation → synthesis.

Same tools. Three different optimal routes, because the weights differ. No prompt
tweak gives you that reliably. The math does.

## Do the math yourself

Here are two tools, by their property vectors (accuracy, speed, cost_efficiency):

```
Academic_Search = [0.95, 0.3, 0.4]
Web_Search       = [0.6, 0.9, 0.8]
```

And a user who's leaning fast and cheap:

```
Values W = [accuracy 0.8, speed 0.6, cost 0.4]
```

**Pause. Which tool wins?** Most people's gut says "academic, it's more
accurate." Compute it before you scroll. Utility is `Wᵀ × P(tool)`.

Academic: `0.8×0.95 + 0.6×0.3 + 0.4×0.4 = 0.76 + 0.18 + 0.16 = 1.10`

Web: `0.8×0.6 + 0.6×0.9 + 0.4×0.8 = 0.48 + 0.54 + 0.32 = 1.34`

**Web search wins, 1.34 to 1.10.** For *this* user, who weights speed and cost,
the "more accurate" tool is the wrong call, and the math says so plainly. That's
the whole pitch in one calculation: explicit values turn a gut argument into a
number you can check, defend, and change. Bump that user's accuracy weight to 0.95
and rerun it. Watch the winner flip.

## What's coming

**Module 2 (static scoring):** you'll implement immediate alignment wins, tool
characterization, value integration, multi-objective optimization, constraint
handling. You walk out with a working system that makes consistent, optimal tool
choices.

**Module 3 (dynamic learning):** you'll build agents that *improve* tool
selection over time while staying aligned, state-space design, action-space
optimization, learning with constraints, curriculum training.

**Module 4 (the algorithms):** the RL methods underneath it all, from Monte Carlo
and DQN through PPO, GRPO, DPO, and RLVR, plus a capstone that puts the whole
thing to an honest, measured test.

## The takeaways

- The path is **static → dynamic → curriculum**, and you should start with static.
  You don't need ML to fix the action-space problem.
- The **12-tool research agent** is our throughline. The job: pick the right
  combination for a given query, context, and value set.
- **The same query has different right answers** for different users. Value
  weights are what encode that, and a prompt can't do it reliably.
- The worked example makes it concrete: a speed-weighted user's *best* tool is the
  "less accurate" one, and the math shows exactly why. Explicit values turn
  arguments into checkable numbers.

## Next

Module 2 gets your hands dirty. You'll build the static scoring system end to end,
the first working piece of a mathematically aligned agent.
