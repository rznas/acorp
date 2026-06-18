# Grouping exceptions into issues, fingerprints, and merging

Table of contents
- [Why grouping matters to a PM](#why-grouping-matters-to-a-pm)
- [How automatic fingerprints are generated](#how-automatic-fingerprints-are-generated)
- [Grouping precedence](#grouping-precedence)
- [Custom grouping rules](#custom-grouping-rules)
- [Client-side custom fingerprint](#client-side-custom-fingerprint)
- [Merging issues](#merging-issues)
- [Grouping best practices](#grouping-best-practices)

## Why grouping matters to a PM

An **issue** should represent a single problem fixable by a single change. Bad grouping distorts your priorities: over-grouping makes issues meaningless ("everything is one giant issue"); under-grouping inflates the issue count and hides true volume. If volume/impact numbers look wrong, grouping is usually the cause. PostHog notes its grouping algorithm is still improving — if two issues should be one (or one should be two), report it in-app.

## How automatic fingerprints are generated

Every exception gets a fingerprint; one fingerprint maps to exactly one issue. Generation is iterative:

1. Start with the **exception type**.
2. If there's **no resolved stack trace**, add the **error message**.
3. If there are stack traces but **no in-app frames** (your code vs. a dependency), use the **first frame**.
4. If there are in-app frames **with** source maps, use the **resolved in-app frames**.
5. If in-app frames but **no** source maps, use the **first in-app frame**.

Chained exceptions (e.g. Python error-causes-error) are recorded as one chain and get a single fingerprint.

Key consequence: if the exception **type or message changes** between versions, the fingerprint changes — so a "new" issue may just be a reworded old one. Accurate **source maps / resolved stack traces** are critical for consistent grouping.

## Grouping precedence

When multiple methods apply, first match wins:

1. Client-side `$exception_fingerprint` set during capture
2. Custom grouping rules defined in PostHog (during ingestion)
3. Automatic fingerprinting (fallback)

## Custom grouping rules

Configured in error tracking settings -> custom grouping. Rules run during ingestion, so the unminified stack trace is available.

Steps: Add rule -> choose **Any** (match any criterion) or **All** -> add filters on event properties -> Save.

Common filterable properties:

| Property | Event property | Description |
| --- | --- | --- |
| Exception type | `$exception_types` | Type of exception |
| Exception message | `$exception_values` | Detected message |
| Exception source | `$exception_sources` | Source file(s) |
| Exception level | `$exception_level` | Severity level |
| Exception was handled | `$exception_handled` | Handled by the app or not |

Plus any custom property set on capture. Rules evaluate in order; the first match wins, so put **specific rules first** and avoid a catch-all rule at the top (it blocks everything below). Rules can become **disabled** on ingestion errors (a banner shows the message); edit + save to re-enable.

Example PM intents this serves: group all checkout errors together; group all messages containing `postgres`; group everything for a specific feature.

## Client-side custom fingerprint

Engineering can force grouping at capture time:

```js
posthog.captureException(error, { $exception_fingerprint: "MyCustomGroup" })
```

For autocaptured exceptions, set it in the `before_send` hook before the event is sent. A PM specs the desired grouping; engineering implements the fingerprint.

## Merging issues

If PostHog created several issues that are really one problem, select them in the list and **Merge** (also available in bulk actions). After merging, multiple fingerprints point to the same issue (many-to-one). Use merging only to clean up over-splitting; prefer assignment rules or custom properties for routing/filtering rather than forcing merges.

## Grouping best practices

- Only customize grouping when default grouping creates real noise.
- Good reasons: a common error like `ClickHouseTimeout` scattered across many issues; a specific message that shouldn't vary by stack trace.
- Don't use grouping to solve routing (use auto-assignment) or filtering (use custom properties).
- Over-grouping kills specificity; keep one issue = one fixable problem.
