# Phase 3 — Prepare the data and engineer features

Goal: a reproducible pipeline that turns raw data into model-ready features without leaking information and while preserving meaning. Domain knowledge beats algorithmic cleverness here. A good feature can make a simple model strong.

## Steps

1. **Build an analysis table at the decision grain.** One row per unit at decision time (for example, customer at month-end). Features look **backward only** and targets look **forward only**.
2. **Clean** with documented rules: fix types, units, currencies and time zones; standardise categories; resolve duplicates; handle sentinel values; cap or flag impossible values. Log every rule.
3. **Handle missing values on purpose**: drop the column, impute (median or mode, learned on training data), add a missing-indicator, or model the missingness. Choose based on the mechanism found in phase 2.
4. **Handle outliers by cause**: correct data errors; keep legitimate extremes (consider transforms or robust metrics); treat known events (promotions, outages, holidays) as explicit features or flags.
5. **Encode categoricals**: one-hot for small cardinality (about 5–20 levels); grouping, frequency, target or native-categorical methods for high cardinality. Unseen categories must be handled safely.
6. **Transform skewed variables.** Log right-skewed money, durations and counts. A log target gives roughly percentage effects; log on both sides gives an elasticity. Square-root is a milder transform. Use polynomial terms for curvature.
7. **Engineer features from domain knowledge**:
   - **Ratios and normalised rates** (per customer, per seat, per employee, share of total) instead of raw counts.
   - **Counts and flags** (number of stakeholders, is_new_client, rule hits).
   - **Time features**: day of week, month, holiday proximity, recency, frequency, tenure, rolling windows (last 7, 30 or 90 days), trends and slopes, change vs the entity's own baseline, lags at the same seasonal position.
   - **Interactions** where an effect depends on context (discount × new customer, distance × peak hour).
   - **Rule outputs as features**: deterministic business rules can become inputs.
   - **Text, image or sequence representations** where relevant (word counts, TF-IDF, embeddings).
8. **Scale when the model needs it.** Distance- or penalty-based methods (k-NN, SVM, regularised linear models, clustering, neural nets) need scaling. Trees and Naive Bayes mostly don't.
9. **Put every learned step in one pipeline** fitted on training rows only, then applied unchanged to validation, test and production.
10. **Record feature lineage**: source, logic, availability time and owner. This is the basis of the leakage audit and of production parity.

## Questions to ask

- Which signals do experts already use to judge a case? Can each be computed at decision time?
- Which behaviours change *before* the outcome happens, such as declines, spikes or new contacts?
- What is the natural denominator for each count?
- Which known events or calendar effects should be represented explicitly?
- Will the same feature be computed identically in production, and how fresh will it be?

## Decision rules

- Prefer behavioural and recent-change features over static descriptors when the outcome is behavioural.
- Prefer rates over counts whenever group sizes differ.
- If a feature looks too good, re-run the leakage audit before celebrating.
- Keep features that are stable and explainable over fragile ones with a tiny gain.
- Add complexity (interactions, polynomials, embeddings) only when residuals or error analysis show the need.

## Deliverable

A reproducible preparation pipeline, a feature catalogue (name, definition, source, availability time, transform), cleaning and imputation rules, and the analysis table with its grain and time window documented.
