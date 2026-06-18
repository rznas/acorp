# Supply Chain Dashboard Templates

Role-based Power BI dashboard layouts. Every visual named is a confirmed native Power BI visual. Measures referenced are defined in `dax-measure-library.md`. Each template lists the visual, what it shows, and the measure(s) that drive it.

## Contents

- Template 1: Inventory Manager
- Template 2: Procurement Manager
- Template 3: Logistics / Fulfillment Manager
- Template 4: Plant Manager (Manufacturing)
- Shared layout and interactivity guidance

## Shared layout and interactivity guidance

- **Top row = KPI cards** (single number cards or KPI visuals) for at-a-glance status.
- **Slicers** on a left rail or top: date range, warehouse/plant, product category, vendor. Slicers cross-filter all visuals.
- Keep one **semi-additive rule** in mind: any inventory-level visual must use a `(Latest)` or `Avg Daily` measure, never a raw `SUM` across dates.
- Limit to roughly a dozen visuals per page for performance; push detail to drill-through pages.
- Use conditional formatting (data bars, color scales, red/yellow/green) in **tables** and **matrix** visuals instead of non-native "heatmap" visuals.

## Template 1: Inventory Manager

Audience: operational. Focus: levels, velocity, stockout/excess risk.

| Position | Power BI visual | Shows | Measure(s) |
|---|---|---|---|
| Top row | **Cards** / **KPI** | Total inventory value, days of supply, stockout count, excess value | `Inventory Value (Latest)`, `Days of Supply`, `Stockout Incidents`, `Excess Inventory Value` |
| Upper left | **Gauge** | Current on-hand vs. target range for selected product | `On Hand (Latest)` |
| Upper right | **Scatter chart** | Stockout-risk quadrant (X = days of supply, Y = sales velocity, bubble = value) | `Days of Supply`, `Sales Velocity`, `Inventory Value (Latest)` |
| Middle | **Matrix** | Products × warehouses on-hand, conditional formatting (red = below min) | `On Hand (Latest)` |
| Lower left | **Line chart** | Average inventory level trend by category | `Avg Daily On Hand` |
| Lower right | **Clustered bar chart** | Inventory turnover by product, reference line = target turns | `Inventory Turnover` |
| Detail | **Table** | Low-stock alert list (rows = product; conditional-format `On Hand (Latest)` red when below the `'Inventory'[Min_Stock]` column) | `On Hand (Latest)` (measure); `'Inventory'[Min_Stock]` (column) |
| Rail | **Slicers** | Date, warehouse, category | — |

## Template 2: Procurement Manager

Audience: tactical. Focus: spend, vendor performance, PO aging.

| Position | Power BI visual | Shows | Measure(s) |
|---|---|---|---|
| Top row | **Cards** / **KPI** | Total spend, vendor count, on-time delivery %, avg lead time | `Total Spend`, `Vendor Count`, `On-Time Delivery %`, `Avg Lead Time Days` |
| Upper left | **Clustered bar chart** | Top vendors by spend | `Total Spend` |
| Upper right | **Scatter chart** | Vendor performance (X = avg price, Y = on-time %, bubble = order volume) | `On-Time Delivery %` + price/volume |
| Middle | **Line chart** | Monthly procurement spend trend | `Total Spend` |
| Lower left | **Clustered bar chart** | On-time delivery % by vendor, conditional format (green ≥95%, red <85%) | `On-Time Delivery %` |
| Lower right | **Key influencers chart** | Factors driving late deliveries (target = `Late Deliveries`) | `Late Deliveries` |
| Detail | **Matrix** | POs by status with aging, conditional formatting | `PO Days Open` |
| Rail | **Slicers** | Date, vendor, category | — |

## Template 3: Logistics / Fulfillment Manager

Audience: operational. Focus: order pipeline, fulfillment speed, SLA, shipment geography.

| Position | Power BI visual | Shows | Measure(s) |
|---|---|---|---|
| Top row | **Gauge** + **Cards** | Backlog vs. capacity; avg order-to-delivery days | `Backlog vs Capacity %`, `Avg Order-to-Delivery Days` |
| Upper | **Funnel chart** | Order pipeline (ordered → picked → packed → shipped → delivered) | `Orders Created`, `Reached Picked`, `Reached Packed`, `Reached Shipped`, `Reached Delivered` |
| Middle left | **Line chart** | Fulfillment-time trend by order month | `Avg Order-to-Delivery Days` |
| Middle right | **Waterfall chart** | Average days per pipeline stage (bottleneck view) | `Avg Days Order-to-Pick`, `Avg Days Pick-to-Pack`, `Avg Days Pack-to-Ship`, `Avg Days Ship-to-Deliver` |
| Lower left | **Table** | Delayed orders with days overdue, conditional formatting | `Days Overdue SLA` |
| Lower right | **Filled map (choropleth)** | Shipment volume by destination region | `Reached Shipped` by region |
| Rail | **Slicers** | Date, carrier, destination region | — |

## Template 4: Plant Manager (Manufacturing)

Audience: operational. Focus: throughput, WIP, cycle time, on-time-to-promise.

| Position | Power BI visual | Shows | Measure(s) |
|---|---|---|---|
| Top row | **Cards** / **KPI** | Units completed, WIP count, avg cycle time, on-time-to-promise % | `Quantity Manufactured`, `WIP Count`, `Avg Cycle Time`, `On-Time Delivery %` |
| Upper left | **Line chart** | Daily plant throughput | `Quantity Manufactured` by day |
| Upper right | **Funnel chart** | Work-in-process by manufacturing stage | per-milestone counts |
| Middle | **Waterfall chart** | Average days per manufacturing stage (slowest stage) | per-stage lag measures |
| Lower left | **Clustered bar chart** | Output by product line, reference line = target | `Quantity Manufactured` |
| Lower right | **Combo chart** | Gross profit (column) + margin % (line) by product | `Gross Profit`, `Gross Margin %` |
| Rail | **Slicers** | Date, plant, product line | — |
