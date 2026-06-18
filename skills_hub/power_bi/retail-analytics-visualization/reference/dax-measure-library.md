# Retail DAX Measure Library

Reusable, syntactically valid DAX for retail Power BI models. Adjust table/column names to your model. Assumed tables: `'POS Sales'`, `'Inventory Snapshot'`, `'Profitability'`, `'Clickstream Session'`, `'Date'`, `'Product'`, `'Store'`, `'Promotion'`, `'Customer'`.

## Contents
- Core sales measures
- Margin and profitability (non-additive ratios)
- Same-store (comparable) sales
- Basket / transaction measures
- Promotion lift and comparison
- Inventory (semi-additive) and turnover
- Sell-through and GMROI
- Conversion / clickstream
- Time intelligence helpers

---

## Core sales measures

```dax
Total Revenue = SUM ( 'POS Sales'[Extended Sales Dollars] )

Total Units = SUM ( 'POS Sales'[Sales Quantity] )

Total Cost = SUM ( 'POS Sales'[Extended Cost Dollars] )

Transaction Count =
DISTINCTCOUNT ( 'POS Sales'[POS Transaction Number] )

Distinct Customers = DISTINCTCOUNT ( 'POS Sales'[Customer Key] )
```

## Margin and profitability (non-additive ratios)

Always divide sums; never average a stored row-level percent.

```dax
Total Gross Profit = SUM ( 'POS Sales'[Gross Profit Dollars] )

Gross Margin % =
DIVIDE ( [Total Gross Profit], [Total Revenue], 0 )

Net Profit = SUM ( 'Profitability'[Net Profit Dollars] )

Net Margin % =
DIVIDE ( [Net Profit], SUM ( 'Profitability'[Net Revenue Dollars] ), 0 )
```

## Same-store (comparable) sales

Compares revenue only for stores that recorded sales in BOTH the current and prior comparable period (e.g. same period last year).

The comparable-store set is a TABLE, so it CANNOT be a measure (measures must
return a scalar). Inline it inside each scalar measure with a variable, then
apply it as a filter via `TREATAS` + `KEEPFILTERS`. (Alternatively, define it
once as a calculated table - never as a measure - and relate it to the model.)

```dax
Revenue PY =
CALCULATE ( [Total Revenue], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

Same-Store Revenue =
-- revenue this period for stores that sold in BOTH this period and last year
VAR StoresNow =
    VALUES ( 'POS Sales'[Store Key] )
VAR StoresPY =
    CALCULATETABLE (
        VALUES ( 'POS Sales'[Store Key] ),
        SAMEPERIODLASTYEAR ( 'Date'[Date] )
    )
VAR Comparable =
    INTERSECT ( StoresNow, StoresPY )
RETURN
    CALCULATE (
        [Total Revenue],
        KEEPFILTERS ( TREATAS ( Comparable, 'POS Sales'[Store Key] ) )
    )

Same-Store Revenue PY =
-- last-year revenue for the SAME comparable stores
VAR StoresNow =
    VALUES ( 'POS Sales'[Store Key] )
VAR StoresPY =
    CALCULATETABLE (
        VALUES ( 'POS Sales'[Store Key] ),
        SAMEPERIODLASTYEAR ( 'Date'[Date] )
    )
VAR Comparable =
    INTERSECT ( StoresNow, StoresPY )
RETURN
    CALCULATE (
        [Total Revenue],
        SAMEPERIODLASTYEAR ( 'Date'[Date] ),
        KEEPFILTERS ( TREATAS ( Comparable, 'POS Sales'[Store Key] ) )
    )

Same-Store Sales % =
DIVIDE (
    [Same-Store Revenue] - [Same-Store Revenue PY],
    [Same-Store Revenue PY],
    0
)
```

## Basket / transaction measures

```dax
Average Basket Size =
DIVIDE ( [Total Revenue], [Transaction Count], 0 )

Units per Basket =
DIVIDE ( [Total Units], [Transaction Count], 0 )

Avg Units per Transaction =
AVERAGEX ( VALUES ( 'POS Sales'[POS Transaction Number] ), [Total Units] )
```

## Promotion lift and comparison

Lift compares promoted-period sales to a baseline. Use the "No Promotion" promotion row as baseline and/or a prior comparable period.

```dax
Promoted Revenue =
CALCULATE ( [Total Revenue], 'Promotion'[Promotion Name] <> "No Promotion" )

Baseline Revenue =
CALCULATE ( [Total Revenue], 'Promotion'[Promotion Name] = "No Promotion" )

-- True promotional lift: promoted vs No-Promotion baseline.
-- Lift = (Promoted - Baseline) / Baseline  (Kimball causal-dimension lift).
-- For incremental lift, normalize per unit / per day and net out time-shifting
-- (Pre/During/Post, see Promotion Period Bucket) and cannibalization
-- (compare the whole category, not just the promoted product).
Promotion Lift % =
DIVIDE ( [Promoted Revenue] - [Baseline Revenue], [Baseline Revenue], 0 )

-- Year-over-year growth (NOT promoted-vs-baseline). Use for trend, not lift.
Revenue YoY % =
VAR PriorYear =
    CALCULATE ( [Total Revenue], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
RETURN
    DIVIDE ( [Total Revenue] - PriorYear, PriorYear, 0 )

-- Period bucket for time-shifting analysis (Pre / During / Post)
-- Define StartDate / EndDate as model parameters or hardcode per promotion.
Promotion Period Bucket =
VAR d = MAX ( 'Date'[Date] )
RETURN
    SWITCH (
        TRUE (),
        d < [Promo Start Date], "Pre-Promotion",
        d <= [Promo End Date], "During Promotion",
        "Post-Promotion"
    )

Units Change vs Prior Period =
[Total Units] - CALCULATE ( [Total Units], PREVIOUSMONTH ( 'Date'[Date] ) )
```

## Inventory (semi-additive) and turnover

On-hand quantity is SEMI-ADDITIVE: sum across product/store, but take period-end or average across time.

```dax
Ending On Hand Qty =
CALCULATE (
    SUM ( 'Inventory Snapshot'[On Hand Quantity] ),
    LASTNONBLANK (
        'Date'[Date],
        CALCULATE ( SUM ( 'Inventory Snapshot'[On Hand Quantity] ) )
    )
)

Average On Hand Qty = AVERAGE ( 'Inventory Snapshot'[On Hand Quantity] )

Ending Inventory Value =
CALCULATE (
    SUM ( 'Inventory Snapshot'[On Hand Value] ),
    LASTNONBLANK (
        'Date'[Date],
        CALCULATE ( SUM ( 'Inventory Snapshot'[On Hand Value] ) )
    )
)

-- COGS over the period / average inventory value
Inventory Turnover =
DIVIDE ( [Total Cost], [Average Inventory Value], 0 )

Average Inventory Value = AVERAGE ( 'Inventory Snapshot'[On Hand Value] )

Days of Supply =
DIVIDE (
    [Ending On Hand Qty],
    DIVIDE ( [Total Units], COUNTROWS ( VALUES ( 'Date'[Date] ) ), 0 ),
    0
)

Sales Velocity =
DIVIDE ( [Total Units], COUNTROWS ( VALUES ( 'Date'[Date] ) ), 0 )
```

## Sell-through and GMROI

```dax
-- Units sold / (units sold + units remaining on hand)
Sell-Through % =
DIVIDE (
    [Total Units],
    [Total Units] + [Ending On Hand Qty],
    0
)

-- Gross Margin Return On Inventory Investment
GMROI =
DIVIDE ( [Total Gross Profit], [Average Inventory Value], 0 )
```

## Conversion / clickstream

```dax
Total Sessions = COUNTROWS ( 'Clickstream Session' )

Sessions With Order =
CALCULATE ( [Total Sessions], 'Clickstream Session'[Orders Placed] > 0 )

Conversion Rate % =
DIVIDE ( [Sessions With Order], [Total Sessions], 0 )

Abandoned Sessions =
CALCULATE (
    [Total Sessions],
    'Session'[Success Status] = "Abandoned Cart"
)

Abandonment Rate % =
DIVIDE ( [Abandoned Sessions], [Total Sessions], 0 )

Order Dollars = SUM ( 'Clickstream Session'[Order Dollars] )

Revenue per Session =
DIVIDE ( [Order Dollars], [Total Sessions], 0 )
```

## Time intelligence helpers

```dax
YTD Revenue =
TOTALYTD ( [Total Revenue], 'Date'[Date] )

Revenue MoM % =
VAR Prev = CALCULATE ( [Total Revenue], PREVIOUSMONTH ( 'Date'[Date] ) )
RETURN DIVIDE ( [Total Revenue] - Prev, Prev, 0 )

Average Daily Revenue =
DIVIDE ( [Total Revenue], DISTINCTCOUNT ( 'Date'[Date] ), 0 )
```

---

## Usage rules

- Wrap every ratio in `DIVIDE(num, den, 0)` to avoid divide-by-zero.
- For inventory and any balance, use `Ending On Hand Qty` / `Average On Hand Qty`, never `SUM` directly in a time-series visual.
- Same-store logic depends on a clean Store Key and a contiguous Date dimension; verify both stores and dates exist in both periods.
- `[Promo Start Date]` / `[Promo End Date]` in the period-bucket measure should come from the Promotion dimension or report parameters, not be invented per query.
