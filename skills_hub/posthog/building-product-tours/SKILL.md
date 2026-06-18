---
name: building-product-tours
description: Explains the status of PostHog Product Tours (in-app guides, tooltips, walkthroughs, guided onboarding flows) and routes product managers to the supported PostHog alternatives for new-user onboarding and feature adoption. Use when a PM asks how to build a product tour, in-app guide, onboarding walkthrough, tooltip flow, or interactive checklist in PostHog, how to onboard new users, or how to drive adoption of a feature. Clarifies that the Product Tours docs have been removed from PostHog and points to surveys (in-app popovers), feature flags (gated rollout and targeting), and product analytics/funnels for measuring adoption instead.
---

# Building product tours in PostHog

## Status: Product Tours docs have been removed

The PostHog docs directory for Product Tours (`/docs/product-tours`) no longer contains feature documentation. The only remaining file is an explicit stub:

> "Product Tours has been removed from the site, but a cached MDX scope from earlier builds still references this path."

Treat this as authoritative: **there is no current, documented PostHog "Product Tours" product** to spec against. Do not describe a product-tours builder, tour steps, tooltip sequencing, or hotspot UI as if it exists in PostHog — none of that is in the docs.

> Self-hosted note: Because the feature is undocumented/removed, its availability on self-hosted PostHog cannot be confirmed from the docs. Assume it is not available; do not direct a self-hosted PM to a Product Tours UI.

## What a PM actually wants — and how to do it with supported features

PM goals behind "build a product tour" are usually: **onboard new users** and **drive adoption of a feature**. PostHog has documented, supported building blocks for both. Map the underlying goal to the right feature:

| PM goal / question | Supported PostHog feature | What it does |
|---|---|---|
| Show an in-app message, tip, or prompt to users | Surveys (popover display) | Render an in-app popover targeted by URL, event, person property, or feature flag |
| Announce or nudge adoption of a new feature in-app | Surveys (popover) + targeting | Trigger a contextual message to the right segment at the right moment |
| Gradually expose a new feature / control who sees onboarding | Feature flags | Gate the feature or the onboarding experience; roll out by %, cohort, or property |
| Decide who is "new" and should be onboarded | Feature flags + cohorts/person properties | Target by signup date, plan, or behavior |
| Measure whether onboarding worked / feature got adopted | Product analytics (funnels, retention, trends) | Quantify activation and adoption to inform the decision |

### 1. In-app messaging via Surveys (closest supported substitute)

PostHog Surveys can be displayed as **in-app popovers**, which is the supported way to deliver an in-app message without writing custom UI. A PM can use a survey popover to deliver an onboarding nudge or feature announcement (and optionally collect a response).

- Where it lives: **Surveys** in the PostHog UI. See the `surveys` docs (`/docs/surveys`).
- High-level setup: create a survey, choose the popover presentation, and set **display conditions** (URL, events, person/group properties, or a linked feature flag) so it appears only to the target segment at the right moment.
- How to read the result: survey response/impression data tells you who saw it and how they reacted; pair with analytics to see downstream behavior.
- Decision it supports: whether the in-app nudge is reaching and converting the intended new-user/adoption segment.

### 2. Targeted rollout via Feature flags

To control who experiences an onboarding flow or new feature, use feature flags.

- Where it lives: **Feature flags** in the PostHog UI (`/docs/feature-flags`).
- High-level setup: create a flag, set release conditions (percentage rollout, cohort, or person/group properties), and gate the onboarding experience or feature behind it.
- How to read/interpret: flag evaluation and exposure data show which users are in the rollout.
- Decision it supports: expand, hold, or roll back the rollout based on adoption.

### 3. Measure onboarding & adoption with Product analytics

Use funnels (activation steps), retention, and trends to quantify whether onboarding and adoption are working. See `/docs/product-analytics`.

## Workflow checklist for a PM (onboarding / adoption without Product Tours)

1. Define the goal: new-user activation, or adoption of a specific feature.
2. Define the target segment (new users by signup date, plan, cohort, or behavior) — express it as feature flag conditions and/or cohorts.
3. Deliver the in-app prompt via a **Survey popover** with display conditions matching the segment and trigger moment.
4. Optionally gate the underlying feature or experience with a **feature flag** for controlled rollout.
5. Define the success metric and build a **funnel/retention** insight to measure activation/adoption.
6. Read the result, then decide: iterate the messaging/targeting, expand the rollout, or stop.

## AI / MCP helpers

PostHog has AI helpers for the substitute features that may speed PM work:
- `create-surveys-ai` and `summarize-responses-ai` (surveys) — draft a survey and summarize responses.
- `create-flags-mcp` (feature flags) — create flags via MCP.

Briefly: these let a PM generate a survey or flag and interpret results faster. For details on PostHog's assistant capabilities, see the **using-posthog-ai-assistant** skill.

> Self-hosted note: PostHog AI / Max and some hosted LLM features may be limited or require extra configuration on self-hosted instances; confirm availability for your deployment before relying on them.

## Related skills

- **running-surveys** — in-app popovers, display conditions, response analysis (the primary substitute for in-app guides).
- **managing-feature-flags** — targeted, gradual rollout of onboarding experiences and features.
- **analyzing-product-funnels** / **measuring-retention** — quantify onboarding activation and feature adoption.
- **using-posthog-ai-assistant** — AI/MCP helpers for surveys and flags.
