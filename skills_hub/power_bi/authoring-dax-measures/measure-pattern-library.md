# DAX Measure Pattern Library

Copy-paste measure patterns. Replace `Sales` / `Inventory` / `'Date'` / column names with your model's. Every measure is syntactically valid DAX. Ratios use `DIVIDE`; semi-additive balances never naive-`SUM` across time.

## Contents
- Base measures (additive)
- Safe ratios (DIVIDE)
- Percentage of total
- Running total
- Year-over-year (YoY) and period change
- Ranking (RANKX)
- Iterators (row-level math before aggregation)
- Semi-additive (no SUM across time)
- Variables (VAR/RETURN) for clarity and speed

> Many of these correspond to **quick measures** categories (Totals → Running total; Time intelligence → YoY change, rolling average; Aggregate per category). You can generate a starter via right-click field → **New quick measure**, then adapt the formula shown in the formula bar. Detailed time-intelligence (YTD/QTD/MTD, moving averages, period-end balances) lives in `applying-time-intelligence`.

---

## Base measures (additive)

```dax
Total Sales    = SUM(Sales[SalesAmount])
Total Quantity = SUM(Sales[SalesQuantity])
Total Cost     = SUM(Sales[TotalCost])
Total Profit   = [Total Sales] - [Total Cost]
Order Count    = COUNTROWS(Sales)
Distinct Orders = DISTINCTCOUNT(Sales[OrderNumber])
```

Reference base measures inside other measures rather than re-deriving the aggregation.

## Safe ratios (DIVIDE)

`DIVIDE` returns blank on divide-by-zero instead of an error; pass a third argument for an alternate result. Compute the **ratio of sums**, never the average of per-row ratios.

```dax
Gross Margin %   = DIVIDE([Total Profit], [Total Sales])
Avg Unit Price   = DIVIDE(SUM(Sales[ExtendedAmount]), SUM(Sales[SalesQuantity]))
Return Rate      = DIVIDE([Returned Quantity], [Total Quantity], 0)
```

## Percentage of total

Use `CALCULATE` with `ALL`/`ALLSELECTED` to build a denominator that ignores the relevant filter.

```dax
-- % of the grand total of the whole category column
Pct of All Categories =
    DIVIDE([Total Sales], CALCULATE([Total Sales], ALL(Product[Category])))

-- % of the visible (slicer-selected) total
Pct of Selected =
    DIVIDE([Total Sales], CALCULATE([Total Sales], ALLSELECTED(Product[Category])))
```

## Running total

Cumulative total up to the date in the current filter context.

```dax
Running Total Sales =
    CALCULATE(
        [Total Sales],
        FILTER(
            ALLSELECTED('Date'[Date]),
            'Date'[Date] <= MAX('Date'[Date])
        )
    )
```

`Running total` is also available directly as a quick measure (Totals category).

## Year-over-year (YoY) and period change

```dax
-- Prior-year value
Sales PY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))

-- YoY absolute change
Sales YoY = [Total Sales] - [Sales PY]

-- YoY percentage (DIVIDE handles a blank/zero prior period)
Sales YoY % = DIVIDE([Total Sales] - [Sales PY], [Sales PY])

-- Previous quarter (from the DAX-basics doc pattern)
Previous Quarter Sales =
    CALCULATE(SUM(Sales[SalesAmount]), PREVIOUSQUARTER('Date'[DateKey]))
```

These require a proper date table; for full time-intelligence patterns and pitfalls (incomplete/non-contiguous ranges, fiscal calendars) use `applying-time-intelligence`.

## Ranking (RANKX)

`RANKX` iterates a table and ranks each row by an expression.

```dax
-- Rank products by sales (1 = highest); Dense: ties share a rank, next rank is consecutive — 1,1,2
Product Rank =
    RANKX(
        ALL(Product[ProductName]),
        [Total Sales],
        ,
        DESC,
        Dense
    )

-- Top-N flag for conditional display
Is Top 10 = IF([Product Rank] <= 10, 1, 0)
```

## Iterators (row-level math before aggregation)

```dax
-- Extended amount when only quantity and unit price are stored
Total Revenue = SUMX(Sales, Sales[SalesQuantity] * Sales[UnitPrice])

-- Average of a row-level expression
Avg Line Margin = AVERAGEX(Sales, Sales[SalesAmount] - Sales[TotalCost])
```

If an extended-amount column already exists, `SUM` of it is equivalent and cheaper than `SUMX`.

## Semi-additive (no SUM across time)

For balances/levels (inventory, account balance, headcount) — sum across non-time dimensions, average or take a period-end value across time. See `understanding-fact-additivity` for classification.

```dax
-- True daily average (divides by distinct day count, not row count)
Avg Daily Inventory =
    AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))

-- Period-end balance (latest non-blank date in current context)
Ending Inventory =
    CALCULATE(
        SUM(Inventory[QtyOnHand]),
        LASTNONBLANK('Date'[Date], CALCULATE(SUM(Inventory[QtyOnHand])))
    )
```

**Never** write `Bad Inventory = SUM(Inventory[QtyOnHand])` and place it on a date axis — it adds the same stock across every day. Time-intelligence variants (`CLOSINGBALANCEMONTH`, `OPENINGBALANCE*`) belong to `applying-time-intelligence`.

## Variables (VAR/RETURN)

Name a subexpression once: clearer, and it's evaluated a single time.

```dax
Sales YoY % =
VAR CurrentSales = [Total Sales]
VAR PriorSales   = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
VAR Change       = CurrentSales - PriorSales
RETURN
    DIVIDE(Change, PriorSales)
```

Variables are evaluated where they're defined and don't change with later context shifts — useful for capturing a value before `CALCULATE` modifies the context.
