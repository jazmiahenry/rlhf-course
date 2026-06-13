# Module 1B: The Consequences and Costs of Misalignment

## Introduction: When Good AI Goes Wrong

In Module 1A, we identified the core problem: current AI systems understand
what users want (good state space) but lack systematic frameworks for choosing
what to do (poor action space). Now we need to understand why this matters in
practice.

**The question**: What are the real-world consequences when AI systems make
arbitrary or inconsistent decisions, and what does the evidence actually say?

A note on method before we start: this lesson cites only documented,
verifiable sources. Claims about AI failure costs are frequently inflated or
invented in vendor marketing, and learning to ask "says who, measured how?"
is itself an evaluation skill, arguably the first one this course teaches.
Every figure below links to a primary source you can check.

## The Evidence: AI Failure Is the Norm, Not the Exception

Three independent research efforts, using three different methodologies,
converge on the same picture:

**Most AI projects fail.** RAND Corporation's 2024 study
[*The Root Causes of Failure for Artificial Intelligence Projects*](https://www.rand.org/pubs/research_reports/RRA2680-1.html)
(structured interviews with 65 experienced data scientists and engineers)
found that **more than 80% of AI projects fail, roughly twice the failure
rate of IT projects that don't involve AI**. The leading root causes were not
algorithmic: miscommunicated objectives, inadequate data, missing
infrastructure, and leadership switching priorities before deployment.

**Generative AI pilots fail at an even higher rate.** MIT's 2025 report
[*The GenAI Divide: State of AI in Business*](https://www.legal.io/blog/5719519/MIT-Report-Finds-95-of-AI-Pilots-Fail-to-Deliver-ROI-Exposing-GenAI-Divide)
(52 executive interviews, 153 leader surveys, 300 public deployments analyzed)
found that **about 95% of enterprise GenAI pilots delivered no measurable
P&L impact**. The 5% that succeeded shared a trait directly relevant to this
course: they treated deployment as a measurement problem, not a demo problem.

**The agentic wave is heading for a correction.** Gartner
[predicted in June 2025](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027)
that **over 40% of agentic AI projects will be canceled by the end of 2027**,
citing escalating costs, unclear business value, and inadequate risk
controls, and estimated that of the thousands of vendors claiming "agentic
AI," **only about 130 are real** (the rest engage in what Gartner calls
"agent washing").

And the failures that reach the public are growing: the
[AI Incident Database](https://hai.stanford.edu/ai-index/2025-ai-index-report/responsible-ai),
as reported in Stanford's AI Index, recorded **233 AI incidents in 2024, a
record high and a 56.4% increase over 2023, and 362 in 2025**. These are
realized harms or near-harms from deployed systems, not hypotheticals.

## The Five Categories of Misalignment Consequences

The taxonomy below organizes *how* these failures show up. For each category,
we look at documented evidence rather than constructed scenarios.

### 1. Inconsistent Behavior: The Trust Erosion Problem

**The Pattern**: Same inputs producing materially different outputs across
sessions, so users can never calibrate when to rely on the system.

The cleanest documented example is AI in legal research. Since *Mata v.
Avianca* (S.D.N.Y. 2023), where two lawyers were sanctioned $5,000 for a
brief citing six nonexistent ChatGPT-invented cases, [Damien Charlotin's AI Hallucination Cases database](https://www.scientificamerican.com/article/why-lawyers-keep-citing-fake-cases-invented-by-ai/)
has tracked **more than 1,300 cases worldwide in which a court or tribunal
addressed AI-generated hallucinations in filings**. Sanctions have escalated
from thousands of dollars to a record **$110,000 penalty against two Oregon
lawyers** who submitted 23 fabricated citations. The pattern persists three
years after the first sanction made global news, which tells you something
important: *warnings don't fix reliability problems; measurement and process
do*. Professionals keep getting burned because the system is right often
enough to invite trust and wrong often enough to destroy it.

**Why this matters for alignment**: a system that is excellent 80% of the
time and arbitrary 20% of the time is, for professional use, worse than a
consistently mediocre one, because the user cannot predict which mode
they're in. Consistency is a property you must *measure across sessions*,
which is why evaluation (the other half of this course's curriculum) is
inseparable from alignment.

### 2. Poor Value Trade-offs: The Optimization Failure Problem

**The Pattern**: AI systems failing to balance competing user values, speed vs. accuracy, cost vs. coverage, helpfulness vs. safety, in a way
that matches what the user actually needed.

RAND's interviewees described this as the dominant *root cause* of project
failure: stakeholders misunderstand or miscommunicate **which problem needs
solving and which trade-offs are acceptable**, so the system optimizes
something other than what the organization values. MIT's GenAI Divide data
shows the same mechanism at the portfolio level: enterprise AI budgets
flowed disproportionately to sales and marketing use cases while the
measurable ROI was concentrated in back-office operations, a value
misallocation, not a capability gap.

This is exactly the gap Module 1C's value-weight framework addresses: if
trade-offs aren't represented explicitly, they get resolved implicitly and
arbitrarily.

### 3. Context Insensitivity: The One-Size-Fits-None Problem

**The Pattern**: The same behavior applied regardless of stakes, urgency, or
user expertise.

The benchmark literature quantifies how far current agents are from
context-appropriate competence on real tasks. When
[WebArena](https://webarena.dev/) (Zhou et al., 2023) introduced realistic
web tasks, the best GPT-4-based agent achieved an **end-to-end success rate
of 14.4%, versus 78% for humans**. [OSWorld](https://os-world.github.io/)
(Xie et al., 2024) found agents completed about **12% of real computer
tasks, versus 72% for humans**, with failures concentrated precisely in
grounding and operational knowledge, i.e., understanding the context of the
environment they're acting in. Agents have improved substantially since
these papers, but the gap they document is the reason context-sensitivity
must be *evaluated*, not assumed. (Both benchmarks, and their successors,
are runnable against current models, later modules use exactly this kind
of harness.)

### 4. No Learning From Feedback: The Groundhog Day Problem

**The Pattern**: Systems that make the same mistake repeatedly because
nothing connects outcomes back to behavior.

McKinsey's [State of AI 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)
survey notes that most organizations deploying AI still lack the basic
feedback machinery, monitoring, evaluation pipelines, incident review, to
detect drift or regression. The AI Incident Database's 56% year-over-year
growth in recorded incidents is partly a reporting effect, but it is also
what you'd expect from a deployment ecosystem that ships faster than it
instruments. The fix is the subject of Modules 2D and 2E: feedback loops
are an architectural decision, made up front, not a patch.

### 5. Manipulation Vulnerability: The Adversarial Problem

**The Pattern**: Systems whose decision process can be steered by inputs
crafted to exploit it.

Prompt injection has topped the
[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
(LLM01) since the list's first release in 2023 through its current revision, the most systematic ranking of deployed-LLM security risks available. The
agentic shift raises the stakes: an agent that takes actions (sends emails,
executes code, moves money) turns a manipulated *answer* into a manipulated
*action*. Gartner's risk-control concerns in the 40%-cancellation prediction
are largely about exactly this. Module 3F's adversarial robustness stage and
the safety evaluations covered in the evals track exist because this
vulnerability class is measurable, and unmeasured systems should be assumed
vulnerable.

## The Other Side of the Ledger: A Historic Talent Gap

The failure evidence above has a flip side: the people who can *prevent*
these failures, who can design evaluations, build feedback loops, and apply
RL methods correctly, are among the scarcest and best-paid professionals in
the industry. The demand-vs-supply picture, from primary labor-market data:

**Demand is compounding.**

- [LinkedIn's Jobs on the Rise 2026](https://www.linkedin.com/pulse/linkedin-jobs-rise-2026-25-fastest-growing-roles-us-linkedin-news-dlb1c)
  ranks **AI engineer as the fastest-growing role in the United States**;
  LinkedIn counted **639,000 AI-related job postings added between 2023 and
  2025**, and [estimates AI has already added ~1.3 million new roles](https://www.weforum.org/stories/2026/01/ai-has-already-added-1-3-million-new-jobs-according-to-linkedin-data/).
- The [Stanford AI Index (Lightcast data)](https://lightcast.io/resources/research/stanford-ai-index-2026)
  reports that **2.5% of *all* U.S. job postings now request AI skills, up
  55% in a single year and nearly 300% over the decade**, and that Lightcast
  added **"Agentic AI" as a tracked skill cluster for the first time in
  2026**, with demand shifting from research skills toward deployment,
  scaling, and operations: exactly the territory of evaluation.
- McKinsey's [State of AI 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)
  found **job postings mentioning agentic AI grew nearly 1,000% from 2023 to
  2024**, that **46% of leaders cite skill gaps as a major barrier to AI
  adoption**, and that in financial services and healthcare **filling an AI
  position now takes 6-7 months on average**.
- The [WEF Future of Jobs Report 2025](https://www.weforum.org/publications/the-future-of-jobs-report-2025/)
  (1,000+ employers, 14 million workers represented) ranks **AI and big data
  as the single fastest-growing skill category to 2030**, with AI and data
  processing expected to create ~11 million roles while displacing ~9 million.

**The price signal confirms the shortage.** Self-reported compensation data
([Levels.fyi aggregations](https://ctaio.dev/en/salary/anthropic-salary/))
puts median total compensation for engineers at the frontier labs around
**$555K (OpenAI) and $600K (Anthropic)**, with base salaries of $250-400K, multiples of the national software median. Post-training (RLHF/RL) and
evaluation expertise are precisely the specializations these labs hire for.

**Supply has not caught up, especially for evals and RL.** Reinforcement
learning is taught at research depth in a few hundred graduate programs;
AI *evaluation* as an engineering discipline is so new that **no standard
curriculum exists at all**, practitioners assemble it from papers, blog
posts, and folklore. Industry hiring guides now list eval design as the
single strongest signal of real LLM experience in interview loops, yet there
is no established pathway to learn it. That asymmetry, compounding demand
documented by LinkedIn, Lightcast, McKinsey, and the WEF, against a supply
pipeline that barely exists, is the gap this course exists to close. The
80% project-failure rate and the 6-month hiring cycles are the same fact
viewed from two sides: organizations cannot find people who know how to make
AI systems measurably work.

## Key Takeaways

1. **Failure is the documented norm**: >80% of AI projects (RAND), ~95% of
   GenAI pilots (MIT), >40% of agentic projects projected canceled (Gartner).
   The causes are systemic, objectives, data, measurement, not exotic.
2. **The five consequence categories**, trust erosion, bad trade-offs,
   context insensitivity, no feedback learning, manipulation vulnerability, are all *measurable*, which means they are all addressable.
3. **Inconsistency is deadlier than mediocrity** for professional adoption;
   1,300+ court cases of AI-hallucinated citations show that warnings alone
   change nothing.
4. **The talent gap is the inverse of the failure rate**: demand for
   evaluation and RL skills is compounding (fastest-growing role, +1,000%
   agentic postings, 6-7 month hiring cycles) while structured training
   barely exists.
5. **Check every number**, including ours. Each claim above links to its
   source. That habit is the foundation of evaluation work.

## Looking Ahead

Module 1C introduces the mathematical framework, explicit tool
characterization, value weights, and utility maximization, that turns the
implicit, arbitrary decisions documented here into explicit, measurable,
improvable ones.
