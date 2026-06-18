---
name: analyzing-product-usage
description: Helps a product manager pick and read the right PostHog Product Analytics insight to answer product-usage questions. Use when someone asks about feature adoption, active users (DAU/WAU/MAU), conversion or drop-off in a flow, retention curves and product-market fit, feature stickiness or engagement depth, user journeys and navigation paths, product health (new/returning/resurrecting/dormant), what correlates with conversion, or how a metric changed over time. Covers trends, funnels, retention, user paths, stickiness, lifecycle, correlation analysis, breakdowns, formulas, aggregations, group (company/account-level) analytics, confidence intervals, and chart types. Maps PM questions to the correct insight, explains setup and how to interpret results, and supports decisions. Self-hosted aware.
---

# Analyzing product usage in PostHog

This skill maps a PM's product question to the right PostHog **insight** type, explains where it lives, how to read the result, and what decision it supports. Insights visualize **events** and **actions** and live under **Product Analytics** in the left sidebar (`New insight`, or `+` next to "Product analytics"). One insight can sit on many **dashboards**.

## Pick the insight: PM question → insight type

| PM question | Insight | Reads as |
|---|---|---|
| How is this metric trending? How many DAU/WAU/MAU? How does feature usage compare side-by-side? | **Trends** | Line/bar/number over time, or total value |
| Where do users drop off in a flow? What's the signup→activation conversion? | **Funnels** | % converting per step + drop-off |
| Are new users coming back? Did a change improve return rate? Do we have PMF? | **Retention** | Cohort table / curve, % retained per period |
| How do users navigate? Where do they get stuck or fail to discover features? | **User Paths** | Sankey of top step-to-step transitions |
| How deeply do users engage? Who are my power users? | **Stickiness** | # users by how many times/days they did an event |
| Is my product healthy under the surface (leaky bucket)? | **Lifecycle** | New / Returning / Resurrecting / Dormant bands |
| What factor (no hypothesis yet) drives conversion in this funnel? | **Correlation analysis** | "Nx more/less likely to convert" per event/property |
| Free-form table / anything the above can't express | **SQL insight** | Custom query results |

Cross-cutting tools that apply to most insight types: **breakdowns** (split by a property/cohort), **formulas** (combine series math), **aggregations** (how a series is counted), **group analytics** (aggregate by company/account, not user). See reference files below.

> Self-hosted note: All six core insight types (trends, funnels, retention, paths, stickiness, lifecycle), breakdowns, formulas, aggregations, and confidence intervals are marked `selfServe: full` in the docs and work on self-hosted. Some sub-features are paid/cloud-gated: **correlation analysis** is `free: none` (needs a paid plan), **group analytics** is a paid add-on (`free: none`), and paths **cohort filters / wildcard groups / advanced options** are not on the free tier. PostHog AI / Max and MCP-driven analysis are hosted AI features — confirm availability on your self-hosted instance (see the AI section).

## Reading each insight (PM cheat sheet)

### Trends — "how is this changing?"
- **Setup:** pick an event/action, choose an **aggregation** (Total count, Unique users, DAU, WAU, MAU, Count per user, Unique sessions, Unique groups, or property math like average/percentiles). Add multiple series to compare features; combine events inline (logical OR) without making an action.
- **Read:** line shows shape over time; **number** chart shows total + "Compared to previous period" delta; dotted line = period still collecting. Add **confidence intervals** (Options) for noisy/small data — non-overlapping bands between two lines ≈ likely significant.
- **Decides:** is the metric up/down, is a feature gaining traction, did a release move the number.
- Detail: [reference/trends.md](reference/trends.md), aggregations + percentiles in [reference/aggregations-and-formulas.md](reference/aggregations-and-formulas.md).

### Funnels — "where do users drop off?"
- **Setup:** add ordered steps (first event → success event). Choose step order: **Sequential** (B after A, anything between), **Strict order** (B immediately after A), **Any order**. Optionally **exclusion steps** (drop users who did X between two steps) and per-step **first-ever / first-matching** occurrence. Funnel steps are *actions to perform*, not filters — two identical steps require two events.
- **Read:** **Overall conversion** = each step vs step 1 (whole-funnel view); **Relative to previous step** = the per-step view that surfaces the biggest fixable leak. Graph types: **Conversion steps** (drop-off), **Time to convert** (friction — long = hard step), **Historical trends** (did conversion improve over time; enable "Hide incomplete periods").
- **Act:** click a step's count to see the people who **completed** or **dropped**, save them as a cohort, watch their session replays, or "show user paths" before/after/around the drop.
- **Decides:** which step to fix first, whether a fix worked, who to talk to.
- Breakdown nuance (attribution: first/last touchpoint, all steps, specific step) and seasonality in [reference/funnels.md](reference/funnels.md).

### Retention — "are they coming back?"
- **Setup:** a **start event** (defines the cohort) and a **return event** (proves they came back) — often the same. Pick period (hour/day/week/month) and **first time / first ever / recurring** cohorting.
- **Read:** cohort table — column = period, row = a start cohort, Week/Day 0 always 100%. Curve that flattens = users found durable value; curve to zero = leaky. Choose reference **Period 0** vs **Previous period**, returning criteria **On** (came back that exact period) vs **On or after** (rolling/unbounded = inverse of churn), and **weighted vs simple mean** across cohorts.
- **Decides:** product-market fit signal, whether onboarding/feature changes lift return rate; breakdown by country/plan/feature-flag to see who retains best.
- Detail incl. relative retention and breakdown rules: [reference/retention.md](reference/retention.md).

### User Paths — "how do they move through the product?"
- **Setup:** choose event types (page views by `Current URL`, screen views by `$screen_name`, custom events, or SQL). Set start/end step, number of steps, min/max people per path, exclusions, **wildcard groups** (`/product/*`) and **path cleaning** to collapse high-cardinality URLs.
- **Read:** each node = a page/event, numbers = distinct users on that transition. Paths show the **top 50 transitions only**, count transition frequency (not unique users), cap at the step count, and split on 30-min session gaps — so **path numbers won't match funnel numbers** and shouldn't. Use paths to *discover* patterns, then validate the exact flow with a funnel.
- **Act:** click a step's number → users → save cohort / view recordings; "View funnel" to turn a step into a funnel.
- Detail: [reference/paths.md](reference/paths.md).

### Stickiness — "how engaged are they?"
- **Setup:** pick an event and a period.
- **Read:** x-axis = number of times/days the user (or group) did the event in the period; y-axis = how many users hit exactly that count. It measures *frequency* (vs retention's came-back-at-least-once). 938 people did a pageview on exactly 3 of the last 7 days.
- **Decides:** identify power users (click a high-count bar → save as cohort), and whether engagement depth is rising month over month.

### Lifecycle — "is the product healthy?"
- **Setup:** pick a repeated activity event and a time interval matching the product's natural frequency.
- **Read:** above zero = active — **New** (blue, created + active this interval), **Returning** (green, active last interval too), **Resurrecting** (purple, was inactive then came back); below zero = **Dormant** (red, active last interval, not this one). Anonymous events excluded; lifecycle starts when a user is identified.
- **Watch for:** returning should grow or hold (else leaky bucket); too many resurrecting often means your interval doesn't match the real usage cadence.
- **Decides:** whether growth is real retention or churn-and-replace; where to dig next (retention/stickiness).
- > Self-hosted note: group aggregation is supported in lifecycle only via group analytics, but **group types are not supported for lifecycle or paths** per the docs.

### Correlation analysis — "what drives conversion when I have no hypothesis?"
- Lives inside a **funnel** ("Correlation analysis" section → Load results). Auto-surfaces events/properties where users are e.g. "8.0x more likely to convert." Best when you don't already know which property to break down by. Accuracy depends on sample size; exclude noisy events. Save converters/droppers as cohorts.
- > Self-hosted note: paid feature (`free: none`); available on self-serve/enterprise plans.

## Cross-cutting techniques

- **Breakdowns** — split any insight by event/session/person/group property, feature flag, cohort, or SQL. Trends allow up to 3 breakdowns; other insight types allow 1 (funnels: a single cohort). Numeric properties can be **binned**. See [reference/breakdowns-and-groups.md](reference/breakdowns-and-groups.md).
- **Group analytics** — answer questions at the **company/account/channel/project** level instead of per user (daily active *companies*, company churn, B2B onboarding funnel, per-account retention). Set "Aggregating by" to a group type. Paid add-on; max 5 group types. See [reference/breakdowns-and-groups.md](reference/breakdowns-and-groups.md).
- **Formulas** — combine series with math (`A / B` for a ratio/rate). Enable formula mode; series are referenced by letter. See [reference/aggregations-and-formulas.md](reference/aggregations-and-formulas.md).
- **Chart types** — 10 trends visualizations (line, cumulative line, bar, area, box plot, number, value bar, table, pie, world map; calendar heatmap in beta). Picking the right one is part of answering the question — see [reference/chart-types.md](reference/chart-types.md).

## Spec-a-metric checklist (for a PM defining an insight)

1. State the **question and the decision** it informs.
2. Choose the **insight type** from the table above.
3. Name the exact **event(s)/action(s)** (insist on explicit growth events like `user_signed_up` — autocapture alone makes funnels/retention unreliable).
4. Choose **aggregation** (Total vs Unique users vs DAU/WAU/MAU vs property math) and **time range + grouping**.
5. Add **breakdown** (or correlation) to segment; decide if the question is per-user or per-**group**.
6. Pick the **chart type** that reads cleanly for the audience.
7. Set conversion/return/order options correctly (funnel order & attribution; retention reference & criteria).
8. Sanity-check: filter out internal/test users; confidence intervals for small samples; remember paths ≠ funnels by design.
9. Save → add to **dashboard** / notebook; optionally **subscribe** (email/Slack) or set an alert on a trend metric.

## Analyze with AI and MCP (PM value)

- **PostHog AI / Max** builds any insight type, full dashboards, alerts, and runs correlation from a natural-language prompt (e.g. "What changed across my product this week?", "Show me a funnel from sign up to first project", "Which events have the highest drop-off this week?"). Name events explicitly and specify time ranges for best results.
- **PostHog MCP server** lets a coding/agent client run queries, generate HogQL, and create/update insights and dashboards from the editor (`query-run`, `insight-create-from-query`, `dashboard-create`, …).
- > Self-hosted note: these are hosted AI features; confirm enablement on your self-hosted instance. For the assistant/Max workflow, see the **using-posthog-ai-assistant** skill.

## Related skills

- **using-posthog-ai-assistant** — PostHog AI/Max and MCP for generating and investigating insights.
- **building-dashboards-and-sharing** — assembling insights into dashboards, subscriptions, alerts, sharing/embedding.
- **managing-data-and-cohorts** — events, actions, persons, properties, cohorts, group setup, identity resolution that feed these insights.
- **running-experiments** — A/B tests that build on funnels/trends and group-level metrics.
