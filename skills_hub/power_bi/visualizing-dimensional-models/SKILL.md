---
name: visualizing-dimensional-models
description: Translates dimensional model structure (grain, fact table type, additivity, hierarchies, dimension cardinality) into correct Power BI visualization choices and DAX aggregations. Use when designing Power BI reports or dashboards from a star schema, deciding which visual fits a fact table, validating that an aggregation is safe for a fact's additivity, mapping grain to detail vs summary visuals, or choosing visuals based on dimension cardinality. Triggers include "which chart for this fact", "transaction vs snapshot visual", "semi-additive balance", "grain", "star schema in Power BI", "dimension hierarchy drill-down".
---

# Visualizing Dimensional Models

Dimensional modeling decisions made upstream (grain, fact table type, additivity, hierarchies, cardinality) determine which Power BI visuals are correct and which DAX aggregation is safe. This skill maps those structural properties to specific, real Power BI visuals before any chart is placed on the canvas.

In Power BI's star schema model, **dimension tables enable filtering and grouping** (report row/column headers, slicers, axes) and **fact tables enable summarization** (the numeric values). Table type is determined by relationship cardinality: the "one" side is a dimension, the "many" side is a fact.

## Prerequisites and sibling skills

This is a Layer 1 foundation skill. It assumes the dimensional model already exists (built via the Kimball `four-step-dimensional-modeling` and `fact-table-design-patterns` skills) and informs the domain skills built on top of it.

- **Build on this skill**: `mapping-questions-to-charts` (question-intent → visual), `understanding-fact-additivity` (deep additivity + DAX templates), and the domain skills (`retail-analytics-visualization`, `financial-services-visualization`, `supply-chain-visualization`, `services-analytics-visualization`).
- **For drill design**, see `drill-pattern-design`. For AI/"why" visuals, see `ai-powered-analysis-selection`.

## Workflow

Copy this checklist and work through it for each visual you design:

```
Dimensional Visualization Checklist:
- [ ] Step 1: Identify the fact table TYPE (transaction / periodic snapshot / accumulating snapshot)
- [ ] Step 2: Classify each fact's ADDITIVITY (additive / semi-additive / non-additive)
- [ ] Step 3: Identify the GRAIN (atomic vs aggregated) the visual will display
- [ ] Step 4: Check DIMENSION CARDINALITY for any field on an axis/legend/category
- [ ] Step 5: Pick the visual from the decision matrix + cardinality thresholds
- [ ] Step 6: Validate the aggregation with the additivity checklist (run the validation loop)
```

### Step 1: Identify the fact table type

The fact table type signals the analytic shape and the visual family. (See `fact-table-design-patterns` for design detail.)

| Fact table type | What it captures | Visual affinity |
|---|---|---|
| **Transaction** | One row per discrete event (sale, order line, call) | Line chart, area chart (trend over time); scatter/bubble (individual points); decomposition tree, table (atomic detail) |
| **Periodic snapshot** | One row per entity per regular interval; balances/levels | Card, KPI, gauge chart (current level vs goal); matrix; line chart with **average** (not sum) over time |
| **Accumulating snapshot** | One row per pipeline item, updated across milestones | Funnel chart (stage counts), waterfall chart (stage contribution), card/gauge (milestone lag) |

### Step 2: Classify fact additivity

Additivity decides which DAX aggregation is valid, which then constrains the visual. Three classes (from Kimball four-step Step 4):

- **Additive** — summable across all dimensions (sales quantity, extended sales amount, extended cost, gross profit). Safe in any aggregating visual with `SUM`.
- **Semi-additive** — summable across some dimensions but **not time** (account balance, inventory on hand, headcount). Never `SUM` across dates; use `AVERAGE` or a last-period-balance measure.
- **Non-additive** — not summable across any dimension (unit price, ratios, percentages). Store the numerator and denominator as separate additive facts; compute the ratio as `DIVIDE(SUM(num), SUM(den))` — a ratio of sums, never a sum or average of ratios.

DAX examples (syntactically valid):

```dax
Total Revenue = SUM(Sales[ExtendedSalesAmount])

Avg Daily Inventory = AVERAGE(Inventory[QuantityOnHand])

Ending Balance =
CALCULATE (
    SUM ( AccountSnapshot[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( AccountSnapshot[Balance] ) ) )
)

Gross Margin % = DIVIDE ( SUM ( Sales[GrossProfit] ), SUM ( Sales[ExtendedSalesAmount] ), 0 )
```

Power BI implicit measures default to `SUM`, which silently overcounts semi-additive balances. Hide such columns and expose explicit measures so report authors cannot sum a balance by accident (star schema guidance: control aggregation with explicit measures).

### Step 3: Map grain to detail vs summary

Atomic grain (the lowest level captured) supports maximum dimensionality and detail; aggregated grain supports summary trend visuals. You can always roll atomic data up; you cannot drill summary data down.

| Grain shown | Recommended visuals |
|---|---|
| **Atomic** (one row per event/line) | Table (exact values, drill-through target), scatter / dot plot (individual points), decomposition tree (drill to leaf) |
| **Aggregated** (daily/monthly rollups) | Column chart, bar chart, line chart, area chart, KPI, card |
| **Multi-process consolidated** (drill-across via conformed dimensions) | Combo chart (two measures, e.g., actual vs target), matrix (metric columns), or coordinated visuals sharing one dimension |

Cross-fact combinations require **conformed dimensions** (identical attribute names, values, and definitions). In Power BI a conformed dimension is one shared dimension table with relationships to multiple fact tables; place each fact's measure in its own visual or as separate series in a combo chart. See `drill-pattern-design` for drill-across.

### Step 4 and 5: Cardinality and visual selection

The number of distinct values in a dimension field placed on an axis, legend, or category determines which visuals stay readable. Low-cardinality fields suit part-to-whole and slicer visuals; high-cardinality fields require ranking/Top N or filters.

For the full **decision matrix** (fact type + question type → visual) and the **dimension cardinality thresholds** per visual, see [reference/decision-matrix.md](reference/decision-matrix.md).

### Step 6: Validate the aggregation (feedback loop)

Run this validation loop before finalizing. If any check fails, fix the measure or change the visual, then re-check.

For the per-chart-category **additivity validation checklist**, see [reference/additivity-validation.md](reference/additivity-validation.md).

## Worked examples (question → fact property → visual + DAX)

**Example 1 — Transaction fact, additive, trend.**
Q: "How has revenue trended over the past 12 months?"
Fact: POS sales transaction (additive `ExtendedSalesAmount`), aggregated to month.
Visual: **Line chart** (Date month on X-axis, measure on Y).
```dax
Total Revenue = SUM(Sales[ExtendedSalesAmount])
```

**Example 2 — Periodic snapshot, semi-additive, current level.**
Q: "What is the average daily inventory by product?"
Fact: inventory periodic snapshot (semi-additive `QuantityOnHand`).
Visual: **Card** (single value) or **matrix** by product. Do not `SUM` over time.
```dax
Avg Daily Inventory = AVERAGE(Inventory[QuantityOnHand])
```

**Example 3 — Non-additive ratio.**
Q: "What is gross margin % by product category?"
Fact: sales transaction; margin is non-additive, so use stored numerator/denominator.
Visual: **Combo chart** — column for `Total Profit`, line for `Gross Margin %`.
```dax
Total Profit  = SUM(Sales[GrossProfit])
Gross Margin % = DIVIDE ( SUM ( Sales[GrossProfit] ), SUM ( Sales[ExtendedSalesAmount] ), 0 )
```

**Example 4 — Accumulating snapshot, pipeline.**
Q: "How many orders are at each fulfillment stage?"
Fact: order fulfillment accumulating snapshot (milestone dates + 0/1 milestone flags).
Visual: **Funnel chart** (stage on category, count of orders as value).
```dax
Orders Shipped = SUM(OrderPipeline[ShippedFlag])
```

**Example 5 — Low-cardinality part-to-whole.**
Q: "What share of revenue comes from each of our 5 product categories?"
Fact: additive revenue; dimension cardinality = 5 (low).
Visual: **Donut chart** or **pie chart** (≤ low-cardinality threshold).
```dax
Total Revenue = SUM(Sales[ExtendedSalesAmount])
```

**Example 6 — High-cardinality ranking.**
Q: "What are the top 10 products by revenue?" (Product = thousands of SKUs.)
Visual: **Bar chart** (horizontal, reads long product names) with a Top N filter; never a pie chart at this cardinality.
```dax
Total Revenue = SUM(Sales[ExtendedSalesAmount])
```

## Anti-patterns

- Summing a semi-additive balance across the Date dimension (e.g., a line chart of `SUM(inventory)` over months) — overcounts. Use `AVERAGE` or a last-balance measure.
- Averaging a ratio (`AVERAGE(Margin%)`) instead of a ratio of sums — wrong result. Use `DIVIDE(SUM(num), SUM(den))`.
- Pie/donut/treemap on a high-cardinality dimension — unreadable. Rank with bar + Top N, or filter.
- Showing atomic transaction grain in an aggregating chart hoping to "see detail" — use a table or drill-through to a detail page instead.
- Mixing two fact grains in one visual without conformed dimensions — produces meaningless cross-products.

---
*Provenance: Implements Layer 1 / Skill 1 "Visualizing Dimensional Models" from `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
