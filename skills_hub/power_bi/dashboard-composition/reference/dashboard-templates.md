# Dashboard Templates by Audience

Ready-to-adapt dashboard (report page) layouts for executive, operational, and analytical audiences. Each template lists the visuals (top-to-bottom, left-to-right), the slicer/interactivity setup, and key DAX. Every visual name is a confirmed Power BI visual. Validate each measure's aggregation with **understanding-fact-additivity** before placing it.

## Contents

- How to use these templates
- Executive templates (1-4): Sales, Bank, CFO budget-vs-actual, Retail performance
- Operational templates (5-9): Store manager, Inventory manager, Procurement, Logistics/fulfillment, Insurance claims
- Analytical templates (10-12): Churn driver, Profit decomposition, Self-service exploration
- Mobile-optimized variant notes

## How to use these templates

1. Match the audience archetype (executive = monitor / operational = act / analytical = explore).
2. Copy the layout; replace columns/measures with the real semantic model.
3. Keep the top row for headline KPI **Card** / **KPI** / **Multi-row card** visuals.
4. Keep slicers in the same on-page position across pages and **Sync slicers** where the same filter applies.
5. Keep visuals to only what the page needs (aim for fewer than ~20 as a heuristic, not a hard Microsoft limit); push detail to drillthrough pages and tooltips.

Aggregation reminders used throughout:
- **Additive** (revenue, profit, units, counts) -> `SUM` is safe in any visual.
- **Semi-additive** (balances, on-hand, occupancy, headcount) -> never `SUM` across time; use `AVERAGE` or a period-end measure.
- **Non-additive** (ratios, %, rates) -> compute as a weighted measure with `DIVIDE`, never average a ratio.

---

## EXECUTIVE TEMPLATES (low density, one screen, no scroll, 4-8 visuals)

### Template 1 — Executive Sales Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | Total Revenue, Gross Margin %, Units, Revenue vs Target % |
| Mid left | **Line** | Revenue by month |
| Mid right | **Filled map** | Revenue by region |
| Bottom left | **Clustered bar** (horizontal) | Top 10 products by revenue |
| Bottom right | **Donut** | Revenue by category (only if < 8 categories) |
| Left rail | **Slicer** (synced) | Date, Region, Category |

```dax
Total Revenue = SUM ( Sales[Revenue] )
Gross Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )
Revenue vs Target % = DIVIDE ( [Total Revenue], SUM ( Target[Revenue_Target] ), 0 )
```

### Template 2 — Bank Executive Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | Total Deposits (period-end), Total Loans (period-end), Net Interest Margin, NPL Ratio |
| Mid left | **Line** | Average balance by account type over time |
| Mid right | **Waterfall** | Profit bridge: interest income -> interest expense -> provisions -> operating cost |
| Bottom left | **Treemap** | Customer segments by profitability |
| Bottom right | **Filled map** | Branch performance by geography |
| Left rail | **Slicer** (synced) | Date, Product type, Region |

Balances are **semi-additive** — use period-end, not SUM across dates:

```dax
Deposit Balance =
CALCULATE (
    SUM ( AccountSnapshot[Balance] ),
    LASTNONBLANK ( 'Date'[Date], 1 )
)
Average Daily Balance = AVERAGE ( AccountSnapshot[Balance] )
Net Interest Margin =
DIVIDE ( [Interest Income] - [Interest Expense], [Average Earning Assets], 0 )
```

### Template 3 — CFO Budget vs Actual Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | YTD Actual, YTD Budget, Variance %, Budget Remaining |
| Mid (wide) | **Waterfall** | Budget -> favorable variances -> unfavorable variances -> Actual |
| Bottom left | **Matrix** | Department x (Budget, Actual, Variance %) with conditional formatting (red = over) |
| Bottom right | **Line** | Monthly Actual vs Budget |
| Left rail | **Slicer** (synced) | Fiscal period, Division, Account |

```dax
Actual = SUM ( GL[Actual_Amount] )
Budget = SUM ( Budget[Budget_Amount] )
Variance = [Actual] - [Budget]
Variance % = DIVIDE ( [Variance], [Budget], 0 )
```

### Template 4 — Retail Executive Performance Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | Net Sales, Same-Store Sales %, Avg Basket, Conversion Rate |
| Mid left | **Line** | Weekly sales trend |
| Mid right | **Clustered column** | Sales by channel |
| Bottom (wide) | **Treemap** | Department -> Category contribution |
| Left rail | **Slicer** (synced) | Date, Region, Channel |

```dax
Net Sales = SUM ( Sales[Net_Amount] )
Average Basket = DIVIDE ( [Net Sales], DISTINCTCOUNT ( Sales[Transaction_ID] ), 0 )
Conversion Rate = DIVIDE ( DISTINCTCOUNT ( Sales[Transaction_ID] ), SUM ( Traffic[Visitors] ), 0 )
```

---

## OPERATIONAL TEMPLATES (medium density, status-led, alerts, 1-2 pages)

### Template 5 — Store Manager Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row | **Gauge** | Today's revenue vs target |
| Top row | **KPI** | Transactions today (trend vs yesterday) |
| Mid (wide) | **Matrix** | Product x (Sales, Margin %, On-hand) with conditional formatting |
| Bottom left | **Column** | Hourly sales pattern |
| Bottom right | **Funnel** | Promotion participation -> purchase |
| Footer | **Table** | Low-stock alerts (Top N filter, red conditional format) |
| Top rail | **Slicer** (Tile, responsive) | Department |

```dax
Revenue Today = CALCULATE ( [Net Sales], 'Date'[Date] = TODAY () )
On Hand = AVERAGE ( Inventory[Quantity_On_Hand] )   -- semi-additive
Below Min Flag = IF ( [On Hand] < MAX ( Product[Min_Stock] ), 1, 0 )
```

### Template 6 — Inventory Manager Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | Inventory Value, Days of Supply, Stockout Count, Excess Value |
| Mid left | **Gauge** | Current inventory vs target range |
| Mid right | **Scatter** | Stockout-risk quadrant (X: days of supply, Y: sales velocity) |
| Bottom (wide) | **Matrix** | Product x Warehouse, on-hand with conditional formatting |
| Footer | **Table** | Low-stock alerts (Top N) |
| Left rail | **Slicer** (synced) | Category, Warehouse |

Inventory quantity is **semi-additive** — define a per-product on-hand measure first, then value it. Never put a column `AVERAGE` inside a `SUMX` row iterator over the same fact table: the `AVERAGE` ignores the iterator's row context, returns the one overall average quantity for every row, and so `SUMX` produces (overall average qty) x (sum of all unit costs) — an inflated, meaningless number.

```dax
-- semi-additive on-hand per product (use AVERAGE, or LASTNONBLANK for period-end)
On Hand = AVERAGE ( Inventory[Quantity_On_Hand] )

-- value the already-aggregated on-hand per product, then sum across products
Inventory Value = SUMX ( Product, [On Hand] * Product[Unit_Cost] )

-- both measures reuse [On Hand] for a consistent basis
Days of Supply = DIVIDE ( [On Hand], [Avg Daily Units Sold], 0 )
Inventory Turnover = DIVIDE ( [Units Sold], [On Hand], 0 )
```

### Template 7 — Procurement Manager Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row (x4) | **Card** | Total Spend, Vendor Count, On-Time Delivery %, Avg Lead Time |
| Mid left | **Clustered bar** | Top vendors by spend |
| Mid right | **Scatter** | Vendor price vs on-time % (bubble = volume) |
| Bottom left | **Line** | Monthly spend trend |
| Bottom right | **Matrix** | PO status with aging, conditional formatting |
| Left rail | **Slicer** (synced) | Category, Vendor, Date |

```dax
Total Spend = SUM ( PO[Spend_Amount] )
On Time Delivery % = DIVIDE ( [On Time Receipts], [Total Receipts], 0 )
Avg Lead Time = AVERAGE ( PO[Lead_Time_Days] )
```

### Template 8 — Logistics / Fulfillment Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row | **Gauge** | Current backlog vs capacity |
| Top row | **Card** | Avg days order-to-delivery |
| Mid (wide) | **Funnel** | Ordered -> Picked -> Packed -> Shipped -> Delivered |
| Bottom left | **Line** | Fulfillment-time trend |
| Bottom right | **Filled map** | Shipment volume by destination region |
| Footer | **Table** | Delayed orders, days overdue (conditional formatting) |
| Left rail | **Slicer** (synced) | Warehouse, Date, Carrier |

```dax
Avg Order To Delivery =
AVERAGEX ( Orders, DATEDIFF ( Orders[Order_Date], Orders[Delivered_Date], DAY ) )
Backlog = CALCULATE ( COUNTROWS ( Orders ), Orders[Status] <> "Delivered" )
```

### Template 9 — Insurance Claims Manager Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row | **Gauge** | Claim backlog vs target |
| Top row | **Card** | Avg days to settlement |
| Mid (wide) | **Funnel** | Reported -> Reserved -> Payment -> Closed |
| Bottom left | **Line** | Avg settlement-time trend |
| Bottom right | **Clustered column** | Claim count by coverage type |
| Footer | **Table** | High-value claims with aging (conditional formatting) |
| Left rail | **Slicer** (synced) | Coverage type, Status, Date |

```dax
Avg Days To Settlement =
AVERAGEX ( Claims, DATEDIFF ( Claims[Reported_Date], Claims[Payment_Date], DAY ) )
Open Claims = CALCULATE ( COUNTROWS ( Claims ), Claims[Status] <> "Closed" )
```

---

## ANALYTICAL TEMPLATES (high density, drill + AI, 2-4 pages)

### Template 10 — Customer Churn Driver Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row | **Card** | Churn Rate, Active Customers |
| Mid (wide) | **Key influencers** | Target: Churned; explain by contract type, tenure, support calls |
| Bottom left | **Scatter** | Tenure vs monthly spend (bubble = lifetime value) |
| Bottom right | **Line** | Churn rate trend (enable anomaly detection) |
| Footer | **Textbox** with **Smart narrative** | Auto summary responding to slicers |
| Left rail | **Slicer** (synced) | Segment, Region, Plan |

```dax
Churn Rate = DIVIDE ( [Churned Customers], [Customers At Start], 0 )
```

### Template 11 — Profit Decomposition Dashboard

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top row | **Card** | Total Profit, Gross Margin % |
| Mid (wide) | **Decomposition tree** | Profit drilled by Channel -> Category -> Product |
| Bottom left | **Waterfall** | Profit contribution by segment |
| Bottom right | **Matrix** | Segment x metrics with conditional formatting |
| Left rail | **Slicer** (synced) | Date, Channel, Region |

```dax
Total Profit = SUM ( Sales[Profit] )
Gross Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )
```

### Template 12 — Self-Service Exploration Page

| Zone | Visual | Field / measure |
|------|--------|-----------------|
| Top (wide) | **Q&A** | Free-text natural-language queries |
| Mid left | **Matrix** | Drill-down enabled across the date/product hierarchy |
| Mid right | **Scatter** | Configurable measures |
| Footer | **Textbox** with **Smart narrative** | Page summary |
| Left rail | **Slicer** (synced) | Date, Product, Geography |

Requires a well-named semantic model for Q&A (coordinate with **ai-powered-analysis-selection**).

---

## Mobile-optimized variant notes

For each template, build a mobile layout alongside the desktop layout:

- Include only the most-used visuals (typically the top-row KPIs plus one trend).
- Arrange top-to-bottom (flow is vertical on mobile); most important first.
- Don't place visuals side by side except single-element **Card** / **KPI** visuals and non-data **Button** / **Slicer** objects.
- Place slicers/page navigators horizontally; minimize titles (no font below 9 pt); remove axis titles, gridlines, legends.
- Max canvas width 323 pt; keep 6-8 pt between elements; size each visual so no inner scroll bar appears.
- Recommended minimum sizes: XL 323x270 pt, L 323x180 pt, M 323x100 pt, S 158x100 pt.
