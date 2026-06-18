# Metrics & exposures reference

- [Metric types in detail](#metric-types-in-detail)
- [Conversion windows](#conversion-windows)
- [Outlier handling (Winsorization)](#outlier-handling-winsorization)
- [Shared metrics](#shared-metrics)
- [Exposures: what counts](#exposures-what-counts)
- [Custom exposure events](#custom-exposure-events)
- [Multiple exposures handling](#multiple-exposures-handling)
- [Traffic allocation & distribution after rollout](#traffic-allocation--distribution-after-rollout)

## Metric types in detail

Across all types, only events occurring **after a user's first exposure** are counted.

### Funnel
Measures conversion through a sequence. The **first step is always the exposure event** (`$feature_flag_called` by default, or your custom exposure event). You only add the steps that occur after exposure — e.g. for "exposure → purchase" you add just `purchase`. Significance is calculated between the first step (exposure) and the final step; intermediate steps are shown for breakdown but don't affect significance. Supports ordered and unordered funnels.

### Mean
Aggregates a chosen value per user (count, sum, or average), then takes the **mean of those per-user values** across the variant.
- "Sum of revenue" → total revenue per user, then averaged across users.
- "Average revenue" → each user's average (total / events), then averaged across users.
- **Important:** the result is a per-user mean, never a raw grand total. For a raw total, use a Trends insight outside the experiment.

### Ratio
Numerator / denominator, where the denominator can differ from exposed users.
- Revenue per purchase = total revenue / number of purchases.
- Items per session = interactions / unique sessions.
- Both parts support count / sum / average / unique. Significance uses the delta method.

### Retention
Start event + completion event + retention window (e.g. 1–7 days). Measures the share of users who performed the completion event in the window. Start handling: **First seen** (one-time actions like signup) or **Last seen** (recurring actions). Use for onboarding effectiveness, feature stickiness, re-engagement, churn reduction.

## Conversion windows
Restrict metric events to those within a time window after first exposure. Without a window, trend metrics count conversions through the experiment end date with no upper bound. Set a window matching your conversion cycle (e.g. 7 days). The window extends past the experiment end date (a 7-day window on an experiment ending March 1 counts through March 8 for users exposed March 1). If a user is re-exposed later, the window end extends from that later exposure but earlier conversions still count.

## Outlier handling (Winsorization)
Clamp metric values at configurable lower/upper percentile bounds. Values below the lower bound become the lower bound; values above the upper bound become the upper bound. **No users are excluded.** Especially useful for skewed metrics (revenue, total event counts). You can optionally **ignore zeros** when computing the upper-bound percentile — useful when many participants never fired the event, so the percentile focuses on actual positive values.

## Shared metrics
Reuse a funnel/trend metric across experiments (ideal for company metrics: conversion, revenue, churn). Create at **Experiments → Shared metrics → New shared metric**. Add to an experiment by choosing **Shared** as the metric source. Searchable by name/description/tags; tags can batch-select related metrics.

## Exposures: what counts
A user is exposed when PostHog receives a `$feature_flag_called` event containing `$feature_flag` = your flag key and `$feature_flag_response` = a variant. With PostHog SDKs this fires automatically on `getFeatureFlag()` / server-side flag reads. Only post-exposure events count toward metrics. The exposures panel shows daily cumulative count, total exposures, distribution %, and the exposure criteria — use it to confirm the experiment is live, the split matches expectation, and sample size is sufficient.

## Custom exposure events
Use when you want to count a user as exposed only when they reach the relevant part of the product, when using a non-PostHog flag system, or for server-side custom instrumentation. Configure via **Include people when** dropdown (or **Edit exposure criteria** on saved experiments) → Custom event → pick event + property filters. With custom events you must manually include `$feature/<flag-key>` = variant on the event (PostHog SDKs do this automatically for standard flags).

## Multiple exposures handling
A user may hit multiple variants (multiple devices, cleared storage, implementation bug, gradual rollout, flag evaluated before/after `identify` without persistence). Options:
- **Exclude from analysis (default, recommended):** cleanest; removes cross-contaminated users.
- **Use first seen variant:** maximizes sample size but adds noise.
Exposures are deduplicated to one per user per variant, so duplicate `$feature_flag_called` events don't inflate counts.

## Traffic allocation & distribution after rollout
Two questions: (1) *who sees what variant* and (2) *who is in the analysis*. Assignment is stable by `distinctId`.

- **Increasing rollout** only pulls new users in; existing A/B assignments don't change. Safe.
- **Revoking rollout** removes some users from seeing the test variant, but users already exposed to B still count toward B statistically (even at 0% rollout, prior exposures keep showing metrics).
- **Anti-pattern — changing split after rollout:** moves buckets, reassigns users across variants, biases analysis. Avoid.
- **Anti-pattern — adding variants after rollout:** users may flip across variants; creates multivariate users (excluded) and demands more traffic. Avoid.
- **Low volume caveat:** under ~1,000 users/variant, exposure between variants can differ by up to 20% (e.g. 800 vs 1,000). PostHog's calculations account for this discrepancy.
- **Best practice:** equal split, no changes after release other than increasing total rollout, no biased placement or inclusion criteria.
