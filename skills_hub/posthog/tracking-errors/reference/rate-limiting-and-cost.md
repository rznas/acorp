# Rate limiting and controlling error costs

Table of contents
- [Pricing model in brief](#pricing-model-in-brief)
- [Rate limiting: server-side caps](#rate-limiting-server-side-caps)
- [Project-wide vs per-issue](#project-wide-vs-per-issue)
- [Configuring rate limits](#configuring-rate-limits)
- [Client-side controls](#client-side-controls)
- [Suppression](#suppression)
- [PM cost-control checklist](#pm-cost-control-checklist)

## Pricing model in brief

Error Tracking is usage-based, not per-seat. First **100K exceptions/month are free**; above that it's per-exception with volume discounts. No credit card to start. Set **billing limits** to avoid surprise charges. Dropped exceptions (rate-limited or suppressed before capture) are never ingested, so they don't count toward the bill.

## Rate limiting: server-side caps

Rate limits cap how many `$exception` events PostHog ingests so one noisy client or runaway issue can't burn through quota or budget. They run **server-side at ingestion** and apply to every client. Implemented as a **token bucket**: set a max number of exceptions and a time window; the bucket holds up to the max and refills steadily, absorbing short bursts while capping sustained over-limit volume. Exceptions arriving when the bucket is empty are dropped (and not billed). Configure in settings -> rate limits.

## Project-wide vs per-issue

Two independent limit types — use either or both:

- **Project-wide** — one cap across the whole project regardless of issue. A single safety net on total exception volume.
- **Per-issue** — one configured limit applied to every issue individually (each issue gets its own bucket). A single noisy issue gets throttled while other issues keep flowing. When configuring, you can preview a chosen active issue's recent volume against the limit before saving.

## Configuring rate limits

| Option | Meaning |
| --- | --- |
| Maximum exceptions | Most exceptions to ingest per window (leave empty for no limit) |
| Per | Time window: 15 min, 30 min, or 1 hour |

## Client-side controls

To cut exceptions before they leave the client (complementary to server-side limits):

- **Burst protection** — limit rapid-fire exceptions at the source.
- **Suppression rules** — never capture matching exceptions.
- **`before_send` hook** — drop or sample exception events in the SDK.

These reduce both noise and cost at the source; rate limits are the server-side backstop.

## Suppression

Setting an issue's status to **Suppressed** drops its associated exceptions going forward — use for noisy/unhelpful issues you won't action. PostHog recommends also adding **client-side suppression** for those so they're never captured (better for cost and performance). Note suppression is a triage + cost lever, distinct from resolving (resolved issues auto-reopen on recurrence; suppressed ones drop).

## PM cost-control checklist

1. Set **billing limits** so spend can't surprise you.
2. Add a **project-wide rate limit** as a total safety cap.
3. Add a **per-issue rate limit** so one runaway issue can't dominate.
4. **Suppress** chronically noisy issues and ask engineering for client-side suppression / burst protection / `before_send` sampling.
5. Watch for high-occurrence/low-user issues — usually noise worth limiting, not customer pain.
