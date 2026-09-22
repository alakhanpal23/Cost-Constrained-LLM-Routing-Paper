# Model routing optimization notebook

This course project uses Pyomo to select a small pool of language models and route prompts to them under quality and cost constraints. The notebook reads RouterBench-style rows with `dataset`, `prompt_id`, `model`, `score`, and `cost` columns, then formulates an assignment and model-selection problem.

## Run

Open `IND_ENG_Final_Project.ipynb` in Jupyter or Google Colab. Provide a local `routerbench.csv` when prompted. The dataset is not included in this repository.

The notebook uses pandas, NumPy, Matplotlib, Pyomo, and the HiGHS solver through `highspy`; it can fall back to GLPK when available. Review the notebook's assumptions and input validation before applying its results to another dataset.

## Scope

This is a research and coursework notebook. Costs and quality scores come from the supplied CSV, so results depend on that dataset and the selected constraints. They are not live model pricing or a current benchmark claim.
