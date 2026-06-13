# Module 3E: Policies, Risk, and Learning Values from Preferences

## Learning Objectives

By the end of this lesson you will be able to:

- Write a value-weighted softmax policy and predict how temperature reshapes it
  (with a worked numeric example).
- Choose between expected-value and CVaR objectives and justify the choice.
- State the Bradley-Terry preference model and compute a simple maximum-likelihood
  strength estimate.
- Explain how the same Bradley-Terry likelihood becomes the reward model of RLHF,
  and where the KL-to-reference term comes from.

## Introduction: From Values to Decisions

Module 3A established what must be *observable* for aligned behavior; 3C
established that rewards are *stochastic*. This lesson connects the two through
the **policy**, the rule that turns an observed state and a set of values into an
action, and then asks the question that the rest of the course answers
algorithmically: *where do the values themselves come from?* The answer,
preferences, is the same idea that underlies all of modern post-training, so this
lesson is also the conceptual bridge into Module 4.

We proceed in four parts: (1) policies that encode values, (2) risk-aware
objectives, (3) learning values from preferences, and (4) the bridge to RLHF.

## Part 1: Policies That Encode Values

A **policy** maps states to distributions over actions, $\pi: \mathcal{S} \to
\Delta(\mathcal{A})$. For our multi-tool research agent, $\pi(a \mid s)$ is the
probability of selecting tool $a$ in state $s$.

The workhorse form is the **value-weighted softmax**. Suppose each action has a
utility $U(s, a)$ that already folds in the user's value weights, for instance
$U(s,a) = \mathbf{u}^\top \mathbf{f}(s,a)$, a weighted sum of how well action $a$
serves each value (accuracy, speed, cost) with weights $\mathbf{u}$. The policy
turns utilities into probabilities:

$$
\pi(a \mid s) = \frac{\exp(\beta\, U(s, a))}{\sum_{a'} \exp(\beta\, U(s, a'))}.
$$

The **inverse temperature** $\beta$ controls how sharply the policy commits to the
highest-utility action. Large $\beta$ → nearly greedy; small $\beta$ → nearly
uniform exploration.

### Worked example

Two tools, with utilities (under this user's value weights) $U_A = 2.0$ and
$U_B = 1.0$. The softmax probabilities:

- **At $\beta = 1$:** $\pi(A) = \dfrac{e^{2}}{e^{2} + e^{1}} = \dfrac{7.389}{10.107}
  = 0.731$, so $\pi(B) = 0.269$. The better tool is preferred about 73% of the
  time, but the agent still explores B.
- **At $\beta = 2$:** $\pi(A) = \dfrac{e^{4}}{e^{4} + e^{2}} = \dfrac{54.598}{61.987}
  = 0.881$, so $\pi(B) = 0.119$. Doubling $\beta$ sharpens the same utilities into
  a much more committed policy.

The utilities did not change, only the temperature. This is the lever that trades
exploration for exploitation, and it is why stochastic policies are useful for
alignment: a little probability mass on the second-best action lets the agent
discover when its value estimates are wrong, and avoids brittle overcommitment to
a locally optimal but globally misaligned choice.

**Constrained policy classes.** Some actions must never be chosen regardless of
utility, a tool that would violate a hard privacy constraint, say. We handle this
not by tuning utilities but by restricting the action set the softmax ranges over
to the feasible set $\mathcal{A}_{\text{safe}}(s) = \{a : \text{constraints
satisfied}\}$. Hard safety constraints belong in the *support* of the policy, not
in its scores, so that no amount of utility can buy a forbidden action.

## Part 2: Risk-Aware Objectives

Because rewards are stochastic (Module 3C), maximizing *expected* utility is not
always the right objective. Two policies can share an expected reward while one
occasionally produces catastrophic outcomes and the other never does. For
alignment, the tail often matters more than the mean.

The standard expected-value objective is

$$
J(\pi) = \mathbb{E}_{\tau \sim \pi}\Big[\sum_t \gamma^t r_t\Big].
$$

When worst-case outcomes carry disproportionate cost, replace or constrain it with
**Conditional Value at Risk (CVaR)** (Rockafellar & Uryasev, 2000). For a
confidence level $\alpha$, CVaR is the expected reward in the worst $\alpha$
fraction of outcomes:

$$
\text{CVaR}_\alpha[R] = \mathbb{E}\big[R \mid R \leq \text{VaR}_\alpha[R]\big],
$$

where $\text{VaR}_\alpha[R]$ is the $\alpha$-quantile of the reward distribution.
A risk-sensitive objective trades a little expected reward for protection against
the tail:

$$
J_{\text{robust}}(\pi) = \mathbb{E}_{\tau \sim \pi}\Big[\sum_t \gamma^t r_t\Big] - \lambda\, \text{CVaR}_\alpha\Big[\sum_t \gamma^t r_t\Big].
$$

**When to use which.** Use the plain expected-value objective when outcomes are
roughly symmetric and individual failures are cheap and recoverable, routine,
low-stakes tasks where averaging over many interactions is what matters. Use a
CVaR objective (or a CVaR *constraint*, $\text{CVaR}_\alpha[\text{violation}] \le
\epsilon$) when a single bad outcome is expensive or irreversible, medical,
legal, financial, safety-critical settings. The choice is itself a value
statement: it encodes how much the user cares about the worst case relative to the
average, and it should be made deliberately, not defaulted.

## Part 3: Learning Values From Preferences

So far we have *assumed* the value weights $\mathbf{u}$. In practice they are
rarely handed to you as numbers; people are far better at saying *"I prefer this
output to that one"* than at stating "$u_{\text{accuracy}} = 0.7$." Preference
learning recovers the values from comparisons.

### The Bradley-Terry model

The foundational tool is the **Bradley-Terry model** (Bradley & Terry, 1952).
Assign each option a latent **strength** $s_i$; the probability that option $i$ is
preferred to option $j$ is

$$
P(i \succ j) = \frac{\exp(s_i)}{\exp(s_i) + \exp(s_j)} = \sigma(s_i - s_j),
$$

where $\sigma$ is the logistic sigmoid. Only the *difference* in strengths is
identifiable, and the model says: the bigger the strength gap, the more reliably
the stronger option wins; equal strengths give a coin flip.

When the options are *actions under a value model*, the strength is the utility:
$s_a = \mathbf{u}^\top \mathbf{f}(s, a)$. A user's pairwise choices then become
data about $\mathbf{u}$.

### Maximum-likelihood estimation

Given a set of observed comparisons, we fit the strengths (or the underlying value
weights) by **maximum likelihood**: choose the parameters that make the observed
preferences most probable, i.e. maximize $\prod_{\text{comparisons}} \sigma(s_i -
s_j)$ over the wins.

A minimal worked case builds the intuition. Suppose, in a single repeated
comparison, a user prefers tool A to tool B in **2 of 3** trials. With one pair,
the MLE strength difference $\Delta = s_A - s_B$ is the value that makes the
model's predicted win-rate match the empirical one: $\sigma(\Delta) = 2/3$,
so

$$
\Delta = \sigma^{-1}\!\left(\tfrac{2}{3}\right) = \ln\frac{2}{1} \approx 0.693.
$$

(Check: $\sigma(0.693) = 0.667 = 2/3$.) The estimate is positive but modest, A is preferred, but 2-of-3 is weak evidence, so the inferred strength gap is
small. Had A won all 3, the unregularized MLE would push $\Delta \to \infty$
(perfect separation), which is exactly why in practice we regularize or take a
Bayesian view.

### The Bayesian and active view

Treating $\mathbf{u}$ as a point estimate discards uncertainty. A Bayesian
treatment places a prior over value weights, a **Dirichlet** prior is natural
when the weights are non-negative and sum to one, and updates a posterior as
comparisons arrive:

$$
P(\mathbf{u} \mid \text{data}) \propto P(\text{data} \mid \mathbf{u})\, P(\mathbf{u}).
$$

This unlocks **active preference learning**: rather than asking the user to compare
random options, ask the comparison that most reduces uncertainty about
$\mathbf{u}$, an information-maximizing query. **Thompson sampling** gives a
simple, effective procedure: sample a plausible $\tilde{\mathbf{u}}$ from the
posterior, act (or pose the next comparison) as if it were true, and update on the
result. Over time the posterior concentrates on the user's real values, with fewer
queries than random elicitation would need. This is the principled version of the
"ask a clarifying question when value-uncertainty is high" behavior that Module 3A
motivated from the POMDP structure.

## Part 4: The Bridge to Post-Training

Here is the payoff that makes this lesson the hinge of the course. **The exact
same Bradley-Terry likelihood, applied to *LLM responses* instead of tools,
is the reward model of RLHF.**

In RLHF (Christiano et al., 2017; Ouyang et al., 2022), human annotators compare
pairs of model responses to the same prompt. A **reward model** $r_\phi(x, y)$, scoring response $y$ to prompt $x$, is trained by maximizing the Bradley-Terry
likelihood of the observed preferences:

$$
\mathcal{L}(\phi) = -\mathbb{E}_{(x, y_w, y_l)}\big[\log \sigma\big(r_\phi(x, y_w) - r_\phi(x, y_l)\big)\big],
$$

where $y_w$ is the preferred ("winning") response and $y_l$ the dispreferred one.
This is *identically* the Bradley-Terry MLE from Part 3, strengths are now reward
scores, options are now responses.

The learned reward model then supplies the signal for **policy optimization**. The
RLHF objective maximizes reward while staying close to a reference (the
supervised-fine-tuned) policy $\pi_{\text{ref}}$:

$$
\max_{\pi_\theta}\; \mathbb{E}_{x,\, y \sim \pi_\theta}\big[r_\phi(x, y)\big] - \beta\, \mathrm{KL}\big(\pi_\theta(\cdot \mid x)\,\|\,\pi_{\text{ref}}(\cdot \mid x)\big).
$$

The **KL-to-reference** term is essential: without it the policy drifts into
degenerate text that scores high on the imperfect reward model but is useless, the reward-hacking failure mode. $\beta$ controls how far the policy may move from
the trusted reference. (Notice this $\beta$ plays a *constraint* role, distinct
from the temperature $\beta$ of Part 1, same letter, different job; the field
overloads it.)

That is the entire conceptual pipeline: **preferences → Bradley-Terry reward model
→ KL-regularized policy optimization.** Module 4D derives the algorithms that
perform that optimization (PPO and its successor GRPO). Module 4E then shows how
DPO collapses the two stages, reward modeling and policy optimization, into a
single supervised loss on the policy, eliminating the separate reward model
entirely. Both build directly on the Bradley-Terry foundation laid here; you do
not need DPO's derivation yet, only the recognition that it starts from this same
likelihood.

## Key Takeaways

- A **value-weighted softmax** policy turns utilities into action probabilities;
  **inverse temperature $\beta$** trades exploration for commitment (worked
  example: $U_A=2, U_B=1$ gives $\pi(A) = 0.73$ at $\beta=1$, $0.88$ at $\beta=2$).
  Hard constraints belong in the policy's **support**, not its scores.
- Choose **expected value** for symmetric, low-stakes, recoverable settings;
  **CVaR** when the tail is expensive or irreversible. The choice encodes a value.
- The **Bradley-Terry model** $P(i \succ j) = \sigma(s_i - s_j)$ turns pairwise
  comparisons into latent strengths; MLE fits them (2-of-3 wins ⇒ $\Delta =
  \ln 2 \approx 0.69$). A **Dirichlet** prior plus **Thompson sampling** enables
  active, query-efficient elicitation.
- The **same Bradley-Terry likelihood on LLM responses is the RLHF reward model**;
  the RLHF objective adds a **KL-to-reference** term to prevent reward hacking.
  Module 4D/4E supply the algorithms.

## Next Steps

Module 3F closes the module with curriculum design, how to *train* an agent so the
value-aligned policies discussed here emerge reliably and the constraints hold from
the start. Then Module 4 turns the preference-learning pipeline of Part 4 into
working algorithms: PPO and GRPO (4D), and DPO and its successors (4E).

## References

- Bradley, R. A., & Terry, M. E. (1952). Rank analysis of incomplete block designs: I. The method of paired comparisons. *Biometrika*, 39(3/4), 324-345.
- Rockafellar, R. T., & Uryasev, S. (2000). Optimization of conditional value-at-risk. *Journal of Risk*, 2, 21-42.
- Christiano, P., et al. (2017). Deep reinforcement learning from human preferences. *NeurIPS*.
- Ouyang, L., et al. (2022). Training language models to follow instructions with human feedback (InstructGPT). *NeurIPS*.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
