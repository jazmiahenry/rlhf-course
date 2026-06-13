# Module 4c: Deep Q-Networks

> **What you'll get out of this:** how value-based RL scales from a lookup table to a
> neural network, why that nearly breaks it, and the tricks that made it work, plus an
> honest read on where DQN is and isn't used.

## When the table doesn't fit

Module 4b gave you TD learning with a lookup table. The notebook had you run tabular
Q-learning on a small problem. That's the warm-up. The real question is what happens
when the state space is too big to tabulate, like Atari from raw pixels, where the
number of distinct screens dwarfs the atoms in the universe.

You can't store a value for every state. You need a function that *generalizes* across
similar states. That's a neural network, and pointing one at Q-learning turns out to be
surprisingly hard to do without it blowing up. This lesson is that story.

## What you'll be able to do

- Derive the Q-learning update and explain why tables fail at scale.
- Name the **deadly triad** and why it destabilizes naive deep Q-learning.
- State the DQN loss and what experience replay and target networks each fix.
- Recall the one-line change behind Double DQN, Dueling, PER, and Rainbow.
- Say honestly where value-based RL is used, and why your LLM orchestrator isn't a DQN.

## From tabular Q-learning to a network

The **Q-learning** update (Watkins, 1989; Watkins & Dayan, 1992):

$$Q(s,a) \leftarrow Q(s,a) + \alpha\big[r + \gamma \max_{a'} Q(s', a') - Q(s,a)\big]$$

The bracket is the TD error from 4b. The $\max$ over next actions makes it
**off-policy**: it learns about the greedy policy regardless of how the data was
collected. Watkins & Dayan proved tabular Q-learning converges to the optimal $Q^*$
under mild conditions.

But tables don't scale, and worse, they don't *generalize*: a table treats two nearly
identical states as unrelated entries, so it must visit every state many times.
Impossible in large spaces. The fix is **function approximation**: replace the table
with $Q_\theta(s,a)$, a neural net that generalizes.

That generalization comes with a catch. Sutton & Barto (2018) name the **deadly
triad**, three ingredients that together can make value estimates diverge:

1. **Function approximation** (one update perturbs estimates everywhere),
2. **Bootstrapping** (the target is built from the network's own wrong estimates),
3. **Off-policy learning** (data distribution differs from the policy being evaluated).

Any one or two is usually safe. All three, which is exactly Q-learning with a neural
net, can spiral away from $Q^*$. That's why a naive implementation diverges, and why
DQN's whole contribution is a set of tricks to tame the triad.

## DQN: the two tricks

Mnih et al. (2015, *Nature*) introduced the **Deep Q-Network**, the first method to
learn control from high-dimensional pixels across many tasks. It minimizes a squared
TD-error loss:

$$L(\theta) = \mathbb{E}_{(s,a,r,s') \sim \mathcal{D}}\Big[\big(r + \gamma \max_{a'} Q_{\theta^-}(s', a') - Q_\theta(s,a)\big)^2\Big]$$

Two design choices, both aimed straight at the triad:

**Experience replay.** Online RL data is highly correlated (consecutive frames look
nearly identical), which violates the near-i.i.d. assumption SGD relies on. DQN stores
transitions in a **replay buffer** $\mathcal{D}$ and trains on **random minibatches**,
breaking correlation, reusing each transition, and smoothing the data distribution.

**Target networks.** The target $r + \gamma \max_{a'} Q_\theta(s',a')$ depends on the
very $\theta$ you're updating, so you're chasing a target that jumps every step. DQN
uses a separate **target network** $Q_{\theta^-}$, held fixed for many steps and
periodically copied from $\theta$, so the target stays still while you regress toward
it.

```
Initialize buffer D, online net Q_θ, target net Q_θ⁻ ← Q_θ
for each step:
    a ← ε-greedy(Q_θ, s)
    execute a, observe r, s'; store (s,a,r,s') in D
    sample minibatch from D
    y ← r                            if s' terminal
    y ← r + γ max_a' Q_θ⁻(s',a')     otherwise
    gradient step on (y − Q_θ(s,a))²
    every C steps: θ⁻ ← θ
```

The result, stated honestly: with the *same* architecture and hyperparameters across
games, DQN reached **human-level or above on many Atari 2600 games** from pixels and
score alone. It was strong on reactive games, poor on ones needing long-horizon
exploration (*Montezuma's Revenge*). The headline was *generality from one recipe*, not
superhuman play everywhere.

## The big variants (one line each)

- **Double DQN** (van Hasselt et al., 2016): the $\max$ both selects and evaluates with
  the same noisy network, which **overestimates** values. Fix: select with the online
  net, evaluate with the target net. $y = r + \gamma Q_{\theta^-}(s', \arg\max_{a'}
  Q_\theta(s',a'))$.
- **Dueling networks** (Wang et al., 2016): split into a state-value $V(s)$ and an
  advantage $A(s,a)$, recombined with mean-subtraction for identifiability: $Q = V +
  (A - \frac{1}{|\mathcal{A}|}\sum_{a'} A)$.
- **Prioritized Experience Replay** (Schaul et al., 2016): sample by TD-error magnitude
  $p_i \propto |\delta_i|^\omega$ instead of uniformly, with an importance-sampling
  correction to keep the gradient unbiased.
- **Rainbow** (Hessel et al., 2018): combine six of these and they stack, outperforming
  every component alone. The standard reference for "value-based deep RL, tuned."

## Where value-based RL is actually used (honestly)

Value-based deep RL fits a specific shape of problem: **discrete actions, a clear
scalar reward, cheap and abundant interaction.** Games and simulated control, candidate
ranking (research), robotics skill selection, scheduling (research). It struggles with
large or continuous action spaces (the $\max_{a'}$ goes intractable), which is where the
policy-based methods of 4d take over.

> ### Why your LLM orchestrator is not a DQN
>
> It's tempting to imagine a system coordinating LLM agents learns a $Q$-function over
> which agent to call next. It almost certainly does not. Production coordination in
> 2026 is **LLM planning plus engineered scaffolding**, a capable model proposes and
> sequences steps, deterministic code executes and constrains them. The mismatch is
> structural: a DQN needs a small fixed discrete action set (orchestration's "what next"
> is open-ended natural language), resettable episodes with a clean per-step reward
> (agent workflows are heterogeneous and single-shot), and millions of cheap
> interactions (each orchestrator step is an expensive model call against a
> non-resettable world). When you read that an orchestrator "learns to route," that's
> the LLM's in-context reasoning plus engineering, not a value network. Be skeptical of
> any claim of a deployed DQN scheduling agents.

Where do value functions *actually* show up in LLM-land? The **critic in PPO-based
RLHF** (a value head estimating reward-to-go) and **process reward models** (scoring
intermediate reasoning steps) are value-*like* signals over partial generations. Same
idea, applied to text. We pick that up in 4d.

## Hands-on

Reading the equations isn't the same as watching a TD error shrink. The canonical path:

1. **Gymnasium `CartPole-v1`.** A 4-D state, two actions, reward +1 per step upright. A
   small MLP, a replay buffer (a `deque`), a target net, ε-greedy. Trains in minutes on
   a CPU, so you can fight the deadly-triad bugs in real time.
2. **`ALE/Atari`.** Add a convolutional encoder and frame stacking (Mnih et al., 2015).
   Compute-hungry but reproduces the original setting.
3. **Reference:** `stable-baselines3`'s `DQN` is a clean, tested implementation. Use it
   as an oracle to diff against, after you've written the core loop yourself.

Good milestone: solve CartPole from scratch, then add Double DQN (one line in the
target) and watch the value estimates stop drifting up.

## The takeaways

- **Tabular Q-learning** converges but can't scale; **function approximation**
  generalizes at the cost of stability.
- The **deadly triad** (function approximation + bootstrapping + off-policy) is why
  naive deep Q-learning diverges.
- **DQN** tamed it with **experience replay** and **target networks**, reaching
  human-level Atari from pixels on one recipe.
- The variants are each a small principled fix (**Double, Dueling, PER, Rainbow**).
- Value-based RL fits **discrete-action, cheap-interaction** problems. It is **not** how
  LLM orchestration works; value functions show up as the **PPO critic** and **PRMs**.

## Think about it

1. Why does using the *same* network to select and evaluate the next action
   systematically *over*estimate Q-values? (This is the Double DQN motivation.)
2. Your task has a continuous action (a steering angle in [-1, 1]). Why is DQN a bad
   fit, and what should you reach for instead?

## Next

Module 4d crosses over to **policy-based** methods: parameterize the policy directly,
optimize with the policy gradient, and arrive at PPO, the engine of RLHF.

## References

- Watkins, C. J. C. H., & Dayan, P. (1992). Q-learning. *Machine Learning*, 8, 279-292.
- Mnih, V., et al. (2015). Human-level control through deep reinforcement learning. *Nature*, 518, 529-533.
- van Hasselt, H., Guez, A., & Silver, D. (2016). Deep RL with double Q-learning. *AAAI*.
- Wang, Z., et al. (2016). Dueling network architectures for deep RL. *ICML*.
- Schaul, T., et al. (2016). Prioritized experience replay. *ICLR*.
- Hessel, M., et al. (2018). Rainbow. *AAAI*.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
