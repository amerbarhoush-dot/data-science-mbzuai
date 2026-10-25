# Phase 2 — Audit the data (can we trust it?)

Goal: establish what the data can and cannot tell you before anyone looks for patterns. The most expensive mistakes come from skipping this phase.

## Steps

1. **Inventory.** List sources, row counts, time coverage, owners, refresh cadence and join keys. Note how each table was produced and filtered.
2. **Grain (unit of analysis).** What does one row represent: a customer, transaction, day or branch-month? Never mix grains silently. Customer-level data overweights large branches, and transaction-level data overweights frequent customers.
3. **Types and semantics.** Classify each field as nominal, ordinal, interval or ratio, and as discrete, continuous, temporal or spatial. The type decides valid operations and charts. Example: a satisfaction rating is ordinal, so its gaps aren't equal.
4. **Duplicates.** Decide which fields define a genuinely unique event. Exact duplicates suggest ingestion errors. Apparent duplicates may be real repeated events.
5. **Definitions.** Confirm business definitions such as "active customer", "revenue" (booked, collected or net) and "resolved". Check that they are consistent across sources and time.
6. **Missing data and why.** Missing at random (a glitch) is relatively safe. Missing *because of the value itself* (high earners skip the income question) is dangerous. Also ask **who is missing from the data**: applicants vs approved, app users vs all customers, survivors vs everyone.
7. **Exclusions and selection.** What was filtered out upstream? Is the sample representative of the deployment population?
8. **Censoring and incomplete outcomes.** "No bad outcome yet" doesn't mean "good outcome". Recent customers and loans have had less time to churn or default. Align observation windows.
9. **Process changes and drift.** Look for policy, system, pricing, staffing and definition changes, and for seasonal or calendar effects such as holidays and school cycles. A KPI dip can mean the business changed or that someone redefined the KPI.
10. **Value sanity.** Look for ranges, impossible values, sentinel codes (−1, 999, default dates), rounding spikes, unit mismatches and inconsistent currencies or time zones.
11. **Label quality** (supervised problems):
    - Audit a sample of about 100 labels. Two qualified reviewers relabel independently, then compare. Separate ambiguous policy, annotation mistakes and obsolete categories.
    - Expect 20–30% noisy labels in operational systems. Human agreement caps achievable accuracy.
    - **Delayed labels**: if confirmation takes 90 days, recent cases look clean. Exclude immature periods.
    - **Proxy labels**: ask "what did we actually label?", because the measurable label may differ from the outcome you care about.
12. **Leakage audit** (critical). For every candidate feature ask: *Would I know this, with this exact value, at the moment of decision?*
    - **Target leakage**: fields created after the outcome (case status, recovery amount, days until paid, investigation notes, cancellation requests).
    - **Proxy leakage**: fields that encode who handled the outcome (assigned investigator, routed to legal, extra documents requested).
    - **Time-travel leakage**: reference data rebuilt after the fact (today's watchlist instead of the watchlist as of the event date).
    - **Pipeline leakage**: scaling, imputing, encoding or feature selection fitted on all rows before splitting.
13. **Fitness for purpose.** Is there enough data overall, per segment and per class? Does history contain the signal, or only static descriptors?

## Questions to ask

- What exactly is one row? Which fields uniquely identify an event?
- How are the key business terms defined in each source? Did the definitions change?
- What was excluded upstream, and why?
- Why is this field missing? Is missingness related to the outcome?
- When does each field become known relative to the decision time?
- How and when are labels created, by whom, and how reliable are they?
- What changed in the business or systems during this period?

## Decision rules

- Do not impute missing values with zero by default. Flag them, understand the mechanism, and consider a "was missing" indicator.
- Never delete outliers just because they are inconvenient. They may be errors, rare legitimate events, or the most important cases (fraud, safety).
- Any field that is not available at decision time is removed, whatever it does to the score.
- If labels are unreliable, fix definitions and build a small trusted gold set before modeling at scale.

## Deliverable: data audit report

Sources and grain · definitions · data dictionary with types · missingness table and suspected mechanisms · duplicates and exclusions · censoring windows · known process changes · label quality findings · leakage audit table (feature → available at decision time? yes/no/depends → action) · risks and open questions.
