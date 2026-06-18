---
name: querying-the-data-warehouse
description: Guides product managers using PostHog's data warehouse to blend product analytics with external business data (Stripe billing, HubSpot/Salesforce CRM, Postgres, Zendesk, etc.) and build custom metrics standard insights can't. Use when a PM asks to combine product usage with revenue, MRR, subscription, CRM, or support data; join external sources to PostHog events/persons/groups; query data with SQL/HogQL in the SQL editor; create views or materialized views; surface a metric not available as a normal trend/funnel; calculate per-customer revenue, ARPU, churn-by-plan, or LTV; or turn a SQL query into an insight, chart, or dashboard tile. Covers connecting sources, joins, query patterns, and reading query results.
---

# Querying the data warehouse

PostHog's data warehouse lets a PM treat external business data (billing, CRM, support, your own Postgres) as queryable tables alongside PostHog's own `events`, `persons`, `sessions`, and `groups`. You write SQL (HogQL) in the **SQL editor**, join across sources, save reusable **views**, and turn any query into an **insight**, chart, or dashboard tile.

Use this when the answer to a product question lives partly outside PostHog — e.g. "which feature do our highest-paying customers use most?" — or when you need a metric no standard trend/funnel produces.

## PM question -> what to use

| PM question | Approach |
| --- | --- |
| "How much revenue do users of feature X generate?" | Join `events` (or `persons`) to Stripe/billing source, group by feature usage |
| "Which CRM lifecycle stage converts best?" | Join HubSpot/Salesforce source to PostHog persons, break down funnel by stage |
| "Show me ARPU / MRR / churn by plan" | Query billing source directly in SQL editor, visualize as big number / line chart |
| "Top customers by event count" | `INNER JOIN` events to `stripe_customer` on email, group by customer |
| "A custom metric no trend/funnel gives me" | Write the SQL, save as view, then **Create insight** |
| "Reuse this complex query everywhere" | Save it as a **view**; **materialize** it if it's slow |
| "Filter/break down a normal insight by CRM field" | Set up a **join** so source properties appear in filters & breakdowns |
| "Audit which warehouse tables we actually use / cut cost" | Query `query_log`; see [reference/cost-and-performance.md](reference/cost-and-performance.md) |
| "Have someone write the SQL for me" | Use PostHog AI / Max — see [PostHog AI helpers](#posthog-ai-and-mcp-helpers) |

## Where it lives

- **SQL editor** — write and run queries, see the schema of every PostHog table, external source, and saved view in the left panel, and create insights. (app: SQL section.)
- **Data warehouse / Sources** — connect external sources and manage syncs.
- **Insights** — add a warehouse table directly to a trend, funnel, lifecycle, or stickiness via the **Data warehouse tables** tab on a series.

## Two ways to get value (pick based on the PM job)

1. **SQL editor first** — exploratory or custom metric. Write SQL, run, visualize, optionally **Create insight** and add to a dashboard. Best for one-off questions and bespoke metrics.
2. **Insight first** — when the metric fits a standard trend/funnel/lifecycle/stickiness. Add the warehouse table to a series and map a few fields (ID, distinct ID, timestamp). Best when you also want PostHog's built-in breakdowns and actor drill-through.

## Workflow: blend product data with external data

1. **Connect the source.** Managed sources (Stripe, HubSpot, Salesforce, Zendesk, Postgres, MySQL, Snowflake, BigQuery, etc.) — you give credentials and PostHog syncs the data. Self-hosted sources — link CSV/Parquet in your own object storage. See the full catalog and setup at `/docs/cdp/sources`; overview in [reference/sources-and-joins.md](reference/sources-and-joins.md).
2. **Decide how to combine.** For reuse across many insights, define a saved **join** on the source table. For a single query, just `JOIN` or `WHERE ... IN` inline. See [reference/sources-and-joins.md](reference/sources-and-joins.md).
3. **Write the query** in the SQL editor. Pick a connecting key that actually matches — commonly PostHog `events.distinct_id` or a person `email` property to the source's customer `email`. (Note: `persons.id` is a UUID, not an email.)
4. **Read the result** — check the table, then choose a visualization. Watch **Execution details** (memory, data read, duration) to keep the query cheap.
5. **Operationalize** — **Create insight** and add to a dashboard, or **Save as view** to reuse the query.

### Example: revenue per active user (events + Stripe)

```sql
SELECT events.distinct_id, COUNT() AS event_count
FROM events
INNER JOIN prod_stripe_customer ON events.distinct_id = prod_stripe_customer.email
GROUP BY events.distinct_id
ORDER BY event_count DESC
```

### Example: count events for CRM contacts (events + HubSpot)

```sql
SELECT COUNT() AS event_count, distinct_id
FROM events
WHERE distinct_id IN (SELECT email FROM hubspot_contacts)
GROUP BY distinct_id
ORDER BY event_count DESC
```

For the full SQL/HogQL command and function reference (joins, CTEs, window functions, `CASE`, property access, cohorts, actions), see [reference/sql-hogql-reference.md](reference/sql-hogql-reference.md).

## Joins: three flavors

- **Table join** — saved join between any source table and a PostHog table (or two sources). Set up in the SQL editor: three dots next to a table -> **Add join**. After joining, source fields are usable in filters, breakdowns, and SQL expressions. Edit/delete via **View table schema**.
- **Person join** — a table join onto the `persons` table. Start the join *from* `persons` so the joined fields become **extended person properties** usable like any person filter across insights.
- **Query join** — `JOIN` / `WHERE IN` for a single query only, no saved relationship.

Detail and gotchas (matching keys, UUID vs email) in [reference/sources-and-joins.md](reference/sources-and-joins.md).

## Reading and visualizing results

In the SQL editor, the result is a **table** by default. Click the **Visualization** tab to switch to big number, line, bar, stacked bar, or area, and configure axes, legend, scale, and goal lines. The **Series** tab formats numbers (Number, Short Number like "1.2K", Percentage, prefix/suffix/decimals). Tables support **conditional formatting** to highlight cells by value.

**Decision read:** a big number is the headline metric (e.g. current MRR); a line chart shows the trend (revenue per week); a table with conditional formatting flags outliers (accounts below a threshold). After running, hover **Execution details** to see memory, data read, CPU, and duration — a spike in bytes read is the signal to add a tighter filter.

## Using warehouse data in standard insights

Add a warehouse table to a trend, funnel, lifecycle, or stickiness via the **Data warehouse tables** tab on a series. You then map a few fields PostHog can't infer reliably:

- **Trend / stickiness:** ID, Distinct ID, Timestamp.
- **Funnel:** ID, Aggregation target (the actor matched across steps — person/group id), Timestamp. Use **Custom aggregation target** if steps don't map to people/groups (actor drill-through then won't work).
- **Lifecycle:** Aggregation target, Created at (when the actor first existed), Timestamp (when activity happened). These are often different columns (e.g. `created_at` vs `invoice_paid_at`).

You can add the **same table multiple times** with different timestamp fields to model stages — e.g. a Salesforce `opportunity` funnel using `created_date` for "created" and `close_date` for "closed". Source properties work as filters and breakdowns (e.g. `zendesk_tickets.status = 'open'`, `hubspot_companies.lifecyclestage = 'lead'`).

> Self-hosted note: dashboard property-filter overrides do **not** apply to insights containing data warehouse series (a warning shows on the card); dashboard date-range overrides still apply. This is a product behavior, not a self-host limitation.

## Views and reuse

- **Save as view** — turn a query into a named, composable table referenced in other queries. Every selected field must be aliased with `AS` (so `SELECT *` can't be a view). Nested views are supported. You can also **link a view to a PostHog table** so the view's fields are accessible directly on that table.
- **Materialized view** — precompute and store a slow/frequent query for fast reads (e.g. KPI dashboards, embedded analytics). Set up via the **Materialization** tab -> **Save and materialize** (name without spaces). Schedule refresh from every 15 minutes up to monthly, or never.

**PM decision:** materialize only the slow part of a query, accept that a materialized view is as fresh as its last refresh (a dashboard on it won't update between refreshes), and remember materialization runs time out after 1 hour. Full guidance in [reference/views-and-materialization.md](reference/views-and-materialization.md).

## SQL variables and dashboard filters

Create reusable variables in the SQL editor (**Variables** button) and reference them as `{variables.<name>}` — a List variable like `event_names` can be flipped per dashboard. To make a SQL insight respect a dashboard's date range and filters, include `{filters}` in the `WHERE` clause, or use `{filters.dateRange.from}` / `{filters.dateRange.to}` explicitly. See [reference/sql-hogql-reference.md](reference/sql-hogql-reference.md).

## PostHog AI and MCP helpers

- **PostHog AI (Max)** writes, fixes, and explains HogQL from plain English, knows your schema (including Stripe/HubSpot tables), and can list tables, describe columns, and show sample rows. Click the AI icon in the SQL editor. Great for PMs who spec metrics but don't write SQL. See the **using-posthog-ai-assistant** skill.
- **PostHog MCP server** lets an AI coding agent (Cursor, Claude Code, VS Code, etc.) run HogQL, manage views (create/materialize/refresh), and configure/sync sources without opening the app.

> Self-hosted note: PostHog AI/Max is a hosted LLM feature and requires AI data-processing access; availability and setup may differ on self-hosted instances. Confirm in your deployment before relying on it. The SQL editor, joins, views, and materialization are core features (free/self-serve/enterprise: full).

## What the warehouse does NOT change

PostHog is an **integrated** warehouse: data is ingested into PostHog and all query execution runs on PostHog compute, whether the source is native events or a synced external database (Snowflake, Postgres, etc.). You don't query your external warehouse directly — you sync the needed tables in. Pricing is usage-based with a free tier (first 1M rows/month free); set billing limits.

## Related skills

- **using-posthog-ai-assistant** — natural-language SQL, schema exploration, MCP.
- **analyzing-product-trends**, **building-conversion-funnels**, **measuring-retention-and-lifecycle** — standard insights that warehouse tables plug into.
- **running-experiments** — use warehouse tables as experiment metrics.
- **building-dashboards** — surface warehouse-powered insights and SQL tiles.
- **managing-data-pipelines** — connecting and configuring external sources (`/docs/cdp/sources`).
