# Module 4a: From MDP Theory to Modern Agent Architecture

In Module 3 we built the formal machinery of sequential decision-making:
states, actions, rewards, policies, returns, and the Bellman equations that tie
them together. That formalism is exact, elegant, and, on its own, silent
about silicon. It tells you what an optimal policy *is*; it does not tell you
what a "state" looks like when the agent is a large language model reading a
200,000-token context, or what an "action" is when the agent emits a JSON tool
call.

This lesson is the bridge. Before we start deriving algorithms in 4b through
4f, we need a shared, honest picture of how the formal objects $(S, A, R, \pi)$
actually appear inside the AI agent systems people build in 2026, and, just
as importantly, where the MDP framing is load-bearing versus where it is a
metaphor we should retire. The old version of this lesson claimed that
production orchestration systems train value networks online to choose tools.
That was false, and the falsehood obscured the most useful thing a practitioner
can know: which parts of a modern agent are *learned* and which are
*engineered*.

## Learning objectives

By the end of this lesson you should be able to:

- Map each MDP component, state, action, reward, policy, onto a concrete part
  of a modern LLM-based agent.
- Explain why the same agent can be modeled as an MDP at two different action
  resolutions, and choose the right resolution for a given optimization goal.
- Identify where reward signals genuinely come from in 2026 systems, and name
  the failure modes of the most fragile source.
- State clearly which components of a production agent are trained with
  reinforcement learning and which are hand-built, and defend the distinction.

## State in practice: the context window as observation history

Recall the distinction from Module 3a between a fully observed MDP and a
*partially observed* one (a POMDP). In a POMDP the agent never sees the true
environment state $s_t$ directly; it sees an observation $o_t$, and the
principled response is to condition behavior on the entire history
$h_t = (o_0, a_0, o_1, a_1, \dots, o_t)$ rather than on any single observation.
The optimal policy is a function of a *belief* over states, and the history is
a sufficient statistic for that belief.

An LLM agent is a POMDP agent in exactly this sense, and the context window is
its history representation. Everything the model conditions on at step $t$, the system prompt, the user's messages, every prior tool call and tool result,
retrieved documents, scratchpad reasoning, is concatenated into one token
sequence. That sequence *is* $h_t$. The agent has no hidden recurrent state
that survives outside the context; its entire memory of the episode is the text
it can currently see. This is why context management is not an engineering
footnote but a first-class modeling decision: truncating, summarizing, or
reordering the context is literally choosing what belief the policy gets to act
on.

Two refinements matter. First, the model does not consume raw tokens as
features. Each token is mapped to a learned embedding, and the transformer's
self-attention layers (Vaswani et al., 2017) compute contextualized
representations that serve as the agent's *state features*. This is the quiet
revolution that separates modern agents from the systems of twenty years ago.
Classical pipelines hand-engineered a small fixed feature vector and sometimes
compressed it with linear methods such as PCA before feeding a controller.
Those features were chosen by a human and were linear, fixed, and
task-specific. Learned embeddings are none of those things: they are
high-dimensional, trained end-to-end, and they adapt their geometry to whatever
structure the data contains. We mention the contrast only to retire it, the
relevant fact is that *the state representation is now learned, not designed*.

Second, the context is *structured*, and the structure carries meaning the
policy relies on. A system prompt fixes the agent's objective and constraints.
Tool schemas declare what actions exist and how they are shaped. Retrieved
documents inject information the weights do not contain. Memory files persist
facts across episodes. From the MDP's point of view these are all just part of
$o_t$, but treating them as undifferentiated text would miss that each is a
different *channel* into the belief state, with different freshness, trust, and
update dynamics.

## Actions in practice: typed tool calls and the resolution problem

In a textbook MDP the action set $A$ is given and discrete: move left, move
right. A modern agent's action space is *typed*. The dominant pattern is
structured tool calling: the agent emits a function call whose name and
arguments must conform to a declared JSON schema, and increasingly these tools
are exposed over standardized interfaces such as the Model Context Protocol
(MCP). The schema defines the action space the way the rules of a board game
define legal moves, a call that violates the schema is, in effect, an illegal
action.

Here is the conceptual move that the rest of Module 4 depends on. **The same
agent is an MDP at two different resolutions, and which one you use depends on
what you are optimizing.**

- **Token-level.** Zoom all the way in. The state is the token sequence so far;
  the action is the next token; the policy is the language model's next-token
  distribution. At this resolution the episode is a few hundred to a few
  thousand transitions, and the action space is the vocabulary. This is the
  right lens for *post-training* the model itself, RLHF, PPO, GRPO, DPO all
  operate here, because the thing being updated is the weight matrix that
  produces the token distribution.

- **Tool-level.** Zoom out. Treat one full model turn, reason, then emit a
  tool call, as a single action, and the tool's result as the next
  observation. The state is the conversation/tool history; the action space is
  the set of available tools and their arguments; an episode is a handful to a
  few dozen steps. This is the right lens for *orchestration*: reasoning about
  whether the agent will accomplish a multi-step task, how to sequence tools,
  where it loops or stalls.

Neither resolution is "more correct." They are two valid MDP abstractions of
one system, related the way a coarse-grained and fine-grained simulation of the
same physics are related. The cost of being sloppy about which one you mean is
real: a claim like "we trained the policy with PPO" is precise at the token
level and a category error at the tool level if no weights are being updated
there. Keep asking: *at what resolution, and is anything being learned at that
resolution?*

## Rewards in practice: where the signal actually comes from

Module 3 treated the reward function $R(s, a)$ as given by the environment. In
real systems, manufacturing a usable reward is often the hardest part of the
whole enterprise. In 2026 the signal comes from a small number of sources, each
with a different trust profile.

**Verifiable outcomes (RLVR).** The cleanest reward is one a program can check.
Did the unit tests pass? Does the final answer match the known solution? Did
the SQL return the expected rows? When a task has a checkable ground truth, you
can compute reward with a deterministic verifier and skip the
learned-reward-model machinery entirely. This is the basis of *reinforcement
learning from verifiable rewards*, and it is attractive precisely because the
reward cannot be gamed by fooling a model, it can only be earned by being
correct. Its limit is coverage: most things we want from agents (a helpful
explanation, a tasteful refactor, a kind tone) have no verifier.

**Learned reward models from preferences.** When outcomes are not verifiable,
the standard move, the one Module 3 set up and Module 4d revisits, is to
learn a reward model from human preference comparisons. Annotators rank
outputs; a model is trained to score the preferred one higher; that scalar
score becomes the reward signal for policy optimization. This buys coverage
over fuzzy objectives at the price of a reward that is only as good as the
preference data and that the policy will, given the chance, learn to exploit.

**LLM-as-judge.** A cheaper proxy is to prompt a capable model to score or
compare outputs. It is fast and scales, and it is genuinely useful for
evaluation and for some reward shaping, but be honest about its failure modes.
LLM judges exhibit position bias (preferring the first or last option
presented), verbosity bias (rating longer answers higher regardless of
quality), self-preference (favoring text in their own style), and sensitivity
to superficial formatting; they can also be moved by adversarial phrasing in
the thing they are judging. A judge is a measurement instrument with systematic
error, not an oracle. Treat its scores as evidence to be calibrated and
audited, never as ground truth, and never optimize hard against a single judge
without checking what the policy is actually learning to do.

**Per-step versus episode-level signal.** A final structural choice cuts across
all three sources: *when* does reward arrive? A verifier that only fires "tests
passed" at the end of a long coding episode gives you one scalar for dozens of
decisions. That is the sparse-reward, long-horizon credit-assignment problem
from Module 3d in its full severity, the learning signal must somehow be
propagated back across every intermediate step to discover which ones mattered.
Denser per-step signals (a reward model scoring each turn, partial credit for
sub-goals) make credit assignment far easier but introduce their own bias,
because now you are rewarding a *proxy* for progress rather than the outcome
itself. Much of the craft in 4b through 4d is exactly managing this tension
between signal density and signal fidelity.

## Policies in practice: the agent loop, and what is actually learned

A policy $\pi(a \mid s)$ is a mapping from states to action distributions. In an
LLM agent that mapping is realized as a loop: **observe** the current context,
**reason** about what to do, **act** by emitting a tool call or a final answer,
then fold the result back into the context and repeat. That loop, with its
surrounding prompts and tool set, *is* the policy at the tool-level resolution.

Now the central honesty of this lesson, and the reason it replaces the old one.
Inside that policy, two very different things are doing the work:

- **What is learned:** the model weights. They were shaped by pre-training and
  then by *post-training*, supervised fine-tuning and reinforcement-learning
  methods such as PPO (Schulman et al., 2017) or GRPO (Shao et al., 2024). This
  is where genuine RL lives in the modern stack.
- **What is engineered:** the scaffold. The system prompt, the choice and
  wording of tools, the control flow of the loop, retry logic, memory policy,
  when to stop, these are written by humans and fixed at deployment. No
  gradient touches them while the agent runs.

So when we call a production system an "agent," what we almost always mean is
**an engineered policy wrapped around a learned model.** The model weights came
from RL (among other methods); the orchestration around them did not. At
runtime, the agent is *executing* a fixed policy, not improving one. It is not
running gradient descent between turns, it is not estimating a value function
online, and it is not training a DQN to pick tools, the picture the previous
version of this lesson painted. Each user request is, from the learning
algorithm's perspective, an inference pass through frozen weights.

This is not a knock on production agents; it is the correct mental model, and
it is liberating once internalized. It tells you where your levers are. If the
agent reasons poorly, you can engineer the scaffold (better prompts, better
tools, better context) *or* you can change the weights via post-training, and
those are completely different projects with different costs, feedback loops,
and risks. RL enters this whole story in exactly one place by default:
**post-training the model, offline, before deployment**, not at runtime. There
are research and production systems that learn at the tool level too, but that
is a deliberate, comparatively rare design choice, not the default meaning of
the word "agent." Knowing which world you are in is the first question to ask
of any agent system, including your own.

## Key takeaways

- The **context window** is the agent's POMDP history $h_t$; learned
  **embeddings** are its state features, having replaced hand-engineered ones;
  and that context is **structured** (system prompt, tool schemas, retrieval,
  memory) in ways the policy depends on.
- The action space is **typed** (JSON-schema tool calls, MCP), and the agent is
  an MDP at **two resolutions**, tokens for post-training, tool calls for
  orchestration. State the resolution before you reason.
- Reward in 2026 comes from **verifiable outcomes (RLVR)**, **learned reward
  models**, and **LLM-as-judge** (a biased instrument, not an oracle).
  **Sparse, episode-level** rewards make Module 3d's credit-assignment problem
  bite hardest.
- A production agent is an **engineered policy over a learned model.** Genuine
  RL lives in **post-training**, offline; runtime is inference through frozen
  weights, not online learning.

## Looking ahead

With the formalism now grounded in real systems, the rest of Module 4 builds
the algorithms. **4b** develops the learning foundations, Monte Carlo and
temporal-difference methods, that make credit assignment tractable from
sampled experience. **4c** scales value estimation into deep RL. **4d** turns to
policy-gradient methods, where we derive PPO and GRPO, the workhorses of LLM
post-training. **4e** covers DPO and the family of direct-alignment methods that
optimize from preferences without an explicit reward model or RL loop. And
**4f** returns to the agent loop itself, examining reflective agents and
persistent memory, the place where the engineered scaffold starts to take on
some of the adaptivity we have, so far, reserved for the weights.

## References

- Vaswani, A., et al. (2017). Attention is all you need. *NeurIPS*.
- Schulman, J., et al. (2017). Proximal policy optimization algorithms. arXiv:1707.06347.
- Shao, Z., et al. (2024). DeepSeekMath: Pushing the limits of mathematical reasoning in open language models (GRPO). arXiv:2402.03300.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
