# Phase 5 — Design the evaluation (before building any model)

Goal: decide how success will be measured so that the reported performance predicts real-world performance. **Evaluation must simulate deployment.** Write this plan before modeling, so that the results cannot shape the yardstick.

## Steps

1. **Assign data roles.**
   - **Training** (about 60–70%): learn parameters and all preprocessing.
   - **Validation** (about 15–20%, or cross-validation inside training): choose features, models, hyperparameters, thresholds and policies.
   - **Test** (about 15–20%): a sealed vault, opened **once** for the locked choice.
   - Optionally keep a small **expert-labelled gold set** covering rare, costly and ambiguous cases.
2. **Choose the split strategy that mirrors deployment.**
   - **Random**: large, independent rows with no time or entity structure.
   - **Stratified**: rare positives, so class proportions stay equal across splits. Stratification doesn't fix leakage.
   - **Temporal (out-of-time)**: the model will score the future. Train on the past and test on later periods. Leave a **gap** when label windows would overlap the test period.
   - **Group**: repeated entities (customer, patient, clinic, store) appear on one side only. Use it when the model must work for unseen entities.
   - **Temporal + group**: unseen entities in a future period.
   - **Rolling-origin backtest** for forecasting: repeatedly train up to time t, forecast the horizon, and move t forward.
   - **k-fold cross-validation** (k = 5–10) for stable estimates on small data. Report the mean **and** the spread across folds. Use grouped or time-ordered folds when needed.
3. **Define the baseline ladder** and score every rung on the same split and metric.
   - **Regression**: global mean or median → segment mean → current business rule or expert estimate → simple linear model.
   - **Classification**: majority class → current rule or process → simple logistic regression or shallow tree.
   - **Forecasting**: naive (last value) → seasonal naive → moving average.
   - **Recommenders**: most popular or trending → segment, context or "bought together" → personalised.
   - **Anomaly detection**: business rules → statistical deviation (z-score or IQR).
   - The value created is the **gap between baseline and model**. If the gap is tiny, ship the baseline.
4. **Pick metrics that match the business** (see the catalogue below). Use one **primary decision metric**, usually expected cost or value, plus supporting technical metrics and guardrails.
5. **Write the cost function** from the phase 1 cost matrix and normalise it, for example cost per 1,000 cases. Include review or labour costs and capacity limits.
6. **Plan segment evaluation.** Choose the critical segments: customer tier, geography, new vs existing, product, channel, and lawful demographic groups where appropriate. An 88% overall score can hide total failure in one segment.
7. **Plan the uncertainty reporting**: bootstrap confidence intervals, paired comparisons and sensitivity to cost assumptions.
8. **Set the deployment bar**: the minimum gain over baseline, with confidence, that justifies launch.
9. **Multi-component systems**:
   - Choose two or three metrics per component, plus one or two decision-level metrics. Designate one **primary ML metric** and one **primary decision metric**, and justify each in business terms.
   - Evaluate each component on its own split (for example, temporal for forecasting, user-level or leave-one-out holdout for recommenders) **and** the whole chain together.
   - Handle sequential dependencies: when one component's output feeds another, the downstream component must be trained and evaluated on **out-of-sample** upstream outputs, never on the upstream component's training-set predictions.
   - Prevent leakage across components: shared splits, no test-period information flowing upstream.
10. **Evaluate the decision layer on held-out data.** Make decisions using only information available at decision time. Score them on outcomes that actually occurred, or on clearly stated assumptions from the register. Compare them with a **baseline decision policy**, such as the current business rule or "act on the highest-risk cases until the budget runs out".
11. **Plan the deeper analyses now**: which components the ablation will remove or swap, which assumptions the sensitivity analysis will vary, and which challenge sets you will test on (see [deeper-analyses.md](deeper-analyses.md)).

## Metric catalogue

**Regression**
- MAE: typical absolute miss in real units; robust and an honest default.
- RMSE: punishes large misses; RMSE ≫ MAE signals a few spectacular failures.
- MAPE: percentage error; scale-free but undefined or unstable near zero and biased by small actuals.
- R²: improvement over predicting the mean; relative, so domain-dependent; it can be negative.
- Bias: mean of (prediction − actual), which shows systematic over- or under-prediction.
- Pinball (quantile) loss when a quantile is the decision.
- Choose by the shape of the pain: large misses especially costly → RMSE; relative error across very different scales → MAPE (avoid near zero); typical miss → MAE.

**Classification**
- Accuracy is meaningful only with balanced classes and symmetric costs; always compare it with the majority-class baseline (the base-rate trap).
- Precision = TP / (TP + FP): "when we act, how often are we right?" Use it when false actions are expensive or capacity is limited.
- Recall = TP / (TP + FN): "how much of the problem did we catch?" Use it when misses are expensive.
- F1: balances precision and recall but does not encode costs.
- ROC-AUC: ranking quality across all thresholds; insensitive to prevalence, so it can look great when the event is rare.
- PR-AUC / average precision: the no-skill line is the prevalence. Use it for rare positives, and compare only on the same population.
- **Precision@K / Recall@K**: for fixed review capacity K.
- Log-loss and calibration (reliability curve): when the probabilities themselves are used.
- **Expected cost** = FP × cost_FP + FN × cost_FN (+ TP handling cost + review costs). This is the primary metric when costs are asymmetric.
- Metric selection by situation:
  - Balanced classes, symmetric costs → accuracy.
  - Balanced classes, asymmetric costs → precision, recall or cost.
  - Rare positives, symmetric costs → accuracy vs baseline plus recall.
  - Rare positives, asymmetric costs → precision and recall, Precision@K, expected cost and PR-AUC.

**Ranking / recommenders**
- Offline: hide real interactions, then measure Precision@K, Recall@K, MRR (rank of the first relevant item), NDCG, catalogue coverage and diversity.
- Online: A/B test conversion, revenue and retention with guardrails (complaints, returns, churn, diversity), measuring **incremental** impact (watch for cannibalisation).

**Pattern mining**
- Support, confidence and **lift** (always check lift; lift ≈ 1 means no real association).
- Stability across months, regions and segments.
- Economic value of the action vs its cost.

**Forecasting**
- MAE, MAPE (avoid near zero), RMSE and bias, evaluated by **horizon**.
- Prediction-interval **coverage** (a 90% interval should contain about 90% of actuals) and interval width.
- Cost of the decision rule (over- vs under-stock).

**Anomaly detection**
- Precision, recall and F1 at the alert threshold.
- Alerts per period vs analyst capacity.
- Ops cost = FP × cost_FP + FN × cost_FN.

**Clustering**
- Within-cluster tightness, separation (for example silhouette) and stability across samples and seeds.
- Above all, **actionability**: do the segments differ in ways that change decisions?

**System level** (when a model sits inside a process)
- Service quality: misroutes, costly harms, repeat contacts.
- Automation rate and the quality of automated cases.
- Human workload: queue size and age, reviewer agreement.
- Latency, cost per resolved case and subgroup reliability.

## Questions to ask

- Will the model score future cases, new entities, or both?
- Do rows repeat per entity? Do labels mature with a delay?
- What review or action capacity exists per period?
- Which segments must not be harmed?
- Which metric does each stakeholder care about (finance: losses; operations: workload; customer experience: false alarms; compliance: fairness and deadlines)?

## Deliverable: evaluation plan

Data roles and split strategy (with dates or groups) per component, baseline ladder (including the baseline decision policy), primary ML and decision metrics with justifications, cost function, capacity constraint, segments and challenge sets, uncertainty method, ablation and sensitivity design, and deployment bar. Freeze it before modeling. The evaluation code should be reproducible with one command and write a structured results table.
