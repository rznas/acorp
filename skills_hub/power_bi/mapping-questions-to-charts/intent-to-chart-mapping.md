# Intent-to-Chart Mapping Reference

Full reference for the 8-category question-intent framework. Each section lists the keyword patterns that trigger the intent, the primary Power BI visual, alternatives, when to prefer each, a worked question with the matching DAX measure, and the data-shape caveat. Every visual named here is confirmed in the Power BI visualization-types documentation.

## Contents
- [How to use this table](#how-to-use-this-table)
- [1. Comparison](#1-comparison--barcolumn)
- [2. Trend](#2-trend--linearea)
- [3. Part-to-whole](#3-part-to-whole--piedonuttreemap)
- [4. Distribution](#4-distribution--scatterbubble)
- [5. Process / Stage](#5-process--stage--funnelwaterfall)
- [6. Geographic](#6-geographic--maps)
- [7. KPI / Goal](#7-kpi--goal--kpigaugecard)
- [8. Root-cause](#8-root-cause--ai-visuals)
- [Keyword-extraction patterns](#keyword-extraction-patterns)
- [Multi-chart composition rules](#multi-chart-composition-rules)
- [Cardinality guardrails](#cardinality-guardrails)

## How to use this table
Match the question's verb/phrase to an intent, pick the primary visual unless a caveat applies, then write the measure. When two intents compete, the dominant verb wins; if genuinely compound, see [Multi-chart composition rules](#multi-chart-composition-rules).

---

## 1. Comparison → Bar/Column
**Keywords:** compare, top N, bottom N, rank, highest, lowest, most, least, "by [dimension]", versus across categories.

**Primary:** Clustered bar chart (horizontal) when category labels are long or there are many categories; clustered column chart when labels are short.
**Alternatives:**
- Stacked column / stacked bar — compare totals while showing one sub-breakdown.
- 100% stacked column / bar — compare composition proportions across categories.
- Ribbon chart — comparison where rank *changes over time* (highest always on top per period).
- Matrix — compare many measures across many categories in a grid with conditional formatting (data bars, color scales).

**Examples:**
- "Top 10 products by revenue" → clustered bar, Top-N filter = 10.
  ```dax
  Total Revenue = SUM(Sales[Revenue])
  ```
- "How do regions compare on sales and margin?" → Matrix (rows=Region, values=`Total Revenue`, `Gross Margin %`).
  ```dax
  Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)
  ```

**Caveat:** Beyond ~50 categories a bar/column chart is unreadable — apply a Top-N filter or switch to a Treemap. See [Cardinality guardrails](#cardinality-guardrails).

---

## 2. Trend → Line/Area
**Keywords:** over time, trend, growth, decline, since, history, month-over-month, year-over-year, "past N months", trajectory.

**Primary:** Line chart — emphasizes the shape of a series over time.
**Alternatives:**
- Area chart (basic) — emphasize magnitude/total of a single trend.
- Stacked area chart — show how multiple series accumulate into a total over time.
- Combo chart — one measure as columns and another (e.g. a target or a rate) as a line on a second Y axis.

**Examples:**
- "How have sales trended over the past 12 months?" → Line chart, Date[Month] on X.
  ```dax
  Total Revenue = SUM(Sales[Revenue])
  ```
- "Show monthly revenue with margin % overlaid" → Combo chart (column=`Total Revenue`, line=`Gross Margin %`).

**Caveat:** For a **semi-additive** balance (inventory, account balance, headcount) plotted over time, average across dates — never SUM across dates.
```dax
Avg Daily Inventory = AVERAGE(Inventory[QuantityOnHand])
```

---

## 3. Part-to-whole → Pie/Donut/Treemap
**Keywords:** percentage of, share, proportion, breakdown, make up, contribution to total, "split by".

**Primary:** Donut chart or Pie chart — best with fewer than ~8 slices.
**Alternatives:**
- Treemap — many parts, or a hierarchical part-to-whole (Department → Category → Brand); size = value.
- 100% stacked column/bar — compare part-to-whole composition across several entities at once.

**Examples:**
- "What percentage of revenue comes from each category?" → Donut chart, Product[Category] in legend.
  ```dax
  Total Revenue = SUM(Sales[Revenue])
  Revenue % of Total =
  DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALLSELECTED(Product[Category])))
  ```
- "How do categories nest into departments by sales?" → Treemap with a Department→Category hierarchy.

**Caveat:** Pie/donut lose meaning past ~8 slices — switch to Treemap. Part-to-whole only makes sense for additive facts that sum to a meaningful whole; do not build it on a ratio.

---

## 4. Distribution → Scatter/Bubble
**Keywords:** relationship between, correlation, distribution, spread, outliers, how are X distributed, "X vs Y".

**Primary:** Scatter chart — two numeric axes reveal correlation and outliers.
**Alternatives:**
- Bubble chart — scatter plus a third measure encoded as bubble size.
- Dot plot — one numeric axis against a categorical axis.
- Column chart over a binned column — the Power BI way to draw a histogram (there is no dedicated histogram visual). Create the bin column on the dimension, then count rows per bin.

**Examples:**
- "Relationship between discount depth and sales lift?" → Bubble chart (X=`Avg Discount %`, Y=`Sales Lift %`, size=`Total Units`).
  ```dax
  Avg Discount % = AVERAGE(Sales[DiscountPct])
  ```
- "Distribution of customers by purchase-frequency band" → Column chart over a `FrequencyBand` column.
  ```dax
  Customer Count = DISTINCTCOUNT(Sales[CustomerKey])
  ```

**Caveat:** Scatter samples high-density data (high-density scatter). For an exact relationship at atomic grain confirm sampling does not hide the points you care about.

---

## 5. Process / Stage → Funnel/Waterfall
**Keywords:** stages, pipeline, conversion, drop-off, "from X to Y", funnel, variance, "what caused", running total, beginning-to-ending.

**Primary by sub-intent:**
- **Sequential stages with attrition** → Funnel chart (Lead → Qualified → Prospect → Close).
- **Variance / running total** → Waterfall chart (initial → +/- changes → final; e.g. budget → variances → actual, or beginning balance → receipts → sales → ending).

**Alternative:** Column chart showing count of items by current stage when you only need the per-stage tally, not the flow.

**Examples:**
- "How many leads convert at each pipeline stage?" → Funnel chart.
  ```dax
  Opportunity Count = DISTINCTCOUNT(Pipeline[OpportunityID])
  ```
- "What drove the profit variance from last year?" → Waterfall chart, decomposing the change.
  ```dax
  Profit Variance = [Total Profit] - CALCULATE([Total Profit], PREVIOUSYEAR(Date[Date]))
  Total Profit = SUM(Sales[Profit])
  ```

**Caveat:** A true multi-path flow diagram (Sankey) is not a built-in visual — only reach for an AppSource custom visual when flow between many nodes genuinely matters.

---

## 6. Geographic → Maps
**Keywords:** where, by region / state / country / city / store location, geographic, map, density by area.

**Primary:** Filled map (choropleth) — shade regions by relative value (light=low, dark=high).
**Alternatives:**
- Azure map — plot precise point locations and categorical+quantitative spatial info.
- Shape map — relative comparison of regions by color (not precise locations).
- ArcGIS map — rich base maps, reference layers (e.g. census), advanced spatial analysis.
- Basic map — associate categorical and quantitative info with spatial locations.

**Example:**
- "Which states have the highest sales density?" → Filled map, Geography[State], color saturation = `Total Revenue`.

**Caveat:** Maps need a column with a geographic Data Category set (City, State, Country/Region, Zip). Without it Q&A and auto-visuals will not pick a map. The Bing Maps visual is being removed from the default visuals list; prefer Azure map for point maps.

---

## 7. KPI / Goal → KPI/Gauge/Card
**Keywords:** vs target, vs goal, on track, current value, today's, "are we ahead/behind", single headline number.

**Primary by sub-intent:**
- **Progress toward a goal over a trend** → KPI visual (indicator value, target goal, trend axis).
- **Single value within a known min/max** → Gauge (radial) — needle = goal, shading = progress.
- **Bare headline number** → Card (single number) or Multirow card (several data points).

**Examples:**
- "Are we on track to the quarterly revenue goal?" → KPI visual.
  ```dax
  Total Revenue = SUM(Sales[Revenue])
  Revenue Target = SUM(Targets[Amount])
  ```
- "Average sales per month vs the 200k goal" → Gauge (value=`Avg Monthly Sales`, target=200000, min=100000, max=250000).
  ```dax
  Avg Monthly Sales = AVERAGEX(VALUES(Date[YearMonth]), CALCULATE(SUM(Sales[Revenue])))
  ```

**Caveat:** A card/KPI over a **semi-additive** balance must show a period-end or average value, not a SUM across dates.
```dax
Ending Balance =
CALCULATE(SUM(Inventory[QuantityOnHand]), LASTNONBLANK(Date[Date], 1))
```

---

## 8. Root-cause → AI visuals
**Keywords:** why, what factors, what drives, what influences, what caused, what explains.

**Primary by sub-intent:**
- **Rank the factors behind an outcome** → Key influencers (Analyze = target, Explain by = candidate drivers).
- **Drill through dimensions in any order to find the driver** → Decomposition tree (one metric, multiple dimensions; AI suggests the next split).

**Alternatives:** Q&A visual (free-form natural-language probing), Smart narrative (auto-written summary of what changed).

**Examples:**
- "What factors influence customer churn?" → Key influencers.
  ```dax
  Churn Rate =
  DIVIDE(CALCULATE(DISTINCTCOUNT(Customer[CustomerKey]), Customer[Churned] = TRUE()),
         DISTINCTCOUNT(Customer[CustomerKey]))
  ```
  Analyze = Customer[Churned]; Explain by = Customer[ContractType], Customer[Tenure], [Support Call Count].
- "Why is profit down — drill to the driver" → Decomposition tree, metric=`Total Profit`, drill fields Product, Region, Store.

**Caveat:** AI visuals require a well-modeled semantic layer: defined relationships, business-friendly names, and correctly typed columns. A poorly normalized model yields misleading influencers.

---

## Keyword-extraction patterns

Parse a question into (intent, measures, dimensions, filters):

1. **Intent verb** → table above. The verb/phrase is the strongest signal.
2. **Measures** = the metric nouns the question tracks (sales, revenue, count, balance, margin). These map to facts.
3. **Dimensions** = phrases after "by", "per", "across", "for each" → dimension attributes for the axis/legend.
4. **Filters** = explicit scoping ("in 2024", "for the West region", "this quarter") → page/visual filters or slicers.
5. **Aggregation hint** = "average", "total", "ratio", "%" → drives SUM vs AVERAGE vs DIVIDE (cross-check additivity).

Example parse — "Compare average basket size by store region for last quarter":
- Intent: Comparison → clustered bar.
- Measure: average basket size → `DIVIDE(SUM(Sales[Revenue]), DISTINCTCOUNT(Sales[TransactionID]))`.
- Dimension: Store[Region] on the axis.
- Filter: Date last quarter (relative-date filter).

---

## Multi-chart composition rules

| Compound pattern | Composition |
|---|---|
| "X and Y" sharing an axis | One Combo chart (column for X, line for Y) |
| "X and Y" not sharing an axis | Two coordinated visuals bound by the same slicers |
| "Summary then detail" | Summary visual + Matrix drill-down or Decomposition tree (see `drill-pattern-design`) |
| "Compare two facts for the same entity" | Drill-across via a conformed dimension into small multiples or a Matrix |
| "Let users explore" | Add Slicers (date range, category, text search); add a Q&A visual for free-form questions |
| "Tell me what's interesting" | Smart narrative beside the visuals to auto-summarize |

Keep coordinated visuals on a shared set of conformed dimension slicers so cross-filtering stays consistent. Hand layout, density, and audience tailoring to `dashboard-composition`.

---

## Cardinality guardrails

| Dimension distinct values | Comfortable visuals |
|---|---|
| < 10 | Pie, Donut, Slicer, single Gauge/Card per value |
| 10–50 | Clustered column/bar, Treemap |
| 50+ | Top-N filtered bar, Treemap, Scatter, or a Slicer/search — avoid pie and unfiltered bar |

High cardinality is the most common reason a "correct" intent maps to the wrong visual: a part-to-whole question over 200 SKUs is a Treemap, not a 200-slice pie.
