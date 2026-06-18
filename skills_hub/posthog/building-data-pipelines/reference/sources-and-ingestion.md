# Sources & ingestion details

How to bring external and inbound data into PostHog. See SKILL.md for the overview and catalog.md for the full connector list.

## Table of contents
- [Setup flow](#setup-flow)
- [Sync methods in depth](#sync-methods-in-depth)
- [Managing schemas after setup](#managing-schemas-after-setup)
- [Custom object-storage source](#custom-object-storage-source)
- [Custom REST source](#custom-rest-source)
- [Incoming webhooks source](#incoming-webhooks-source)
- [Inbound IP allow-list](#inbound-ip-allow-list)

## Setup flow
Data pipeline → **Sources** (Integrations tab) → **+ New source** → choose connector → enter credentials → select tables/schemas → pick a sync method per table. After linking, query the data in the [data warehouse](/docs/data-warehouse) and join it to person/event data.

## Sync methods in depth
| Method | Behaviour | PM trade-off |
|---|---|---|
| **Webhook** | Source pushes inserts/updates/deletes in real time (e.g. Stripe). Configured at setup. | Freshest + cheapest; only some sources support it. Recommended when shown. |
| **Incremental** | Syncs new/changed rows using a replication key (`integer`/`bigint`/`smallint`, `datetime`/`date`/`timestamp`, `numeric` for Snowflake). Dedupes on primary key (auto-detected; can set custom; locked after first sync). | **Deletions are not synced.** Nullable key rows are skipped. |
| **Append only** | Appends rows using an incremental cursor; no merges. Best for immutable log tables. Use `created_at` not a mutable field to avoid dupes. | No updates/deletes. |
| **Full table** | Reloads the whole table each sync. | Reflects all changes incl. deletes; most expensive. Good fallback. |

**Frequency:** CDC schemas → as low as every 1 minute. Incremental/Full/Append → 5-minute minimum (enforced in UI and API).

### Stripe example (illustrative)
Stripe strongly recommends **webhook sync** — it's the only mode that reliably captures mutations (subscription cancelled, invoice paid, email corrected) and is cheapest. Append-only misses updates (Stripe has no "updated since" filter for most resources); full refresh is expensive. Connect with a restricted API key that has **Write on Webhooks** (or OAuth) so PostHog can create the real-time sync webhook.

## Managing schemas after setup
On the Sources page you can: see status / sync frequency / last successful run / rows synced; **Pull new schemas** (discover new tables without a sync); **Sync all enabled schemas**; filter/sort by status, sync method, frequency; and bulk-act on selected schemas (Disable, Set frequency, Sync now, Delete & resync, Delete from PostHog). Rows being synced aren't queryable until the sync completes.

**Webhook tables:** disabling stops consuming immediately (data sent while disabled is lost); re-enabling triggers a full refresh; enabling a new one auto-adds required provider-side events (needs appropriate permissions, e.g. Stripe Write on Webhooks).

## Custom object-storage source
Link data in your own S3/GCS/Cloudflare R2:
1. Create a bucket.
2. Set up an access key + secret.
3. Add data to the bucket (optionally via Airbyte/Fivetran/Stitch — only for getting data *into* your bucket, not out of PostHog).
4. Link the table in PostHog.

This is the most self-host-portable source type since it reads from *your* storage.

## Custom REST source
For any REST API not covered by a native connector — see `/docs/cdp/sources/custom-rest-source`.

## Incoming webhooks source
Push events into PostHog from external systems via a generated URL `https://webhooks.<region>.posthog.com/public/webhooks/<UUID>`. Accepts POST (JSON body) or GET (query params); max payload 500KB (else `413`). You template the payload to PostHog's event format with the [Hog language](/docs/hog) using the `request` object (`request.method/query/body/headers/ip`) and call `postHogCapture({event, distinct_id, properties})`.

> Feature-preview / experimental. For most use cases PostHog recommends a [workflow with a webhook trigger](/docs/workflows/workflow-builder#webhook-triggers) + a capture-event action instead (no code, supports delays/branching). For controlled senders, prefer the [SDKs](/docs/libraries) or [capture API](/docs/api/capture).

## Inbound IP allow-list
Managed connectors reach your systems from fixed PostHog IPs — add them to inbound security rules:

| US | EU |
| --- | --- |
| 44.205.89.55 | 3.75.65.221 |
| 44.208.188.173 | 18.197.246.42 |
| 52.4.194.122 | 3.120.223.253 |
