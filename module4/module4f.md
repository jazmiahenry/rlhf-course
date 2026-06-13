# Module 4f: Reflective Agents (and the Course, Pulled Together)

> **What you'll get out of this:** how an agent can improve *at inference time*,
> without any weight update, the honest evidence on when that works, and a closing map
> of everything you've learned.

## Improvement without gradients

Every method in Module 4 so far changes a model by changing its *weights*. This last
lesson asks a different question: can an agent get better *within a task, at inference
time, with no gradient step at all?*

The answer is a qualified yes, and the qualifications are the whole content. We're
replacing the old lesson's hand-waving about "recursive self-improvement" with the
concrete mechanisms that actually work in 2026, and an honest account of where each one
breaks.

## What you'll be able to do

- Explain the Reflexion mechanism and walk one concrete example.
- Say honestly when self-correction works and when it doesn't, with the evidence.
- Distinguish *prompted* reflection (a scaffold) from *trained* reflection (reasoning
  models).
- Name the three layers of agent memory and the decision that governs them.

## Reflexion: verbal reinforcement learning

The cleanest case of inference-time improvement is **Reflexion** (Shinn et al., 2023).
The insight: an agent that fails can turn the failure into a *natural-language lesson*
and carry it into the next attempt, getting the benefit of a learning signal without
touching the weights. The authors call it "verbal reinforcement learning" because the
improvement lives in the context, not the parameters.

Three roles:

1. **Actor** generates a trajectory (a solution attempt), conditioned on any prior
   reflections.
2. **Evaluator** scores it, usually an *external* signal: unit tests, a success check, a
   reward function.
3. **Self-reflection model** reads the trajectory and the verdict and writes a short
   lesson into an **episodic memory buffer**.

Next attempt, the actor's prompt includes the accumulated reflections, so it doesn't
repeat the mistake.

**Walk it.** A coding agent is graded by hidden unit tests.

- *Attempt 1:* writes a plausible function. Tests: 5 of 6 pass; the failure feeds an
  empty list. The reflection model writes: *"Crashed on empty input. Next time, handle
  the empty-list case before indexing."* Into the buffer.
- *Attempt 2:* the prompt now contains that reflection. The actor adds an empty-list
  guard. All 6 pass.

No weights changed. The improvement was a verbal lesson carried forward, credit
assignment done in natural language.

**The real limits**, because they matter:

- It needs an **evaluator**. The whole loop hinges on a signal that says you failed. No
  external signal, nothing honest to reflect on.
- It needs a **resettable task**. Reflexion improves *retries of the same task*. It's
  within-task learning, not durable skill, the lessons live in the episode's buffer,
  not the weights.
- It's **bounded by context**. Reflections eat context window; long tasks with many
  failures crowd everything else out.

## Self-correction, honestly

A tempting generalization: just ask the model to critique and fix its own answer, no
external evaluator. The evidence says be careful. Huang et al. (2024), *Large Language
Models Cannot Self-Correct Reasoning Yet*, found that **intrinsic** self-correction
(revising using only the model's own judgment) often *fails to improve and can
degrade* performance. The model is about as likely to "correct" a right answer into a
wrong one, because the same limits that produced the error also limit its self-judgment.

The dividing line is the **source of the signal.** Self-correction works when it's
grounded in something outside the model's own opinion: tests, a tool error, a retrieval
that contradicts a claim, a verifier. It works poorly when the model is both author and
sole grader. (Same lesson as LLM-as-judge in 4a: a model grading itself inherits its
own blind spots.) Build reflective loops around an *external* check whenever you can;
treat unverified self-critique as a weak prior, not a fix.

## Trained reflection: from scaffold to weights

Reflexion is a *scaffold* you build around a fixed model. The big shift of 2024-25 was
to move that loop *inside* the model through training. **Reasoning models** (OpenAI's
o1, 2024; DeepSeek-R1, Guo et al., 2025) internalize the generate-evaluate-revise cycle
into an extended chain of thought, learning through RL on verifiable rewards (the RLVR
of 4d) to second-guess, check steps, and backtrack, all in a single generation, no
external scaffold orchestrating retries.

The contrast, stated sharply:

- **Prompted reflection (Reflexion):** signal in context, weights frozen, needs an
  external loop at inference.
- **Trained reflection (reasoning models):** signal baked into weights by RL, so the
  deliberation is native at inference.

The field moved from scaffold to training signal because the trained version is more
reliable and self-contained. But they compose: a reasoning model inside a Reflexion
loop with real tests is often the strongest production pattern.

## Agent memory, in three layers

Reflexion's buffer is one slice of a bigger design space. Practical agents juggle three
kinds of memory:

- **Working memory:** the context window. Fast, bounded, gone when the episode ends.
  (The POMDP history from 4a.)
- **Episodic memory:** session logs, reflection buffers. What happened recently;
  persists within a task.
- **Persistent / semantic memory:** files, vector stores. Durable facts that outlive any
  session.

The hard part isn't storage, it's the **write policy**: what to save, when to consult
it, and how to avoid acting on stale or wrong memories. A memory that records a mistaken
conclusion and keeps resurfacing it is worse than no memory. And note the link back to
Module 3D: a reflection *is* a credit-assignment artifact, it attributes a failure to an
earlier decision and records the fix, the same attribution problem, done at inference in
natural language.

The production-reliable shape pulls it together: **generate → verify → revise**, with an
*external* verifier (tests, schema validators, judges). It's the same shape as RLVR, and
that symmetry is the point: the verifier that gives the reward at *training* time can
give the revision signal at *inference* time. Build the verifier once, use it in both.

## The whole course, in one picture

You now have everything this course set out to teach.

- **Module 1** named the gap: deployed AI fails not because it misunderstands users
  (state space) but because it chooses what to do implicitly, inconsistently, and
  unmeasurably (action space), with documented costs and a historic talent shortage for
  the people who can fix it.
- **Module 2** gave the design methods: deriving value-aligned scoring from user values,
  and the two-stage score-then-generate architecture.
- **Module 3** built the RL formalism: state and observability, action spaces, stochastic
  rewards and risk (CVaR), trajectory-level alignment, preference-learned policies, and
  curricula.
- **Module 4** built the algorithms: MC/TD foundations, value-based deep RL, policy
  gradients into PPO and GRPO, DPO and the direct-alignment family, and reflective
  agents, all stress-tested in a capstone that measured what's actually true.

What can you build now? An agent whose *information* is value-scored before it acts; whose
*policy* you can reason about as an MDP at the right resolution, with rewards you source
deliberately; that you can *improve* by post-training or by an inference-time
verify-revise loop; and whose behavior you can *measure*, because every lesson here
insisted that a claim you can't check is not a claim. That measurement discipline, more
than any single algorithm, is what separates the AI systems that make it into the
documented successful minority from the ones that don't.

## The takeaways

- **Reflexion** improves an agent across retries by writing verbal lessons from an
  *external* evaluator into an episodic buffer. No weight updates; bounded to resettable
  tasks and by context.
- **Intrinsic self-correction is unreliable** (Huang et al., 2024). Ground reflection in
  an external signal, not the model's own opinion.
- **Reasoning models** are the *trained* version of reflection: RLVR bakes the
  generate-evaluate-revise loop into the weights.
- Agent memory has three layers (**working, episodic, persistent**); the real work is the
  **write/consult policy**. A reflection is an inference-time credit-assignment artifact.
- The reliable shape is **generate → verify → revise** with an external verifier, the
  same verifier you'd use for RLVR.

## Where to go from here

- **Build and measure.** Take the multi-tool agent from Module 2, wire it to a real eval
  harness, and score every change. The discipline is the whole point.
- **Post-train something small.** Run DPO (4e) on a preference set, or GRPO (4d) on a
  verifiable task, on a small open model. The algorithms get real once you've watched a
  reward curve move. (The car-navigation capstone is a ready-made, runnable place to
  start.)
- **Read the primary sources.** Every lesson cites them. The papers are where the details
  and the honest caveats live.

## References

- Shinn, N., et al. (2023). Reflexion: Language Agents with Verbal Reinforcement Learning. *NeurIPS*.
- Huang, J., et al. (2024). Large Language Models Cannot Self-Correct Reasoning Yet. *ICLR*.
- OpenAI (2024). Learning to reason with LLMs (o1).
- Guo, D., et al. (2025). DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning. arXiv:2501.12948.
- Park, J. S., et al. (2023). Generative Agents. *UIST*.
