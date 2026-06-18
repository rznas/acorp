# SQL / HogQL reference

PostHog's SQL is HogQL (ClickHouse-flavored). This is a PM-oriented cheat sheet — enough to read, tweak, and spec queries. No trailing semicolon needed.

## Contents

- [Core commands](#core-commands)
- [Property and type access](#property-and-type-access)
- [Cohorts and actions](#cohorts-and-actions)
- [Time-series patterns](#time-series-patterns)
- [SQL variables and dashboard filters](#sql-variables-and-dashboard-filters)
- [Useful function categories](#useful-function-categories)

## Core commands

| Command | What it does |
| --- | --- |
| `SELECT` | Choose columns/aggregations. `SELECT DISTINCT` for unique rows. |
| `FROM` | Table or subquery. Subqueries enable multi-group comparisons. |
| `JOIN` | `INNER` / `LEFT` / `RIGHT` / `FULL`, combined with `ON`. Combine PostHog + external data. |
| `WHERE` | Filter rows before grouping (comparisons, `AND`/`OR`, `LIKE`, `IN`, `IS NULL`). |
| `GROUP BY` | Group for aggregates like `count()`. |
| `HAVING` | Filter groups after aggregation. |
| `ORDER BY` | Sort `ASC`/`DESC`; supports `WITH FILL` + `INTERPOLATE` for gap-free time series. |
| `LIMIT` / `OFFSET` | Default `LIMIT 100`. Raise it for dashboards/long ranges. |
| `WITH` | CTE — name a temporary result. Recomputed each use. |
| `WINDOW` | Running totals, moving averages, ranking, percentiles (`OVER w`). |
| `CASE` | Conditional buckets (e.g. group OS into mobile/desktop/other). |
| `AS` | Alias columns/tables — **required for every field** if the query will become a view. |

Comments use `--`.

### Revenue/plan bucketing example
```sql
SELECT
  CASE
    WHEN amount >= 10000 THEN 'enterprise'
    WHEN amount >= 1000  THEN 'mid'
    ELSE 'self-serve'
  END AS tier,
  count() AS customers
FROM prod_stripe_customer
GROUP BY tier
ORDER BY customers DESC
```

## Property and type access

- Dot notation: `properties.$browser`, `person.properties.$initial_browser`. Bracket for special chars: `properties['$feature/cool-flag']`. Nested works: `properties.$some.nested.property`.
- PostHog properties are prefixed with `$`; custom ones are not.
- Types: `STRING` (default), `JSON`, `DATETIME` (ISO 8601), `INTEGER`, `FLOAT`, `BOOLEAN`. Cast with `toString`, `toDate`, `toFloat`, `JSONExtractString`, `JSONExtractInt`.
- Quotes: single `'` for string literals, double `"` or backticks `` ` `` for identifiers.
- Null safety: wrap with `COALESCE`, filter `IS NOT NULL`, or `assumeNotNull`.

## Cohorts and actions

- Cohort membership: `WHERE person_id IN COHORT 'Power users'` (on `events`) or `WHERE id IN COHORT 'Power users'` (on `persons`).
- Actions: `WHERE matchesAction('clicked homepage button')`. To customize, open the action in data management -> Matching events -> export dropdown -> **Edit SQL directly**, then copy its `WHERE`/`SELECT` into your query.

## Time-series patterns

Short time ranges keep queries cheap and fast — almost always filter on `timestamp`:
```sql
SELECT count() FROM events WHERE timestamp >= now() - INTERVAL 7 DAY
```

Gap-free daily series with `WITH FILL`:
```sql
SELECT toStartOfDay(timestamp) AS day, count() AS event_count
FROM events
WHERE timestamp > now() - INTERVAL 7 DAY
GROUP BY day
ORDER BY day WITH FILL
  FROM toStartOfDay(now() - INTERVAL 7 DAY)
  TO toStartOfDay(now())
  STEP INTERVAL 1 DAY
INTERPOLATE (event_count AS 0)
```

## SQL variables and dashboard filters

- Create variables via the **Variables** button in the SQL editor; reference as `{variables.<name>}`. A List variable (e.g. `event_names` = `$pageview`, `$autocapture`) can be set per dashboard, swapping the value everywhere it appears.
- Respect dashboard date range + filters: add `{filters}` to the `WHERE` clause:
  ```sql
  SELECT * FROM events
  WHERE event = '$pageview' AND {filters}
  ```
- Or reference the range explicitly: `{filters.dateRange.from}` and `{filters.dateRange.to}`.

## Useful function categories

(Browse the full list in the SQL editor / `/docs/data-warehouse/sql/useful-functions`.)

- **Aggregations:** `count()`, `sum`, `avg`, `min`, `max`, `uniq`, percentile functions; ClickHouse aggregate states like `uniqExactMerge()` for `session_replay_features`.
- **Date/time:** `now()`, `today()`, `toDate`, `toStartOfDay`, `dateDiff`, `INTERVAL`.
- **Strings:** `concat`, `lower`/`lowerUTF8`, `replaceRegexpAll`, `splitByChar`, `splitByString`, `trim`.
- **JSON:** `JSONExtractString`, `JSONExtractInt` for parsing stored `properties`.
- **Conditional/null:** `CASE`, `COALESCE`, `assumeNotNull`, `if`.
