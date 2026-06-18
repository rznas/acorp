---
name: analyzing-customer-health
description: Guides product managers using PostHog Customer Analytics to assess account health, spot churn risk and expansion opportunities, and understand the customer journey. Covers the customer analytics dashboard (active users DAU/WAU/MAU, power users, signups, free-to-paid conversion, lifecycle engagement), customer profiles for persons and groups, usage metrics, customer journeys, saved views for customer lists, and B2B mode. Use when a PM asks "how healthy is this account?", "who is at risk of churning?", "who are my power users?", "which accounts can I expand?", "what stage is this customer in?", "how do I segment customers by usage?", or wants to build a single 360 view of a customer including events, errors, LLM traces, and support tickets.
---

# Analyzing customer health with PostHog Customer Analytics

PostHog **Customer Analytics** (beta, free) gives a PM a fast, opinionated read on customer health without building dashboards from scratch. It answers PMF-stage and account-management questions: who is active, who is a power user, who is converting, who is at risk, and what each customer has actually done.

> Self-hosted note: These docs do not state self-host limitations for the core dashboard, profiles, usage metrics, journeys, or saved views — they should work on a self-hosted instance. The exceptions are AI-assisted features (PostHog AI / Max) and managed data warehouse sources (e.g. Zendesk), which are typically cloud-managed; availability and setup may differ on self-hosted. B2B mode additionally requires the group analytics add-on.

## PM question -> feature map

| PM question | Feature / insight | Where it lives |
| --- | --- | --- |
| Is the product growing / sticky? | Active users (DAU/WAU/MAU) | Customer analytics dashboard |
| Who are my most engaged users to talk to? | Power users | Dashboard (top activity, last 30 days) |
| Is my acquisition + signup flow working? | Signups & signup conversion rate | Dashboard |
| Does the product prove its value? | Paying customers & free-to-paid conversion | Dashboard |
| Am I retaining or leaking users? | Engagement (lifecycle breakdown) | Dashboard |
| What's the full story on one customer? | Customer profile | Click a customer / Persons / Groups |
| Is this account at risk or expanding? | Usage metrics on the profile | Profile + Settings > Customer Analytics |
| Where is this customer in conversion? | Customer journeys (journey node) | Journeys + profile |
| How do I segment & save a customer list? | Saved views | Persons / Groups list |
| Account-level (company) health, not per-user? | B2B mode + group profiles | Dashboard toggle (needs group analytics) |
| What events should I even track? | Dashboard event config (+ PostHog AI) | Customer analytics > configuration |

## The dashboard: reading the 5 core metrics

The [customer analytics dashboard](https://app.posthog.com/customer_analytics) works immediately with `$pageview` events. To make it meaningful, configure your real events first (see config below). How to interpret each insight and what decision it supports:

| Insight | What it shows | How to read it | Decision it supports |
| --- | --- | --- | --- |
| **Active users (DAU/WAU/MAU)** | Unique users who fired the activity event in the period | Watch the *trend*, not the absolute. Week-over-week growth = building something people return to; flat/declining = not sticky yet | Keep building vs. fix retention before scaling |
| **Power users** | Users who fired the activity event the most in last 30 days | Your "canary" — a small, intense group is a scaling signal | Pick users to interview; find your advocates |
| **Signups & signup conversion** | New accounts + % of signup-page viewers who sign up | Signups alone are a vanity metric; low conversion = signup-flow friction | Fix positioning/targeting vs. fix the signup flow |
| **Paying customers & free-to-paid conversion** | Count on paid plan + % of free users converting | Strongest PMF signal; conversion rate = how well product demonstrates value | Validate pricing/value; prioritize activation |
| **Engagement (lifecycle)** | Customers split by [lifecycle](/docs/product-analytics/lifecycle) stage (new, returning, resurrecting, dormant) | Healthy = returning stable/up, dormant low. "Leaky bucket" = dormant growing faster than returning. Returning plateau = stickiness ceiling | Whether to fix activation/retention before spending on acquisition |

The four lifecycle patterns to flag (healthy growth, leaky bucket, resurrection spikes, returning plateau) are detailed in [reference/dashboard-metrics.md](reference/dashboard-metrics.md).

### Configuring dashboard events (required for real signal)

Set these in **Customer analytics > configuration** (or Settings > Customer Analytics):

- **Activity event** — what makes a user "active." Avoid raw `$pageview`; narrow by URL/property or use an action grouping product events. PostHog tracks activity as an action spanning many product events (e.g. `insight created`, `feature flag created`).
- **Signup pageview event** — pageviews of the signup page (drives signup rate + conversion time).
- **Signup event** — account creation. Keep distinct from login/sign-in (signup happens once).
- **Subscription event** — user enrolls in a plan.
- **Payment event** — successful payment (drives paying customers + free-to-paid conversion).

Tips: use **actions** to group multiple events under one name; be specific ("sloppy definitions = sloppy metrics"). Full event-config guide: [reference/dashboard-config.md](reference/dashboard-config.md).

## Customer profiles: the 360 view

A **customer profile** consolidates everything about one customer in a single view. Open it by clicking a customer in the dashboard, or from [Persons](https://app.posthog.com/persons) / [Groups](https://app.posthog.com/groups/0). It pulls from:

| Data | Source | Requirement |
| --- | --- | --- |
| Events | Product analytics | Automatic |
| Errors | [Error tracking](/docs/error-tracking) | Error tracking enabled |
| LLM traces | [LLM observability](/docs/ai-engineering/observability) | LLM observability enabled |
| Support tickets | Zendesk | Zendesk data warehouse source connected |
| Usage metrics | (your definitions) | Metrics defined |
| Customer journey node | Customer journeys | Journey configured |

- **Person profile** = one user's activity. **Group profile** = aggregated company/org activity — use this for B2B account health.
- **Support tickets**: connect [Zendesk as a data warehouse source](/docs/cdp/sources/zendesk). For persons, tickets match by the person's email; for groups, set the Zendesk org `external_id` to the PostHog `groupKey`. Decision support: see *what they did, what broke, and what they asked for help with* in one place.

> Self-hosted note: Zendesk is a managed data warehouse source; AI-summarized profile context relies on PostHog AI. Both may behave differently or require extra setup on self-hosted — verify availability.

## Usage metrics: quantifying account health

Usage metrics turn any event (or a Data Warehouse table row) into a per-customer activity measurement shown on profiles and in customer lists. This is the core mechanism for **churn risk** (usage dropping) and **expansion** (usage climbing).

Create from **Settings > Customer Analytics > Add usage metric** or from any profile. Key options:

- **Interval**: 7 / 30 / 90 days. **Calculation**: count of events/rows, or sum of a property/column (e.g. API tokens, storage, revenue).
- **Display**: Number (single value + % change) or Sparkline (daily bars + trend). Use sparkline to eyeball a declining account at a glance.
- **Match events** (add multiple; counts any match) or a single **Data Warehouse table** (specify timestamp column + group key column). Add **Filters** to events (e.g. `status = 200`).

Example metrics: `api_request` -> heavy users to expand; `$exception` -> error rate / customers in trouble; `export_created` -> power users.

> Data Warehouse usage metrics currently render only on **group** profiles, not person profiles.

Full field reference and MCP automation: [reference/usage-metrics.md](reference/usage-metrics.md).

## Customer journeys: where each customer is in conversion

A **customer journey** is a funnel-based interactive flow graph combining [funnels](/docs/product-analytics/funnels) and [paths](/docs/product-analytics/paths). It shows conversion rates between steps, drop-off points, and what users do between/before/after steps.

- View at **Customer analytics > Customer journeys**; click a step to explore paths and promote a path event into a required/optional step.
- On a profile, the **Customer journey** node shows which steps a specific person/group has completed, is missing, or has optional — i.e. their exact conversion stage.
- Create via **New journey**: a template (Signup conversion, Free-to-paid conversion), an existing saved funnel insight, or from scratch. Note: a journey imported from a funnel does not stay synced to that funnel.

Decision support: spot friction in the aggregate journey, then check an individual account's journey node to see if it's stalled.

## Saved views: usage-based segmentation of customer lists

[Saved views](reference/usage-metrics.md) store **column configuration + filters + sort order** per table (Persons or Groups), private or shared. Build segments like "High value customers" (LTV, last active, subscription columns + revenue filter).

Create: **People & groups > Persons > Configure columns**, add Filters, then **Save current view**. Switch via the dropdown; **Update** appears when you change an active view; gear icon renames/deletes. Combine usage-metric columns + filters to surface at-risk or expansion-ready accounts in one click.

## B2B mode: account-level health

Toggle **B2C / B2B** on the dashboard to show all insights for a chosen **group type** (companies/orgs) instead of individual users. Pair with **group profiles** and Data Warehouse usage metrics for company-level health.

> Self-hosted note: B2B mode requires the [group analytics](/docs/product-analytics/group-analytics) add-on; without it the toggle is unavailable.

## AI & MCP helpers (PM value)

- **Configure dashboard with AI**: a sparkle button beside each dashboard event pre-fills PostHog AI with a tailored prompt to find the right event from your existing data — fast onboarding when you don't know your event names.
- **MCP**: AI agents (Cursor, Claude Code, VS Code) can create/list/update/delete usage metrics via the [PostHog MCP server](/docs/model-context-protocol) (`usage-metrics-*` tools). Lets a PM spin up metrics by prompt, e.g. "Create a usage metric counting `api_request` over 30 days."

> Self-hosted note: PostHog AI / Max features are typically cloud-managed and may not be available (or need extra config) on self-hosted. MCP requires server setup. See the using-posthog-ai-assistant skill.

## Workflow: assess one account's health

1. Open the customer profile (group profile for a B2B account).
2. Read usage metrics — flag declining sparklines (churn risk) or climbing ones (expansion).
3. Check the Customer journey node — is the account stalled at a step?
4. Scan errors and support tickets — is poor health driven by bugs or unresolved issues?
5. Cross-check the dashboard lifecycle insight to see if this is an isolated case or a base-wide pattern.
6. Use a saved view to find all similar accounts, then prioritize outreach or product fixes.

## Customer analytics vs product analytics

Customer Analytics gives good defaults and a framework for initial customer understanding; once needs get specific, move to [product analytics](/docs/product-analytics) for custom dashboards, more insight types, and SQL. Underlying data is billed normally (events as product analytics; warehouse sources as data warehouse rows).

## Related skills

- using-posthog-ai-assistant — PostHog AI / Max and MCP setup referenced above
- analyzing-product-funnels — funnels and paths that power customer journeys
- segmenting-users-with-cohorts — deeper cohort-based segmentation beyond saved views
- tracking-retention-and-lifecycle — lifecycle/retention behind the engagement insight
