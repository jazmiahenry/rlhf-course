# Module 4a: From MDP Theory to Real Agents

> **What you'll get out of this:** the bridge from Module 3's clean math to how
> agents are *actually* built in 2026, and an honest answer to "where is the RL,
> really?"

## The gap nobody warns you about

Module 3 built beautiful machinery: states, actions, rewards, policies, Bellman
equations. All exact. All silent about silicon. The formalism tells you what an
optimal policy *is*. It does not tell you what a "state" is when your agent is a
language model reading a 200,000-token context, or what an "action" is when it emits
a JSON tool call.

This lesson is the bridge before the algorithms. We map $(S, A, R, \pi)$ onto how
real agent systems are built, and we're honest about where the MDP framing is
load-bearing versus where it's a metaphor you should retire. (The old version of this
lesson claimed production orchestration systems train value networks online to pick
tools. That was false, and the falsehood hid the most useful thing you can know:
which parts of an agent are *learned* and which are *engineered*.)

## What you'll be able to do

- Map each MDP piece onto a concrete part of an LLM agent.
- Explain why the same agent is an MDP at two resolutions, and pick the right one.
- Name where reward signals actually come from in 2026, and the failure mode of the
  fragile one.
- Say which parts of a production agent are trained with RL and which are hand-built.

## State: the context window is the history

Recall the POMDP from Module 3A: the agent never sees the true state $s_t$, only an
observation $o_t$, so the right move is to condition on the whole history $h_t =
(o_0, a_0, \dots, o_t)$, and the belief over states is a sufficient statistic for it.

An LLM agent is a POMDP agent in exactly this sense, and **the context window is its
history.** Everything it conditions on (system prompt, messages, prior tool calls and
results, retrieved docs, scratchpad) is the token sequence. That sequence *is* $h_t$.
There's no hidden state surviving outside the context; the agent's entire memory of
the episode is the text it can currently see. So context management (what to truncate,
summarize, reorder) is literally choosing the belief the policy gets to act on. Not a
footnote, a modeling decision.

Two refinements. First, the model doesn't eat raw tokens; each is mapped to a
**learned embedding**, and self-attention (Vaswani et al., 2017) computes the state
features. This is the quiet revolution: twenty years ago you hand-engineered a small
fixed feature vector. Now the state representation is *learned*, high-dimensional, and
trained end-to-end. Second, the context is **structured**: a system prompt fixes
constraints, tool schemas declare actions, retrieved docs inject knowledge, memory
files persist across episodes. From the MDP's view they're all just $o_t$, but each is
a different channel with different freshness and trust.

## Action: typed tool calls, at two resolutions

A textbook action set is discrete labels: move left, move right. A real agent's
action space is **typed**: structured tool calls validated against a JSON schema,
increasingly over a standard interface (MCP). The schema defines the action space the
way the rules of a board game define legal moves.

Here's the move the rest of Module 4 depends on. **The same agent is an MDP at two
resolutions, and which one you use depends on what you're optimizing.**

- **Token-level.** Zoom all the way in. State = tokens so far, action = next token,
  policy = the model's next-token distribution. Episodes are hundreds to thousands of
  steps. This is the lens for **post-training** the model (RLHF, PPO, GRPO, DPO all
  live here, because the thing being updated is the weights that produce tokens).
- **Tool-level.** Zoom out. One full turn (reason, then emit a tool call) is a single
  action; the tool result is the next observation. Episodes are a handful to a few
  dozen steps. This is the lens for **orchestration**: reasoning about whether a
  multi-step task gets done.

Neither is "more correct." They're two valid abstractions of one system. The cost of
being sloppy is real: "we trained the policy with PPO" is precise at the token level
and a category error at the tool level if no weights are being updated there. Always
ask: *at what resolution, and is anything actually being learned there?*

## Reward: where the signal really comes from

Module 3 treated $R(s,a)$ as given. In real systems, *manufacturing* a usable reward
is often the hardest part of the whole job. In 2026 it comes from a few sources, each
with a different trust profile:

- **Verifiable outcomes (RLVR).** A program checks it: tests pass, the answer matches,
  the SQL returns the right rows. Can't be gamed by fooling a model, only by being
  correct. Limit: most things we want (a kind explanation, a tasteful refactor) have
  no verifier.
- **Learned reward models from preferences.** The Module 3E pipeline: annotators
  rank, a model learns to score, the score is the reward. Buys coverage over fuzzy
  goals, at the price of a reward the policy will learn to exploit.
- **LLM-as-judge.** Prompt a capable model to score outputs. Fast and scalable, and a
  measurement instrument with *systematic error*: position bias (prefers first or
  last), verbosity bias (longer = better, regardless), self-preference (likes its own
  style). A judge is evidence to calibrate and audit, never an oracle. Don't optimize
  hard against a single judge without checking what the policy actually learns.

And one structural choice cuts across all of it: **when does reward arrive?** A
verifier that fires only "tests passed" at the end of a long episode gives you one
scalar for dozens of decisions, the sparse, long-horizon credit-assignment problem
from Module 3D at full strength. Denser per-step signals make credit assignment easier
but reward a *proxy* for progress. Managing that tension is most of the craft in 4b
through 4d.

## Policy: what's learned vs what's engineered

A policy maps states to actions. In an LLM agent it's a loop: **observe** the context,
**reason**, **act** (emit a tool call or final answer), fold the result back in,
repeat. That loop plus its prompts and tools *is* the policy at the tool level.

Now the honest core of this lesson. Inside that policy, two very different things do
the work:

- **What's learned:** the model weights, shaped by pre-training and **post-training**
  (SFT plus RL methods like PPO or GRPO). This is where real RL lives.
- **What's engineered:** the scaffold, the system prompt, the tool set, the control
  flow, retries, memory policy, when to stop. Written by humans, fixed at deployment.
  No gradient touches it while the agent runs.

So when we say "agent," we almost always mean **an engineered policy wrapped around a
learned model.** At runtime, the agent *executes* a fixed policy. It is not running
gradient descent between turns, not estimating a value function online, not training a
DQN to pick tools. Each request is an inference pass through frozen weights.

This isn't a knock, it's the correct mental model, and it's freeing once you have it.
It tells you where your levers are: bad reasoning? Engineer the scaffold (prompts,
tools, context) *or* change the weights via post-training. Totally different projects,
different costs, different risks. RL enters by default in exactly one place:
**post-training the model, offline, before deployment.** Knowing which world you're in
is the first question to ask of any agent system, including yours.

## The takeaways

- The **context window is the POMDP history**; **learned embeddings** are the state
  features; the context is **structured** in ways the policy depends on.
- The action space is **typed**, and the agent is an MDP at **two resolutions**:
  tokens for post-training, tool calls for orchestration. State the resolution first.
- Reward comes from **verifiable outcomes, learned reward models, and LLM-judges** (a
  biased instrument). **Sparse, episode-level** reward makes credit assignment bite.
- A production agent is an **engineered policy over a learned model.** Real RL lives in
  **offline post-training**, not at runtime.

## Think about it

1. Take an agent you've used. At what resolution is anything being *learned*, if at
   all? Or is it a fixed policy over a frozen model?
2. Your agent uses an LLM-as-judge for reward. Name two ways the policy could learn to
   game the judge instead of doing the task.

## Next

Module 4b builds the learning foundations, Monte Carlo and temporal-difference
methods, that make credit assignment from sampled experience tractable.

## References

- Vaswani, A., et al. (2017). Attention is all you need. *NeurIPS*.
- Schulman, J., et al. (2017). Proximal Policy Optimization Algorithms. arXiv:1707.06347.
- Shao, Z., et al. (2024). DeepSeekMath (GRPO). arXiv:2402.03300.
- Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
