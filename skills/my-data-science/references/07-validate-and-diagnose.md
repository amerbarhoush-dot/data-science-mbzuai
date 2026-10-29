# Phase 7 — Validate and diagnose

Goal: establish whether the improvement is real, robust, safe across segments and free of leakage, and understand *where and why* the model fails.

## Steps

For deeper debugging, including the performance ladder, learning curves, leakage and label-noise diagnostics, explanation tools, the error taxonomy and ceiling analysis, follow [debugging-playbook.md](debugging-playbook.md).

1. **Fit diagnosis (train vs validation gap)**. Place the model on the ladder: expert/human level → training → validation → production. Training − expert is avoidable bias, validation − training is variance, production − validation is data mismatch.
   - Both low with a small gap → **underfitting**: add signal or features, or reconsider whether the signal exists.
   - Both high with a small gap → healthy.
   - Training far above validation → **overfitting**: simplify, regularise or get more data.
   - Both near perfect → **almost certainly leakage**. Re-run the leakage audit before believing it.
2. **Residual and error patterns** (regression and forecasting):
   - Plot actual vs predicted (the 45° line), residuals vs predicted and residuals vs key features.
   - A curve means a missing nonlinearity or transform.
   - A funnel means heteroscedasticity.
   - Clusters by group mean a missing categorical effect or interaction.
   - A trend over time means drift or a missing time feature.
   - Extreme points mean outliers or data errors.
   - Check bias (the systematic direction of error). Large individual misses matter even when the average looks fine.
3. **Error analysis** (classification and other tasks). Sample about 100 errors, tag them with the taxonomy (label error, out of scope, missing information, data quality, edge case, model weakness), then rank fixes by ceiling × recovery rate − cost (see the playbook):
   - Inspect samples of false positives and false negatives.
   - Slice errors by segment, time, input length, language and data source.
   - Find the costliest error types first.
   - Decide whether the fix is more data, better labels, a new feature, a specialist model or a rule.
4. **Segment performance.** Report the primary metrics for each critical segment. Never accept a good average that hides a failing segment. Check fairness across lawful groups where relevant.
5. **Uncertainty**:
   - Bootstrap the evaluation set without retraining, resampling the right unit (entities, not rows, when rows repeat). Report percentile intervals.
   - Compare models **pairwise on the same resamples**: if the interval of the difference spans zero, the evidence is insufficient (it is not proof that they are equal).
   - Compare the effect size with a business-relevant minimum, not just with zero.
   - Small test sets and few positives give wide intervals.
6. **Stability**: re-run with different seeds and time windows. Rankings that flip are noise.
7. **Calibration** (when probabilities drive decisions): compare predicted and observed rates with a reliability curve. Recalibrate if needed. Class weighting and resampling distort probabilities.
8. **Prevalence sensitivity**: if the base rate may change in deployment, show how precision, workload and cost move. ROC-AUC will not show this.
9. **Validation overfitting (the winner's curse)**: many trials against the same validation set inflate its score. Every experiment counts, including ones directed by an LLM. Keep the number of trials modest and log them all.
10. **Lock the choice**, including model, features, hyperparameters and decision policy, **then open the test set once**. Do not change anything after seeing the test results. If they disappoint, report it and plan a new cycle with fresh holdout data.
11. **Prove each component earns its keep**: run the **ablation** (remove or swap one component at a time and re-score the decision metric), the **sensitivity/stability** analysis, **challenge-set evaluation** on hard slices (cold start, rare patterns, extreme values, volatile periods), and **oracle/headroom** analysis to find where improvement effort should go. See [deeper-analyses.md](deeper-analyses.md).
12. **Explainability check**: global (feature importance, coefficients) and local (why this case). Confirm the explanations make domain sense; a nonsensical top feature often reveals leakage.

## Questions to ask

- If this result looks amazing, what could be wrong with the evaluation?
- Which errors are the most expensive, and where do they concentrate?
- Is the gain over baseline larger than the noise between seeds, folds and months?
- Would a stakeholder accept the explanation for the top drivers?
- Does the model behave sensibly outside the training range?

## Deliverable

A validation report covering:
- train vs validation diagnosis
- residual and error analysis findings
- segment table
- uncertainty intervals and paired comparisons
- calibration
- leakage re-check
- the locked configuration
- the single final test result with its interval
