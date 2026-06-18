# Marketing analytics reference

Deep reference for PostHog Marketing Analytics (ad-spend + conversion ROI). Read alongside [SKILL.md](../SKILL.md). Currently an opt-in beta — enable in feature-preview settings.

## Table of contents
- [Onboarding](#onboarding)
- [Dashboard insights](#dashboard-insights)
- [Native ad sources & required tables](#native-ad-sources--required-tables)
- [Data warehouse & self-managed sources](#data-warehouse--self-managed-sources)
- [Conversion goals](#conversion-goals)
- [Attribution settings](#attribution-settings)
- [Reported vs PostHog conversions](#reported-vs-posthog-conversions)
- [Column definitions](#column-definitions)
- [Self-host notes](#self-host-notes)

## Onboarding

First visit to `/web/marketing` launches a wizard: Welcome → Add source → Conversion goals (optional, configurable later in environment-marketing-analytics settings).

## Dashboard insights

- **Marketing costs over time** — stacked area of spend per platform, in your base currency, over the selected range. Reveals budget allocation and spend trends.
- **Campaign breakdown table** — per-campaign metrics (see [column definitions](#column-definitions)).

## Native ad sources & required tables

PostHog auto-syncs cost/campaign data from these platforms. Each needs its two tables synced:

| Platform | Required tables |
|---|---|
| Google Ads | `campaign`, `campaign_overview_stats` |
| LinkedIn Ads | `campaign_groups`, `campaign_group_stats` |
| Meta Ads | `campaigns`, `campaign_stats` |
| Pinterest Ads | `campaigns`, `campaign_analytics` |
| TikTok Ads | `campaigns`, `campaign_report` |
| Reddit Ads | `campaigns`, `campaign_report` |
| Bing Ads | `campaigns`, `campaign_performance_report` |
| Snapchat Ads | `campaigns`, `campaign_stats_daily` |

Setup guides live under `/docs/cdp/sources/<platform>`.

## Data warehouse & self-managed sources

- **Data warehouse** (currently BigQuery): map required fields — Campaign (→ `utm_campaign`), Cost, Date, Source (→ `utm_source`). Optional: Clicks, Currency, Impressions.
- **Self-managed files**: S3, GCS, Cloudflare R2, Azure. JSON format recommended. Map the same fields.

See [/docs/cdp/sources](/docs/cdp/sources).

## Conversion goals

Define goals from events, actions, or data warehouse tables.

| Field | Notes |
|---|---|
| Conversion goal name | e.g. purchase, sign up, download |
| Event / action / DW table | What to track |
| Schema mapping | Events use `utm_campaign` + `utm_source`; DW tables map attribution fields |

For DW-table goals, required mappings: UTM Campaign Name, UTM Source Name, Timestamp field; optional Distinct ID field.

## Attribution settings

Tune how ad campaign data matches your conversion events:

- **Campaign field preference** (per platform): match `utm_campaign` against the human-readable **campaign name** or the platform **campaign ID**. Pick whichever your UTMs actually use.
- **UTM campaign manual mappings**: map arbitrary `utm_campaign` values to a specific campaign name/ID. Use when UTM naming differs from campaign naming, to consolidate variants, or across multi-channel naming. PostHog suggests matches by string similarity with a match %.
- **Custom source mappings**: allowlist which `utm_source` values attribute to each platform (e.g. Google Ads ← `google`, `google_ads`, `paid_search`; Meta ← `facebook`, `instagram`, `meta`).
- **Base currency**: all costs/values convert to the project base currency before display; undetectable source currency is assumed to be base.

## Reported vs PostHog conversions

- **Reported** = counted by the ad platform's own pixel and attribution model (e.g. 7-day click, 1-day view).
- **PostHog conversion goals** = your events/actions with your attribution rules.
- They differ due to attribution windows, tracking method, and data freshness. Use reported metrics to reconcile against the ad platform's dashboard; use PostHog goals for custom attribution.

## Column definitions

Display: `Campaign`, `Source`, `ID`. Base metrics: `Impressions`, `Clicks`, `Cost`, `Reported Conversion`, `Reported Conversion Value`.

Calculated metrics seen in the breakdown table:
- **Cost per click** = total cost ÷ total clicks
- **CTR** = total clicks ÷ total impressions × 100
- **Cost per reported conversion** = total cost ÷ reported conversions
- **Cost per conversion** = total cost ÷ PostHog-goal conversions

Full per-platform field mappings and formulas: [/docs/web-analytics/marketing-analytics-schema](/docs/web-analytics/marketing-analytics-schema).

## Self-host notes

Native ad-platform connectors rely on PostHog's managed data-pipeline infrastructure. On self-hosted PostHog these may require extra configuration or be unavailable — the data-warehouse and self-managed file sources are the portable fallback. Verify availability in your own instance's settings before promising native-source dashboards.
