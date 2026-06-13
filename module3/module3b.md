# Module 3B: Action Spaces and Value Trade-offs

## Introduction: The Architecture of Choice

While state spaces define what an agent can observe, **action spaces** define what an agent can do. For alignment, this distinction is crucial: an agent might perfectly understand a situation (complete state information) but still behave poorly if it lacks the right tools or approaches to express aligned behavior.

**The Core Insight**: The mathematical structure of an action space directly determines what kinds of aligned behaviors can emerge. A poorly designed action space makes alignment impossible, while a well-designed one makes alignment natural and learnable.

This lesson explores how to design action spaces that enable agents to navigate complex value trade-offs while maintaining alignment with human preferences.

## Mathematical Foundation of Action Spaces

### Basic Action Space Structure

An **action space** A defines the complete set of decisions available to an agent:

$$A = \{a_1, a_2, ..., a_n\}$$

for discrete spaces, or:

$$A \subseteq \mathbb{R}^d$$

for continuous action spaces, where $d$ is the dimensionality.

### Beyond Simple Actions: Tool-Based Action Spaces

For complex AI agents, actions aren't just simple choices, they're sophisticated tools with different capabilities, costs, and alignment properties. Each action can be characterized by a **feature vector**:

$$\mathbf{f}(a_i) = [c_i, t_i, p_i, \mathbf{s}_i, \mathbf{v}_i]$$

where:
- $c_i$ = computational/resource cost
- $t_i$ = time requirement  
- $p_i$ = reliability/success probability
- $\mathbf{s}_i$ = strength vector (accuracy, coverage, depth, etc.)
- $\mathbf{v}_i$ = value alignment vector (how well this action serves different human values)

### Multi-Tool Research Agent Action Space

For our research assistant, the action space contains tools that embody different approaches to information gathering and analysis:

$$A = \{a_{\text{academic}}, a_{\text{web}}, a_{\text{news}}, a_{\text{fact}}, a_{\text{sentiment}}, a_{\text{cite}}, a_{\text{summarize}}, a_{\text{cross}}, a_{\text{bias}}, a_{\text{confidence}}, a_{\text{human}}, a_{\text{synthesis}}\}$$

Each tool represents a fundamentally different value trade-off:

**Academic Search** ($a_{\text{academic}}$):
$$\mathbf{f}(a_{\text{academic}}) = [3, 2, 0.85, [0.9, 0.6, 0.8], [0.9, 0.3, 0.7]]$$
- Cost: 3 units, Time: 2 steps, Reliability: 85%
- Strengths: [accuracy: 0.9, speed: 0.6, coverage: 0.8]  
- Values: [truth: 0.9, efficiency: 0.3, comprehensiveness: 0.7]

**Web Search** ($a_{\text{web}}$):
$$\mathbf{f}(a_{\text{web}}) = [1, 1, 0.65, [0.6, 0.9, 0.9], [0.5, 0.9, 0.8]]$$
- Cost: 1 unit, Time: 1 step, Reliability: 65%
- Strengths: [accuracy: 0.6, speed: 0.9, coverage: 0.9]
- Values: [truth: 0.5, efficiency: 0.9, comprehensiveness: 0.8]

**Human Consultation** ($a_{\text{human}}$):
$$\mathbf{f}(a_{\text{human}}) = [8, 5, 0.95, [0.95, 0.2, 0.7], [0.95, 0.1, 0.6]]$$
- Cost: 8 units, Time: 5 steps, Reliability: 95%
- Strengths: [accuracy: 0.95, speed: 0.2, coverage: 0.7]
- Values: [truth: 0.95, efficiency: 0.1, comprehensiveness: 0.6]

## Mathematical Properties for Alignment-Friendly Action Spaces

### 1. Value Separability

An action space promotes alignment when it provides explicit choices between competing values. Formally, for any two competing values $v_1, v_2$:

$$\exists a_i, a_j \in A : \underset{a \in A}{\arg\max} \, v_1(\mathbf{f}(a)) = a_i, \underset{a \in A}{\arg\max} \, v_2(\mathbf{f}(a)) = a_j$$

**Example**: Our action space has value separability because:
- $\underset{a}{\arg\max} \text{accuracy}(\mathbf{f}(a)) = a_{\text{human}}$ (human consultation maximizes accuracy)
- $\underset{a}{\arg\max} \text{speed}(\mathbf{f}(a)) = a_{\text{web}}$ (web search maximizes speed)
- $\underset{a}{\arg\max} \text{cost\_efficiency}(\mathbf{f}(a)) = a_{\text{web}}$ (web search minimizes cost)

**Why This Matters**: Without value separability, the agent cannot learn to distinguish between different value priorities. All actions would represent similar trade-offs, making contextual alignment impossible.

### 2. Trade-off Continuity

For any two values, there should exist actions representing different points along the trade-off spectrum:

$$\forall \lambda \in [0,1], \exists a_\lambda \in A : \text{utility}(a_\lambda) \approx \lambda \cdot v_1(\mathbf{f}(a_\lambda)) + (1-\lambda) \cdot v_2(\mathbf{f}(a_\lambda))$$

**Example Trade-off Spectrum** (Speed vs. Accuracy):
- **Maximum Speed**: Web search (speed=0.9, accuracy=0.6)
- **Balanced**: News aggregator (speed=0.7, accuracy=0.7)  
- **High Accuracy**: Academic search (speed=0.6, accuracy=0.9)
- **Maximum Accuracy**: Human consultation (speed=0.2, accuracy=0.95)

**Mathematical Verification**:
We can verify trade-off continuity by checking that the Pareto frontier is well-covered:
$$\text{Pareto}(A) = \{a \in A : \nexists a' \in A \text{ such that } \mathbf{f}(a') \succ \mathbf{f}(a)\}$$

A good action space has a rich Pareto frontier with many non-dominated options.

### 3. Safety Preservation

There must always exist at least one action that maintains safety constraints, regardless of context:

$$\exists a_{\text{safe}} \in A : P(\text{constraint violation} | a_{\text{safe}}, s) < \epsilon, \forall s \in S$$

**In Our Action Space**:
- $a_{\text{human}}$ serves as the safety-preserving action
- Regardless of how complex or risky the situation becomes, human consultation provides a way to maintain alignment
- This creates a "safety valve" that prevents catastrophic alignment failures

**Mathematical Property**:
$$P(\text{alignment\_violation} | a_{\text{human}}, s) < 0.05, \forall s \in S$$

### 4. Expressiveness

The action space should be able to express the full range of behaviors needed for the domain:

$$\forall b \in \text{DesiredBehaviors}, \exists \tau = (a_1, ..., a_k) : \text{behavior}(\tau) = b$$

where $\tau$ is a sequence of actions that produces the desired behavior.

## Why Action Space Design Determines Alignment Capability

### The Fundamental Constraint

The action space defines the **possibility space** for alignment. Consider two different action space designs:

**Narrow Action Space**:
$$A_{\text{narrow}} = \{\text{search}, \text{analyze}, \text{respond}\}$$

**Rich Action Space**:
$$A_{\text{rich}} = \{\text{academic\_search}, \text{web\_search}, \text{fact\_check}, \text{bias\_detect}, \text{human\_consult}, ...\}$$

### Mathematical Analysis of Alignment Capability

**Alignment Expressiveness** can be measured as:
$$E(A) = |\{v \in \text{Values} : \exists a_{\max}, a_{\min} \in A \text{ such that } v(a_{\max}) - v(a_{\min}) > \delta\}|$$

This counts how many values the action space can meaningfully trade off.

**For Narrow Action Space**: $E(A_{\text{narrow}}) \leq 2$
- Limited ability to express value trade-offs
- Agent learns "one size fits all" behavior
- Cannot adapt to different user preferences or contexts

**For Rich Action Space**: $E(A_{\text{rich}}) \geq 5$  
- Can express trade-offs between accuracy, speed, cost, coverage, bias-resistance
- Agent can learn contextually appropriate behaviors
- Different users and situations get appropriately different treatments

### Value Learning Through Action Choice

The agent learns about values by observing which actions lead to higher rewards in different contexts. With rich action spaces:

$$\text{value\_learning}(v_i) = \sum_{s,a} P(s) \pi(a|s) \frac{\partial R(s,a)}{\partial v_i(a)}$$

Rich action spaces provide more gradient information for learning value preferences.

## Designing Actions for Multi-Objective Optimization

### The Multi-Objective Challenge

Real-world alignment involves optimizing multiple, often conflicting objectives simultaneously:

$$\max_{a \in A} [w_1 \cdot \text{accuracy}(a) + w_2 \cdot \text{speed}(a) + w_3 \cdot \text{cost\_efficiency}(a) + w_4 \cdot \text{comprehensiveness}(a)]$$

where weights $w_i$ represent user preferences that may vary across contexts.

### Pareto-Optimal Action Design

Each action should be **Pareto-optimal** for some combination of user preferences:

$$\forall a \in A, \exists \mathbf{w} : a = \underset{a' \in A}{\arg\max} \mathbf{w}^T \mathbf{f}(a')$$

This ensures that every action is "best" for some user preference profile.

**Verification for Our Action Space**:

**Academic Search** is optimal when: $w_{\text{accuracy}} > 0.7, w_{\text{speed}} < 0.3$
**Web Search** is optimal when: $w_{\text{speed}} > 0.8, w_{\text{cost}} > 0.7$
**Human Consultation** is optimal when: $w_{\text{accuracy}} > 0.9, w_{\text{safety}} > 0.8$

### Mathematical Tools for Action Space Analysis

**Coverage Analysis**:
How well does the action space cover the value space?
$$\text{Coverage}(A) = \frac{|\text{reachable\_values}(A)|}{|\text{total\_value\_space}|}$$

**Efficiency Analysis**:
How efficiently can the action space reach different points in value space?
$$\text{Efficiency}(A) = \min_{v \in \text{Values}} \min_{a \in A} \|\mathbf{f}(a) - v\|_2$$

**Robustness Analysis**:
How sensitive is alignment to action selection errors?
$$\text{Robustness}(A) = \min_{a, a' \in A} \frac{\text{alignment\_difference}(a, a')}{\text{action\_distance}(a, a')}$$

## Context-Dependent Action Availability

### Dynamic Action Spaces

In realistic environments, not all actions are always available:

$$A_t = A_{\text{base}} \cap A_{\text{available}}(s_t)$$

**Resource Constraints**:
$$a \in A_t \iff \text{cost}(a) \leq \text{budget\_remaining}(s_t)$$

**Time Constraints**:
$$a \in A_t \iff \text{time}(a) \leq \text{deadline\_remaining}(s_t)$$

**API/Service Availability**:
$$a \in A_t \iff \text{service\_status}(a, t) = \text{available}$$

### Maintaining Alignment Under Constraints

**Critical Property**: Even with constrained action spaces, alignment must be maintainable:

$$\forall s_t, \exists a \in A_t : \text{alignment\_violation}(a, s_t) < \epsilon$$

**Design Principle**: Ensure that safety-preserving actions (like human consultation) are available even under severe resource constraints:

$$a_{\text{human}} \in A_t \text{ whenever } \text{stakes}(s_t) > \text{high\_threshold}$$

## Action Hierarchies and Compositional Alignment

### Hierarchical Action Structure

Complex behaviors often require sequences of actions. We can structure the action space hierarchically:

$$A = A_{\text{atomic}} \cup A_{\text{composite}}$$

where:
- $A_{\text{atomic}}$ = individual tool uses
- $A_{\text{composite}}$ = pre-defined sequences or strategies

**Example Composite Actions**:
- $a_{\text{verify}} = [a_{\text{academic}}, a_{\text{fact\_check}}, a_{\text{cross\_reference}}]$
- $a_{\text{comprehensive}} = [a_{\text{web}}, a_{\text{academic}}, a_{\text{news}}, a_{\text{synthesis}}]$
- $a_{\text{conservative}} = [a_{\text{human}}, a_{\text{verify}}, a_{\text{confidence\_estimate}}]$

### Compositional Value Properties

**Additivity**: For some values, composite actions should have additive properties:
$$\text{comprehensiveness}([a_1, a_2]) = \text{comprehensiveness}(a_1) + \text{comprehensiveness}(a_2)$$

**Submodularity**: For others, diminishing returns apply:
$$\text{accuracy}([a_1, a_2]) \leq \text{accuracy}(a_1) + \text{accuracy}(a_2)$$

**Synergy**: Some combinations work better together:
$$\text{credibility}([a_{\text{academic}}, a_{\text{fact\_check}}]) > \text{credibility}(a_{\text{academic}}) + \text{credibility}(a_{\text{fact\_check}})$$

## Learning to Choose: Policy-Action Space Interaction

### How Policies Interact with Action Spaces

A policy $\pi(a|s)$ maps states to probability distributions over actions. The **expressiveness** of possible policies is directly constrained by the action space:

$$\text{PolicySpace}(\pi) \subseteq \Delta(A)^{|S|}$$

**Rich Action Spaces Enable Rich Policies**:
With more actions representing different value trade-offs, policies can learn more nuanced, context-dependent behaviors.

### Value-Sensitive Action Selection

For alignment, we want policies that consider value trade-offs explicitly:

$$\pi_{\text{aligned}}(a|s) = \frac{\exp(\beta[Q(s,a) + \alpha \sum_i w_i(s) v_i(a)])}{\sum_{a'} \exp(\beta[Q(s,a') + \alpha \sum_i w_i(s) v_i(a')])}$$

where:
- $Q(s,a)$ = learned state-action value
- $w_i(s)$ = context-dependent weight for value $i$
- $v_i(a)$ = how well action $a$ serves value $i$
- $\alpha, \beta$ = hyperparameters balancing performance and alignment

**This requires** that actions have well-defined value properties $v_i(a)$, which comes from careful action space design.

## Common Action Space Design Mistakes

### 1. Insufficient Value Differentiation

**Problem**: All actions represent similar trade-offs
$$\forall a_i, a_j \in A : \|\mathbf{v}(a_i) - \mathbf{v}(a_j)\|_2 < \epsilon$$

**Result**: Agent cannot learn value-sensitive behavior

**Solution**: Ensure action space spans the value space with sufficient separation

### 2. Missing Safety Actions

**Problem**: No action is safe in all contexts
$$\nexists a \in A : \text{safe}(a, s) = \text{true}, \forall s$$

**Result**: Agent has no recourse when facing novel or high-risk situations

**Solution**: Always include human consultation or conservative fallback actions

### 3. Poor Compositionality

**Problem**: Actions don't combine well to create complex behaviors
$$\text{behavior}([a_1, a_2]) \not\approx \text{combine}(\text{behavior}(a_1), \text{behavior}(a_2))$$

**Result**: Agent limited to simple, single-step behaviors

**Solution**: Design actions with clear interfaces and predictable composition properties

### 4. Context Insensitivity

**Problem**: Same actions available regardless of context
$$A_t = A, \forall t$$

**Result**: Agent wastes resources or violates constraints

**Solution**: Make action availability depend on context, resources, and constraints

## Key Design Principles for Alignment-Friendly Action Spaces

### 1. Value Separability is Essential

Ensure your action space provides clear choices between competing values. If all actions represent similar trade-offs, the agent cannot learn contextually appropriate alignment.

### 2. Safety Must Always Be Achievable

Include actions that maintain safety and alignment even in worst-case scenarios. This provides a "safety valve" preventing catastrophic failures.

### 3. Cover the Pareto Frontier

Design actions that span the space of reasonable value trade-offs. Each action should be optimal for some combination of user preferences.

### 4. Enable Compositional Complexity

Structure actions so they can combine to create complex, multi-step behaviors while maintaining predictable value properties.

### 5. Respect Resource and Context Constraints

Make action availability depend on context, ensuring the agent cannot choose inappropriate actions due to resource limitations or situational constraints.

## Measuring Action Space Quality

### Quantitative Metrics

**Value Coverage**:
$$C_{\text{value}} = \frac{|\{v : \exists a \in A, v(a) > \text{threshold}\}|}{|\text{all\_values}|}$$

**Pareto Efficiency**:
$$E_{\text{pareto}} = \frac{|\text{pareto\_optimal}(A)|}{|A|}$$

**Safety Guarantee**:
$$S_{\text{safety}} = \min_{s \in S} \max_{a \in A} \text{safety\_score}(a, s)$$

**Alignment Expressiveness**:
$$E_{\text{align}} = \max_{\mathbf{w}} \max_{a \in A} \mathbf{w}^T \mathbf{v}(a) - \min_{\mathbf{w}} \max_{a \in A} \mathbf{w}^T \mathbf{v}(a)$$

## Next Steps

Understanding action space design provides the foundation for creating agents that can express aligned behavior. In the next lesson (Module 3C), we'll explore how stochastic rewards create the learning signal that guides agents toward aligned action selection, building on the value-differentiating action spaces covered here.

**Key Takeaway**: The action space defines what kinds of aligned behaviors are possible. Everything else, learning algorithms, policies, rewards, can only work with the alignment capabilities that the action space provides.