# Module 2b: Designing for Alignment (the Three Questions)

> **What you'll get out of this:** a design framework that forces you to answer
> three questions in order, and stops you from skipping the one everyone skips.

## The order matters

Here's how most teams build an "aligned" system: they pick an architecture, ship
it, then try to bolt values on with a system prompt. Backwards. By the time you're
arguing about the prompt, you've already baked in decisions that fight you.

The fix is to answer three questions, *in this order*:

1. **What do users actually value?** (conceptual)
2. **How do we measure those values?** (empirical)
3. **What architecture serves the values and the measurement?** (technical)

Skip question one and you optimize the wrong thing beautifully. Skip question two
and you can't tell if you succeeded. This lesson is those three questions, done
right.

## What you'll be able to do

- Pull a user's *real* values out from under their stated requests.
- Turn fuzzy values into metrics you can actually optimize.
- Pick an architecture because it serves alignment, not because it's elegant.

## Question 1: What do users actually value?

Not what they ask for. What they value. These are different, and the gap is where
systems die.

> **The classic miss:** a user asks for "faster responses." What they actually
> value is "timely, actionable insight that doesn't sacrifice accuracy." Build for
> the literal request and you'll happily make them something fast and wrong.

So go looking for the values underneath. Four kinds worth digging for:

- **Professional obligations:** what rules, regulations, or reputational lines can
  *never* be crossed?
- **Practical constraints:** time, budget, skills, what's actually feasible?
- **Quality standards:** how accurate is accurate enough, and when does that change?
- **Usability:** how much learning curve, integration friction, or workflow change
  will they tolerate?

Three questions reliably surface the real values:

- **What can never be violated, no matter the upside?** (the non-negotiables)
- **When time runs short, what gets sacrificed first?** (the trade-off ranking)
- **When does "more accurate" stop being worth more time?** (the good-enough line)

And don't just ask. Watch. Observe real workflows, study actual decision patterns,
push on the edge cases where values collide. People's stated priorities and
revealed priorities are rarely the same.

## Question 2: How do you measure it?

Now translate those values into numbers you can steer by. The trap here is
measuring the *system* when you meant to measure *user success*.

> **Alignment metrics vs performance metrics.** Accuracy, latency, throughput
> measure what the system *can do*. They don't tell you if its behavior matched
> what the user *needed*. You want the second kind.

Three families of alignment metric:

- **Trust and credibility:** Does user confidence rise over time? How often do they
  re-verify the AI's output? Do they accept or override its suggestions? Over the
  long run, do its recommendations track their actual success?
- **Contextual fit:** Is it relevant to *this* situation? Does it adapt when
  priorities shift? Does it read urgency and stakes correctly?
- **Trade-off quality:** Does it balance speed against accuracy the way this user
  would? Does it respect their quality bar and risk tolerance?

The hard part is the subjective stuff (trust, satisfaction, value). You measure it
with proxies: re-verification rate is a proxy for trust; override rate is a proxy
for confidence; long-term outcome correlation is the real prize. Define those
proxies *before* you build, or you'll rationalize whatever you ship.

## Question 3: What architecture serves all that?

Only now do you pick the technical approach, and you pick it to serve the answers
above, not because it's the latest thing.

The decisions that actually matter for alignment:

- **Agent-based vs rule-based.** Agents learn and handle novelty; rules give strict
  compliance and predictability. Most real systems are **hybrid**: rule-based
  guardrails around an agent-based optimizer.
- **Where control lives.** Prompt constraints are flexible but inconsistent;
  algorithmic controls are reliable but rigid. Layer them: hard rules for the
  things that must never break, flexible control for everything else.
- **Timing.** Real-time is responsive but shallow; batch is thorough but slow;
  adaptive timing matches effort to the stakes.
- **Centralized vs distributed.** Centralized gives consistency; distributed gives
  scale and local adaptation.

> **The selection rule:** choose the approach that best supports *aligned
> behavior*, and make interpretability a hard requirement for any high-stakes
> decision. Elegance is not a tiebreaker.

## The three answers have to talk to each other

The framework's power is in the loops between the questions, not the questions
alone:

- **Values shape which metrics matter**, and your metrics, once running, reveal
  where you misread the values. Refine both.
- **Metrics drive architecture choices**, and what's technically measurable feeds
  back into which metrics are even possible.
- **Values constrain the architecture** (some technical choices simply can't be
  made aligned), and technical limits sometimes force an honest renegotiation of a
  trade-off.

You'll see this concretely in this module's financial-agent example (2e and the
notebooks): the user's values become scoring weights, the weights become a
measurable selection rule, and the architecture is built to keep that rule
inspectable.

## The takeaways

- Answer three questions **in order**: values, then measurement, then architecture.
  The order is the whole point.
- **Real values hide under stated requests.** Dig for the non-negotiables, the
  trade-off ranking, and the good-enough line, and watch behavior, don't just ask.
- **Measure user success, not system performance.** Define your trust and trade-off
  proxies before you build.
- **Pick architecture to serve alignment**, with interpretability mandatory for
  high stakes.
- The three answers **inform each other**. Iterate the loop.

## Think about it

1. For a domain you know, what would users *say* they want versus what they
   actually value? Write both columns.
2. How would you measure whether a system genuinely respects those values, using
   only things you can observe?
3. Which architecture decision would matter most for keeping *your* use case
   aligned?

## Next

Module 2c flips from design to code: three perspectives (subject-matter expert,
mathematical modeler, implementer) that make sure the system you build actually
delivers the alignment you designed.
