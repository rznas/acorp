# Date-Table Prerequisites Checklist

Time intelligence is only as correct as the date table under it. This file covers the readiness checklist, why a marked date table beats Auto date/time, the three time-intelligence tool options, and how to generate a date table.

## Contents
- Readiness checklist
- The three time-intelligence tools (Auto date/time vs. classic vs. calendar-based)
- Why a dedicated date table beats Auto date/time
- Date-table requirements (validation rules)
- When you MUST mark your date table
- How to mark a date table
- Generating a date table (Power Query M and DAX)
- Role-playing dates (cloning a date table)

---

## Readiness checklist

```
Date-table readiness:
- [ ] One dedicated date table exists (not Auto date/time) so one time filter reaches every fact table
- [ ] Date column is data type Date or Date/Time
- [ ] Date column has unique values
- [ ] Date column has no blank/null values
- [ ] Date column is contiguous — no missing dates between first and last (classic requirement)
- [ ] Date column spans full years (a year need not be Jan–Dec)
- [ ] Table is "Mark as date table" (required for classic time intelligence)
- [ ] Active relationship from each fact's date FK to the date table's date column
- [ ] Auto date/time turned OFF (File > Options and settings > Options > Time intelligence)
```

If you fail "contiguous" with classic functions, you get an error. Either generate a complete table, or switch to calendar-based time intelligence (preview as of authoring — verify current status in Power BI docs before relying on it; it tolerates sparse dates).

---

## The three time-intelligence tools

The docs describe three tools. Recommended order: a dedicated date table with calendar-based (preview) for flexibility/performance; otherwise classic; Auto date/time only for simple ad-hoc models.

| Tool | Setup effort | Management | Flexibility | Notes |
|------|-------------|-----------|-------------|-------|
| **Auto date/time** | virtually zero | hard | low | Hidden date table per date column; increases model size |
| **Classic time intelligence** | medium | easy | low | Requires a marked date table; assumes Gregorian / shifted Gregorian; contiguous dates required; some perf issues |
| **Calendar-based time intelligence (preview)** | high | medium | high | Recommended; works with any calendar (Gregorian, 4-4-5, 13-month, lunar), tolerates sparse dates, supports week-based calcs; docs claim "best performance" but the same docs caution preview performance is not representative — verify at GA |

The docs advise **against** ad-hoc techniques that add offset columns to date tables (they bloat the model and slow refresh) except in specific cases.

### Classic time intelligence (most common today)

Requires a date table that is marked. Then reference the date column directly:

```dax
Sales SPLY = CALCULATE ( SUM ( Sales[Amount] ), SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
```

### Calendar-based time intelligence (preview)

A **calendar** is metadata on a table that tags which columns represent Year, Quarter, Month, Week, Date, etc. Enable it under File > Options and settings > Options > Preview features > **Enhanced DAX Time Intelligence**, then restart. Manage calendars via right-click table > **Calendar options** (or the Table tools ribbon). Reference the calendar name instead of a date column:

```dax
Total Quantity MTD = TOTALMTD ( [Total Quantity], 'ISO-454' )
TOTALWTD ( [Sales], 'Fiscal Calendar' )
```

Key points from the docs:
- Define one or more calendars per table; each needs a unique name and at least one primary column assigned to a category.
- Functions like `TOTALQTD` need enough categories to "walk up" to a year (assign **Quarter**, or **Quarter of Year** + **Year**).
- `DATEADD` and `SAMEPERIODLASTYEAR` shift only laterally; all others allow hierarchical shifts.
- Limitations: can't author calendars in the Power BI Service yet; don't use with Auto date/time tables; can't use with live-connected or composite models; can't nest calendar-based functions.

---

## Why a dedicated date table beats Auto date/time

Auto date/time creates a hidden calculated table (via `CALENDAR`) for **every** Import date column that isn't on the many side of a relationship, each with Day/MonthNo/Month/QuarterNo/Quarter/Year columns and a Year→Quarter→Month→Day hierarchy.

Problems (from the guidance docs):
- **Model size & refresh:** each hidden table inflates the model and extends refresh time.
- **No cross-fact filtering:** because each date column has its own hidden table, one time filter can't propagate to multiple fact tables (e.g., Sales and Sales Budget). A single shared date table can.
- **Calendar periods only:** years are fixed Jan 1–Dec 31; you can't customize fiscal year start/end or add weeks.
- **Year not embedded:** Quarter/Month/Day values omit the year, so grouping by Month sums across all years unless you also group by Year.
- **Tool gaps:** unavailable in Analyze in Excel, paginated-report Analysis Services query designers, and non-Power BI report designers; hidden tables can't be referenced directly by DAX.

Recommendation: keep Auto date/time on only for simple calendar-period, ad-hoc, or profiling models. For anything with multiple facts or fiscal/week needs, use a dedicated date table and disable the global Auto date/time option.

---

## Date-table requirements (validation rules)

A model date table for DAX time intelligence must:
- Have a column of data type **Date** (or **Date/Time**) — the date column.
- Date column contains **unique** values.
- Date column has **no BLANKs**.
- Date column has **no missing dates** (contiguous).
- Date column spans **full years** (year need not be calendar Jan–Dec).
- For **classic** time intelligence, the table is **marked as a date table**.

When you Mark as date table, Power BI validates: unique values, no nulls, contiguous dates, and (for Date/Time) the same timestamp across each value.

---

## When you MUST mark your date table

Even with calendar-based time intelligence, you must mark the table if:
- The relationship to other tables uses a **non-Datetime key** — e.g., a warehouse surrogate key stored as an integer like `20241231`. Mark the table and select the true date column.
- You want **advanced date filters in Excel PivotTables/PivotCharts** on Power BI data.

For classic time intelligence, marking is always required.

---

## How to mark a date table

Fields pane: right-click the table > **Mark as date table > Mark as date table**. Or select the table and use the **Table tools** ribbon > **Mark as date table**. Then **Mark as date table > Date table settings** to choose the date column.

Notes:
- Marking removes the auto-created built-in (hidden) date table for that table's date columns; deselecting it recreates them.
- Set the date column's **Data type** to **Date** or **Date/Time** (Column tools > Data type).
- With calendar-based time intelligence, the **Mark as date table** option appears inside **Calendar options** instead.

---

## Generating a date table

Prefer connecting (Power Query) to an existing date dimension from a data warehouse — a single organization-wide source of truth. For DirectQuery models with no date table, add one to the source. If you must generate, use Power Query M or DAX.

### Power Query M

```powerquery-m
let
    StartDate = #date(2010, 1, 1),
    EndDate = #date(2030, 12, 31),
    NumberOfDays = Duration.Days(EndDate - StartDate) + 1,
    DateList = List.Dates(StartDate, NumberOfDays, #duration(1,0,0,0)),
    DateTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),
    AddYear = Table.AddColumn(DateTable, "Year", each Date.Year([Date]), Int64.Type),
    AddMonthNumber = Table.AddColumn(AddYear, "Month Number", each Date.Month([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddMonthNumber, "Month Name", each Date.ToText([Date], "MMMM"), type text),
    AddQuarter = Table.AddColumn(AddMonthName, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    AddDay = Table.AddColumn(AddQuarter, "Day", each Date.Day([Date]), Int64.Type)
in
    AddDay
```

### DAX

```dax
DateTable =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2010, 1, 1 ), DATE ( 2030, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month Name", FORMAT ( [Date], "MMMM" ),
    "Month Year", FORMAT ( [Date], "MMM YYYY" ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    "Year Quarter", FORMAT ( [Date], "YYYY" ) & " Q" & FORMAT ( [Date], "Q" ),
    "Day", DAY ( [Date] )
)
```

- `CALENDAR(start, end)` — explicit range; bound to data with `MIN(Sales[OrderDate])` / `MAX(Sales[OrderDate])`.
- `CALENDARAUTO([fiscalEndMonth])` — auto-spans all model dates and **auto-extends on refresh**, guaranteeing full years. Pass the fiscal year-end month if not December.

Tip from the docs: define the date table once in a Power BI Desktop **template (.pbit)** or a **dataflow**, and share it so every model uses a consistent definition.

---

## Role-playing dates (cloning a date table)

When date is a role-playing dimension (Order Date and Ship Date), clone the date table with a one-line calculated table, then build a separate relationship:

```dax
Ship Date = 'Date'
```

Cloning copies only columns — not formats, descriptions, or hierarchies; reapply those. For activating an inactive role-playing relationship inside a measure, use `USERELATIONSHIP` (see `modeling-semantic-data` and `authoring-dax-measures`). Maps to Kimball role-playing dimensions in `fact-table-design-patterns`.
