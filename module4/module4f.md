# Module 4f: Reflective Agents — Self-Critique, Memory, and Test-Time Improvement

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the Reflexion mechanism precisely and walk through one concrete example.
- State honestly when self-correction works and when it does not, with the
  supporting evidence.
- Distinguish *prompted* reflection (a scaffold) from *trained* reflection
  (reasoning models), and connect the latter to RLVR.
- Name the three layers of agent memory and the design decisions that govern
  each.
- Describe the generate–verify–revise loop and its relationship to RLVR.

---

## Introduction: Improvement Without Weight Updates

Everything in Module 4 so far changes a model by changing its *weights* — MC/TD
estimates feed value functions (4b, 4c), policy gradients update the policy (4d),
DPO optimizes directly on preferences (4e). This final lesson asks a different
question: **can an agent improve within a task, at inference time, without any
gradient step at all?** The answer is a qualified yes — and the qualifications
are exactly the teaching content. We replace the old lesson's hand-waving about
"recursive self-improvement" with the concrete mechanisms that actually work in
2026, and an honest account of where each one breaks.

## Reflexion: Verbal Reinforcement Learning

The cleanest instance of inference-time improvement is **Reflexion** (Shinn et
al., 2023). Its insight: an agent that fails a task can convert the failure into
a *natural-language lesson* and carry that lesson into its next attempt — gaining
the benefit of a learning signal without ever updating weights. The authors call
this "verbal reinforcement learning" because the improvement signal lives in the
context, not in the parameters.

The mechanism has three roles:

1. **Actor** — generates a trajectory (a solution attempt) from the current
   context, which includes any prior reflections.
2. **Evaluator** — scores the trajectory. Crucially, this is usually an
   *external* signal: unit tests, a task-success check, a reward function.
3. **Self-reflection model** — reads the trajectory and the evaluator's verdict
   and writes a short verbal lesson ("the test failed because the function
   didn't handle the empty-list case") into an **episodic memory buffer**.

On the next attempt, the actor's prompt includes the accumulated reflections, so
it does not repeat the same mistake.

### A concrete walkthrough

A coding agent is asked to implement a function and is graded by a hidden unit
test suite.

- **Attempt 1.** The actor writes a plausible implementation. The evaluator runs
  the tests: 5 of 6 pass; the failing one feeds an empty list. The self-reflection
  model writes: *"My solution crashed on empty input. Next time, handle the
  empty-list case before indexing."* This sentence goes into the buffer.
- **Attempt 2.** The actor's prompt now contains that reflection. It adds an
  empty-list guard. All 6 tests pass.

No weights changed. The improvement came entirely from a verbal lesson carried
forward in context — credit assignment performed in natural language.

### The real limits

Reflexion is powerful but bounded, and a practitioner must know the boundaries:

- **It needs an evaluator.** The whole loop hinges on a signal that tells the
  agent it failed — tests, a verifier, a graded environment. With no external
  signal, there is nothing honest to reflect on (see the next section).
- **It needs a reset-able task.** Reflexion improves performance across *retries
  of the same task*; it is within-task learning, not durable across-task skill
  acquisition. The lessons live in the episode's buffer, not in the weights.
- **It is bounded by context.** Reflections consume context window; on long tasks
  with many failures the buffer competes with everything else the agent must
  attend to.

## Self-Correction, Honestly

A tempting generalization of Reflexion is: *just ask the model to critique and
fix its own answer, no external evaluator needed.* The evidence says be careful.
Huang et al. (2024), "Large Language Models Cannot Self-Correct Reasoning Yet,"
found that **intrinsic** self-correction — revising an answer using only the
model's own judgment, with no external feedback — often **fails to improve and
can degrade** performance: the model is as likely to "correct" a right answer
into a wrong one as the reverse, because the same limitations that produced the
error also limit its self-judgment.

The dividing line is the **source of the signal**. Self-correction works well
when grounded in something outside the model's own opinion — tests that pass or
fail, a tool that returns an error, a retrieval that contradicts a claim, a
separate verifier or judge. It works poorly when the model is both author and
sole grader. This is the same lesson Module 4a taught about LLM-as-judge: a model
grading itself inherits its own blind spots. Design reflective loops around an
*external* check whenever you can; treat unverified self-critique as a weak prior,
not a fix.

## Trained Reflection: From Scaffold to Weights

Reflexion is a *scaffold* — a loop you build around a fixed model. The major
shift of 2024–2025 was to move that loop *inside* the model via training.
**Reasoning models** (OpenAI's o1, 2024; DeepSeek-R1, Guo et al., 2025)
internalize the generate–evaluate–revise cycle into an extended chain of thought:
the model learns, through reinforcement learning on verifiable rewards (the RLVR
of Module 4d), to second-guess itself, check intermediate steps, and backtrack —
all within a single forward generation, without an external scaffold orchestrating
retries.

The contrast is worth stating sharply:

- **Prompted reflection (Reflexion):** improvement signal carried in context,
  weights frozen, requires an external loop and evaluator at inference time.
- **Trained reflection (reasoning models):** improvement signal baked into
  weights by RL during post-training, so the deliberation happens natively at
  inference.

The field moved from *scaffold* to *training signal* because the trained version
is more reliable and self-contained — but the two compose. A reasoning model can
still sit inside a Reflexion loop with external tests, and often that combination
is the strongest production pattern.

## Agent Memory Architectures

Reflexion's "episodic buffer" is one slice of a larger design space. Practical
agents juggle three layers of memory, each with different lifetime and update
dynamics:

- **Working memory — the context window.** Everything the model currently
  attends to. Fast, capacity-bounded, gone when the episode ends. (This is the
  POMDP history $h_t$ from Module 4a.)
- **Episodic memory — session logs and reflection buffers.** A record of *what
  happened* across recent steps or attempts: Reflexion's lessons, prior tool
  results, a running scratchpad. Persists within a task or session.
- **Persistent / semantic memory — files and vector stores.** Durable facts and
  artifacts that outlive any single session: a notes file, a knowledge base, a
  retrieval index of past work.

The hard part is not storage but **memory-write policy**: *what* to save, *when*
to consult it, and *how to avoid acting on stale or wrong memories*. A memory
that records a mistaken conclusion and then keeps surfacing it is worse than no
memory. These are genuine design decisions with no universal default — they
depend on how often the world changes and how costly a wrong recollection is.

There is a clean conceptual link back to Module 3d. A reflection is a
**credit-assignment artifact**: it attributes an episode's failure to a specific
earlier decision and records the correction. Where Module 3d framed credit
assignment as a learning-time problem (which step in the trajectory caused the
bad outcome?), reflective memory performs the same attribution at inference time,
in natural language, and stores the result for reuse.

## Verification Loops in Practice

Pulling the threads together, the production-reliable pattern for reflective
agents is **generate → verify → revise**, with the *verify* step grounded in
something external:

- **Generate** a candidate (code, an answer, a plan).
- **Verify** it with an external checker: run the tests, validate against a
  schema, execute the tool and inspect the result, consult a judge (knowing its
  biases from 4a).
- **Revise** using the verifier's concrete feedback, then repeat until the check
  passes or a retry budget is exhausted.

This is the same shape as RLVR (Module 4d), and the symmetry is the point: a
verifier that provides the reward signal *at training time* can provide the
revision signal *at inference time*. Tests that grade a coding model during RL
post-training are the same tests that drive a Reflexion loop in deployment. Build
the verifier once; use it in both places.

## Course Capstone: The Whole Arc

You now have the full picture this course set out to teach.

- **Module 1** named the gap: deployed AI fails not because it misunderstands
  users (state space) but because it chooses what to do implicitly, inconsistently,
  and unmeasurably (action space) — with documented costs and a historic talent
  shortage for the people who can fix it.
- **Module 2** gave the design methods: deriving value-aligned scoring from user
  values, and the two-stage architecture that pre-filters information before an
  LLM acts on it.
- **Module 3** built the RL formalism: state and observability, action spaces,
  stochastic rewards and risk (CVaR), trajectory-level alignment and credit
  assignment, preference-learned policies, and curriculum design.
- **Module 4** built the algorithms: MC/TD foundations (4b), value-based deep RL
  (4c), policy gradients → PPO → GRPO (4d), DPO and the direct-alignment family
  (4e), and reflective agents (4f).

What can you now build? An agent whose *information* is value-scored before it
acts (Module 2); whose *policy* you can reason about as an MDP at the right
resolution, with rewards you source deliberately (4a); that you can *improve*
either by post-training (4c–4e) or by an inference-time verify–revise loop (4f);
and whose behavior you can *measure* — because every step of this course insisted
that a claim you cannot check is not a claim. That measurement discipline, more
than any single algorithm, is what separates the AI systems that make it into the
documented successful minority from the ones that don't.

## Key Takeaways

- **Reflexion** improves an agent across retries by writing verbal lessons from
  an *external* evaluator into an episodic buffer — verbal RL, no weight updates;
  bounded to reset-able tasks and by context.
- **Intrinsic self-correction is unreliable** (Huang et al., 2024). Ground
  reflection in an external signal — tests, verifiers, tools, judges — not the
  model's own opinion.
- **Reasoning models** are the *trained* version of reflection: RLVR bakes the
  generate–evaluate–revise loop into the weights, so deliberation is native at
  inference.
- Agent memory has three layers — **working** (context), **episodic** (logs,
  reflections), **persistent** (files, vector stores) — and the real work is the
  **write/consult policy**. A reflection is an inference-time credit-assignment
  artifact (cf. Module 3d).
- The production-reliable shape is **generate → verify → revise** with an external
  verifier — the same verifier you use for RLVR at training time.

## Where to Go From Here

- **Build and measure.** Take the multi-tool agent idea from Module 2 and wire it
  to a real eval harness; the discipline of scoring every change is the whole
  point.
- **Post-train something small.** Run DPO (4e) on a preference set, or GRPO (4d)
  on a verifiable task, on a small open model — the algorithms are most real once
  you have watched a reward curve move.
- **Read the primary sources.** Every lesson cites them; the papers are where the
  details and the honest caveats live.

## References

- Shinn, N., et al. (2023). Reflexion: Language agents with verbal reinforcement learning. *NeurIPS*.
- Madaan, A., et al. (2023). Self-Refine: Iterative refinement with self-feedback. *NeurIPS*.
- Huang, J., et al. (2024). Large language models cannot self-correct reasoning yet. *ICLR*.
- OpenAI (2024). Learning to reason with LLMs (o1).
- Guo, D., et al. (2025). DeepSeek-R1: Incentivizing reasoning capability in LLMs via reinforcement learning. arXiv:2501.12948.
- Park, J. S., et al. (2023). Generative agents: Interactive simulacra of human behavior. *UIST*.
