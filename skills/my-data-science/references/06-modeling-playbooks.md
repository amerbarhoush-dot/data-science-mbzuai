# Phase 6 — Model (playbooks by problem type)

Goal: find the simplest approach that clearly beats the baselines under the project's constraints. Every algorithm is an assumption about how the world works, so pick the one whose assumption matches the data and the decision.

## General method (any problem type)

1. Start with the baseline ladder from phase 5. Then try **one simple, explainable model**, then **one robust strong model**. Add complexity only if the gain pays for its cost.
2. Use the same split, features pipeline and metric for every candidate.
3. Tune only a few influential settings with a clear reason. Prefer random search or successive halving over exhaustive grids. Use early stopping. Remember that more search makes the validation score optimistic.
4. Track build time, model size, prediction latency and maintenance burden next to accuracy.
5. Use AutoML **to benchmark quickly, not to decide for you**. It optimises the framing you give it and never questions it.
6. Effort order when stuck: **better framing > better data and labels > better features > better model or tuning.**

## Model-selection questions (ask before choosing)

1. Who must understand the output: an auditor, a regulator, an operator, or nobody?
2. How much labelled data is there (hundreds or millions of rows)? How many per class or segment?
3. What feature types: tidy numbers, free text, images, sequences, or mixed types with holes?
4. What latency, memory and infrastructure budget: milliseconds online, or an overnight batch?
5. How fast does the world change, and how often must you retrain?

## Regression

- **Use it when** the target is a number. Avoid it for bounded targets, binary targets or censored time-to-event targets.
- **Ladder**: mean → segment mean → business rule → linear regression with good features → regularised linear (Ridge shrinks; Lasso shrinks and selects; ElasticNet blends them; standardise first) → tree ensembles (random forest, gradient boosting) for nonlinear patterns and interactions.
- **Read coefficients** as the change in target per unit of X, *holding the others constant*. Interpret coefficient × a realistic change in X. Log target → % effects. Log–log → elasticity.
- **Multicollinearity** (unstable, huge or sign-flipping coefficients) hurts interpretation, not usually prediction. Detect it with a correlation matrix, coefficient instability or VIF (above 5 is a warning, above 10 is serious).
- **Statistical output** for explanation work: standard errors, p-values and confidence intervals of coefficients.
- **Uncertainty**: a confidence interval (for the mean response) vs a prediction interval (for one new case, wider). Use **quantile regression** for downside, median or upside scenarios (Q10 / Q50 / Q90). Check interval coverage.
- **Heteroscedasticity** (errors grow with the prediction, a "funnel" shape): consider a log target, segmented models or weighted least squares, and use quantile intervals instead of symmetric ones.
- **Extrapolation**: models interpolate confidently and extrapolate recklessly. State the valid operating range.

## Classification

- **Classifier "personalities"** (pick by the pattern you expect and by your constraints):
  - Logistic regression: weighted evidence or scorecard; additive, smooth boundary, probabilities, highly explainable. The regulatory default and baseline.
  - Decision tree: if-then flowchart; captures interactions; explainable only when shallow (depth 3–4); overfits when deep.
  - k-NN: "similar past cases"; explanation by example; needs scaling; slow at prediction time; suffers in high dimensions.
  - Naive Bayes: independent clues multiplied; great for text, tiny data and streaming; probabilities are overconfident.
  - SVM: widest margin, with kernels for curves; good when there are many features and few rows; weak on probabilities, scale and explainability.
  - **Ensembles** (see below): random forest as the robust tabular default; gradient boosting (XGBoost, LightGBM, CatBoost, histogram boosting) when accuracy matters on tabular data.
  - Deep learning or LLMs: unstructured text, images or audio at scale; high latency and cost.
- **Requirement → choice** rules of thumb:
  - Printable reason for every decision → logistic regression or a shallow tree (optionally boosting with per-case explanations, if allowed).
  - A rule operators will follow → shallow tree.
  - "Show me similar cases" → k-NN.
  - High-volume text routing → Naive Bayes or linear models on text features.
  - Many features, few rows → SVM or regularised logistic regression.
- **Imbalance**:
  - Train normally, then **set the threshold from costs or capacity** (phase 8).
  - Alternatively, use class weights that mirror the cost ratio; "balanced" weights fix frequency, not cost. Re-select the threshold afterwards.
  - Resampling (under- or over-sampling) only on training data.
- **Multi-class**: native multinomial models, or one-vs-rest / one-vs-one. **Multi-label**: independent per-label models or classifier chains when the labels are correlated. **Hierarchical labels**: fall back to the parent class when the child is uncertain.
- **Encoding and scaling** as in phase 3. k-NN, SVM and logistic regression need scaling; trees don't.

## Ensembles (any supervised task)

- Ensembles work only when members are **accurate (better than chance) and diverse (uncorrelated errors)**. Copies of the same model add nothing. If all models miss the same cases, fix the data or representation instead.
- **Bagging / random forest**: many deep trees on bootstrap rows and random feature subsets. Reduces variance, hard to overfit by adding trees, out-of-bag score as a free check, low tuning.
- **Boosting**: sequential small trees correcting earlier errors. Reduces bias and captures interactions. Use a smallish learning rate with early stopping and small-to-medium trees; it can overfit noise or bad labels.
- **Voting**: average the members' probabilities (soft voting). Good when members have similar quality.
- **Stacking**: a meta-model learns whom to trust, from out-of-fold predictions. Use it only if the gain is large and the operational overhead is acceptable.
- Rule of thumb: simple baseline → random forest → boosting → stacking only if it pays; AutoML to benchmark.

## Recommenders (ranking)

- **Frame first**: who is the *user* (person, household, account, session, device)? What is the *item* (product, category, content, offer, action)? Which *signal* counts as interest (explicit ratings are scarce and biased; implicit signals are abundant and ambiguous; weight purchase > cart > click > view)? Which action does the ranking trigger?
- **Absence of a signal is not dislike.** The user may never have seen the item.
- **Baseline ladder**: non-personalised (popular, trending, top-rated, shown with counts, not just averages) → semi-personalised (segment, context, "bought together") → fully personalised. Context often captures most of the value at a fraction of the cost.
- **Approaches**:
  - Content-based (item attributes or text similarity to the user profile).
  - Collaborative filtering: user-based ("taste twins") or item-based ("people who chose this also chose"; stable, precomputable, explainable, usually preferred).
  - Matrix factorisation (latent taste dimensions).
  - Hybrids.
- **Failure modes**:
  - Cold start: mitigate with onboarding questions, popularity fallback, content features, hybrid weighting and exploration.
  - Popularity bias and feedback loops: the system learns from its own past exposure.
  - Filter bubbles, fairness to items and groups, and privacy (use minimal data, give users control, reserve exploration slots).
- **Evaluate** offline (Precision@K, Recall@K, MRR, coverage), then online (A/B with guardrails, incremental impact).

## Pattern mining (association and sequences)

- **Define the basket and the item first.** The basket boundary (transaction, session, user-month, time window) changes every answer. Too narrow → patterns are rare; too broad → meaningless. User-level baskets are dense and flatten lift.
- **Rules A → B** are not symmetric. Judge them with:
  - **Support**: is it common enough to act on?
  - **Confidence**: when A happens, how often B also happens; compare it to B's base rate.
  - **Lift**: above 1 means a real positive link; about 1 means nothing beyond popularity.
- Use efficient algorithms (FP-Growth style) with minimum support and lift. Apriori is fine for teaching but slow in production.
- **Traps**:
  - Popularity trap: high confidence but lift ≈ 1.
  - Rare-but-real rules: low support, high lift; useful only if the volume justifies action.
  - Spurious rules that vanish on new data.
- Keep rules that are **surprising, stable** (hold across months, regions and segments) and **economically useful** (value > cost of acting).
- **Sequential patterns** when order matters: journeys, failure precursors, churn pathways, fraud sequences. Define the entity, the ordering, any max-gap or timing window and the support of a sequence. A → B ≠ B → A.
- Turn each retained pattern into an action. Example: target users who have A but lack B; expected uptake ≈ audience size × confidence. Validate it over time by mining the past and re-checking later.

## Forecasting

- **Frame**: entity, variable, horizon and frequency. Changing any one of them changes the problem. Define the action the forecast drives. If no action changes, it is a report.
- **Workflow**: patterns → baseline → model → backtest → uncertainty → decision.
- **Patterns**: decompose into trend, seasonality (often several: daily, weekly, yearly), cycles and irregular events. Tag known one-off events (promotions, holidays, outages, weather) so they don't distort the baseline. Watch for the zoom illusion, where the visible pattern depends on the time window.
- **Baselines**: naive, seasonal naive, moving average (short windows react quickly but are noisy; long windows are stable but slow).
- **Model menu**: exponential smoothing (alpha = memory), Holt-Winters (level + trend + season; needs two or more seasonal cycles), ARIMA, Prophet-style (holidays and events), gradient boosting with lag and calendar features (many series, nonlinear patterns), deep learning (massive cross-series data).
- **Heuristics by data and pattern**:
  - Under 2 years of history → naive, moving average or smoothing.
  - 3 or more years → seasonal and ARIMA-type models.
  - 5 or more years or many series → machine learning becomes viable.
  - No pattern → naive. Trend only → trend or double smoothing. Seasonality → seasonal naive or decomposition. Trend + seasonality → Holt-Winters or Prophet-style.
- **Evaluate** with rolling-origin backtests by horizon (MAE, MAPE, bias). Analyse where and when it fails; wavy residuals mean seasonality was missed. Error grows with the horizon (the horizon cliff).
- **Uncertainty**: prediction intervals (for example, from empirical residual quantiles) as a fan chart, with coverage checked against the claimed level.
- **When forecasting fails**: no history, regime shifts, adversarial settings, one-off events. Use scenarios and judgment instead.

## Anomaly detection

- **Workflow**: define normal → score unusualness → threshold → investigate → feedback. It is a loop; most of the impact comes from defining normal and setting thresholds, not from the method.
- **Define normal: compared to what?** The whole population, a peer group, the entity's own history, or a forecast (residual). The same event can be normal under one lens and odd under another.
- **Types**: point anomalies, contextual anomalies (odd for this time or context) and collective anomalies (each item fine, the group odd).
- **Method ladder** (climb only as needed; combine, don't replace):
  1. Business rules: transparent, fast, regulator-friendly, but rigid and gameable.
  2. Statistical deviation: z-score (use the robust median/MAD version) and IQR fences. These break with skew, heavy tails and shifting baselines.
  3. Neighbourhood and density: k-NN distance, Local Outlier Factor. Need a sensible distance measure; expensive.
  4. Isolation Forest: the scalable, unsupervised default.
  5. Supervised classification when labelled history exists: accurate on known patterns but misses new ones; handle the imbalance.
  - Also: low-variance detectors for stuck sensors, and forecast residuals for time series.
- **An alert means "unusual", not "bad".** A human or process must decide whether it is an outlier, an error, fraud or a new normal, and each calls for a different response.
- **Layered system**: rules → models → human review. Ensembles can combine detectors (any, all, majority vote, weighted rank); the remaining residual risk is the cases every layer misses. Set thresholds from the alert budget and the miss vs false-alarm costs.

## Clustering and segmentation

- **Use it to** discover natural groups without labels (customer segments, product families, behaviour archetypes). Frame it by the decision the segments will drive.
- **Steps**: choose behaviour-relevant features → scale them → choose a distance or similarity → choose a method (centroid-based for compact round groups; hierarchical for nested structure; density-based for arbitrary shapes and noise; mixture models for soft membership) → choose the number of clusters (elbow, silhouette, stability **and** business usefulness) → profile each cluster (size, defining traits, value) → name it → validate its stability across samples and time.
- A segmentation is good only if the segments differ in ways that change actions and stay stable enough to operationalise.

## Causal questions (what happens if we intervene?)

- Predictive models are not causal. For "should we do X?", prefer a **randomised experiment (A/B test)** with a pre-registered metric, guardrails, enough sample size and enough duration.
- Without an experiment, use quasi-experimental designs (before/after with a control group, matched comparisons). State the confounders and the assumptions explicitly.

## Deliverable

A model comparison table: each baseline and candidate with its primary business metric, technical metrics, uncertainty range, training and prediction cost, size, explainability and notes. Name the recommended candidate and the reason.
