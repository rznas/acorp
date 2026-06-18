# Additivity Validation Checklist per Chart Category

Run this validation loop before finalizing any visual: classify the fact's additivity, then confirm the visual's aggregation behavior is safe for that class. If a check fails, fix the DAX measure or change the visual, then re-validate.

## Contents
- Quick classification
- Validation checklist by chart category
- Safe / caution / avoid matrix (additivity × visual)
- DAX correction templates
- Common errors and fixes

---

## Quick classification

Classify each fact before placing it on a visual:

- **Additive** — sums correctly across every dimension including time (extended sales amount, quantity, cost, profit).
- **Semi-additive** — sums across non-time dimensions but **not** time (account balance, inventory on hand, headcount, occupied beds).
- **Non-additive** — cannot be summed across any dimension (unit price, ratio, percentage, rate). Store numerator and denominator separately.

---

## Validation checklist by chart category

### Aggregating charts (column, bar, line, area, pie, donut, treemap, ribbon, waterfall)
These visuals apply an aggregation (default `SUM`) over the grouping dimension.

```
- [ ] Fact is additive? If yes, SUM is safe in all of these visuals.
- [ ] Fact is semi-additive? Time on the axis/legend => DO NOT use default SUM.
      Use AVERAGE or a last-balance measure; otherwise restrict to a non-time axis.
- [ ] Fact is non-additive? Replace with a measure: DIVIDE(SUM(num), SUM(den)).
      Never let the visual SUM or AVG a stored ratio column.
- [ ] Part-to-whole visual (pie/donut/treemap)? Confirm fact is additive AND
      shares one consistent total (no semi-additive balances, no ratios).
```

### Level / goal visuals (card, multirow card, KPI, gauge)
These show one value (often a single period or a goal comparison).

```
- [ ] Semi-additive balance? Card/KPI/gauge are the preferred home; show ONE period
      or a last-balance measure, not a SUM across time.
- [ ] Non-additive ratio? Card is fine when the measure is DIVIDE(SUM(num),SUM(den)).
- [ ] Gauge target/min/max set from a measure or constant, not a summed balance.
```

### Detail visuals (table, matrix)
Tables can show raw values; matrices auto-aggregate.

```
- [ ] Matrix totals row present? Confirm the measure aggregates correctly for the
      fact's additivity (semi-additive balances must NOT sum down a Date hierarchy).
- [ ] Showing raw non-additive values (unit price) without a total? Safe in a table.
- [ ] A matrix subtotal over time on a balance is wrong — switch the measure to
      AVERAGE / last-balance, or remove the time subtotal.
```

### Distribution / relationship visuals (scatter, bubble, dot plot, high-density scatter)
Plot points or measures, not sums over a category.

```
- [ ] Plotting non-additive facts as points (e.g., unit price vs volume)? Safe —
      points are not summed.
- [ ] Bubble size uses an additive measure (SUM)? Confirm size is meaningful.
- [ ] Axes use measures that are valid for the fact's additivity.
```

### Pipeline visuals (funnel, waterfall)
```
- [ ] Funnel stage values from additive counts / SUM of 0/1 milestone flags? Safe.
- [ ] Waterfall increments are additive amounts (not balances or ratios)? Required.
```

---

## Safe / caution / avoid matrix (additivity × visual)

| Visual category | Additive | Semi-additive | Non-additive |
|---|---|---|---|
| Column / bar (non-time axis) | Safe (`SUM`) | Safe (`SUM` across non-time) | Avoid raw; use `DIVIDE` measure |
| Line / area / column (time axis) | Safe (`SUM`) | Caution — use `AVERAGE`/last balance, NOT `SUM` | Avoid raw; use `DIVIDE` measure |
| Pie / donut / treemap | Safe (`SUM`) | Avoid (balances do not form a clean whole) | Avoid |
| Card / KPI / gauge | Safe | Safe (single period / last balance) | Safe (ratio-of-sums measure) |
| Table | Safe | Safe (label the period) | Safe (raw values) |
| Matrix (with totals) | Safe | Caution (no time subtotal sums) | Caution (suppress meaningless totals) |
| Scatter / bubble / dot plot | Safe (size/axis = `SUM`) | Safe (points) | Safe (points) |
| Funnel / waterfall | Safe | Avoid (balances) | Avoid |

---

## DAX correction templates

All templates are syntactically valid DAX.

```dax
-- Additive: default and safe everywhere
Total Revenue = SUM ( Sales[ExtendedSalesAmount] )

-- Semi-additive: average across time
Avg Daily Inventory = AVERAGE ( Inventory[QuantityOnHand] )

-- Semi-additive: ending (last non-blank) balance in the period
Ending Balance =
CALCULATE (
    SUM ( AccountSnapshot[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( AccountSnapshot[Balance] ) ) )
)

-- Non-additive: ratio of sums (never AVERAGE of a stored ratio)
Gross Margin % = DIVIDE ( SUM ( Sales[GrossProfit] ), SUM ( Sales[ExtendedSalesAmount] ), 0 )

-- Time-correct average of a semi-additive balance (avoid SQL-style AVG over rows)
Avg Balance Over Period =
DIVIDE (
    SUM ( AccountSnapshot[Balance] ),
    DISTINCTCOUNT ( 'Date'[Date] )
)
```

---

## Common errors and fixes

| Symptom | Cause | Fix |
|---|---|---|
| Inventory total balloons when months are added to a line chart | `SUM` of a semi-additive balance across Date | Switch to `AVERAGE` or `Ending Balance` measure |
| Margin % total looks wrong / averaged | Aggregating a stored ratio column | Use `DIVIDE(SUM(num), SUM(den))` |
| Report author accidentally sums a balance | Implicit measure defaults to `SUM` | Hide the column; expose explicit semi-additive measures |
| Pie chart of inventory across stores "doesn't add up" | Balance is not a clean part-to-whole | Use matrix or card; reserve pie/donut for additive facts |
| Funnel stage counts overcount | Counting rows that repeat across milestones | `SUM` the 0/1 milestone flag, or count distinct pipeline items |
