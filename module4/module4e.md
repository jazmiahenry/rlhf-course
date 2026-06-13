# Module 4e: DPO and the Direct-Alignment Family

> **What you'll get out of this:** the method that made preference-tuning
> accessible to everyone, stated *correctly* (it's misdescribed constantly), plus
> the family of methods that grew out of it and where the field actually is in 2026.

## The thing everyone gets wrong about DPO

Module 4d gave you RLHF: train a reward model from preferences, then optimize the
policy against it with PPO and a KL penalty. Two stages, two models, an RL loop.
Powerful, and a pain.

Then DPO showed up, and it's the most misexplained method in modern ML. You'll read
that "DPO trains a preference model and uses it to guide the policy." That is
**exactly wrong**, and getting it right is the point of this lesson. Let's be
precise.

## What you'll be able to do

- State what DPO actually eliminates (and write its loss).
- Explain why it took over open-weight preference tuning, and its real trade-off.
- Place DPO in the 2024-26 lineage: KTO, SimPO, ORPO, GRPO, RLVR.
- Describe Constitutional AI honestly, as RLAIF, not as governance hand-waving.

## DPO: no reward model, no RL loop

Recall the classic RLHF pipeline (Christiano et al., 2017; Ouyang et al., 2022):
(1) train a separate **reward model** on preference pairs, then (2) optimize the
policy against it with RL, KL-anchored to a reference.

**DPO's entire contribution (Rafailov et al., 2023, "Your Language Model is Secretly
a Reward Model") is deleting stage 1.** No separate reward model. No preference model.
Ever. Rafailov et al. showed the RLHF objective has a closed-form optimal policy, and
that the reward can be re-expressed *in terms of the policy itself*. Plug that
re-parameterization into the Bradley-Terry likelihood (the one from Module 3E) and
you get a plain supervised loss, directly on the policy:

$$\mathcal{L}_{\text{DPO}}(\pi_\theta; \pi_{\text{ref}}) = -\mathbb{E}_{(x, y_w, y_l)}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y_w \mid x)}{\pi_{\text{ref}}(y_w \mid x)} - \beta \log \frac{\pi_\theta(y_l \mid x)}{\pi_{\text{ref}}(y_l \mid x)}\right)\right]$$

Decode it: $(x, y_w, y_l)$ is a prompt with a preferred ($y_w$) and dispreferred
($y_l$) response, $\pi_{\text{ref}}$ is the frozen reference (usually the SFT model),
and $\beta$ controls how far the policy may drift, the same job the KL penalty did in
PPO-based RLHF. Training is **gradient descent on preference pairs.** No reward model,
no rollouts, no RL loop. That's why DPO became the default preference-tuning method
for open-weight models: dramatically simpler and more stable than the PPO pipeline,
at comparable quality.

The honest trade-off: DPO is stuck with the preference pairs you have. It can't
explore beyond them the way an RL loop can, and it inherits the Bradley-Terry
assumption that preferences are pairwise and consistent.

## The post-DPO family (2024-2026)

DPO opened a whole lineage, and a current course has to place it in context:

- **Variants on the loss.** KTO (learns from binary good/bad labels, not pairs),
  SimPO (drops the reference model entirely), ORPO (folds preference optimization
  into SFT, one stage), IPO (fixes a DPO overfitting pathology). Same spirit:
  preference optimization without an RL loop.
- **GRPO** (Shao et al., 2024) went *back* to real RL but ditched PPO's learned value
  function for a group-relative baseline (Module 4d). Cheap enough to scale, it
  powered the reasoning-model wave (DeepSeek-R1, 2025).
- **RLVR** (RL from verifiable rewards) threw out the *learned* reward signal where
  outputs can be **checked**: math answers, passing tests, valid tool calls. When the
  reward is a verifier instead of a model, reward hacking has far less room. As of
  2026 this is the dominant paradigm for training reasoning and agentic capability,
  with preference methods layered on for style and safety.

> **The arc to carry with you:** RLHF (learned reward + RL) → DPO (no reward model,
> no RL) → GRPO/RLVR (RL again, but with cheap baselines and verifiable rewards).
> Each step traded modeling machinery for either simplicity or verifiability. That's
> the whole story of post-training in one line.

## Constitutional AI, the honest version

Now the other method people invoke and rarely explain: **Constitutional AI** (Bai et
al., 2022, Anthropic). It is *not* a "governance framework for orchestration." It's a
concrete training technique, and here's what it actually does.

The expensive part of RLHF is collecting human labels, especially for harmlessness,
where you're asking people to read and rank potentially toxic content. Constitutional
AI replaces a lot of that human labeling with **AI feedback (RLAIF)**, guided by a
short written **constitution** (a list of principles). The loop:

1. The model produces a response.
2. The model **critiques its own response** against a constitutional principle ("point
   out ways this is harmful or unethical").
3. The model **revises** the response to fix it.
4. You train on the revised responses (supervised), then run preference tuning where
   the *preference labels come from an AI judging against the constitution*, not from
   a human.

So the constitution is a handful of natural-language principles, and the model does
the critiquing and labeling. It scales oversight by spending model compute instead of
human hours, and it was used to train real assistants for harmlessness. That's the
honest description: **a critique-and-revise loop plus AI-generated preference labels**,
not a magic rulebook bolted onto an agent.

## How these fit together in 2026

You don't pick one. A modern post-training stack tends to layer them:

- **SFT** to get a baseline instruction-follower.
- **RLVR** (GRPO-style) for *capability* on verifiable tasks (math, code, reasoning).
- **Preference tuning** (DPO-family or RLHF) for *style, helpfulness, and tone*.
- **Constitutional / RLAIF** signals for *harmlessness* at scale.

The throughline back to Module 3E: every one of these, except the pure verifier
reward, traces to the **Bradley-Terry preference likelihood**. DPO uses it directly.
RLHF uses it to build a reward model. Constitutional AI uses it with AI-generated
labels. Learn that likelihood once and the whole zoo makes sense.

## The takeaways

- **DPO deletes the reward model.** It re-expresses reward in terms of the policy and
  optimizes a supervised loss on preference pairs. No reward model, no RL loop. (If
  you remember one correction from this course, make it this one.)
- DPO won open-weight tuning on **simplicity and stability**; its trade-off is being
  bounded by the pairs you have.
- The lineage runs **RLHF → DPO → GRPO/RLVR**, plus a family of loss variants (KTO,
  SimPO, ORPO, IPO).
- **Constitutional AI is RLAIF**: a critique-and-revise loop with AI-generated
  preference labels against a written constitution. It scales harmlessness oversight,
  it isn't governance hand-waving.
- Real stacks **layer** these, and almost all of it traces back to one Bradley-Terry
  likelihood.

## Think about it

1. DPO needs a frozen reference policy $\pi_{\text{ref}}$ and a $\beta$. What goes
   wrong if $\beta$ is tiny? What goes wrong if it's huge? (Tie it back to the KL term
   in 4d.)
2. RLVR can't give a reward for "write a kind, well-structured explanation." DPO/RLHF
   can't *verify* a math answer. Why does a real system need both?

## Next

Module 4f closes the course with **reflective agents**: getting an agent to improve
*at inference time* through self-critique and memory, and the honest evidence on when
that works and when it doesn't.

## References

- Rafailov, R., et al. (2023). Direct Preference Optimization: Your Language Model is Secretly a Reward Model. *NeurIPS*.
- Christiano, P., et al. (2017). Deep RL from human preferences. *NeurIPS*.
- Ouyang, L., et al. (2022). Training language models to follow instructions with human feedback. *NeurIPS*.
- Shao, Z., et al. (2024). DeepSeekMath (GRPO). arXiv:2402.03300.
- Bai, Y., et al. (2022). Constitutional AI: Harmlessness from AI Feedback. Anthropic.
