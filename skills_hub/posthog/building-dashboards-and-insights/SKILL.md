---
name: building-dashboards-and-insights
description: Guides a product manager on composing PostHog insights into dashboards, sharing and embedding them, scheduling email/Slack subscriptions, setting alerts on trend metrics, using notebooks for analysis narratives, and applying color themes. Use when a PM asks how to "build a reporting view for stakeholders", "create a dashboard", "share findings with the team", "get notified when a metric crosses a threshold", "set up an alert on signups/pageviews/revenue", "schedule a weekly report", "send a dashboard to Slack", "detect anomalies in metrics", "write up an analysis", "use a dashboard template", or "brand my charts". Covers PostHog dashboards, dashboard templates, sharing/embedding, subscriptions, alerts, anomaly detection, notebooks, canvases, and color themes.
---

# Building dashboards and insights

This skill covers how a PM turns individual [insights](/docs/product-analytics/insights) into shareable, monitored reporting surfaces in PostHog: **dashboards** (recurring metric tracking), **subscriptions** (push reports to email/Slack), **alerts** (threshold/anomaly notifications), **notebooks** (analysis narratives), and **color themes** (branding).

For choosing and building the underlying insight types (trends, funnels, retention, paths, stickiness, lifecycle, SQL), see the **analyzing-product-usage** skill. This skill is about assembling, sharing, and monitoring them.

## PM question -> feature map

| PM question | Use this | Section |
| --- | --- | --- |
| "Give stakeholders one screen with our key metrics" | Dashboard | [Dashboards](#dashboards-recurring-reporting-views) |
| "Don't make me build it from scratch" | Dashboard template | [Templates](#dashboard-templates) |
| "Let me compare segments without rebuilding insights" | Dashboard filters / date overrides | [Filters](#filters-and-date-overrides) |
| "Send this report to my team every Monday" | Subscription | [Subscriptions](#subscriptions-push-reports) |
| "Post the weekly report to a Slack channel" | Slack subscription | [Subscriptions](#subscriptions-push-reports) |
| "Tell me when signups drop below X" | Alert (threshold) | [Alerts](#alerts-threshold--anomaly-monitoring) |
| "Flag unusual spikes automatically" | Alert (anomaly detection) | [Alerts](#alerts-threshold--anomaly-monitoring) |
| "Write up a findings doc with charts inline" | Notebook | [Notebooks](#notebooks-analysis-narratives) |
| "Quickly share an exploration, no save needed" | Canvas | [Notebooks](#notebooks-analysis-narratives) |
| "Show this dashboard publicly / embed in our site" | Sharing & embedding | [Sharing](#sharing--embedding) |
| "Match charts to our brand colors" | Color themes | [Color themes](#color-themes) |

## Dashboards: recurring reporting views

**Question it answers:** "What's the live status of the metrics this team cares about?" Dashboards are for tracking common metrics over time (vs. notebooks, which are for ad-hoc narrative analysis).

**Set up (where it lives):** **Dashboards** in the left nav -> **New Dashboard** -> pick a template or **Blank Dashboard**. Add insights via **+New insight**, or from any existing insight click **Add to dashboard**.

**Key behavior to know:**
- An insight can live on **multiple dashboards at once** — it's the same underlying insight, so edits propagate everywhere. **Move to / Copy to** (card ⋯ menu) keep this link; **Duplicate** makes an independent copy.
- Organize dashboards into **folders** (⋯ -> Move to another folder; editor access required). The folder filter persists in the URL (`?folder=`) so you can share filtered views.
- **Copy to another project** (actions sidebar) reuses a dashboard or project-scoped template across projects in the same org. Copies are independent.
- Beyond insights, dashboards can hold **text cards** (Markdown annotations), **button tiles** (links to internal/external pages), and live **widgets** (early access): Recent events, top Error Tracking issues, recent Session Replay recordings.

**Read/interpret:** Use **Edit layout** mode (`E` key, or ⋯ -> Edit layout) to arrange tiles. Card ⋯ menu toggles legend, values/labels on series, and annotations without opening the editor. `F` = full screen (useful for a TV/wallboard).

**Decision it supports:** Establishes the shared source of truth a team checks repeatedly — e.g., a launch dashboard, a product-health wallboard, or a weekly exec review.

See `reference/dashboards-and-themes.md` for layout details, widgets, auto-refresh, AI refresh analysis, and Terraform-as-code.

### Dashboard templates

When creating a dashboard you can start from a **template** (e.g., website metrics, product health, LLM/AI observability) instead of blank. The **Templates** tab on the Dashboards page lists them; the **...** menu lets you **Copy to another project**. Only project-scoped templates can be copied; global templates can't.

### Filters and date overrides

Apply filters at the **dashboard** level to slice every insight at once without rebuilding them:
- **Event properties, person properties, group properties** (groups need the group analytics add-on), **feature flags**, **cohorts**.
- **Date range override:** default is *No date range override* (each insight uses its own range). Setting one forces all insights to that range on the dashboard view only — it doesn't change the saved insight.

PM pattern: **duplicate a dashboard** and apply different filters per copy (e.g., one per market or plan tier) to compare segments.

> Note: dashboard property-filter overrides don't apply to insights backed by data-warehouse series (a warning shows on the card). Date overrides still work.

## Subscriptions: push reports

**Question it answers:** "How do I get this report in front of stakeholders on a schedule without them logging in?"

**Set up:** Click **Subscribe** in the sidebar of any insight or dashboard. Manage all of them at `/subscriptions` (or Cmd/Ctrl+K -> "Subscriptions"), filterable by channel, creator, and source.

**Channels & options:**
- **Email** — multiple recipients (members or not); non-members get a one-time notice with an optional personal message.
- **Slack** — requires installing the PostHog Slack app to the workspace and to the target channel first; private channels appear only if PostHog is already in them.
- **Dashboard subscriptions** let you pick up to **6 insights** to include (pre-selects the first 6).
- **Schedule:** daily, weekly, monthly, or custom interval; monthly supports ordinal rules like "first weekday" or "last Friday" (good for business reports that skip weekends).
- **Custom name** distinguishes multiple subscriptions to the same source (shows in the Slack header).
- **Test delivery** (⋯ on the subscription) sends immediately to verify config.

**Read/interpret:** Each delivery shows the next scheduled date. Optional **AI summaries** add a short narrative of key changes vs. the previous delivery.

**Decision it supports:** Keeps stakeholders aligned on metrics passively; the AI summary surfaces what changed so the PM knows where to dig.

> Self-hosted note: **Prompt subscriptions** (free-text AI-generated recurring reports) are **PostHog Cloud only** and need AI data-processing consent plus AI credits. AI summaries and `@PostHog` Slack follow-ups also require AI data-processing consent; on self-hosted, hosted AI features may be unavailable or need extra config — verify in your instance.

## Alerts: threshold & anomaly monitoring

**Question it answers:** "Notify me when a metric crosses a line or behaves abnormally — without me watching the dashboard."

**Scope:** Alerts are supported on **trends only**. Funnels, retention, etc. don't support alerts.

**Set up:** Open a trend insight -> **Actions** (sidebar) -> **Alerts** -> **New alert**. Then:
1. Pick the **series** (e.g., `A - $pageview` vs `B - signed_up`).
2. Pick the type: **has value** (absolute), **increases by** / **decreases by** (relative, supports % thresholds).
3. Set at least one bound (**more than** / **less than**).
4. Choose check frequency: every 15 min (Boost/Scale/Enterprise), hourly, daily, weekly, monthly.
5. Pick recipients: subscribed users get in-app notifications; also add email, Slack, Discord webhook, or webhook URL.

**Read/interpret:** View all alerts under Product analytics -> **Alerts** tab. On a **breakdown** trend, the alert fires when *any* breakdown value breaches the bound. Use **Check ongoing period** to alert mid-period; **Quiet hours** / skip-weekends suppress off-hours noise.

**Anomaly detection (beta):** Instead of a fixed number, a statistical detector learns "normal" and flags outliers — useful when traffic varies by time or has seasonality. Start with **Z-score** or **MAD**, **simulate** on history before saving, and use **ensemble AND** to cut false positives.

**Decision it supports:** Turns a metric into an early-warning system (incident detection, conversion drops, traffic anomalies) so the PM reacts instead of polling.

> Note: alerts auto-disable on invalid config (e.g., insight changed from trend to funnel, series removed, no bounds set) and must be **manually re-enabled** from the Alerts tab after fixing.

See `reference/alerts-anomaly-detection.md` for the full detector catalog (13 algorithms), parameters, window sizes, and tuning tips.

## Notebooks: analysis narratives

**Question it answers:** "How do I piece findings into a story — charts, replays, events, and commentary on one page?"

**Set up:** Notebooks tab -> **New notebook**. Add content by typing `/` (slash commands like `/trend`, `/table`, `/latex`), the **+** icon, or **dragging** any PostHog link into the right-side popover (accessible from anywhere as a scratchpad).

**What it holds:** Markdown text; all insight types (trends, funnels, retention, paths, stickiness, lifecycle, SQL); event/person/group/cohort lists; session replays and playlists with timestamped comments; feature flags; surveys; images; links; LaTeX; tables (paste from Sheets/Excel).

**Dashboards vs. notebooks:** dashboards = repeated metric tracking; notebooks = building data into a narrative (bug investigations, launch retros, research write-ups). Insights created inside a notebook aren't auto-saved to "Your Insights."

**Sharing:** Org members can view/edit (share icon -> link); no real-time multiplayer (conflict detection warns on save); external sharing not yet available. A **Canvas** is a notebook whose contents live entirely in the URL — great for a quick, unsaved exploration to hand a colleague (the link changes as you edit).

**Decision it supports:** Documents the *why* behind a decision and the supporting evidence in one linkable place.

## Sharing & embedding

**Question it answers:** "Show this to customers/partners/the public without giving them account access."

**Set up:** Insight: **...** -> **Share or embed** -> toggle public sharing -> **Copy public link** or copy the iframe HTML. Dashboard: **Share** (top right) -> toggle public -> copy link/iframe. Viewers can't change visualization or date range.

**Key behavior:** Anyone with the link can view (no auth) — for finer control use embedded analytics. Add `?refresh=true` to an embedded insight URL to refresh on load. Shared/embedded **dashboards auto-refresh every 30 min** while the tab is open. Enabling/disabling dashboard sharing is recorded in activity logs.

**Decision it supports:** External reporting (status pages, customer-facing metrics, public transparency) without seat costs.

## Color themes

**Question it answers:** "Make charts match our brand."

**Set up:** Project settings -> Product analytics -> **Data colors** -> **Add theme**, name it, set the palette, **Save**. Pick a project-wide default from the dropdown. Override per insight via **...** -> **View source** -> add `"dataColorTheme": <id>` to the query node. Themes also apply to shared/embedded views.

To pin a specific color to a result (trend/funnel), click the color icon in the **Detailed results** table.

> Self-hosted note: custom color **themes** require a Teams or Enterprise plan per the docs; the per-result color assignment is available to everyone.

## AI & MCP helpers (PM value)

- **PostHog AI** can build dashboards, generate insights, create trend alerts, and run an **AI refresh analysis** that summarizes what changed on a dashboard. PMs use it to spin up a reporting view from a sentence ("Build a dashboard for onboarding metrics") or set alerts in natural language ("Alert me when daily signups drop below 100").
- **MCP server** exposes alert and insight tools (create/list/update/delete) to an AI coding agent — useful for managing reporting from the dev workflow.

For how to drive these, see the **using-posthog-ai-assistant** skill.

> Self-hosted note: PostHog AI / Max and prompt subscriptions are hosted/AI features that may not be available or may need extra configuration on self-hosted instances — confirm availability before relying on them.

## Workflow: stand up a stakeholder reporting view

1. Confirm the metrics and insight types needed (use **analyzing-product-usage** to build them).
2. Create a dashboard from a **template** or blank; add the insights (reuse existing ones — don't duplicate).
3. Add **text cards** for context and arrange with **Edit layout**.
4. Apply a **color theme** for brand consistency.
5. Set a dashboard **date range override** / filters if the view needs a fixed window or segment.
6. Add **alerts** on the critical trend metrics (threshold or anomaly) with the right recipients.
7. Create a **subscription** (email/Slack) on the schedule stakeholders expect; **Test delivery** to verify.
8. If external visibility is needed, enable **public sharing** or embed.

## Related skills

- **analyzing-product-usage** — building the underlying insight types (trends, funnels, retention, paths, stickiness, lifecycle, SQL).
- **using-posthog-ai-assistant** — driving PostHog AI / MCP to generate dashboards, insights, and alerts.
- **understanding-posthog-core-concepts** — events, persons, groups, cohorts, and feature flags referenced in filters.
