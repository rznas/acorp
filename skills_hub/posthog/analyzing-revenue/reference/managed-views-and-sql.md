# Managed views & SQL for revenue

Revenue analytics transforms every source into 5 standardized **managed views**, auto-created when you add a revenue source. Query them in the SQL editor (`views` tab). Use them for analysis the dashboard can't do: custom revenue funnels, cohort revenue, segmentation, multi-source aggregation.

## Naming

- Event sources: `revenue_analytics.events.<event_name>.revenue_analytics_<view_type>`
- Data warehouse sources: `<source_name>.revenue_analytics_<view_type>` (e.g. `stripe.prod.revenue_analytics_revenue_item`)
- Global aggregate across all sources: `revenue_analytics.all.revenue_analytics_<view_type>`

## The 5 view types

| View | What it holds | PM use |
| --- | --- | --- |
| `product` | List of all products (from your product property or inferred from source) | Product/plan dimension |
| `charge` | Low-level raw charges, currency-converted, **no aggregation, no deferral** | Debugging raw data only |
| `customer` | All customers; connect to `persons`/`groups` for per-customer MRR | Customer dimension |
| `subscription` | All subscriptions (events need a subscription property) | Subscription analysis |
| `revenue_item` | All revenue items with **deferred revenue + currency conversion applied** | **Preferred view for revenue analysis** |

**Prefer `revenue_item` over `charge`** — it applies deferred revenue. `revenue_item` covers all invoices plus invoiceless charges (you can omit invoiceless charges in settings).

## Currency conversion caveats

- Conversion happens automatically **only** inside Revenue analytics and the managed views (the `amount` field is already converted).
- In Product analytics (Trends/Funnels/Experiments) "Sum of Revenue" sums **raw** values with **no** conversion.
- Managed views **cannot** be used in the visual Insights builder yet.

Two ways to get converted revenue in Trends/SQL:

Query a managed view directly (already converted):
```sql
SELECT toDate(timestamp) AS day, sum(amount) AS revenue_usd
FROM revenue_analytics.events.Order_Completed.revenue_analytics_revenue_item
GROUP BY day ORDER BY day
```

Use `convertCurrency()` in a Trends SQL expression or any query:
```sql
sum(convertCurrency(upper(properties.currency), 'USD',
    toDecimal(properties.revenue, 10), toDate(timestamp)))
```

Signature: `convertCurrency(from_currency, to_currency, value: Decimal, timestamp: Date|DateTime) -> Decimal`.

## Per-customer revenue tables

If customers are connected to persons/groups, you also get `persons_revenue_analytics` / `groups_revenue_analytics` (person_id/group_key → all-time revenue):
```sql
SELECT COUNT(*) FROM persons_revenue_analytics WHERE amount > 1000000
```

## Not yet supported (as of docs)

- Ad-spend sources → ROAS, CAC, CPC not yet computable (planned; see Marketing analytics).
- Custom currency conversion rates (Open Exchange Rates only).
