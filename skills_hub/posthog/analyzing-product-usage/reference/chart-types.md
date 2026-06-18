# Chart types — detail

Ten trends visualizations in two families. Choosing the right one is part of answering the question. Source: `product-analytics/trends/charts.mdx`.

## Time series (how a value fluctuates over time)
| Chart | Reads as / when to use |
|---|---|
| **Line (linear)** | Standard trend over time; extra series/breakdowns = extra lines |
| **Line (cumulative)** | Running tally; is growth exponential or leveling off |
| **Bar (time series)** | Same data as linear; stacked bars show each series/breakdown's contribution to the total |
| **Area** | Like line, but stacked — each series' share of the total over time |
| **Box plot** | Distribution of a numeric property over time (min, Q1, median, mean, Q3, max); spot outliers. Requires a numeric property; no breakdowns/formulas. "Exclude outliers" (default on) clips whiskers at 1.5× IQR |

## Total value (one number per series over the whole range)
| Chart | Reads as / when to use |
|---|---|
| **Number** | Single big total; tick "Compared to previous period" for a % delta. Only with one series/formula and no breakdowns |
| **Bar (value)** | Total per series over the whole range; pair with a breakdown to get a top-N ranking. "Stack breakdown values" groups bars by series |
| **Table** | Raw numbers over the range; "Compare to previous period" adds a side-by-side column |
| **Pie** | Relative distribution across series/breakdowns over the range |
| **World map** | Values by country code; can't combine with a non-country breakdown |
| **Calendar heatmap** (beta) | Unique users or total events by hour-of-day × day-of-week; "All" row/column give per-hour and per-day totals. Use 7-day or multiples-of-7 ranges. Opt-in via feature previews |

## Quick picks for PM questions
- Trend of a metric → line; smooth noisy data with rolling average.
- Is growth accelerating → cumulative line.
- Top countries/browsers/plans → value bar (or pie) + breakdown; geography → world map.
- One headline KPI vs last period → number chart with comparison.
- Distribution / outliers of a numeric value (e.g. load time, duration) → box plot.
- Engagement by time-of-day/weekday → calendar heatmap (beta).
