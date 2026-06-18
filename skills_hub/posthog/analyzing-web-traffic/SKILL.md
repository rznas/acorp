---
name: analyzing-web-traffic
description: Guides product managers using PostHog Web Analytics to answer where website traffic comes from, which marketing channels and campaigns convert, and whether site performance is hurting users. Covers the pre-built web analytics dashboard (visitors, pageviews, sessions, bounce rate, paths, channels, referrers, UTM attribution, world map, retention, active hours), conversion goals, Core Web Vitals (LCP, FCP, INP, CLS), the Live real-time tab, and Marketing Analytics (ad cost, CTR, cost per conversion across Google/Meta/LinkedIn/TikTok). Use for questions like "where is my traffic coming from", "which channel converts best", "what's my bounce rate", "is my site slow", "how is my ad spend performing", or when deciding between web vs product analytics.
---

# Analyzing web traffic with PostHog

PostHog **Web Analytics** is a single, pre-built dashboard for high-level website metrics. It answers aggregate "what happened on my site" questions fast, without building insights. For deep per-user behavior, funnels, or custom insights, use **Product Analytics** instead (see decision table below).

> Self-hosted note: The core web analytics dashboard, web vitals, conversion goals, and the Live tab are standard features (`free`/`selfServe`/`enterprise` = full in docs). PostHog AI/Max (`investigate traffic with AI`) and some managed ad-platform data sources for Marketing Analytics depend on hosted/managed infrastructure — availability may differ on self-hosted. Confirm in your instance settings.

## Web analytics vs product analytics — pick the right tool

| If the PM needs... | Use | Why |
|---|---|---|
| At-a-glance site traffic, top pages, top sources | **Web Analytics** | One pre-built dashboard, no setup |
| Bounce rate, sessions, channels, geography | **Web Analytics** | Pre-defined, marketer-friendly |
| Custom multi-step funnels, cohorts, breakdowns | **Product Analytics** | Unlimited custom insights |
| Per-user journeys, feature adoption, stickiness | **Product Analytics** | Built for engineering/product/data teams |
| "Right now" live event stream | **Web Analytics → Live tab** | Seconds-fresh stream |

Web Analytics events are limited to pageview, pageleave, and conversions; properties are limited/pre-defined. For anything custom, the docs explicitly say "use the product analytics tab as usual."

## PM question → feature map

| PM question | Where in Web Analytics | What it tells you |
|---|---|---|
| How many people visit? Trending up/down? | Top tiles: Visitors, Views, Sessions (vs previous period) | Traffic volume + direction |
| How long do people stay? | Avg session duration tile | Engagement depth |
| Are people leaving immediately? | Bounce rate tile + Paths table | Page/landing quality |
| What pages are popular? Where do they land/leave? | **Paths** (Top / Entry / End / Outbound clicks tabs) | Content performance, exit leaks |
| Where does traffic come from? | **Channels**, **Referrers**, **UTMs** sections | Acquisition mix |
| Which campaign drove visits? | UTM source/campaign breakdown | Campaign attribution |
| Where are users located? | **World map** (countries, regions, cities, languages, timezones) | Geo targeting |
| What devices/browsers/OS? | Demographic sections | Build/QA priorities |
| Are people converting? | **Goals** + **Add conversion goal** | Conversion rate |
| Is my site slow / unstable? | **Web Vitals** tab (LCP, FCP, INP, CLS) | Performance health |
| What's happening right now? | **Live** tab | Launch/campaign monitoring |
| What does my ad spend buy? | **Marketing Analytics** dashboard | Cost, CTR, cost per conversion |

Dashboard lives at `/web`. Web Vitals at `/web/web-vitals`. Live at `/web` (Live tab). Marketing at `/web/marketing`.

## Core metrics — how to read them

- **Visitor** = one unique person. **Session** = one visit (has start, duration, end). One visitor has many sessions; one session has many pageviews. Don't conflate visitors and sessions when reporting.
- **Bounce rate** = % of sessions with one pageview, no autocapture, and < 10 seconds. Requires [autocapture](/docs/product-analytics/autocapture) enabled plus `$pageleave` and `$autocapture` events, or the number is wrong. Threshold is configurable in project web analytics settings.
- **LCP tile** on the main dashboard = p75 of LCP for the first pageview of each session. Good < 2.5s, poor > 4s.
- Every tile compares the selected period against the **previous period** — that delta is the headline a PM should report, not the raw number alone.

## Filtering and drilling — the PM workflow

1. Set the date range (24h, 7d, YTD, or custom) at the top.
2. Click **+ Add filter** to slice by Path Name, Device Type, Country, Referrer URL, UTMs, browser, etc.
3. Or click any row in Paths/Channels/Referrers to filter the whole dashboard to that value (e.g. click `/pricing` to see only pricing-page stats).
4. For attribution debugging, open the **session attribution explorer** — it shows session counts per channel/referrer/UTM combination, example entry URLs, and the underlying SQL.

Combine filters to answer targeted questions, e.g. "conversion rate for mobile visitors from organic search in Germany."

## Channels, referrers, UTMs (acquisition)

- **Channels** = auto-classified acquisition buckets (Direct, Paid/Organic Search, Paid/Organic Social, Paid/Organic Video, Email, Referral, Affiliate, Display, SMS, etc.). PostHog classifies from UTMs + referring domain. Define **custom channel types** in project settings (e.g. an "AI" channel for `chatgpt.com`, `perplexity.ai`).
- **Referrers** = raw source domains; direct traffic shows as `$direct`.
- **UTMs** = source, medium, campaign, content, term. Correct UTM tagging is what makes both UTM reports *and* channel classification accurate — flag bad UTM hygiene as a data-quality risk.

Full channel-type catalog and custom-channel setup: see [reference/dashboard-and-acquisition.md](reference/dashboard-and-acquisition.md).

## Conversion goals — measuring outcomes

A conversion goal is a key action (signup, demo booked, subscribe, purchase). It maps to GA4 "key events" / Google Ads "conversion actions."

1. First capture a **custom event** (e.g. `user_signed_up`) or create an **action**. Tip from docs: for signups, use an explicit custom event, not autocapture, so tracking survives UI changes.
2. On the dashboard, **Add conversion goal** → pick the event/action.
3. Read three metrics: **Total conversions**, **Unique conversions**, **Conversion rate** (% of visitors who converted). All plottable over time.

**Decisions it supports:** which channels/pages convert best (add filters), where to invest. To improve: filter by segment, watch [session replays](/docs/session-replay) of converters/non-converters to find friction, send conversions to ad platforms (e.g. [Google Ads](/docs/cdp/destinations/google-ads)) to optimize spend, and run [A/B tests](/docs/experiments) on copy/checkout/forms. For multi-step or breakdown analysis, graduate to [funnels](/docs/product-analytics/funnels).

## Web vitals — site performance

Captures Core Web Vitals automatically (when web vitals autocapture is on; separate toggle from regular autocapture). Dashboard at `/web/web-vitals` trends each metric and lists pages by Good / Needs improvement / Poor.

| Metric | Measures | Good | Poor |
|---|---|---|---|
| FCP | Time to first content painted | ≤ 1.8s | > 3.0s |
| LCP | Time to largest element loaded | ≤ 2.5s | > 4.0s |
| INP | Responsiveness to interaction | ≤ 200ms | > 500ms |
| CLS | Visual stability (layout shift) | ≤ 0.1 | > 0.25 |

**How to read:** Use the **p90** percentile (PostHog default recommendation) — 90% of users get a better experience than shown. p99 over-optimizes for outliers. The top-level number is the *latest non-zero day*, not the period average. The bottom table ranks worst pages — that's the engineering backlog.

**Decision:** prioritize fixing pages with Poor scores; correlate slow LCP with high bounce. Deeper config (autocapture enablement, sampling, thresholds, `$web_vitals` event): [reference/web-vitals.md](reference/web-vitals.md).

## Live tab — real-time monitoring

Continuously updating feed of recent events (last few minutes): timestamp, path, referrer/UTM, country, device. Filter by host/path/referrer to isolate a launch or campaign.

**Use it for:** launch days (Hacker News / Product Hunt), watching paid campaigns convert within minutes, catching deploy regressions, verifying a new install. **Not for:** historical analysis — no scroll-back, may be sampled on high-volume sites; use the main dashboard or SQL editor for that. (Live is alpha; layout may change.)

## Marketing analytics — ad spend & ROI

Separate dashboard at `/web/marketing` that joins ad-platform cost data with PostHog conversions. (Currently an **opt-in beta** — enable in feature-preview settings.)

Answers "what is my ad spend buying?" Key columns: Total cost, Clicks, Impressions, **Cost per click**, **CTR**, **Cost per reported conversion**, **Cost per conversion** (PostHog goals). Shows stacked spend-over-time across platforms in your base currency.

- **Native ad sources:** Google, Meta, LinkedIn, TikTok, Pinterest, Reddit, Bing, Snapchat Ads (each syncs campaign + stats tables).
- Or connect a **data warehouse** (BigQuery) / **self-managed** source (S3, GCS, R2, Azure) and map Campaign / Cost / Date / Source fields.
- Attribution matches ad campaign ↔ your events via `utm_campaign` / `utm_source`; tune with campaign-field preference, manual UTM mappings, and custom source allowlists.
- **Reported vs PostHog conversions:** ad platforms count via their own pixel/attribution window; PostHog goals use your rules. Use reported metrics to reconcile with platform dashboards, PostHog goals for custom attribution.

> Self-hosted note: native ad-platform sources rely on PostHog's managed data-pipeline connectors; on self-hosted these may need extra configuration or may be unavailable. Data-warehouse/self-managed file sources are the fallback. Confirm availability in your instance.

Full source list, required tables, column formulas, and field mappings: [reference/marketing-analytics.md](reference/marketing-analytics.md).

## Data quality checklist

- [ ] UTMs tagged consistently on every campaign link (source, medium, campaign) — drives both UTM and channel reports.
- [ ] Autocapture + `$pageleave` + `$autocapture` enabled so bounce rate is valid.
- [ ] Web vitals autocapture enabled (SDK v1.141.2+) if performance matters.
- [ ] Conversion goals use explicit custom events, not autocapture, for stable signups.
- [ ] [Path cleaning](/docs/web-analytics/path-cleaning) rules set so dynamic URLs (e.g. `/user/123`) collapse into one path.
- [ ] Base currency set so marketing costs report consistently.

## AI & MCP helpers

- **PostHog AI / Max** — ask in natural language: "why did traffic drop on Tuesday?", "which landing pages have the highest bounce rate?", "compare organic vs paid conversion rates." It breaks traffic down by source, geo, device, page, and time, and investigates anomalies across dimensions automatically. See the `using-posthog-ai-assistant` skill.
- **PostHog MCP server** — query web analytics from your code editor/agent (Cursor, Claude Code, etc.): traffic after deploys, top landing pages, bounce-rate regressions. Install via `npx @posthog/wizard mcp add`.

## Related skills

- `understanding-web-vs-product-analytics` · `building-funnels` · `running-experiments` · `capturing-session-replays` · `setting-up-conversion-tracking` · `using-posthog-ai-assistant` · `managing-data-pipelines-and-sources`
