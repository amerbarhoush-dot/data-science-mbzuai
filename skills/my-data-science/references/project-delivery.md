# Project delivery: milestones, reproducibility and documentation

How to deliver a data science project so that others can understand it, rerun it and trust it. This applies to any project, whatever the language or tools.

## Staged milestones

1. **Proposal** (before touching data in depth). Keep it to two pages or less, written for a domain expert without a deep technical background:
   - Title: the domain plus the analytical ambition.
   - Business context and decision: the problem, why it matters, who decides, what is done poorly today and what a better system would enable.
   - Dataset: the source, rough size, key attributes, the target of each component, any extra data needed and how to get it.
   - Data ethics and confidentiality: permission to use the data, plus a masking and aggregation plan.
   - Analytical plan: the techniques combined and how they connect into the decision, the baseline, the planned ablation and sensitivity tests, and the deeper analyses chosen (see [deeper-analyses.md](deeper-analyses.md)).
   - Expected output and value: the working artefact (scoring tool, decision-support app, scenario simulator, dashboard), the decision it supports, and the analytical and decision-level success metrics.
   - Scope the ambition to the team's size and time. A large team with a small idea reads as under-ambitious.
2. **Intermediate milestone: a thin, working end-to-end slice.** Before any tuning, have all of these working:
   - The finalised problem definition.
   - Data that loads and prepares reproducibly.
   - The evaluation plan.
   - EDA.
   - **Simple baselines plus crude first versions of each core component, feeding a basic decision policy, scored end to end.**
   - At least one decision-level result: for example, the baseline policy vs the policy using the first-pass models under the constraint.
   - A credible plan for the ablation, the sensitivity analysis and the deeper analyses.
   - Any scope changes since the proposal, with the reasons.
3. **Final**: iterate on the components, run the ablation, sensitivity and deeper analyses, add the stakeholder dashboard, and write up the recommendation with its limitations.

Scope changes are normal once you are inside the data. Record what changed and why.

## Composite systems: connect the components

When a problem combines techniques (for example, a classifier with a recommender, regression with forecasting, or anomaly detection with classification), they must form **one connected system**:
- the output of one component feeds another, or feeds the decision
- the decision layer chooses actions under a real constraint (budget, capacity, risk appetite)

Draw a simple diagram: data → components → decision. Avoid two unrelated analyses stapled together, and avoid a model that never becomes a decision.

## Reproducibility checklist

- **Environment**: a project file declaring the dependencies with version constraints, a committed lock file, a pinned language version, and dev dependencies (tests, notebooks) kept separate from runtime ones. Install everything through the project's package manager, never with ad-hoc installs.
- **One-command pipeline**: a task runner (for example a Justfile or Makefile) with named steps:
  - install
  - download-data → `data/raw/`
  - prepare-data → `data/processed/`
  - eda (charts → `outputs/`)
  - run-baseline / run-models (predictions → decision policy → evaluation)
  - test
  - pipeline (all of the above in order)
  - clean (remove generated artefacts; keep or separately remove the raw data)
  The steps "install" then "pipeline" should run end to end on a clean checkout in a reasonable time, and the README states that runtime.
- **Determinism**: fixed random seeds everywhere. The same raw data and seed give the same processed data and results.
- **Portability**: no absolute paths, no hard-coded credentials or API keys, no GPU requirement unless essential. It runs on a clean machine or container.
- **Data access**: a scripted download with a stable source, and a checksum where possible. If the data cannot be downloaded automatically, commit a copy when the licence and size allow, or document the manual steps.
- **Assumptions as configuration**: quantities the decision layer needs but the data lacks (cost per intervention, value of a retained customer, capacity) live in one config file. Each has its source or rationale and is never silently hard-coded. This is what makes sensitivity analysis possible.
- **Structured outputs**: metrics and results saved as tidy tables (for example CSV) under `outputs/` or `results/`, ready for charting and the dashboard.
- **Tests**: at least minimal tests of data preparation, metric calculations and decision-policy logic.

## Suggested project layout

- `README.md`: purpose, how to run, runtime, data access notes
- `data/raw/`, `data/processed/`
- `src/`: data loading and preparation, baselines, models, decision policy, evaluation
- `scripts/`: download, prepare, EDA entry points
- `notebooks/`: exploration only, not the source of truth
- `tests/`
- `outputs/`: charts, result tables, model artefacts
- `docs/`:
  - `project_definition.md`: decision, system design, success criteria, constraints, assumptions
  - `data_dictionary.md`: column, type, meaning, transformations, known limitations
  - `baseline_report.md`: methods, settings, results table, interpretation, limitations, next steps

## Data ethics and confidentiality

- Confirm you are allowed to use the data. For internal data, get explicit permission from its owner.
- Strip or mask personal identifiers: names, IDs, exact addresses and anything traceable to a person.
- Aggregate or bucket sensitive fields (for example, salary bands instead of exact salaries).
- Never commit confidential raw data to a shared repository.
- When in doubt, leave it out, or use a public dataset or a clearly labelled synthetic one.

## Writing for the reader

- Keep documents short: a couple of paragraphs per section, diagrams where they help, and tables for results.
- Write for someone outside data science. Lead with the decision and the business impact.
- Code that others will read: sensible file and function names, short docstrings, organised modules.
- Always include honest limitations and what the next iteration must address.
