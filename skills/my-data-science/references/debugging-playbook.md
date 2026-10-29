# Debugging playbook: why is the model failing, and what should we fix first?

Use this playbook when a model underperforms, when results look too good, when production performance falls below validation, or before you trust any result (phases 7 and 10). A high headline score is not a reason to ship. Inspect the failure profile that matters to the business.

The order is: **name the gap → audit the data and evaluation → explain the behaviour → analyse the errors → prioritise fixes by value → verify.**

## 1. Start from value, not accuracy

- Models with **the same accuracy can make very different mixes of mistakes** (cautious, aggressive, balanced), and so create very different value. Compare candidates on value at the real operating constraint, not on accuracy.
- Always score the **reference policies** on the same data: do nothing, a simple business rule, random action at capacity, and act on everyone. Accuracy and value often move in opposite directions. "Predict nobody" can have the highest accuracy and zero value.
- **Break-even probability for an intervention**: act when p × (success rate × value saved) > cost of acting, so the break-even point is p* = cost ÷ (success rate × value). Cases above p* are profitable to act on.
- **Capacity turns classification into ranking.** Plot value against the number of cases acted on. Find the unconstrained optimum, see how much value the capacity limit costs, and note that acting beyond the optimum lowers value.

## 2. Name the gap: the performance ladder

Compare the error rate (or miss rate on the business metric) at four levels:

| Level | Meaning |
|---|---|
| Irreducible (Bayes) error | The best achievable, given noise in the problem |
| Expert / human-level | The best realistic reference, usually above the irreducible error |
| Training | How well the model fits the data it saw |
| Validation | Generalisation within the same "world" as training |
| Production | What the business actually experiences |

Each gap points to a different suspect and a different fix:

| Gap | Name | Suspect | Typical fixes |
|---|---|---|---|
| Training − expert | **Avoidable bias** | The model is too simple or missing signal (underfitting) | Richer features, a more flexible model, less regularisation, better labels |
| Validation − training | **Variance** | The model learns noise or quirks (overfitting) | More data, stronger regularisation, a simpler model, fewer features, ensembling (bagging) |
| Production − validation | **Data mismatch** | Production is a different world, not a worse fit | Representative validation data, drift handling, retraining on recent data, domain-specific features |

If training error is already **below** expert level, there is no avoidable bias left to chase, and suspiciously low training error is a warning flag.

**Learning curves**: plot training and validation error against training-set size.
- Wide gap that narrows as data grows → variance; more data helps. Estimate whether doubling the data is worth its cost.
- Both errors high and converged → bias; more data will not help, so change the model or features.
- Validation flat → data volume is not the lever.

**Complexity curves**: plot error against model complexity (depth, number of features, regularisation strength).
- Training error keeps falling while validation error eventually rises → overfitting past that point.
- Optima are often **flat**. Among settings within the noise margin, prefer the simplest.
- More data supports more complexity.
- Beware of validation overfitting from repeated tuning; use nested cross-validation when tuning and evaluating on the same data.

## 3. Audit the data and the evaluation (before believing any result)

**Leakage routes** (ask of every feature: "would this value exist at prediction time?"):
- the target hidden inside a feature
- future information
- preprocessing fitted on all data instead of training data only
- duplicate or grouped rows across splits

**Split contamination**:
- A row-level random split can put snapshots of the same entity in both training and test. A time split alone is not enough if entities or label windows overlap.
- Default for entity data over time: **grouped by entity and time-ordered**. Each entity sits in exactly one of training, validation or test, and training precedes test.
- Tuning hyperparameters on the same data used to report performance contaminates the estimate.

**Four quick diagnostics**:

| Test | How | How to read it |
|---|---|---|
| **Fit a tiny subset** | Train a flexible model on about 10 hand-checked rows | Should reach near-perfect training accuracy. Passing proves little. **Failing** reveals bugs: identical inputs with different labels, label-encoding errors, a broken target, preprocessing bugs, over-regularisation. |
| **Shuffled-label test** | Shuffle the labels in training only (repeat many times), score on the real validation labels | Ranking quality should be at chance (AUC ≈ 0.5). Above-chance scores mean label information is leaking through preprocessing or evaluation. It cannot say which step leaks. |
| **Single-feature trap** | Score a model on each feature alone | Any single feature with very high ranking quality (for example AUC above about 0.85) needs interrogation: it may be legitimate, a proxy or leakage. Also check **lead time**: a real signal that only appears late (for example, after a long period of inactivity) may arrive too late to act on. |
| **Adversarial validation** | Train a classifier to tell training rows from validation or production rows | AUC ≈ 0.5 means the sets look alike; a high AUC means a distribution shift. Inspect the features that drive the separation and check whether errors rise there. It shows *where* distributions differ, not whether the difference hurts. |

**Label noise** (accuracy is capped by how consistent the labels are):
- Find likely label errors with confidence-based flags (the model confidently disagrees with the label), ensemble disagreement (most of several models disagree), and out-of-fold cross-validation predictions that keep disagreeing. Dedicated label-cleaning libraries can automate this.
- Have two or three experts audit the flagged cases, recording their disagreements. Classify each case as a label error, a genuinely hard case or a model bug, and estimate the share of each.
- Label noise is rarely uniform; it often concentrates in particular segments. Fix the labels **and** the labelling process.

## 4. Explain the behaviour (with the right tool for the question)

An explanation answers a specific question. Interpretation tools **generate hypotheses**; ablations and experiments **test** them.

| Question | Tool | Caveats |
|---|---|---|
| Which features does performance depend on? | **Permutation importance**: shuffle one feature on validation data and measure the drop in the metric | Correlated features share credit and can each look unimportant. It shows what matters, not why, and not whether it is used correctly. |
| How did each feature push *this* prediction relative to a reference? | **Local attributions** (for example SHAP): contributions relative to a baseline prediction; aggregate them for a global view | For classifiers the contributions are often in log-odds, so convert them to probabilities when explaining. Correlated features complicate attribution. It describes model behaviour, not causality or correctness. |
| How does the average prediction change as a feature varies? | **Partial dependence** (plus individual curves to see differences between cases) | It can create unrealistic feature combinations when features are correlated, and averaging hides individual differences. |
| What minimal change would flip this outcome? | **Nearest counterfactual**: the most similar real case with the opposite outcome | Reject implausible changes and changes to immutable features. Check stability under small perturbations. |

## 5. Error analysis (the highest-yield, most-skipped activity)

Aggregate metrics tell you how much the model fails, never why.

1. **Sample about 100 errors**: a random sample to avoid anecdote bias, plus cases sorted by confidence (confident mistakes are the most informative). It takes a few hours and often saves weeks.
2. **Tag each error** with a taxonomy, ideally by two or three reviewers, measuring how often they disagree:
   - **Label error**: the target is wrong.
   - **Out of scope**: the case is outside the intended deployment.
   - **Missing information**: the signal isn't in the features.
   - **Data quality**: the signal exists but arrives broken (duplicates, outages, bad imputation).
   - **Edge case**: a rare configuration with too few examples.
   - **Model weakness**: the signal is present and clean but still missed.
3. **Quantify**: the share of each category with a confidence interval, extrapolated to the total number of errors.
4. **Error trees**: train a simple tree to predict *whether the model is wrong* from the original features. Its rules reveal high-error subgroups.
5. **Segment and intersectional slicing**: slice performance by what matters to the business, to harm and to the law, and by combinations (region × tier × product). Report **both the proportion and the raw count**, because small slices are noisy. A strong overall score can hide a segment where almost nothing is caught.

## 6. Prioritise, fix and verify

- **Ceiling analysis by segment**: the overall metric if a segment were fixed perfectly equals current correct cases plus that segment's errors, divided by the total. Weight it by business value: a small segment with high value per case can matter more than a large cheap one.
- **Ceiling analysis by error type**: the value at stake = errors in the category × cost per error. Multiply by a realistic **recovery rate** (how much a fix can actually recover). Out-of-scope cases and label errors usually recover nothing from model work.
- **Rank fixes** by potential ROI = (mistakes in the category × cost per mistake × recovery rate) − cost of the fix. Give optimistic and realistic estimates, discounted for effort, uncertainty and risk.
- **Stop debugging** when the marginal cost exceeds the expected business value.
- **Verify every fix** on held-out data with the decision-level metric, and re-check the vulnerable slices. Make sure a fix for one segment didn't break another.

## Checklist

1. Did we compare candidates on value at the real constraint, against do-nothing, rule-based and random baselines?
2. Which gap is largest: avoidable bias, variance or data mismatch?
3. Did the tiny-subset, shuffled-label, single-feature and adversarial-validation tests pass?
4. Is the split grouped and time-ordered, with tuning kept separate from evaluation?
5. Were label errors estimated and audited by experts?
6. Do the explanations make domain sense, and were the hypotheses they suggested tested?
7. Were about 100 errors categorised, with shares and intervals?
8. Were fixes ranked by ceiling × recovery − cost, and verified on the decision metric and the vulnerable slices?
