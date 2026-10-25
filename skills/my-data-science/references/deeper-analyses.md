# Deeper analyses: prove the system earns its complexity

Use these after a working end-to-end system exists (phases 7–8). Pick the ones that fit **this** problem, not every one. For each, write one or two sentences on why it matters here. A strong project combines a credible baseline, an **ablation**, a **sensitivity/stability analysis** and at least two or three of the deeper ideas below.

## Always do

### Credible baseline
- The baseline must be a plausible simpler alternative that someone would genuinely use, such as the current business rule, a simple ranking or a simple model. A strawman built to lose doesn't count.
- Every component needs a baseline, and so does the **decision layer**. Example of a baseline policy: "act on the highest-risk cases until the budget runs out".

### Ablation analysis
**Question**: how much value does each part of the system actually contribute?
- Remove or swap one component at a time and re-score the **decision-level metric** on the same held-out data. Examples: drop the second model, replace a personalised recommendation with the most popular item, rank by risk only instead of risk × value, remove a feature group.
- A component that adds complexity but no decision-level value should be cut.

### Sensitivity and stability analysis
**Question**: do the main conclusions survive reasonable changes?
- Vary the cost assumptions, thresholds, budgets or capacity, prevalence, time windows and random seeds over plausible ranges. Report whether the recommended model or policy still wins.
- Name the assumptions the decision is most sensitive to. Those are the ones to measure better.

## Deeper ideas (choose those that belong to the problem)

| Analysis | Question it answers | How |
|---|---|---|
| **Uncertainty-aware decisioning** | Which cases should be automated, and which should go to a human? | Use confidence, interval width or model disagreement to route uncertain but important cases to review. Compare cost and automation rate with and without routing. |
| **Oracle / headroom analysis** | How much would a perfect component improve the decision? Where should the next effort go? | Replace one component's output with the true outcome and re-score the decision metric. The gap is the maximum value left in that component. The bottleneck may be something else, such as the intervention's effectiveness or the budget. |
| **Decision-regret analysis** | How much value do the mistakes actually lose? | Score each error by the money lost instead of counting errors equally. A wrong call on a high-value case should weigh more. Report regret by segment and case size. |
| **Decision frontier** | How does the best policy change as resources or constraints change? | Re-optimise the policy across a range of budgets, capacities or risk appetites. Plot the value achieved and the optimal policy against the resource level. |
| **Cross-model reconciliation** | What happens when components disagree, and which rule resolves it? | Find the cases where the components conflict (for example, low predicted risk but highly anomalous recent behaviour). Define a principled rule such as priority, escalation, a combined score or human review, and measure its effect. |
| **Challenge-set evaluation** | Does the system hold up on the hard cases that averages hide? | Build separate test slices for cold-start entities, rare patterns, extreme values, volatile periods and new segments. Report the metrics on each slice. |
| **Value-of-information analysis** | Which single new data source would improve the decision most? | Estimate the decision-level gain from adding a candidate source, using a proxy, a simulation or a small pilot. Rank the sources by expected value minus acquisition cost. |
| **Segment-specific decisioning** | Do different populations need different models or policies? | Compare one global policy with per-segment ones. Specialise only where the segments differ a lot and each has enough data. |
| **Multi-objective analysis** | What trade-off must the organisation balance? | Make the tension explicit (for example fraud reduction vs customer friction, revenue vs churn, availability vs carrying cost). Show the trade-off curve and let the stakeholders pick the operating point. |

## Reporting

For each analysis report:
- the question
- the design: what was removed, varied or replaced, and on which data
- a results table using the decision-level metric
- the conclusion and what it changes (cut a component, invest in data, change the policy, add reviewers)
