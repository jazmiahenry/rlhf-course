# Module 2e: A Worked Example (Stock-Analysis Assistant)

> **What you'll get out of this:** everything from Module 2, design, code, and
> the scoring math, applied end to end to one real scenario, with numbers you can
> check and a prediction you should make before you scroll.

## The scenario

A financial analyst needs help right after a surprise earnings announcement from a
big tech company. The situation is messy in exactly the ways that break naive AI:

- data scattered across platforms,
- news arriving asynchronously,
- regulatory and market shocks moving prices unpredictably,
- different clients with different risk appetites,
- and a hard compliance rule: no unverified sources in client recommendations.

So the question isn't "can the AI summarize an article." It's "which information
should it even be looking at, given this analyst's values?" That's a scoring
problem. Let's build the score.

## What you'll be able to do

- Read a real multi-factor scoring function and know what each term does.
- Hand-compute document scores and predict the ranking.
- See why the "freshest" or "most exciting" source is usually the wrong pick.

## Step 1: the values become weights (the design)

From talking to the analyst (Hat 1 from 2c), four values dominate, and they map
straight to weights:

- **Credibility** is king during volatility. **β = 0.4**, the heaviest weight.
- **Relevance** matters but breadth is fine. **α = 0.3**.
- **Personalization** to the client's profile. **w = 0.2**.
- **Exploration** (novel takes) gets the least, since this is a "be careful" moment.
  **λ = 0.1**.

That weighting *is* the alignment. A day-trader client would get different numbers.
A compliance-first institution, different again.

## Step 2: the scoring function (the math)

Here's the function that turns those values into a single number per document:

$$V_d = \alpha R_d + \beta S_d + \mathbf{w}^\top X_d + \lambda \cdot E_d \cdot Q_d$$

Decoded:

- $R_d$ = relevance (semantic match to the query)
- $S_d$ = source credibility (reputation, verification)
- $X_d$ = personalization (fit to the client profile)
- $E_d$ = exploration bonus (how novel)
- $Q_d$ = quality (recency, completeness, format)

Notice the last term: exploration only counts *multiplied by quality*. A novel take
from a garbage source contributes almost nothing. That's a deliberate design choice
to stop the system chasing exciting-but-unreliable content. Small detail, big
effect.

## Step 3: three documents walk in

Right after the announcement, the agent has three candidates:

| Document | Relevance $R$ | Credibility $S$ | Personalization $X$ | Exploration $E$ | Quality $Q$ |
|---|---|---|---|---|---|
| **A: Reuters earnings report** | 0.9 | 0.95 | 0.8 | 0.1 | 0.9 |
| **B: anonymous social post** | 0.8 | 0.2 | 0.6 | 0.7 | 0.3 |
| **C: competitor-analysis report** | 0.7 | 0.85 | 0.9 | 0.6 | 0.8 |

**Pause and predict.** Document B is the freshest and most novel (posted minutes
after the announcement, a take nobody else has). Document A is the establishment
wire report. Document C is a careful third-party analysis. Which ranks *first*, and
which ranks *last*? Commit to an answer before you compute it.

## Step 4: do the arithmetic

With $\alpha=0.3,\ \beta=0.4,\ \mathbf{w}=0.2,\ \lambda=0.1$:

**A (Reuters):**
$$0.3(0.9) + 0.4(0.95) + 0.2(0.8) + 0.1(0.1)(0.9) = 0.27 + 0.38 + 0.16 + 0.009 = \mathbf{0.819}$$

**B (social post):**
$$0.3(0.8) + 0.4(0.2) + 0.2(0.6) + 0.1(0.7)(0.3) = 0.24 + 0.08 + 0.12 + 0.021 = \mathbf{0.461}$$

**C (competitor analysis):**
$$0.3(0.7) + 0.4(0.85) + 0.2(0.9) + 0.1(0.6)(0.8) = 0.21 + 0.34 + 0.18 + 0.048 = \mathbf{0.778}$$

**Ranking: A (0.819) > C (0.778) > B (0.461).**

The freshest, most novel source finished *last*, by a mile. Its low credibility
(0.2) and quality (0.3) tanked it, and the exploration bonus couldn't rescue it
because it's gated on quality. A naive "use the latest, most interesting take"
system would have led with exactly the wrong document. The scoring function, built
from the analyst's values, refuses to.

That's the entire point of this module in three numbers.

## Step 5: the architecture serves the math (the code)

This score doesn't live in a prompt. It lives in the scoring pipeline from Module
2d: preprocess the documents, compute the five components, integrate with the
weights, rank, then hand the *top-ranked, pre-aligned* document to the language
model for analysis. The model never sees the social post as a co-equal source. It
reasons over what the values already approved.

> **The two-stage spine:** score and select with explicit, inspectable math, then
> generate. You build exactly this in the Financial Agent notebooks (Parts 1 to 3),
> including the aligned-versus-naive comparison and, in Part 3, the statistical test
> of whether the difference is real.

## The takeaways

- Values become **weights** ($\alpha,\beta,\mathbf{w},\lambda$), and the weights are
  the alignment. Different client, different numbers.
- The scoring function makes trade-offs **explicit and checkable**. You computed the
  ranking by hand.
- The "freshest / most novel" source **lost**, because credibility and quality
  dominate during volatility, and exploration is gated on quality.
- The architecture runs **score-then-generate**: the model reasons over pre-aligned
  inputs, never the raw firehose.

## Think about it

1. Re-rank the three documents for a **day-trader** client who weights speed and
   novelty (try $\alpha=0.2,\ \beta=0.2,\ \mathbf{w}=0.2,\ \lambda=0.4$). Does B ever
   win? Should it?
2. What other component would your domain need in the scoring function?
3. When two values genuinely conflict (speed vs accuracy in a crisis), how should
   the function resolve it, and who decides?

## Next

Module 3 goes under the hood. Where Module 2 hand-designed the scoring, Module 3
brings in reinforcement learning: how an agent can *learn* aligned decisions, from
state and reward design through preference learning and curricula.
