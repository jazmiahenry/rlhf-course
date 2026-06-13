# Module 3C: Stochastic Rewards and Uncertainty

## Introduction: Embracing Uncertainty for Robust Alignment

Traditional approaches to AI alignment often assume deterministic relationships: if an agent takes the "right" action in a given situation, it should reliably produce the "right" outcome. However, real-world environments are fundamentally stochastic, the same action in the same situation can produce dramatically different results due to factors beyond the agent's control.

**The Paradigm Shift**: Rather than treating uncertainty as a problem to be minimized, stochastic reward modeling treats uncertainty as an essential feature of realistic alignment. By learning to maintain alignment across uncertain outcomes, agents develop more robust and generalizable aligned behavior.

This lesson explores the mathematical foundations of stochastic reward systems and why they are crucial for developing AI agents that remain aligned even when facing unpredictable outcomes.

## Mathematical Foundation of Stochastic Rewards

### From Deterministic to Stochastic Rewards

**Traditional Deterministic Approach**:
$$R(s,a) = \text{fixed value}$$

**Stochastic Approach**:
$$R(s,a) \sim \mathcal{D}_{s,a}$$

where $\mathcal{D}_{s,a}$ is a probability distribution that depends on both state $s$ and action $a$.

### Multi-Dimensional Stochastic Structure

For complex environments like our multi-tool research agent, the reward structure captures multiple sources of uncertainty:

$$R(s,a,c) = R_{\text{base}}(s,a) + R_{\text{context}}(c) + R_{\text{alignment}}(s,a,v) + \epsilon$$

where:
- $R_{\text{base}}(s,a) \sim \mathcal{N}(\mu_{s,a}, \sigma_{s,a}^2)$ = base tool effectiveness
- $R_{\text{context}}(c) \sim \mathcal{U}(-\delta_c, \delta_c)$ = contextual modifiers (time pressure, resource constraints)
- $R_{\text{alignment}}(s,a,v) \sim \mathcal{B}(\phi_{s,a})$ = alignment bonus/penalty based on value satisfaction
- $\epsilon \sim \mathcal{N}(0, \sigma_{\text{noise}}^2)$ = random environmental noise

### Problem-Type Dependent Distributions

The same tool can have dramatically different effectiveness distributions depending on the problem type:

**For Controversial Topics**:
$$R_{\text{controversial}}(s, a_{\text{academic}}) \sim \mathcal{N}(8.0, 1.5^2)$$
$$R_{\text{controversial}}(s, a_{\text{web}}) \sim \mathcal{N}(3.0, 2.5^2)$$
$$R_{\text{controversial}}(s, a_{\text{bias\_detect}}) \sim \mathcal{N}(9.0, 1.2^2)$$

**For Time-Sensitive Queries**:
$$R_{\text{urgent}}(s, a_{\text{academic}}) \sim \mathcal{N}(4.0, 2.0^2)$$
$$R_{\text{urgent}}(s, a_{\text{web}}) \sim \mathcal{N}(7.5, 1.8^2)$$
$$R_{\text{urgent}}(s, a_{\text{news}}) \sim \mathcal{N}(8.5, 2.2^2)$$

**For Factual Queries**:
$$R_{\text{factual}}(s, a_{\text{academic}}) \sim \mathcal{N}(9.0, 1.0^2)$$
$$R_{\text{factual}}(s, a_{\text{web}}) \sim \mathcal{N}(6.0, 3.0^2)$$
$$R_{\text{factual}}(s, a_{\text{fact\_check}}) \sim \mathcal{N}(8.5, 1.5^2)$$

**Key Insight**: Notice how the same action (e.g., academic search) has completely different expected performance and variance depending on the problem type. This captures the reality that no single approach works optimally for all situations.

## Why Stochastic Rewards Matter for Alignment

### 1. Preventing Overfitting to Specific Outcomes

**Deterministic Problem**:
With deterministic rewards, agents can develop brittle strategies:
$$\text{If } R(s, a_{\text{fast}}) = 10 \text{ always, then always choose fast tools}$$

**Stochastic Solution**:
With stochastic rewards, agents must consider distributions:
$$\mathbb{E}[R(s, a_{\text{fast}})] = 8, \text{Var}[R(s, a_{\text{fast}})] = 9$$
$$\mathbb{E}[R(s, a_{\text{reliable}})] = 7.5, \text{Var}[R(s, a_{\text{reliable}})] = 1$$

A risk-aware agent might prefer the reliable tool despite lower expected reward, especially in high-stakes situations.

### 2. Learning Robust Value Trade-offs

**Expected Value vs. Risk Management**:
Agents must learn to balance expected performance with risk tolerance:

$$\text{Utility}(a|s) = \mathbb{E}[R(s,a)] - \gamma \cdot \text{Var}[R(s,a)] + \lambda \cdot \text{Alignment}(s,a)$$

where:
- $\gamma$ = risk aversion parameter
- $\lambda$ = alignment weight parameter

**User-Dependent Risk Preferences**:
Different users have different risk tolerances, leading to different optimal action selections:

$$\text{Conservative User: } \gamma_{\text{conservative}} = 0.8$$
$$\text{Aggressive User: } \gamma_{\text{aggressive}} = 0.2$$

### 3. Alignment Under Uncertainty

Real-world alignment means maintaining values even when outcomes are unpredictable. Stochastic rewards force agents to learn policies that are robust across the full distribution of possible outcomes.

**Alignment Robustness Condition**:
$$P(\text{alignment\_violation} | a, s) < \epsilon, \forall \text{realizations of } R(s,a)$$

This requires that aligned behavior emerges not just on average, but across the entire range of possible stochastic outcomes.

### 4. Preventing Gaming and Reward Hacking

**Deterministic Vulnerability**:
With deterministic rewards, agents can find loopholes:
$$\text{If } R(\text{query satisfied}) = +10 \text{ always, exploit technicalities to "satisfy" queries}$$

**Stochastic Protection**:
With stochastic rewards based on genuine user satisfaction:
$$R(\text{query satisfaction}) \sim \mathcal{N}(\text{true\_satisfaction}, \sigma^2)$$

Gaming becomes much harder because agents must succeed across many random variations of user reaction.

### 5. Modeling Real-World Complexity

Stochastic rewards reflect genuine uncertainty in tool effectiveness:

**API Variability**: Services have variable response times and quality
$$R_{\text{api\_call}} \sim \mathcal{N}(\mu_{\text{service}}, \sigma_{\text{load}}^2)$$

**Source Quality**: Information quality varies unpredictably
$$R_{\text{source\_quality}} \sim \text{Beta}(\alpha_{\text{reputation}}, \beta_{\text{reputation}})$$

**User Satisfaction**: User reactions depend on factors beyond agent control
$$R_{\text{user\_satisfaction}} \sim \mathcal{N}(\text{objective\_quality}, \sigma_{\text{subjective}}^2)$$

## Mathematical Analysis of Stochastic Learning

### Expected Value Estimation

Under stochastic rewards, the agent must learn to estimate:

$$Q^*(s,a) = \mathbb{E}[R(s,a) + \gamma \max_{a'} Q^*(s',a') | s,a]$$

This becomes:

$$Q^*(s,a) = \mathbb{E}_{\mathcal{D}_{s,a}}[r] + \gamma \mathbb{E}_{s' \sim P(s'|s,a)}[\max_{a'} Q^*(s',a')]$$

The agent must estimate both the mean and higher moments of the reward distribution.

### Variance-Aware Value Functions

For alignment, we often need to consider not just expected rewards but their variability:

**Mean-Variance Value Function**:
$$V_{\text{robust}}(s) = \mathbb{E}[R] - \alpha \cdot \text{Var}[R]$$

**Conditional Value at Risk (CVaR)**:
$$\text{CVaR}_\beta(s,a) = \mathbb{E}[R(s,a) | R(s,a) \leq \text{VaR}_\beta(s,a)]$$

This focuses on worst-case performance, crucial for safety-critical alignment.

**Risk-Sensitive Bellman Equation**:
$$Q^*(s,a) = \mathbb{E}[R(s,a)] - \alpha \cdot \text{Var}[R(s,a)] + \gamma \mathbb{E}[V^*(s')]$$

### Exploration Under Uncertainty

Stochastic rewards create richer exploration dynamics:

**Epistemic vs. Aleatoric Uncertainty**:
- **Epistemic**: Uncertainty about the mean $\mathbb{E}[R(s,a)]$ (reducible through experience)
- **Aleatoric**: Inherent randomness in $R(s,a)$ (irreducible)

**Thompson Sampling for Stochastic Rewards**:
$$\pi(a|s) = P(a = \underset{a'}{\arg\max} \tilde{Q}(s,a'))$$

where $\tilde{Q}(s,a)$ is sampled from the posterior distribution over Q-values.

## Stochastic Reward Design for Multi-Tool Agents

### Context-Dependent Variance

Different contexts should have different levels of uncertainty:

**Low-Stakes Contexts** (routine fact-checking):
$$R(s,a) \sim \mathcal{N}(\mu, 0.5^2)$$ (low variance)

**High-Stakes Contexts** (medical information):
$$R(s,a) \sim \mathcal{N}(\mu, 2.0^2)$$ (high variance)

**Novel Contexts** (emerging technologies):
$$R(s,a) \sim \mathcal{N}(\mu, 3.0^2)$$ (very high variance)

### Tool Interaction Effects

Reward distributions can depend on the sequence of tools used:

**Independent Tools**:
$$R([a_1, a_2]) = R(a_1) + R(a_2) + \epsilon$$

**Synergistic Tools**:
$$R([a_{\text{academic}}, a_{\text{fact\_check}}]) \sim \mathcal{N}(\mu_1 + \mu_2 + \text{synergy}, \sigma^2)$$

**Conflicting Tools**:
$$R([a_{\text{web}}, a_{\text{academic}}]) \sim \mathcal{N}(\mu_1 + \mu_2 - \text{conflict}, \sigma_{\text{increased}}^2)$$

### Temporal Dynamics

Reward distributions can change over time:

**Tool Degradation**:
$$\mu_t = \mu_0 \cdot \exp(-\lambda t)$$ (performance decreases with overuse)

**Learning Effects**:
$$\sigma_t^2 = \sigma_0^2 \cdot (1 + \alpha \cdot \text{experience}_t)^{-1}$$ (variance decreases with experience)

**Environmental Shifts**:
$$\mathcal{D}_{s,a,t} = (1-\beta) \mathcal{D}_{s,a,t-1} + \beta \mathcal{D}_{\text{new}}$$ (gradual distribution shift)

## Alignment Robustness Through Stochastic Training

### Worst-Case Alignment Analysis

For safety-critical applications, we need alignment guarantees across the entire reward distribution:

**Worst-Case Constraint**:
$$\inf_{r \sim \mathcal{D}_{s,a}} \text{alignment\_score}(s,a,r) \geq \tau_{\text{min}}$$

**Probabilistic Constraint**:
$$P(\text{alignment\_score}(s,a,r) \geq \tau | r \sim \mathcal{D}_{s,a}) \geq 1-\delta$$

### Risk-Aware Policy Optimization

**Traditional Objective**:
$$J(\pi) = \mathbb{E}_{\tau \sim \pi}[\sum_t \gamma^t r_t]$$

**Risk-Aware Objective**:
$$J_{\text{robust}}(\pi) = \mathbb{E}_{\tau \sim \pi}[\sum_t \gamma^t r_t] - \alpha \cdot \text{CVaR}_\beta[\sum_t \gamma^t r_t]$$

This balances expected performance with worst-case protection.

### Alignment-Constrained Exploration

**Standard ε-greedy**:
$$\pi(a|s) = \begin{cases}
\text{random action} & \text{with probability } \epsilon \\
\underset{a}{\arg\max} Q(s,a) & \text{otherwise}
\end{cases}$$

**Alignment-Constrained Exploration**:
$$\pi(a|s) = \begin{cases}
\text{random aligned action} & \text{with probability } \epsilon \\
\underset{a \in A_{\text{aligned}}}{\arg\max} Q(s,a) & \text{otherwise}
\end{cases}$$

where $A_{\text{aligned}} = \{a : P(\text{alignment\_violation}|a,s) < \delta\}$

## Measuring Stochastic Learning Progress

### Distributional Learning Metrics

**Moment Matching**:
How well does the agent's learned distribution match the true reward distribution?
$$\text{KL}(\mathcal{D}_{\text{true}}(s,a) \| \mathcal{D}_{\text{learned}}(s,a))$$

**Risk Assessment Accuracy**:
How well does the agent predict worst-case outcomes?
$$\text{Error}_{\text{CVaR}} = |\text{CVaR}_{\text{true}} - \text{CVaR}_{\text{estimated}}|$$

**Alignment Robustness**:
How consistent is alignment across stochastic realizations?
$$\text{Robustness} = 1 - \text{Var}_{r \sim \mathcal{D}}[\text{alignment\_score}(s,a,r)]$$

### Convergence Under Stochasticity

**Sample Complexity**: How many samples are needed to learn good policies under stochastic rewards?

For sub-Gaussian rewards with variance $\sigma^2$:
$$N_{\text{samples}} = O\left(\frac{\sigma^2 \log(1/\delta)}{\epsilon^2}\right)$$

**Alignment Convergence**: How quickly does alignment improve?
$$\mathbb{E}[\text{alignment\_violation}_t] \leq \mathbb{E}[\text{alignment\_violation}_0] \cdot \exp(-\lambda t)$$

## Practical Implementation Considerations

### Reward Distribution Estimation

**Maximum Likelihood Estimation**:
$$\hat{\mu}, \hat{\sigma}^2 = \underset{\mu, \sigma^2}{\arg\max} \prod_i P(r_i | \mu, \sigma^2)$$

**Bayesian Estimation**:
$$P(\mu, \sigma^2 | \{r_i\}) \propto P(\{r_i\} | \mu, \sigma^2) P(\mu, \sigma^2)$$

**Non-parametric Estimation**:
Use kernel density estimation or empirical distributions when the parametric form is unknown.

### Online Distribution Updates

As new data arrives, update reward distribution estimates:

**Exponential Moving Average**:
$$\hat{\mu}_t = (1-\alpha) \hat{\mu}_{t-1} + \alpha r_t$$
$$\hat{\sigma}_t^2 = (1-\alpha) \hat{\sigma}_{t-1}^2 + \alpha (r_t - \hat{\mu}_t)^2$$

**Sliding Window**:
Use only the most recent $N$ observations to handle non-stationarity.

### Computational Considerations

**Monte Carlo Estimation**:
For complex distributions, use sampling to estimate expectations:
$$\mathbb{E}[f(R)] \approx \frac{1}{N} \sum_{i=1}^N f(r_i)$$

**Variance Reduction**:
Use control variates, importance sampling, or stratified sampling to reduce estimation variance.

## Common Pitfalls in Stochastic Reward Design

### 1. Underestimating Variance

**Problem**: Assuming low variance when true variance is high
$$\sigma_{\text{assumed}}^2 \ll \sigma_{\text{true}}^2$$

**Result**: Agent overconfident in predictions, makes poor risk assessments

**Solution**: Conservative variance estimation, especially early in training

### 2. Ignoring Distribution Shape

**Problem**: Assuming normal distributions when true distributions are skewed or multi-modal

**Result**: Poor tail risk estimation, inadequate worst-case planning

**Solution**: Use flexible distribution families or non-parametric methods

### 3. Static Distribution Assumptions

**Problem**: Assuming reward distributions don't change over time

**Result**: Agent fails to adapt to changing environments

**Solution**: Implement distribution shift detection and adaptation mechanisms

### 4. Correlation Neglect

**Problem**: Treating correlated rewards as independent
$$\text{Cov}[R(s,a_1), R(s,a_2)] \neq 0 \text{ but assumed } = 0$$

**Result**: Poor portfolio-like decision making when combining tools

**Solution**: Model correlation structure explicitly

## Integration with Alignment Objectives

### Stochastic Alignment Constraints

Rather than deterministic constraints, use probabilistic alignment guarantees:

$$P(\text{value\_violation}(a,s) | r \sim \mathcal{D}_{s,a}) \leq \epsilon$$

### Multi-Objective Stochastic Optimization

Balance multiple stochastic objectives:

$$\max_\pi \mathbb{E}[R_{\text{performance}}] \text{ subject to } P(R_{\text{alignment}} < \tau) \leq \delta$$

### Uncertainty-Aware Value Learning

Learn user values while accounting for uncertainty in their expression:

$$V_{\text{user}} \sim \mathcal{N}(\hat{v}, \Sigma_v)$$

Update beliefs based on stochastic feedback:

$$P(V_{\text{user}} | \text{feedback}) \propto P(\text{feedback} | V_{\text{user}}) P(V_{\text{user}})$$

## Key Takeaways

### 1. Embrace Uncertainty, Don't Fight It

Stochastic rewards reflect genuine uncertainty in real-world environments. Trying to eliminate uncertainty often leads to brittle, overconfident systems.

### 2. Risk Awareness Enables Robust Alignment

By considering not just expected outcomes but their variability, agents can maintain alignment even when facing unpredictable situations.

### 3. Stochasticity Prevents Gaming

Random variation in rewards makes it much harder for agents to exploit loopholes or find unintended shortcuts to high rewards.

### 4. Distribution Learning is Crucial

Agents must learn not just means but full reward distributions to make good decisions under uncertainty.

### 5. Context Affects Uncertainty

Different situations should have different levels of uncertainty. High-stakes or novel contexts warrant higher variance and more conservative approaches.

## Next Steps

Understanding stochastic rewards provides the foundation for learning robust policies under uncertainty. In the next lesson (Module 3D), we'll explore how agents can maintain alignment throughout entire trajectories of actions, building on the uncertainty-aware reward modeling covered here.

**Key Insight**: Stochastic rewards force agents to develop alignment strategies that work across distributions of outcomes, not just average cases. This creates more robust, generalizable aligned behavior.