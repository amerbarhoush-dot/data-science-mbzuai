# Phase 10 — Deploy, monitor and iterate

Goal: make sure the solution keeps creating value in the real world, and detect quickly when it stops. Optimise and monitor the **whole system** (model + rules + humans + process), not only the model.

## Steps

1. **Go/no-go check**:
   - Does it beat the baseline and the current process on the primary business metric, with uncertainty, on an out-of-time or held-out test?
   - Are segments acceptable and the constraints met?
   - Are the explanations and governance requirements satisfied?
   - Possible verdicts: **launch**, **pilot first**, or **do not launch** (and say what new evidence is needed).
2. **Roll out safely**:
   - **Shadow mode**: the model runs alongside the current process without acting, and the decisions are compared.
   - **Limited pilot**: a subset of traffic or regions.
   - **A/B test**: randomised, with a pre-registered primary metric, guardrails (complaints, returns, churn, fairness, latency), enough sample size and duration, and measurement of **incremental** impact.
   - Define the **rollback triggers** in advance.
3. **Production parity**: features are computed identically and with the expected freshness in production. Version the data, the code and the model, and log inputs and outputs for audit.
4. **Monitoring plan** (what, how often, who is alerted, and what threshold triggers action):
   - **Data**: volume, missing rates, schema changes, and feature distribution drift compared with training.
   - **Predictions**: score distribution, flag rate and workload against capacity, abstention or escalation rate.
   - **Performance**: precision, recall and error once labels arrive, accounting for **label delay**; calibration and interval coverage.
   - **Business**: cost or value realised, guardrail metrics, segment performance.
   - **System**: latency, cost per case, queue age, reviewer agreement, rework.
5. **Feedback loops and bias**: models change the behaviour they observe (popularity bias, intervention bias, adversaries adapting). Keep exploration slots or holdout groups, and record what was *shown* or *acted on* so that future training data isn't just a mirror of past decisions.
6. **Learning loop**: capture human corrections and investigation outcomes as labels. Maintain a small trusted expert-labelled evaluation set. Improve labels cheaply:
   - **Active learning**: label the most informative cases first (uncertainty plus diversity, or highest decision regret).
   - **Weak supervision**: rules, old systems or LLM labels combined and checked against the gold set.
   - **Teacher–student**: an expensive model labels data, and a cheap model is trained to serve it.
7. **Retraining policy**: on a schedule or triggered by drift. Every retrain is re-validated against the current model and the baseline on fresh out-of-time data before promotion.
8. **Governance and documentation**: a **model card** covering purpose, data, features, metrics, segments, limitations, safe operating range, fairness checks, owners, update history and how to contest a decision. Also cover privacy (minimum necessary data, legal obligations), explainability for affected people, and an audit trail for overrides.
9. **Decommission criteria**: when the model no longer beats a simpler alternative, or the decision it serves has changed.

## Questions to ask

- What exactly will be different on day one of launch, and who is affected?
- How long until true outcomes (labels) are known?
- What level of drift or performance drop triggers a review, and who decides?
- How will we know whether the model changed behaviour in a way that biases future data?
- Who maintains the model, the dashboard and the cost assumptions?

## Deliverable

Launch decision memo · pilot or A/B plan with guardrails and rollback triggers · monitoring specification · retraining and labeling plan · model card · owners and the review cadence.
