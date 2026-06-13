# Module 3C: Stochastic Rewards and Risk

> **What you'll get out of this:** why "maximize the average reward" is sometimes
> exactly the wrong objective, and the math (CVaR) for caring about the bad days
> instead.

## The reward isn't a number, it's a distribution

Here's a habit worth breaking early. People write $R(s, a)$ as if the same action
in the same state always pays the same. It doesn't. Real environments are
stochastic: same action, same situation, different outcome, because of things the
agent doesn't control. APIs vary, sources vary, users react differently.

So stop treating reward as a fixed value:

$$R(s, a) \sim \mathcal{D}_{s,a}$$

It's a distribution. And once you accept that, a more interesting question opens up:
do you care about the *average* outcome, or the *worst* outcomes? For alignment,
the answer is often "the worst ones," and that changes everything.

## What you'll be able to do

- Model reward as a distribution, not a point.
- Tell mean-variance from CVaR objectives, and pick the right one.
- Explain epistemic versus aleatoric uncertainty and why it matters for exploration.

## Same tool, different distribution

The same action has a totally different reward distribution depending on the
problem. Academic search is a star for factual queries and mediocre under time
pressure:

$$R_{\text{factual}}(s, a_{\text{academic}}) \sim \mathcal{N}(9.0, 1.0^2) \qquad R_{\text{urgent}}(s, a_{\text{academic}}) \sim \mathcal{N}(4.0, 2.0^2)$$

That's the reality a single average hides. No tool is best everywhere, and the
*spread* matters as much as the mean.

## Why average isn't always the goal

Watch two tools:

$$\mathbb{E}[R(a_{\text{fast}})] = 8,\ \text{Var} = 9 \qquad \mathbb{E}[R(a_{\text{reliable}})] = 7.5,\ \text{Var} = 1$$

The fast tool wins on average. But in a high-stakes situation, a risk-aware agent
should often prefer the reliable one, lower mean, far lower chance of a disaster.
That preference shows up in the utility:

$$\text{Utility}(a \mid s) = \mathbb{E}[R(s,a)] - \gamma \cdot \text{Var}[R(s,a)] + \lambda \cdot \text{Alignment}(s,a)$$

where $\gamma$ is risk aversion. Conservative users have high $\gamma$, aggressive
users low. The point: **risk preference is itself a value you have to represent.**

## CVaR: caring about the bad tail

Variance penalizes spread in both directions, but you usually only fear the
downside. **Conditional Value at Risk** focuses there. For a confidence level
$\alpha$, it's the expected reward in the worst $\alpha$ fraction of outcomes:

$$\text{CVaR}_\alpha(s,a) = \mathbb{E}\big[R(s,a) \mid R(s,a) \leq \text{VaR}_\alpha(s,a)\big]$$

(Rockafellar & Uryasev, 2000). For safety-critical alignment, CVaR is usually the
right lens, because alignment failures live in the tail, not the average. You can
fold it straight into the Bellman equation:

$$Q^*(s,a) = \mathbb{E}[R(s,a)] - \alpha \cdot \text{Var}[R(s,a)] + \gamma\, \mathbb{E}[V^*(s')]$$

or constrain it directly: maximize expected reward *subject to* $\text{CVaR}_\alpha[\text{violation}] \le \epsilon$.

> **When to use which.** Expected value for symmetric, low-stakes, recoverable
> situations, where averaging over many interactions is what matters. CVaR (or a
> CVaR constraint) when a single bad outcome is expensive or irreversible. The
> choice encodes how much the user cares about the worst case. Make it on purpose.

## Two kinds of uncertainty (they call for opposite responses)

- **Epistemic:** uncertainty about the *mean* reward. You don't know it yet, but you
  can *reduce* it by gathering data. This is what exploration is for.
- **Aleatoric:** the *irreducible* randomness in the outcome. No amount of data
  removes it; you can only model it and plan around it.

Conflating them is a classic mistake: you can't explore your way out of aleatoric
noise, and you shouldn't stop exploring just because outcomes are noisy. **Thompson
sampling** handles the epistemic part cleanly: sample a plausible Q-value from your
posterior and act greedily on it, so you explore exactly in proportion to your
uncertainty.

$$\pi(a \mid s) = P\big(a = \arg\max_{a'} \tilde{Q}(s, a')\big),\quad \tilde{Q} \sim \text{posterior}$$

## Learning the distribution online

You're estimating means and spreads from a stream of noisy rewards. The workhorse
is an exponential moving average, cheap and adaptive:

$$\hat{\mu}_t = (1-\eta)\hat{\mu}_{t-1} + \eta r_t \qquad \hat{\sigma}_t^2 = (1-\eta)\hat{\sigma}_{t-1}^2 + \eta (r_t - \hat{\mu}_t)^2$$

How much data do you need? For sub-Gaussian rewards with variance $\sigma^2$,
reaching an $\epsilon$-accurate estimate takes on the order of

$$N \sim \mathcal{O}\!\left(\frac{\sigma^2 \log(1/\delta)}{\epsilon^2}\right)$$

samples. The lesson in that formula: noisier rewards cost you quadratically more
data. Variance isn't just a risk concern, it's a sample-efficiency tax.

## A claim to be careful with

You'll read that "stochastic rewards prevent reward hacking." Be skeptical. Noise
makes *some* brittle exploits harder, an agent can't lock onto a single
deterministic loophole. But it does **not** prevent specification gaming: if your
reward is misspecified, the agent will still find the misspecification, noise or no
noise. Stochasticity buys robustness against fragile shortcuts, not immunity from a
wrong objective. Don't oversell it.

## Where this shows up in modern training

Two direct connections you'll meet in Module 4:

- **GRPO's group baseline is Monte Carlo estimation of a reward distribution.**
  Sample several responses per prompt, use the group mean as the baseline. That's
  exactly estimating $\mathbb{E}[R]$ from samples (Shao et al., 2024).
- **RLVR (verifiable rewards)** is the low-variance dream of this lesson: when a
  program can *check* the answer, the reward distribution collapses toward
  deterministic, and reward hacking has far less room to operate, because you can't
  fool a verifier the way you can fool a learned model.

## The takeaways

- Reward is a **distribution**, not a number. The spread matters as much as the
  mean.
- **Average isn't always the objective.** Use expected value for low-stakes,
  recoverable settings; **CVaR** when the tail is expensive or irreversible.
- Separate **epistemic** (reducible, explore it) from **aleatoric** (irreducible,
  model it) uncertainty. Thompson sampling explores by uncertainty.
- **Noise is a sample-efficiency tax** ($N \sim \sigma^2/\epsilon^2$), and it does
  **not** immunize you against a misspecified reward.

## Think about it

1. In your domain, name a decision where you'd optimize CVaR instead of the mean.
   What's the tail you're afraid of?
2. You see an agent's reward variance spike on a tool. Is that epistemic (keep
   exploring) or aleatoric (model and move on)? How would you tell?

## Next

Module 3D zooms out from single rewards to whole **trajectories**: how to keep an
agent aligned across a multi-step sequence, not just one decision at a time.

## References

- Rockafellar, R. T., & Uryasev, S. (2000). Optimization of conditional value-at-risk. *Journal of Risk*, 2, 21-42.
- Shao, Z., et al. (2024). DeepSeekMath (GRPO). arXiv:2402.03300.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
