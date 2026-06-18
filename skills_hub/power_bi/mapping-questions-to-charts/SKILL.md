---
name: mapping-questions-to-charts
description: Translates a business stakeholder's question into the right Power BI visual by classifying the question's intent (comparison, trend, part-to-whole, distribution, process/stage, geographic, KPI/goal, or root-cause) and mapping that intent to a specific Power BI chart type with a worked DAX example. Use when someone asks a business question ("what are the top products?", "how have sales trended?", "why did churn rise?", "where are our customers?") and you must choose a chart, or when picking, recommending, or justifying a Power BI visualization for a report or dashboard. Covers keyword extraction from questions and multi-chart composition for compound questions.
---

# Mapping Questions to Charts

Pick the right Power BI visual by classifying what the question is *asking for*, not by guessing. Workflow: (1) extract intent keywords, (2) classify into one of 8 intents, (3) map to a Power BI visual, (4) write the aggregating DAX measure, (5) validate against data shape.

This is a Layer 1 foundation skill. It pairs with `visualizing-dimensional-models` (fact-type and grain affinity) and `understanding-fact-additivity` (correct SUM/AVERAGE/ratio aggregation). Consult those when the question touches balances, inventory, or ratios.

## Step 1: Extract intent keywords

Scan the question for the trigger verb/phrase. These keywords drive classification.

| Signal in question | Likely intent |
|---|---|
| "top N", "rank", "compare", "by category", "which X has most" | Comparison |
| "over time", "trend", "growth", "month over month", "since", "history" | Trend |
| "percentage of", "share", "breakdown", "make up", "contribution to total" | Part-to-whole |
| "relationship between", "correlation", "distribution", "outliers", "spread" | Distribution |
| "stages", "pipeline", "conversion", "drop-off", "funnel", "from X to Y", "variance" | Process/Stage |
| "where", "by region/state/country/store location", "map", "geographic" | Geographic |
| "vs target", "vs goal", "on track", "current value", "today's", single number | KPI/Goal |
| "why", "what factors", "what drives", "what influences", "what caused" | Root-cause |

If a question contains multiple signals (e.g. "trend AND regional breakdown"), it is compound: see [Multi-chart composition](#step-5-multi-chart-composition).

A question's named measures map to facts and its "by X" phrases map to dimension attributes. "Total sales **by** product category **for** 2024" → measure `Sales`, dimension `Product[Category]`, filter `Date[Year]=2024`. This is the same dimensional listening pattern used in `business-requirements-gathering` (facts = metrics, dimensions = "analyze by").

## Step 2-3: Classify intent and map to a Power BI visual

The full mapping table with multiple examples per intent lives in **[intent-to-chart-mapping.md](intent-to-chart-mapping.md)**. Read it when you need alternatives or edge cases. Quick reference:

| # | Intent | Question shape | Primary Power BI visual | Strong alternatives |
|---|---|---|---|---|
| 1 | **Comparison** | "Compare / top N / which is highest" | Clustered bar chart (horizontal — fits long labels) or clustered column chart | Stacked column, Ribbon chart (rank change over time), Matrix |
| 2 | **Trend** | "How has X changed over time" | Line chart | Area chart, Combo chart (actual column + target line) |
| 3 | **Part-to-whole** | "What share / % of total" | Donut chart or Pie chart (<8 slices) | Treemap (hierarchical or many parts), 100% stacked column |
| 4 | **Distribution** | "Relationship / spread / outliers" | Scatter chart (bubble chart adds a 3rd measure as size) | Column chart over binned values (histogram), Dot plot |
| 5 | **Process/Stage** | "Stages / conversion / variance" | Funnel chart (conversion stages) or Waterfall chart (variance/running total) | Column chart by stage |
| 6 | **Geographic** | "Where / by location" | Filled map / choropleth (relative density) | Azure map (precise points), Shape map (region comparison), ArcGIS map |
| 7 | **KPI/Goal** | "vs target / current value" | KPI (progress vs goal) or Gauge (single value within min/max) | Card / Multirow card (bare number) |
| 8 | **Root-cause** | "Why / what drives" | Key influencers (ranks factors) or Decomposition tree (drill to driver) | Q&A visual, Smart narrative |

Every visual named above is a real Power BI visual confirmed by the visualization-types documentation. Do not substitute invented chart names. Sankey and histogram are not built-in visuals — express a histogram as a **column chart over a binned column**; mention Sankey only as an AppSource custom visual if a true flow diagram is required.

## Step 4: Write the aggregating DAX measure

The chart is half the answer; the measure must aggregate correctly for the fact's additivity. Get this wrong and a correct chart shows wrong numbers.

```dax
// Comparison / Trend / Part-to-whole over an ADDITIVE fact: plain SUM is safe
Total Revenue = SUM(Sales[Revenue])

// Part-to-whole as a true percent of the visible total
Revenue % of Total =
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALLSELECTED(Product[Category]))
)

// KPI/Goal variance vs a target measure
Revenue vs Target = [Total Revenue] - [Revenue Target]

// Distribution: a ratio is NON-additive — divide sums, never average a ratio column
Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)

// Trend over a SEMI-ADDITIVE balance (inventory, account balance): average across
// time, never SUM across dates (that overcounts)
Avg Daily Inventory = AVERAGE(Inventory[QuantityOnHand])
```

When the fact is semi-additive or non-additive, stop and apply `understanding-fact-additivity` before finalizing the visual.

## Step 5: Multi-chart composition

Compound questions need more than one visual. Rules:

1. **"X and Y" in one question** → two coordinated visuals sharing slicers, or one Combo chart when X and Y share an axis. Example: "sales trend and the actual-vs-budget gap" → Combo chart (column = actual, line = budget) on a shared Date axis.
2. **"Summary, then detail"** → a summary visual (e.g. clustered bar) plus a drill path. Add row fields to a Matrix for drill-down, or a Decomposition tree to drill in any order. See `drill-pattern-design`.
3. **"Let users explore / filter"** → add Slicers (date range, category, search). For free-form natural-language questions add a Q&A visual.
4. **Different audiences** → compose visuals into an audience-specific dashboard; defer layout to `dashboard-composition`.

A model that answers ad-hoc Q&A well also produces better automatic visuals: ensure relationships exist, name tables/columns for the business term, set non-summable columns (Year, ID, Age) to "Don't summarize", and set Data Category on date/geography columns so a geographic question can render on a map. These are the Q&A optimization rules from the Power BI Q&A best-practices guidance and apply to any model.

## Worked examples

**"What are the top 10 products by revenue?"** → keyword "top N / by" = Comparison → **Clustered bar chart** (horizontal, product names on the axis), Product[Name] on Y, `Total Revenue` on X, apply a Top-N filter of 10.

**"How have sales trended over the past 12 months?"** → "trended / over months" = Trend → **Line chart**, Date[Month] on X, `Total Revenue` on Y, relative-date filter last 12 months.

**"What percentage of revenue comes from each category?"** → "percentage of" = Part-to-whole → **Donut chart** (or Treemap if >8 categories), Product[Category] in legend, `Total Revenue` as value; show `Revenue % of Total` in the tooltip/label.

**"What is our revenue vs the quarterly target?"** → "vs target" = KPI/Goal → **KPI visual**, indicator `Total Revenue`, target goal `Revenue Target`, trend axis Date[Quarter].

**"Why did customer churn increase?"** → "why" = Root-cause → **Key influencers**, Analyze = Customer[Churned], Explain by = Customer[ContractType], Customer[Tenure], [Support Call Count].

## Validation checklist

```
- [ ] Identified the intent keyword and chose exactly one of the 8 intents
- [ ] Selected a visual that appears in the Power BI visualization-types doc (no invented charts)
- [ ] Wrote a DAX measure whose aggregation matches the fact's additivity
- [ ] For part-to-whole, used DIVIDE against the correct total (not raw values)
- [ ] For semi-additive/non-additive facts, deferred to understanding-fact-additivity
- [ ] For compound questions, applied a composition rule (combo, drill, or slicers)
```

---
*Provenance: Layer 1, Skill 2 of the Power BI Visualization Skills Master Plan (`power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`). Depends on no other skill; consumed by the Layer 2 domain skills (retail, financial, supply chain, services).*
