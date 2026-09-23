# Next evaluations: from an offline optimum to a usable router

The notebook's current assignment is an **in-sample oracle**: it sees the measured score for each prompt–model pair before choosing that prompt's model. These experiments separate that upper bound from policies that can make decisions on new prompts. They are proposed evaluations, **not completed results**.

## 1. Held-out routing gap (highest priority)

**Question:** How much of the five-model oracle's quality survives when model selection and routing rules are learned from other prompts?

Split by `prompt_id`, stratified by `dataset`, into five folds. For each fold, use only training prompts to select a pool of at most five models. Before looking at test outcomes, fix one of these routing rules: a single global model; a model chosen from the known benchmark category, if that category really is available at request time; or a learned router using prompt text/features, if those are added to the data. Evaluate those fixed decisions on test prompts. Compare against the original per-prompt oracle **as a labeled upper bound**, never as a deployable baseline. Repeat the folds with several fixed random seeds and report prompt-level uncertainty intervals.

**Show:** a score-versus-recorded-cost plot with points for the global baseline, deployable router, and oracle; a second chart showing the test-score gap to the oracle by benchmark. Include the selected-model count and the fraction of folds that actually meet the 0.80 overall and 0.70 group floors. A training-set quality constraint is not a test-set guarantee.

The current CSV has only `dataset`, `prompt_id`, `model`, `score`, and `cost`. It does **not** contain prompt text. Without additional prompt-time features, the strongest honest deployable comparisons are global and, conditionally, category-level rules. Never use the held-out prompt's realized model scores to make its routing decision.

## 2. Cost assumptions and the price of a five-model pool

**Question:** Is the five-model recommendation stable when zero-cost entries reflect realistic operating costs?

First audit the CSV: reject missing or nonnumeric costs instead of filling them with zero; document cost units, measurement date, whether token and hosting costs are included, and the coverage of each model. Then run explicit *assumption scenarios* for models whose recorded cost is zero: a positive per-request serving cost and an optional fixed per-model maintenance cost. Sweep those assumptions without calling them measured prices. Re-optimize for each scenario and record which models enter the pool, total variable cost, fixed pool cost, score, and infeasibility.

**Show:** a small sensitivity heatmap of selected pool size or total cost across serving and maintenance assumptions, plus a cost–quality frontier with scenario labels. This directly tests whether the reported zero-cost operating point is a data artifact.

## 3. Stability and specialization under changing prompt mix

**Question:** Which selected models are consistently useful, and where does the pool fail when the workload changes?

Bootstrap prompts *within each benchmark* and rerun the pool selection. Separately, choose a pool on three benchmark groups and test its available-model coverage and performance on the fourth. For every run, distinguish the chosen pool from the routing rule used on test prompts; use a precommitted global fallback for an unseen category. Track model-selection frequency, score and cost by benchmark, and how often the quality floors fail. Check whether every selected model has outcomes for each test prompt; missing pairs need an explicit fallback or must count as unroutable.

**Show:** a model-selection-frequency heatmap, with rows for models and columns for prompt mixes or held-out groups, plus interval bars for test score. This is more informative than a single five-model list.

## Execution gate

The original `routerbench.csv` used for the saved notebook run is not in this repository or the checked local project/download folders. The saved notebook output is insufficient to reconstruct prompt-level splits or cost scenarios. To execute these evaluations, provide that CSV or a documented replacement with the same columns. For a genuinely prompt-aware learned router, also provide prompt text or other features available *before* inference.

Before publishing results, verify unique `(dataset, prompt_id, model)` rows, consistent `prompt_id` use across groups, score bounds, nonnegative finite costs, and a documented policy for missing prompt–model outcomes. Keep split IDs, seeds, solver configuration, cost assumptions, and all generated figures with the reported metrics. If a public replacement dataset is used, label the results as a new study rather than a reproduction of the course paper. The official [RouterBench code](https://github.com/withmartian/routerbench) and [dataset](https://huggingface.co/datasets/withmartian/routerbench) are potential starting points, but their schema and model coverage must be checked before comparison.
