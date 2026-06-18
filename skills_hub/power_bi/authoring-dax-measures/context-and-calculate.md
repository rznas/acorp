# Context, Iterators, and CALCULATE

The mechanics behind every measure: the two evaluation contexts, iterators that create row context, and `CALCULATE` which modifies filter context. Grounded in `desktop-quickstart-learn-dax-basics.md`.

## Contents
- Row context vs filter context
- How filter context gets applied
- Iterators (functions ending in X)
- CALCULATE and filter modifiers
- Context transition (measures inside iterators)
- USERELATIONSHIP and inactive relationships
- Worked examples

---

## Row context vs filter context

DAX has exactly two contexts. Understanding both is the foundation for correct measures.

| Context | What it is | Where it exists | Example |
|---------|-----------|-----------------|---------|
| **Row context** | "The current row" | Automatically in a calculated column, and inside any iterator | In `Price * Qty` as a calculated column, both columns resolve to the current row |
| **Filter context** | One or more filters that determine the value of a calculation | Created by the visual, slicers, filter pane, relationships, and `CALCULATE` filters | A matrix cell for "USA / 2024" filters the measure to that country and year |

Key facts:
- Filter context does **not** replace row context — it applies **in addition**.
- A bare measure has **no row context**. It only has the filter context surrounding it. To get row-by-row behavior inside a measure, use an **iterator**.
- A DAX function "always references a complete column or a table"; to use particular values you add filters.

## How filter context gets applied

Filter context is set by:
1. **Adding fields to a visualization** — putting `Year` and `Region` on a chart filters the measure to a year/region subset.
2. **DAX functions** — `CALCULATE`, `FILTER`, `ALL`, `RELATED`, etc.
3. **Relationships** — a filter on a dimension flows across the relationship to the fact table.
4. **Other measures and columns** referenced in the formula.

Example from the docs — restrict `[Total Sales]` to the "Store" channel:
```dax
Store Sales = CALCULATE([Total Sales], Channel[ChannelName] = "Store")
```
Here `Channel[ChannelName]` supplies the row context for the filter, and the value `"Store"` is the filter context applied to the calculation.

## Iterators (functions ending in X)

Iterators evaluate an expression **once per row** of a table (creating row context), then aggregate the per-row results. Use them when row-level math must happen **before** aggregation.

| Iterator | Aggregates by |
|----------|---------------|
| `SUMX` | Sum |
| `AVERAGEX` | Average |
| `MINX` / `MAXX` | Min / Max |
| `COUNTX` | Count |
| `RANKX` | Rank |

```dax
-- Per-row qty * price, THEN sum
Total Revenue = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])

-- True daily average of a semi-additive balance: iterate distinct dates
Avg Daily Inventory =
    AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))
```

**When NOT to use an iterator:** if the per-row product already exists as a stored extended-amount column, `SUM(that column)` is equivalent and usually cheaper than re-deriving it with `SUMX`.

## CALCULATE and filter modifiers

`CALCULATE(<expression>, <filter1>, <filter2>, ...)` evaluates the expression in a filter context **modified** by the filter arguments. It is the most powerful function in DAX (and the most misused). It has at least two arguments: the expression to evaluate, then one or more filters.

Common filter modifiers:

| Modifier | Effect |
|----------|--------|
| `ALL(table or column)` | Removes filters from the table/column (often for "% of total" denominators or "all years" totals) |
| `ALLEXCEPT(table, col1, ...)` | Removes all filters on the table except the listed columns |
| `ALLSELECTED()` | Respects outer slicer selection but ignores filters inside the visual |
| `REMOVEFILTERS(...)` | Clears filters (clearer synonym for `ALL` used as a modifier) |
| `KEEPFILTERS(<filter>)` | Adds a filter without overwriting an existing same-column filter |
| `FILTER(table, condition)` | Returns a filtered table to use as a CALCULATE argument |
| `USERELATIONSHIP(col1, col2)` | Activates an inactive relationship for this calculation |

Examples:
```dax
-- Denominator that ignores the category filter -> enables % of total
Pct of Total Sales =
    DIVIDE([Total Sales], CALCULATE([Total Sales], ALL(Product[Category])))

-- Total across all years regardless of the Year filter
Sales All Years = CALCULATE([Total Sales], ALL('Date'[Year]))
```

A formula can nest up to 64 functions (you'll rarely approach that). `PREVIOUSQUARTER`, `SAMEPERIODLASTYEAR`, etc. are time-intelligence functions commonly nested inside `CALCULATE` — but route detailed time-intelligence work to `applying-time-intelligence`. Example from the DAX-basics doc:
```dax
Previous Quarter Sales =
    CALCULATE(SUM(Sales[SalesAmount]), PREVIOUSQUARTER('Calendar'[DateKey]))
```

## Context transition (measures inside iterators)

When you reference a **measure** (or wrap an expression in `CALCULATE`) inside an iterator, DAX performs **context transition**: the current row context becomes a filter context for that evaluation. This is why the semi-additive average works:
```dax
Avg Daily Inventory = AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))
```
For each distinct date (row context), `CALCULATE` transitions that date into a filter, so `SUM(QtyOnHand)` returns that day's balance; `AVERAGEX` then averages over the days. Without the transition (e.g. plain `SUM` over a date axis) you would add the same stock across every day.

## USERELATIONSHIP and inactive relationships

A model can hold multiple relationships between two tables but only one **active** at a time (e.g. order date vs ship date both pointing at the date table — role-playing dimensions). Activate an inactive one for a single calculation:
```dax
Sales by Ship Date =
    CALCULATE([Total Sales], USERELATIONSHIP(Sales[ShipDateKey], 'Date'[DateKey]))
```
For relationship/cardinality/cross-filter design itself, see `modeling-semantic-data`.

## Worked examples

**1. Per-row math before aggregation** → iterator.
`Weighted Revenue = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])`

**2. % of total** → `CALCULATE` + `ALL` for the denominator.
`Pct = DIVIDE([Total Sales], CALCULATE([Total Sales], ALLSELECTED(Product[Category])))`

**3. Restrict to a value regardless of visual filters** → `CALCULATE` with a filter.
`Online Sales = CALCULATE([Total Sales], Channel[ChannelName] = "Online")`

**4. Use the ship-date relationship** → `USERELATIONSHIP`.
`Shipped Sales = CALCULATE([Total Sales], USERELATIONSHIP(Sales[ShipDateKey], 'Date'[DateKey]))`
