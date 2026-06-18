# Sessions, attribution (channel/UTM), and annotations

## Contents
- [Sessions: definition and rules](#sessions-definition-and-rules)
- [Session properties](#session-properties)
- [Analyzing sessions (PM uses)](#analyzing-sessions-pm-uses)
- [UTM parameters](#utm-parameters)
- [Channel type](#channel-type)
- [Annotations](#annotations)
- [Properties and filter operators (cross-reference)](#properties-and-filter-operators-cross-reference)

## Sessions: definition and rules

A **session** is the set of events for a single use/visit. The SDK adds a `$session_id` to each event; events from the same user, browser, and device share it.

- A **new session** starts after **30 minutes of inactivity** or when the session hits the **24-hour** max.
- A session can span multiple tabs/windows on the **same** browser+device; switching browsers (Chrome→Firefox) starts a new session. `posthog.reset()` also starts a new one.
- "Activity" = any client event, including autocapture and replay activity (mouse movement). Inactivity timeout is configurable (`session_idle_timeout_seconds`).
- **Server SDKs do not set `$session_id` by default** — you must pass it through (e.g. tracing headers, or read `posthog.get_session_id()` and forward it) if you want server events to join sessions/replays.

## Session properties

Autocaptured, separate from event/person data; queryable from the `sessions` table:

| Property | Meaning |
|---|---|
| `$session_id` | Unique session identifier |
| `$start_timestamp` / `$end_timestamp` | Session start/end |
| `$session_duration` | Total duration |
| `$entry_current_url` / `$entry_pathname` | Where the session started |
| `$exit_current_url` / `$exit_pathname` | Where it ended |
| `$entry_utm_source/medium/campaign/term/content` | Attribution at session start |
| `$entry_referring_domain`, `$entry_gclid`, `$entry_gad_source` | Source at session start |
| `$pageview_count`, `$autocapture_count` | Activity counts |
| `$channel_type` | Acquisition channel (see below) |
| `$is_bounce` | Single-pageview session |

Some are also exposed on events with a `$session_entry_` prefix (`$session_entry_url`, `$session_entry_referrer`, plus UTM/gclid), constant for the whole session — handy so you can filter events by entry attribution without joining the sessions table.

## Analyzing sessions (PM uses)

- **Session replay** — watch exactly what happened in a session.
- **Web analytics** — out-of-the-box sessions, duration, bounce rate, entry/exit paths, top channels, device types.
- **Unique sessions aggregation** — pick "Unique sessions" in a trend (like "unique users") to count sessions where an event happened; combine in formulas for "sessions per user" or "events per session."
- **Filter / plot / break down** by session properties — e.g. drop sessions <10s, plot median session duration, break down by entry URL or UTM.

## UTM parameters

Five campaign parameters appended to URLs; PostHog auto-captures any present when a user visits:

| Parameter | Purpose |
|---|---|
| `utm_source` | Which site sent traffic (google, facebook) |
| `utm_medium` | Link type (cpc, social) |
| `utm_campaign` | Specific promotion/campaign |
| `utm_content` | Differentiate same link in different places |
| `utm_term` | Paid keyword |

Captured in three places: as **event** properties, as **initial + latest person** properties, and as **session entry** properties. Person UTM properties require a person profile (identified). You can capture custom campaign params via `custom_campaign_params` in `posthog-js`.

**PM use:** filter pageviews/funnels to one campaign (`utm_campaign = spring sale`) and break down by `utm_source` to see which platform converts best. You can only filter by a UTM once at least one event has carried it.

Related auto-captured attribution: `$referrer`/`$referring_domain` (event), `$initial_referrer`/`$initial_referring_domain` (person), `$entry_referring_domain` (session).

## Channel type

`$channel_type` classifies acquisition, computed from referring domain + UTM tags + ad IDs **on first landing**. Values include: Direct, Paid Search, Paid Social, Paid Video, Paid Shopping, Cross-Network, Organic Search/Social/Video/Shopping, Affiliate, Referral, Email, Display, SMS, Audio, Push, Unknown.

High-level rules: traffic counts as **Paid** if `utm_medium` is `cpc`/`cpm`/`cpv`/`cpa`/`ppc`/`retargeting`, starts with `paid`, or a `gclid`/`gad_source` is present; then source/medium/referring-domain are matched against known lists to pick the specific channel. Rules apply in order, first match wins.

**Custom channel types:** define your own in **Settings → Web analytics**; they take priority over defaults. Useful when default rules misclassify your specific sources.

**PM use:** break down acquisition or conversion by channel type for a clean marketing-attribution view without manually parsing UTMs.

## Annotations

Dated notes overlaid on trends and dashboards to explain metric changes — releases, version bumps, marketing campaigns, incidents.

- **Create:** while editing a Trends/Sessions/Dashboard chart, hover the date axis to reveal a `+`, then add the note and choose **this chart only** or **all charts**.
- Can be written after the fact (reference) or before (to assess future impact).
- Optional **emoji** for quick visual ID (🚀 release, 🔥 incident); a shared emoji replaces the count badge on the chart.
- **Show/hide** per insight or dashboard card (Trends time-series and funnel historical-trends only; not pie/table). Shown by default; setting saves to the insight.
- Manage all annotations in **Data management → Annotations** (sortable by date, scope, creator).

## Properties and filter operators (cross-reference)

Cohorts, flags, surveys, and insights share the same property-filter operators. See `properties-and-filters.md` for the full operator catalog (string/numeric/presence/date/semver/cohort) — needed when defining channel/UTM filters, cohort criteria, or flag release conditions.
