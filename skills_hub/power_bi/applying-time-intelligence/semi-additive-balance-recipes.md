# Semi-Additive Period-End Balance Recipes

Balances (inventory on hand, account balance, headcount, backlog) are **semi-additive**: additive across product/store/department, but **not across time**. Naive-SUMming a balance over dates double-counts the same level every day and inflates totals (a year total ≈ 365× reality). Use these period-end / average patterns instead.

Replace `Inventory`, `[QtyOnHand]`, and `'Date'` with your model's names. `'Date'` is a marked, contiguous date table (see [date-table-prerequisites.md](date-table-prerequisites.md)). Every snippet is valid DAX.

## Contents
- The rule (never naive-SUM across time)
- Closing balance (CLOSINGBALANCEMONTH/QUARTER/YEAR)
- Opening balance (OPENINGBALANCE*)
- Last value via LASTDATE
- Last value on a day that had data via LASTNONBLANK
- Average level over the period (AVERAGEX over dates)
- Beginning / ending balance + additive net change
- Headcount example
- Choosing the right pattern (decision table)
- Cross-references

---

## The rule

Determine additivity first with `understanding-fact-additivity`. If a fact is a **balance/level captured at a point in time**, it is semi-additive:

- **Across product/store/account:** SUM is correct.
- **Across time (dates):** SUM is WRONG. Take the period-end value, the last value, or the average over days.

These balances correspond to **periodic snapshot** fact tables in Kimball `fact-table-design-patterns` (`data_warehouse_kimball/SKILL/fact-table-design-patterns/SKILL.md`): grain "one row per product per store per day", facts like `quantity_on_hand` are semi-additive.

---

## Closing balance — value at the exact period boundary

`CLOSINGBALANCEMONTH/QUARTER/YEAR(expr, dates[, filter])` returns the value as of the **last date** of the month/quarter/year in context, ignoring days with no movement. Requires a marked date table.

```dax
Month End Inventory   = CLOSINGBALANCEMONTH   ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )
Quarter End Inventory = CLOSINGBALANCEQUARTER ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )
Year End Inventory    = CLOSINGBALANCEYEAR    ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )

-- Account balances
Quarter End Cash = CLOSINGBALANCEQUARTER ( SUM ( Accounts[BalanceAmount] ), 'Date'[Date] )
```

Use closing balance when the report should show the balance "as of period end" regardless of which day in the period the last movement happened.

---

## Opening balance — value at the period start

```dax
Month Start Inventory   = OPENINGBALANCEMONTH   ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )
Quarter Start Inventory = OPENINGBALANCEQUARTER ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )
Year Start Inventory    = OPENINGBALANCEYEAR    ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )
```

---

## Last value via LASTDATE

`LASTDATE` returns the last date in the current filter context. Use it for "value on the last day of whatever period is selected" when every day has data.

```dax
Ending Balance = CALCULATE ( SUM ( Accounts[BalanceAmount] ), LASTDATE ( 'Date'[Date] ) )
```

`LASTDATE` takes the chronologically last date in context even if that day has no fact rows (giving blank). If your snapshot skips days (weekends, holidays), prefer `LASTNONBLANK`.

---

## Last value on a day that actually had data via LASTNONBLANK

`LASTNONBLANK(column, expression)` returns the last value in `column` for which `expression` is not blank — i.e., the most recent day that had a balance. Robust to sparse snapshots.

```dax
Ending Inventory =
CALCULATE (
    SUM ( Inventory[QtyOnHand] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( Inventory[QtyOnHand] ) ) )
)
```

There is a matching `FIRSTNONBLANK` for the first day with data.

---

## Average level over the period

For "average daily inventory" / "average balance over the month", average across **distinct dates** with `AVERAGEX` over `VALUES('Date'[Date])`. This divides by the number of days, not by the row count (products × days).

```dax
Avg Daily Inventory =
AVERAGEX ( VALUES ( 'Date'[Date] ), CALCULATE ( SUM ( Inventory[QtyOnHand] ) ) )

Avg Daily Balance =
AVERAGEX ( VALUES ( 'Date'[Date] ), CALCULATE ( SUM ( Accounts[BalanceAmount] ) ) )
```

Do **not** use `AVERAGE(Inventory[QtyOnHand])` — it divides by fact-row count and is skewed by how many products exist per day.

---

## Beginning / ending balance + additive net change

Store the balance (semi-additive) and the movement delta (fully additive) so you can analyze level **and** flow. The delta is safe to SUM anywhere.

```dax
Beginning Balance = CALCULATE ( SUM ( Ledger[Balance] ), FIRSTDATE ( 'Date'[Date] ) )  -- semi-additive
Ending Balance    = CALCULATE ( SUM ( Ledger[Balance] ), LASTDATE  ( 'Date'[Date] ) )  -- semi-additive
Net Change        = SUM ( Ledger[ActivityDelta] )                                       -- ADDITIVE
```

Relationship: `Ending Balance = Beginning Balance + Net Change`.

---

## Headcount example

Headcount sums across departments but averages (or takes last) across time.

```dax
Avg Headcount     = AVERAGEX ( VALUES ( 'Date'[Date] ), CALCULATE ( SUM ( HR[HeadcountSnapshot] ) ) )
Current Headcount = CALCULATE ( SUM ( HR[HeadcountSnapshot] ), LASTDATE ( 'Date'[Date] ) )
```

---

## Choosing the right pattern

| Business question | Pattern |
|-------------------|---------|
| "Balance as of period end (month/qtr/year)" | `CLOSINGBALANCEMONTH / QUARTER / YEAR` |
| "Balance at period start" | `OPENINGBALANCEMONTH / QUARTER / YEAR` or `FIRSTDATE` |
| "Value on the last selected day (every day has data)" | `LASTDATE` inside CALCULATE |
| "Value on the last day that actually had data (sparse)" | `LASTNONBLANK` inside CALCULATE |
| "Average level held over the period" | `AVERAGEX ( VALUES ( 'Date'[Date] ), CALCULATE ( SUM (...) ) )` |
| "Total movement/flow during the period" (additive) | plain `SUM` of the delta column |

Validation: drop the balance measure into a Matrix with Year > Quarter > Month. The year total must equal a single period-end value (or the true average) — **not** the sum of monthly balances. If the total grows with the number of months, you used a naive SUM.

---

## Cross-references

- `understanding-fact-additivity` — classify additive / semi-additive / non-additive before choosing a measure.
- `authoring-dax-measures` — CALCULATE, VAR/RETURN, DIVIDE, iterators.
- `modeling-semantic-data` — relationships and marking a date table.
- `financial-services-visualization` and `supply-chain-visualization` — visualizing balances (cash/account balances; inventory/backlog) safely.
- Kimball `fact-table-design-patterns` — periodic snapshot fact tables and the semi-additive fact rule.
