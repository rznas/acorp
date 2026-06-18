---
name: supply-chain-visualization
description: Recommends Power BI visualizations and DAX measures for supply chain analytical questions across inventory, procurement, order fulfillment, logistics, and manufacturing. Maps business questions to specific Power BI visuals (matrix, gauge, funnel, waterfall, scatter, combo, KPI, card, line, clustered bar) using inventory dimensional models (periodic, transaction, and accumulating snapshots), and handles semi-additive inventory balances and accumulating-snapshot lag calculations. Use when building Power BI reports or dashboards for inventory management, on-hand stock levels, stockouts, inventory turnover, days of supply, procurement and vendor performance, on-time delivery, fill rate, lead time, order fulfillment pipelines, backlog, shipment logistics, or plant/manufacturing throughput.
---

# Supply Chain Visualization

Recommend Power BI visuals and DAX for supply chain reporting. Supply chain combines three inventory fact-table types plus procurement and order-management facts, each with a different visualization affinity:

- **Periodic snapshot** (on-hand levels) → **semi-additive** balances. Status visuals: matrix, gauge, KPI, card, treemap. Never `SUM` a balance across dates.
- **Transaction fact** (receipts, picks, ships, adjustments) → **fully additive**. Trend/frequency visuals: line, column, clustered bar.
- **Accumulating snapshot** (order/lot pipeline) → milestone dates + lag facts. Pipeline visuals: funnel, waterfall, card/gauge (lag), scatter (SLA).

Prerequisites from the master plan: this is a Layer 2 skill depending on `visualizing-dimensional-models`, `mapping-questions-to-charts`, and especially `understanding-fact-additivity` (semi-additive inventory balances). Apply those skills' fundamentals first.

## Workflow

```
- [ ] Step 1: Identify the source fact type (periodic snapshot / transaction / accumulating snapshot / procurement / invoice)
- [ ] Step 2: Classify fact additivity (additive flow vs. semi-additive balance vs. non-additive ratio)
- [ ] Step 3: Classify question intent (status, trend, comparison, part-to-whole, pipeline, distribution, root cause)
- [ ] Step 4: Pick the Power BI visual from the mapping table (supply-chain-question-mapping.md)
- [ ] Step 5: Write the DAX measure (dax-measure-library.md), guarding balances with averaging/last-value patterns
- [ ] Step 6: If composing a full report, apply a role-based template (dashboard-templates.md)
```

**Critical rule (Step 2 governs Step 4):** A balance fact (`quantity on hand`, inventory value) is **semi-additive** — additive across product/location, NOT across dates. Do not place a raw `SUM` of on-hand in a time-series line chart; use an average-over-days or last-date measure. See `dax-measure-library.md`. Flow facts (quantity sold, received, shipped) and lag facts are fully additive — safe in any aggregating visual.

## Fact type → visual affinity (quick reference)

| Source fact | Additivity | Typical question | Power BI visual |
|---|---|---|---|
| Inventory periodic snapshot | Semi-additive balance | Current on-hand by product/warehouse | **Matrix**, **Treemap** (by value), **Gauge** (vs. min/max), **KPI** |
| Inventory periodic snapshot | Additive flow (sold) | Turnover, days of supply ranking | **Clustered bar chart** with reference line |
| Inventory transaction | Additive | Receipts/adjustments over time; stockout count trend | **Line chart**, **Column chart**, **Waterfall** (beg→receipts→sales→adj→end) |
| Procurement (PO receipt) | Additive | On-time delivery %, spend, vendor compare | **Clustered bar chart**, **Scatter**/bubble (price vs on-time) |
| Order accumulating snapshot | Milestone + lag | Pipeline stage counts; order→delivery lag | **Funnel chart**, **Card**/**Gauge** (lag), **Scatter** (SLA) |
| Invoice / order line | Additive revenue & cost | Gross margin, P&L by order | **Combo chart** (profit + margin %), **Waterfall** |

All visual names above are confirmed Power BI visuals. Do not substitute chart types that are not in `mapping`-confirmed Power BI visuals (e.g. there is no native "histogram" — build one with a **column chart** over bins; "heatmap" is not native — use **matrix** with conditional formatting).

## Concrete examples

**Q: "What is current on-hand inventory by product and warehouse?"**
Source: periodic snapshot (semi-additive). Visual: **Matrix** (rows = product, columns = warehouse). Measure (use latest snapshot date, not SUM across dates):
```dax
On Hand (Latest) =
CALCULATE (
    SUM ( 'Inventory'[Quantity_On_Hand] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( 'Inventory'[Quantity_On_Hand] ) ) )
)
```

**Q: "How many orders are at each fulfillment stage (ordered → picked → packed → shipped → delivered)?"**
Source: order accumulating snapshot. Visual: **Funnel chart** (one measure counting orders that reached each milestone).
```dax
Reached Shipped = CALCULATE ( COUNTROWS ( 'OrderPipeline' ), NOT ISBLANK ( 'OrderPipeline'[Ship_Date] ) )
```

**Q: "What is the average time from order to delivery, and is it trending up?"**
Source: accumulating snapshot lag. Visual: **Card** (current average) plus **Line chart** (trend by order month).
```dax
Avg Order-to-Delivery Days = AVERAGE ( 'OrderPipeline'[Days_Order_To_Delivery] )
```

**Q: "Gross margin by product (revenue and margin % together)?"**
Source: invoice line (additive revenue & cost). Visual: **Combo chart** (column = gross profit, line = margin %). Margin is non-additive — compute as a ratio of sums:
```dax
Gross Margin % = DIVIDE ( SUM ( 'Invoice'[Extended_Net_Amount] ) - SUM ( 'Invoice'[Extended_Total_Cost] ), SUM ( 'Invoice'[Extended_Net_Amount] ), 0 )
```

## Reference files

- **`supply-chain-question-mapping.md`** — 35+ business questions → source fact, recommended Power BI visual, and the measure to use. Read this first when choosing a visual.
- **`dax-measure-library.md`** — Validated DAX for inventory turnover, days of supply, fill rate, on-time delivery, lead time, backlog, gross margin, plus semi-additive balance handling and accumulating-snapshot lag patterns.
- **`dashboard-templates.md`** — Role-based dashboard layouts: inventory manager, procurement manager, logistics/fulfillment, plant manager.

## Common mistakes to avoid

1. `SUM`-ing on-hand inventory across dates (overcounts) — use `AVERAGEX` over distinct dates or `LASTNONBLANK`.
2. Putting a balance fact in a stacked-over-time visual — prefer point-in-time visuals (matrix, gauge, KPI, card).
3. Averaging a precomputed ratio (margin %, fill rate) across rows — recompute as `DIVIDE(SUM(numerator), SUM(denominator))`.
4. Using a transaction fact to answer "current inventory level" — that needs the periodic snapshot.
5. Summing local-currency procurement amounts across regions — aggregate the corporate-currency fact instead.

---
*Provenance: SKILL 6 (Layer 2, Supply Chain Visualization) of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
