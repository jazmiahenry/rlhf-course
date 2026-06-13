# Module 4c: Deep Q-Networks, Value-Based Deep RL

## Learning Objectives

By the end of this lesson, you will be able to:

- Derive the tabular Q-learning update and explain why lookup tables break down
  in large or continuous state spaces.
- Define a parameterized action-value function $Q_\theta(s, a)$ and articulate
  the **deadly triad** that makes naive value-based deep RL unstable.
- Explain the two stabilizing ideas in DQN, **experience replay** and **target
  networks**, and state the DQN loss precisely.
- Recall the one-line mathematical change behind each major DQN variant: Double
  DQN, Dueling networks, Prioritized Experience Replay, and how Rainbow combines
  them.
- Describe honestly where value-based deep RL is and is not used in 2026,
  including why LLM agent orchestration is *not* a DQN, and where value functions
  genuinely do appear in LLM training.
- Identify the canonical hands-on path (Gymnasium CartPole-v1 → ALE/Atari) for
  implementing DQN yourself.

---

## 1. From Tabular Q-Learning to Function Approximation

In Module 3 you met the value-based view of reinforcement learning, and the
**Module 3 notebook (Part 2)** had you implement a *simplified tabular
Q-learning* loop. That exercise was deliberately scoped down: it stored one
number per state-action pair and updated them one at a time. This lesson is
where Q-learning is treated fully, where we confront what happens when the
table no longer fits, and what we must do to make learning work anyway.

Recall the **Q-learning** update rule (Watkins, 1989; Watkins & Dayan, 1992).
Given a transition $(s, a, r, s')$, we nudge the estimated action-value toward a
bootstrapped target:

$$
Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma \max_{a'} Q(s', a') - Q(s, a) \right]
$$

The bracketed term is the **temporal-difference (TD) error**: the gap between
our current estimate $Q(s,a)$ and a one-step-better estimate
$r + \gamma \max_{a'} Q(s', a')$. The $\max$ over next actions is what makes
Q-learning **off-policy**, it learns about the greedy policy regardless of the
(possibly exploratory) policy that actually generated the data. Watkins & Dayan
proved that, under suitable conditions (every state-action pair visited
infinitely often, a decaying learning rate), tabular Q-learning converges to the
optimal action-value function $Q^*$.

### Why tables fail at scale

The convergence guarantee assumes one independent estimate per state-action
pair. That assumption is fatal in practice:

- **Atari from pixels.** A single frame is roughly $210 \times 160$ pixels with
  128 colors. The number of distinct screens dwarfs the number of atoms in the
  observable universe. You cannot tabulate it.
- **No generalization.** A table treats $s$ and a near-identical $s'$ as wholly
  unrelated entries. It learns nothing about one state by visiting a similar one, so it must visit *every* state many times. That is impossible in large
  spaces.

The fix is **function approximation**: replace the table with a parameterized
function $Q_\theta(s, a)$, a neural network with weights $\theta$, that maps a
state to estimated values and *generalizes* across similar states. For Atari,
$Q_\theta$ is a convolutional network that takes a stack of recent frames and
outputs one value per discrete action.

### The deadly triad

Function approximation does not come for free. Sutton & Barto (2018) name three
ingredients that, when combined, can cause value estimates to diverge, the
**deadly triad**:

1. **Function approximation**, $Q_\theta$ shares parameters across states, so an
   update for one state perturbs estimates everywhere.
2. **Bootstrapping**, the update target $r + \gamma \max_{a'} Q_\theta(s', a')$
   is built from the network's own (wrong) estimates, not from ground-truth
   returns. Errors feed back into targets.
3. **Off-policy learning**, the data distribution (from an exploratory or
   replayed behavior policy) differs from the policy being evaluated, so the
   updates are not a well-behaved expectation under any fixed distribution.

Any one or two of these is usually safe. All three together can make $Q_\theta$
spiral away from $Q^*$. Q-learning with a neural network sits squarely in the
triad, which is exactly why a naive implementation diverges, and why DQN's
contribution was a set of tricks to tame it.

---

## 2. DQN: Making Deep Q-Learning Work

Mnih et al. (2015), published in *Nature*, introduced the **Deep Q-Network
(DQN)**: the first method to learn control policies directly from
high-dimensional pixel input across a wide range of tasks. DQN minimizes a
squared TD-error loss:

$$
L(\theta) = \mathbb{E}_{(s, a, r, s') \sim \mathcal{D}} \left[ \left( r + \gamma \max_{a'} Q_{\theta^-}(s', a') - Q_\theta(s, a) \right)^2 \right]
$$

Two design choices, both aimed directly at the deadly triad, make this loss
trainable.

### Experience replay

Online RL data is highly **correlated**: consecutive frames look almost
identical, and the agent's current policy biases which transitions it sees.
Training a neural network on such a stream violates the near-i.i.d. assumption
that stochastic gradient descent relies on, and lets recent experience dominate
and overwrite earlier learning.

DQN stores transitions $(s, a, r, s')$ in a **replay buffer** $\mathcal{D}$ and
trains on **random minibatches** sampled from it. This breaks temporal
correlation (samples in a batch come from many different episodes and time
points), reuses each transition many times (sample efficiency), and smooths the
data distribution so gradients are better behaved. The expectation in $L(\theta)$
is taken over $\mathcal{D}$ precisely because of replay.

### Target networks

The TD target $r + \gamma \max_{a'} Q_\theta(s', a')$ depends on the very
parameters $\theta$ we are updating. Chasing a target that moves every gradient
step is like trying to hit a target that jumps each time you aim, it couples the
prediction and the target and amplifies the bootstrapping instability.

DQN introduces a separate **target network** $Q_{\theta^-}$ with parameters
$\theta^-$ that are held *fixed* for many steps and only periodically copied from
$\theta$. The target $r + \gamma \max_{a'} Q_{\theta^-}(s', a')$ therefore stays
stationary between updates, giving the online network a stable objective to
regress toward. This is the $\theta^-$ that appears in the loss above.

### The training loop

```
Initialize replay buffer D, online net Q_θ, target net Q_θ⁻ ← Q_θ
for each environment step:
    with probability ε: a ← random action          # explore
    otherwise:          a ← argmax_a Q_θ(s, a)      # exploit
    execute a, observe r, s'
    store (s, a, r, s') in D
    sample minibatch {(s, a, r, s')} from D
    y ← r                            if s' terminal
    y ← r + γ max_a' Q_θ⁻(s', a')    otherwise
    take a gradient step on (y − Q_θ(s, a))²  w.r.t. θ
    every C steps: θ⁻ ← θ            # refresh target network
```

### The Atari result, stated accurately

Trained on raw pixels with the *same* network architecture and hyperparameters
across games, DQN reached **human-level or above performance on many of the
Atari 2600 games** in the Arcade Learning Environment, learning end-to-end from
screen and score alone. It performed strongly on reactive games and notably
poorly on those requiring long-horizon planning or exploration (e.g.,
*Montezuma's Revenge*). The headline was generality from a single recipe, not a
claim of superhuman play everywhere.

---

## 3. The Big Variants

DQN spawned a family of improvements. Each is, at heart, a one-line change to the
target or the sampling.

### Double DQN (van Hasselt et al., 2016)

The $\max$ operator in the DQN target both **selects** the best next action and
**evaluates** it using the same network. Because $Q$ estimates are noisy, taking
the max of noisy values systematically **overestimates** action-values. Double
DQN decouples the two roles: the online network *selects* the action, the target
network *evaluates* it.

$$
y = r + \gamma \, Q_{\theta^-}\!\left(s', \; \arg\max_{a'} Q_\theta(s', a')\right)
$$

This corrected target reduces overoptimistic value estimates and improves both
stability and final performance.

### Dueling networks (Wang et al., 2016)

Often the value of *being in a state* matters far more than the differences
between actions there. Dueling architectures split the network into two streams, a state-value $V(s)$ and an advantage $A(s, a)$, and recombine them. A naive sum
$Q = V + A$ is **unidentifiable** (you can add a constant to $V$ and subtract it
from $A$ without changing $Q$), so the advantage is mean-subtracted to pin it
down:

$$
Q_\theta(s, a) = V_\theta(s) + \left( A_\theta(s, a) - \frac{1}{|\mathcal{A}|} \sum_{a'} A_\theta(s, a') \right)
$$

The shared $V(s)$ stream learns state value efficiently even for actions rarely
taken, which helps wherever action choice is often irrelevant.

### Prioritized Experience Replay (Schaul et al., 2016)

Uniform replay wastes effort on transitions the network already predicts well.
**Prioritized Experience Replay (PER)** samples transitions in proportion to the
magnitude of their TD error, the "most surprising" experiences, with priority
$p_i \propto |\delta_i|^\omega$, where $\delta_i$ is the TD error and $\omega$
controls how aggressively we prioritize. Non-uniform sampling biases the
expectation, so PER corrects it with **importance-sampling weights**:

$$
P(i) = \frac{p_i^{\omega}}{\sum_j p_j^{\omega}}, \qquad w_i = \left( \frac{1}{N \cdot P(i)} \right)^{\beta}
$$

The weight $w_i$ (annealed via $\beta \to 1$) down-weights frequently sampled
high-priority transitions so the gradient remains an unbiased estimate. PER
consistently speeds learning by focusing compute where the error is largest.

### Rainbow (Hessel et al., 2018)

**Rainbow** asked the obvious question: are these improvements complementary? It
combined six independent extensions, Double Q-learning, dueling networks,
prioritized replay, multi-step (n-step) returns, distributional RL (C51, which
models the full return distribution rather than its mean), and noisy networks for
exploration, into a single agent. The combination outperformed every component
in isolation on the Atari benchmark, and ablations showed prioritized replay and
multi-step returns contributed the most. Rainbow is the standard reference for
"value-based deep RL, tuned."

---

## 4. Where Value-Based RL Is Actually Used (2026, Honestly)

Value-based deep RL is a mature, useful tool, in a specific shape of problem:
**discrete action spaces, a clear scalar reward, and cheap, abundant environment
interaction.** Where those hold, it shines:

- **Games and simulated control**, Atari, board and video games, and other
  simulators remain the canonical proving ground and a live research setting.
- **Recommendation candidate ranking (research)**, framing next-item or
  candidate selection as a discrete-action value problem appears in the
  literature, though production recommenders lean heavily on supervised and
  bandit methods.
- **Robotics primitives**, discrete skill or option selection (which low-level
  controller to invoke) is a natural fit, even when the underlying control is
  continuous.
- **Resource scheduling (research)**, job, cache, and compute scheduling cast as
  discrete sequential decisions are an active research area.

Be precise about the boundary: many of these are *research* framings, not
deployed defaults. Value-based methods also struggle with large or continuous
action spaces (the $\max_{a'}$ becomes intractable), which is where the
**policy-based** methods of the next lesson take over.

> ### Sidebar, Why your LLM orchestrator is not a DQN
>
> It is tempting to imagine that a system coordinating multiple LLM "agents"
> learns, by reinforcement, a $Q$-function over which agent to call next. **It
> almost certainly does not.** Production agent coordination in 2026 is **LLM
> planning plus engineered scaffolding**: a capable model proposes and sequences
> steps in natural language, and deterministic code (routers, tool dispatch,
> retries, guardrails, state machines) executes and constrains those steps. No
> DQN is trained to schedule agents. The mismatch is structural:
>
> - **Action space.** A DQN needs a small, fixed, discrete action set so
>   $\max_{a'} Q(s', a')$ is well-defined. "What should the orchestrator do next"
>   is an open-ended space of natural-language plans and tool calls, not
>   enumerable.
> - **Episode structure.** DQN assumes many resettable episodes with a clean
>   scalar reward per transition. Agent workflows are heterogeneous, often
>   single-shot, and lack a dense, well-defined reward per step.
> - **Data economics.** DQN needs millions of cheap environment interactions in a
>   replay buffer. Each "step" of an LLM orchestrator is an expensive model call
>   against a non-resettable, real-world-coupled environment. You cannot collect
>   DQN-scale experience.
>
> When you read that an orchestrator "learns to route," that intelligence is the
> LLM's in-context reasoning and the engineering around it, not a value network.
> Treat any claim of a deployed DQN scheduling LLM agents with deep skepticism.

### Where value functions *do* appear in LLM-land

Value-based *thinking* is not absent from LLM training, it just does not look
like a DQN choosing agents. The clearest place is the **critic in PPO-based
RLHF**: a value head $V_\phi(s)$ estimates the expected reward-to-go of a partial
generation and is used to compute advantages that reduce gradient variance.
**Process reward models (PRMs)**, which score intermediate reasoning steps, can
be read as value-*like* signals over partial trajectories. Both are estimating
"how good is this state," which is exactly the value-function idea, applied to
text generation rather than to discrete game actions. We pick this thread up in
the next lesson.

---

## 5. Hands-On: Implement DQN Yourself

Reading the equations is not the same as watching a TD error shrink. The
canonical exercise path:

1. **Gymnasium `CartPole-v1`.** The right first target: a 4-dimensional
   continuous state, two discrete actions (push left/right), and a reward of +1
   per step the pole stays up. A DQN here is a small multilayer perceptron, a
   few dozen lines around an `env.step()` loop, a replay buffer (a `deque`), a
   target network refreshed every $C$ steps, and an $\epsilon$-greedy policy with
   $\epsilon$ annealed over training. CartPole trains in minutes on a CPU, so you
   can iterate on the bugs the deadly triad will hand you.

2. **`ALE/Atari` environments.** Once CartPole is solved, graduate to pixel input
   through the Arcade Learning Environment (Gymnasium environments such as
   `ALE/Pong-v5`). Now you add the convolutional encoder, frame stacking, and
   preprocessing from Mnih et al. (2015). This is substantially more
   compute-hungry (GPU-hours) but reproduces the original DQN setting.

3. **Reference implementation.** Before scaling up, compare against
   **`stable-baselines3`**, whose `DQN` class is a well-tested, readable
   implementation. Run their DQN on CartPole, then diff its behavior against
   yours to localize discrepancies. Use it as an oracle, not a substitute, implement the core loop yourself first.

A good milestone: get your from-scratch CartPole DQN to reach the maximum episode
return reliably, then add **Double DQN** (one line in the target computation) and
confirm the value estimates stop drifting upward.

---

## Key Takeaways

- **Tabular Q-learning** (Watkins & Dayan, 1992) converges to $Q^*$ but cannot
  scale; **function approximation** with $Q_\theta$ generalizes across states at
  the cost of stability.
- The **deadly triad**, function approximation + bootstrapping + off-policy
  learning, is why naive deep Q-learning diverges (Sutton & Barto, 2018).
- **DQN** (Mnih et al., 2015) tamed the triad with **experience replay**
  (decorrelate data) and a **target network** (stop chasing a moving target),
  reaching human-level play on many Atari games from pixels with one recipe.
- The major variants are each a small, principled fix: **Double DQN** removes
  max-bias, **Dueling** factors $Q = V + A$ with mean-subtraction, **PER** samples
  by TD error with importance-sampling correction, **Rainbow** combines them.
- Value-based RL fits **discrete-action, cheap-interaction** problems. It is
  **not** how LLM agent orchestration works, that is LLM planning plus
  engineering. Value functions *do* show up in LLM training as the **PPO critic**
  and **process reward models**.

---

## Looking Ahead, Module 4d

Value-based methods learn *how good* each action is and act greedily with respect
to those values. That breaks down when the action space is large or continuous, the $\max_{a'}$ has no tractable solution, and it gives you no direct,
differentiable handle on the policy itself. **Module 4d** crosses over to
**policy-based** methods: we parameterize the policy $\pi_\theta(a \mid s)$
directly, optimize it with the policy-gradient theorem, and arrive at **PPO**, the algorithm at the heart of RLHF. The value function does not disappear; it
returns as the **critic** that estimates advantages. Keep the value-vs-policy
distinction in mind: 4c taught you to estimate value, 4d will teach you to
optimize policy, and RLHF uses both at once.

---

## References

- Watkins, C. J. C. H. (1989). *Learning from Delayed Rewards.* PhD thesis, University of Cambridge.
- Watkins, C. J. C. H., & Dayan, P. (1992). Q-learning. *Machine Learning*, 8(3-4), 279-292.
- Mnih, V., et al. (2015). Human-level control through deep reinforcement learning. *Nature*, 518(7540), 529-533.
- van Hasselt, H., Guez, A., & Silver, D. (2016). Deep reinforcement learning with double Q-learning. *AAAI*.
- Wang, Z., et al. (2016). Dueling network architectures for deep reinforcement learning. *ICML*.
- Schaul, T., Quan, J., Antonoglou, I., & Silver, D. (2016). Prioritized experience replay. *ICLR*.
- Hessel, M., et al. (2018). Rainbow: Combining improvements in deep reinforcement learning. *AAAI*.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
