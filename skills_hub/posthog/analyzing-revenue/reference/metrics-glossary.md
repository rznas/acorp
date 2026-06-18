# Revenue metrics glossary (formulas & edge cases)

Each metric, what it answers, the formula PostHog uses, and how to read it for a decision.

## MRR — Monthly Recurring Revenue
- Recurring revenue from customers with **active subscriptions**.
- Auto-calculated from your data. For event sources, only counted if a **subscription property** is configured.
- Monthly view reports the value on the **last day of the month**.
- Decision: headline recurring-revenue health and growth rate.

## ARR — Annual Run Rate
- `MRR × 12`. A projection of current MRR over 12 months, **not a forecast** — it assumes the same customers paying the same amounts.
- Decision: investor headline number; do not treat as predicted future revenue.

## MRR breakdown
Calculated day by day (aggregated to the month in monthly views):

| Component | Meaning |
| --- | --- |
| New MRR | MRR from new customers |
| Expansion | Existing customers who paid more this subscription period than last |
| Contraction | Existing customers who paid less this subscription period than last |
| Churn | Customers who cancelled OR whose charge was 0 this period |

- Decision: attribute MRR movement. Net MRR change = New + Expansion − Contraction − Churn.

## Gross revenue
- **All** revenue regardless of recurrence: includes one-time purchases and **negative charges (refunds)**.
- Decision: total cash picture vs the recurring (MRR) picture.

## Subscriptions & customers
- Counts of active subscriptions and customers (paying or not), plus gained (new) and lost (churned) per period.
- Meaningful only with a configured subscription property.
- **Churn rate** = `# churned customers / # total customers` → feeds LTV.

## ARPU — Average Revenue Per User
- Average revenue per user in the period across **all** revenue (not subscriptions only).
- Decision: monetization efficiency, pricing.

## LTV — Lifetime Value
- `ARPU / churn rate`.
- No churn rate → no datapoints shown. Churn rate but no revenue → shows 0.
- Decision: unit economics; compare against acquisition cost.

## Top customers
- Biggest revenue contributors, computed using **deferred revenue** (prepaid multi-month subs spread across months).
- Data warehouse sources show the source identifier unless customers are connected to `persons`/`groups` — then names show. See data-sources.md.

## Revenue goals
- Targets set in [revenue settings](https://app.posthog.com/data-management/revenue); you choose whether each shows on the MRR/ARR chart or the gross revenue chart.
- Displayed only if within the selected range, OR past but not yet achieved.

## Filters & breakdowns
- Filter the dashboard by product name, initial coupon, customer ID, transaction amount, and more (Add filter).
- Break down any chart by product/plan/customer/country to answer "revenue by X".
- For anything more complex, use Product analytics insights or the SQL editor managed views.
