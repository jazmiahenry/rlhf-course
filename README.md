# RL Alignment Course

A hands-on course on aligning AI agents with reinforcement learning: from
"why do deployed AI systems fail?" through the mathematics of value-aligned
decision-making, to runnable simulations, Q-learning curricula, and the
modern post-training landscape (RLHF → DPO → GRPO → RLVR).

## Why this course exists

The demand for these skills is compounding faster than any training pipeline
is producing them:

- **AI engineer is the fastest-growing job in the United States**
  ([LinkedIn Jobs on the Rise 2026](https://www.linkedin.com/pulse/linkedin-jobs-rise-2026-25-fastest-growing-roles-us-linkedin-news-dlb1c)),
  with 639,000 AI-related postings added in the U.S. between 2023 and 2025.
- **Job postings mentioning agentic AI grew ~1,000% from 2023 to 2024**, and
  46% of leaders cite skill gaps as a major barrier to AI adoption
  ([McKinsey, The State of AI 2025](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)).
- **2.5% of *all* U.S. job postings now request AI skills** — up 55% in one
  year ([Stanford AI Index 2026 / Lightcast](https://lightcast.io/resources/research/stanford-ai-index-2026)),
  which added *Agentic AI* as a tracked skill cluster for the first time.
- Meanwhile, **more than 80% of AI projects fail** — twice the rate of non-AI
  IT projects ([RAND, 2024](https://www.rand.org/pubs/research_reports/RRA2680-1.html)) —
  and **~95% of enterprise GenAI pilots deliver no measurable P&L impact**
  ([MIT, The GenAI Divide, 2025](https://www.legal.io/blog/5719519/MIT-Report-Finds-95-of-AI-Pilots-Fail-to-Deliver-ROI-Exposing-GenAI-Divide)).

Evaluation and reinforcement learning are the two disciplines that close
that gap, and neither has a standard curriculum. This course is built to be
one — every statistic above (and throughout the lessons) links to a primary
source you can verify, because *checking the numbers is the first
evaluation skill*.

## Course map

| Module | Content | Format |
|---|---|---|
| **1 — The Alignment Problem** | Why agents make arbitrary decisions; the documented costs of misalignment; the utility-maximization framework for tool selection | 4 lessons (1a–1d) |
| **2 — Designing Aligned Systems** | The tripartite design/code methods; values → metrics; a complete financial-analyst case study with a hand-built scoring function | 5 lessons (2a–2e) + 3 notebooks (Financial Agent Parts 1–3) |
| **3 — RL Foundations for Alignment** | State/action spaces, stochastic rewards and uncertainty (CVaR), trajectories, preference learning, curriculum design | 6 lessons (3a–3f) + 3 notebooks (RL Alignment Parts 1–3) |
| **4 — Algorithms** | Monte Carlo & Markov chains, DQN, PPO (with the clipped objective and its RLHF role), DPO and the post-DPO landscape (GRPO, RLVR), reflective agents | 6 lessons (4a–4f) |

### The notebooks

All six notebooks run locally with no API key required:

- **Financial Agent Part 1** — build a value-weighted document scoring
  framework from scratch (numpy/pandas only).
- **Financial Agent Part 2** — wire the scoring stage into an LLM analyst
  via the OpenAI Responses API; aligned vs. naive comparison. Runs in
  simulation mode without a key; set `OPENAI_API_KEY` (and optionally
  `OPENAI_MODEL`) for live calls.
- **Financial Agent Part 3** — multi-client experiment harness and
  evaluation dashboards.
- **RL Alignment Part 1** — implement the state/action/reward formalism of
  Module 3 as runnable code.
- **RL Alignment Part 2** — trajectories with alignment constraints, plus
  tabular Q-learning through a 4-stage curriculum.
- **RL Alignment Part 3** — an integrated research-agent simulation with
  guided exercises. Optional Claude API path: set `ANTHROPIC_API_KEY`.

```bash
pip install numpy pandas matplotlib seaborn jupyter
jupyter lab
```

## Sources and corrections

Lessons cite primary sources inline. If you find a claim that doesn't hold
up — a stale model ID, a number whose source has been revised, an equation
error — open an issue. The course practices what it teaches: claims are
falsifiable or they're not claims.

## License

See [LICENSE](LICENSE).
