# Module 2c: From Design to Code (Three Hats)

> **What you'll get out of this:** the three perspectives you switch between when
> you actually build an aligned system, and the pitfalls that come with each.

## The gap between a good design and a working system

You can nail the design from 2b and still ship garbage. The translation from
"here's what users value" to "here's running code that respects it" is where most
alignment quietly dies.

The trick is to wear three hats, one at a time, and keep switching:

1. **The subject-matter expert**, who understands the human context.
2. **The mathematical modeler**, who turns that context into decisions a computer
   can make.
3. **The implementer**, who builds it so alignment survives scale.

Wear only one and you get a predictable failure. Let's take them in order.

## What you'll be able to do

- Develop real empathy for a user's workflow, not just a requirements list.
- Turn user trade-offs into a math model that's tunable and explainable.
- Build systems where the alignment-critical parts can be tested and watched.

## Hat 1: Be the subject-matter expert

This is not requirements-gathering. It's stepping into the user's world until you
understand how they actually work and what "success" means to them.

Go after three things:

- **Their workflow.** When do which decisions get made? What sources do they
  already trust, in what order? How do they handle missing or conflicting info
  *today*?
- **The real stakes.** What's the downstream effect of a recommendation? When is a
  mistake a minor annoyance versus a career problem?
- **The hidden constraints.** Formal rules, sure, but also the informal ones:
  organizational culture, client expectations, professional norms.

> **The cry-wolf insight.** Getting urgency *wrong* is as dangerous as being slow.
> A system that screams "URGENT" at routine things gets ignored, and then it's
> silent for the genuinely on-fire requests. Read timing from the workflow, not
> from the user's adjectives.

## Hat 2: Be the mathematical modeler

Now turn all that human texture into something a machine can optimize. This is the
bridge from values to computation.

Three moves:

- **Make the trade-offs explicit.** Real decisions balance competing goals. A model
  forces those trade-offs into the open where you can tune them. (This is the
  weighted scoring from Module 1C, applied for real.)
- **Model the uncertainty.** Users act under incomplete information. Your model
  should represent risk the way *they* think about it, not pretend the world is
  certain.
- **Account for time.** Information decays, urgency spikes. Bake temporal effects
  into the value function.

But keep it **interpretable**. A model nobody can explain is a model nobody will
trust or tune.

> **The discipline:** every model should explain not just *what* it recommends but
> *why it made that trade-off*. If you can't write the one-sentence reason, the
> model is too complex.

## Hat 3: Be the implementer

A perfect model still fails if the implementation leaks alignment. So build for it:

- **Isolate the alignment-critical functions** so you can monitor, test, and improve
  them on their own.
- **Separate performance optimization from alignment optimization.** Tune each
  without breaking the other.
- **Feed the model an aligned diet.** Don't hand a language model raw, unfiltered
  data. Filter, rank, and contextualize it *first*, so the model reasons over
  pre-aligned inputs. (This two-stage idea, score then generate, is exactly what
  the financial-agent notebooks build.)
- **Plan for failure.** Graceful degradation for edge cases and for moments when two
  alignment principles collide.

And prove it works: compare aligned versus naive approaches head to head, test the
edge cases on purpose, and watch the alignment metrics over time so learning
doesn't drift away from the user's values.

## The pitfalls, one per hat

- **SME pitfall:** assuming you understand users without watching them; trusting
  stated requirements over observed behavior.
- **Modeler pitfall:** optimizing for mathematical elegance over user
  understanding; building models too complex to interpret or tune.
- **Implementer pitfall:** prioritizing raw performance over alignment in the
  architecture; building a monolith you can't inspect or adjust.

Notice they're all the same mistake in different clothes: losing sight of the user
while chasing the thing your hat cares about. Switching hats on purpose is the
cure.

## The takeaways

- Building aligned systems means switching between **three perspectives**: the SME
  (human context), the modeler (math), and the implementer (scale).
- **Read urgency from the workflow, not the adjectives.** False urgency is as
  dangerous as being slow.
- **Every model needs a one-sentence reason.** If it doesn't have one, it's too
  complex to trust.
- **Feed the model pre-aligned inputs.** Score and select *before* you generate.
- The pitfalls all reduce to **losing the user** while chasing your hat's priority.

## Think about it

1. For a system you know, what would deep SME observation reveal that a
   requirements doc would miss?
2. Take one real user trade-off and sketch the math model that would capture it.
   What's the one-sentence reason it would output?
3. Where have you seen performance prioritized over alignment in a real system?
   What did it cost?

## Next

Module 2d gets specific about implementation: the architecture, the scoring
pipeline, and the feedback loops that keep an aligned system aligned as it scales.
