# Destinations & batch exports details

Sending PostHog data OUT — realtime streaming destinations and scheduled batch exports. See SKILL.md for the overview and catalog.md for the full destination list.

## Table of contents
- [Realtime destination setup](#realtime-destination-setup)
- [Payload templating](#payload-templating)
- [Webhook destination](#webhook-destination)
- [Slack destination](#slack-destination)
- [Health, retries, auto-disable](#health-retries-auto-disable)
- [Batch exports](#batch-exports)
- [Batch export models](#batch-export-models)
- [Backfills](#backfills)
- [Duplicates & timing gotchas](#duplicates--timing-gotchas)

## Realtime destination setup
Data pipeline → **Destinations** → **+ New** → **Destination** → search template → **+ Create** → connect (OAuth or API key) → set **Filters** (event type / properties / SQL — only matching events are sent) → set **Trigger options** (immediate vs scheduled, where supported) → template the payload → **Create & Enable**. Use the **Testing** tab to send a real event on demand.

## Payload templating
Insert event/person/group/project data with curly-brace templating, e.g.:
```
{event.event} was triggered by {person.properties.first_name} {person.properties.last_name}
```
Use in webhook URLs (`?username={person.name}`) or custom JSON bodies. **Liquid templating** is also supported for transforming values (e.g. string→number for Google Ads: `{% assign v = v | plus: 0 %}{{ v }}`).

Available global object: `event` (uuid, event, distinct_id, properties, timestamp, url), `person?` (id, name, url, properties), `groups?`, `project` (id, name, url), `source?`. `person` reflects the profile **at event-processing time** — for `$identify`-set fields, filter to events at/after identification.

Advanced: click **show source code** to edit inputs (mark secrets as encrypted) and the Hog implementation. Most destinations wrap `fetch` (retry-able HTTP). Keep functions short; **max 5 `fetch` calls** per function.

## Webhook destination
Universal fallback — send events to any HTTP endpoint with any method/URL params/JSON body. Setup: Destinations → + New → Destination → "Webhook" → enter **Webhook URL** (POST + templated JSON by default) → Create & Enable → test. Test with webhook.site or ngrok; enable **Log responses** to debug.

## Slack destination
Sends custom messages to Slack on events or person updates. Steps: create destination → choose source (events vs person updates) + filters + trigger options → **Choose Slack connection** (OAuth-install the PostHog Slack app) → add the PostHog app to the target channel → template the message with Slack [block kit](https://docs.slack.dev/block-kit/) or plain text using `{person.name}` / `{event.event}`.

**Add a session-replay button** by templating the URL `{project.url}/replay/{event.properties.$session_id}?t={event.timestamp}` (requires `$session_id` on the event and an associated recording). For scheduled insight/dashboard digests use [subscriptions](/docs/product-analytics/subscriptions) instead.

> Self-hosted note: Slack uses an OAuth app connection; the docs state Slack requires a PostHog Cloud account. On self-hosted, native Slack/OAuth destinations may be unavailable — fall back to the Webhook destination posting to a Slack incoming webhook URL.

## Health, retries, auto-disable
- **Metrics** + **Logs** tabs monitor volume and errors.
- Non-2xx responses are retried up to 3×. **4xx is not treated as failure** (some APIs use 409 etc. as signals), but non-OK statuses are logged.
- A destination that errors too much or is too slow for a prolonged period is **quarantined and eventually auto-disabled**; PostHog tries to re-enable temporarily before stopping. Fix filters/config and re-enable in the UI.
- No hard event limit, but the destination must respond healthily and promptly.

## Batch exports
Reliable, scheduled bulk export to a warehouse/lake, built on Temporal (resilient retries, efficient bulk transfers). **Destinations:** S3, BigQuery, Snowflake, Redshift, Postgres, Databricks, Azure Blob Storage.

- Create from the Destinations area; pick destination + frequency (5-min / hourly / daily / weekly) + model.
- **Test** the config (built-in test detects bad credentials/permissions) before and after every change.
- Workers run on **PostHog infrastructure**; only encrypted config (never event/person/session data) is sent to Temporal's coordination servers.

> Self-hosted note: batch exports depend on PostHog-hosted Temporal workers and infrastructure. Availability/operation on self-hosted may differ — verify in-app.

## Batch export models
| Model | Nature | Notes |
|---|---|---|
| **Events** (default) | Immutable, append-only | Each event has a UUID for de-dup. Schema varies slightly per destination — check the destination's doc. |
| **Persons** | Mutable | Maps all anonymous + identified `distinct_id`s to one ID per user via `identify`. Requires higher destination access (merge, not just append). Export alongside events and join on person. |
| **Sessions** | Per `session_id` | Requires capturing `$session_id` (automatic in JS web + mobile SDKs; manual for server SDKs). |

New fields aren't auto-added to existing DB destination tables — add the column manually to populate future exports (S3 picks up new fields automatically).

## Backfills
Export historical data: destination → **Backfills** tab → **Start backfill** → set start/end date. Start date auto-adjusts to earliest available data; end can't be in the future. Empty ranges complete instantly. The Backfills table shows estimated/final row counts. Backfills can be cancelled (also cancels associated runs).

## Duplicates & timing gotchas
- **Batch runs are bounded by ingestion time, not event timestamp** — late-arriving events appear in later runs, so none are missed (but a run won't match the event's own timestamp window).
- Runs can occasionally start before async de-dup completes → **duplicates**. Re-running/overlapping backfills also duplicate. **Plan a de-dup step in the warehouse** (use each destination's tooling).
- A small randomized start delay is added per run to spread load.
- No data for a period → run succeeds as "Completed" (not a failure); but with no data a run may skip connecting, hiding config issues — so always run the built-in test after changes.

## Don't use Fivetran/Airbyte to export PostHog data
PostHog recommends against ETL tools for PostHog→warehouse: they can't handle high-volume, bursty product-analytics data, sync too slowly (hourly/daily), are hard to debug (black box), copy data multiple times, and cost more. Batch exports read directly from PostHog's ClickHouse, scale to billions of events/day, output compressed Parquet, offer 5-minute freshness, retry automatically, and are cheaper. Migration: create the batch export with the same warehouse credentials → test → start → verify data → disable the ETL tool.
