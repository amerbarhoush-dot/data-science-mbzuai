# Phase 9 — Communicate the results and build the stakeholder dashboard

Goal: give each audience exactly what it needs to act, honestly. Your job is shifting from *making* charts to *judging* them. A chart bypasses careful thinking and goes straight to the gut, so every choice is a decision.

## Steps

1. **Identify audiences and their decisions.**

   | Audience | Wants | Typical content |
   |---|---|---|
   | Analyst | Full detail, ability to drill in | Distributions, scatter plots, raw tables |
   | Technical peer or manager | Enough to verify the logic | Charts plus key statistics, moderate annotation |
   | Executive | One message, fast | One chart, one number, one annotation, no jargon |

   The same finding should look different for each audience.
2. **Write the message first.** Each executive view must state:
   - what changed
   - compared with what (baseline or target)
   - whether it matters (size vs noise)
   - what action it suggests
   - the most important uncertainty or caveat
3. **Translate into business language.**
   - Weak: "coefficient = 38.2". Better: "each integration adds about a week". Best: "our 6-integration quotes are systematically 200 hours light, about X per deal".
   - End at the money or the decision.
   - Choose absolute vs percentage figures deliberately; small percentages on big bases and vice versa can mislead.
4. **Avoid false precision.** Round, report ranges ("about 1,250, usually within 1,050–1,450"), and say where the model is unreliable and must not be used.
5. **Choose each chart by the analytical task.**

   | Task | Use | Avoid |
   |---|---|---|
   | Comparison | Bar chart (axis starts at zero) | 3D bars |
   | Ranking | Sorted bar or dot plot | Unsorted bars |
   | Distribution | Histogram, box or violin plot, ECDF | A lone average |
   | Relationship | Scatter (with colour for groups) | Dual-axis lines |
   | Trend | Line chart | A pie chart "over time" |
   | Composition | Stacked bar, small multiples | A pie with 6+ slices |
   | Before/after | Slope chart | Grouped bars |
   | Two categories × measure | Heatmap | Cluttered grouped bars |
   | Anomaly | Annotated line or scatter | Unannotated charts |
   | Uncertainty | Error bars, bands, fan charts | An unlabelled point estimate |
   | Model trade-off | Threshold vs cost/precision/recall curve, coverage vs accuracy curve | A single headline accuracy |

6. **Encode by perceptual accuracy.**
   - Ranking of channels, best first: position on a common scale > length > angle or slope > area > colour intensity > volume.
   - Use hue for categories, luminance for magnitude and saturation for emphasis.
   - **Visual hierarchy**: decide the one thing the audience must notice first, make it pre-attentive (colour, size, position), and keep everything else muted. If everything is salient, nothing is.
7. **Use colour correctly.**
   - Categorical palettes for unordered groups (about 6 colours at most). Sequential palettes for ordered magnitudes. Diverging palettes when there is a meaningful midpoint such as zero, a target or an average.
   - Use perceptually uniform, colourblind-safe palettes. Avoid rainbow/jet.
   - Get the chart right in black and white first; use colour as a redundant channel.
   - Use semantically resonant colours (for example, red = bad only where that is culturally clear).
8. **Label and format.**
   - Make charts self-documenting: the title states the takeaway, a subtitle gives context, axes carry units, there is a legend or direct labels, and the data source and date are shown.
   - Use readable number formats (thousand separators, no scientific notation, consistent currency).
   - Remove chart junk: heavy gridlines, borders, 3D effects, needless top/right spines. Maximise the data-ink ratio.
   - Start bar axes at zero; truncate only when zero is meaningless or it is the domain norm.
   - Avoid dual y-axes; use separate aligned panels or a common metric.
   - Use small multiples with shared scales instead of overloaded charts.
   - Choose a sensible aspect ratio (bank slopes towards 45°) and tick density.
   - Handle overplotting with transparency, sampling or density plots. Show sample sizes next to rates and averages.
9. **Audit every chart before it ships**, especially generated ones, at four levels:
   1. **Code**: does it plot the intended fields, filters and time window?
   2. **Statistics**: are the aggregation, denominators, binning and smoothing valid?
   3. **Perception**: does the design distort comparison or attention (scale, colour, dual axes, truncation)?
   4. **Decision**: does it answer the real question for this audience?
   Common failures: wrong aggregation, wrong denominator, wrong scale, bad binning, over-smoothing that hides the incident, misleading colour, cherry-picked windows, aggregation hiding subgroups, estimates shown as observations, and dual axes that invent correlations.

## Building the stakeholder dashboard

**Purpose first.** Name the decision or decisions it supports, the users, how often they look at it, and the actions they can take from it.

**Recommended layout, from top to bottom:**
1. **Headline KPIs** tied to the decision: business outcome (cost saved, value, prevented losses), workload or capacity use, and the primary model or forecast quality. Each KPI is shown against a baseline or target and the previous period, with a trend.
2. **Recommended action and status**: the current policy (threshold, top-K, tiers), what to do this period, and the expected impact with a range.
3. **Trade-off panel**: how cost, precision, recall and workload change with the threshold or capacity, ideally with a what-if control for costs, capacity and scenarios.
4. **Segment breakdown**: performance and impact by critical segment, with sample sizes; flag segments below standard.
5. **Uncertainty and data health**: intervals or fan charts, calibration or coverage, data freshness, missing-data and drift indicators, and the date of the last model update.
6. **Drill-down**: a case list or table for operators (top-ranked cases, reasons or explanations, status), with filters.
7. **Definitions and caveats**: metric definitions, data sources, known limitations and the safe operating range.

**Design rules**:
- Prefer at most one screen of headline content.
- Keep chart types and colours consistent across views.
- Put the most important view at top-left.
- Every number has a comparison; every chart has a takeaway title.
- Never show a metric without its denominator.
- Accessibility matters (contrast, colourblind safety, readable fonts).

**Technology decision rule** (the agent picks one, states why, and asks if unsure):
- The organisation already has a BI platform the audience uses (Power BI, Tableau, Looker, etc.) → design for it and deliver the data model, metric definitions and layout spec.
- The audience needs interactivity (filters, what-if sliders) and someone can host a small Python app → a lightweight interactive web app (for example Streamlit or Dash).
- The report must be shareable, offline or emailed with no server → a self-contained static HTML report with interactive charts, or a PDF for print.
- A one-off executive readout → a short slide deck or one-page summary with one key chart per message.
- Real-time operational monitoring → the organisation's monitoring or BI stack, with alerts.

**Questions to ask before building**: Who opens it, and how often? Which decision do they take from it? Which three numbers must they see in five seconds? What filters do they need? Where must it live, and what tools do they already use? How fresh must the data be? Who maintains it?

## Executive summary (one page)

Problem and decision → recommendation → expected impact with a range vs the status quo → how it was validated (baseline, out-of-time test, segments) → key risks and caveats → what is needed (approvals, cost owners, pilot) → next steps.
