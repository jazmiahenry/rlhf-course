# Module 1B: What Misalignment Actually Costs

> **What you'll get out of this:** the real, documented price of the gap from 1A,
> with every number linked to a primary source, plus the flip side: why the
> people who can fix this are some of the scarcest and best-paid in tech.

## First, a house rule

Before a single statistic: we cite only documented, checkable sources here.
Claims about "AI failure costs" are inflated or invented constantly in vendor
decks, and learning to ask *"says who, measured how?"* is itself an evaluation
skill. Arguably the first one this course teaches. Every figure below links to
something you can open and verify. That includes our numbers. Especially our
numbers.

So let's answer the real question: when AI systems make arbitrary, inconsistent
decisions, what does it actually cost? Not in theory. In the field.

## Failure is the norm, not the exception

Three independent research efforts, three different methods, same picture.

**Most AI projects fail.** RAND's 2024 study
[*The Root Causes of Failure for Artificial Intelligence Projects*](https://www.rand.org/pubs/research_reports/RRA2680-1.html)
(structured interviews with 65 working data scientists and engineers) found
**more than 80% of AI projects fail, roughly twice the rate of IT projects that
don't involve AI.** And the top causes weren't algorithmic. They were
miscommunicated objectives, bad data, missing infrastructure, and leadership
yanking priorities before deployment.

**Generative AI pilots fail even harder.** MIT's 2025
[*GenAI Divide: State of AI in Business*](https://www.legal.io/blog/5719519/MIT-Report-Finds-95-of-AI-Pilots-Fail-to-Deliver-ROI-Exposing-GenAI-Divide)
(52 executive interviews, 153 leader surveys, 300 deployments analyzed) found
**about 95% of enterprise GenAI pilots delivered no measurable P&L impact.** The
5% that worked shared a trait that should sound familiar by the end of this
course: they treated deployment as a measurement problem, not a demo.

**The agentic wave is heading for a correction.**
[Gartner predicted, in June 2025](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027),
that **over 40% of agentic AI projects will be canceled by the end of 2027**,
citing runaway costs, fuzzy value, and weak risk controls. They also estimate
that of the thousands of vendors claiming "agentic AI," **about 130 are real**
(the rest do what Gartner calls "agent washing").

And the failures that go public keep climbing. The
[AI Incident Database](https://hai.stanford.edu/ai-index/2025-ai-index-report/responsible-ai),
via Stanford's AI Index, logged **233 AI incidents in 2024 (a record, up 56% over
2023) and 362 in 2025.** Those are realized harms from deployed systems, not
hypotheticals.

## The five ways it shows up

Those numbers are the *what*. Here's the *how*: the five shapes misalignment
takes. For each, real evidence, not a made-up scenario.

### 1. Inconsistent behavior (the trust killer)

Same input, materially different output across sessions, so you can never
calibrate when to rely on the thing.

The cleanest documented case is legal research. Since *Mata v. Avianca* (2023),
where two lawyers were sanctioned $5,000 for a brief citing six ChatGPT-invented
cases,
[Damien Charlotin's AI Hallucination Cases database](https://www.scientificamerican.com/article/why-lawyers-keep-citing-fake-cases-invented-by-ai/)
has tracked **more than 1,300 cases worldwide where a court addressed
AI-generated hallucinations in filings.** Sanctions have climbed from thousands
of dollars to a record **$110,000 against two Oregon lawyers** for 23 fabricated
citations. The pattern persists *three years* after the first sanction made
global news, and that's the lesson: warnings don't fix reliability. Measurement
and process do. Professionals keep getting burned because the system is right
often enough to earn trust and wrong often enough to wreck it.

> **Why this matters for you as a builder:** a system that's excellent 80% of the
> time and arbitrary 20% of the time is, for professional use, *worse* than one
> that's consistently mediocre. You can't predict which mode you're in.
> Consistency is a property you measure *across sessions*, which is why
> evaluation and alignment are the same project.

### 2. Poor value trade-offs

The system balances speed vs accuracy, cost vs coverage, helpfulness vs safety,
in a way that doesn't match what the user needed.

RAND's interviewees called this the dominant *root cause* of project failure:
stakeholders miscommunicate which problem to solve and which trade-offs are okay,
so the system optimizes the wrong thing. MIT's data shows the same at the
portfolio level: budgets poured into sales and marketing use cases while the
measurable ROI sat in back-office operations. A value misallocation, not a
capability gap. This is exactly what Module 1C's value-weight framework is built
to fix.

### 3. Context insensitivity

The same behavior regardless of stakes, urgency, or who's asking.

The benchmark literature quantifies how far agents are from context-appropriate
competence on real tasks. When [WebArena](https://webarena.dev/) (Zhou et al.,
2023) introduced realistic web tasks, the best GPT-4 agent hit a **14.4%
end-to-end success rate, against 78% for humans.**
[OSWorld](https://os-world.github.io/) (Xie et al., 2024) found agents finished
about **12% of real computer tasks, against 72% for humans**, with failures
concentrated in grounding and operational know-how, i.e. understanding the
context they're acting in. Agents have improved a lot since these papers, but the
gap is *why* context sensitivity has to be measured, not assumed. (Both
benchmarks run against current models, and later modules use exactly this kind of
harness.)

### 4. No learning from feedback

The same mistake, over and over, because nothing connects outcomes back to
behavior.

McKinsey's [State of AI 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)
notes that most organizations deploying AI still lack the basic machinery,
monitoring, eval pipelines, incident review, to catch drift or regression. The
56% year-over-year jump in recorded incidents is partly better reporting, but
it's also what you'd expect from an ecosystem that ships faster than it
instruments. The fix is the subject of Modules 2D and 2E: feedback loops are an
architecture decision you make up front, not a patch.

### 5. Manipulation vulnerability

A decision process you can steer with crafted inputs.

Prompt injection has topped the
[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
(LLM01) since the list launched in 2023, the most systematic ranking of deployed
LLM risk we have. The agentic shift raises the stakes: an agent that *takes
actions* (sends email, runs code, moves money) turns a manipulated answer into a
manipulated action. Gartner's risk-control worry in that 40%-cancellation
forecast is largely this. Module 3F's adversarial-robustness stage and the safety
evals later in the course exist because this is measurable, and unmeasured
systems should be assumed vulnerable.

## The other side of the ledger: a historic talent gap

Here's the flip side of all that failure. The people who can *prevent* it, who can
design evaluations, build feedback loops, and apply RL correctly, are among the
scarcest and best-paid professionals in the industry. The demand-versus-supply
picture, from primary labor data:

**Demand is compounding.**

- [LinkedIn's Jobs on the Rise 2026](https://www.linkedin.com/pulse/linkedin-jobs-rise-2026-25-fastest-growing-roles-us-linkedin-news-dlb1c)
  ranks **AI engineer the fastest-growing role in the United States**; LinkedIn
  counted **639,000 AI-related postings added in the US between 2023 and 2025**,
  and [estimates AI has already added ~1.3 million new roles](https://www.weforum.org/stories/2026/01/ai-has-already-added-1-3-million-new-jobs-according-to-linkedin-data/).
- The [Stanford AI Index, on Lightcast data](https://lightcast.io/resources/research/stanford-ai-index-2026),
  reports **2.5% of *all* US job postings now ask for AI skills, up 55% in a
  single year**, and added **"Agentic AI" as a tracked skill cluster for the
  first time in 2026**, with demand shifting from research skills toward
  deployment, scaling, and operations. Exactly the territory of evaluation.
- McKinsey's [State of AI 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)
  found **postings mentioning agentic AI grew nearly 1,000% from 2023 to 2024**,
  that **46% of leaders cite skill gaps as a major barrier to adoption**, and that
  in financial services and healthcare, **filling an AI role now takes 6 to 7
  months.**
- The [WEF Future of Jobs Report 2025](https://www.weforum.org/publications/the-future-of-jobs-report-2025/)
  ranks **AI and big data the single fastest-growing skill category to 2030.**

**The price tag confirms the shortage.** Self-reported comp data
([Levels.fyi aggregations](https://ctaio.dev/en/salary/anthropic-salary/)) puts
median total compensation for engineers at the frontier labs around **$555K
(OpenAI) and $600K (Anthropic)**, base salaries $250K to $400K, multiples of the
national software median. Post-training (RLHF/RL) and evaluation expertise are
precisely what these labs hire for.

**Supply hasn't caught up, especially for evals and RL.** Reinforcement learning
is taught at research depth in a few hundred grad programs. AI *evaluation* as an
engineering discipline is so new that **no standard curriculum exists at all**.
Practitioners stitch it together from papers, blog posts, and folklore. Hiring
guides now call eval design the single strongest signal of real LLM experience in
interview loops, yet there's no established way to learn it. That asymmetry,
compounding demand against a supply pipeline that barely exists, is the gap this
course is built to close. The 80% failure rate and the 6-month hiring cycles are
the same fact from two sides: organizations can't find people who know how to make
AI systems measurably work.

## The takeaways

- **Failure is the documented norm:** >80% of AI projects (RAND), ~95% of GenAI
  pilots (MIT), >40% of agentic projects projected canceled (Gartner). The causes
  are systemic (objectives, data, measurement), not exotic.
- **The five failure shapes**, trust erosion, bad trade-offs, context
  insensitivity, no feedback learning, manipulation, are all *measurable*. Which
  means they're all addressable.
- **Inconsistency is deadlier than mediocrity** for professional use. 1,300+ court
  cases of AI-hallucinated citations prove warnings alone change nothing.
- **The talent gap is the failure rate inverted:** demand for evals and RL is
  compounding while structured training barely exists.
- **Check every number, including ours.** Every claim above links to its source.
  That habit is the foundation of evaluation work.

## Next

Module 1C gives you the mathematical framework, explicit tool characterization,
value weights, utility maximization, that turns the implicit, arbitrary decisions
behind these costs into decisions you can write down, justify, and measure.
