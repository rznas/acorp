# Breakdowns & group analytics — detail

Segmenting insights, and analyzing at company/account level. Sources: `product-analytics/trends/breakdowns.mdx`, `group-analytics.mdx`, `correlation.mdx`.

## Breakdowns
Split an insight by the values of a property or a cohort. Add via **+ Add breakdown**.

| Breakdown by | Example |
|---|---|
| Event properties | `Current URL`, `Country Name` |
| Session properties | `Entry URL` |
| Person properties | `company_name` |
| Feature flag | events where a flag was enabled |
| Group properties | needs group analytics |
| Cohorts | a predefined list of users (can use multiple cohorts at once) |
| SQL | custom, e.g. `concat(properties.$browser, ' - ', properties.$os)` |

- Trends load the first **25 breakdown values** (Load more for the rest); supports up to **3 breakdowns**. Other insight types: **1 breakdown** (funnels: a single cohort). SQL insights have no limit.
- **Numeric binning**: choose number of bins to group numeric values into ranges (e.g. 20–22, 22–24); aggregates all values, not just top 25.
- Total-value bar charts: enable **Stack breakdown values** to group stacked bars by series (useful with formulas).
- SQL label tip: add `AS column_name` or `-- column_name` for readable chart labels.

## Group analytics (company / account / channel / project level)
Aggregate events by an entity instead of a user. Set **Aggregating by** to a group type in an insight.

**Concepts**: a **group type** is a category ("company", "channel", "project") — max **5 per project**; a **group** is an instance ("Acme Corp"), unlimited per type. Set up requires code (`posthog.group(...)` / `groups` param) and identified events.

**PM use cases**:
- B2B product activation — daily active *companies*, company churn, % of companies completing onboarding, feature adoption by company (group funnels & trends).
- Channel/account retention — monthly active channels, retention by channel/account.
- Project-level feature flags & experiments — roll out / measure at the project or company level.

**In the UI**: People tab → Groups section to view a group's properties; "Unique [group]" aggregation in trends; "Aggregating by [group]" in funnels & retention; rename group types in Settings → Customer Analytics.

**Limitations**: max 5 group types; can't delete individual groups (only types); one event can't belong to two groups of the same type; **group types not supported for lifecycle insights or user paths**.

> Self-hosted / billing note: group analytics is a **paid add-on** (`free: none`). Once enabled, billing applies to **all identified events** in the project, not just events with group properties. A "Customer Analytics" preview is replacing the classic group analytics UI.

## Groups vs cohorts
- **Groups** aggregate events by an entity, require app code, don't need to map to users.
- **Cohorts** are sets of users with something in common, created in PostHog with no code.
- Use cohorts if you just need a list of users (simpler/faster); use groups for company/account-level metrics.

## Correlation analysis (finds what drives funnel conversion)
Inside a funnel → **Correlation analysis** → Load results / Select properties. Auto-ranks events/properties by how strongly they move conversion ("Nx more/less likely to convert"). Use when you have no hypothesis for the breakdown. Accuracy depends on sample size; exclude trivial events via `...` → Exclude event from project. Save completers/droppers as a cohort. Paid feature (`free: none`).
