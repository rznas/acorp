---
name: building-data-pipelines
description: Guides a product manager through PostHog's data pipelines / CDP — getting data IN (sources), OUT (realtime destinations, batch exports), and reshaping it on the way (transformations, hog functions). Use when a PM asks "how do I send PostHog events to Slack / a data warehouse / our CRM / an ad platform", "export events to BigQuery, Snowflake, Redshift, S3", "import Stripe / HubSpot / Salesforce / Postgres data into PostHog", "enrich, anonymize, filter, or drop events before ingestion", "set up a webhook from PostHog", "reduce event volume / billing", "stream conversions to Google/Meta/Reddit ads", or asks which integrations work on self-hosted PostHog. Covers sources, destinations, batch exports, transformations, hog functions, and integration availability.
---

# Building data pipelines (CDP) in PostHog

PostHog's data pipelines turn it into a customer data platform: pull external data **in**, push PostHog data **out**, and **reshape** events in flight. As a PM you spec *what data goes where and why* — engineers rarely need to write code because most flows are pre-built templates configured in the UI.

Everything lives under **Data pipeline** in the left sidebar (`Data pipeline` → tabs: **Sources**, **Transformations**, **Destinations**). Batch exports are created from the Destinations area too.

## The four building blocks

| Block | Direction | What it does | When a PM reaches for it |
|---|---|---|---|
| **Sources** | IN | Ingest data from CRM, payments, DBs, ads into PostHog (data warehouse) | "Join Stripe revenue / HubSpot deals to product usage" |
| **Transformations** | IN (pre-ingest) | Modify/filter/drop events *before* they're stored | "Anonymize IPs, drop bot/test events, enforce schema, cut billing" |
| **Realtime destinations** | OUT (streaming) | Send each event to a tool as it arrives | "Ping Slack on signup, send conversions to ads, enrich CRM" |
| **Batch exports** | OUT (scheduled) | Reliably bulk-export events/persons on a schedule to a warehouse | "Get all events into BigQuery/Snowflake/S3 for our analysts" |

All four are powered by **hog functions** (PostHog's lightweight function runtime). You configure them with templated inputs and filters; advanced users can edit the underlying Hog source.

## Choosing the right block (decision guide)

- **Need data analyzed alongside product data inside PostHog?** → **Source**.
- **Need PostHog data in your own warehouse/lake for your data team?** → **Batch export** (NOT a realtime destination, NOT Fivetran/Airbyte — see below).
- **Need to react to events in seconds (Slack alert, ad conversion, CRM update, automation trigger)?** → **Realtime destination**.
- **Need to clean/shrink/enrich events at the front door?** → **Transformation**.

> Self-hosted note: managed sources, realtime destinations, and batch exports all rely on PostHog-operated infrastructure (connectors run on PostHog's servers; batch exports run on PostHog-hosted Temporal workers). On a self-hosted instance the set of available managed integrations and OAuth-based connections (Slack, Stripe webhooks, ad platforms, hosted warehouse connectors) may be limited or require extra configuration. The webhook destination, incoming-webhook source, custom Hog transformations, and self-hosted object-storage sources (your own S3/GCS/R2 bucket) are the most portable options. Where the docs don't state self-host support explicitly, treat availability as "may differ on self-hosted" and verify in-app.

---

## Sources — getting data IN

Sources let you combine external data with person/event data and query across it via the [data warehouse](/docs/data-warehouse). Two kinds:

- **Managed sources** — give PostHog credentials; it runs the full extract-and-load and stores the data (e.g. Stripe, HubSpot, Salesforce, Postgres, MySQL, BigQuery, Snowflake, Zendesk, Shopify). 120+ available.
- **Self-hosted sources** — you drop CSV/Parquet into your own S3/GCS/Cloudflare R2 bucket and link it; PostHog reads it on every query. You control freshness and volume.

### Setup (high level)
Data pipeline → **Sources** → **+ New source** → pick connector → enter credentials → choose which tables/schemas to sync and a **sync method** per table.

### Sync methods (the PM-relevant trade-off)
| Method | Use when | Cost / freshness note |
|---|---|---|
| **Webhook** | Source supports it (e.g. Stripe) — real-time, captures inserts/updates/deletes | Cheapest + freshest; recommended when offered |
| **Incremental** | Only new/changed rows; needs an `updated_at`/ID replication key | Cheap; **does not capture deletions** |
| **Append only** | Immutable log tables (events) | Cheap; no row merges |
| **Full table** | Tables with frequent deletes or no incrementing field | Most expensive; reflects all changes |

CDC schemas can sync as often as every 1 minute; all other methods have a 5-minute minimum. Source status, last run, and row counts show on the Sources page.

### Don't use Fivetran/Airbyte to move data OUT of PostHog
PostHog explicitly recommends **against** ETL tools (Fivetran, Airbyte) for exporting PostHog → warehouse: they choke on high-volume product-analytics data, sync too slowly, are hard to debug, and cost more. Use **batch exports** instead.

➡️ Full source catalog (120+ connectors) and per-category list: see [reference/catalog.md](reference/catalog.md). Custom REST and incoming-webhook sources: see [reference/sources-and-ingestion.md](reference/sources-and-ingestion.md).

---

## Transformations — reshaping events before ingestion

Transformations run **during ingestion, before the event is stored**. Common PM use cases: anonymize sensitive data (e.g. mask IP last octet), enrich with geo, drop properties, filter bot traffic, enforce a schema, label events, and **reduce billing**.

Key constraint to spec correctly: transformations only see the `event` and `project` data — **no `person` or `group` properties** (those aren't resolved yet). If your logic needs person/group data, use a **realtime destination** instead. Transformations also can't make external HTTP calls.

### Drop events (billing lever)
The **Drop events** transformation discards matching events permanently. **Dropped events do not count toward usage/billing** — a fast way to cut volume without a code deploy. Filters: *Match events* (OR logic) + *Property filters* (AND logic). Decision it supports: trim noisy/test/bot events to control cost. Caution: dropped data is gone forever and won't appear in analytics, cohorts, or exports.

Setup: Data pipeline → **Transformations** → search template → configure filters → **Create & Enable**. Every transformation has built-in **Testing**, plus **Metrics**/**Logs** tabs (and a **History** audit trail on platform packages). Transformations are free and have no event limit.

➡️ Custom Hog transformations, the event-globals object, and the anonymize-IP example: see [reference/transformations-and-hog.md](reference/transformations-and-hog.md).

---

## Realtime destinations — streaming data OUT

Send each event to a tool the moment it arrives. Use cases: conversion tracking, key-event alerts to chat, enriching CRM records, triggering automations.

| PM question | Destination(s) |
|---|---|
| "Alert us in chat when X happens" | Slack, Microsoft Teams, Discord |
| "Send conversions to our ad platform" | Google Ads, Meta Ads, TikTok, Reddit, Snapchat, LinkedIn (via sources), Google Pub/Sub |
| "Update / enrich our CRM" | HubSpot, Salesforce, Attio, Intercom, Customer.io |
| "Trigger an automation" | Zapier, Make, Webhook |
| "Email / messaging tools" | Mailchimp, Klaviyo, Brevo, SendGrid, Twilio, WhatsApp, Loops, Knock |
| "Anything else / our own backend" | **Webhook** (any HTTP endpoint, fully templated payload) |

40+ destinations exist; the **Webhook** destination is the universal fallback when no native one exists.

### Setup (high level)
Data pipeline → **Destinations** → **+ New** → **Destination** → pick template → connect (OAuth or API key) → set **Filters** (event types/properties/SQL so only relevant events are sent) → optionally **template the payload** with `{event.…}` / `{person.…}` variables → **Create & Enable**.

### Reading results & keeping it healthy
- **Testing** tab: send a real event on demand to verify the integration.
- **Metrics** + **Logs** tabs: monitor volume and inspect errors.
- PostHog **retries** non-2xx responses up to 3×. A destination that is too slow or errors for a prolonged period gets **auto-disabled (quarantined)** — fix filters/config and re-enable. 4xx codes are *not* treated as failure.
- `person` reflects the profile at event-processing time — for fields set on `$identify`, filter to events at/after identification.

➡️ Full destination catalog, payload templating (`{}` + Liquid), Slack session-replay links, webhook setup: see [reference/destinations-and-export.md](reference/destinations-and-export.md).

---

## Batch exports — scheduled bulk export OUT

The reliable way to get PostHog data into a **data warehouse / lake**. Built on Temporal for resiliency (automatic + manual retries) and efficient bulk transfers.

**Supported destinations:** S3, BigQuery, Snowflake, Redshift, Postgres, Databricks, Azure Blob Storage.

### What a PM specs
- **Model**: *events* (default, immutable, append-only, deduped by UUID), *persons* (mutable identity mapping — export alongside events and join), or *sessions* (needs `$session_id` captured).
- **Frequency**: 5-minute / hourly / daily / weekly (5-min ≈ near-real-time).
- **Backfill**: export historical data via the **Backfills** tab (set start/end date).

### Reading results & gotchas
- **Runs** list shows state (Starting/Running/Failed/Completed), interval, start time, and retry/cancel controls.
- Runs are bounded by **ingestion time, not event timestamp** — late-arriving events land in later runs (so no events are missed).
- **Expect occasional duplicates** (pre-dedup runs, overlapping backfills) — plan a de-dup step in the warehouse.
- Test the config (built-in test) before/after every change; empty periods complete instantly as "Completed".

➡️ Per-destination schemas, permissions, and backfill detail: see [reference/destinations-and-export.md](reference/destinations-and-export.md).

---

## Hog functions, AI, and MCP

All blocks are **hog functions** under the hood; you can click **show/edit source code** to customize inputs (mark secrets as encrypted), templates, and logic in the [Hog language](/docs/hog). Limits: keep functions short; destinations may do at most 5 `fetch` calls.

**PostHog AI can write hog functions for you** from natural language — transformations, filters, and destinations ("Create a destination that sends high-value events to Slack", "Filter out internal traffic by IP", "Add a `user_tier` property based on plan type"). It picks inputs, event-property access, and API calls; iterate with follow-ups. Great for PMs specifying logic without writing Hog.

> Self-hosted note: PostHog AI / Max is a hosted LLM feature and may be unavailable or need extra setup on self-hosted instances.

You can also create transformations/destinations programmatically via the **Hog functions API** (enable the `Hog functions` scope on a personal API key). MCP tools exist across PostHog products for AI-assisted workflows.

## Quick workflow checklists

**Send key events to Slack/CRM/ads (realtime destination)**
1. Destinations → + New → pick template (or Webhook).
2. Connect (OAuth/API key).
3. Filters → narrow to the exact events/properties (avoid the firehose).
4. Template the message/payload with `{event.*}` / `{person.*}`.
5. Test → Create & Enable → watch Metrics/Logs.

**Get all data into the warehouse (batch export)**
1. Destinations → create batch export → pick S3/BigQuery/Snowflake/etc.
2. Create one export for **events**, another for **persons**; choose frequency.
3. Run the built-in test; **Start backfill** for history.
4. Add a de-dup step in the warehouse.

**Cut event volume / billing (transformation)**
1. Transformations → **Drop events** → + Create.
2. Match events (OR) + property filters (AND).
3. Test that the right events drop → Create & Enable. Dropped events don't bill.

## Related skills
- **using-posthog-ai-assistant** — let Max write/iterate on hog functions and answer "send X to Y" questions in natural language.
- **importing-and-querying-warehouse-data** — once a source is linked, join and query external data with product data.
- **managing-data-and-events** — event/person/group model, identify, sessions (the data these pipelines move).
- **instrumenting-product-analytics** — deciding which events to capture (so you drop/transform less later).
