# Date Table Requirements & Setup

Reference for adding, configuring, and marking a date table in Power BI Desktop, grounded in `transform-model/desktop-date-tables.md` and `guidance/model-date-tables.md`. The date table is the prerequisite for time intelligence — see the `applying-time-intelligence` skill for the DAX calculations that depend on it.

## Contents
- Why a single date table
- Date table requirements (the validation checklist)
- Four ways to create a date table
- Mark as date table (how + validation)
- When you MUST mark your date table
- Data type and the DateTime gotcha
- Role-playing date tables
- Pitfalls

## Why a single date table

A date table stores one row per date and supports filtering/grouping by year, quarter, month, etc. To use DAX **time intelligence functions** the model must have at least one date table. A single, consistently defined date table can propagate filters to multiple fact tables — Auto date/time cannot. Within an organization the date table should be defined consistently; consider sharing a Power BI Desktop **template** or a **dataflow** as the single source of truth for time.

## Date table requirements (validation checklist)

A model table qualifies as a date table only if its **date column**:

```
Date table requirements:
- [ ] Data type is Date (or Date/Time)
- [ ] Contains unique values
- [ ] Contains no BLANK / null values
- [ ] Has contiguous dates (no missing dates from start to end)
- [ ] Spans full years (a "year" need not be a calendar Jan–Dec; fiscal years are allowed)
- [ ] If Date/Time, has the same timestamp across every value
- [ ] Marked as a date table when using Classic time intelligence (see below)
```

If any requirement fails, Mark as date table will reject the column and time intelligence will misbehave.

## Four ways to create a date table

Pick by your situation (the docs describe these techniques):

| Technique | Use when | Notes |
|-----------|----------|-------|
| **Auto date/time** | Calendar periods, simple needs, ad-hoc/profiling | Convenient but creates hidden per-column date tables; **cannot** serve as one shared date table across fact tables. Disable for serious models. |
| **Connect with Power Query** | Source already has a date dimension (e.g., a data warehouse) | Recommended — reuse the org's single source of truth. For DirectQuery without one, add a date table to the source. |
| **Generate with Power Query** | No warehouse, want it built in the model | Build a calendar query; consider publishing it as a dataflow for reuse. |
| **Generate with DAX** | Want it in-model, convenient | Calculated table via `CALENDAR(start, end)` or `CALENDARAUTO([fiscalYearEndMonth])`. Extend with calculated columns (Year, Month, etc.). |
| **Clone with DAX** | You already have a date table and need another role | Calculated table whose expression is the existing table name. |

### DAX generation examples
```dax
-- Fixed range driven by the data
Date = CALENDAR(MIN(Sales[OrderDate]), MAX(Sales[OrderDate]))

-- Auto-spanning full years across all model dates;
-- pass the fiscal year-end month if not December (e.g., 6 for a July–June year)
Date = CALENDARAUTO()
Date = CALENDARAUTO(6)
```
`CALENDARAUTO` ensures **full years** are returned (meeting the marked-date-table requirement) and auto-extends the range on each refresh as new-year dates load. Each function returns a single-column table — add calculated columns (e.g., Year, Quarter, Month, MonthNumber) for filtering/grouping.

## Mark as date table

To mark a date table: select the table in the **Fields** pane, right-click > **Mark as date table > Mark as date table** (or use **Mark as date table** on the **Table tools** ribbon). Then choose the date column via **Mark as date table > Date table settings**.

On marking, Power BI **validates** the date column against the requirements above and:
- **Removes** the built-in auto-created date tables for that table — any visuals/DAX that relied on the built-in tables stop working.
- Stops auto-creating built-in hierarchies; you control the hierarchies. If you later unmark it (and have no other marked date table), Power BI recreates the built-in date tables.

> If you use **Calendar-based time intelligence** (preview as of authoring — verify current status in Power BI docs before relying on it), the **Mark as date table** option appears after selecting **Calendar options** in the ribbon / right-click menu.

## When you MUST mark your date table

You do **not** need to mark a date table if you use the recommended **Calendar-based time intelligence** — except in specific circumstances. You **must** mark it when:
- You use the **Classic time intelligence** functions, OR
- Relationships between the date table and other tables use a column whose data type is **not** Datetime — e.g., a **surrogate date key** stored as a whole number formatted `yyyymmdd` (like `20241231`). Marking is required to enable time intelligence in this case, OR
- You want to use **advanced date filters in Excel PivotTables/PivotCharts** on Power BI data.

## Data type and the DateTime gotcha

Set the date column's **Data type** to **Date** or **Date/Time** (Column tools > Data type). Note the engine stores everything as **DateTime**: `Date`, `Time`, and `Date/Time/Timezone` are formatting constructs on top. So a relationship on a date column can fail to match if a stray **time portion** differs between tables, even after choosing "Date" on the Modeling tab. Fix it in **Power Query Editor** by removing the time portion so all values are identical at the engine level.

## Role-playing date tables

When `Date` plays multiple roles (order date, ship date, delivery date), do **not** rely on multiple relationships to one date table for simultaneous filtering. Clone the date table per role (preferred):
```dax
Ship Date = 'Date'
```
Give each clone an active 1:* relationship to its fact column and rename columns by role (`Ship Year`, `Ship Month`). Cloning copies columns only — reapply formats, descriptions, and hierarchies, and re-mark each clone as a date table if you use Classic time intelligence. (Alternatively, keep one date table with inactive relationships and `USERELATIONSHIP` — see relationship-design-matrix; remember RLS won't propagate through inactive relationships.)

## Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Relying on Auto date/time | Can't share one calendar across fact tables; bloated model | Disable Auto date/time; build/connect one date table and mark it |
| Non-contiguous or partial-year dates | Mark as date table rejects the column; time intelligence wrong | Generate a complete, full-year, contiguous calendar (`CALENDARAUTO`) |
| Time portion on Date/Time values | Date relationship won't match | Remove time portion in Power Query |
| Surrogate key (yyyymmdd integer) relationship, table not marked | Time intelligence unavailable | Mark the date table explicitly |
| Marked a table, old visuals break | Visuals/DAX on built-in date tables stop working | Rebuild them against the marked date table's columns |
| Cloned date table missing formats/hierarchies | Inconsistent labels/drilldowns | Reapply formats, descriptions, hierarchies on the clone |
