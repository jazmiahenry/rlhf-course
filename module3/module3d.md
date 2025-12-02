# Module 3D: Trajectories and Sequential Alignment

## Introduction: Beyond Single Actions to Sequential Behavior

While individual actions represent isolated decisions, real-world AI behavior consists of **trajectories**—sequences of actions taken over time that collectively accomplish complex goals. For alignment, this temporal dimension is crucial: an agent might make individually reasonable decisions that collectively violate user values, or it might need to make seemingly suboptimal choices early in a sequence to enable better long-term alignment.

**The Core Challenge**: Alignment isn't just about making good individual decisions—it's about maintaining values consistently throughout entire sequences of actions, even when those sequences involve uncertainty, changing contexts, and evolving constraints.

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

where $u_v