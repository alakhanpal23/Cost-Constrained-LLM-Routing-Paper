# Cost-Constrained LLM Routing with a Bounded Model Pool

**IND ENG 164 · course paper and optimization notebook**

How should a system choose a small set of language models, then assign prompts to those models while meeting quality targets at the lowest measured cost? This project formulates that question as a mixed-integer optimization problem in Pyomo. It explores model-pool size, minimum quality, benchmark-specific floors, score uncertainty, and changes in the prompt mix.

**[Read the full paper (PDF)](paper.pdf)** · [Technical summary and limitations](PAPER.md) · [Explore the notebook](IND_ENG_Final_Project.ipynb) · [Next evaluations](EVALS.md)

The [15-page paper](paper.pdf), titled *Cost-Aware LLM Routing for AI Coding Assistants: A Two-Stage Stochastic and Robust Optimization Approach*, is also maintained in [Overleaf](https://www.overleaf.com/project/69fa6a312464f606f451d7ce) (project access is limited to collaborators). The PDF here is the public reading copy.

The saved notebook run contains 240 prompts, 33 candidate models, and four benchmark groups. With at most five selected models, a minimum average score of 0.80, and a 0.70 floor for each group, its in-sample solution scores **0.8125** at **0 recorded cost**. That zero comes from the input CSV's cost values; it is **not** a claim that serving those models is free.

## What the work demonstrates

- Binary decisions for model selection and prompt assignment under a pool-size limit.
- An explicit cost–quality frontier and sensitivity checks for score margins and prompt distribution.
- A reproducible optimization *method*, with the original input data and validation limits clearly disclosed.

## Reproduce the analysis

Open the notebook in Jupyter or Google Colab and provide a local `routerbench.csv` with `dataset`, `prompt_id`, `model`, `score`, and `cost` columns. The CSV used for the saved run is **not included**, so the reported numbers cannot be independently regenerated from this repository alone. The notebook uses pandas, NumPy, Matplotlib, Pyomo, and HiGHS via `highspy` (with a GLPK fallback).

The optimizer sees each candidate model's outcome on each prompt before assigning that prompt. The reported score is therefore an **in-sample oracle result**, not a measured routing policy for unseen prompts. See the [technical summary's limitations](PAPER.md#limitations-and-next-steps) before using the results for deployment or comparison with a learned router.
