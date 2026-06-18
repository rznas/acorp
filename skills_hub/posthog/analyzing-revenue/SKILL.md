---
name: analyzing-revenue
description: Guides product managers using PostHog Revenue analytics to answer money questions — MRR and ARR trends, revenue growth, new/expansion/contraction/churn MRR, gross revenue, ARPU, LTV, top customers, revenue by product/plan/cohort/country, and revenue goals. Covers connecting payment platforms (Stripe), capturing manual revenue events, deferred revenue recognition, connecting revenue to persons/groups, managed SQL views, and currency conversion. Use when a PM asks how is revenue trending, what is our MRR/ARR, how much are we losing to churn, which plan or customer drives revenue, how to track revenue from Stripe, or how to set a revenue target.
---

# Analyzing revenue in PostHog

PostHog Revenue analytics turns payment data (Stripe or manual events) into a dashboard of MRR/ARR, growth components, churn, ARPU, LTV, and top customers, plus standardized SQL views you can query anywhere.

> Deprecation note: PostHog plans to remove the dedicated Revenue analytics dashboard on or after **June 30, 2026**. The direction is to expose revenue properties on persons/groups so you can use revenue in any insight, SQL, or profile — and have PostHog AI / MCP build the dashboard each use case needs. Everything in this skill (metrics, managed views, deferred revenue, Stripe connection) still informs how revenue is computed; favor SQL managed views and persons/groups revenue for durability.

> Self-hosted note: Revenue analytics is built on the data warehouse, managed SQL views, and currency conversion (sourced from Open Exchange Rates). PostHog AI / Max and some managed data-warehouse source integrations may be cloud-only or require extra config on self-hosted; availability may differ on self-hosted. Confirm your instance has data warehouse sources and AI enabled before promising those workflows.

## PM question → where to look

| PM question | Feature / metric | Where it lives |
| --- | --- | --- |
| How is revenue trending overall? | Gross revenue chart | Revenue analytics dashboard |
| What's our MRR / ARR right now and over time? | MRR / ARR chart (toggle) | Dashboard, top of MRR section |
| How much MRR are we gaining vs losing? | MRR breakdown: New / Expansion / Contraction / Churn | Dashboard, MRR breakdown |
| How many paying customers / active subscriptions? | Subscriptions & customers charts | Dashboard |
| What's our churn rate? | Churned customers → churn rate (feeds LTV) | Dashboard, subscriptions section |
| How much revenue per user? | ARPU | Dashboard |
| What's a customer worth over their lifetime? | LTV (= ARPU / churn rate) | Dashboard |
| Who are our biggest customers? | Top customers | Dashboard (needs customer connection) |
| Revenue by product / plan / coupon / country / customer? | Filters & breakdowns | Dashboard "Add filter" / breakdown |
| Are we on track to a revenue target? | Revenue goals | Settings → shown on MRR/ARR or gross chart |
| Revenue split correctly across a multi-month prepayment? | Deferred revenue | Automatic for Stripe with `period` |
| Custom revenue funnel / cohort / segment analysis | Managed views in SQL editor | `views` tab in SQL editor |
| Quick natural-language revenue answer / auto dashboard | PostHog AI / MCP | See reference/ai-and-mcp.md |

## Getting data in (prerequisite)

Revenue analytics needs a revenue source. Two paths:

1. **Connect a payment platform** — currently **Stripe** only. The simplest path. If you already have a Stripe-connected data warehouse source, just toggle **Stripe** on in [revenue settings](https://app.posthog.com/data-management/revenue). PostHog imports and transforms the data into managed views automatically. Stripe gives you deferred revenue and subscription/MRR metrics for free.
2. **Capture manual revenue events** — if you don't use Stripe. Send events with a revenue amount (and optional currency, product, coupon, subscription properties), then map those property names in the **Revenue** tab of data management. MRR and subscription metrics only appear if you configure a **subscription property**.

Then **choose a reporting currency** (settings). PostHog converts all revenue into it using hourly-updated exchange rates at event time.

See `reference/data-sources.md` for the full event property table, currency minor-unit / zero-decimal rules, churn-window config, and Stripe toggle details.

## Reading the dashboard (interpretation for decisions)

The dashboard opens with total revenue, paying customer count, and average revenue per customer for the selected time range. Key charts:

- **MRR / ARR** — recurring revenue from active subscriptions. Monthly view reports the value on the **last day of the month**. ARR is simply `MRR × 12` — a run-rate snapshot (not a forecast), but it's the first number investors ask for. *Decision:* track recurring-revenue health and report headline growth.
- **MRR breakdown** — splits MRR change into:
  - **New** — MRR from new customers
  - **Expansion** — existing customers paying more than last period
  - **Contraction** — existing customers paying less than last period
  - **Churn** — customers who cancelled OR whose charge was 0 this period
  *Decision:* diagnose *why* MRR moved. Falling net-new despite high gross-new → churn/contraction problem (focus retention). Strong expansion → upsell motion is working.
- **Gross revenue** — all revenue including one-time purchases and refunds (negative charges). *Decision:* total cash view vs recurring view.
- **Subscriptions & customers** — counts of active subs/customers plus gained/lost per period (needs a subscription property). Churned customers feed `churn rate = churned / total customers`, which feeds LTV.
- **ARPU** — average revenue per user across all revenue (not just subs). *Decision:* monetization efficiency, plan pricing.
- **LTV** — `ARPU / churn rate`. No churn rate → no datapoints; churn but no revenue → 0. *Decision:* compare against CAC mentally to judge unit economics.
- **Top customers** — biggest revenue contributors. Uses **deferred revenue** so prepaid multi-month subs are spread correctly. For data warehouse sources, names only show if you've connected customers to `persons`/`groups` (see below); otherwise you see the source identifier.

**Filters & breakdowns:** click **Add filter** next to the date range to filter by product name, initial coupon, customer ID, transaction amount, etc. Use **breakdown** to split any chart by product/plan/customer/country — this is how a PM answers "revenue by plan" or "revenue by region."

See `reference/metrics-glossary.md` for exact formulas and edge cases.

## Common PM workflows

**Answer "how is revenue trending and why?"**
1. Open the Revenue analytics dashboard, set the date range.
2. Read gross revenue (total) and MRR/ARR (recurring) trend.
3. Open MRR breakdown to attribute the change to New/Expansion/Contraction/Churn.
4. Decision: if churn/contraction dominate, prioritize retention; if expansion lags, prioritize upsell.

**Answer "what's churn costing us?"**
1. Look at Churn in the MRR breakdown (dollar impact) and churned customers in the subscriptions chart (logo churn).
2. Note the resulting churn rate and its effect on LTV.
3. Decision: quantify retention investment vs LTV uplift.

**Answer "which plan / cohort / segment drives revenue?"**
1. Use breakdown by product/plan, or filter by a plan/coupon.
2. For richer cohort/segment analysis, jump to the SQL editor managed views (`revenue_analytics.all.revenue_analytics_revenue_item`) or Product analytics insights.
3. Decision: where to invest product/pricing effort.

**Set and track a revenue target**
1. In [revenue settings](https://app.posthog.com/data-management/revenue) configure a revenue goal and choose whether it shows on the MRR/ARR or gross revenue chart.
2. The goal line renders on the dashboard if it's in range, or if it's past and not yet achieved.
3. Decision: monitor pacing to target.

**Make top customers show real names (data warehouse / Stripe)**
- Revenue from events auto-links to persons/groups. For Stripe you must tag customers with `posthog_person_distinct_id` metadata (on the customer, or on a charge/subscription/invoice for legacy customers). This also unlocks the `persons_revenue_analytics` / `groups_revenue_analytics` tables. See `reference/data-sources.md`.

## Deferred revenue (revenue recognition)

For data warehouse sources (Stripe), PostHog automatically **spreads revenue across the service period** instead of booking it all on payment day — matching accounting practice. A $1,200 annual prepayment shows as $100/month for 12 months. This is driven by the `period` (start/end) on Stripe invoice line items; nothing to configure if Stripe is filling it. Filtering to future dates shows revenue that *will* be recognized.

> Deferred revenue does **NOT** apply to manual revenue **events** — PostHog can't know which period an event belongs to. Defer it yourself before sending if you need it. Details in `reference/data-sources.md`.

## Going deeper with SQL (managed views)

Every source is transformed into 5 standardized [managed views](https://app.posthog.com/sql) (under the `views` tab): **product, charge, customer, subscription, revenue_item**. Prefer **revenue_item** over charge — it applies deferred revenue and currency conversion. Use these for custom revenue funnels, cohort revenue, segmentation, and anything the dashboard can't do.

> Managed views can't be used in the visual Insights builder yet. In Trends/Funnels, raw revenue is summed **without** currency conversion — use the `convertCurrency()` HogQL function or query the managed views directly. See `reference/managed-views-and-sql.md`.

## PostHog AI & MCP (fastest answers)

Ask MRR/ARR/churn/ARPU/LTV and "revenue by segment" questions in natural language, or have it build a revenue dashboard. PostHog AI (Max) answers in-app; the MCP server (`query-run`, `query-generate-hogql-from-question`, `insight-create-from-query`) brings the same to Cursor/Claude Code/VS Code. Tip: name the source ("from Stripe"/"from `purchase_completed` events") and the exact metric. See `reference/ai-and-mcp.md` and the `using-posthog-ai-assistant` skill.

## Related skills
- `using-posthog-ai-assistant` — natural-language revenue queries and auto-built dashboards via PostHog AI / MCP
- `building-insights` / `analyzing-product-data` — custom revenue funnels, trends, cohorts beyond the dashboard
- `connecting-data-warehouse-sources` — setting up Stripe and other data warehouse sources
- `analyzing-web-and-marketing` — marketing analytics, future ROAS/CAC once ad-spend sources land
