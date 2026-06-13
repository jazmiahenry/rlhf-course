# Module 3E: Policies, Risk, and Learning Values from Preferences

> **What you'll get out of this:** how values turn into a decision rule, how you
> *learn* those values from comparisons, and the punchline: the exact same math is
> the reward model of RLHF.

## Where this is going

Modules 3A through 3D set up what the agent sees, what it can do, and how rewards
and trajectories work. This lesson connects them through the **policy**, the rule
that turns a state and a set of values into an action, and then asks the question
the whole course has been circling: *where do the values themselves come from?*

The answer, preferences, is the same idea underneath all of modern post-training.
So this lesson is also the bridge into Module 4. Four parts: policies that encode
values, risk-aware objectives, learning values from preferences, and the jump to
RLHF.

## What you'll be able to do

- Write a value-weighted softmax policy and predict how temperature reshapes it.
- Choose between expected-value and CVaR objectives and defend it.
- State the Bradley-Terry model and compute a simple max-likelihood estimate.
- Explain how that exact likelihood becomes the RLHF reward model, KL term and all.

## Part 1: Policies that encode values

A **policy** maps states to distributions over actions. The workhorse form is the
**value-weighted softmax**. Give each action a utility that already folds in the
user's value weights, $U(s,a) = \mathbf{u}^\top \mathbf{f}(s,a)$, and turn utilities
into probabilities:

$$\pi(a \mid s) = \frac{\exp(\beta\, U(s,a))}{\sum_{a'} \exp(\beta\, U(s,a'))}$$

The **inverse temperature** $\beta$ controls how hard the policy commits. High
$\beta$ goes nearly greedy; low $\beta$ explores.

**Work it.** Two tools, utilities $U_A = 2.0$, $U_B = 1.0$.

- At $\beta = 1$: $\pi(A) = \frac{e^2}{e^2 + e^1} = \frac{7.389}{10.107} = 0.731$. So
  $\pi(B) = 0.269$. The better tool wins ~73% of the time, but B still gets tried.
- At $\beta = 2$: $\pi(A) = \frac{e^4}{e^4 + e^2} = \frac{54.598}{61.987} = 0.881$.

Same utilities. Doubling $\beta$ sharpened a 73/27 policy into 88/12. That one knob
trades exploration for commitment, and a little mass on the second-best action is
how the agent discovers when its value estimates are wrong.

> **Hard constraints don't go in the scores.** A tool that would violate privacy
> shouldn't be merely down-weighted, it should be *off the menu*. Put safety
> constraints in the policy's **support** (the feasible action set), not its
> utilities, so no amount of utility can buy a forbidden action.

## Part 2: Risk-aware objectives

Because rewards are stochastic (Module 3C), maximizing the *average* isn't always
right. Two policies can share an expected reward while one occasionally does
something catastrophic. For alignment, the tail often matters more than the mean.

The default objective is $J(\pi) = \mathbb{E}_\tau[\sum_t \gamma^t r_t]$. When
worst-case outcomes carry outsized cost, swap in or constrain with **CVaR** (the
worst-$\alpha$-fraction expectation from 3C):

$$J_{\text{robust}}(\pi) = \mathbb{E}_\tau\Big[\sum_t \gamma^t r_t\Big] - \lambda\, \text{CVaR}_\alpha\Big[\sum_t \gamma^t r_t\Big]$$

Use plain expected value for low-stakes, recoverable, symmetric settings. Use CVaR
(or a CVaR constraint) when a single bad outcome is expensive or irreversible. The
choice itself is a value statement, make it deliberately.

## Part 3: Learning values from preferences

So far we *assumed* the weights $\mathbf{u}$. In practice nobody hands you
"$u_{\text{accuracy}} = 0.7$." People are way better at "I prefer this output to
that one." Preference learning recovers the values from comparisons.

**The Bradley-Terry model** (Bradley & Terry, 1952) is the foundation. Give each
option a latent **strength** $s_i$; the probability that $i$ beats $j$ is

$$P(i \succ j) = \frac{e^{s_i}}{e^{s_i} + e^{s_j}} = \sigma(s_i - s_j)$$

Only the *difference* in strengths is identifiable: bigger gap, more reliable the
stronger option wins; equal strengths, coin flip. When the options are actions
under a value model, the strength *is* the utility $\mathbf{u}^\top \mathbf{f}(s,a)$,
so a user's pairwise choices become data about $\mathbf{u}$.

**A tiny worked case.** Suppose a user prefers tool A to tool B in **2 of 3**
trials. With one pair, the max-likelihood strength gap $\Delta = s_A - s_B$ is
whatever makes the model's predicted win-rate match the data: $\sigma(\Delta) =
2/3$, so

$$\Delta = \sigma^{-1}(2/3) = \ln\frac{2}{1} \approx 0.693$$

(Check: $\sigma(0.693) = 0.667 = 2/3$.) Positive but modest, 2-of-3 is weak
evidence, so the inferred gap is small. If A had won all 3, the unregularized MLE
would shoot $\Delta \to \infty$, which is exactly why in practice you regularize or
go Bayesian.

**The Bayesian view.** Put a prior on the weights, a **Dirichlet** is natural when
they're non-negative and sum to one, and update a posterior as comparisons arrive.
That unlocks **active preference learning**: instead of asking random comparisons,
ask the one that most reduces uncertainty about $\mathbf{u}$. **Thompson sampling**
does it simply, sample a plausible $\tilde{\mathbf{u}}$ from the posterior, act (or
pose the next comparison) as if it's true, update. This is the principled version of
"ask a clarifying question when value-uncertainty is high" from Module 3A.

## Part 4: The bridge to post-training

Here's the payoff. **The exact same Bradley-Terry likelihood, applied to LLM
responses instead of tools, is the reward model of RLHF.**

In RLHF (Christiano et al., 2017; Ouyang et al., 2022), annotators compare pairs of
model responses. A **reward model** $r_\phi(x, y)$ is trained by maximizing the
Bradley-Terry likelihood of those preferences:

$$\mathcal{L}(\phi) = -\mathbb{E}_{(x, y_w, y_l)}\big[\log \sigma\big(r_\phi(x, y_w) - r_\phi(x, y_l)\big)\big]$$

That is *identically* the MLE from Part 3. Strengths are now reward scores, options
are now responses. Then the learned reward drives policy optimization, maximizing
reward while staying close to a reference model:

$$\max_{\pi_\theta}\ \mathbb{E}_{x, y \sim \pi_\theta}[r_\phi(x, y)] - \beta\, \mathrm{KL}\big(\pi_\theta(\cdot \mid x)\,\|\,\pi_{\text{ref}}(\cdot \mid x)\big)$$

That **KL-to-reference** term is doing real work: without it, the policy drifts into
degenerate text that games the imperfect reward model. $\beta$ controls how far it
can wander. (Note this $\beta$ is a *constraint* knob, different job from Part 1's
temperature $\beta$. The field overloads the letter.)

So the whole pipeline is: **preferences → Bradley-Terry reward model →
KL-regularized policy optimization.** Module 4D derives the algorithms that do the
optimizing (PPO, GRPO). Module 4E shows how DPO collapses both stages into one
supervised loss and drops the separate reward model entirely. Both start from the
likelihood you just met. You don't need DPO's derivation yet, just the recognition
that it begins here.

## The takeaways

- A **value-weighted softmax** turns utilities into action probabilities; inverse
  temperature $\beta$ trades exploration for commitment ($U_A=2, U_B=1$ gives 0.73
  at $\beta=1$, 0.88 at $\beta=2$). Hard constraints live in the **support**, not
  the scores.
- Pick **expected value** for low-stakes, recoverable settings; **CVaR** when the
  tail is expensive.
- The **Bradley-Terry model** turns comparisons into strengths; MLE fits them
  (2-of-3 wins gives $\Delta = \ln 2 \approx 0.69$). Dirichlet prior plus Thompson
  sampling makes elicitation query-efficient.
- **That same likelihood on LLM responses is the RLHF reward model**, and RLHF adds
  a **KL-to-reference** term to stop reward hacking.

## Think about it

1. Re-run the softmax with $U_A = 2, U_B = 1$ at $\beta = 0.5$. Does B's share go up
   or down? Why?
2. Why is "only the strength *difference* is identifiable" the right property for a
   preference model, and not a bug?

## Next

Module 3F closes the module with curriculum design: how to *train* an agent so these
aligned policies emerge reliably. Then Module 4 turns the preference pipeline above
into working algorithms.

## References

- Bradley, R. A., & Terry, M. E. (1952). Rank analysis of incomplete block designs. *Biometrika*, 39, 324-345.
- Rockafellar, R. T., & Uryasev, S. (2000). Optimization of conditional value-at-risk. *Journal of Risk*, 2, 21-42.
- Christiano, P., et al. (2017). Deep reinforcement learning from human preferences. *NeurIPS*.
- Ouyang, L., et al. (2022). Training language models to follow instructions with human feedback. *NeurIPS*.
