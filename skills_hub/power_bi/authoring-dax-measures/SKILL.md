---
name: authoring-dax-measures
description: Writes correct DAX in Power BI Desktop — choosing measures vs calculated columns vs calculated tables, reasoning about row context vs filter context, using CALCULATE with filter modifiers, iterators (SUMX/AVERAGEX), safe ratios with DIVIDE, VAR/RETURN variables, quick measures, and calculation groups. Use when authoring or debugging DAX measures, calculated columns, or calculated tables; when a total is inflated or a ratio is wrong; when deciding whether a calculation belongs in a measure or a column; or when building running totals, %-of-total, year-over-year, rankings, or safe ratios. Enforces additivity-correct aggregation so semi-additive balances are never naive-SUMmed across time.
---

# Authoring DAX Measures

## Overview

DAX (Data Analysis Expressions) is the formula language for calculations in a Power BI semantic model. The two failure modes this skill prevents are: (1) putting a calculation in the wrong object (column where a measure belongs, or vice-versa), and (2) writing a measure that aggregates incorrectly — most dangerously, naive-`SUM` of a semi-additive balance across time. Both produce numbers that look plausible but are wrong.

**Prerequisites (from the full-lifecycle plan):** This is the Calculation-stage skill (Skill 14). It assumes a built model — relationships, star schema, a marked date table. If the model isn't built, do `modeling-semantic-data` first. For period-over-period / YTD / semi-additive period-end logic, hand off to `applying-time-intelligence` (Skill 15). For *which* aggregation a fact needs, this skill defers to `understanding-fact-additivity` (Skill 3).

## When to Use

- Writing or fixing a measure, calculated column, or calculated table.
- A total looks inflated, a percentage is being averaged, or a balance grows every day.
- Deciding where a calculation belongs (measure vs column vs table).
- Building running totals, %-of-total, YoY change, rankings, or safe ratios.
- Reusing time/format logic across many measures (calculation groups).

## Authoring workflow

Copy this checklist and work top-down:

```
DAX authoring:
- [ ] Step 1 — Decide the object: measure, calculated column, or calculated table?
- [ ] Step 2 — Classify the fact's additivity (additive / semi-additive / non-additive).
- [ ] Step 3 — Pick the aggregation that matches additivity (SUM, AVERAGEX-over-time, DIVIDE, ...).
- [ ] Step 4 — Decide if you need an iterator (row-by-row math before aggregating) or a context change (CALCULATE).
- [ ] Step 5 — Write the measure: fully-qualify columns, use VAR for repeated/complex subexpressions, DIVIDE for ratios.
- [ ] Step 6 — Validate: check the grand total and at least one subtotal; if a balance, confirm it is not summed across dates.
```

## Step 1 — Measure vs calculated column vs calculated table

This is the most common mistake. Use the decision guide below; full rationale and storage/performance tradeoffs are in [object-decision-guide.md](object-decision-guide.md).

| Object | Evaluated | Use when | Default to |
|--------|-----------|----------|------------|
| **Measure** | At query time, in the **filter context** of the visual | You need a value that re-aggregates as users filter/slice (sums, averages, ratios, KPIs) | **Yes — prefer measures** |
| **Calculated column** | Row by row at **refresh**, stored in the model | You need a per-row value for slicing, grouping, an axis, a relationship key, or as input to a measure | Only when a measure can't do the job |
| **Calculated table** | At **refresh**, materialized as a table | Intermediate/derived tables stored in the model (e.g. `UNION` of two tables, a date table via `CALENDAR`) | Rarely |

**Fast rule:** if the result must respond to slicers and visual context, it is a **measure**. If you need to put the value on an axis/legend/slicer or use it as a relationship key, it is a **calculated column**. Measures consume no storage and don't enlarge the model; calculated columns and tables are stored and increase model size.

> Calculated tables don't refresh under DirectQuery; they reflect changes only after the semantic model is refreshed (and if a table needs DirectQuery, keep the calculated table in DirectQuery too).

## Step 2-3 — Additivity-correct aggregation (CRITICAL)

Before choosing an aggregation, classify the fact. **Defer to `understanding-fact-additivity` for the full classification** — the core rule:

- **Additive** (sales qty, revenue, cost, counts, durations) → `SUM` is correct across all dimensions.
- **Semi-additive** (inventory on hand, account balance, headcount, backlog) → sum across product/store/etc., but **never naive-`SUM` across time**. Average over the distinct dates or take a period-end value.
- **Non-additive** (price, %, margin, rate) → never sum; compute the **ratio of sums** with `DIVIDE`.

```dax
-- ADDITIVE: SUM across every dimension is correct
Total Sales = SUM(Sales[SalesAmount])

-- SEMI-ADDITIVE: average over distinct days (divides by day count, not row count)
Avg Daily Inventory =
    AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))

-- SEMI-ADDITIVE: period-end balance (latest non-blank date in context)
Ending Inventory =
    CALCULATE(
        SUM(Inventory[QtyOnHand]),
        LASTNONBLANK('Date'[Date], CALCULATE(SUM(Inventory[QtyOnHand])))
    )

-- NON-ADDITIVE: ratio of sums, NOT average of per-row ratios
Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[SalesAmount]))
```

**Never** drop a raw balance column (e.g. `QtyOnHand`) onto a visual or wrap it in `SUM` over a date axis — the implicit/explicit SUM adds the same stock across every day and inflates the total to roughly N× reality. This is the single most important rule when crossing from a Kimball periodic-snapshot fact table (DW `fact-table-design-patterns`) into a Power BI measure: semi-additive snapshot facts must use the average-over-time or period-end patterns above. Time-intelligence variants (`CLOSINGBALANCE*`, `OPENINGBALANCE*`) belong to `applying-time-intelligence`.

## Step 4 — Context: row vs filter, iterators, and CALCULATE

There are two contexts in DAX (see `desktop-quickstart-learn-dax-basics.md`):

- **Row context** = "the current row." It exists automatically in a calculated column and inside any iterator. It does *not* exist by default inside a measure.
- **Filter context** = the set of filters applied by the visual, slicers, the filter pane, and relationships. A measure evaluates inside whatever filter context surrounds it.

**Iterators** (`SUMX`, `AVERAGEX`, `MINX`, `MAXX`, `RANKX`, ...) create row context: they evaluate an expression once per row of a table, then aggregate. Use an iterator when the row-level math must happen *before* aggregating:

```dax
-- Need per-row qty * price BEFORE summing -> iterator
Total Revenue = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])
```

If `Sales[Quantity] * Sales[UnitPrice]` already exists as an extended-amount column, `SUM` of that column is equivalent and usually cheaper than re-deriving it.

**CALCULATE** evaluates an expression in a *modified* filter context. Its first argument is the expression; the rest are filters that add to or override the current context:

```dax
-- Sales restricted to one channel, regardless of what the visual filters
Store Sales = CALCULATE([Total Sales], Channel[ChannelName] = "Store")

-- Remove the Year filter so the total spans all years (filter modifier ALL)
Sales All Years = CALCULATE([Total Sales], ALL('Date'[Year]))
```

`CALCULATE` is the most powerful — and most misused — DAX function. Common filter modifiers: `ALL`, `ALLEXCEPT`, `ALLSELECTED`, `KEEPFILTERS`, `REMOVEFILTERS`, `FILTER`, `USERELATIONSHIP`. Full context rules and modifier behavior: [context-and-calculate.md](context-and-calculate.md).

## Step 5 — Variables, DIVIDE, and clean syntax

```dax
-- VAR/RETURN: name subexpressions, evaluate once, improve readability and speed
Sales YoY % =
VAR CurrentSales = [Total Sales]
VAR PriorSales   = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN
    DIVIDE(CurrentSales - PriorSales, PriorSales)
```

- **Always use `DIVIDE(num, den)` instead of `/`** for ratios — it returns blank (not an error) on divide-by-zero. Pass a third argument for an alternate result, e.g. `DIVIDE(a, b, 0)`.
- **Fully qualify** column references as `Table[Column]`; reference measures bare as `[Measure]`. Quote table names with spaces in single quotes: `'Reseller Sales'`.
- Reuse measures inside other measures (e.g. `[Net Sales] / SUM(Sales[SalesQuantity])`) rather than re-deriving aggregations.

## Quick measures — let Power BI write starter DAX

For common calculations you don't have to write DAX by hand. Right-click a field (or the value well of a visual) → **New quick measure**, pick a calculation, and Power BI generates the DAX and shows it in the formula bar — a good way to seed and learn patterns. Confirmed quick-measure categories include **Time intelligence** (year-to-date, year-over-year change, rolling average), **Totals** (running total, total for category with/without filters applied), **Filters** (difference / percentage difference from filtered value), **Aggregate per category** (average/min/max/weighted-average per category), and **Mathematical/Text** (division, percentage difference, star rating). Limitations: time-intelligence quick measures can't be created in DirectQuery mode, and quick-measure DAX uses commas as argument separators.

## Calculation groups — reuse logic across many measures

A calculation group lets you define DAX once (as **calculation items**) and apply it to *any* measure, eliminating dozens of near-duplicate measures (e.g. one set of time-intelligence variants instead of YTD/PY/YoY% per measure). Create one in **Model view** → **Calculation group** button. `SELECTEDMEASURE()` is the placeholder for whichever measure the item is applied to:

```dax
calculationItem 'YTD' = TOTALYTD(SELECTEDMEASURE(), 'Date'[Date])
```

Caveats from the docs:
- Adding a calculation group enables **Discourage implicit measures** (you must use explicit measures; dragging a raw column to a visual as a value is blocked). Calculation items only apply to explicit measures, not implicit ones.
- All model measures become the **variant** data type while any calculation group exists; this can break dynamic format strings (fix with `FORMAT([…], "")`).
- Guard math items against non-numeric measures with `ISNUMERIC(SELECTEDMEASURE())`.
- Multiple calculation groups apply in **precedence** order.

Reference: [calculation-groups-guide.md](calculation-groups-guide.md). For the time-intelligence items themselves, use `applying-time-intelligence`.

## Pattern library and error corrections

- **Measure pattern library** — running total, %-of-total, YoY, ranking, safe ratios, iterators: [measure-pattern-library.md](measure-pattern-library.md).
- **Common DAX-error corrections** — wrong object, summed balance, averaged percentage, missing context transition, `/` vs `DIVIDE`: [common-dax-errors.md](common-dax-errors.md).

## Edge case — value filter behavior (preview)

When multiple columns from the **same table** are filtered, DAX may combine them and drop non-existing combinations ("auto-exist"), which can make an across-all measure return a surprising total. The **Value filter behavior** model setting (model view → properties) has **Automatic / Independent / Coalesced** options to control this. Flag this only when a same-table multi-column filter yields an unexpected total; otherwise leave the default. This is a **preview** feature — verify the current behavior in the docs before relying on it.

## Success criteria

- The calculation lives in the right object (measure unless slicing/keying requires a column).
- Aggregation matches additivity; no semi-additive balance is SUMmed across time.
- Ratios use `DIVIDE` (ratio of sums), not `/` and not average-of-ratios.
- Iterators are used only when per-row math must precede aggregation.
- `CALCULATE` filters are intentional; `ALL`/`REMOVEFILTERS` used deliberately for totals.
- Grand total and at least one subtotal verified to be correct.

---
*Provenance: built per SKILL 14 "Authoring DAX Measures" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI transform-model docs (`desktop-measures.md`, `desktop-calculated-columns.md`, `desktop-calculated-tables.md`, `desktop-quickstart-learn-dax-basics.md`, `desktop-tutorial-create-measures.md`, `desktop-quick-measures.md`, `calculation-groups.md`, `value-filter-behavior.md`); cross-references Power BI skills `understanding-fact-additivity`, `modeling-semantic-data`, `applying-time-intelligence`, and Kimball `fact-table-design-patterns`.*
