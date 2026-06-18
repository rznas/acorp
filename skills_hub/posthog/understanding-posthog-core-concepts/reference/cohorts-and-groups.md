# Cohorts and groups

## Contents
- [Cohorts vs groups at a glance](#cohorts-vs-groups-at-a-glance)
- [What cohorts answer and where they're used](#what-cohorts-answer-and-where-theyre-used)
- [Static vs dynamic cohorts](#static-vs-dynamic-cohorts)
- [Dynamic cohort criteria](#dynamic-cohort-criteria)
- [Building static cohorts (CSV, snapshot)](#building-static-cohorts-csv-snapshot)
- [Cohort gotchas a PM must know](#cohort-gotchas-a-pm-must-know)
- [Groups and group analytics](#groups-and-group-analytics)

## Cohorts vs groups at a glance

| | Cohort | Group |
|---|---|---|
| Is | A **list of users** with something in common | An **entity** (org, company, project) events roll up to |
| Answers | "Which users churned?" | "Is *Company X* active? Which features do top accounts use?" |
| Setup | In-app, no code | Requires SDK code (`$groupidentify`) |
| Requires identified events | Yes | Yes |
| Use for | Filtering, breakdown, flag/experiment/survey targeting, replay playlists | Account-level (B2B) trends, insights, dashboards |

Rule of thumb: need a list of users -> **cohort**. Need account-level metrics -> **group**. Cohorts are quicker because they need no extra instrumentation.

## What cohorts answer and where they're used

Examples: users at the same company, active in the last week, churned, signed up recently, viewed signup but didn't convert. Questions like "are people at company X using our app frequently?", "what % of last week's signups still use the app?", "which features do power users use most?"

Cohorts can be used as:

- A **filter** in trends, funnels, retention, paths, stickiness, lifecycle.
- A **breakdown** dimension in trends and funnels.
- A filter on any product-analytics dashboard.
- **Targeting** for feature flags, experiments, and surveys.
- A filter for the session-replay list / playlists.
- A filter on live events (Activity) and on the People list.
- Test-account filters (exclude internal/test users).

Find them under **People → Cohorts**.

> Cohorts rely on person properties, so identified events are required to create them.

## Static vs dynamic cohorts

- **Static** — a fixed list that doesn't change unless you edit it (e.g. "users who signed up in January 2024"). Built from an insight, a CSV upload, manual selection, by duplicating a dynamic cohort, or as a **one-time snapshot from criteria**.
- **Dynamic** — auto-updates from a condition (e.g. "anyone who performed event X"). **Refreshes every 24 hours.**

Three ways to create: (1) from an insight (click a data point → **Save as cohort**), (2) **People → Cohorts → + New Cohort**, (3) via AI/MCP agent.

## Dynamic cohort criteria

**Behavioral** (time ranges in days/weeks/months/years):

- Completed / did not complete an event or action
- Completed an event or action multiple times
- Completed / did not complete a *sequence* of events or actions

**Person properties** — has / doesn't have a property, with operators `equals`, `contains`, `matches regex`, `greater/less than`, `is set`, `between`, `min`/`max` (full list incl. semver in `properties-and-filters.md`).

**Lifecycle:**

- Completed an event for the first time
- Completed an event regularly (e.g. ≥3×/week for 3 of last 5 periods)
- Stopped doing an event
- Started doing an event again

Dynamic cohorts can be built on top of existing cohorts.

## Building static cohorts (CSV, snapshot)

Identify rows by **distinct ID**, **person ID** (PostHog UUID), or **email**. Single-column CSV (distinct IDs need no header) or multi-column with a recognized header (`distinct_id`, `person_id`, `email`, etc.). If both `distinct_id` and `person_id` columns exist, `person_id` wins. The person profile for each row must already exist in PostHog.

**Criteria · One-time snapshot**: define dynamic-style criteria, and on save PostHog freezes everyone currently matching into a static list (criteria then locked). Useful for survey targeting where you want a point-in-time snapshot.

View edit history via the **History** tab on a saved cohort (who changed criteria, when).

## Cohort gotchas a PM must know

- **Feature-flag targeting:** you **cannot** directly target a dynamic cohort that has **behavioral or lifecycle** criteria with a flag/experiment/survey. Workaround: duplicate it as a static cohort, or use **Criteria · One-time snapshot**. Dynamic cohorts with **only person-property** criteria *can* be used directly.
- **CDP destination filters:** cohorts used there must be **person-property-only** (those get inlined and work in real time). Cohorts with behavioral filters error in CDP destinations (but still work fine in analytics queries).
- **Refresh lag:** dynamic cohorts update once per 24h — don't expect instant membership.
- **Groups in cohorts:** not yet supported.

## Groups and group analytics

Groups let you aggregate events by an entity rather than a user — e.g. organizations, companies, projects, instances. They're created/updated via `$groupidentify` events and `$group` properties (this needs engineering, unlike cohorts). Once set up, you can run trends, insights, and dashboards at the entity level ("active companies" rather than "active users") and view a person's **Related groups** on their profile.

Group properties are added during event processing when an event is assigned to a group. Because groups require code and identified events, scope them deliberately: they're the right tool for **B2B account-level metrics** (e.g. seats per org, feature adoption per account), not for ad-hoc user lists.
