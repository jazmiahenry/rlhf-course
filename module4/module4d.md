# Module 4d: Policy Gradients and PPO

> **What you'll get out of this:** the algorithm that actually trained the models
> you use every day. PPO is the workhorse of RLHF, and by the end you'll know its
> one-line objective and why it works.

## Two ways to learn

Module 4c learned *values* ($Q(s,a)$) and acted greedily on them. That breaks when
the action space is huge or continuous (the $\max_{a'}$ has no tractable answer) and
it gives you no direct handle on the policy itself.

Policy-gradient methods flip it: **parameterize the policy directly** as
$\pi_\theta(a \mid s)$ and optimize it. No value function required to *act* (though
one sneaks back in to help). This is the family PPO belongs to, and PPO is the one
that post-trained essentially every instruction-following LLM in production. So it's
worth knowing for real, not by analogy.

## What you'll be able to do

- Write the policy-gradient objective and explain its high-variance problem.
- State PPO's clipped surrogate objective and read what the clip does.
- Explain exactly how PPO is used in RLHF (the KL term, tokens as actions).
- Say what GRPO changed and why it took over reasoning RL.

## The policy gradient, and its problem

The basic idea (REINFORCE): push up the probability of actions that led to good
returns. The gradient is

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}\big[\nabla_\theta \log \pi_\theta(a \mid s)\, \cdot\, A(s, a)\big]$$

where $A(s,a)$ is the **advantage**, how much better an action did than expected.
Good actions get their log-probability pushed up, bad ones down.

It works, and it's brutally **high-variance**. A single trajectory's return is a
noisy estimate, so the gradient jitters, and a big step in the wrong direction can
wreck a policy you spent a lot of compute building. Two fixes, stacked:

1. **Use advantage, not raw return.** Subtracting a baseline (how good the state
   already was) cuts variance without adding bias. That's what $A(s,a)$ is doing.
2. **Actor-critic.** Train a second network, the **critic** $V_\phi(s)$, to estimate
   that baseline, and use it to compute advantages (often via GAE over the
   trajectory). The **actor** is the policy; the critic is the variance-reducer. The
   value function you learned in 4c comes back here as a helper, not the star.

Even with both, plain policy gradients are twitchy: nothing stops a single update
from moving the policy *far* from where it collected its data, and once you're off
that distribution, your gradient estimate is garbage. That's the problem PPO solves.

## PPO: stay close to where your data came from

The "proximal" in PPO means: keep each update *near* the current policy. Define the
probability ratio between new and old policy:

$$r_t(\theta) = \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}$$

PPO maximizes the **clipped surrogate objective** (Schulman et al., 2017):

$$L^{\text{CLIP}}(\theta) = \mathbb{E}_t\Big[\min\big(r_t(\theta)\, \hat{A}_t,\; \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\, \hat{A}_t\big)\Big]$$

where $\hat{A}_t$ is the advantage estimate and $\epsilon$ (usually 0.1 to 0.2)
bounds the move. Read the $\min$ slowly, because the whole idea is in it: when the
advantage is positive, the objective stops rewarding ratio increases past
$1+\epsilon$; when negative, it stops rewarding decreases below $1-\epsilon$. Either
way, **the policy can't profit by moving far from where it gathered its data.**
That's the entire trust-region idea, written in one line. No second-order math, no
constraint solver, just a clip. That simplicity is why PPO won.

## Where PPO actually earns its keep: RLHF

PPO's most consequential job isn't game-playing or robotics. It's **post-training
language models.** In the classic RLHF pipeline (Ouyang et al., 2022, InstructGPT),
PPO optimizes the LLM policy against the learned reward model from Module 3E, with
two adaptations worth knowing exactly:

1. **The reward is per-response, with a per-token KL penalty.** The objective is

   $$\mathbb{E}[r_{\text{RM}}(x, y)] - \beta\, \text{KL}(\pi_\theta \,\|\, \pi_{\text{ref}})$$

   The KL term to the frozen reference (the SFT model) is load-bearing: without it,
   the policy drifts into reward-hacked gibberish that scores high on the imperfect
   reward model and reads like nonsense. You met this exact term in 3E.
2. **Tokens are the actions.** A "trajectory" is the generated response, one token
   per timestep, with GAE computed over the token sequence and a value head
   predicting expected reward from each prefix.

That's it. The thing that made ChatGPT-style models follow instructions is the
algorithm on this page, pointed at a reward model built from human preferences.

## GRPO: drop the critic

By 2025-26, **GRPO** (Group Relative Policy Optimization, Shao et al., 2024) largely
displaced PPO for reasoning-focused RL. Its move is simple and clever: **throw away
the learned value function.** Instead of training a critic to estimate the baseline,
sample a *group* of responses for the same prompt and use the group's mean reward as
the baseline. (Recognize that? It's the Monte Carlo baseline from Module 4b.)

No critic network means roughly half the memory and far simpler training, which is
exactly what made large-scale reasoning RL (RLVR on math and code verifiers)
practical. And notice what *didn't* change: GRPO keeps the clipped-ratio objective
from this lesson. It only swaps where the advantage baseline comes from. PPO is still
the right mental model.

## An honest note on where this is used

Policy gradients and PPO are the standard for **continuous control** (robotics,
simulated locomotion), some game-playing, and, most importantly for this course,
**LLM post-training**. What they are *not*, despite a lot of hype, is the thing that
runs production "agent orchestration." Coordinating multiple LLM agents is LLM
planning plus engineered scaffolding, not a PPO policy trained to schedule agents.
(Module 4a and 4c make this case in detail.) When you see PPO, think *post-training
a model offline*, not *picking tools at runtime*.

## The takeaways

- **Policy-gradient** methods optimize $\pi_\theta$ directly; raw returns make them
  high-variance, so you use **advantage** and an **actor-critic** to tame it.
- **PPO's clipped objective** keeps each update near the data-collection policy, the
  trust-region idea in one line. That simplicity is why it dominates.
- **PPO is the engine of RLHF**: optimize against a reward model with a **KL-to-
  reference** penalty, tokens as actions.
- **GRPO** drops the critic for a **group-mean baseline** (Monte Carlo), keeping PPO's
  clip. It made large-scale reasoning RL practical.

## Think about it

1. In the clipped objective, what happens to a good action ($\hat{A}_t > 0$) whose
   new probability is already 3x the old? Why is that the behavior you want?
2. Remove the KL-to-reference term from RLHF. Predict the failure mode. (This is a
   real thing that happens.)

## Next

Module 4e covers **DPO** and the direct-alignment family: methods that optimize from
preferences *without* a separate reward model or an RL loop at all, collapsing the
whole pipeline from 3E into a single supervised loss.

## References

- Schulman, J., et al. (2017). Proximal Policy Optimization Algorithms. arXiv:1707.06347.
- Ouyang, L., et al. (2022). Training language models to follow instructions with human feedback (InstructGPT). *NeurIPS*.
- Shao, Z., et al. (2024). DeepSeekMath: Pushing the Limits of Mathematical Reasoning (GRPO). arXiv:2402.03300.
