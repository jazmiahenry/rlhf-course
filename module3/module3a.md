# Module 3A: State, Observation, and Sufficiency

> **What you'll get out of this:** the single hard rule that decides whether an
> agent can *ever* behave in a value-aligned way, before you pick a single
> algorithm.

## Start with the bluntest version

An agent acts on what it can see. So here's the rule, stated as harshly as it
deserves: **an agent can be no more aligned than its state representation
allows.** If the information that determines the right action isn't in what the
agent observes, no learning algorithm on Earth recovers the right behavior.
There's nothing to condition on. Game over before training starts.

This lesson makes that precise with standard tools (MDPs, POMDPs, belief states),
and fixes a popular but wrong way people state it.

## What you'll be able to do

- Write down an MDP and a POMDP, and explain belief states.
- Explain **state aliasing**, the property that actually matters, and why "distinct
  states need distinct actions" is the wrong rule.
- Apply feature sufficiency: if a feature that decides the right action isn't
  observable, no policy can use it.

## The MDP and its state

A **Markov decision process** is a tuple $(\mathcal{S}, \mathcal{A}, P, R,
\gamma)$: states, actions, a transition function $P(s' \mid s, a)$, a reward
$R(s, a)$, and a discount $\gamma$. The defining property is that the future
depends on the present alone:

$$P(s_{t+1} \mid s_0, a_0, \dots, s_t, a_t) = P(s_{t+1} \mid s_t, a_t)$$

When that holds, the current state is a **sufficient statistic** for the future.
Remembering more history buys nothing. And here's the part people miss: that's a
property of the *representation*, not the world. A representation is Markov when it
captures everything decision-relevant, and fails to be when it throws something
away. That's your design lever (Sutton & Barto, 2018; Puterman, 1994).

## Partial observability: the POMDP

Real agents don't see the true state. A **partially observable MDP** (Kaelbling,
Littman & Cassandra, 1998) adds an observation set $\Omega$ and a function
$O(o \mid s', a)$. The agent never sees $s_t$. It sees $o_t$.

One observation usually isn't sufficient: different true states can produce the
same observation. The principled response is a **belief state**, a distribution
over true states given everything seen so far:

$$b_t(s) = P(s_t = s \mid o_0, a_0, \dots, o_t)$$

updated by Bayes' rule as observations arrive. The key theorem: **the belief state
is a sufficient statistic for an optimal policy in a POMDP.** Condition on $b_t$
and you lose nothing versus conditioning on the full history. (Hold that thought.
It's exactly why LLM agents condition on their entire context window.)

## The mistake everyone makes, and the fix

A common (and wrong) claim: a good state representation must satisfy "distinct
states demand distinct actions." That's false. Two genuinely different situations
can perfectly well share the same best action, and forcing them apart is neither
necessary nor desirable.

The property that *actually* matters is its opposite: **state aliasing.** Aliasing
is when your representation collapses two situations that *need different actions*
into the same observed state:

$$o(s_i) = o(s_j) \quad \text{but} \quad a^*(s_i) \neq a^*(s_j)$$

When that happens, the agent literally can't tell them apart, so it has to pick one
action for both, and it's wrong on at least one. So the rule is:

> **The real rule:** your state must *distinguish situations that require different
> actions*. It need not, and should not, distinguish situations that share a best
> action. For alignment specifically, the most dangerous aliasing is between
> situations that differ in **stakes** or **user values** but look similar on the
> surface.

## Feature sufficiency (the provable version)

This gives a clean, correct restatement of the fuzzy "alignment-complete state"
idea, with no invented theorems:

> **Feature-sufficiency principle.** If a feature $f$ decides the value-correct
> action (two situations differing only in $f$ have different best actions), but
> $f$ isn't recoverable from the agent's observations, then no policy over those
> observations can be reliably aligned.

The proof is one breath: a policy is a function of the observation; if the
observation doesn't vary with $f$, neither can the policy; but the right action
does vary with $f$; so the policy is wrong whenever $f$ mattered. Not deep. Just
load-bearing. And it hands you an engineering checklist: for our research agent,
the state had better expose **user values** (the accuracy/speed/cost weighting),
**stakes**, **resource budget**, and **relevant history**. Drop any one and you've
guaranteed a class of misalignment.

## The central alignment POMDP: hidden user intent

The single most important partially-observed variable for alignment is the user's
true values. People state preferences badly. "Do this quickly" can sit right next
to an unspoken "but be accurate, the stakes are high." Formally, the true value
vector $v^*$ is a hidden state component, and the agent sees only noisy evidence of
it: the phrasing, the context, the reactions.

Treating this as a real POMDP justifies two behaviors that are otherwise just vibes:

1. **Maintain a belief over values** instead of committing to a guess, and act
   conservatively when being wrong about values is expensive.
2. **Ask.** When the value-belief is too uncertain to pick the right action, a
   clarifying question is the observation that most reduces uncertainty about
   $v^*$. "Should I prioritize speed or thoroughness here?" isn't a UX nicety, it's
   the optimal POMDP action when your belief is too diffuse to act on.

## How LLM agents actually represent state

The abstractions land directly on 2026 systems:

- **The context window is the observation-action history.** An LLM agent has no
  hidden state surviving outside the context. Everything it conditions on (system
  prompt, messages, prior tool calls and results, retrieved docs) is the token
  sequence. By the sufficiency theorem, conditioning on the full history is
  optimal, which is what the architecture does. So context management (truncating,
  summarizing) is a *belief-state* decision. Drop an alignment-determining feature
  and you induce aliasing.
- **Tool schemas define the action space; the system prompt encodes constraints.**
- **Partial observability of user intent** is the central POMDP feature, made
  concrete: the model sees words, not values, and must infer the latter.

The companion notebook (`RL_Alignment_Part1`) makes the state decomposition real
with four dataclasses, and you can read them as a feature-sufficiency checklist:
`ProblemState` (what's asked), `ContextState` (the situational features whose
aliasing causes misalignment), `ResourceState` (the constraints), and
`ConstraintState` (the user-value weights, the alignment-determining features).
Every field exists because dropping it would alias situations that need different
actions.

## The takeaways

- An MDP's **Markov property** makes the state a sufficient statistic; under
  partial observability, the **belief state** plays that role.
- The property that matters is **avoiding state aliasing**: distinguish situations
  that need different actions. "Distinct states need distinct actions" is false.
- **Feature sufficiency**: if an alignment-determining feature (values, stakes,
  budget) isn't observable, no policy can use it. So it must be in the state.
- **Hidden user intent** is the central alignment POMDP feature, which is what
  makes "maintain a belief over values" and "ask when unsure" the *optimal* moves,
  not just nice ones.

## Think about it

1. Name two situations in your domain that look identical to a model but demand
   different actions. What feature distinguishes them, and is it observable?
2. When is a clarifying question the mathematically correct action, and when is it
   just stalling?

## Next

Module 3B turns to the action space: how the *menu* of actions you give an agent
decides which aligned behaviors are even reachable.

## References

- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
- Puterman, M. L. (1994). *Markov Decision Processes*. Wiley.
- Kaelbling, L. P., Littman, M. L., & Cassandra, A. R. (1998). Planning and acting in partially observable stochastic domains. *Artificial Intelligence*, 101(1-2), 99-134.
