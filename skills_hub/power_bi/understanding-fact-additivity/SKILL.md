---
name: understanding-fact-additivity
description: Determines whether a numeric fact is additive, semi-additive, or non-additive and selects the correct DAX aggregation (SUM, AVERAGE, LASTNONBLANK, DIVIDE) and Power BI visual so totals are mathematically correct. Use when designing measures, choosing how to aggregate a field, or picking a chart for sums/balances/ratios — especially for inventory levels, account balances, headcount, prices, percentages, margins, or rates — and when a total looks inflated, a balance is being summed across dates, or a percentage is being averaged.
---

# Understanding Fact Additivity for Visualization

## Purpose

Power BI visuals aggregate automatically (usually SUM). For balances and ratios that default is **wrong** and silently produces inflated or meaningless totals. This skill classifies each fact's additivity, then prescribes the correct DAX measure and a compatible Power BI visual.

**Prerequisites (from the master plan):** This is a Layer 1 skill. It assumes dimensional-modeling basics (grain, fact vs. dimension) and DAX measure fundamentals. Pair it with `picking-the-right-visual` when the goal is matching a question to a chart.

## The three additivity types

| Type | Can sum across… | Cannot sum across… | Typical facts | Default DAX |
|------|-----------------|--------------------|---------------|-------------|
| **Additive** | ALL dimensions | — | sales quantity, revenue, cost, profit, durations, counts | `SUM` |
| **Semi-additive** | all dimensions EXCEPT time | time (dates) | inventory on hand, account balance, headcount, backlog | `AVERAGE` over time, or last/closing balance |
| **Non-additive** | NO dimension | every dimension | unit price, %, margin, rate, ratio, temperature, score | store components, `DIVIDE` of sums |

**Core rule for ratios:** compute the **ratio of sums**, never the **sum (or average) of ratios**.

## Detection workflow

Copy this checklist and work top-down. The first YES classifies the fact.

```
Fact additivity check:
- [ ] Step 1 — Accumulated amount/quantity (flow)? e.g. units sold, $ deposited, seconds elapsed
       YES -> ADDITIVE. Use SUM.
- [ ] Step 2 — Balance/level captured at a point in time? e.g. on-hand qty, account balance, headcount
       YES -> SEMI-ADDITIVE. Sum across non-time dims; AVERAGE or last/closing balance across time.
- [ ] Step 3 — Ratio, rate, percentage, price, or intensity? e.g. margin %, unit price, °F, score
       YES -> NON-ADDITIVE. Store numerator + denominator; DIVIDE in a measure.
- [ ] Step 4 — None of the above -> the fact is likely misclassified; re-examine its grain and meaning.
```

Fast validation (the SUM-across-dates test): mentally sum two consecutive daily values.
- "Mon 50 + Tue 50 = 100 units sold" makes sense -> additive.
- "Mon 50 + Tue 50 = 100 on hand" is double-counting the same 50 units -> semi-additive.
- "40% + 30% = 70% margin" is nonsense -> non-additive.

## DAX by additivity (quick reference)

```dax
-- ADDITIVE
Total Revenue = SUM(Sales[Revenue])

-- SEMI-ADDITIVE: average over time, still sums over product/store
Avg Daily Inventory = AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))

-- SEMI-ADDITIVE: period-end balance (latest non-blank date in context)
Ending Inventory =
    CALCULATE(SUM(Inventory[QtyOnHand]), LASTNONBLANK('Date'[Date], CALCULATE(SUM(Inventory[QtyOnHand]))))

-- NON-ADDITIVE: ratio of sums (NOT average of per-row margins)
Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))

-- NON-ADDITIVE: weighted-average price = ratio of sums
Avg Unit Price = DIVIDE(SUM(Sales[ExtendedAmount]), SUM(Sales[Quantity]))
```

Full templates (CLOSINGBALANCE variants, headcount, NPS, distinct count, sparse/NULL facts): see [dax-measure-templates.md](dax-measure-templates.md).

## Choosing a visual

A visual is **Safe** when its default SUM over every dimension on the visual is correct for that fact. Pick by additivity:

- **Additive** → any visual with confidence: Line chart and Area chart (trend over time), Column chart and Bar chart (compare categories), Stacked area chart, Pie chart and Donut chart and Treemap (part-to-whole), Waterfall chart (running total / contribution), Ribbon chart (rank over time), Map / Filled map (geographic totals), single-number Card and KPI (one grand total).
- **Semi-additive** → safe as a point-in-time snapshot: single-number Card or KPI (one period's balance), Gauge chart (current balance vs. target), Table or Matrix with an explicit period grouping. A Line chart of the balance is safe **only if** the measure averages or takes the last value over time — never raw SUM. Avoid Pie / Donut / Treemap / Stacked area / Waterfall, because they sum across time into the part-to-whole math.
- **Non-additive** → never let the visual SUM the raw value. Use a `DIVIDE`/ratio-of-sums measure, then Card / KPI (one value), Line or Column chart (the measure over categories/time), Gauge (vs. target), Matrix or Table for per-row display, Scatter chart (relationship between two non-additive measures), Combo chart (a non-additive line over an additive column). Avoid Pie / Donut / Treemap / Stacked area on raw percentages or prices.

Full grid with Safe / Caution / Avoid per fact type × chart type, plus the reason for each Caution: see [visualization-compatibility-matrix.md](visualization-compatibility-matrix.md).

## Worked examples (question → visual + DAX)

**1. "Total revenue by month" (additive)**
Line chart. X = `Date[Month]`, Y = `Total Revenue = SUM(Sales[Revenue])`. SUM over months is correct; the line shows the trend.

**2. "Average daily inventory by product this month" (semi-additive)**
Clustered bar chart (product on axis) or a Matrix. Measure:
`Avg Daily Inventory = AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))`.
Do NOT drop raw `QtyOnHand` onto a chart — the implicit SUM adds the same stock across every day.

**3. "Current cash balance vs. target" (semi-additive)**
Gauge chart. Value = `Ending Balance` (LASTNONBLANK pattern above), Target = the goal measure. A single-number Card also works for one period.

**4. "Gross margin % by category" (non-additive)**
Column chart (or Matrix). Measure: `Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))`. The ratio-of-sums reaggregates correctly at every level. A simple average of per-product margins would weight a $100 product equally with a $1M product and report the wrong number.

**5. "Relationship between unit price and units sold" (non-additive vs. additive)**
Scatter chart. X = `Avg Unit Price` (DIVIDE), Y = `Total Quantity` (SUM), bubbles = products. Plots each product as a point rather than summing prices.

## Common errors and corrections

| Error | Symptom | Correction |
|-------|---------|------------|
| Raw balance on a time axis (implicit SUM) | Inventory/cash total grows every day; year total = ~365× reality | Use AVERAGEX-over-dates or LASTNONBLANK; never raw `SUM` on a balance over time |
| `AVERAGE(QtyOnHand)` for "average daily inventory" | Result skewed by row count (products × days), not day count | `AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(...)))` so it divides by distinct days |
| Averaging a stored percentage column | Small and large segments weighted equally; wrong total margin | `DIVIDE(SUM(numerator), SUM(denominator))` — ratio of sums |
| Summing unit prices | Meaningless total price; "avg" wrong | Store `ExtendedAmount = qty × price`; `DIVIDE(SUM(ExtendedAmount), SUM(Quantity))` |
| Balance in a Pie/Donut/Treemap | Slices sum across time, total exceeds the real balance | Filter to one period, or switch to a Card / point-in-time measure |
| Storing 0 instead of NULL for inapplicable facts | Averages deflated, counts inflated | Leave inapplicable facts blank; `SUM`/`AVERAGE`/`COUNT` ignore blanks |

More cases (distinct-count overlap, percentage facts that *are* additive via extended values, beginning/ending balance pattern): see [common-errors-and-corrections.md](common-errors-and-corrections.md).

---
*Provenance: built per SKILL 3 "Understanding Fact Additivity for Visualization" in `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`; sourced from Kimball `fact-additivity-guide.md`, `industry-supply-chain-patterns`, and the Power BI visualization-types and transform-model docs.*
