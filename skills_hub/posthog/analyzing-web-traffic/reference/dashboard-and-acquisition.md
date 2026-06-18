# Dashboard sections & acquisition reference

Deep reference for the PostHog Web Analytics dashboard layout, channel classification, and acquisition reporting. Read alongside [SKILL.md](../SKILL.md).

## Table of contents
- [Full dashboard section list](#full-dashboard-section-list)
- [Paths tabs explained](#paths-tabs-explained)
- [Channel types (full catalog)](#channel-types-full-catalog)
- [Custom channel types](#custom-channel-types)
- [UTM parameters](#utm-parameters)
- [Session attribution explorer](#session-attribution-explorer)
- [World map & demographics](#world-map--demographics)
- [Retention & active hours](#retention--active-hours)

## Full dashboard section list

Top tiles (each shows current vs previous period delta):
- Visitors, Views, Sessions
- Average session duration
- Bounce rate
- LCP score (p75 of first-pageview LCP per session; good < 2.5s, poor > 4s)
- Conversions + conversion rate (only after a conversion goal is added)

Sections below the tiles: Paths, Channels, Referrers, UTMs, World map, Demographics (device/browser/OS/language), Retention, Active hours, Goals.

The whole dashboard is filterable via **+ Add filter** (browser, path name, device type, country, UTMs, etc.) and by clicking rows.

## Paths tabs explained

| Tab | Shows |
|---|---|
| Top paths | Views, visitors, bounce rate, and scroll depth per page |
| Entry path | First page of each session (landing pages) |
| End path | Last page before session ended (exit pages) |
| Outbound clicks | URLs users clicked to leave your site |

PM uses: Entry path = where to optimize landing experience; End path = where users drop; Outbound clicks = which external links pull people away.

Example: filter Path Name contains `/docs/` to rank your most popular documentation.

## Channel types (full catalog)

PostHog auto-classifies each session into a channel using UTMs, referring domain, and more.

| Channel Type | Where the user came from |
|---|---|
| Direct | Typed URL or saved link |
| Paid Search | Search-engine ad (Google, Bing, Baidu) |
| Paid Social | Social-platform ad (Facebook, LinkedIn, Twitter) |
| Paid Video | Video-platform ad (YouTube, Twitch) |
| Paid Shopping | Shopping-platform ad (Amazon, eBay) |
| Paid Other | Ad from unknown platform |
| Cross-Network | Cross-network ad |
| Organic Search | Non-ad search result |
| Organic Social | Non-ad social link |
| Organic Video | Non-ad video link (YouTube, TikTok) |
| Organic Shopping | Non-ad shopping link |
| Affiliate | Affiliate link |
| Referral | Referral link |
| Email | Link from an email |
| Display | Display ad (e.g. Google Display Network) |
| SMS | Link from an SMS |
| Audio | Audio ad (e.g. podcast) |
| Push | Push notification |
| Other | Unknown source |

Full calculation logic: [/docs/data/channel-type](/docs/data/channel-type).

## Custom channel types

If predefined channels don't fit, define rules in project web analytics settings. First matching rule wins; if none match, the default channel type applies. Common PM use: an **AI** channel matching referring domains `chatgpt.com`, `www.perplexity.ai`, etc., to measure LLM-referred traffic.

## UTM parameters

Five UTMs are autocaptured from URL params: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`.

Example: `https://posthog.com/?utm_source=twitter&utm_medium=social&utm_campaign=twitter-campaign`

Correct UTM tagging is critical — it drives both the UTM breakdowns AND channel classification. A vanity URL (e.g. `/fireship`) can redirect to a UTM-tagged URL so the source shows up in reports. See [/docs/data/utm-segmentation](/docs/data/utm-segmentation).

## Session attribution explorer

At `/web/session-attribution-explorer`. Shows session count for each combination of channel type + referrer + UTM, plus example entry URLs and the SQL query used. Use it to debug "why is this traffic classified as Direct/Other?" and to validate UTM hygiene.

## World map & demographics

The world map plots visitor locations; switch it to show top countries, regions (e.g. California, England, Ontario), cities, timezones, or languages. Demographic sections cover device type (desktop/mobile/tablet), browser, OS, and language — useful for QA/build prioritization and localization decisions.

## Retention & active hours

- **Retention**: cohorts unique users who did any event for the first time last week, then tracks the % returning in following weeks. Same engine as [product analytics retention](/docs/product-analytics/retention).
- **Active hours**: heatmap of unique users or total pageviews by hour-of-day × day-of-week. Tabs switch the metric. Use closed 7-day (or multiples of 7) ranges for clean buckets — longer ranges double-count weekdays. Times follow project timezone (UTC default). PM use: timing newsletters, launches, and support staffing.
