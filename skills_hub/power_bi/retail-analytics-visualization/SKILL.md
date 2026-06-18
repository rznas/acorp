---
name: retail-analytics-visualization
description: Recommends Power BI visualizations and DAX measures for retail and e-commerce analytical questions built on Kimball-style retail dimensional models (POS transactions, product hierarchies, promotions/causal dimension, customer behavior, inventory snapshots, clickstream). Use when designing Power BI reports or dashboards for stores, grocery chains, specialty or department retail, or web retailers; or when the user asks how to visualize retail metrics such as same-store sales, basket size, sell-through, conversion rate, GMROI, promotion lift, or category/product/store performance.
---

# Retail Analytics Visualization

Map retail business questions to the right Power BI visual and DAX measure, grounded in the underlying retail dimensional model (grain, fact type, additivity).

## Prerequisites (sibling skills)

This is a Layer 2 domain skill. Apply these foundation skills first when in doubt:
- `visualizing-dimensional-models` - how grain, fact type, and additivity drive visual choice.
- `mapping-questions-to-charts` - question-intent taxonomy (comparison, trend, part-to-whole, distribution, process, geographic, KPI, root-cause).
- `understanding-fact-additivity` - safe aggregation for additive / semi-additive / non-additive facts (critical for inventory and margin %).

## Retail data model assumptions

Recommendations assume these standard retail facts and dimensions:

| Fact table | Grain | Key facts | Additivity note |
|---|---|---|---|
| POS Sales (transaction) | One row per product line item per POS ticket | sales quantity, extended sales $, extended cost $, gross profit $ | quantity/dollars additive; unit prices and gross margin % non-additive |
| Inventory snapshot (periodic) | One row per product per store/warehouse per day/week | on-hand quantity, on-hand value | **semi-additive** - never SUM across dates |
| Profitability (transaction) | One row per line item per ticket, all channels | full P&L (net revenue, costs, net profit) | additive dollar facts |
| Clickstream session | One row per completed session | session seconds, pages visited, orders placed, units, order $ | additive |
| Clickstream page event | One row per page event per session | page seconds, units ordered, order $ (mostly null) | additive |

Common dimensions: Date (with holiday/season/weekday text indicators), Product (flattened SKU->Brand->Category->Department + many non-hierarchical attributes), Store (geographic + organizational hierarchies), Promotion (causal: price reduction, ad, display, coupon; includes a "No Promotion" row), Customer, plus clickstream Page / Event / Session / Referral.

## Workflow

```
- [ ] Step 1: Identify the question category (product, store, promotion, customer, trend, inventory, e-commerce, profitability)
- [ ] Step 2: Locate the fact table + grain + additivity for the measure
- [ ] Step 3: Pick the visual from reference/question-visual-map.md
- [ ] Step 4: Pull or write the DAX from reference/dax-measure-library.md
- [ ] Step 5: For semi-additive (inventory) or ratio (margin %) facts, verify aggregation is NOT a naive SUM
- [ ] Step 6: Assemble into an audience dashboard (reference/dashboard-templates.md)
- [ ] Step 7: Add drill-through to atomic transaction detail (see Drill-through patterns below)
```

## Reference files

- **Question-to-visual mapping (30+ questions):** [reference/question-visual-map.md](reference/question-visual-map.md)
- **Dashboard templates (executive, store manager, merchandiser, e-commerce):** [reference/dashboard-templates.md](reference/dashboard-templates.md)
- **DAX measure library (same-store sales, basket size, conversion, sell-through, GMROI, lift):** [reference/dax-measure-library.md](reference/dax-measure-library.md)

## Quick visual selection by intent

Only the Power BI visuals below are used in this skill; all are confirmed standard visuals.

| Retail intent | Power BI visual |
|---|---|
| Rank products / stores / vendors | Bar chart (horizontal, for long labels) or Column chart |
| Category contribution / part-to-whole | Treemap (hierarchical), Donut chart or Pie chart (<10 parts) |
| Sales / traffic over time | Line chart; Area chart for cumulative emphasis |
| Actual + rate together (e.g. profit + margin %) | Combo chart (column + line, two Y axes) |
| Rank change across periods | Ribbon chart |
| Price vs volume, RFM, inventory quadrant | Scatter chart / bubble chart |
| Sales by geography | Filled map (choropleth), Shape map, or Azure map |
| Single KPI vs target | Card, Multirow card, KPI, or Gauge chart |
| Funnel / checkout / promo participation stages | Funnel chart |
| Profit bridge / inventory roll-forward | Waterfall chart |
| Detail with conditional formatting (low-stock alerts) | Table or Matrix |
| Cross-dimension drill / root cause | Decomposition tree |
| "What drives X?" | Key influencers chart |
| Ad-hoc natural-language query | Q&A visual |
| Auto text summary of a page | Smart narrative |
| Interactive filtering | Slicers |

## Concrete examples

**Top products by revenue** (POS sales, additive)
- Visual: Bar chart (horizontal), Product[SKU Description] on Y, measure on X, sorted descending, Top N filter.
- DAX: `Total Revenue = SUM ( 'POS Sales'[Extended Sales Dollars] )`

**Gross margin trend by category** (POS sales; margin % is NON-additive)
- Visual: Combo chart - column = Total Gross Profit, line = Gross Margin %.
- DAX (compute ratio from sums, never average a row-level %):
  ```dax
  Gross Margin % =
  DIVIDE ( SUM ( 'POS Sales'[Gross Profit Dollars] ), SUM ( 'POS Sales'[Extended Sales Dollars] ), 0 )
  ```

**Current inventory vs target** (inventory snapshot; SEMI-additive)
- Visual: Gauge chart (value = on-hand, target = reorder point) or KPI.
- DAX (period-end balance, not a SUM across dates):
  ```dax
  Ending On Hand Qty =
  CALCULATE (
      SUM ( 'Inventory Snapshot'[On Hand Quantity] ),
      LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( 'Inventory Snapshot'[On Hand Quantity] ) ) )
  )
  ```

**Checkout funnel** (clickstream page events / session)
- Visual: Funnel chart with stages Visit -> Add to Cart -> Checkout -> Purchase.
- DAX: distinct session counts per stage (see dax-measure-library.md, "Conversion rate").

## Critical rules

- Inventory on-hand and any balance is **semi-additive**: use period-end (LASTNONBLANK) or AVERAGE across time; SUM across dates over-counts.
- Margin %, sell-through %, conversion %, lift % are **non-additive ratios**: compute with `DIVIDE(SUM(...), SUM(...))`, never `AVERAGE` of a stored percent.
- Promotion lift requires a baseline; analysis spans pre/during/post windows (time-shifting) and the full category (cannibalization) - see dax-measure-library.md.
- Promotion cost is NOT a fact in the POS line-item table (wrong grain); treat it as a promotion attribute or a separate promotion-grain fact.
- Use the "No Promotion" promotion row (no NULL keys) when splitting promoted vs baseline.

## Drill-through patterns

- **Summary -> transaction detail:** Configure a detail page filtered by Product (or Store/Promotion) showing a Table at POS atomic grain (ticket number degenerate dimension, line items). Right-click a bar/column to drill through.
- **Cross-dimension exploration:** Use a Decomposition tree on a measure (e.g. Net Profit) drilling Channel -> Department -> Category -> SKU; enable the AI "high/low value" split for root cause.
- **Market basket:** Detail page grouped by POS ticket number (degenerate dimension) to show co-purchased products.
- **Web-to-store drill-across:** Use conformed Date/Product/Customer to place a clickstream visual and a POS visual on one page; cross-filter via Customer slicer.

---

Provenance: implements SKILL 4 "Retail Analytics Visualization" from `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.
