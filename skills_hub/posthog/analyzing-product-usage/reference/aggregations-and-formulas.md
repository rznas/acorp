# Aggregations & formulas — detail

How a trends series is counted, and how to combine series. Sources: `product-analytics/trends/aggregations.mdx`, `formulas.mdx`, `statistical-analysis.mdx`.

## Event aggregation (how a series is counted)
| Aggregation | Means |
|---|---|
| **Total count** | Every occurrence of the event in the period |
| **Count per user** | Per-user occurrence, summarized as Average / Min / Max / Median / 75th / 90th / 95th / 99th percentile |
| **Unique users** | Distinct users who did it (same user twice = 1) |
| **Weekly active users (WAU)** | Rolling distinct users over trailing 7 days |
| **Monthly active users (MAU)** | Rolling distinct users over trailing 30 days |
| **Unique sessions** | Distinct sessions containing the event |
| **Unique groups** | Distinct groups (needs group analytics) |

PM mapping: "how many people use feature X?" → Unique users. "How active is our base?" → DAU/WAU/MAU. "How heavily do users use X?" → Count per user (average or a percentile). "How many companies are active?" → Unique groups.

## Property value aggregation (numeric properties only)
Average, Sum, Minimum, Maximum, Median, 75th / 90th / 95th / 99th percentile. E.g. "95th percentile load time", "average session duration", "sum of revenue". Percentiles answer tail questions ("95% of events have a value below this") better than averages.

## SQL aggregations
Custom counting via SQL expressions when the built-ins aren't enough.

## Formulas (combine series)
- Click **Enable formula mode**; type an expression using `+ - * /` and series letters `A`, `B`, `C`.
- `A / B` plots series A divided by series B at each point — use for rates/ratios (e.g. conversions ÷ visitors).
- Add multiple formulas; each renders as its own series. Pairs well with stacked breakdown value bars.

## Confidence intervals (significance for trends)
- Trends Options → **Show confidence intervals** (default 95%; settable 90/95/99).
- Read: shaded band = plausible range of the true value; wider = less certain.
- Two lines: non-overlapping bands ≈ likely significant difference; overlapping = inconclusive, gather more data.
- Most useful with sampling on or when detecting small changes — keeps PMs from reacting to noise.
