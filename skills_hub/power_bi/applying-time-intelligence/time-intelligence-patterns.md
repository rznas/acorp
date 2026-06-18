# Time-Intelligence Pattern Library

Copy-paste DAX. Replace `Sales`, `[Amount]`, and `'Date'` with your model's names. `'Date'` is a dedicated date table that meets the [date-table-prerequisites.md](date-table-prerequisites.md) checklist (marked, contiguous, full years). Every snippet is syntactically valid DAX.

These are **classic time intelligence** patterns (reference the date column directly). For calendar-based equivalents, pass a calendar name instead of `'Date'[Date]` (e.g., `TOTALYTD ( [Sales], 'Fiscal Calendar' )`) — see [date-table-prerequisites.md](date-table-prerequisites.md).

## Contents
- Base measure
- Running totals (YTD / QTD / MTD / WTD)
- DATESYTD/QTD/MTD inside CALCULATE (custom variants)
- Same period last year
- Prior single period (PREVIOUSMONTH / DAY / QUARTER / YEAR)
- Shift by N periods (DATEADD)
- Full parallel period (PARALLELPERIOD)
- Period-over-period deltas and percentages
- Moving / rolling averages (DATESINPERIOD)
- Fiscal year-end variants
- Calendar-based notes (lateral vs. hierarchical, nesting)

---

## Base measure

Define the additive base once and reuse it everywhere. (Confirm additivity first — see `understanding-fact-additivity`.)

```dax
Total Sales = SUM ( Sales[Amount] )
```

---

## Running totals (YTD / QTD / MTD / WTD)

`TOTAL*` functions take an expression and a date column (or calendar). They accumulate from the start of the year/quarter/month/week to the current context.

```dax
Sales YTD = TOTALYTD ( [Total Sales], 'Date'[Date] )
Sales QTD = TOTALQTD ( [Total Sales], 'Date'[Date] )
Sales MTD = TOTALMTD ( [Total Sales], 'Date'[Date] )
-- WTD: classic week-to-date is best done with calendar-based time intelligence:
-- Sales WTD = TOTALWTD ( [Total Sales], 'Fiscal Calendar' )
```

---

## DATESYTD / DATESQTD / DATESMTD inside CALCULATE

`TOTALYTD(expr, dates)` is shorthand for `CALCULATE(expr, DATESYTD(dates))`. Use the `DATES*` form when you need to add more filters, or to set a custom year-end:

```dax
Sales YTD (explicit) = CALCULATE ( [Total Sales], DATESYTD ( 'Date'[Date] ) )

-- Fiscal year ending June 30:
Sales FYTD = CALCULATE ( [Total Sales], DATESYTD ( 'Date'[Date], "06-30" ) )
```

---

## Same period last year (lateral shift)

`SAMEPERIODLASTYEAR` shifts the current date context back exactly one year at the same level of detail. It only shifts laterally.

```dax
Sales SPLY = CALCULATE ( [Total Sales], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
```

---

## Prior single period

```dax
Sales Prev Day     = CALCULATE ( [Total Sales], PREVIOUSDAY ( 'Date'[Date] ) )
Sales Prev Month   = CALCULATE ( [Total Sales], PREVIOUSMONTH ( 'Date'[Date] ) )
Sales Prev Quarter = CALCULATE ( [Total Sales], PREVIOUSQUARTER ( 'Date'[Date] ) )
Sales Prev Year    = CALCULATE ( [Total Sales], PREVIOUSYEAR ( 'Date'[Date] ) )
```

---

## Shift by N periods (DATEADD)

`DATEADD(dates, number, interval)` shifts by an arbitrary count of DAY / MONTH / QUARTER / YEAR. Use a negative number to go back. Like `SAMEPERIODLASTYEAR`, it shifts laterally only.

```dax
Sales 1 Month Ago   = CALCULATE ( [Total Sales], DATEADD ( 'Date'[Date], -1, MONTH ) )
Sales 1 Quarter Ago = CALCULATE ( [Total Sales], DATEADD ( 'Date'[Date], -1, QUARTER ) )
Sales 1 Year Ago    = CALCULATE ( [Total Sales], DATEADD ( 'Date'[Date], -1, YEAR ) )
Sales Next Month    = CALCULATE ( [Total Sales], DATEADD ( 'Date'[Date], 1, MONTH ) )
```

> Calendar-based `DATEADD` adds `extension` / `truncation` parameters to control shifts between months of unequal length (e.g., shifting Mar 31 back one month → Feb 28 with `anchored`, or *blank* with `blank`). These don't appear in IntelliSense. See [date-table-prerequisites.md](date-table-prerequisites.md).

---

## Full parallel period (PARALLELPERIOD)

`PARALLELPERIOD` returns the **entire** shifted period even when the current context is partial. Use it for "full prior year" totals shown against a partial current year.

```dax
Sales Full Last Year = CALCULATE ( [Total Sales], PARALLELPERIOD ( 'Date'[Date], -1, YEAR ) )
```

`DATEADD` vs. `PARALLELPERIOD`: DATEADD shifts the dates currently in context (partial in → partial out); PARALLELPERIOD returns the whole period at the chosen interval regardless of how much is in context.

---

## Period-over-period deltas and percentages

Always use `DIVIDE` (returns blank, not an error, on zero denominator).

```dax
YoY Change = [Total Sales] - [Sales SPLY]
YoY %      = DIVIDE ( [Total Sales] - [Sales SPLY], [Sales SPLY] )

MoM Change = [Total Sales] - [Sales Prev Month]
MoM %      = DIVIDE ( [Total Sales] - [Sales Prev Month], [Sales Prev Month] )

-- Reusable with VAR for clarity
YoY % (var) =
VAR Current = [Total Sales]
VAR Prior   = [Sales SPLY]
RETURN DIVIDE ( Current - Prior, Prior )
```

---

## Moving / rolling averages

`DATESINPERIOD(dates, end_date, number, interval)` returns a sliding window; average the base measure over it with `AVERAGEX` (or sum for a rolling total).

```dax
-- Trailing 3-month average of monthly sales
Sales 3M Moving Avg =
AVERAGEX (
    DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -3, MONTH ),
    [Total Sales]
)

-- Trailing 30-day average of daily sales
Sales 30D Moving Avg =
AVERAGEX (
    DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -30, DAY ),
    [Total Sales]
)

-- Rolling 12-month total
Sales Rolling 12M =
CALCULATE (
    [Total Sales],
    DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -12, MONTH )
)
```

---

## Fiscal year-end variants

Classic time intelligence supports **shifted Gregorian** fiscal years (still Gregorian, just a different year start). Pass the year-end date as the last argument:

```dax
-- Fiscal year ending June 30
Sales FYTD = TOTALYTD ( [Total Sales], 'Date'[Date], "06-30" )
Sales FYTD (explicit) = CALCULATE ( [Total Sales], DATESYTD ( 'Date'[Date], "06-30" ) )
```

For retail 4-4-5, 13-month, lunar, or week-based calendars (which are **not** shifted Gregorian), classic functions don't fit — use **calendar-based time intelligence** with a calendar that tags Year/Quarter/Month/Week categories. See [date-table-prerequisites.md](date-table-prerequisites.md).

---

## Calendar-based notes

- **Lateral vs. hierarchical:** only `DATEADD` and `SAMEPERIODLASTYEAR` shift laterally (same level of detail); every other time intelligence function allows hierarchical shifts. With calendars, hierarchical functions clear context on dependent/dependency categories.
- **Required categories:** a function must be able to "walk up" to a year. For `TOTALQTD`, assign **Quarter**, or both **Quarter of Year** and **Year**; otherwise it errors.
- **No nesting** of calendar-based functions. Instead of `PREVIOUSDAY ( PREVIOUSMONTH ( 'Calendar' ) )`, wrap:

```dax
ThisWorks = CALCULATETABLE ( PREVIOUSDAY ( 'Calendar' ), PREVIOUSMONTH ( 'Calendar' ) )
```

- **RLS caution:** a time intelligence calculation on a fact table that both defines a calendar and is subject to row-level security can give unexpected results.
