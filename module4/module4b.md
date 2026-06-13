# Module 4b: Monte Carlo Methods, Markov Chains, and Temporal-Difference Learning

## Learning Objectives

By the end of this lesson, you will be able to:

- Write down a Markov chain's transition matrix, compute multi-step transition
  probabilities, and state the stationary-distribution condition.
- Implement first-visit Monte Carlo value estimation and explain its
  bias-variance profile.
- Write the TD(0) update from memory, define the TD error, and explain
  bootstrapping precisely.
- State the ε-greedy and UCB action-selection rules.
- Describe the four phases of MCTS and the UCT formula, and place MCTS honestly
  in the 2026 LLM landscape.
- Recognize that GRPO's group sampling is Monte Carlo baseline estimation.

---

## Introduction: Conversations Are Stochastic Processes

Module 4a established the bridge from MDP theory to real agent architecture.
This lesson supplies the *learning foundations* the rest of Module 4 builds on:
how an agent estimates value from sampled experience when it does not know the
environment's dynamics in advance. Almost everything downstream — DQN's TD
target in 4c, PPO's advantage estimates in 4d, GRPO's group baseline — is a
variation on the two ideas introduced here: **Monte Carlo** (learn from
complete sampled returns) and **temporal-difference** (learn from one-step
bootstrapped estimates).

We keep the previous version of this lesson's good instinct — that a
multi-turn interaction is naturally a stochastic process — but where that
version only gestured at the algorithms, this one writes them down.

## Markov Chains: The Substrate

A **Markov chain** is a sequence of states where the next state depends only on
the current one. With a finite state set $\mathcal{S} = \{1, \dots, n\}$, the
chain is fully described by a **transition matrix** $P$, where $P_{ij} =
P(s_{t+1} = j \mid s_t = i)$. Each row sums to 1.

### A concrete conversation chain

Model a support conversation with four states: **Q** (user asks a question),
**C** (agent asks a clarifying question), **A** (agent answers), **F**
(user follows up). Order them $[\text{Q}, \text{C}, \text{A}, \text{F}]$. A
plausible transition matrix:

$$
P = \begin{bmatrix}
0.0 & 0.4 & 0.6 & 0.0 \\
0.0 & 0.0 & 1.0 & 0.0 \\
0.0 & 0.0 & 0.0 & 1.0 \\
0.5 & 0.0 & 0.5 & 0.0
\end{bmatrix}
$$

Reading row Q: from a question, the agent clarifies 40% of the time and answers
directly 60%. From C it always answers. From A the user always follows up. From
F the user either asks a fresh question (50%) or the agent answers the follow-up
directly (50%).

**Two-step transition.** What is the probability of being in state A two steps
after a question? Sum over the intermediate state $k$: $(P^2)_{\text{Q},\text{A}}
= \sum_k P_{\text{Q},k} P_{k,\text{A}}$. The only intermediate states reachable
from Q are C and A. From C, $P_{\text{C},\text{A}} = 1.0$; from A,
$P_{\text{A},\text{A}} = 0.0$. So

$$
(P^2)_{\text{Q},\text{A}} = P_{\text{Q},\text{C}} \cdot P_{\text{C},\text{A}} + P_{\text{Q},\text{A}} \cdot P_{\text{A},\text{A}} = 0.4 \times 1.0 + 0.6 \times 0.0 = 0.4.
$$

(The 0.6 mass that went straight to A at step 1 has already moved on to F by
step 2 — a small reminder that "where you are now" and "where you were" are
different questions.)

**Stationary distribution.** A distribution $\pi$ over states is **stationary**
if it is unchanged by one step of the chain:

$$
\pi P = \pi, \qquad \sum_i \pi_i = 1.
$$

This balance equation says the long-run fraction of time spent in each state is
self-consistent under the dynamics. For an ergodic chain it is unique and
describes where the process spends its time regardless of where it started.
(For the matrix above the chain is not ergodic as written — A and F form a tight
cycle — which is itself a useful diagnostic: the *structure* of $P$ tells you
whether a conversation design has absorbing traps or healthy mixing before you
ever run it.)

Why this matters for agents: transition matrices let you reason about a
conversation or tool-use *design* analytically — expected length, probability of
reaching a resolution state, where loops form — before collecting any data.

## Monte Carlo Value Estimation

Now suppose we do not know $P$ or the reward function, only that we can *run
episodes* and observe what happens. **Monte Carlo (MC)** methods estimate the
value of a state as the average **return** observed after visiting it.

The return from time $t$ is the discounted sum of future rewards:

$$
G_t = r_{t+1} + \gamma r_{t+2} + \gamma^2 r_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k r_{t+1+k}.
$$

**First-visit MC** estimates $V(s)$ as the mean return following the *first*
time $s$ is visited in each episode:

```
Initialize V(s) ← 0, N(s) ← 0 for all s
for each episode:
    generate a full episode s_0, a_0, r_1, s_1, ..., s_T following the policy
    G ← 0
    for t = T-1 down to 0:
        G ← r_{t+1} + γ·G                      # accumulate return backward
        if s_t not visited earlier in this episode:   # first-visit check
            N(s_t) ← N(s_t) + 1
            V(s_t) ← V(s_t) + (1/N(s_t)) · (G − V(s_t))   # incremental mean
```

The last line is the **incremental mean update**: $V(s) \leftarrow V(s) +
\frac{1}{N(s)}(G - V(s))$, which is exactly a running average that never has to
store past returns.

**Strengths.** MC is **unbiased** — it averages actual sampled returns, making
no assumption about the dynamics — and it **does not bootstrap**, so estimation
error in one state never contaminates another.

**Costs.** MC has **high variance** (a single return sums many random rewards)
and requires **complete episodes** — you cannot update until the episode ends,
which is useless for never-ending interaction and slow when episodes are long.
This is the same sparse, episode-level signal problem Module 4a flagged.

## Temporal-Difference Learning: TD(0)

**Temporal-difference (TD)** learning fixes MC's two costs by updating from a
single step instead of a full return. The **TD(0)** update is:

$$
V(s_t) \leftarrow V(s_t) + \alpha \big[\, \underbrace{r_{t+1} + \gamma V(s_{t+1}) - V(s_t)}_{\delta_t,\ \text{the TD error}} \,\big]
$$

The bracketed quantity $\delta_t$ is the **TD error**: the difference between the
current estimate $V(s_t)$ and a *bootstrapped* one-step target $r_{t+1} + \gamma
V(s_{t+1})$. "Bootstrapping" means the target is built partly from the agent's
own current estimate $V(s_{t+1})$ rather than from observed returns alone — we
update a guess toward a slightly-better guess.

**Bias-variance trade-off vs MC.** Because the TD target uses only one real
reward plus an estimate, it has **much lower variance** than a full MC return —
but it is **biased** while $V$ is still wrong, since the bootstrap inherits that
error. MC is the opposite: unbiased, high variance. In practice TD usually
learns faster, and it works **online**, updating every step without waiting for
the episode to end. This is why TD, not MC, is the engine inside DQN (4c) and
the value critic in PPO (4d).

**n-step returns** interpolate between the two: use $n$ real rewards then
bootstrap, $G_t^{(n)} = r_{t+1} + \cdots + \gamma^{n-1} r_{t+n} + \gamma^n
V(s_{t+n})$. Small $n$ is TD-like (low variance, more bias); large $n$ is MC-like.

**TD(λ)** averages all n-step returns geometrically with weight $(1-\lambda)
\lambda^{n-1}$, giving a single knob $\lambda \in [0, 1]$ that slides from TD(0)
($\lambda = 0$) to MC ($\lambda = 1$), often implemented efficiently with
eligibility traces.

## Exploration vs Exploitation

To estimate values you must *visit* states and *try* actions — but a greedy
agent that always exploits its current estimates may never discover better
options. Two standard rules manage this trade-off.

**ε-greedy.** With probability $1 - \epsilon$ take the action of highest
estimated value; with probability $\epsilon$ take a uniformly random action:

$$
a_t = \begin{cases} \arg\max_a Q(s_t, a) & \text{with prob. } 1 - \epsilon \\ \text{uniform random action} & \text{with prob. } \epsilon. \end{cases}
$$

Annealing $\epsilon$ from high to low explores early, exploits late.

**UCB (Upper Confidence Bound).** Rather than explore uniformly at random, prefer
actions that are either high-value *or* under-explored:

$$
a_t = \arg\max_a \left[ Q(s_t, a) + c \sqrt{\frac{\ln t}{N(s_t, a)}} \right],
$$

where $N(s_t, a)$ counts how often $a$ has been tried in $s_t$ and $c$ tunes the
exploration bonus. The square-root term shrinks as an action is tried more,
focusing exploration on genuine uncertainty.

## Monte Carlo Tree Search

**Monte Carlo Tree Search (MCTS)** combines tree search with MC sampling to plan
when a model (or simulator) of the dynamics is available. It repeats four phases:

1. **Selection** — from the root, descend the existing tree by repeatedly
   choosing children via a bandit rule (UCT, below) until reaching a node with
   unexpanded actions.
2. **Expansion** — add one new child node for an untried action.
3. **Simulation (rollout)** — from the new node, play out to a terminal state
   using a fast default policy, observing the outcome.
4. **Backpropagation** — propagate the rollout's result back up the visited path,
   updating each node's visit count and value estimate.

Selection uses the **UCT** rule (Kocsis & Szepesvári, 2006) — UCB applied to the
tree:

$$
a = \arg\max_a \left[ \bar{Q}(s, a) + c \sqrt{\frac{\ln N(s)}{N(s, a)}} \right],
$$

with $\bar{Q}(s,a)$ the mean rollout value and $N$ the visit counts. AlphaGo
(Silver et al., 2016) famously paired MCTS with deep value and policy networks to
defeat a top human Go player.

**Where MCTS sits in 2026 LLM work.** MCTS over *reasoning traces* — treating
partial chains of thought as tree nodes and search to find better solution paths
— is an active research direction, and it informs how people think about
inference-time search. But it is **not** a standard component of production
post-training: the dominant reasoning-model recipe (4d's GRPO/RLVR) trains the
model to produce good traces directly rather than searching a tree at inference.
Know MCTS as a powerful planning tool and a research thread, not as the default
production answer.

## Where Monte Carlo Lives in Modern Post-Training

Do not file MC away as "the slow, high-variance one you skip for TD." Its core
idea — *estimate value by averaging sampled outcomes* — is exactly what **GRPO**
(Group Relative Policy Optimization; Shao et al., 2024) does. For a given prompt,
GRPO samples a **group** of $k$ responses, scores each, and uses the **group mean
reward as the baseline** against which each response's advantage is measured.
That group mean is a Monte Carlo estimate of the prompt's expected reward,
computed fresh from samples rather than from a learned value network. It is why
GRPO can drop PPO's critic entirely: the sampled group *is* the baseline. More
broadly, rollout-based evaluation — sample many completions, average a metric —
is Monte Carlo estimation, and it pervades how modern models are both trained and
measured.

A note on the course notebooks: `RL_Alignment_Part2` implements a simplified
**tabular Q-learning** loop. Q-learning is itself a TD method (its update is the
one-step bootstrap with a $\max$ over next actions); it gets its full treatment,
and its scaling to neural networks, in Module 4c.

## Key Takeaways

- A **Markov chain** is a transition matrix $P$; $P^2$ gives two-step
  probabilities, and the **balance equation** $\pi P = \pi$ defines the
  stationary distribution. You can analyze a conversation design analytically
  before running it.
- **Monte Carlo** estimates value by averaging complete sampled returns —
  unbiased, no bootstrapping, but high variance and episode-completion-bound.
- **TD(0)** bootstraps from a one-step target with **TD error** $\delta_t =
  r_{t+1} + \gamma V(s_{t+1}) - V(s_t)$ — lower variance, biased while learning,
  works online. **n-step** and **TD(λ)** interpolate MC↔TD.
- **ε-greedy** and **UCB** manage exploration; **MCTS** plans via UCT-guided
  rollouts and is a research thread for LLM reasoning, not a production default.
- **GRPO's group mean baseline is Monte Carlo estimation** — the same idea,
  alive at the center of modern post-training.

## Looking Ahead — Module 4c

TD learning paired with a lookup table works only when states are few. **Module
4c** replaces the table with a neural network — $Q_\theta(s,a)$ — and confronts
the instability that creates (the "deadly triad"), arriving at Deep Q-Networks.
The TD error you met here is the exact quantity DQN regresses on.

## References

- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
- Kocsis, L., & Szepesvári, C. (2006). Bandit based Monte-Carlo planning. *ECML*.
- Silver, D., et al. (2016). Mastering the game of Go with deep neural networks and tree search. *Nature*, 529, 484–489.
- Shao, Z., et al. (2024). DeepSeekMath: Pushing the limits of mathematical reasoning in open language models (GRPO). arXiv:2402.03300.
