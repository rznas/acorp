# Sources and joins reference

Connecting external data and combining it with PostHog data.

## Contents

- [PostHog data available by default](#posthog-data-available-by-default)
- [Source types](#source-types)
- [Saved joins](#saved-joins)
- [Person joins and extended person properties](#person-joins-and-extended-person-properties)
- [Query joins (inline)](#query-joins-inline)
- [Matching keys: common gotchas](#matching-keys-common-gotchas)

## PostHog data available by default

These tables exist with no setup — they are your product-side data for blending:

| Table | PM use |
| --- | --- |
| `events` | Core usage data: feature events, pageviews. Access props with `properties.$current_url`, `person.properties.$initial_browser`. |
| `persons` | The user behind events. `id` (UUID), `created_at`, `last_seen_at`, `is_identified`, `properties`. Connect via `events.person_id = persons.id`. |
| `groups` | Org/account/team aggregation (needs group analytics). `index`, `key`, `properties`. |
| `sessions` | Session-level metrics: `$session_duration`, `$is_bounce`, `$channel_type`, `$entry_utm_source`, `$pageview_count`. Heavily used in web/marketing analytics. |
| `session_replay_features` | Behavioral signals per replay session: `rage_click_count`, `dead_click_count`, `console_error_count`. (`unique_url_count` needs `uniqExactMerge()`.) |
| `query_log` | Your team's internal PostHog query usage — for governance, cost, and performance audits. |
| `system.*` tables | Project metadata: `system.actions`, `system.activity_logs`, `system.batch_exports`, `system.integrations`, `system.data_warehouse_tables`, `system.notebooks`, `system.support_tickets`, and more. Query with the `system.` prefix. |

## Source types

**Managed sources** — provide credentials; PostHog runs extract + load and syncs automatically into PostHog storage. Examples referenced in docs: Stripe (billing/revenue), HubSpot and Salesforce (CRM), Zendesk (support), Postgres (your own app DB). The full, current catalog and per-source setup (IP allowlisting, sync config, custom sources) lives at `/docs/cdp/sources`.

**Self-hosted sources** — upload CSV or Parquet to your own object storage and link it as a source. PostHog reads from your storage on every query, so you control freshness and volume.

> Self-hosted note: some managed connectors and the managed extract/load pipeline are operated by PostHog; on a self-hosted PostHog instance, availability of specific managed connectors may differ. Self-hosted CSV/Parquet sources read from object storage you control. Confirm connector availability in your deployment.

### Cost-aware syncing

- Prefer **incremental** or **append-only** sync over full-table sync to cut rows synced (ensure the replication key isn't nullable — null rows are skipped).
- Match **sync frequency** to the freshness you actually need (every 5 min up to monthly).
- **Audit and disable** unused tables; sources keep syncing (and costing) even when unused. See cost reference for the `query_log` audit query.

## Saved joins

A saved join is a persistent relationship interpreted whenever the origin table is queried.

**Set up:** SQL editor -> three dots next to the source table -> **Add join**. Define source table key, joining table, joining table key, and how fields are accessed.

**Example:** join `events.distinct_id` to `stripe_customer.email`. Afterward you can reach customer fields through events: `SELECT stripe_customer.id FROM events`. The joined source properties then appear in **filters, breakdowns, and SQL expressions** across insights.

**Edit/delete:** three dots -> **View table schema** -> three dots next to the joined table -> **Edit** or **Delete**.

## Person joins and extended person properties

A person join is a saved join with the `persons` table as the **source** table. Start it from `persons` (three dots next to `persons` -> **Add join**) so the joined fields become **extended person properties** — usable like native person filters and breakdowns in any insight.

This is the pattern for "filter/break down product insights by a CRM or billing attribute" without writing SQL each time.

## Query joins (inline)

For a one-off, skip the saved join and combine inline:

`WHERE ... IN` subquery:
```sql
SELECT COUNT() AS event_count, distinct_id
FROM events
WHERE distinct_id IN (SELECT email FROM hubspot_contacts)
GROUP BY distinct_id
ORDER BY event_count DESC
```

`JOIN` (INNER / LEFT / RIGHT / FULL):
```sql
SELECT events.distinct_id, COUNT() AS event_count
FROM events
INNER JOIN prod_stripe_customer ON events.distinct_id = prod_stripe_customer.email
GROUP BY events.distinct_id
ORDER BY event_count DESC
```

## Matching keys: common gotchas

- The two keys must genuinely match values. A common bridge is **email**: PostHog `events.distinct_id` (if you identify by email) or a `person.properties.email` against the source's customer email.
- `persons.id` is a **UUID**, even if you capture events with an email as `distinct_id`. Don't join `persons.id` directly to an external email column — add/join on an `email` person property instead.
- Source table names depend on the connector and any prefix you set (e.g. `prod_stripe_customer`, `stripe.prod.charge`, `hubspot_contacts`). Browse the left panel of the SQL editor (or ask PostHog AI "what tables do I have?") to confirm exact names.
