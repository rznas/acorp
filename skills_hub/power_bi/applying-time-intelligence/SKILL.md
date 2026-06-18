---
name: applying-time-intelligence
description: Builds time-based DAX in Power BI — YTD/QTD/MTD totals, period-over-period, same-period-last-year, moving averages, and semi-additive period-end balances (closing/opening) — on top of a proper date table. Use when writing year-to-date, quarter/month-to-date, year-over-year, prior-period, rolling/moving-average, or month-end/quarter-end/year-end balance calculations, when comparing a period to the previous period or the same period last year, or when deciding between Auto date/time and a marked or calendar date table for time intelligence.
---

# Applying Time Intelligence

## Overview

Power BI time intelligence functions (TOTALYTD, SAMEPERIODLASTYEAR, DATEADD, PARALLELPERIOD, and the CLOSINGBALANCE/OPENINGBALANCE family) let you compute YTD/QTD/MTD, period-over-period, same-period-last-year, moving averages, and period-end balances. They only work correctly on top of a proper **date table** — a single dedicated table with one contiguous row per date that propagates filters to every fact table. This skill prescribes the date-table prerequisites first, then the calculation patterns, then the semi-additive balance recipes.

**Prerequisites (from the full-lifecycle plan):** Depends on `authoring-dax-measures` (filter/row context, CALCULATE, VAR/RETURN, DIVIDE) and `modeling-semantic-data` (relationships, marking a date table). It is a Calculation-stage skill that builds on, and must respect, fact additivity — see `understanding-fact-additivity` before writing any balance measure.

## When to Use

- Year-to-date, quarter-to-date, month-to-date, or week-to-date running totals.
- Period-over-period: vs. previous day/month/quarter/year, or vs. same period last year.
- Moving / rolling averages (e.g., trailing 3-month or 30-day average).
- Semi-additive **period-end** balances: month-end inventory, quarter-end cash, year-end headcount.
- Choosing between Auto date/time, a marked date table, and calendar-based time intelligence.

## Step 0 — Verify date-table prerequisites FIRST

Time intelligence silently misbehaves without a valid date table. Before writing any measure, confirm the prerequisites. A model date table must have a date (or date/time) column with **unique, non-blank, contiguous values that span full years** (a year need not be Jan–Dec). For classic time intelligence the table must also be **Mark as date table**.

Copy this checklist:

```
Date-table readiness:
- [ ] One dedicated date table exists (not Auto date/time) for cross-fact filtering
- [ ] Date column is data type Date or Date/Time
- [ ] Date column has unique, non-blank, contiguous values (no gaps), spanning full years
- [ ] Table is "Mark as date table" (required for classic time intelligence functions)
- [ ] Active relationship from each fact's date FK to the date table
- [ ] Auto date/time is OFF when using a dedicated date table
```

**Why a marked date table beats Auto date/time:** Auto date/time creates a hidden table per date column, which (1) bloats model size and slows refresh, (2) cannot propagate one time filter to multiple fact tables (sales + budget), and (3) is unavailable in Analyze in Excel and non-Power BI designers. A single dedicated date table fixes all three. Full rationale and the three tools (Auto date/time vs. classic vs. calendar-based) are in [date-table-prerequisites.md](date-table-prerequisites.md).

## Step 1 — Generate a date table (if one does not exist)

Prefer connecting to an existing date dimension from a data warehouse (single source of truth). If none exists, generate one. Build it in Power Query M or DAX. DAX is convenient because `CALENDARAUTO` auto-extends on refresh.

```dax
-- DAX calculated table: fixed range
DateTable =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2010, 1, 1 ), DATE ( 2030, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month Name", FORMAT ( [Date], "MMMM" ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    "Day", DAY ( [Date] )
)
```

Use `CALENDAR(start, end)` for a fixed range, or `CALENDARAUTO()` to auto-span all model dates and extend on each refresh. The full M and DAX recipes are in [date-table-prerequisites.md](date-table-prerequisites.md).

## Step 2 — Pick the calculation pattern

| Need | Function family |
|------|-----------------|
| Running total within year/quarter/month | `TOTALYTD` / `TOTALQTD` / `TOTALMTD` (or `DATESYTD`/`DATESQTD`/`DATESMTD` inside CALCULATE) |
| Same period last year (lateral shift) | `SAMEPERIODLASTYEAR` |
| Shift by N days/months/quarters/years | `DATEADD` |
| Full prior parallel period (full year/quarter even when partial in context) | `PARALLELPERIOD` |
| Prior single day/month/quarter/year | `PREVIOUSDAY` / `PREVIOUSMONTH` / `PREVIOUSQUARTER` / `PREVIOUSYEAR` |
| Moving / rolling average | `AVERAGEX` over `DATESINPERIOD` |
| Period-end (semi-additive) balance | `CLOSINGBALANCEMONTH/QUARTER/YEAR`, `LASTDATE`, `LASTNONBLANK` |

> **Note:** There is no `TOTALWTD` in classic time intelligence — week-based (WTD) running totals are NOT available. Do week-to-date with calendar-based time intelligence instead (see [time-intelligence-patterns.md](time-intelligence-patterns.md)).

Quick examples (classic time intelligence; `'Date'` is the marked date table):

```dax
Sales YTD       = TOTALYTD ( SUM ( Sales[Amount] ), 'Date'[Date] )
Sales SPLY      = CALCULATE ( SUM ( Sales[Amount] ), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Sales Prev Mo   = CALCULATE ( SUM ( Sales[Amount] ), DATEADD ( 'Date'[Date], -1, MONTH ) )
YoY %           = DIVIDE ( SUM ( Sales[Amount] ) - [Sales SPLY], [Sales SPLY] )
```

The complete copy-paste library (running totals, PoP deltas/percentages, moving averages, fiscal YTD) is in [time-intelligence-patterns.md](time-intelligence-patterns.md).

## Step 3 — Handle semi-additive period-end balances

Balances (inventory on hand, account balance, headcount) are **semi-additive**: they sum across product/store/department but must **never be naive-SUMmed across time** — that double-counts the same stock every day. Take the value at the period boundary, or average across the days.

```dax
-- Month-end balance at the exact period boundary
Month End Inventory = CLOSINGBALANCEMONTH ( SUM ( Inventory[QtyOnHand] ), 'Date'[Date] )

-- Value on the last day that actually had data in context
Ending Inventory =
CALCULATE (
    SUM ( Inventory[QtyOnHand] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( Inventory[QtyOnHand] ) ) )
)

-- Average level held over the period (divides by distinct days, not row count)
Avg Daily Inventory = AVERAGEX ( VALUES ( 'Date'[Date] ), CALCULATE ( SUM ( Inventory[QtyOnHand] ) ) )
```

`LASTDATE` vs. `LASTNONBLANK` vs. `CLOSINGBALANCE*` decision guidance and the full recipe set are in [semi-additive-balance-recipes.md](semi-additive-balance-recipes.md). For classifying a fact as additive/semi-additive/non-additive before you choose, use `understanding-fact-additivity`. These balances map to **periodic snapshot** fact tables — see Kimball `fact-table-design-patterns` (`data_warehouse_kimball/SKILL/fact-table-design-patterns/SKILL.md`).

## Key concepts

- **Date table is mandatory.** DAX time intelligence requires at least one date table; classic functions additionally require it be marked. Calendar-based time intelligence (preview) relaxes the marking and the contiguous-dates requirement but needs calendars defined on the table.
- **Lateral vs. hierarchical shifts.** `DATEADD` and `SAMEPERIODLASTYEAR` only shift laterally (same level of detail). All other time intelligence functions allow hierarchical shifts. This matters when a visual mixes granularities.
- **Semi-additive ≠ SUM over time.** Always use closing-balance, last-value, or average-over-days patterns for balances.

## Common pitfalls

- **Using Auto date/time for a multi-fact model.** One hidden table per date column can't propagate a single time filter to sales + budget; switch to a dedicated date table. (Disable Auto date/time in File > Options.)
- **Gaps in the date column.** Classic time intelligence throws an error if any date is missing between the first and last date. Generate a complete, contiguous table.
- **Date column not spanning full years.** A partial year breaks YTD/SPLY math; ensure full-year coverage (use `CALENDARAUTO`).
- **Naive `SUM` on a balance over time.** Inflates totals ~365×; use the semi-additive recipes.
- **Surrogate integer date keys (e.g., 20241231) without marking.** When the relationship uses a non-Datetime key, you **must** Mark as date table and choose the true date column, or time intelligence won't work.
- **Fiscal calendars with classic functions.** Classic functions assume Gregorian or shifted Gregorian; for retail 4-4-5, 13-month, or week-based calendars, use calendar-based time intelligence or fiscal-anchored variants (see patterns file).
- **Nesting calendar-based functions.** Not supported; wrap with `CALCULATETABLE` instead (see patterns file).

## Success criteria

- A single dedicated date table exists, validated against the Step 0 checklist, with Auto date/time off.
- YTD/PoP/SPLY/moving-average measures return correct values at every granularity in a matrix (year, quarter, month).
- Balance measures never grow with the number of days in context; the year total equals a single period-end value (or the true average), not the sum of daily balances.
- Period-over-period measures use `DIVIDE` (no divide-by-zero errors) and blank gracefully outside the data range.

---
*Provenance: built per SKILL 15 "Applying Time Intelligence" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `transform-model/desktop-time-intelligence.md`, `transform-model/desktop-auto-date-time.md`, `transform-model/desktop-date-tables.md`, `guidance/auto-date-time.md`, `guidance/model-date-tables.md`. Cross-references sibling skills `authoring-dax-measures`, `modeling-semantic-data`, `understanding-fact-additivity`, and Kimball `fact-table-design-patterns`.*
