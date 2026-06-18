# Common Additivity Errors and Corrections

Each entry: the mistake, how to spot it in a Power BI report, and the fix. All DAX is valid; all visual names are confirmed Power BI visuals.

## Contents
- Error 1: Summing a balance across time
- Error 2: Using AVERAGE() for "average daily" balances
- Error 3: Averaging a stored percentage
- Error 4: Summing unit prices
- Error 5: Balance in a part-to-whole or running-total visual
- Error 6: Storing 0 instead of blank for inapplicable facts
- Error 7: Summing a distinct count across dimensions
- Edge case: percentage facts that ARE additive (extended values)
- Edge case: beginning/ending balance with additive delta

---

## Error 1: Summing a balance across time

**Mistake:** Dropping `Inventory[QtyOnHand]` or `Accounts[Balance]` onto a Line chart or Column chart with a date axis. Power BI applies implicit SUM.

**Symptom:** The value grows steadily over time; a yearly total is roughly 365× the real balance. Mon 50 + Tue 50 reports 100 on hand when it is the same 50 units.

**Fix:** Use an averaging or last-value measure.
```dax
Avg Daily Inventory = AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))
-- or, for the period-end level:
Ending Inventory = CALCULATE(SUM(Inventory[QtyOnHand]), LASTNONBLANK('Date'[Date], CALCULATE(SUM(Inventory[QtyOnHand]))))
```

---

## Error 2: Using AVERAGE() for "average daily" balances

**Mistake:** `Avg Inventory = AVERAGE(Inventory[QtyOnHand])`.

**Symptom:** Result is biased by row count. A category with 100 products and one with 10 products are not comparable, because `AVERAGE` divides by the number of fact rows (products × days), not by the number of days. Total looks plausible but is wrong.

**Fix:** Divide by distinct days, which `AVERAGEX` over the date column does.
```dax
Avg Daily Inventory = AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))
```

---

## Error 3: Averaging a stored percentage

**Mistake:** Storing `GrossMarginPct` per row and showing `AVERAGE(Sales[GrossMarginPct])`, or letting a visual average it.

**Symptom:** A $100 product at 40% and a $1M product at 30% report 35% average margin; the true blended margin is ~30%. Every segment is weighted equally regardless of size.

**Fix:** Ratio of sums — never sum or average of ratios.
```dax
Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))
```
Store the additive components (profit, revenue) in the fact, not the percentage.

---

## Error 4: Summing unit prices

**Mistake:** Putting `UnitPrice` on a Card, Column chart, or Table that aggregates it. `$5 + $3 = $8` is meaningless.

**Symptom:** A "total price" that has no business meaning; a built-in average that ignores volume.

**Fix:** Store `ExtendedAmount = Quantity × UnitPrice` (additive). Compute the volume-weighted price as a ratio of sums.
```dax
Avg Unit Price = DIVIDE(SUM(Sales[ExtendedAmount]), SUM(Sales[Quantity]))
```

---

## Error 5: Balance in a part-to-whole or running-total visual

**Mistake:** Inventory or account balance shown in a Pie chart, Donut chart, Treemap, Stacked area chart, Waterfall chart, or Funnel chart spanning multiple dates.

**Symptom:** Slice sizes / running totals sum the balance across every time period in scope, so the "whole" far exceeds the real balance and all proportions are wrong.

**Fix:** Filter to a single point in time and use a Card, Column chart, or Bar chart; or switch to a flow/additive fact (units sold, net change) for the part-to-whole view.

---

## Error 6: Storing 0 instead of blank for inapplicable facts

**Mistake:** Writing 0 for facts that do not apply to a row (freight on non-freight orders, return qty on non-returns).

**Symptom:** `AVERAGE` is deflated (zeros pull it down), `COUNT` is inflated (counts rows that should not apply), and you cannot distinguish "no value" from "genuinely zero".

**Fix:** Store blank (NULL) for inapplicable facts. `SUM`, `AVERAGE`, and `COUNT` ignore blanks automatically.
```dax
Avg Freight = AVERAGE(Orders[FreightAmount])   -- divides only by orders that actually had freight
```

---

## Error 7: Summing a distinct count across dimensions

**Mistake:** Storing per-store unique customer counts and summing them (Store A 100 + Store B 150 = 250).

**Symptom:** Overcounts customers who shopped at both stores. Distinct counts do not add.

**Fix:** Compute distinct count directly from transaction detail at the needed grain.
```dax
Unique Customers = DISTINCTCOUNT(Sales[CustomerKey])
```

---

## Edge case: percentage facts that ARE additive (extended values)

A percentage that represents an **allocation of a known whole** can be stored as an extended value, which is then additive.

Example: each product's share of a $1M market.
```dax
-- Store extended_market_share_value = total_market_value * share_percent per row
Market Share Value = SUM(Market[ExtendedShareValue])
-- 40% of $1M + 30% + 30% = $400k + $300k + $300k = $1M  (sums correctly)
```
The raw `share_percent` is still non-additive; the **extended value** is additive. This justifies a Treemap or Stacked area on `Market Share Value` (additive) but never on `share_percent`.

---

## Edge case: beginning/ending balance with additive delta

To analyze both level and movement, store the semi-additive balance **and** an additive delta.
```dax
Beginning Balance = CALCULATE(SUM(Ledger[Balance]), FIRSTDATE('Date'[Date]))   -- semi-additive
Ending Balance    = CALCULATE(SUM(Ledger[Balance]), LASTDATE('Date'[Date]))    -- semi-additive
Net Change        = SUM(Ledger[ActivityDelta])                                  -- ADDITIVE, safe in any visual
```
Use `Net Change` (additive) in a Waterfall chart to audit movement; use `Ending Balance` (semi-additive) in a Card or Gauge for the point-in-time level. Relationship: `Ending Balance = Beginning Balance + Net Change`.
