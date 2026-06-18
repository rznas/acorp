# Decision Matrix and Cardinality Thresholds

All visual names below are confirmed Power BI visuals from the Power BI "Visualization types" documentation. Do not substitute invented chart names.

## Contents
- Decision matrix: fact type + question type → Power BI visual
- Dimension cardinality thresholds per visualization type
- Cardinality decision rules
- Confirmed Power BI visual inventory (reference)

---

## Decision matrix: fact type + question type → Power BI visual

Read a row by the **fact table type** (Step 1) and the **question type** the stakeholder is asking. The cell names the primary Power BI visual(s); the rightmost column gives the aggregation note tied to additivity.

| Fact type | Question type | Recommended Power BI visual(s) | Aggregation / additivity note |
|---|---|---|---|
| **Transaction** | Trend over time | Line chart; basic/stacked area chart | Additive measures: `SUM`. One series per low-cardinality dimension member. |
| **Transaction** | Comparison across categories | Column chart (vertical); bar chart (horizontal, for long labels) | Additive measures: `SUM`. |
| **Transaction** | Ranking (Top/Bottom N) | Bar chart with Top N filter; ribbon chart (rank change over time) | Additive `SUM`; apply Top N when dimension cardinality is high. |
| **Transaction** | Part-to-whole | Pie chart, donut chart (low cardinality); treemap (hierarchical) | Additive `SUM`; only meaningful for additive facts. |
| **Transaction** | Relationship / distribution / outliers | Scatter chart, bubble chart (third measure = size), dot plot | Plot measures or per-row atomic values; non-additive facts shown as points, not summed. |
| **Transaction** | Atomic detail / exact values | Table; matrix; decomposition tree (drill to leaf) | Atomic grain; drill-through target. |
| **Transaction** | Two measures, one with target/different scale | Combo chart (column + line) | e.g., column = `SUM(profit)`, line = `DIVIDE(SUM(profit),SUM(revenue))`. |
| **Periodic snapshot** | Current level / single value | Card (single number); multirow card | Semi-additive balance: show one period; never `SUM` across dates. |
| **Periodic snapshot** | Progress vs goal / health of one metric | Gauge chart (radial); KPI | Single semi-additive value vs target. |
| **Periodic snapshot** | Balance trend over time | Line chart with `AVERAGE` (or last-balance measure) | Semi-additive: `AVERAGE`/`LASTNONBLANK`, NOT `SUM`. |
| **Periodic snapshot** | Status grid across two dimensions | Matrix (rows × columns of dimensions, cells = balance) | Use explicit semi-additive measure; matrix auto-aggregates. |
| **Periodic snapshot** | Period activity (received/sold) within snapshot | Column chart; waterfall chart (beginning → +/- → ending) | Period-activity facts are additive; balances are not. |
| **Accumulating snapshot** | Pipeline stage counts / conversion | Funnel chart | Count rows or `SUM` milestone 0/1 flags. |
| **Accumulating snapshot** | Stage contribution to a total | Waterfall chart | Additive stage amounts. |
| **Accumulating snapshot** | Milestone lag / cycle time | Card; gauge chart; line chart (lag trend) | Lag facts (e.g., `order_to_ship_lag_days`) averaged with `AVERAGE`. |
| **Any** | Geographic distribution | Filled map (choropleth); shape map; Azure map; ArcGIS map; basic map | Color/size from additive measure; map needs a geo-categorized dimension. |
| **Any** | Free-form / ad-hoc question | Q&A visual | Requires well-named semantic model. See `ai-powered-analysis-selection`. |
| **Any** | Root cause / "why" | Decomposition tree; key influencers chart | AI visuals; see `ai-powered-analysis-selection`. |
| **Any** | Page narrative summary | Smart narrative | Auto-text; responds to filters. |
| **Any** | User-driven filtering | Slicer | Driven by dimension attribute; low-to-medium cardinality preferred. |

Forecasting/advanced statistics that no native visual covers can use the R script visual. Custom visuals (e.g., Sankey, heatmap) come from Microsoft AppSource and are not native; prefer native visuals first.

---

## Dimension cardinality thresholds per visualization type

Cardinality = distinct values of the dimension field placed on an axis, legend, category, or slicer. Thresholds are practical readability guidance, not hard product limits.

| Cardinality band | Distinct values | Best-fit Power BI visuals | Avoid |
|---|---|---|---|
| **Low** | < 10 | Pie chart, donut chart, gauge chart, card, slicer (list/buttons), column chart | High-density scatter |
| **Medium** | 10 – 50 | Column chart, bar chart, treemap, line chart (as series count stays low), matrix, ribbon chart | Pie/donut (slices unreadable) |
| **High** | 50 – ~hundreds | Bar chart with **Top N** filter, matrix with drill-down, decomposition tree, scatter / scatter (high density), table | Pie, donut, gauge, individual slicer buttons |
| **Very high** | thousands+ | Slicer (search) / filters only, Top N bar chart, scatter (high-density sampling), table with search | Any chart that renders one mark per value without sampling |

Notes:
- **Legend / series cardinality** must stay low (typically < 8) regardless of axis cardinality; many overlapping line/area series become unreadable. Use small multiples or a single series with a slicer instead.
- **Pie and donut** show part-to-whole only; reserve for low cardinality and additive facts.
- **Treemap** handles more categories than pie and supports hierarchy (e.g., Department → Category → Brand), good for medium cardinality and large hierarchical sets.
- **Scatter (high density)** samples points so very-high-cardinality data stays responsive while preserving outliers.
- **Matrix** and **decomposition tree** auto-aggregate and let users drill, so they tolerate high cardinality by revealing detail progressively rather than all at once.

---

## Cardinality decision rules

1. Count distinct values of the field destined for the axis/legend/category.
2. < 10 and additive part-to-whole question → pie or donut chart.
3. 10–50 → column or bar chart; if hierarchical, treemap.
4. 50+ → bar chart with Top N filter, or matrix / decomposition tree for drill-down.
5. Thousands+ → restrict to slicers/filters, Top N, or high-density scatter; do not place on a categorical axis raw.
6. Keep legend/series ≤ ~8 members; otherwise switch to a slicer or separate visuals.

---

## Confirmed Power BI visual inventory (reference)

Native visuals confirmed by the visualization-types documentation, usable in recommendations:

Area chart (basic and stacked); bar chart; column chart; card (single number and multirow); combo chart; decomposition tree; donut chart; funnel chart; gauge chart (radial); key influencers chart; KPI; line chart; maps (basic map, ArcGIS map, Azure map, filled map / choropleth, shape map); matrix; pie chart; Power Apps visual; Q&A visual; R script visual; ribbon chart; scatter chart, bubble chart, dot plot, and high-density scatter; slicer; smart narrative; standalone image; table; treemap; waterfall chart.
