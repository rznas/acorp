# DAX Measure Templates by Additivity Type

Copy-paste templates. Replace `Sales` / `Inventory` / `'Date'` and column names with your model's. Every measure below is syntactically valid DAX.

## Contents
- Additive measures (SUM, COUNT, durations)
- Semi-additive measures (AVERAGEX over time, LASTNONBLANK, CLOSINGBALANCE, OPENINGBALANCE, headcount)
- Non-additive measures (DIVIDE ratios, weighted average price, NPS, distinct count)
- Sparse / optional facts (NULL handling)
- Beginning / ending balance pattern
- Choosing AVERAGEX vs. LASTNONBLANK vs. CLOSINGBALANCE

---

## Additive measures

Sum across every dimension. Implicit aggregation (dragging the column onto a visual) is already correct, but explicit measures are preferred for reuse and naming.

```dax
Total Revenue = SUM(Sales[Revenue])

Total Quantity = SUM(Sales[Quantity])

Total Cost = SUM(Sales[Cost])

Total Profit = SUM(Sales[Profit])               -- or [Total Revenue] - [Total Cost]

-- Always store and sum the EXTENDED amount (qty * rate), never the rate itself
Extended Sales Amount = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])

-- Count of fact rows (transactions)
Transaction Count = COUNTROWS(Sales)

-- Duration totals are additive
Total Talk Time (sec) = SUM(Calls[DurationSeconds])
```

---

## Semi-additive measures

Sum across product / store / etc., but **average or take a single value across time**. Pick the pattern that matches the business question (see decision guide at the bottom).

### Average over the time period (true daily average)

Use this for "average daily inventory", "average balance over the month". `AVERAGEX` over the distinct dates divides by the number of days, not by row count.

```dax
Avg Daily Inventory =
AVERAGEX(
    VALUES('Date'[Date]),
    CALCULATE(SUM(Inventory[QtyOnHand]))
)

Avg Daily Balance =
AVERAGEX(
    VALUES('Date'[Date]),
    CALCULATE(SUM(Accounts[BalanceAmount]))
)
```

### Period-end balance (last non-blank date in context)

Use this for "ending inventory", "closing balance" where you want the value on the last day that actually had data within the current filter.

```dax
Ending Inventory =
CALCULATE(
    SUM(Inventory[QtyOnHand]),
    LASTNONBLANK('Date'[Date], CALCULATE(SUM(Inventory[QtyOnHand])))
)
```

### Closing / opening balance via time intelligence

`CLOSINGBALANCE*` and `OPENINGBALANCE*` return the value at the boundary of the period, ignoring days with no movement. Requires a marked Date table.

```dax
Month End Inventory =
CLOSINGBALANCEMONTH(SUM(Inventory[QtyOnHand]), 'Date'[Date])

Quarter End Balance =
CLOSINGBALANCEQUARTER(SUM(Accounts[BalanceAmount]), 'Date'[Date])

Year End Inventory =
CLOSINGBALANCEYEAR(SUM(Inventory[QtyOnHand]), 'Date'[Date])

Month Start Inventory =
OPENINGBALANCEMONTH(SUM(Inventory[QtyOnHand]), 'Date'[Date])
```

### Headcount (semi-additive)

Headcount sums across departments but averages across time.

```dax
Avg Headcount =
AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(HR[HeadcountSnapshot])))

Current Headcount =
CALCULATE(SUM(HR[HeadcountSnapshot]), LASTDATE('Date'[Date]))
```

---

## Non-additive measures

Never sum the raw value. Store the additive components and compute the **ratio of sums**. `DIVIDE` returns blank (or the optional 3rd argument) on divide-by-zero instead of erroring.

```dax
-- Gross margin % — ratio of summed profit to summed revenue
Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))

-- With an explicit alternate result for zero denominator
Gross Margin % (safe) = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)

-- Weighted-average unit price — ratio of extended amount to quantity
Avg Unit Price = DIVIDE(SUM(Sales[ExtendedAmount]), SUM(Sales[Quantity]))

-- Sell-through rate
Sell-Through % = DIVIDE(SUM(Sales[UnitsSold]), SUM(Inventory[UnitsReceived]))

-- Net Promoter Score from component counts (each count is additive; NPS is not)
NPS =
VAR Promoters = CALCULATE(COUNTROWS(Survey), Survey[Score] >= 9)
VAR Detractors = CALCULATE(COUNTROWS(Survey), Survey[Score] <= 6)
VAR Total = COUNTROWS(Survey)
RETURN DIVIDE(Promoters - Detractors, Total) * 100

-- Distinct count does NOT add across dimensions (overlap); compute it directly
Unique Customers = DISTINCTCOUNT(Sales[CustomerKey])
```

For temperature / scores where an average is genuinely meaningful, `AVERAGE(Readings[TempF])` is acceptable — but confirm the business wants an average rather than min/max/distribution before using it.

---

## Sparse / optional facts (NULL handling)

Facts that do not apply to every row (freight only on some orders, returns only on returns) must be **blank, not zero**, in the source. `SUM`, `AVERAGE`, and `COUNT` ignore blanks, so the math stays correct automatically.

```dax
Total Freight = SUM(Orders[FreightAmount])      -- blanks excluded, total accurate
Avg Freight = AVERAGE(Orders[FreightAmount])     -- divides only by orders that had freight
Orders With Freight = COUNT(Orders[FreightAmount])
```

If zeros were stored instead of blanks, averages are deflated and counts inflated — fix at the source or filter `<> 0`.

---

## Beginning / ending balance pattern

Storing both a semi-additive balance and an additive delta lets you analyze level **and** movement. The delta is fully additive (safe to SUM anywhere).

```dax
Beginning Balance = CALCULATE(SUM(Ledger[Balance]), FIRSTDATE('Date'[Date]))   -- semi-additive
Ending Balance    = CALCULATE(SUM(Ledger[Balance]), LASTDATE('Date'[Date]))    -- semi-additive
Net Change        = SUM(Ledger[ActivityDelta])                                  -- ADDITIVE
```

Relationship: `Ending Balance = Beginning Balance + Net Change`.

---

## Choosing AVERAGEX vs. LASTNONBLANK vs. CLOSINGBALANCE

| Business question | Pattern |
|-------------------|---------|
| "Average level held over the period" (avg daily inventory/balance) | `AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(...)))` |
| "Value on the last day that had data" | `LASTNONBLANK('Date'[Date], ...)` inside CALCULATE |
| "Value at the exact period boundary (month/qtr/year end)" | `CLOSINGBALANCEMONTH / QUARTER / YEAR` |
| "Value at period start" | `OPENINGBALANCEMONTH` or `FIRSTDATE` |
| "Total movement/flow during the period" (additive) | plain `SUM` of the delta column |
