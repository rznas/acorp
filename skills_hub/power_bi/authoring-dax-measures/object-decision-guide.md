# Measure vs Calculated Column vs Calculated Table — Decision Guide

How to choose where a DAX calculation belongs. Wrong-object choices are the most common DAX mistake: they produce static numbers that don't respond to slicers, or bloat the model with stored columns that should have been measures.

## Contents
- Quick decision table
- Decision questions (top-down)
- Measures in detail
- Calculated columns in detail
- Calculated tables in detail
- Storage and performance implications
- Worked decisions

---

## Quick decision table

| Need | Object | Why |
|------|--------|-----|
| A value that re-aggregates as users filter/slice (sum, average, ratio, KPI) | **Measure** | Evaluated at query time in the visual's filter context |
| A per-row value to put on an **axis, legend, or slicer** | **Calculated column** | Stored per row; can group/filter |
| A per-row **key for a relationship** | **Calculated column** | Relationships need stored columns |
| A per-row value used as input to a measure (e.g. a flag) | **Calculated column** | Computed at refresh, available row by row |
| A stored derived **table** (union of two tables, a generated date table) | **Calculated table** | Materialized at refresh |
| Row-level math that must happen before aggregating, but value not needed on an axis | **Measure with an iterator** (e.g. `SUMX`) | Avoids a stored column; iterator supplies row context |

**Fast rule:** if it must respond to visual context → measure. If it must sit on an axis/legend/slicer or be a relationship key → calculated column.

---

## Decision questions (top-down)

```
- [ ] Q1 — Does the result need to change as the user filters/slices the report?
        YES -> Measure (unless Q2/Q3 force a column/table).
- [ ] Q2 — Do you need the value on an axis, in a legend, in a slicer, or as a relationship key?
        YES -> Calculated column (these require a stored, per-row value).
- [ ] Q3 — Do you need a whole derived TABLE stored in the model (union, crossjoin, generated dates)?
        YES -> Calculated table.
- [ ] Q4 — Could the row-level math be done inside a measure with an iterator (SUMX/AVERAGEX)?
        YES -> Prefer the measure + iterator; skip the stored column.
```

---

## Measures in detail

- Calculate a result from a DAX expression **at query time**, in the filter context created by the visual, slicers, the filter pane, and relationships.
- Results change as the user interacts with the report — this is the whole point of a measure.
- Appear in the **Data** pane with a calculator icon; created via **Modeling > New measure** or right-click a table > **New measure**.
- Consume **no row storage** and don't enlarge the model.
- Can be referenced inside other measures (`[Net Sales] / SUM(Sales[SalesQuantity])`), which keeps logic DRY.
- A measure has a **home table** that only controls where it appears in the Data pane; it has no effect on the calculation. Organize many measures with **display folders** (backslash for subfolders, semicolon to appear in multiple folders) or a dedicated measures-only table.
- **Report-level measures** can be authored directly in a report against an existing dataset / live connection when you can't (or don't want to) change the model.

Default to measures. Most sums, averages, counts, ratios, and KPIs should be measures.

## Calculated columns in detail

- Defined by a DAX formula that computes a value **for every row** of a table, **at refresh time**, and **stored** in the model.
- Created in Report/Table/Model view via **New column**; distinct from Power Query custom columns (those are computed during the query, before load).
- Recalculated when the underlying data is refreshed.
- Use when you need a per-row value to **slice, group, sort, put on an axis/legend/slicer, or key a relationship** — things a measure cannot do.
- Example (concatenate fields for a map label):
  ```dax
  CityState = [City] & ", " & [State]
  ```
- Have row context automatically (the "current row"), so they can reference other columns in the row directly.

**Don't** use a calculated column for something that should re-aggregate (e.g. a margin %). A stored per-row margin averaged by a visual gives the wrong total; use a `DIVIDE` measure instead.

## Calculated tables in detail

- Defined by any DAX expression that returns a **table**; **materialized at refresh** and stored in the model.
- Created via **New table** (Table/Model view, Calculations group).
- Best for **intermediate calculations and stored derived data** rather than on-the-fly results.
- Can have relationships, data types, formatting, data categories, and can host measures/columns like any table.
- Common table functions: `DISTINCT`, `VALUES`, `CROSSJOIN`, `UNION`, `NATURALINNERJOIN`, `NATURALLEFTOUTERJOIN`, `INTERSECT`, `CALENDAR`, `CALENDARAUTO`.
- Example (combine two tables):
  ```dax
  Western Region Employees = UNION('Northwest Employees', 'Southwest Employees')
  ```
- A calculated table can be a simple reference to another table:
  ```dax
  New Western Region Employees = 'Western Region Employees'
  ```

**DirectQuery caveat:** calculated tables don't refresh under DirectQuery — they reflect changes only after the semantic model is refreshed. If a table needs DirectQuery, keep the calculated table in DirectQuery too.

---

## Storage and performance implications

| Object | When evaluated | Stored? | Effect on model size | Responds to filters? |
|--------|----------------|---------|----------------------|----------------------|
| Measure | Query time | No | None | Yes |
| Calculated column | Refresh | Yes (per row) | Increases (esp. high-cardinality) | No (it's a stored value; visuals can filter *by* it) |
| Calculated table | Refresh | Yes (whole table) | Increases | No (stored) |

Implications:
- Prefer measures to keep the model lean; high-cardinality calculated columns hurt compression and size.
- If you only need row-level math to feed an aggregate (and not an axis/key), use a **measure with an iterator** (`SUMX`) instead of a stored calculated column.
- Calculated columns and tables recompute on every refresh; heavy ones lengthen refresh.

---

## Worked decisions

**1. "Show net sales that updates when I slice by year/region."**
→ **Measure.** It must re-aggregate per filter context: `Net Sales = SUM(Sales[SalesAmount]) - SUM(Sales[DiscountAmount]) - SUM(Sales[ReturnAmount])`.

**2. "Group products by a City, State label on a map."**
→ **Calculated column** (`CityState = [City] & ", " & [State]`) because the label sits on the visual as a field, not as an aggregated value.

**3. "Profit per row to feed a ratio."**
→ Don't store a per-row margin. Keep additive components (`Profit`, `SalesAmount`) and write a **measure**: `Gross Margin % = DIVIDE(SUM(Sales[Profit]), SUM(Sales[SalesAmount]))`.

**4. "One employee table combining two regional tables."**
→ **Calculated table** with `UNION`.

**5. "Revenue = quantity × unit price, only need the total."**
→ **Measure with an iterator**: `Total Revenue = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])` — no stored column needed.
