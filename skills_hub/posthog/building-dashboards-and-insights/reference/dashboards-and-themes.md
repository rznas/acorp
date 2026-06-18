# Dashboards: layout, widgets, refresh, and Terraform

Deep reference for the dashboard features summarized in SKILL.md.

## Table of contents
- [Editing layout](#editing-layout)
- [Insight card options](#insight-card-options)
- [Text cards and button tiles](#text-cards-and-button-tiles)
- [Widgets (early access)](#widgets-early-access)
- [Auto refresh](#auto-refresh)
- [AI-powered refresh analysis](#ai-powered-refresh-analysis)
- [AI starter prompts](#ai-starter-prompts)
- [Managing dashboards with Terraform](#managing-dashboards-with-terraform)
- [Color themes detail](#color-themes-detail)

## Editing layout

Enter Edit layout mode five ways: press `E`; ⋯ at top of dashboard -> Edit layout; ⋯ on any card -> Edit layout; hover a card edge until the resize cursor appears; or drag a card outside its chart area. Press `E` again to save, `Escape` to discard. `F` = full screen.

On small screens, tiles auto-stack in a single column following the desktop order; drag/resize is only available on wider viewports.

## Insight card options

Each card's ⋯ menu offers quick visualization toggles (editor access required), saved back to the insight:
- **Show/Hide legend** — Trends, Stickiness, Lifecycle.
- **Show/Hide values on series** — Trends, Stickiness, Lifecycle, Funnel (trends view). For pie charts this is **Show/Hide labels on series**.
- **Show/Hide annotations** — Trends time-series and Funnel historical-trends.
- **Move to** — moves the card to another dashboard (removes it here).
- **Copy to** — copies the card to another dashboard, keeping it here (same underlying insight, edits propagate).

## Text cards and button tiles

- **Text card** (Add... dropdown -> Text card): Markdown plus drag-and-drop images. Use for context/annotations; for deeper analysis prefer a notebook.
- **Button tile** (Add... -> Button): a clickable link tile. Configure URL (internal path like `/dashboards` or full external URL), button text, placement (left/right), style (primary/secondary), and transparent background.

## Widgets (early access)

Subscribe on the early-access features page to enable. Widgets show live lists (not charts) from other PostHog products. Add via Add... dropdown.

- **Activity / Recent events** — mirrors Activity > Explore. Shows event, person, timestamp, library, URL. Options: date range (default last 24h), limit (default 25, max 50), filter test accounts.
- **Error Tracking / Top issues** — issue title, occurrences, affected users, last seen. Options: date range (default 7d), limit (default 10, max 25), order by (occurrences/last seen/first seen/users/sessions), direction, status filter, assignee, filter test accounts.
- **Session Replay / Recent recordings** — user, duration, activity score, timestamp. Options: date range (default 7d), limit (default 10, max 25), order by (start time/activity/duration/clicks/console errors), direction, saved filter, filter test accounts.

## Auto refresh

Manual refresh anytime; or dropdown next to the refresh button -> turn on auto refresh and pick an interval (good for a TV wallboard). Auto refresh only runs while the browser tab is active. Shared/embedded public dashboards auto-refresh every 30 minutes by default with no config.

## AI-powered refresh analysis

Click the sparkle icon on the refresh button to refresh **and** have AI compare before/after, summarizing significant drops, spikes, and trend reversals across all insights. Rate it thumbs up/down. Not available when dashboard filter overrides, URL filters, or URL variables are applied.

## AI starter prompts

A blank dashboard's empty state shows AI starter prompts (landing page performance, core web metrics, weekly marketing health, full-funnel tracking, user research ops). Clicking one opens PostHog AI in the side panel pre-filled; it builds the insights and adds them automatically.

## Managing dashboards with Terraform

For infrastructure-as-code teams, use the PostHog Terraform Provider for version-controlled, reproducible dashboards across projects and CI/CD.

**Export existing:** the **Manage with Terraform** button (on dashboard pages and insight detail pages) generates HCL — dashboard pages export all insights, layouts, alerts, and hog functions; insight pages export the single insight with its alerts/hog functions.

Provider config:

```hcl
provider "posthog" {
  host       = "https://us.posthog.com"  # or your self-hosted host
  api_key    = var.posthog_api_key
  project_id = var.posthog_project_id
}
```

Resources: `posthog_dashboard`, `posthog_insight` (takes a `query_json`), and `posthog_dashboard_layout` (positions tiles via `layouts_json` with x/y/w/h, plus text/color for text and button tiles).

## Color themes detail

Settings: Project settings -> Product analytics -> **Data colors** (`/settings/project-product-analytics#data-theme`).

- **Create:** Add theme -> name -> set palette -> Save. Results cycle through the theme's colors in order, restarting after the last.
- **Default:** select from the Data colors dropdown.
- **Per-insight override (advanced):** ... -> View source -> on the query node (e.g. `TrendsQuery`) add `"dataColorTheme": <id>`.
- **Per-result color:** click the color icon in the Detailed results table (trend/funnel only; available on all plans).

Custom themes require a Teams or Enterprise plan; themes apply to shared/embedded views too.
