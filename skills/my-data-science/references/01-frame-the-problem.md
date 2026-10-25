# Phase 1 — Frame the decision

Goal: turn a vague request ("analyze churn", "predict sales", "plot revenue by region") into a precise, decision-linked problem with known costs. Most failed projects fail here.

## Steps

1. **Find the decision and its owner.** Ask what action will change because of this work, who takes it, and how often.
2. **Write the reasoning chain.** Analytical question → hypotheses → evidence needed → view or model → decision.
3. **Classify the job**: *prediction* (what will happen; accuracy matters most), *explanation* (which factors move with the outcome; interpretability matters most) or *causality* (what happens if we intervene; study design matters most). One model can do only one job well. Most business disasters come from reading a predictive model as a causal one.
4. **Define the unit, target, horizon and decision time** with the template:
   **"For [unit], using information available at [time], predict [target] over [horizon], so that [owner] can [action]."**
5. **Define the target precisely.** A good target is observable, consistently recorded, close to the real business event, hard to manipulate by accident and **early enough to act on**. Compare candidate definitions (for example, for churn: non-renewal, cancellation request, downgrade, silent inactivity, unprofitable retention). Each one implies a different action and timing.
6. **Choose the horizon from lead time, not from ease of prediction.** A shorter horizon is often easier to predict but too late to act. Ask how many days of warning the intervention needs.
7. **Price the mistakes.** Build a **cost matrix** with the cost of each outcome (TP, FP, FN, TN for classification; over- and under-prediction for regression and forecasting). Include non-financial costs such as customer friction, regulation, reputation, staff morale and adversaries learning the system.
8. **Capture constraints.** These include capacity (reviews per day, offers per month), latency (real time vs overnight batch), explainability (regulator, auditor, frontline staff), infrastructure, retraining cadence, data privacy, legal limits and fairness requirements.
9. **Describe the status quo.** Write down today's process (rule, heuristic, expert estimate) and its measured performance. This becomes the baseline to beat.
10. **Route to a problem type** (see the router in SKILL.md) and check that the problem qualifies. Classification qualifies only if (a) the answer comes from a fixed known set of labels, (b) past labels exist at scale and (c) each label leads to a different action.
11. **Define success.** State a primary business metric (cost saved, value, workload), a technical metric that tracks it, **guardrail metrics** (complaints, fairness, latency) and the minimum improvement over baseline that justifies deployment.
12. **Design the analytical system when more than one technique is needed.** Name each component, what it predicts, and how its output feeds the next component or the decision. Draw data → components → decision. Then define the **decision layer**: the actions available, the real constraint or objective (budget, capacity, risk appetite), and the rule that turns predictions into actions. "A prediction with a threshold slapped on" is not a decision layer unless the threshold comes from a real constraint or cost.
13. **Start an assumptions register.** List every quantity the decision needs but the data doesn't contain (intervention cost, value of a retained customer, reviewer accuracy, capacity), with its value, source or rationale and owner. These become configurable parameters and the inputs to sensitivity analysis.
14. **Scope to resources.** Match the ambition to the team size, the time and the data access. Note the planned ablation, sensitivity analysis and deeper analyses early (see [deeper-analyses.md](deeper-analyses.md)).
15. **Check whether data science is the right tool.** If no decision changes, the output is a report, not a project. Don't build high-precision models for low-value decisions.

## Questions to ask the stakeholder

- What decision will this inform? What would you do differently if the answer were X vs Y?
- Who owns the decision, and who owns the cost numbers and the threshold?
- What does one row or case represent? When exactly is the decision made?
- What counts as the event or outcome, and how is it recorded today? Who records it?
- How much warning or lead time does the action need?
- What does each type of mistake cost? Which is worse, and by how much?
- How many cases can the team act on per day, week or month?
- How is this decided today, and how well does that work?
- Who must understand or audit the output (customer, regulator, executive, operator)?
- What latency, budget and retraining cadence are acceptable?
- Are there legal, ethical or fairness constraints on which data may be used?
- What result would make you *not* act? What would be uncomfortable to present to the CEO?

## Decision rules

- No decision change → stop or reframe as reporting.
- Asymmetric costs → the threshold or quantile must come from costs, never from a default such as 0.5 or the mean (see phase 8).
- If two labels trigger the same action, merge them.
- Bounded target (0–100%, 1–5 stars) → don't use plain linear regression. Use classification or ordinal methods, or transform the target.
- Time-until-event with incomplete follow-up → survival methods.
- No history, a regime shift or an adversarial setting → prefer scenarios, simulation or expert judgment, and be explicit about uncertainty.
- Very rare event with few labels → consider anomaly detection instead of classification.

## Deliverable: problem brief

Decision and owner · problem statement (template above) · problem type(s) and system diagram · decision layer (actions, constraint, rule) · target definition and alternatives considered · horizon and lead time · cost matrix · constraints (data, computational, business/regulatory) · assumptions register · status quo and its performance · success criteria: primary ML metric, primary decision metric, "good enough" bar to act and how it beats the baseline · guardrails · open questions.

Write it for someone outside data science: short, decision first. For a formal proposal, see the template in [project-delivery.md](project-delivery.md).
