# Phase 8 — Turn predictions into decisions

Goal: convert scores, forecasts or rankings into an operating policy that minimises expected cost under real constraints. **A score is not a decision.** The same model with different cut-offs is a different product with different costs. The business owns the cut-off; the data scientist shows the trade-offs.

## Steps

1. **Rebuild the cost matrix** with stakeholders: the value of a TP (prevented loss minus handling cost), the cost of an FP (wasted work plus customer friction), the cost of an FN (the loss) and the cost of a TN (usually 0). Include non-financial costs as notes or penalties.
2. **Sweep the policy on validation data, never on test.** For each threshold record:
   - number flagged (workload)
   - TP, FP and FN
   - precision and recall
   - **total expected cost**, compared with doing nothing and with the current process
3. **Choose the operating point** by the business objective:
   - **Minimum expected cost**: theory gives a cut-off of cost_FP / (cost_FP + cost_FN) when probabilities are calibrated and there is no review option. Confirm it empirically on validation.
   - **Fixed capacity K**: rank by score and act on the top K. Report Precision@K and Recall@K. A fixed K keeps workload steady; a fixed cut-off keeps quality steady.
   - **Constraint-first**: for example, "recall at least 70%" or "precision at least 25%". Show what each constraint costs compared with free cost-minimisation.
4. **Consider multi-tier decisions**:
   - **Two thresholds**: auto-approve below the low one, human review between them, auto-act above the high one. This pays only if reviewers are accurate enough, cheap enough and have the capacity.
   - **Abstention or confidence routing**: automate only when confident; trade coverage for accuracy using a coverage-vs-accuracy curve. The accuracy figure applies to the automated cases only.
   - **Abstention by disagreement**: when diverse models disagree, escalate. Measure whether disagreement really predicts error.
   - **Cascades**: a cheap model handles the easy cases, and a stronger, costlier model (or a human) handles the hard ones. Judge by cost per case. For rare events: a high-recall screener, then a high-precision verifier, then a human.
   - **Hierarchical fallback**: when unsure of the fine category, route to the safe parent category.
   - **Expected-cost routing** over several actions: for each case, compute the expected cost of every possible action (including labour, queue delays and SLA penalties) and pick the cheapest.
   - **Value of information**: pay for an extra signal (an expert label, an extra check) only when the expected cost saved exceeds its fee.
5. **Rules around the model**:
   - **Before**: deterministic cases routed directly.
   - **As features**: rule hits given to the model as inputs.
   - **After**: mandatory policy overrides that block impermissible actions, with conflicts logged.
   - If you can state a rule with certainty, don't ask a model to learn it.
6. **Humans in the loop.** Low-confidence or high-risk cases go to people, with the context pre-filled. Their decisions become new labels. Log every model-vs-rule conflict.
7. **Ranking-value policies.** Rank by **risk × value** (for example, churn probability × customer lifetime value) when an action costs the same for every case but the payoff differs.
8. **Quantity decisions** (forecasting and regression):
   - When over- and under-provisioning have different costs, choose the quantile q* = cost_under / (cost_under + cost_over) (the newsvendor critical fractile), not the mean.
   - Use upper quantiles as buffers where shortages are costly.
   - Compare against a strong heuristic, such as the same period last cycle.
9. **Sensitivity analysis.** Re-run the policy with costs, prevalence and capacity varied, for example ±50% or a cost shock. Report whether the recommended policy and model still win. Name the inputs the decision depends on most.
10. **Decision frontier, regret and reconciliation.**
    - Show how the optimal policy shifts as budget, capacity or risk appetite changes.
    - Measure mistakes by the value lost (**decision regret**), not by raw error counts.
    - Define a principled rule for cases where components disagree (cross-model reconciliation).
    - Make multi-objective tensions explicit with a trade-off curve.
    See [deeper-analyses.md](deeper-analyses.md).
11. **Scenario and what-if analysis.** Offer three to five named scenarios that stay within the model's valid range. Executives engage with scenarios, not coefficients.

## Questions to ask

- Who owns each cost number, and how certain is it?
- What is the review capacity per period, and how accurate are the reviewers?
- Are there actions that must never be automated?
- Which stakeholders have conflicting objectives (finance vs customer experience vs operations vs compliance), and who arbitrates?
- What should happen to cases the system cannot decide?

## Deliverable: decision policy sheet

Chosen policy (thresholds, top-K, tiers or routing rules) · expected cost per period and per 1,000 cases vs doing nothing and vs the current process · workload and automation rate · precision and recall at the operating point · constraints satisfied · sensitivity results · rules and human-review design · owner sign-off items.
