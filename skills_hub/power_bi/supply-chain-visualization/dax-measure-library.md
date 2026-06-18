# Supply Chain DAX Measure Library

Validated, copy-ready DAX measures for supply chain Power BI reports. Table and column names are placeholders (`'Inventory'`, `'OrderPipeline'`, `'PO'`, `'Invoice'`, `'Date'`) — rename to match the semantic model.

## Contents

- Section 1: Semi-additive inventory balance handling (CRITICAL)
- Section 2: Inventory velocity — turnover and days of supply
- Section 3: Stockout and excess inventory
- Section 4: Procurement — spend, on-time delivery, lead time, fill rate
- Section 5: Order fulfillment — accumulating-snapshot lag and pipeline counts
- Section 6: Backlog and capacity
- Section 7: Profitability — gross margin and contribution (non-additive ratios)
- Section 8: Currency and UOM handling
- Section 9: Manufacturing — output, WIP, cycle time
- Measure design rules

## Measure design rules

1. **Balance facts are semi-additive.** Never `SUM` on-hand quantity or inventory value across dates. Average over distinct dates, or take the last non-blank date.
2. **Ratios are non-additive.** Compute margin %, fill rate, on-time % as `DIVIDE(SUM(numerator), SUM(denominator))` — never average a precomputed per-row ratio.
3. **Flow facts and lag facts are fully additive.** `SUM`/`AVERAGE` them freely.
4. Use `DIVIDE(num, den, 0)` to avoid divide-by-zero.
5. Lag facts are precomputed in the accumulating snapshot during ETL (workday-aware); measures aggregate them with `AVERAGE`.

## Section 1: Semi-additive inventory balance handling (CRITICAL)

Inventory on-hand is additive across product and location but NOT across dates. Three correct aggregation patterns:

```dax
-- Period-end balance: value at the latest snapshot date in context
On Hand (Latest) =
CALCULATE (
    SUM ( 'Inventory'[Quantity_On_Hand] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( 'Inventory'[Quantity_On_Hand] ) ) )
)
```

```dax
-- Average balance across the days actually in context (divides by distinct dates, not row count)
Avg Daily On Hand =
AVERAGEX (
    VALUES ( 'Date'[Date] ),
    CALCULATE ( SUM ( 'Inventory'[Quantity_On_Hand] ) )
)
```

```dax
-- Period-end inventory value (semi-additive), used in matrices, KPIs, treemaps
Inventory Value (Latest) =
CALCULATE (
    SUM ( 'Inventory'[Inventory_Value_At_Cost] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( 'Inventory'[Inventory_Value_At_Cost] ) ) )
)
```

Use `On Hand (Latest)` / `Inventory Value (Latest)` in point-in-time visuals (matrix, gauge, KPI, card, treemap). Use `Avg Daily On Hand` as the denominator for velocity measures (Section 2).

## Section 2: Inventory velocity — turnover and days of supply

`Quantity_Sold` is a fully additive flow fact in the periodic snapshot.

```dax
Quantity Sold = SUM ( 'Inventory'[Quantity_Sold] )
```

```dax
-- Inventory turns = total sold / average on-hand over the period
Inventory Turnover =
DIVIDE ( [Quantity Sold], [Avg Daily On Hand], 0 )
```

```dax
-- Days of supply = ending on-hand / average daily sales
Days of Supply =
VAR DistinctDays = DISTINCTCOUNT ( 'Date'[Date] )
VAR AvgDailySales = DIVIDE ( [Quantity Sold], DistinctDays, 0 )
RETURN
    DIVIDE ( [On Hand (Latest)], AvgDailySales, 0 )
```

## Section 3: Stockout and excess inventory

```dax
Stockout Incidents =
CALCULATE ( COUNTROWS ( 'Inventory' ), 'Inventory'[Quantity_On_Hand] = 0 )
```

```dax
-- Excess on-hand value above maximum stock level
Excess Inventory Value =
SUMX (
    'Inventory',
    VAR Excess = 'Inventory'[Quantity_On_Hand] - 'Inventory'[Max_Stock_Level]
    RETURN IF ( Excess > 0, Excess * 'Inventory'[Unit_Cost], 0 )
)
```

```dax
-- Sales velocity (units/day) for scatter-plot Y axis in stockout-risk quadrant
Sales Velocity =
DIVIDE ( [Quantity Sold], DISTINCTCOUNT ( 'Date'[Date] ), 0 )
```

## Section 4: Procurement — spend, on-time delivery, lead time, fill rate

```dax
Total Spend = SUM ( 'PO'[Extended_Receipt_Amount] )
```

```dax
-- On-time delivery rate: receipts on/before promised date / total receipts
On-Time Delivery % =
DIVIDE (
    CALCULATE ( COUNTROWS ( 'PO' ), 'PO'[Receipt_Date] <= 'PO'[Promised_Date] ),
    COUNTROWS ( 'PO' ),
    0
)
```

```dax
-- Average vendor lead time (lag fact precomputed in ETL: receipt date - PO date, workday-aware)
Avg Lead Time Days = AVERAGE ( 'PO'[Lead_Time_Days] )
```

```dax
-- Fill rate: quantity received / quantity ordered (ratio of sums, NOT average of per-line rates)
Fill Rate % =
DIVIDE ( SUM ( 'PO'[Quantity_Received] ), SUM ( 'PO'[Quantity_Ordered] ), 0 )
```

```dax
-- Late-delivery flag measure (target for Key influencers visual)
Late Deliveries =
CALCULATE ( COUNTROWS ( 'PO' ), 'PO'[Receipt_Date] > 'PO'[Promised_Date] )
```

```dax
-- Distinct vendors with activity in context (KPI card)
Vendor Count = DISTINCTCOUNT ( 'PO'[Vendor_Key] )
```

```dax
-- Average days a PO has been open (PO date -> today for open lines, -> receipt for closed)
-- Drives the PO-aging matrix
PO Days Open =
AVERAGEX (
    'PO',
    DATEDIFF (
        'PO'[PO_Date],
        IF ( ISBLANK ( 'PO'[Receipt_Date] ), TODAY (), 'PO'[Receipt_Date] ),
        DAY
    )
)
```

## Section 5: Order fulfillment — accumulating-snapshot lag and pipeline counts

Each milestone date column is blank until the order reaches that stage. Count orders that reached each milestone for a **funnel chart**:

```dax
Orders Created  = COUNTROWS ( 'OrderPipeline' )
Reached Picked  = CALCULATE ( COUNTROWS ( 'OrderPipeline' ), NOT ISBLANK ( 'OrderPipeline'[Pick_Date] ) )
Reached Packed  = CALCULATE ( COUNTROWS ( 'OrderPipeline' ), NOT ISBLANK ( 'OrderPipeline'[Pack_Date] ) )
Reached Shipped = CALCULATE ( COUNTROWS ( 'OrderPipeline' ), NOT ISBLANK ( 'OrderPipeline'[Ship_Date] ) )
Reached Delivered = CALCULATE ( COUNTROWS ( 'OrderPipeline' ), NOT ISBLANK ( 'OrderPipeline'[Delivery_Date] ) )
```

Lag facts (precomputed milestone differences) aggregated with `AVERAGE`:

```dax
Avg Order-to-Delivery Days = AVERAGE ( 'OrderPipeline'[Days_Order_To_Delivery] )

-- Per-stage averages for a waterfall "where is the bottleneck" view
Avg Days Order-to-Pick   = AVERAGE ( 'OrderPipeline'[Days_Order_To_Pick] )
Avg Days Pick-to-Pack    = AVERAGE ( 'OrderPipeline'[Days_Pick_To_Pack] )
Avg Days Pack-to-Ship    = AVERAGE ( 'OrderPipeline'[Days_Pack_To_Ship] )
Avg Days Ship-to-Deliver = AVERAGE ( 'OrderPipeline'[Days_Ship_To_Deliver] )
```

```dax
-- SLA breach detection for tables / scatter SLA reference
Days Overdue SLA =
AVERAGEX (
    'OrderPipeline',
    MAX ( 'OrderPipeline'[Days_Order_To_Delivery] - 'OrderPipeline'[SLA_Days], 0 )
)
```

If lag facts are not precomputed, derive in DAX with `DATEDIFF`:

```dax
Avg Order-to-Delivery (computed) =
AVERAGEX (
    FILTER ( 'OrderPipeline', NOT ISBLANK ( 'OrderPipeline'[Delivery_Date] ) ),
    DATEDIFF ( 'OrderPipeline'[Order_Date], 'OrderPipeline'[Delivery_Date], DAY )
)
```

## Section 6: Backlog and capacity

```dax
-- Open orders not yet shipped = backlog
Order Backlog =
CALCULATE ( COUNTROWS ( 'OrderPipeline' ), ISBLANK ( 'OrderPipeline'[Ship_Date] ) )
```

```dax
-- Backlog utilization for a gauge (current backlog vs. capacity)
Backlog vs Capacity % =
DIVIDE ( [Order Backlog], SUM ( 'Capacity'[Daily_Capacity] ), 0 )
```

## Section 7: Profitability — gross margin and contribution (non-additive ratios)

Revenue and cost are additive extended amounts; margin % is non-additive — compute as a ratio of sums.

```dax
Net Revenue   = SUM ( 'Invoice'[Extended_Net_Amount] )
Total Cost    = SUM ( 'Invoice'[Extended_Total_Cost] )
Gross Profit  = [Net Revenue] - [Total Cost]
```

```dax
Gross Margin % = DIVIDE ( [Gross Profit], [Net Revenue], 0 )
```

```dax
-- Contribution after activity-based cost layers (for waterfall P&L)
Contribution Amount =
[Net Revenue]
    - SUM ( 'Invoice'[Extended_Manufacturing_Cost] )
    - SUM ( 'Invoice'[Extended_Storage_Cost] )
    - SUM ( 'Invoice'[Extended_Distribution_Cost] )
```

## Section 8: Currency and UOM handling

Always aggregate the ETL-converted corporate-currency fact; never sum raw local amounts across regions.

```dax
Total Spend (Corporate) = SUM ( 'PO'[Extended_Receipt_Amount_USD] )
```

Multiple units of measure: base quantity is stored once, conversion factors stored as facts. Multiply inside the aggregation:

```dax
Order Quantity (Pallets) =
SUMX ( 'Order', 'Order'[Quantity_Base] * 'Order'[Factor_Cases_To_Pallets] )
```

## Section 9: Manufacturing — output, WIP, cycle time

Production output (`Quantity_Manufactured`) is a fully additive flow fact in the manufacturing/production fact (`'Production'`). Work-in-process is identified by orders started but not yet completed. Cycle-time lag (`Cycle_Time_Days`) is precomputed in ETL (completion date - start date, workday-aware) and aggregated with `AVERAGE`.

```dax
-- Total units completed (KPI card, throughput line chart, output bar chart)
Quantity Manufactured = SUM ( 'Production'[Quantity_Manufactured] )
```

```dax
-- Work-in-process: production orders started but not yet completed (KPI card, WIP funnel)
WIP Count =
CALCULATE (
    COUNTROWS ( 'Production' ),
    NOT ISBLANK ( 'Production'[Start_Date] ),
    ISBLANK ( 'Production'[Completion_Date] )
)
```

```dax
-- Average manufacturing cycle time in days (lag fact aggregated with AVERAGE)
Avg Cycle Time = AVERAGE ( 'Production'[Cycle_Time_Days] )
```
