# Visualization Compatibility Matrix (Fact Type × Chart Type)

How to read it:
- **Safe** — the visual's automatic SUM over every dimension on it produces a correct number for this fact type.
- **Caution** — usable only with a corrected measure (AVERAGEX / LASTNONBLANK / DIVIDE) or a single-period filter; raw column drop is wrong.
- **Avoid** — the visual's part-to-whole or cumulative math reaggregates incorrectly even with a corrected measure; do not use for this fact type.

All visual names below are confirmed Power BI visuals from `powerbi-docs/visuals/power-bi-visualization-types-for-reports-and-q-and-a.md`.

## Contents
- The matrix
- Why each Caution / Avoid
- Quick picks by fact type

---

## The matrix

| Power BI visual | Additive | Semi-additive | Non-additive |
|-----------------|----------|---------------|--------------|
| Single-number Card | Safe | Caution (one period only) | Safe (use ratio measure) |
| Multirow Card | Safe | Caution | Safe (ratio measure) |
| KPI | Safe | Caution (point-in-time vs. target) | Safe (ratio measure) |
| Gauge chart | Safe | Safe (current balance vs. target) | Safe (ratio vs. target) |
| Line chart | Safe | Caution (only with avg/last measure) | Caution (ratio measure over time) |
| Area chart (basic) | Safe | Caution | Avoid |
| Stacked area chart | Safe | Avoid | Avoid |
| Column chart | Safe | Caution | Safe (ratio measure) |
| Bar chart | Safe | Caution | Safe (ratio measure) |
| Ribbon chart | Safe | Avoid | Avoid |
| Combo chart | Safe | Caution | Caution (non-additive line over additive column) |
| Waterfall chart | Safe | Avoid | Avoid |
| Pie chart | Safe | Avoid | Avoid |
| Donut chart | Safe | Avoid | Avoid |
| Treemap | Safe | Avoid | Avoid |
| Funnel chart | Safe | Avoid | Avoid |
| Matrix | Safe | Caution (period grouping + measure) | Safe (ratio measure) |
| Table | Safe | Caution (period grouping + measure) | Safe (ratio measure) |
| Scatter / bubble / dot plot | Safe | Caution | Safe (axes are ratio measures) |
| Map / Filled map / Shape map | Safe | Caution (one period) | Caution (ratio measure, no time) |
| Decomposition tree | Safe | Caution | Caution (ratio measure) |
| Key influencers | Safe | Caution | Caution |
| Smart narrative | Safe | Caution | Caution |
| Q&A visual | Safe | Caution (depends on measure) | Caution (depends on measure) |

---

## Why each Caution / Avoid

### Semi-additive

- **Card / Multirow Card / KPI — Caution:** correct for a single period (one date, one month). If the filter spans multiple dates and the measure is raw `SUM`, the card adds the same balance across days. Use a Card with an `AVERAGEX`/last-balance measure or filter to one period.
- **Gauge — Safe:** designed to show one current value against a target; pair with `Ending Balance` or `Current Headcount`.
- **Line / Area / Column / Bar / Matrix / Table — Caution:** safe only when the plotted measure averages or takes the last value over time. A raw balance produces an upward-drifting series that is just cumulative double-counting.
- **Pie / Donut / Treemap / Stacked area / Ribbon / Waterfall / Funnel — Avoid:** these compute a part-to-whole or running total. The "whole" sums the balance across every time slice in scope, so the total exceeds the real balance and every slice is wrong. Filter to one point in time and switch to a Card/Bar if you must.
- **Scatter / Map / Decomposition tree — Caution:** acceptable for a single period or with a corrected measure; raw balance across time is wrong.

### Non-additive

- **Card / KPI / Gauge / Column / Bar / Matrix / Table / Scatter — Safe with a ratio measure:** because `Gross Margin % = DIVIDE(SUM(profit), SUM(revenue))` recomputes from summed components at every cell, it reaggregates correctly. Never drop the raw `%` or `UnitPrice` column — the implicit SUM/AVERAGE is meaningless.
- **Line — Caution:** fine to show a ratio measure over time; just confirm it is a `DIVIDE` of sums, not an average of stored percentages.
- **Combo — Caution:** valid pattern is a non-additive ratio on the line axis over an additive total on the column axis (e.g., margin % line + revenue columns); keep the non-additive value off the summed axis.
- **Pie / Donut / Treemap / Stacked area / Area / Ribbon / Waterfall / Funnel — Avoid:** all assume the values add up to a meaningful whole. Percentages, prices, and rates do not sum, so slice sizes and running totals are nonsense.
- **Map / Decomposition tree / Key influencers / Q&A / Smart narrative — Caution:** work only on a properly defined ratio measure; results are wrong if pointed at a raw non-additive column.

### Additive
Every visual is Safe — the implicit SUM over all dimensions is exactly what these facts support. Choose by analytical intent (trend, comparison, part-to-whole, geography), which is the job of the `picking-the-right-visual` skill.

---

## Quick picks by fact type

- **Additive** (revenue, quantity, cost, profit, duration, counts): Line/Area for trend, Column/Bar for category comparison, Pie/Donut/Treemap for part-to-whole, Waterfall for contribution, Map for geography, Card/KPI for the grand total.
- **Semi-additive** (inventory, balance, headcount, backlog): Card/KPI for one period, Gauge vs. target, Bar/Matrix with a period grouping; Line only with an averaging/last-value measure. Avoid all part-to-whole and running-total visuals.
- **Non-additive** (margin %, price, rate, ratio, score): build a `DIVIDE` ratio-of-sums measure first, then Card/KPI/Gauge, Column/Bar, Matrix/Table, or Scatter (two ratio axes). Avoid Pie/Donut/Treemap/Waterfall/Area on the raw value.
