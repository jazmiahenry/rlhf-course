# Module 3B: Action Spaces and Value Trade-offs

> **What you'll get out of this:** why the menu of actions you hand an agent
> quietly decides which aligned behaviors are even possible, before any learning
> happens.

## The ceiling you set without realizing it

Module 3A was about what the agent can *see*. This one's about what it can *do*. And
here's the under-appreciated part: **the action space sets a ceiling on alignment
that no algorithm can break through.** Give an agent a bad menu and the smartest
policy in the world can only pick badly. Give it a good menu and aligned behavior
becomes learnable, even natural.

So before reaching for clever learning, design the menu.

## What you'll be able to do

- Describe an action as a feature vector, not just a label.
- Check an action space for the properties that make alignment learnable (value
  separability, a safety valve, Pareto coverage).
- Reason about composing actions into multi-step behaviors.

## Actions are not labels, they're vectors

A textbook action space is a set of labels: $A = \{a_1, \dots, a_n\}$. For a real
agent that's not enough, because two "search" actions can have wildly different
costs, reliability, and value profiles. So characterize each action by what it
actually does:

$$\mathbf{f}(a_i) = [c_i,\ t_i,\ p_i,\ \mathbf{s}_i,\ \mathbf{v}_i]$$

cost, time, success probability, a strength vector (accuracy, coverage, depth), and
a **value-alignment vector** (how well it serves each human value). For our research
agent:

| tool | cost | time | reliability | strengths (acc, speed, cov) | values (truth, eff, comp) |
|---|---|---|---|---|---|
| academic search | 3 | 2 | 0.85 | [0.9, 0.6, 0.8] | [0.9, 0.3, 0.7] |
| web search | 1 | 1 | 0.65 | [0.6, 0.9, 0.9] | [0.5, 0.9, 0.8] |
| human consult | 8 | 5 | 0.95 | [0.95, 0.2, 0.7] | [0.95, 0.1, 0.6] |

Each tool *is* a different value trade-off. That's the raw material alignment is
built from.

## Three properties that make an action space alignment-friendly

You don't need a dozen invented metrics here. You need three real properties.

**1. Value separability.** For any two competing values, there should be different
actions that each maximize one of them. In our menu, human consultation maxes
accuracy, web search maxes speed and cost-efficiency. Without separability, every
action is roughly the same trade-off, and the agent *cannot* learn to act
differently in different contexts, there's nothing to choose between.

**2. A safety valve.** There must always be at least one action that holds the
safety line, no matter the situation:

$$\exists\, a_{\text{safe}} \in A : P(\text{violation} \mid a_{\text{safe}}, s) < \epsilon,\ \forall s$$

Human consultation is ours. However weird or high-stakes things get, "escalate to a
human" keeps alignment recoverable. Design this in deliberately, it's what prevents
catastrophic failures in novel situations.

**3. Pareto coverage.** Lay the actions out on the speed-versus-accuracy plane and
you want the **Pareto frontier** well covered: web search (fast, rough), news
(balanced), academic (accurate, slow), human (most accurate, slowest). Formally,
each action should be optimal for *some* preference weighting:

$$\forall a \in A,\ \exists \mathbf{w} : a = \arg\max_{a'} \mathbf{w}^\top \mathbf{f}(a')$$

If an action is never the best choice for any user, it's dead weight. A rich
frontier means different users and contexts get appropriately different treatment.

> **The intuition in one line:** a narrow menu (`search`, `analyze`, `respond`)
> forces one-size-fits-all behavior. A rich menu spanning real trade-offs lets the
> agent be context-appropriate. The menu is the constraint.

## The menu changes with context

In the real world, not every action is always available:

$$A_t = A_{\text{base}} \cap A_{\text{available}}(s_t)$$

A tool drops out if it busts the budget, blows the deadline, or its service is
down. Which raises a sharp design requirement: **alignment has to stay reachable
even on a shrunken menu.** Concretely, keep the safety valve available exactly when
you need it most, when the stakes are high:

$$a_{\text{human}} \in A_t \text{ whenever } \text{stakes}(s_t) > \text{high threshold}$$

Don't let a resource constraint quietly remove the one action that keeps a
high-stakes decision safe.

## Building behaviors by composing actions

Single actions rarely finish a real task. So actions compose into sequences, and
the composition has structure worth knowing:

- **Additive:** coverage of `[a₁, a₂]` is roughly coverage(a₁) + coverage(a₂).
- **Submodular (diminishing returns):** stacking two accuracy tools doesn't double
  accuracy.
- **Synergistic:** some pairs beat the sum of their parts. Academic search *then*
  fact-check yields more credibility together than separately.

That last one is the interesting case for alignment: a "verify" behavior
(`academic → fact_check → cross_reference`) is more than its pieces, and you design
for it on purpose.

## How the policy meets the menu

A policy $\pi(a \mid s)$ is only as expressive as the menu allows: more actions
spanning real trade-offs means more nuanced, context-dependent behavior is
*possible*. A value-sensitive policy makes that explicit:

$$\pi_{\text{aligned}}(a \mid s) \propto \exp\!\Big(\beta\big[Q(s,a) + \alpha \textstyle\sum_i w_i(s)\, v_i(a)\big]\Big)$$

where $w_i(s)$ is the context-dependent weight on value $i$ and $v_i(a)$ is how well
action $a$ serves it. This only works *because* each action has well-defined value
properties, which only exists *because* you characterized the actions as vectors in
the first place. The whole chain depends on the menu.

## The four ways people wreck an action space

- **No value differentiation:** all actions are the same trade-off, so the agent
  can't learn value-sensitive behavior.
- **No safety action:** nothing is safe in every context, so there's no recourse in
  a novel high-risk situation.
- **Poor composition:** actions don't combine cleanly, so the agent is stuck doing
  simple single steps.
- **Context-blind availability:** the same menu always, so the agent wastes
  resources or violates constraints.

All four are failures to *design the menu before training the policy*.

## The takeaways

- The **action space sets the alignment ceiling.** Learning can only work with what
  the menu allows.
- Characterize actions as **feature vectors** (cost, reliability, value profile),
  not labels.
- Three real properties matter: **value separability**, a **safety valve**, and
  **Pareto coverage**. Skip the invented metrics.
- Keep alignment reachable on a **shrunken menu**, and keep the safety action
  available when stakes are high.
- Actions **compose** (additive, submodular, synergistic). Design the synergies.

## Think about it

1. List your domain's action menu as feature vectors. Is it value-separable? Where's
   the safety valve?
2. Plot two of your actions on the speed-accuracy plane. Is the frontier covered, or
   are there gaps where no action lives?

## Next

Module 3C brings in the learning signal: stochastic rewards, and why modeling
uncertainty (not just averages) is what makes aligned behavior robust.
