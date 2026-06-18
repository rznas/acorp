# Properties and filter operators

## Contents
- [The four property scopes](#the-four-property-scopes)
- [Property types](#property-types)
- [Filter operators](#filter-operators)
- [Where property filters are used](#where-property-filters-are-used)

## The four property scopes

| Scope | Describes | Examples | Availability |
|---|---|---|---|
| **Event** | The event itself | `$current_url`, `$browser`, `utm_source`, custom values | Anonymous + identified |
| **Person** | The user | `email`, `plan`, `$initial_utm_source` (initial vs latest variants) | **Identified only** |
| **Group** | The account/entity | `seat_count`, `industry` | Requires group setup + identified |
| **Session** | The visit | `$session_duration`, `$entry_pathname`, `$is_bounce`, `$channel_type` | Autocaptured |

The scope you instrument determines what you can later filter, break down, and target on. Person/group properties are the common blocker — they need identified events.

## Property types

Detected during ingestion: string/text (default), boolean, date/timestamp, number, array, object. Fix a mis-detected type in **Data management → Properties → Edit** (also where you tag and verify properties).

## Filter operators

These are shared across feature flags, surveys, cohorts, and insights.

**String:** `exact`/`is not`, `contains`/`does not contain`, `matches regex`/`does not match regex`. String properties also expose semver operators.

**Numeric:** `equals`/`does not equal`, `greater than`(`or equal to`), `less than`(`or equal to`), `between`/`not between`, `minimum`/`maximum`.

**Presence:** `is set`, `is not set` (existence regardless of value).

**Date:** `is date before`, `is date after`. Values accept relative offsets (`-7d`, `-1w`, `-1m`, `-1y`, `-1h`), ISO dates (`2024-01-15`), or ISO datetimes. Relative magnitudes ≥10,000 are rejected.

**Semver** (on string properties, labeled `(semver)`) — compare versions like `1.2.3`, ideal for `$app_version`/`$lib_version` targeting:

| Op | Meaning |
|---|---|
| `=` / `≠` | exact / not equal |
| `>` `>=` `<` `<=` | version comparisons |
| `~` tilde | patch-level range (`~1.2.3` = `>=1.2.3 <1.3.0`) |
| `^` caret | minor-level range (`^1.2.3` = `>=1.2.3 <2.0.0`) |
| `*` wildcard | `1.2.*` = `>=1.2.0 <1.3.0` |

**Cohort** (cohort properties only): `in` / `not in` (membership).

### Feature-flag operator limitations

Because flag evaluation uses a separate backend:

- `between` / `not between` are **not** supported for flags — use `>=` combined with `<=`.
- `minimum` / `maximum` are auto-converted to `>=` / `<=`.

## Where property filters are used

Property filters appear throughout PostHog, most commonly in:

- **Feature flags** — release conditions targeting person/group/cohort properties.
- **Surveys** — control who sees a survey.
- **Cohorts** — dynamic membership criteria.
- **Insights** — filter trends, funnels, etc.
- **Workflows** — gate which persons/events trigger a workflow.

…and many other places wherever you filter or target by properties.
