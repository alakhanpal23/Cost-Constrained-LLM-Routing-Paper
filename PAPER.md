# Cost-Constrained LLM Routing with a Bounded Model Pool

**Technical summary of the [IND ENG 164 course paper](paper.pdf)**

Arjun Lakhanpal

## Abstract

Using a large menu of language models for every request can complicate deployment and make cost difficult to control. This project studies an offline planning problem: select at most *K* models and assign each observed prompt to one selected model while meeting an average quality target. A mixed-integer program minimizes measured routing cost, with optional quality floors for each benchmark group. In a saved run covering 240 prompts and 33 models, the five-model solution met an average-score floor of 0.80 and achieved a score of 0.8125 at zero *recorded* cost. The result illustrates the formulation and the effect of the supplied cost table. It does not establish a deployable router: assignments use observed prompt–model outcomes, and the source CSV is not present in this repository.

## Question and approach

The planning question is whether a limited model pool can satisfy a target quality level while minimizing cost. Let *P* be the prompts and *M* the candidate models. For each available prompt–model pair, the notebook takes an observed score *s(p,m)* and cost *c(p,m)* from a CSV. It uses binary *z(m)* to indicate whether a model enters the pool and binary *x(p,m)* to assign a prompt to a model.

The program minimizes the weighted sum of assignment costs, with a small score tie-breaker among near-equal-cost solutions. It enforces one assignment per prompt, assignment only to a selected model, at most *K* selected models, and a weighted mean score of at least *Q*. Optional constraints place a separate minimum on each benchmark group. A sensitivity parameter subtracts a fixed margin from observed scores (down to zero) before applying the quality constraint. That margin is a scenario assumption, not a statistically estimated uncertainty interval.

This is an **offline assignment model**: it chooses using the observed outcomes for the same prompts on which it reports quality. A practical router would need features available *before* selecting a model and evaluation on held-out prompts.

## Data and experiment

The saved notebook output reports 7,860 prompt–model rows, 240 distinct prompts, and 33 models. AIME, GPQA, LCB, and MMLU-Pro each contribute 60 prompts. The input is a locally supplied `routerbench.csv` with `dataset`, `prompt_id`, `model`, `score`, and `cost` fields. The CSV is not committed, so its provenance, score construction, cost units, and individual rows cannot be checked here. “RouterBench-style” describes the table shape; this repository does not establish that it uses the official [RouterBench dataset](https://arxiv.org/abs/2403.12031).

The notebook solves a baseline case with *K* = 5, mean-score floor *Q* = 0.80, and a 0.70 score floor for each benchmark group. It also varies *K*, *Q*, the fixed score margin, and the benchmark mix.

## Saved results

| Case | Mean score | Mean recorded cost | Reading |
| --- | ---: | ---: | --- |
| Five-model constrained assignment | 0.8125 | 0 | Meets the specified floors on the observed prompts |
| Single best global model baseline | 0.8875 | 0.042548 | One-model comparison from the same data |
| Cheapest per prompt baseline | 0.5667 | 0 | Does not meet the 0.80 mean-score floor |
| Best score per prompt baseline | 0.9833 | 0.001681 | Uses 24 models and the same observed outcomes |

The five-model solution selects GLM-Z1-9B-0414, MiniCPM4.1-8B, OpenThinker3-7B, Qwen2.5-Coder-7B-Instruct, and internlm3-8b-instruct. Its reported group scores are 0.7333 for AIME, 0.8833 for GPQA, 0.8833 for LCB, and 0.7500 for MMLU-Pro. The solver reports an optimal solution for the supplied formulation and data; this is a mathematical solver status, not proof of out-of-sample performance.

At *K* = 5, the saved quality-frontier run records zero cost through a 0.80 score floor, about 0.00000069 at 0.85, and about 0.00009998 at 0.90. Under a fixed 0.10 score reduction, a separate sensitivity run reports mean cost about 0.00007510 while maintaining a margin-adjusted score above 0.80. These quantities use the CSV's cost column and should not be read as current API or infrastructure prices.

## Limitations and next steps

1. **No independent reproduction of the figures.** The source CSV is absent, although the notebook and its saved outputs are available. Releasing a shareable data extract, generation script, or precise dataset reference would make the reported results auditable.
2. **In-sample oracle assignment.** The program knows prompt–model outcomes when choosing assignments. A useful deployment study would train a router on one split and evaluate routing decisions and cost on unseen prompts.
3. **Costs need validation.** A zero CSV cost may exclude GPU hosting, latency, provider fees, or other operational expenses. Units and collection dates should be documented before comparing configurations.
4. **Input handling can distort results.** The notebook replaces missing or nonnumeric score and cost values with zero and deduplicates by prompt ID and model. Missing costs can make routes appear free; a stronger version should reject invalid costs and verify prompt IDs across datasets.
5. **Sensitivity is illustrative.** A fixed score reduction and reweighted benchmark mixes show how the optimization responds to assumptions. They do not quantify statistical confidence or real-world distribution shift.

The most useful next experiment is a held-out routing evaluation with documented cost units, latency, and a baseline that makes decisions using only prompt-time information.

## Reproduction

Open [the notebook](IND_ENG_Final_Project.ipynb) in Jupyter or Colab. Provide a local `routerbench.csv` containing `dataset`, `prompt_id`, `model`, `score`, and `cost`. Install pandas, NumPy, Matplotlib, Pyomo, and `highspy`; the notebook can use GLPK if available. The saved figures and tables are preserved in the notebook, but the exact run cannot be repeated without its CSV.

## Background

[RouterBench: A Benchmark for Multi-LLM Routing System](https://arxiv.org/abs/2403.12031) provides context on evaluating multi-model routing. This paper's formulation and saved numbers should be assessed on their own terms; no direct comparability with RouterBench is claimed.
