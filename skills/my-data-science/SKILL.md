---
name: my-data-science
description: A step-by-step data science roadmap for coding agents. It takes any data problem from defining the decision through data audit, EDA, evaluation design, modeling, decision policy, stakeholder dashboard and monitoring. Each phase has the questions to ask, checklists, decision rules, cost-based metrics and exit gates. Use it whenever the user mentions "data science" or asks for data analysis, EDA, visualization, dashboards, KPIs, regression, classification, churn/fraud/risk scoring, recommender systems, market basket or pattern mining, forecasting, anomaly or outlier detection, clustering or segmentation, A/B tests, model evaluation, thresholds, ensembles or AutoML. It works for any domain, dataset or tool.
---

# My Data Science Roadmap

A methodology for solving real problems with data. It is not a list of algorithms. The algorithm is the easy part: an agent can write any model in seconds. The hard parts, where projects succeed or fail, are **framing, data trust, honest evaluation, cost-aware decisions and clear communication**. Follow the phases in order. Every phase ends with a **gate**. Do not move on until the gate passes, or until the user explicitly accepts the risk.

> The impact of the work usually ranks: **(1) problem framing and success metric → (2) data and labels → (3) features → (4) model choice and tuning.** Most effort goes to #4. Spend yours on #1–3.

## Operating rules for the agent

1. **Ask before you assume.** When a phase lists "questions to ask", put the important unanswered ones to the user, at most four at a time and each with a recommended default. Never invent costs, capacities or definitions silently. If you must assume, state the assumption and mark it as one.
2. **Decision first, chart/model second.** Always know *what decision this informs* and *what result would change it*.
3. **Try to prove yourself wrong.** For every finding, write down the alternative explanations and the evidence that would weaken it *before* you look.
4. **Baselines always.** Nothing is "good" until it beats the simplest credible alternative and today's business process, on the same data and metric.
5. **Simulate deployment.** Evaluate only on information that would exist at decision time, with a split that mirrors real use. Open the test set once.
6. **Translate to money and actions.** Report business outcomes such as cost, saved value, workload and risk alongside technical metrics. Never report accuracy alone.
7. **Report ranges, not points.** Every key number gets an uncertainty band, a sample size and a caveat.
8. **Distrust clean-looking output.** Amazing results usually mean leakage or a broken evaluation. Check baseline, consistency, units, warnings, pictures and stability across seeds.
9. **Climb the complexity ladder only as far as the evidence pays.** Each rung costs build, explain and maintenance effort.
10. **Keep a decision log.** Record assumptions, choices, rejected options and the reasons in the project. It becomes the methodology section of the final report.

What to delegate and what to own: let tools generate boilerplate, reshaping code, first-draft charts and palettes. Own the question, the fairness of each comparison (denominator, baseline), what *not* to show, the audience and stakes, whether a change is real or noise, and whether the result is honest and useful for the decision.

## The roadmap

| # | Phase | Core question | Deliverable | Reference |
|---|---|---|---|---|
| 1 | Frame the decision | What decision, for whom, at what time, and what does each mistake cost? | Problem statement + cost matrix + success criteria | [01-frame-the-problem.md](references/01-frame-the-problem.md) |
| 2 | Audit the data | Can this data be trusted, and does it contain the answer at decision time? | Data audit report + leakage audit | [02-data-audit.md](references/02-data-audit.md) |
| 3 | Prepare data & features | What representation lets a simple model see the signal? | Reproducible prep pipeline + feature list with availability times | [03-data-preparation.md](references/03-data-preparation.md) |
| 4 | Explore (EDA) | What patterns exist, do they survive segmentation, and why? | Ranked findings → insights | [04-eda.md](references/04-eda.md) |
| 5 | Design the evaluation | How will we judge success honestly, before any model exists? | Split plan, metric set, baseline ladder, cost function | [05-evaluation-design.md](references/05-evaluation-design.md) |
| 6 | Model | Which approach fits the problem type, data and constraints? | Candidate models vs baselines | [06-modeling-playbooks.md](references/06-modeling-playbooks.md) |
| 7 | Validate & diagnose | Is the improvement real, robust and safe in every segment? | Diagnostics, uncertainty, error analysis, locked choice | [07-validate-and-diagnose.md](references/07-validate-and-diagnose.md) |
| 8 | Turn predictions into decisions | Which policy (threshold, ranking, routing, quantile) minimises cost under constraints? | Decision policy with expected cost and workload | [08-decision-policy.md](references/08-decision-policy.md) |
| 9 | Communicate & build the dashboard | What does each audience need in order to act? | Stakeholder dashboard + executive summary | [09-communicate-and-dashboard.md](references/09-communicate-and-dashboard.md) |
| 10 | Deploy, monitor & iterate | Does it keep working, and how will we know when it doesn't? | Pilot plan, monitoring spec, model card, feedback loop | [10-deploy-and-monitor.md](references/10-deploy-and-monitor.md) |

Phases loop. EDA often sends you back to framing, and diagnostics often send you back to data. A pure analysis question, with no model needed, runs 1 → 2 → 4 → 9. A predictive system runs all ten phases.

## Phase gates (quick checklist)

1. **Frame**: one-sentence problem statement exists ("For [unit], using information available at [time], predict/estimate [target] over [horizon], so that [owner] can [action]"); costs of each error type and the capacity constraint are known or flagged as assumptions; current process and its performance are known; problem type is chosen.
2. **Audit**: grain, duplicates, definitions, missingness mechanism, exclusions, censoring, process changes and label quality are checked; every candidate feature is classified as available or not available at decision time.
3. **Prepare**: every transformation is learned on training data only; the pipeline is reproducible; outliers are investigated, not deleted blindly.
4. **EDA**: findings pass the "so what?" test, survive segmentation and time, and have alternative explanations considered; they are ranked by impact × feasibility × confidence.
5. **Evaluation design**: the split mirrors deployment; metrics match costs, imbalance and capacity; the baseline ladder is defined; the test set is locked.
6. **Model**: at least one simple baseline and one stronger model are compared on the same split and metric.
7. **Validate**: the gain over baseline holds with uncertainty (paired comparison); no leakage signature; segments checked; final test opened once.
8. **Decide**: the policy is chosen by expected cost under constraints; stakeholders own the cost numbers and the threshold.
9. **Communicate**: every chart passes the 4-level audit (code, statistics, perception, decision); the dashboard shows outcome, comparison, uncertainty and recommended action.
10. **Deploy**: pilot or A/B plan with guardrails exists; monitoring covers data drift, performance, calibration, workload and cost; there is a rollback trigger.

## Problem-type router (used in phase 1)

| If the decision needs… | Problem type | Playbook section |
|---|---|---|
| A number (amount, time, demand) for a case | Regression | Regression |
| One of a fixed set of labels, known for past cases, where each label maps to a different action | Classification (binary / multi-class / multi-label) | Classification |
| An ordered shortlist of options per person/context | Ranking / recommendation | Recommenders |
| "What happens together / what follows what" rules | Pattern mining (association / sequential) | Pattern mining |
| Future values of a time-indexed quantity | Forecasting | Forecasting |
| "Is this normal?" with few or no labels | Anomaly detection | Anomaly detection |
| Natural groups with no labels | Clustering / segmentation | Clustering |
| "What happens if we intervene?" | Causal inference / experiment (A/B test) | Causal note |
| A description or explanation of what happened | Descriptive analysis (EDA + communication) | Phases 2, 4, 9 |

Signals that you may not need machine learning: the decision would not change, a rule gets almost all the value, the cost of being wrong is near zero, there is no relevant history (new market, regime shift, adversarial setting), or the event is too rare to learn. Use scenario analysis, expert judgment, simulation or simple rules instead.

## Standard outputs of a complete project

1. Problem statement and decision log (phase 1, updated throughout).
2. Data audit and leakage audit (phase 2).
3. EDA findings memo, ranked (phase 4).
4. Evaluation plan, written before modeling (phase 5).
5. Model comparison table: baselines vs candidates, technical **and** business metrics, with ranges (phases 6–7).
6. Decision policy with expected cost, workload and sensitivity to cost assumptions (phase 8).
7. Stakeholder dashboard and a one-page executive summary (phase 9).
8. Model card, monitoring plan and pilot/rollout plan (phase 10).
