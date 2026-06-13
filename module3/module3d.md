# Module 3D: Trajectories and Sequential Alignment

> **What you'll get out of this:** why an agent can make a string of individually
> reasonable decisions that add up to a misaligned mess, and how to catch that
> before it finishes.

## The failure that hides in the gaps

Single decisions are the easy case. The dangerous case is the *sequence*. An agent
can take a dozen perfectly defensible actions that, stitched together, violate the
user's values. Each step looks fine. The trajectory is wrong.

So alignment isn't only about good individual choices. It's about holding values
*consistently across a whole sequence*, even when that sequence runs through
uncertainty, shifting context, and changing constraints. This lesson is about
reasoning over trajectories, not moments.

## What you'll be able to do

- Write a trajectory and the four constraint families that keep one aligned.
- Use a running alignment score to catch a drift mid-sequence instead of after.
- Think about credit assignment: when a trajectory goes bad, which step did it?

## A trajectory, formally

A **trajectory** $\tau$ is a full sequence of state, action, reward:

$$\tau = (s_0, a_0, r_0, s_1, a_1, r_1, \dots, s_T, a_T, r_T)$$

Standard RL optimizes expected cumulative reward, $J(\pi) = \mathbb{E}_\tau[\sum_t
\gamma^t r_t]$. Alignment-aware optimization needs more: value consistency
*throughout* the sequence, which we add as a bonus that depends on the whole
history so far:

$$J_{\text{aligned}}(\pi) = \mathbb{E}_\tau\Big[\sum_t \gamma^t \big(r_t + \lambda \cdot A(s_t, a_t, \tau_{0:t})\big)\Big]$$

That $\tau_{0:t}$ is the key: the alignment of an action depends on what came before
it.

## Four constraints that only make sense over a trajectory

Some properties simply can't be checked one action at a time.

**1. Consistency.** Similar situations should get similar values, across the
sequence. If the agent prioritized accuracy over speed in an early high-stakes
moment, it should do the same later in a similar moment. No flip-flopping.

**2. Progressive refinement.** Information quality should generally *climb* over the
sequence (with an allowance for exploration dips):

$$\mathbb{E}[\text{quality}(s_{t+1})] \geq \mathbb{E}[\text{quality}(s_t)] - \epsilon_{\text{explore}}$$

**3. Resource rationality.** Spend has to be justified by what you learned:

$$\sum_t \text{cost}(a_t) \leq \text{budget} \quad\text{and}\quad \frac{\text{quality gained}}{\text{cost spent}} \geq \text{threshold}$$

**4. Value preservation (the per-step floor).** Core values stay respected at
*every* step:

$$\forall t,\ \sum_{v} u_v \cdot \text{satisfaction}_v(s_t, a_t, \tau_{0:t}) \geq \tau_{\min}$$

That last one is the one that rules out the failure from the intro. It's a *floor*,
not a budget: the agent can't "bank" early aligned behavior to spend on a later
violation.

## Catch the drift while it's still happening

The constraints are defined over a *finished* trajectory, but the agent acts step by
step. So you maintain a **running alignment score** that weights recent steps most:

$$A_{\text{cumulative}}(\tau_{0:t}) = \sum_{k=0}^{t} \alpha^{k} \cdot \text{alignment}(s_k, a_k)$$

with $\alpha \in (0,1)$ (the companion notebook uses 0.95). Now a dip in the running
score flags a possible violation *while there's still time to fix it within the same
trajectory*. That's the difference between catching a problem and writing a
post-mortem.

The consistency check sounds expensive (it quantifies over all pairs of timesteps)
but it's cheap for the short trajectories research agents produce ($T \le 10$). Two
states are "similar" if they share a query type and are close in complexity and
urgency; two actions are "consistent" if they prioritize the same top value. It's
deliberately conservative: it'll occasionally flag a benign pair (a review costs
you little), but it won't silently wave through an agent that treats two nearly
identical high-stakes moments with different care.

## When a trajectory fails, which step broke it?

This is credit assignment, the classic RL problem, in a new outfit. When a
trajectory violates a constraint, *which step* was the mistake?

- **Consistency violations** localize naturally: the check names the specific pair
  $(t_1, t_2)$ that diverged, and the later step is usually the fix target.
- **Progressive-refinement violations** point to the step where expected quality
  dropped past the exploration allowance, often a tool that traded quality for
  speed when the context didn't call for it.
- **Resource-rationality violations** are global: total spend exceeded the value
  gained. Pinning the blame means comparing each step's marginal cost to its
  marginal contribution.

This is *why* you log alignment and quality **per step**, not per trajectory. If you
only record the endpoint, post-hoc attribution is impossible. The notebook's
`TrajectoryStep` keeps a per-step `info` dict for exactly this.

The `AlignedTrajectory` class in `RL_Alignment_Part2` implements the running score
and the consistency check. Run it on the demo trajectories there to watch both a
passing and a failing case.

## The takeaways

- Alignment is a property of **sequences**, not just decisions. Individually
  reasonable actions can compose into a misaligned trajectory.
- **Four constraint families** cover most failures: consistency across similar
  states, progressive refinement, resource rationality, and a per-step value floor.
- A **running alignment score** turns trajectory-level alignment into a signal the
  agent can act on mid-sequence, not after.
- **Log per step, attribute post-hoc.** Credit assignment for a violation needs
  per-step records.

## Think about it

1. Sketch a sequence of individually fine actions that adds up to a value violation
   in your domain. Which constraint would have caught it?
2. The per-step value floor versus a cumulative reward bound: why does the floor
   matter for alignment specifically?

## Next

Module 3E turns to the **policy** itself: how value weights enter the action rule,
how you can *learn* values from preferences, and how that exact machinery becomes
the reward model of RLHF.
