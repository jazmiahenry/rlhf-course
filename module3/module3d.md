# Module 3D: Trajectories and Sequential Alignment

## Introduction: Beyond Single Actions to Sequential Behavior

While individual actions represent isolated decisions, real-world AI behavior consists of **trajectories**, sequences of actions taken over time that collectively accomplish complex goals. For alignment, this temporal dimension is crucial: an agent might make individually reasonable decisions that collectively violate user values, or it might need to make seemingly suboptimal choices early in a sequence to enable better long-term alignment.

**The Core Challenge**: Alignment isn't just about making good individual decisions, it's about maintaining values consistently throughout entire sequences of actions, even when those sequences involve uncertainty, changing contexts, and evolving constraints.

This lesson explores the mathematical foundations of trajectory-level alignment and how agents can learn to maintain human values across complex, multi-step behaviors.

## Mathematical Framework for Trajectories

### Basic Trajectory Structure

A **trajectory** τ represents a complete sequence of state-action-reward transitions:

$$\tau = (s_0, a_0, r_0, s_1, a_1, r_1, ..., s_T, a_T, r_T)$$

The trajectory distribution under policy π is:

$$P(\tau | \pi) = P(s_0) \prod_{t=0}^{T} \pi(a_t | s_t) P(s_{t+1} | s_t, a_t) P(r_t | s_t, a_t)$$

### Trajectory Length and Complexity

**Fixed-Length Trajectories**: 
$$|\tau| = T \text{ (constant)}$$
Useful for well-defined tasks with clear endpoints.

**Variable-Length Trajectories**:
$$|\tau| = T(\tau) \text{ where } T(\tau) \text{ depends on task completion}$$
More realistic for open-ended research tasks.

**Infinite-Horizon Trajectories**:
$$|\tau| = \infty \text{ with discounting } \gamma^t$$
Appropriate for ongoing interaction scenarios.

### Multi-Tool Agent Trajectory Example

For our research assistant, a typical trajectory might look like:

$$\tau_{\text{research}} = ($$
$$s_0: \text{user query received}, a_0: \text{web\_search}, r_0: +2,$$
$$s_1: \text{initial results}, a_1: \text{fact\_check}, r_1: +3,$$
$$s_2: \text{verified info}, a_2: \text{academic\_search}, r_2: +4,$$
$$s_3: \text{comprehensive data}, a_3: \text{synthesis}, r_3: +5$$
$$)$$

Each step builds on previous actions, with states and available actions evolving based on accumulated information and resource consumption.

## Alignment Along Trajectories

### Traditional vs. Trajectory-Level Optimization

**Traditional RL** optimizes expected cumulative reward:
$$J(\pi) = \mathbb{E}_{\tau \sim \pi}[\sum_{t=0}^T \gamma^t r_t]$$

**Alignment-Aware Trajectory Optimization** must ensure value consistency throughout the sequence:
$$J_{\text{aligned}}(\pi) = \mathbb{E}_{\tau \sim \pi}[\sum_{t=0}^T \gamma^t (r_t + \lambda \cdot A(s_t, a_t, \tau_{0:t}))]$$

where $A(s_t, a_t, \tau_{0:t})$ is an alignment bonus/penalty that considers not just the current action, but its relationship to the entire trajectory so far.

### Trajectory-Level Alignment Constraints

Some alignment properties can only be evaluated at the trajectory level:

#### 1. Consistency Constraint
Actions should reflect consistent values throughout the trajectory:
$$\forall t_1, t_2 \in \{0, ..., T\}, \text{if } \text{similar}(s_{t_1}, s_{t_2}) \text{ then } \text{consistent\_values}(a_{t_1}, a_{t_2})$$

**Example**: If the agent prioritizes accuracy over speed early in the trajectory when stakes are high, it should maintain this prioritization in similar high-stakes situations later.

#### 2. Progressive Refinement Constraint
Information quality should generally improve over time:
$$\mathbb{E}[\text{information\_quality}(s_{t+1})] \geq \mathbb{E}[\text{information\_quality}(s_t)] - \epsilon_{\text{exploration}}$$

The exploration term allows for temporary quality decreases during information gathering.

#### 3. Resource Rationality Constraint
Resource usage should be justified by information gain:
$$\sum_{t=0}^T \text{cost}(a_t) \leq \text{budget} \text{ and } \frac{\text{final\_quality} - \text{initial\_quality}}{\sum_{t=0}^T \text{cost}(a_t)} \geq \text{efficiency\_threshold}$$

#### 4. Value Preservation Constraint
Core user values should be respected throughout:
$$\forall t, \sum_{v \in V} u_v \cdot \text{value\_satisfaction}_v(s_t, a_t, \tau_{0:t}) \geq \tau_{\text{min}}$$

where $u_v$ is the user's weight on value $v$ (accuracy, speed, cost, safety),
$\text{value\_satisfaction}_v$ measures how well the action at time $t$ serves
that value given the trajectory so far, and $\tau_{\text{min}}$ is the minimum
acceptable per-step value satisfaction. Unlike a cumulative reward bound, this
is a *per-step floor*: a trajectory cannot "bank" early aligned behavior to
license a later violation. This is the constraint that rules out the
individually-reasonable-but-collectively-misaligned sequences described in the
introduction.

## Evaluating Trajectories in Practice

The constraints above are defined over complete trajectories, but an agent
must act step by step. Two practical mechanisms bridge that gap.

### Discounted Cumulative Alignment Score

Rather than waiting until a trajectory ends to judge it, we maintain a running
alignment score that weights recent steps most heavily:

$$A_{\text{cumulative}}(\tau_{0:t}) = \sum_{k=0}^{t} \alpha^{k} \cdot \text{alignment\_score}(s_k, a_k)$$

with $\alpha \in (0, 1)$ (the companion notebook uses $\alpha = 0.95$). This
gives the agent, and us, as evaluators, a continuously updated signal: a dip
in the running score flags a potential alignment violation while there is
still time to correct course within the same trajectory.

### Operationalizing the Consistency Check

The consistency constraint quantifies over *all pairs* of timesteps, which
sounds expensive but is straightforward for the trajectory lengths research
agents produce (T ≤ 10 in our running example). Two states count as "similar"
when they share a query type and are close in complexity and urgency:

$$\text{similar}(s_i, s_j) \iff \text{type}(s_i) = \text{type}(s_j) \;\wedge\; |c_i - c_j| < \theta \;\wedge\; |u_i - u_j| < \theta$$

and two actions count as value-consistent when they prioritize the same
top-weighted user value. This is deliberately conservative: it will sometimes
flag benign pairs (a false positive costs a review), but it will not silently
pass an agent that treats two near-identical high-stakes situations with
different levels of care.

These two mechanisms are implemented as `compute_alignment_score()` and
`check_consistency_constraint()` in the `AlignedTrajectory` class of the
companion notebook (`RL_Alignment_Part2_Trajectories_and_Curriculum.ipynb`), run them on the demo trajectories there to see both a passing and a failing
case.

## Credit Assignment Across Trajectories

Trajectory-level alignment raises the classic RL credit-assignment problem in
a new form: when a trajectory violates a constraint, *which step* was the
mistake?

- **Consistency violations** localize naturally, the check identifies the
  specific pair $(t_1, t_2)$ that diverged, and the later step is usually the
  correction target.
- **Progressive-refinement violations** point to the step where expected
  information quality dropped beyond the exploration allowance, often a tool
  choice that traded quality for speed when the context didn't call for it.
- **Resource-rationality violations** are global: total spend exceeded the
  information gained. Attribution requires comparing each step's marginal
  cost against its marginal quality contribution, which the per-step `info`
  dictionary in the notebook's `TrajectoryStep` records for exactly this
  purpose.

This is why we log alignment scores and outcome quality *per step* rather
than per trajectory: post-hoc attribution is impossible if only the endpoint
is recorded.

## Key Takeaways

1. **Alignment is a property of sequences, not just decisions.** Individually
   reasonable actions can compose into a misaligned trajectory; trajectory-
   level constraints are what catch this.
2. **Four constraint families cover most failures**: consistency across
   similar states, progressive refinement of information quality, resource
   rationality, and a per-step value-satisfaction floor.
3. **Running scores beat endpoint evaluation.** A discounted cumulative
   alignment score turns trajectory-level alignment into a signal the agent
   can act on mid-sequence.
4. **Log per-step, attribute post-hoc.** Credit assignment for constraint
   violations requires per-step alignment and quality records.

## Looking Ahead

Trajectories are the unit over which *policies* are evaluated. The next
lesson (Module 3E) turns to the policies themselves: how value weights enter
the action-selection rule, how preference information can be learned rather
than specified, and how risk-sensitive objectives (CVaR) reshape what an
"optimal" trajectory means. The curriculum lesson (Module 3F) then asks how
to *train* an agent so that the constraints in this lesson hold from the
start, rather than being bolted on afterward.