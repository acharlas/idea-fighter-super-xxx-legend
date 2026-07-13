# Idea Evaluator — Recap

Building a reliable way to score SaaS ideas before committing time to build them.

## What We Tried

### Gathering Scores

1. **LLM per dimension per idea** — spawned 120 agents (30 SaaS × 4 dimensions), each scoring one dimension from its training knowledge. Unreliable — hallucinated data, inconsistent scale, no evidence trail.
2. **Deterministic data gathering** — used `pytrends` for search interest, scraped G2/Crunchbase/Reddit for competitor and revenue signals, stored results in a static data file. Reliable scores but rigid — data goes stale, scraping breaks, manual upkeep.

### Scoring Engine

3. **Python script** — consumed the static data file, computed weighted scores. Workable but boring.
4. **LLM subagent** — replaced the script with a single prompt. Same rubric, same math, but LLM interprets research data instead of hardcoded lookups. Strict no-fabrication rules, temperature 0.1. The winner.

## Final Design

A dedicated OpenCode subagent (`saas-evaluator`) using `deepseek-v4-flash` at temperature 0.1.

### Scoring Dimensions

| Dimension | Weight | What it measures |
|-----------|--------|------------------|
| Market Demand | 30% | Interest, search volume, discussion activity |
| Competition Landscape | 25% | Incumbent strength, gaps, blue-ocean potential |
| Revenue & Unit Economics | 25% | Pricing precedent, TAM, ARPU viability |
| Growth Trajectory | 20% | YoY trends, funding, category momentum |

Each dimension scored 1–10 with anchors (e.g. RSS reader=2, AI code gen=9).

### Key Rules

- **No fabrication** — insufficient data → mark it, don't guess
- **Fatal flaw** — any dimension ≤2 = automatic Red (non-negotiable)
- **Confidence** — 1–5 per dimension, honesty over bravado
- **Evidence** — every score cites specific data

### Verdict

| Weighted Score | Verdict |
|----------------|---------|
| ≥ 7.0 | Green — worth building |
| 4.0–6.9 | Yellow — needs deeper research |
| ≤ 3.9 | Red — move on |
