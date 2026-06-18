# Views, materialization, cost, and performance

How to make warehouse queries reusable and fast, and keep costs down.

## Contents

- [Views](#views)
- [Linking a view to a PostHog table](#linking-a-view-to-a-posthog-table)
- [Materialized views](#materialized-views)
- [Cost and performance](#cost-and-performance)
- [Auditing usage with query_log](#auditing-usage-with-query_log)

## Views

A view saves a query as a named, composable table you can reference in later queries.

- **Create:** inside a SQL insight, when the query has valid view characteristics the **Save as view** button enables; click it and name the view.
- **Requirement:** every selected field must be aliased with `AS`. So `SELECT * ...` cannot be a view (no aliased columns). The alias names are how you access fields on the view.
- **Nested views** are supported — build a view on top of other views.

**PM value:** define a canonical metric once (e.g. `active_paying_accounts`) and reuse it across insights and dashboards so numbers stay consistent.

## Linking a view to a PostHog table

You can attach a view to a PostHog model so its fields are accessible directly on that table — avoids re-creating a manual join. Go to the data warehouse section -> PostHog tab -> **Link table to view**, pick tables and join keys, save. Querying that PostHog table then exposes the view's fields.

Example use: associate Stripe customer data (a view) with product usage on the PostHog `persons`/`events` table.

## Materialized views

Materializing precomputes and stores a view's result for fast reads — use for expensive, frequently run queries (KPI dashboards, embedded analytics).

- **Set up:** SQL editor -> **Materialization** tab below the query -> **Save and materialize** -> name without spaces. Query it like any table.
- **Schedule refresh:** anywhere from every 15 minutes up to monthly, or never. Match cadence to upstream freshness (e.g. if billing syncs daily, refresh the view daily).

### PM decisions and limits

- Only materialize **slow** queries — fast ones don't benefit.
- You can materialize just the **slow part** (a `WITH` CTE or subquery) and reuse it.
- Data is only as fresh as the **last refresh** — a dashboard built on a materialized view will not update between refreshes. The minimum interval is 15 minutes; if the query takes longer than the interval, it updates only after it finishes and reruns next interval.
- Materialization runs get more compute/memory than standard queries but still **time out after 1 hour**.

## Cost and performance

Pricing is usage-based with a free tier (first 1M rows/month free); set billing limits to avoid surprises. For custom SQL, performance and bytes-read are your responsibility (PostHog optimizes its own product queries, not arbitrary SQL).

Levers, highest impact first:

1. **Shorter time ranges** — always filter `timestamp`; the shorter the better (`now()`, `INTERVAL`, `dateDiff`).
2. **Materialize** the data you need so reads hit precomputed results.
3. **Don't scan a big table (`events`, `persons`) twice** in one query — materialize a base slice once, then query that.
4. **Name your queries** descriptively so you can find slow ones in `query_log`.
5. **Keyset pagination, not `OFFSET`** — `OFFSET` is rejected (HTTP 400) for programmatic `/query` requests via personal API keys. To export, use batch exports, not `/query`.
6. Watch **Execution details** (memory, data read, CPU, duration) after each run — a spike in bytes read means add a tighter filter.

### Sync-side cost (managed sources)

- Use **incremental** or **append-only** sync instead of full-table sync (keep the replication key non-nullable).
- Reduce **sync frequency** to the freshness you actually need.
- **Disable unused tables/sources** — they keep syncing and costing.

## Auditing usage with query_log

The `query_log` table records every query in the project. Use it to find unused/underused tables (candidates for disabling sync) by parsing the table referenced after `FROM`:

```sql
WITH cleaned AS (
    SELECT query,
      replaceRegexpAll(replaceRegexpAll(replaceRegexpAll(lowerUTF8(query), '--[^\\n]*', ' '), '/\\*[^*]*\\*+(?:[^/*][^*]*\\*+)*/', ' '), '\\s+', ' ') AS q
    FROM query_log
),
tail AS (
  SELECT query,
    arrayElement(splitByString(' from ', q), length(splitByString(' from ', q))) AS after_from
  FROM cleaned
),
tokens AS (
  SELECT query,
    trim(BOTH ' ' FROM replaceAll(replaceAll(arrayElement(splitByChar(',', arrayElement(splitByChar('(', arrayElement(splitByChar(' ', after_from), 1)), 1)), 1), '`',''), '"','')) AS tok
  FROM tail
)
SELECT tok AS table_after_from, count() AS hits
FROM tokens
WHERE tok != ''
GROUP BY table_after_from
ORDER BY hits ASC
```

This ranks tables by how often they're referenced (includes views/subqueries, so it's approximate). Low-hit tables are candidates to disable.
