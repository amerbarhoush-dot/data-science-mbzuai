# Phase 4 — Exploratory data analysis (EDA)

Goal: investigate what the data can and cannot say. EDA has three jobs: **quality check** (is the data trustworthy?), **hypothesis generation** (which patterns are worth pursuing?) and **decision support** (does this number support the call we're about to make?). Charts are evidence-gathering tools, not the objective.

## Mental model (work through it in order)

Data quality → **univariate** → **bivariate** → **conditional** (does it survive within subgroups?) → **multivariate** → decision.

## Steps

1. **Start from hypotheses.** For the question, list two to four competing explanations and the evidence that would support, weaken or complicate each one. Commit to "what would change my mind" first.
2. **Univariate** (each variable alone):
   - What values exist (range, categories, missingness)?
   - Where is the mass (histogram, frequency table)?
   - How asymmetric is it (mean vs median, P50/P90/P95/P99, P99/P50)?
   - What is unusual (tails, spikes, gaps, truncation, sentinels, rounding)?
   - What business mechanism explains the shape?
   - Always analyse **spread**, not just averages: "average deal size is up" can come from one huge deal, and high spread often signals an operational problem.
3. **Bivariate** (how one variable behaves as another changes). Chart before summary statistic.
   - Categorical × categorical: cross-tabs with **row percentages** (rates, not raw counts), stacked bars or heatmaps. Look for disproportionate representation.
   - Categorical × numeric: compare medians, IQR and tails (box or violin plots) and **sample sizes**. A gap based on 20 cases deserves far less trust than one based on 20,000.
   - Numeric × numeric: scatter plot first. Correlation detects only straight lines; U-shapes, clusters and a few extreme points can dominate it. Handle overplotting with transparency, sampling, hexbin or density plots.
   - Time: line charts with resampling to the right frequency and rolling averages to separate signal from noise; check seasonality.
4. **Conditional reasoning.** Re-check every important pattern **within relevant segments** (risk band, product, region, tenure, channel).
   - **Simpson's paradox**: an overall relationship can reverse inside every subgroup.
   - **Confounders**: a third variable that drives both, such as tech-savviness driving both app usage and spend.
   - Compare like with like ("controlling for"): unadjusted vs adjusted comparisons.
5. **Multivariate.** Look for interactions (an effect only appears when two conditions meet), segments and hidden structure using facets or small multiples, pivot heatmaps, pair plots and stratified tables.
6. **Separate signal from coincidence.** For each pattern ask:
   - Is it large enough to matter?
   - Does it persist across time?
   - Does it survive segmentation?
   - Could a confounder or seasonality explain it?
   - Is there a plausible mechanism, or is it a spurious correlation?
7. **Test alternative explanations actively.** Examples: competitor exit, seasonality, discounting, a measurement change, a one-off event.
8. **Cover every component and the decision.** EDA should check that the data supports each planned technique, for example:
   - target or class balance for classification
   - time patterns, seasonality and rolling averages for forecasting
   - interaction sparsity and user/item activity for recommenders
   - box plots and flagged points for anomalies
   - co-occurrence frequencies for pattern mining
   It should also include **decision-relevant views**: distributions of value and cost, outcome rates by segment, and how much the constraint binds. Note any surprises or scope changes the EDA triggers.
9. **Prioritise** findings by **impact × feasibility × confidence**. High-impact, low-confidence findings become research questions. Drop findings that are statistically interesting but have tiny business impact.
10. **Convert observations to insights.** An insight = pattern + location + mechanism + consequence. Apply the **"So what?" test** to each.
   - Observation: "churn is 18%".
   - Insight: "churn is concentrated in the first 90 days after onboarding; fixing onboarding protects X per month".
   - The funnel to expect: 100 charts → 20 patterns → 5 findings → 2 insights → 1 decision.

## Questions to ask yourself and the stakeholder

- What decision could this influence? What would make the result misleading?
- What additional data should be collected to settle the open questions?
- Which segments matter most to the business, for example a premium segment that is 20% of customers but 55% of profit?
- What would I be uncomfortable presenting to an executive?

## Common traps

- Counts without denominators.
- Cherry-picked time windows.
- Aggregation that hides subgroups.
- Smoothing that erases the incident.
- Treating estimated values as observed.
- Mean of a bimodal distribution.
- Dual axes implying relationships.
- Correlation read as causation.
- Accepting the first hypothesis.
- Asking a tool to "find something interesting" without context, constraints and an explicit demand for skepticism.

## Deliverable: EDA findings memo

For each finding: the claim, the evidence (chart or table with n), whether it survives segmentation and time, alternative explanations checked, confidence, business impact and the suggested action or next question. Ranked by impact × feasibility × confidence.
