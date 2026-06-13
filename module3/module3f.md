# Module 3F: Curriculum Learning

> **What you'll get out of this:** how to *train* an agent so the aligned policies
> from 3E actually emerge, and an honest account of what a curriculum does and
> doesn't buy you, backed by a real experiment in the capstone.

## The idea, and the honest caveat up front

Curriculum learning is simple to say: don't throw the agent at the hardest task
cold. Start it on easy versions, and ramp up difficulty as it succeeds. Like
teaching anything.

But let's be honest about what that buys you, because the field is full of
overclaiming. Curriculum learning is a **heuristic** with strong empirical support
in specific settings (Bengio et al., 2009; Narvekar et al., 2020), **not** a
technique with general guarantees. Whether a curriculum helps depends on the task,
the stage decomposition, and the transfer between stages. A badly designed
curriculum can *hurt*, by overfitting the easy stages or wasting budget not
practicing the real task. There is no theorem that makes alignment mathematically
guaranteed, and you should treat any claim that there is, here or in any paper,
with skepticism.

What the staged structure *does* demonstrably buy you:

1. **Interpretable checkpoints** where you measure specific capabilities before
   adding complexity.
2. **Safety constraints stay enforced** while capability grows, because you check
   them at every stage gate.

That's real and useful. It's also more modest than "curricula make everything
better," and the honesty is the point.

## What you'll be able to do

- Design a staged curriculum that builds capability without dropping safety.
- Decide whether a curriculum is even the right tool for your task.
- Connect the idea to the one place we actually *measured* it (the RL capstone).

## A five-stage curriculum for our agent

The progression goes from "learn one thing cleanly" to "stay aligned under
attack." Each stage restricts the world, then loosens it.

**Stage 1: Value recognition.** Simple problems with one clear right tool. Low
reward variance, high signal, so the basic value-to-action associations land
cleanly. Example: *"What's the capital of France?"* low stakes, factual, academic
search wins. The lesson: accuracy is the priority for factual queries.

**Stage 2: Multi-objective trade-offs.** Now two values compete, and context
decides the winner. Example: *"Latest vaccine data,"* high stakes *and* time
pressure, so speed and accuracy fight, and the right move is a fast initial search
followed by verification. The lesson: context sets the priorities.

**Stage 3: Sequential decisions.** Multi-step trajectories where early actions
shape later possibilities (this is Module 3D's territory). Example: a controversial
topic, gather broadly, detect bias, then go deep, with objectivity held across the
whole sequence.

**Stage 4: Dynamic context.** The situation shifts mid-task. The agent has to
adapt its value weights on the fly without losing consistency.

**Stage 5: Adversarial robustness.** Inputs crafted to push the agent off its
values (the prompt-injection threat from 1B). The agent has to hold alignment under
pressure.

At every gate, you check two things before promoting the agent: did it learn the
new capability, *and* is it still satisfying the safety constraints? Fail the second
and it doesn't advance, no matter how capable it got.

## A note on the companion notebook

`RL_Alignment_Part2` implements a **condensed 4-stage** version of this so it runs
in minutes on a laptop: Single-Tool Mastery, Sequential Decisions, Stochastic
Adaptation, Adversarial Robustness. The mapping: notebook stage 1 covers lesson
stages 1-2, notebook stages 2-4 line up with lesson stages 3-5. The episode counts
in this lesson describe a production-scale curriculum; the notebook uses 50-80
episodes per stage for demonstration.

## When does a curriculum actually pay off? (We measured.)

This is the part most courses skip, and it's the most important. We didn't just
*assert* that curricula help. In the **RL capstone**, we ran real PPO, across
multiple seeds, with and without a curriculum, and looked at what actually happened.
The result is sharp:

- On an **easy** task, plain training already wins, and the curriculum **does not
  help** (and a curriculum that withholds the target can even hurt).
- On a **hard sparse-exploration** task, plain training is a **coin flip**, it only
  learns if early random exploration happens to find the goal, so it fails outright
  on some seeds. The curriculum (start near, expand) makes it **reliable**, solving
  the task on essentially every seed.

So the honest takeaway isn't "curricula are faster." It's "**curricula make hard
exploration problems reliably solvable, where cold training succeeds only by luck,
and they do nothing on easy tasks.**" Read the capstone for the actual numbers and
the multi-seed plot. That experiment is what turns this lesson from a story into a
verified claim.

## The takeaways

- Curriculum learning is **train easy-to-hard**, a heuristic with real but *bounded*
  benefits, not a guarantee.
- What staging genuinely buys you: **interpretable checkpoints** and **enforced
  safety constraints** as capability grows.
- Design five stages from **value recognition** up to **adversarial robustness**,
  and gate promotion on both new capability and continued safety.
- The capstone **measured** the payoff: curricula give **reliability on hard sparse
  tasks**, not speed on easy ones. Measure before you claim it for your task.

## Think about it

1. For a task you care about, is the bottleneck *exploration* (can't find reward) or
   something else (credit assignment, stability)? A curriculum only helps the first.
   How would you tell?
2. Stage gates check capability *and* safety. Why is gating on capability alone a
   trap?

## Next

Module 4 goes under the hood of everything Module 3 set up: the actual RL
algorithms, from Monte Carlo and temporal-difference learning through DQN, PPO,
GRPO, DPO, and a capstone that puts the whole thing to an honest, measured test.

## References

- Bengio, Y., et al. (2009). Curriculum Learning. *ICML*.
- Narvekar, S., et al. (2020). Curriculum Learning for Reinforcement Learning Domains. *JMLR*.
- Florensa, C., et al. (2017). Reverse Curriculum Generation for Reinforcement Learning. *CoRL*.
