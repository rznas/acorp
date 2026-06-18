# Web vitals reference

Deep reference for PostHog Core Web Vitals capture and the web vitals dashboard. Read alongside [SKILL.md](../SKILL.md).

## Table of contents
- [The four metrics](#the-four-metrics)
- [Thresholds](#thresholds)
- [Reading the dashboard](#reading-the-dashboard)
- [Percentile guidance](#percentile-guidance)
- [Enabling autocapture](#enabling-autocapture)
- [Capture config & sampling](#capture-config--sampling)
- [Toolbar & pricing](#toolbar--pricing)

## The four metrics

| Metric | Full name | What it captures | Common causes of poor scores |
|---|---|---|---|
| FCP | First Contentful Paint | Time until first content renders | Render-blocking CSS/JS, slow server, large HTML |
| LCP | Largest Contentful Paint | Time until main/largest element loads | Slow server, render-blocking resources, unoptimized images, client-side rendering |
| INP | Interaction to Next Paint | Responsiveness to clicks/taps | Slow JS, blocked main thread, layout thrashing |
| CLS | Cumulative Layout Shift | Visual stability (unexpected movement) | Images/ads without dimensions, injected content, font FOIT/FOUT |

## Thresholds

| Metric | Good | Needs improvement | Poor |
|---|---|---|---|
| FCP | ≤ 1.8s | 1.8s – 3.0s | > 3.0s |
| LCP | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| INP | ≤ 200ms | 200ms – 500ms | > 500ms |
| CLS | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

(Note: the main dashboard's standalone LCP tile uses a "poor > 4s, good < 2.5s" framing on p75; the web vitals dashboard uses the table above. The INP "good ≤ 100ms" mentioned in passing in one doc is superseded by the ≤ 200ms threshold table — use the table.)

## Reading the dashboard

At `/web/web-vitals`. Trends each of the four metrics over time; supports the same filters as the main dashboard. Bottom table breaks down pages into Good / Needs improvement / Poor — **the Poor pages are the engineering backlog.** The top-level value shown is the value from the *last day* of the selected period (latest non-zero), NOT the period average — this is intentional so you see current state.

## Percentile guidance

PostHog offers p75, p90, p99. **Recommended: p90** — 90% of users get a better experience than shown. p90 balances catching real issues against filtering extreme outliers (bad networks/devices). p99 risks over-optimizing for edge cases.

## Enabling autocapture

Web vitals autocapture is a SEPARATE toggle from regular autocapture. Web vitals can be captured even if regular autocapture is off.

1. Go to autocapture settings → **Web vitals autocapture** section → **Enable**.
2. Requires SDK v1.141.2+.
3. Toggle which of the four metrics to capture.

Captured under event name `$web_vitals` with properties like `$web_vitals_FCP_value`. FCP/LCP arrive soon after page load; INP/CLS accumulate over the page lifecycle.

## Capture config & sampling

Client-side config via `capture_performance` in `PostHogConfig`:

| Attribute | Default | Purpose |
|---|---|---|
| `__web_vitals_max_value` | 15 min | Excludes implausibly large outlier values; set 0 to include all |
| `web_vitals_allowed_metrics` | `['LCP','CLS','FCP','INP']` | Restrict which metrics are captured |
| `web_vitals_delayed_flush_ms` | 5000 | Max wait to batch metrics into one event |

To cut event cost, sample `$web_vitals`:

```ts
import { sampleByEvent } from 'posthog-js/lib/src/customizations'
posthog.init('<token>', {
  before_send: sampleByEvent(['$web_vitals'], 0.5) // keep 50%
})
```

## Toolbar & pricing

- The [toolbar](/docs/toolbar) shows current-page web vitals plus historical data for that page — handy for spot-checking during QA.
- `$web_vitals` events are billed as regular events; first 1M free. Expect roughly 30% of your `$pageview` volume as `$web_vitals` events.
