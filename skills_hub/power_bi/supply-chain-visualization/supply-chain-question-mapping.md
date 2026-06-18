# Supply Chain Question → Visualization Mapping

Maps 35+ supply chain business questions to the source fact table, the recommended Power BI visual, and the measure to use. All visual names are confirmed native Power BI visuals. Where a measure is referenced, see `dax-measure-library.md`.

## Contents

- Section 1: Inventory level questions (semi-additive)
- Section 2: Inventory movement questions (additive)
- Section 3: Stockout and excess inventory questions
- Section 4: Procurement and vendor performance questions
- Section 5: Order fulfillment pipeline questions (accumulating snapshot)
- Section 6: Fill rate and service-level questions
- Section 7: Multi-currency / multi-UOM procurement questions
- Section 8: Cost and profitability questions
- Section 9: Value-chain integration (drill-across) questions
- Section 10: Manufacturing / plant questions
- How to read this table

## How to read this table

Each row gives: **Question** → **Source fact** (and additivity) → **Power BI visual** → **Measure / note**. Pick the visual that matches both the question intent and the fact additivity. When a question implies a time axis over a balance fact, switch to a point-in-time visual (matrix/gauge/KPI/card) or use an averaging measure.

## Section 1: Inventory level questions (semi-additive)

| # | Question | Source fact (additivity) | Power BI visual | Measure / note |
|---|---|---|---|---|
| 1 | What is the current on-hand inventory by product and warehouse? | Periodic snapshot (semi-additive) | **Matrix** (product × warehouse) | `On Hand (Latest)` — never `SUM` across dates |
| 2 | Which products hold the most inventory value? | Periodic snapshot (semi-additive) | **Treemap** (Category → Product, size by value) | `Inventory Value (Latest)` |
| 3 | Is current on-hand within target range for product X? | Periodic snapshot | **Gauge** (needle = target, min/max thresholds) | `On Hand (Latest)` vs. reorder point |
| 4 | What is total inventory value vs. last period? | Periodic snapshot | **KPI** (value + trend axis = month-end) | `Inventory Value (Latest)` with target |
| 5 | Which products are below minimum stock level? | Periodic snapshot | **Table** with conditional formatting (red = below min) | `On Hand (Latest)` − `Min_Stock` |
| 6 | How does average inventory differ by category? | Periodic snapshot (semi-additive) | **Clustered bar chart** | `Avg Daily On Hand` (avg over distinct dates) |

## Section 2: Inventory movement questions (additive)

| # | Question | Source fact (additivity) | Power BI visual | Measure / note |
|---|---|---|---|---|
| 7 | What is inventory turnover by product? | Periodic snapshot (flow ÷ avg balance) | **Clustered bar chart** with reference line (target turns) | `Inventory Turnover` |
| 8 | What are days of supply by product category? | Periodic snapshot | **Clustered bar chart** | `Days of Supply` |
| 9 | How did inventory change (beginning → receipts → sales → adjustments → ending)? | Transaction / snapshot deltas | **Waterfall chart** | additive flow facts as positive/negative steps |
| 10 | How many receipts/picks/ships occurred per day? | Transaction (additive) | **Line chart** (or **Column chart** by day) | `SUM(Transaction_Quantity)` filtered by type |
| 11 | What is the mix of inventory transaction types this month? | Transaction (additive) | **Donut chart** (or **Clustered bar** if many types) | count by transaction category |
| 12 | At what hour of day do most shipments leave? | Transaction (additive) | **Column chart** (hour bins) | `COUNTROWS` of ship transactions |

## Section 3: Stockout and excess inventory questions

| # | Question | Source fact | Power BI visual | Measure / note |
|---|---|---|---|---|
| 13 | Which products have stockout risk vs. excess inventory? | Periodic snapshot + flow | **Scatter chart** (X = days of supply, Y = sales velocity, bubble = value) | quadrant analysis: low supply + high velocity = stockout risk |
| 14 | What is the trend of stockout incidents? | Transaction / event fact | **Line chart** (count by week) | `Stockout Incidents` |
| 15 | Which warehouses have the most excess inventory value? | Periodic snapshot | **Clustered bar chart** | `Excess Inventory Value` (on-hand above max) |
| 16 | Why are stockouts concentrated somewhere? | Periodic snapshot | **Decomposition tree** (Warehouse → Category → Product) | `Stockout Incidents` (AI root-cause) |

## Section 4: Procurement and vendor performance questions

| # | Question | Source fact (additivity) | Power BI visual | Measure / note |
|---|---|---|---|---|
| 17 | What is on-time delivery rate by vendor? | PO receipt fact (additive) | **Clustered bar chart**, conditional format (green ≥95%, red <85%) | `On-Time Delivery %` |
| 18 | How do vendors compare on price vs. delivery vs. volume? | PO receipt fact | **Scatter chart** / bubble (X = avg price, Y = on-time %, size = volume) | three-measure comparison |
| 19 | What is total procurement spend by vendor? | PO fact (additive) | **Clustered bar chart** (top N) | `Total Spend` |
| 20 | How has monthly procurement spend trended? | PO fact (additive) | **Line chart** | `Total Spend` by month |
| 21 | Which POs are aging past expected receipt? | Procurement accumulating snapshot | **Table** / **Matrix** with conditional formatting on days open | `PO Days Open` |
| 22 | What is average vendor lead time? | Procurement accumulating snapshot (lag) | **Card** or **Clustered bar chart** by vendor | `Avg Lead Time Days` |
| 23 | What factors influence late deliveries? | PO receipt fact | **Key influencers chart** (target = Late flag) | AI: explain by vendor, category, ship mode |

## Section 5: Order fulfillment pipeline questions (accumulating snapshot)

| # | Question | Source fact | Power BI visual | Measure / note |
|---|---|---|---|---|
| 24 | How many orders are at each stage (ordered → picked → packed → shipped → delivered)? | Order accumulating snapshot | **Funnel chart** | one measure per milestone (`Reached Picked`, `Reached Shipped`, …) |
| 25 | What is the count of orders by current stage? | Accumulating snapshot | **Column chart** (or **Clustered bar**) | `COUNTROWS` grouped by Status dimension |
| 26 | What is average order-to-delivery time? | Accumulating snapshot (lag) | **Card** | `Avg Order-to-Delivery Days` |
| 27 | Is fulfillment time trending up or down? | Accumulating snapshot (lag) | **Line chart** (avg lag by order month) | `Avg Order-to-Delivery Days` |
| 28 | Which orders are delayed beyond SLA? | Accumulating snapshot | **Table** with conditional formatting (days overdue) | `Days Overdue SLA` |
| 29 | How does fulfillment time relate to order date (SLA breaches)? | Accumulating snapshot | **Scatter chart** (X = order date, Y = days to fulfill; SLA reference line) | `Days_Order_To_Delivery` |
| 30 | Where is the pipeline bottleneck? | Accumulating snapshot (stage lags) | **Waterfall chart** (avg days per stage) | per-stage lag measures |

## Section 6: Fill rate and service-level questions

| # | Question | Source fact (additivity) | Power BI visual | Measure / note |
|---|---|---|---|---|
| 31 | What is the fill rate by product category? | Order/PO fact (additive qty) | **Clustered bar chart** or **Matrix** | `Fill Rate %` = received ÷ ordered |
| 32 | What is the order backlog vs. capacity? | Periodic snapshot / open orders | **Gauge** (current backlog vs. capacity) | `Order Backlog` |
| 33 | How is on-time shipment performance distributed? | Invoice/shipment fact | **Column chart** over service-level bands ("On Time", "1-5 Late", "6+ Late") | count by service-level descriptor |

## Section 7: Multi-currency / multi-UOM procurement questions

| # | Question | Source fact | Power BI visual | Measure / note |
|---|---|---|---|---|
| 34 | What is total procurement spend in corporate currency? | PO fact (corporate-currency column) | **Card** | `SUM(Corporate_Currency_Amount)` — never sum local amounts |
| 35 | How does spend compare across regions (mixed currencies)? | PO fact (corporate currency) | **Filled map (choropleth)** | corporate-currency spend by region |
| 36 | What is order quantity expressed in pallets vs. consumer units? | Order fact + conversion factors | **Matrix** | use UOM-view measures (base × conversion factor) |

## Section 8: Cost and profitability questions

| # | Question | Source fact (additivity) | Power BI visual | Measure / note |
|---|---|---|---|---|
| 37 | What is gross margin by product (profit + margin %)? | Invoice line (additive revenue & cost) | **Combo chart** (column = gross profit, line = margin %) | `Gross Profit`, `Gross Margin %` (ratio of sums) |
| 38 | What is the P&L structure of an order (revenue → costs → contribution)? | Invoice line | **Waterfall chart** | extended net amount → cost layers → `Contribution Amount` |
| 39 | Which products/channels are most/least profitable? | Invoice line | **Decomposition tree** (Channel → Category → Product) | `Contribution Amount` (AI drill) |

## Section 9: Value-chain integration (drill-across) questions

| # | Question | Source facts | Power BI visual | Measure / note |
|---|---|---|---|---|
| 40 | Show inventory, orders, and shipments for product X together | Drill-across via conformed Product/Date | three small **Line charts** (one per fact) on one page | conformed dimension; one measure per fact |
| 41 | Compare procurement receipts vs. inventory levels vs. shipped qty by month | Drill-across | **Matrix** (product rows, metric columns) | one measure per fact, conformed Date |

## Section 10: Manufacturing / plant questions

| # | Question | Source fact | Power BI visual | Measure / note |
|---|---|---|---|---|
| 42 | What is plant throughput trend (units completed per day)? | Manufacturing transaction (additive) | **Line chart** | `SUM(Quantity_Manufactured)` by day |
| 43 | What is work-in-process by stage? | Manufacturing accumulating snapshot | **Funnel chart** | count reaching each manufacturing milestone |
| 44 | What is average manufacturing cycle time, and which stage is slowest? | Manufacturing accumulating snapshot (lag) | **Waterfall chart** (avg days per stage) + **Card** (total) | per-stage lag measures |
