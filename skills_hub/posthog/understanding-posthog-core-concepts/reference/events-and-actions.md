# Events and actions reference

## Contents
- [Anatomy of an event](#anatomy-of-an-event)
- [Autocapture vs custom events](#autocapture-vs-custom-events)
- [Default properties PostHog captures](#default-properties-posthog-captures)
- [Timestamps: `timestamp` vs `created_at`](#timestamps-timestamp-vs-created_at)
- [Actions: combining events into one metric](#actions-combining-events-into-one-metric)
- [Managing event definitions (PM hygiene)](#managing-event-definitions-pm-hygiene)
- [Deduplication and editing events](#deduplication-and-editing-events)

## Anatomy of an event

An event is the core unit of data. It consists of:

1. **`event`** — name like `$pageview` or `query completed`. Names starting with `$` are PostHog defaults.
2. **`distinct_id`** — unique identifier for the person (commonly a `uuidv7`).
3. **`timestamp`** — ISO 8601 time the event occurred (usually `now()`, can be in the past for imports).
4. **`properties`** — key/value data (`color`, `$current_url`). Properties starting with `$` are defaults.

Events are immutable once written to ClickHouse. View live events (auto-refreshing every 30s) in the **Activity** tab.

**PM takeaway:** much of PostHog is driven by events under the hood — `$identify` creates/links persons, `$groupidentify` updates groups, `$feature_flag_called` powers experiment exposure, `$pageleave` powers bounce rate. When you "spec a metric," you're really specing an event + properties.

## Autocapture vs custom events

| | Autocapture | Custom event |
|---|---|---|
| Code needed | None (frontend SDK) | Engineer sends via SDK/API |
| Captures | Clicks, inputs, form submits, pageviews/screen views | Anything product-specific (`subscription upgraded`) |
| Best for | Quick coverage, retroactive analysis of UI | Business-meaningful moments, server-side actions |

**Autocapture limitation a PM must know:** it only captures interactions on `a`, `button`, `form`, `input`, `select`, `textarea`, and `label` tags. A `<div>` acting as a button is **not** captured automatically. Autocapture is conservative on `input` tags to avoid sensitive data. If a key interaction uses non-standard HTML, spec a custom event or define it manually.

Autocapture also covers navigation, clipboard, heatmaps, dead clicks, web vitals (and, separately, session replay and exception capture). It's enabled by default on JavaScript Web and React; off by default on React Native and Flutter; partial on mobile.

## Default properties PostHog captures

Client-side SDKs attach many properties automatically. The most PM-useful:

| Name | Key | Example |
|---|---|---|
| Current URL | `$current_url` | `https://example.com/page` |
| Path | `$pathname` | `/page` |
| Host | `$host` | `example.com` |
| Browser / Device | `$browser`, `$device_type` | `Chrome`, `Desktop` |
| OS | `$os` | `Mac OS X` |
| Referrer | `$referrer`, `$referring_domain` | `https://google.com` |
| UTM | `$utm_source`, `$utm_medium`, `$utm_campaign`, `$utm_term`, `$utm_content` | `newsletter`, `email` |
| Ad click IDs | `$gclid`, `$fbclid`, `$msclkid`, etc. | for paid attribution |
| Active feature flags | `$active_feature_flags` | `['beta_feature']` |
| Event type | `$event_type` | `click` |

PostHog detects a **type** per property (string, boolean, date/timestamp, number, array, object). If detected wrong, fix it in **Data management → Properties → Edit**, where you can also tag and verify properties.

## Timestamps: `timestamp` vs `created_at`

- `timestamp` — when your app made the capture call (defaults to `now()`; can be backdated). **Build insights and queries on `timestamp`.**
- `created_at` — when PostHog's server received it. Only use to investigate ingestion delay between the two.

## Actions: combining events into one metric

An **action** treats several related events as a single thing you can use in insights, dashboards, funnels, and as an experiment goal.

- Sources: autocapture elements, pageviews (string/exact/regex URL match), screen views (mobile `$screen_name`), and custom events.
- Filters: event properties, person properties, feature flags, cohorts, autocapture elements (CSS selectors).
- **Retroactive**: applies to past events too, not just future ones.
- Also useful to "rename" a single noisy event into a friendly name.

Examples from the docs: combine `insight created`, `insight analyzed`, `dashboard created` into a `Product Analytics Interactions` action; combine all signup CTAs into a `Homepage CTA` action to track conversions and run A/B tests against it.

Create via **Data management → Actions → New action**, or visually with the **Toolbar**.

Autocapture matching supports CSS selectors: attribute selectors `[data-attr="value"]` (recommended), ID `#id`, class `.class`, type `input`, and combinators. Not supported: wildcards (`*`, `~`, `|`), combining multiple attribute selectors, non-alphanumeric classes. Multiple match fields combine with AND.

## Managing event definitions (PM hygiene)

In **Data management → Events** a PM can curate the catalog:

- Edit description and tags (these surface everywhere the event is used, e.g. the insight builder).
- Set status: **Visible** (default), **Verified** (adds a checkmark — signals the canonical event to use), or **Hidden** (removes from search to retire old events without deleting data).
- Attach an **image preview** to a custom event so teammates know which screen/moment it fires when picking events for insights/experiments.

This keeps a large team from building insights on the wrong or stale events.

## Deduplication and editing events

Every event has a `uuid`. Events sharing the same `uuid`, `event` name, `timestamp`, and `distinct_id` are de-duplicated (eventually, during ClickHouse merges). SDKs handle retries for you. You can intentionally **upsert** an event by re-sending with the same four keys — but the re-ingested event fully replaces the original, so include **all** properties you want, keep the `timestamp` identical, and omit `$set`/`$set_once` to avoid clobbering current person properties. Downstream destinations/exports may re-trigger.
