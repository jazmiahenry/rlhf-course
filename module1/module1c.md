# Module 1C: The Mathematical Solution Framework

> **What you'll get out of this:** the shift from "hope the AI chooses well" to
> "make it an optimization problem," and the three pieces you need to actually do
> that.

## Where we are

Two things are nailed down so far. From 1A: AI understands what you want (state
space) but has no systematic way to choose what to do (action space). From 1B:
that failure shows up as a >80% project-failure rate and a stack of documented
costs.

Now the turn. We stop hoping and start optimizing.

## The one idea: alignment as optimization

Here's the whole move in two lines.

- **Today:** hope the model makes good choices through training and prompting.
- **Instead:** *ensure* good choices by solving an optimization problem.

Treat action selection not as an emergent property you cross your fingers for,
but as a thing you compute. Formally:

```
Given:
  tools         A = {a₁, ..., aₙ}
  user values   V = {v₁, ..., vₘ}
  context       C
  tool props    P(a)

Find:
  a* = argmax_a  Σᵢ wᵢ · vᵢ(a, C)   subject to constraints(a, C)
```

That's it. "Pick the action that maximizes weighted user value, subject to your
constraints." Everything else in this lesson is the three pieces that make that
line real.

## The three pillars

### Pillar 1: say what each tool actually does

Right now, tools are "understood" implicitly through training patterns. You can't
inspect that, and you can't optimize over it. So make it explicit: give every
tool a **property vector**.

```
P(tool) = [accuracy, speed, cost_efficiency, coverage, reliability, safety, specialization]

P(academic_search) = [0.95, 0.3, 0.4, 0.6, 0.9, 0.9, 0.8]
P(web_search)       = [0.6, 0.9, 0.8, 0.9, 0.6, 0.5, 0.3]
P(fact_check)       = [0.9, 0.4, 0.5, 0.3, 0.95, 0.9, 0.9]
```

Notice every dimension points the same way: higher is better. (That's why we
encode cost as cost-*efficiency*, not raw cost.) Now the weighted sum below
rewards good properties consistently, tools are comparable, and "best tool for
this context" becomes a calculation instead of a vibe.

### Pillar 2: turn the user's values into numbers

User values live in natural language, which the decision process can't act on.
So convert preferences into **weights**:

```
W(user) = [w_accuracy, w_speed, w_cost, w_coverage, w_reliability, w_safety, w_specialization]
```

And because values shift with context, let context bump them:

```
W(user, context) = W_base(user) + context_modifiers(context)
```

A medical query, for example, should crank up accuracy and safety:

```
W_base          = [0.8, 0.5, 0.6, 0.7, 0.8, 0.9, 0.7]
context_medical = [+0.15, -0.1, 0, 0, +0.1, +0.05, +0.2]
W_final         = [0.95, 0.4, 0.6, 0.7, 0.9, 0.95, 0.9]
```

Now the same framework behaves differently in different situations, *on purpose*,
with a clear rationale you can point at.

### Pillar 3: actually optimize

With tools and values both numeric, selection is a dot product:

```
utility(tool) = Wᵀ × P(tool)
best_tool      = argmax_tool utility(tool)
```

Add the real-world limits as constraints:

```
maximize:    Wᵀ × P(tool)
subject to:  cost(tool) ≤ budget
             time(tool) ≤ deadline
             safety(tool) ≥ safety_threshold
```

And for multi-tool plans, optimize the sequence while accounting for how tools
interact (and stay consistent):

```
maximize:    Σᵢ utility(toolᵢ) − interaction_costs(sequence)
subject to:  Σᵢ cost(toolᵢ) ≤ total_budget
             sequence_time   ≤ deadline
             consistency(sequence) ≥ threshold
```

## Three flavors of this, easy to hard

You'll meet all three later. Here's the map so you know where you're headed.

**Static scoring (Module 2).** Mathematical formulas that score each tool. No
training, works instantly, fully transparent and tunable. Best for getting
consistency *now* and for domains with a clear value hierarchy.

**Dynamic learning (Modules 3A-3D).** Machine learning that optimizes tool
selection over time while honoring alignment constraints. Adapts, handles subtle
trade-offs, scales. Best for production systems in complex domains.

**Curriculum learning (Modules 3E-3F).** Progressive training that builds robust
alignment from simple cases up to adversarial ones. Best for high-stakes, long-
horizon deployments. (You'll see in the RL capstone exactly when this pays off,
and when it doesn't.)

## Why the math actually helps

Five payoffs, fast:

1. **Predictable.** `Wᵀ × P(tool)` gives the same answer for the same inputs.
   "Be helpful and accurate" doesn't.
2. **Optimal.** You select the tool that maximizes utility under constraints,
   instead of a reasonable-looking random pick.
3. **Transparent.** Every choice comes with a receipt: *"academic_search won
   because your accuracy weight 0.9 × its accuracy 0.95 = 0.855, highest total
   utility, and it satisfies your budget."*
4. **Tunable.** Want it faster? Raise `w_speed`. More cautious? Tighten the safety
   threshold. New domain? Adjust the tool vectors. No retraining.
5. **Composable.** Tool selection, value integration, constraints, and learning
   are separate modules you can build incrementally and recombine.

## Two objections worth answering

**"This is too rigid."** It isn't. Context-dependent weights let one framework
behave differently across situations while staying consistent *within* each one.
Mathematical optimization runs the least-rigid industries on earth: trading,
logistics, resource allocation.

**"Users can't state their values precisely."** They don't have to. You infer
values from behavior (preference learning), start from sensible defaults and
adjust, or read them from context. If a user keeps choosing fast over thorough,
`w_speed` goes up on its own. (Module 3E builds exactly this.)

## What to measure (and an honest note on it)

People love to slap precise targets on this ("cut variance to under 5%, hit 2x
utility"). We won't invent numbers, because the right targets depend entirely on
your baseline. What *is* stable is the set of dimensions worth measuring:

- **Consistency:** variance in tool choice for identical queries. Lower is better.
- **Optimality:** how close you get to the best possible choice under constraints.
- **Transparency:** the fraction of decisions you can explain from the math.
- **Adaptability:** how fast the system tracks a real change in preferences.
- **Trust and productivity:** adoption, and how much prompt-engineering whack-a-mole
  you stop doing.

> **The rule, restated:** if you can't measure the improvement, you can't claim
> it. Define these before you write code.

## The economics, without made-up ROI

Costs and returns vary wildly by org, so here's the *structure* instead of fake
figures.

**Where the cost goes:** explicit frameworks front-load work, value elicitation,
tool characterization, scoring design, that prompt-engineering defers. Then the
prompt approach pays for that deferral repeatedly, in maintenance, incidents, and
re-prompting.

**Where the return comes from:** every failure shape in 1B is a *recurring*
operating cost, human verification of inconsistent output, liability exposure,
abandoned deployments (RAND's >80% failure rate is mostly sunk development cost).
A framework that makes decisions consistent and measurable attacks those recurring
costs head-on.

**The early-mover edge:** reliability is a real differentiator while most
deployments stay unmeasured (MIT: ~95% of pilots show no measurable impact). It
buys access to high-stakes customers and supports premium pricing for predictable,
explainable AI.

## The takeaways

- The core move: treat action selection as **optimization**,
  `a* = argmax Σ wᵢ vᵢ`, not as something to hope for.
- It takes **three pillars**: explicit tool properties, numeric value weights, and
  an optimizer with constraints.
- It comes in three flavors, **static → dynamic → curriculum**, easy to hard,
  mapped to Modules 2, 3, and beyond.
- The payoffs are **predictable, optimal, transparent, tunable, composable**
  behavior.
- **Measure before you claim.** Define consistency, optimality, transparency, and
  adaptability up front. No invented targets.

## Next

Module 1D turns this framework into a roadmap, lays out the implementation path,
and introduces the 12-tool research agent we build on for the rest of the course.
