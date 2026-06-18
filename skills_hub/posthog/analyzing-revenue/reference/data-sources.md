# Revenue data sources: setup reference

Table of contents
- [Choosing a source](#choosing-a-source)
- [Option A: Connect Stripe](#option-a-connect-stripe)
- [Option B: Capture revenue events](#option-b-capture-revenue-events)
- [Reporting currency & minor units](#reporting-currency--minor-units)
- [Subscription / churn configuration](#subscription--churn-configuration)
- [Deferred revenue by source](#deferred-revenue-by-source)
- [Connecting revenue to customers (persons/groups)](#connecting-revenue-to-customers-personsgroups)

## Choosing a source

| If... | Use |
| --- | --- |
| Your billing is in Stripe | Connect Stripe (data warehouse source) — gives MRR, subscriptions, deferred revenue automatically |
| No supported platform, or you want manual control | Capture revenue events |

> Self-hosted note: payment-platform / data-warehouse source connections rely on the PostHog data warehouse. Availability and managed-source options may differ on self-hosted; confirm the data warehouse and the source are enabled on your instance.

## Option A: Connect Stripe

- Currently the **only supported payment platform** for revenue analytics. More platforms are planned.
- If you already have a Stripe-connected data warehouse source: open [revenue settings](https://app.posthog.com/data-management/revenue) and toggle **Stripe** on.
- Data is imported, transformed into [managed views](managed-views-and-sql.md), and powers the dashboard automatically.
- **Legacy data warehouse sources have revenue tracking disabled by default** — enable them in the Revenue tab and wait for the first sync to finish, or revenue won't appear.

## Option B: Capture revenue events

Send an event with revenue properties. Event and property **names are arbitrary** — you map them later in the **Revenue** tab of [data management](https://app.posthog.com/data-management/revenue).

| Property | Required? | Description | Example |
| --- | --- | --- | --- |
| Revenue | Yes | Amount, usually in minor unit (cents) | `revenue: 1000` |
| Currency | Optional (recommended) | ISO 4217 code; or set one static currency | `currency: "USD"` |
| Product | Optional | Product/plan name (enables product breakdowns) | `product: "Premium Plan"` |
| Coupon | Optional | Coupon used | `coupon: "SUMMER2024"` |
| Subscription | Optional | Subscription identifier — **required for MRR & subscription metrics** | `subscription_id: "sub_1234"` |

Illustrative capture (a PM specs this; engineering implements):

```js
posthog.capture("purchase_completed", {
    revenue: 1000, currency: "USD",
    product: "Premium Plan", coupon: "SUMMER2024",
    subscription_id: "sub_1234"
})
```

Key PM implications:
- **No subscription property → no MRR, ARR, or subscription/customer counts.** Only gross revenue/ARPU work.
- Currency is optional but recommended; configure a static currency if you bill in one currency only.

## Reporting currency & minor units

- Pick a reporting currency in settings; PostHog converts all revenue into it using Open Exchange Rates (updated hourly, daily granularity, rate at event time).
- **Minor unit ("values are in cents")**: send `2045` for $20.45. Recommended over decimals to avoid floating-point errors.
  - Most currencies treated as two-decimal: `1045 USD` = $10.45.
  - **Zero-decimal currencies** treated as whole: BIF, CLP, DJF, GNF, JPY, KMF, KRW, MGA, PYG, RWF, UGX, VND, VUV, XAF, XOF, XPF. So `1045 JPY` = ¥1045.
  - **Exception:** ISK (Icelandic Krona) is zero-decimal in reality but PostHog treats it as two-decimal for Stripe backward compatibility.
- Unsupported currency on an event → tracked as `0` value (rare given broad support).

## Subscription / churn configuration

For subscription products you can configure, in the Revenue tab:
- How long after a subscription's **last event** to consider it churned (drop-off window).
- Whether churn is counted on the day of the last event or after the drop-off window elapses.

This affects churned-customer counts, churn rate, and therefore LTV.

## Deferred revenue by source

| Source | Deferred revenue? | How |
| --- | --- | --- |
| Stripe (data warehouse) | Yes, automatic | Reads `period` (start/end Unix timestamps) on invoice line items; splits amount evenly across the service months |
| Manual events | No | PostHog can't know the period; defer it yourself before sending |

Effect: a $1,200 annual prepayment shows as $100 in each of 12 months instead of $1,200 in month one. Future-dated ranges show revenue that will be recognized later. Only applies when the period spans multiple billing cycles; one-time and monthly charges are recognized immediately.

## Connecting revenue to customers (persons/groups)

Required so **Top customers** shows names (not just source IDs) and to get per-customer MRR.

- **Events**: auto-linked to the person/group on the event. Nothing to do.
- **Stripe / data warehouse**: tag the Stripe customer with `posthog_person_distinct_id` metadata.
  - New customers: add it on `stripe.Customer.create`.
  - Existing/legacy customers: add it on any **charge, subscription, or invoice** (or Checkout session via `subscription_data`/`payment_intent_data`, plus `client_reference_id`). PostHog resolves it from the most recently created child object.

Illustrative:

```python
stripe.Customer.create(email=user.email,
    metadata={"posthog_person_distinct_id": user.posthog_distinct_id})
```

Once linked you also get the `persons_revenue_analytics` and `groups_revenue_analytics` tables (a map of person_id/group_key → all-time revenue):

```sql
SELECT COUNT(*) FROM persons_revenue_analytics WHERE amount > 1000000
```
