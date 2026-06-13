# Module 3A: State, Observation, and Sufficiency for Aligned Decisions

## Learning Objectives

By the end of this lesson you will be able to:

- State the formal definitions of an MDP and a POMDP, and explain belief states.
- Explain **state aliasing** and why it — not "distinct states need distinct
  actions" — is the property that matters for alignment.
- Articulate the feature-sufficiency principle: if a feature that determines the
  value-correct action is unobservable, no policy can condition on it.
- Connect these abstractions to how LLM agents actually represent state in 2026,
  and to the dataclasses in the companion notebook.

## Introduction: An Agent Acts on What It Can See

In reinforcement learning, an agent's behavior is a function of its state
representation. For alignment this has a blunt consequence: **an agent can be no
more aligned than its state representation permits.** If the information that
determines the value-correct action is not present in what the agent observes,
then no learning algorithm, however sophisticated, can recover the right
behavior — there is simply nothing to condition on. This lesson makes that
intuition precise using the standard machinery of Markov decision processes and
their partially observed cousins, and connects it to how modern LLM agents
represent state.

We build on standard definitions throughout (Sutton & Barto, 2018; Puterman,
1994; Kaelbling, Littman & Cassandra, 1998). Where the previous version of this
lesson reached for invented predicates and an incorrect "context sensitivity"
property, we use the established concepts — which turn out to say the useful
thing more correctly.

## The MDP and Its State

A **Markov decision process** is a tuple $(\mathcal{S}, \mathcal{A}, P, R,
\gamma)$: a state set $\mathcal{S}$, an action set $\mathcal{A}$, a transition
function $P(s' \mid s, a)$, a reward function $R(s, a)$, and a discount $\gamma
\in [0, 1)$. The defining feature is the **Markov property**: the future depends
on the present alone,

$$
P(s_{t+1} \mid s_0, a_0, \dots, s_t, a_t) = P(s_{t+1} \mid s_t, a_t).
$$

When this holds, the current state is a **sufficient statistic** for the future:
remembering more history buys nothing. This is a property *of the state
representation*, not of the world — and that is precisely the design lever. A
representation is Markov when it captures everything decision-relevant; it fails
to be Markov when it throws away something that matters.

## Partial Observability: The POMDP

Real agents rarely see the true state. A **partially observable MDP** (Kaelbling,
Littman & Cassandra, 1998) extends the MDP with an observation set $\Omega$ and an
observation function $O(o \mid s', a)$ giving the probability of seeing
observation $o$ after action $a$ lands the system in $s'$. The agent never sees
$s_t$; it sees $o_t$.

A single observation is generally *not* a sufficient statistic — different true
states can produce the same observation. The principled response is to maintain a
**belief state** $b_t$, a probability distribution over true states given the
history of observations and actions:

$$
b_t(s) = P(s_t = s \mid o_0, a_0, o_1, a_1, \dots, o_t).
$$

The belief is updated by Bayes' rule as new observations arrive:

$$
b_{t+1}(s') \propto O(o_{t+1} \mid s', a_t) \sum_{s} P(s' \mid s, a_t)\, b_t(s).
$$

The key theorem: **the belief state is a sufficient statistic for an optimal
policy in a POMDP** — an agent that conditions on $b_t$ loses nothing relative to
one that conditions on the full history. Equivalently, the full
observation-action history is itself a sufficient statistic. This is exactly why
LLM agents condition on their entire context, as we discuss below.

## State Aliasing: The Property That Actually Matters

The previous version of this lesson claimed that a good state representation must
satisfy $s_i \neq s_j \Rightarrow$ different optimal actions. That is **false** as
stated: two genuinely different states can perfectly well share the same optimal
action (an agent that should "search the academic database" in many distinct
situations is not thereby broken). Requiring distinct states to demand distinct
actions is neither necessary nor desirable.

The correct concern is its converse — **state aliasing** (a standard POMDP
notion). Aliasing occurs when the representation collapses two situations that
*require different actions* into the same observed state:

$$
o(s_i) = o(s_j) \quad \text{but} \quad a^*(s_i) \neq a^*(s_j).
$$

When this happens, the agent literally cannot tell the two apart, so it must
choose one action for both — and is necessarily wrong in at least one. A state
representation must **distinguish situations that require different actions**. It
need not, and should not, distinguish situations that share an optimal action.
The discipline, then, is not "make every state unique" but "make sure no two
situations that demand different responses look identical to the agent." For
alignment specifically, the situations most often dangerously aliased are ones
that differ in *stakes* or *user values* while looking superficially similar in
content.

## Feature Sufficiency for Aligned Behavior

This gives a clean, correct restatement of the old lesson's "alignment-complete
state" idea — without inventing any theorems.

> **Feature-sufficiency principle.** If a feature $f$ determines the
> value-correct action (two situations differing only in $f$ have different
> optimal actions), but $f$ is not recoverable from the agent's observations,
> then no policy over those observations can be reliably value-aligned.

The proof is immediate: a policy is a function of the observation; if the
observation does not vary with $f$, neither can the policy; but the correct action
does vary with $f$; therefore the policy is wrong whenever $f$ would have changed
the answer. This is not a deep result — it is the aliasing argument applied to
alignment-relevant features — but it is the *correct* and load-bearing one, and it
tells you exactly what your state design must capture.

Which features are alignment-determining for our research agent? At minimum:

- **User values** — the relative weight on accuracy, speed, cost. If the agent
  cannot observe (or infer) that this user prioritizes rigor, it cannot
  consistently choose rigor.
- **Stakes** — a casual question and a high-consequence one may share wording but
  demand different care. Aliasing them is a classic alignment failure.
- **Resource budget** — the value-correct action under a tight budget differs from
  the one under a generous budget.
- **Relevant history** — prior tool results and commitments, so the agent stays
  consistent across a multi-step interaction.

Each is a feature whose absence from the state guarantees a class of misalignment.
The design question is not "is my state big enough?" but "for each pair of
situations my users will present that demand different actions, can my agent tell
them apart?"

## The Central Alignment POMDP: Hidden User Intent

The single most important partially observed variable for alignment is **the
user's true values**. Users state preferences imperfectly: a request to "do this
quickly" may coexist with an unstated insistence on accuracy when the stakes are
high. Formally, the true value vector $v^*$ is a hidden state component; the
agent observes only noisy, partial evidence of it (the phrasing, the context, the
reactions to prior outputs).

This is a POMDP in the strict sense, and it licenses two principled behaviors that
are otherwise ad hoc:

1. **Belief maintenance over values.** Rather than committing to a point estimate
   of $v^*$, the agent can maintain a belief and act under uncertainty —
   conservatively when the stakes of being wrong about values are high.
2. **Active information gathering.** When the value-belief is too uncertain to
   determine the right action, the value-of-information calculation favors *asking*
   — a clarifying question is the observation that most reduces uncertainty about
   $v^*$. "Should I prioritize speed or thoroughness here?" is not a UX nicety; it
   is the optimal POMDP action when the belief over values is too diffuse to act
   on.

Module 3E develops how preference evidence is actually turned into a value
estimate; here the point is structural: hidden user intent is *the* POMDP feature
that makes alignment a partial-observability problem rather than a
fully-observed-control problem.

## State in Practice: How LLM Agents Represent It

These abstractions map directly onto 2026 systems.

- **The context window is the observation-action history.** An LLM agent has no
  hidden recurrent state; everything it conditions on — system prompt, messages,
  prior tool calls and results, retrieved documents — is the token sequence in
  context. By the POMDP sufficiency theorem, conditioning on the full history is
  optimal, which is exactly what the architecture does. Context management
  (truncation, summarization) is therefore a *belief-state* decision: drop an
  alignment-determining feature and you induce aliasing.
- **Tool schemas define the action space**, and **the system prompt encodes
  constraints** — both are parts of the observation that shape which actions are
  legal and which are appropriate.
- **Partial observability of user intent** is realized concretely: the model sees
  words, not values, and must infer the latter — the central POMDP feature above.

### Connection to the companion notebook

`RL_Alignment_Part1_Mathematical_Foundations.ipynb` makes the state decomposition
concrete. It represents the agent's state as four dataclasses, each capturing one
slice of the feature-sufficiency requirement:

- **`ProblemState`** — query type, complexity, domain: *what is being asked*.
- **`ContextState`** — time pressure, urgency, user expertise: *the situational
  features that change the value-correct action* (precisely the features whose
  aliasing causes misalignment).
- **`ResourceState`** — budget and limits remaining: *the constraint features*.
- **`ConstraintState`** — including the user-value weights: *the alignment-
  determining features* that must be observable for value-conditioned behavior.

Reading those classes alongside this lesson, you can see the feature-sufficiency
principle as an engineering checklist: every field exists because its absence
would alias situations that demand different actions. The notebook then builds the
action space and stochastic rewards on top of this state.

## Key Takeaways

- An MDP's **Markov property** makes the current state a sufficient statistic;
  under partial observability (a **POMDP**), the **belief state** plays that role.
- The property that matters for alignment is **avoiding state aliasing** — the
  representation must distinguish situations that require different actions. The
  old "distinct states ⇒ distinct actions" claim is false and unnecessary.
- **Feature sufficiency**: if an alignment-determining feature (user values,
  stakes, budget) is unobservable, no policy can condition on it — so it must be
  in the state. This is the correct, provable version of "alignment-complete
  state."
- **Hidden user intent** is the central alignment POMDP feature, justifying belief
  maintenance over values and clarifying questions as the optimal action under
  value-uncertainty.
- In 2026 agents, the **context window is the history**, and the notebook's
  `ProblemState`/`ContextState`/`ResourceState`/`ConstraintState` are a concrete
  feature-sufficiency checklist.

## Next Steps

Module 3B examines how the action space must be designed to expose meaningful
value trade-offs, and 3C turns to the stochastic rewards that those actions
produce. The thread that runs through all of them is the one established here: the
agent acts on its representation, so alignment is won or lost in what that
representation makes observable.

## References

- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
- Puterman, M. L. (1994). *Markov Decision Processes: Discrete Stochastic Dynamic Programming*. Wiley.
- Kaelbling, L. P., Littman, M. L., & Cassandra, A. R. (1998). Planning and acting in partially observable stochastic domains. *Artificial Intelligence*, 101(1–2), 99–134.
