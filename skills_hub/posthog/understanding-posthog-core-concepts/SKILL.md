---
name: understanding-posthog-core-concepts
description: Explains PostHog's foundational data vocabulary for product managers — events (autocapture vs custom, actions), persons and identity resolution (anonymous vs identified, distinct IDs, merges), groups and group analytics, properties (event/person/group/session), cohorts (static vs dynamic), sessions, annotations, and channel type / UTM attribution. Use when a PM asks "what is an event/person/cohort/group/session", "why does a user show as a UUID", "why can't I filter on a person property", "anonymous vs identified", "what's the difference between cohorts and groups", "how do I segment users", "what data do I need to measure X", "what does my tracking let me analyze", or is scoping instrumentation before building insights, funnels, flags, or experiments. This is the base layer every other PostHog skill depends on.
---

# Understanding PostHog core concepts

This is the **shared vocabulary** layer. Every other PostHog capability (insights, funnels, feature flags, experiments, surveys, replay, web analytics) is built on these primitives. A PM's biggest leverage is getting the data model right *before* building anything: the way you instrument events and identity **gates what you can later measure and segment**.

## The mental model in one paragraph

A PostHog **event** is one interaction (a `$pageview`, a `signup completed`). Each event carries a `distinct_id` (who did it), a `timestamp`, and **properties** (the `$current_url`, the `plan`, the `utm_source`). Events from the same logged-in user are stitched to a **person** profile via `identify`. Persons can be grouped into **cohorts** (lists of users with something in common) and analyzed at the account level via **groups** (e.g. per organization). Events from one visit are bundled into a **session**. **Annotations** overlay context (releases, incidents) on charts. **Channel type / UTM** describe where the traffic came from.

## The single most important PM decision: anonymous vs identified

This choice determines what you can analyze for a user. Get it right at instrumentation time — you can't backfill person data onto past anonymous events.

| | Anonymous event | Identified event |
|---|---|---|
| Person profile created? | No (just a distinct ID / UUID) | Yes |
| Set/filter **person properties**? | No | Yes |
| Create / use **cohorts**? | No | Yes |
| **Group analytics**? | No | Yes |
| **Lifecycle** insight? | Excluded | Yes |
| Target **feature flags / experiments / surveys** by person property? | No | Yes |
| Trends, funnels, SQL, filter by **event** property (URL, UTM)? | Yes | Yes |
| Cost | Cheaper (single table) | More expensive (person JOINs) |

A user becomes identified when your app calls `identify` (or sets person properties / `$set`). All **future** events are then identified; **past** anonymous events are attributed to the person but stay billed as anonymous and still carry no person properties.

> If a PM ever asks "why can't I filter/break down by a person property?" or "why is this user just a UUID?", the answer is almost always: those are anonymous events and no `identify` call happened. The fix is a spec change, not a query change.

Decision rule: **identify users as soon as they log in, on every device and browser.** Use anonymous events to cut cost on high-volume pre-login traffic you only ever analyze by event property.

See `reference/identity-and-merges.md` for distinct IDs, how merges work, duplicate-profile pitfalls, and `reset()`.

## PM question -> concept map

| The PM wants to... | Reach for | Notes |
|---|---|---|
| Count an interaction (clicks, pageviews, signups) | **Event** | Autocaptured or custom. See `reference/events-and-actions.md` |
| Treat several related events as one metric | **Action** | e.g. combine 3 CTA clicks into "Homepage CTA" |
| Describe an event (which URL, which plan, value) | **Event property** | Available on anonymous + identified |
| Describe a user (email, plan, company, signup date) | **Person property** | Requires identified events |
| Build a list of users with something in common | **Cohort** | Static or dynamic; requires identified |
| Analyze at the account/company level, not per-user | **Group** | Requires SDK setup + identified |
| Analyze a single visit (entry/exit, duration, bounce) | **Session** | Autocaptured `$session_id` |
| Know where traffic came from | **Channel type / UTM** | Auto-captured on first landing |
| Mark a release/incident on charts | **Annotation** | Manual note on a date |

## Events and actions (the unit of measurement)

- **Autocapture** records frontend interactions (clicks, inputs, form submits, pageviews) with no tracking code, but only for `a`, `button`, `form`, `input`, `select`, `textarea`, `label` tags. A `<div>` styled as a button is **not** autocaptured — that's a spec gap a PM should flag.
- **Custom events** (e.g. `subscription upgraded`) are what you spec for product-specific moments; engineers send them via SDK/API.
- **Actions** combine events (autocapture + custom + pageviews + screen views) into one named metric, with filters, and are **retroactive** (apply to past events). Great as a funnel step or experiment goal.
- Where it lives: **Data management → Events / Actions / Properties**. PMs can add descriptions, tags, and mark definitions **Verified** (checkmark) or **Hidden** to keep the event catalog clean.

Full detail, autocapture matching/selectors, default property catalog, event status: `reference/events-and-actions.md`.

## Properties (how you slice everything)

Four scopes, used for filtering, breakdowns, cohorts, and targeting:

- **Event properties** — about the event (`$current_url`, `$browser`, `utm_source`, custom values). Work on anonymous + identified.
- **Person properties** — about the user (`email`, `plan`, `$initial_utm_source`). Only on identified persons. Have **initial** vs **latest** variants.
- **Group properties** — about the account/company.
- **Session properties** — about the visit (`$session_duration`, `$entry_pathname`, `$is_bounce`, `$channel_type`).

Property **filter operators** (string, numeric, presence `is set`, date with relative `-7d`, semver for version targeting like `$app_version`) are shared across cohorts, flags, surveys, and insights. Full operator list: `reference/properties-and-filters.md`.

## Cohorts vs groups (the most common confusion)

- **Cohort** = a **list of users** with something in common ("trialists who never converted"). Created in-app, no code. Use as a filter/breakdown in insights, to target flags/experiments/surveys, to filter replays.
- **Group** = aggregation by an **entity** (organization, company, project) that may span many users. Answers "is *Company X* active?" rather than "is *this user* active?" Requires extra SDK code (`$groupidentify`).

Rule of thumb: want a list of users -> cohort. Want B2B account-level metrics -> group.

Static vs dynamic cohorts, CSV upload, behavioral/lifecycle criteria, the 24-hour refresh, and the flag-targeting gotcha (dynamic behavioral cohorts can't directly target flags) are in `reference/cohorts-and-groups.md`.

## Sessions, annotations, attribution

- **Session**: events from the same user/browser/device sharing a `$session_id`. New session after **30 min inactivity** or **24 h** max. Powers replay, web analytics, bounce rate, and "unique sessions" aggregation. Server SDKs don't set session IDs by default.
- **Annotation**: a dated note shown on trends/dashboards — mark releases, campaigns, incidents to explain spikes/dips. Created by hovering the chart date axis.
- **Channel type & UTM**: the acquisition channel (Paid Search, Organic Social, Email, Direct, Referral…) is computed from referring domain + UTM tags + ad IDs **on first landing**. UTMs are stored as event, person (initial/latest), and session-entry properties — so a PM can attribute conversions back to campaigns.

Channel-type rules, full UTM behavior, custom channels, and session property catalog: `reference/sessions-attribution-annotations.md`.

## PM workflow: spec instrumentation so you can answer your questions

1. **Write the question first** ("what's trial→paid conversion by acquisition channel for users at companies with >50 seats?").
2. **Decompose into primitives**: events (trial started, paid), event/person/group properties (channel, seat count), grouping (company group).
3. **Check identity**: any per-user or person-property analysis needs **identified** events — confirm `identify` fires at login.
4. **Check group needs**: account-level metric -> spec a group + `$groupidentify`.
5. **Pick the segmentation tool**: list of users -> cohort; account rollup -> group; campaign attribution -> UTM/channel.
6. **Plan event hygiene**: name events consistently, add descriptions/tags, mark Verified, hide deprecated ones in Data management.
7. **Validate**: check **Activity** (live events) and **Data management → Ingestion warnings** (e.g. "Refused to merge an already identified user" = duplicate-profile bug).

## Self-hosted notes

The data primitives in this skill (events, persons, properties, cohorts, groups, sessions, annotations, UTM/channel) are core data-management features and behave the same on self-hosted PostHog as on Cloud per the docs.

> Self-hosted note: AI-assisted creation of cohorts and annotations via the **MCP server** / PostHog AI is referenced in the docs but its availability and setup can differ on self-hosted instances — confirm MCP/AI is enabled on your deployment. The concepts themselves do not require AI.

## AI / MCP helpers for these concepts

The PostHog **MCP server** lets an AI agent create, list, update, and delete **cohorts** and **annotations** from your dev workflow (e.g. "create a dynamic cohort 'power-users' for users who purchased >5 times", or "annotate today: deployed checkout v2.0"). This is a convenience layer over the same concepts. For setup and the broader AI assistant, see the **using-posthog-ai-assistant** skill.

## Related skills

- **using-posthog-ai-assistant** — MCP / AI agent helpers for cohorts, annotations, and querying
- **building-posthog-insights** — trends, funnels, retention, lifecycle, paths built on these events/properties
- **segmenting-with-cohorts-and-groups** — deeper cohort and group analytics usage
- **managing-feature-flags-and-experiments** — targeting by the cohorts/person properties defined here
