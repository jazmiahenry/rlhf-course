# Module 4b: Monte Carlo, Markov Chains, and TD Learning

> **What you'll get out of this:** the two foundational ways an agent learns value
> from experience when it doesn't know the rules in advance. Almost everything in the
> rest of Module 4 is a variation on these two ideas.

## The question under everything

Here's the situation that defines RL: you don't know the environment's dynamics or
reward function up front. You can only *act* and *see what happens*. So how do you
estimate how good a state or action is?

Two answers run through this whole module. **Monte Carlo:** learn from complete
sampled returns. **Temporal-difference:** learn from one-step bootstrapped estimates.
DQN's target (4c), PPO's advantages (4d), GRPO's baseline, they're all variations on
these. The old version of this lesson described them in prose with zero equations.
This one writes them down, because "an algorithm" you can't write is an algorithm you
don't have.

## What you'll be able to do

- Write a Markov chain's transition matrix and compute multi-step probabilities.
- Implement first-visit Monte Carlo and TD(0), and explain their bias-variance trade.
- State the exploration rules (ε-greedy, UCB) and the four phases of MCTS.
- Recognize that GRPO's group sampling *is* Monte Carlo estimation.

## Markov chains: the substrate

A **Markov chain** is a sequence of states where the next depends only on the current.
With states $\mathcal{S} = \{1, \dots, n\}$, it's fully described by a transition
matrix $P$ where $P_{ij} = P(s_{t+1} = j \mid s_t = i)$ and each row sums to 1.

Make it concrete with a support conversation: **Q** (question), **C** (clarify), **A**
(answer), **F** (follow-up), ordered $[\text{Q}, \text{C}, \text{A}, \text{F}]$:

$$P = \begin{bmatrix} 0.0 & 0.4 & 0.6 & 0.0 \\ 0.0 & 0.0 & 1.0 & 0.0 \\ 0.0 & 0.0 & 0.0 & 1.0 \\ 0.5 & 0.0 & 0.5 & 0.0 \end{bmatrix}$$

Row Q says: from a question, the agent clarifies 40% of the time, answers directly
60%. **Two-step question:** what's the probability of being in A two steps after a
question? Sum over the middle state: $(P^2)_{\text{Q},\text{A}} = P_{\text{Q,C}}
P_{\text{C,A}} + P_{\text{Q,A}} P_{\text{A,A}} = 0.4(1.0) + 0.6(0.0) = 0.4$.

A **stationary distribution** $\pi$ is unchanged by a step: $\pi P = \pi$ with
$\sum_i \pi_i = 1$. It's the long-run fraction of time spent in each state. The point
for agents: a transition matrix lets you analyze a conversation or tool-use *design*,
expected length, probability of resolution, where loops form, before you collect a
single episode.

## Monte Carlo: average the returns

Now suppose you don't know $P$, only that you can run episodes. **Monte Carlo**
estimates a state's value as the average **return** seen after visiting it. The return
from time $t$ is the discounted future reward:

$$G_t = r_{t+1} + \gamma r_{t+2} + \gamma^2 r_{t+3} + \cdots$$

**First-visit MC** averages the return following the first visit to each state per
episode:

```
Initialize V(s) ← 0, N(s) ← 0 for all s
for each episode:
    generate a full episode following the policy
    G ← 0
    for t = T-1 down to 0:
        G ← r_{t+1} + γ·G                        # accumulate return backward
        if s_t not visited earlier this episode: # first-visit
            N(s_t) ← N(s_t) + 1
            V(s_t) ← V(s_t) + (1/N(s_t))·(G − V(s_t))   # running mean
```

That last line is a running average that never stores past returns. MC is
**unbiased** (it averages real returns, assuming nothing about dynamics) and **doesn't
bootstrap** (one state's error can't infect another). The costs: **high variance** (a
return sums many random rewards) and it needs **complete episodes** (no updating until
the end). That episode-completion requirement is the sparse-reward pain from Module
3D.

## TD(0): bootstrap from one step

**Temporal-difference** learning fixes both costs by updating from a single step:

$$V(s_t) \leftarrow V(s_t) + \alpha\big[\underbrace{r_{t+1} + \gamma V(s_{t+1}) - V(s_t)}_{\delta_t,\ \text{the TD error}}\big]$$

The bracket $\delta_t$ is the **TD error**: the gap between the current estimate and a
*bootstrapped* one-step target $r_{t+1} + \gamma V(s_{t+1})$. "Bootstrapping" means the
target is built partly from your own current guess $V(s_{t+1})$, you nudge a guess
toward a slightly-better guess.

The trade versus MC: TD uses one real reward plus an estimate, so it's **much lower
variance** but **biased** while $V$ is still wrong. MC is the opposite. In practice TD
usually learns faster and works **online**, updating every step. That's why TD, not
MC, is the engine inside DQN (4c) and the critic in PPO (4d). **n-step returns**
interpolate (use $n$ real rewards then bootstrap); **TD(λ)** geometrically averages all
n-step returns with a single knob $\lambda \in [0,1]$ that slides from TD(0) to MC.

## Exploration: ε-greedy and UCB

To estimate values you must *visit* states and *try* actions, but a greedy agent never
discovers anything. Two standard rules:

**ε-greedy.** With probability $1-\epsilon$ take the best action; with probability
$\epsilon$ take a random one. Anneal $\epsilon$ down over time: explore early, exploit
late.

**UCB.** Prefer actions that are high-value *or* under-explored:

$$a_t = \arg\max_a \left[Q(s_t, a) + c\sqrt{\frac{\ln t}{N(s_t, a)}}\right]$$

The square-root term shrinks as an action gets tried, so exploration focuses on
genuine uncertainty rather than random flailing.

## Monte Carlo Tree Search

**MCTS** combines tree search with MC rollouts when you have a model or simulator. Four
phases, repeated: **selection** (descend the tree by a bandit rule), **expansion** (add
a child for an untried action), **simulation** (roll out to the end with a fast
policy), **backpropagation** (push the result back up, updating visit counts and
values). Selection uses **UCT** (Kocsis & Szepesvári, 2006), UCB applied to the tree.
AlphaGo (Silver et al., 2016) paired MCTS with deep networks to beat a top human at Go.

Where does MCTS sit in 2026 LLM work? Search over *reasoning traces* is an active
research direction, but it is **not** standard in production post-training, the
dominant reasoning recipe trains the model to produce good traces directly (4d's
GRPO/RLVR) rather than searching a tree at inference. Know MCTS as a powerful tool and
a research thread, not the default.

## Where Monte Carlo lives in modern training

Don't file MC away as "the slow one." Its core idea, *estimate value by averaging
samples*, is exactly what **GRPO** does (Shao et al., 2024). For a prompt, GRPO samples
a group of $k$ responses and uses the **group mean reward as the baseline**. That mean
is a Monte Carlo estimate, computed fresh from samples instead of a learned value
network, which is why GRPO can drop PPO's critic entirely. The sampled group *is* the
baseline. (The course notebook `RL_Alignment_Part2` implements tabular Q-learning,
itself a TD method; Q-learning gets its full treatment in 4c.)

## The takeaways

- A **Markov chain** is a transition matrix $P$; $P^2$ gives two-step probabilities,
  $\pi P = \pi$ defines the stationary distribution. You can analyze a design before
  running it.
- **Monte Carlo** averages complete returns: unbiased, no bootstrapping, but
  high-variance and episode-bound.
- **TD(0)** bootstraps with error $\delta_t = r_{t+1} + \gamma V(s_{t+1}) - V(s_t)$:
  lower variance, biased while learning, works online. **n-step** and **TD(λ)**
  interpolate.
- **ε-greedy** and **UCB** manage exploration; **MCTS** plans via UCT rollouts (a
  research thread for LLMs, not a production default).
- **GRPO's group-mean baseline is Monte Carlo estimation**, alive at the center of
  modern post-training.

## Think about it

1. In the conversation chain, what's the probability of being back at Q two steps after
   a question? (Compute $(P^2)_{\text{Q,Q}}$.)
2. MC is unbiased but high-variance; TD is biased but low-variance. For a task with very
   long episodes and a reward only at the end, which would you reach for, and why?

## Next

Module 4c scales TD learning from a lookup table to a neural network, $Q_\theta(s,a)$,
and confronts the instability that creates: Deep Q-Networks.

## References

- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
- Kocsis, L., & Szepesvári, C. (2006). Bandit based Monte-Carlo planning. *ECML*.
- Silver, D., et al. (2016). Mastering the game of Go with deep neural networks and tree search. *Nature*, 529, 484-489.
- Shao, Z., et al. (2024). DeepSeekMath (GRPO). arXiv:2402.03300.
