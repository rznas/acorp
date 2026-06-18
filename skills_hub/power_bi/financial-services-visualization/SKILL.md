---
name: financial-services-visualization
description: Recommends Power BI visualizations and DAX measures for financial-services reports across banking (account balances, transactions, heterogeneous products), accounting/general-ledger (budgets, budget-vs-actual), and insurance (policies, premiums, claims). Maps 40+ stakeholder questions to specific Power BI visuals, provides dashboard templates (bank executive, branch manager, CFO budget-vs-actual, insurance claims, underwriting), and a DAX measure library (net interest margin, efficiency ratio, loss ratio, combined ratio, ROA, ROE). Emphasizes correct semi-additive balance handling (LASTNONBLANK / AVERAGE / period-end) and weighted aggregation for households and joint accounts. Use when building Power BI reports or dashboards for banks, credit unions, insurance companies, or accounting/GL systems, or when handling account balances, premiums, claims, or budget-vs-actual.
---

# Financial Services Visualization

Recommend Power BI visualizations for banking, accounting/general-ledger, and insurance reporting. The defining hazard in this domain is **semi-additive balance facts** (account balances, GL balances, reserves): they must NEVER be summed across time. Get additivity right first, then pick the visual.

## Prerequisite skills

This skill assumes the Layer 1 foundations. Read them first when needed:
- `visualizing-dimensional-models` — fact-type to visual affinity, grain, cardinality
- `mapping-questions-to-charts` — question-intent to chart category
- `understanding-fact-additivity` — additive / semi-additive / non-additive aggregation (critical here)

## Workflow

Copy this checklist and work through it for each visualization request:

```
- [ ] Step 1: Identify the fact type (transaction / periodic snapshot / accumulating snapshot)
- [ ] Step 2: Classify each measure's additivity (additive / semi-additive / non-additive)
- [ ] Step 3: For semi-additive balances, choose the time-aggregation rule (period-end vs average)
- [ ] Step 4: Map the question intent to a confirmed Power BI visual
- [ ] Step 5: Write the DAX measure (use the library; never SUM a balance over time)
- [ ] Step 6: Validate against the additivity guardrails below
```

## Step 1: Fact type drives the visual family

| Fact type | Financial examples | Visual affinity |
|-----------|--------------------|-----------------|
| Transaction | GL journal entries, deposits/withdrawals, policy transactions, claim tasks | Line chart, column chart, decomposition tree, scatter |
| Periodic snapshot | Account balances (monthly), GL account balances, premium snapshot | Card, KPI, gauge, line chart (with AVERAGE) |
| Accumulating snapshot | Claim lifecycle, policy lifecycle, budget→commitment→payment | Funnel chart, card/gauge (lag), column chart by stage |

## Step 2-3: Semi-additive balance guardrails (MOST IMPORTANT)

Account balances and GL balances are **semi-additive**: they sum across accounts, products, branches, and customers, but **NOT across time**. Summing a balance over months overcounts.

| Operation | Rule | DAX approach |
|-----------|------|--------------|
| Total balance "as of" a period | Period-end value | `LASTNONBLANK` over the date column |
| Balance trend over time | Average across the time grain | `AVERAGE` / `AVERAGEX` |
| Balance across accounts/products (same date) | Sum is fine | `SUM` (within a single period context) |
| GL balance over a fiscal year | Average across periods, never SUM | `AVERAGEX(VALUES(Date[Month]), ...)` |

Never put a raw `SUM(balance)` on a line chart with a date axis. See `dax-measure-library.md` for the exact patterns.

## Step 4: Question to visual mapping

Quick reference for the most common questions. The full 40+ question catalog is in `question-to-visual-mapping.md`.

| Question intent | Power BI visual |
|-----------------|-----------------|
| Total deposits / loans "right now" | Card or KPI (period-end balance) |
| Balance trend over time | Line chart (AVERAGE balance) |
| Net interest margin vs target | Gauge or KPI |
| Transaction volume by type | Clustered column chart or donut chart |
| Product-type revenue mix | Treemap or donut chart |
| Budget vs actual by department | Clustered column chart; Waterfall for variance bridge |
| Drill division to branch to account | Decomposition tree or matrix |
| Claim pipeline (reported to closed) | Funnel chart |
| Written vs earned premium over time | Combo chart |
| Top households by relationship value | Clustered bar chart (weighted) |
| What drives loan default / churn | Key influencers chart |
| Accounts exceeding a threshold | Table with conditional formatting; scatter chart |

All visual names above are confirmed Power BI visuals. Do not substitute invented chart types.

## Step 5: DAX measure library

Financial KPIs (net interest margin, efficiency ratio, loss ratio, combined ratio, ROA, ROE) plus the semi-additive balance and weighted-aggregation patterns live in:

**[dax-measure-library.md](dax-measure-library.md)** — copy-paste-ready, syntactically valid DAX.

Two rules that prevent the most common errors:
- **Ratios are non-additive.** Compute net interest margin / loss ratio / margin % as `DIVIDE(SUM(numerator), SUM(denominator))`, never `AVERAGE` of a per-row ratio.
- **Joint accounts and households need weighted aggregation.** Use `SUMX(bridge, balance * weighting_factor)` to avoid double-counting shared accounts. See the library for the impact-vs-allocation distinction.

## Step 6: Validate

Before finalizing, confirm:
- [ ] No balance fact is summed across a date axis (use period-end or average)
- [ ] Every ratio uses `DIVIDE(SUM(...), SUM(...))`, not an average of ratios
- [ ] GL queries constrain to a single ledger (Final Approved / Tax / GAAP) to avoid double-counting
- [ ] Heterogeneous-product reports use supertype (cross-product) vs subtype (line-of-business) deliberately — see below
- [ ] Household/joint-account totals apply weighting factors

## Heterogeneous products: which schema to visualize

Banks and insurers sell heterogeneous products (checking, mortgage, credit card; auto, home, life) to one customer base, modeled as supertype/subtype schemas.

- **Cross-product questions** ("total relationship value by household", "product mix", "profit by product type") → query the **supertype** snapshot fact. Visuals: treemap, donut, clustered bar.
- **Line-of-business deep dives** ("overdraft rate by minimum-balance tier", "premium by vehicle type") → query the **subtype** fact. Visuals: matrix, scatter, clustered column.

Subtype fact tables duplicate the common supertype facts (balance, fees), so a single subtype visual needs no cross-fact join.

## Mini-dimensions: segmentation visuals

Large customer/account bases use Type 4 mini-dimensions (credit bureau band, demographics band, behavior score band) to track rapidly changing attributes. These banded attributes are ideal slicer and matrix axes:
- Segment matrix: rows = age band, columns = credit-score band, values = customer count or AVERAGE balance.
- Use the banded attribute for filtering/grouping; use the discrete fact value (e.g. `credit_score_value`) for scatter charts and precise analysis.

## Reference files

- **[question-to-visual-mapping.md](question-to-visual-mapping.md)** — 40+ financial-services questions to specific visuals, by sub-domain (banking, accounting/GL, insurance), with the DAX measure each needs.
- **[dax-measure-library.md](dax-measure-library.md)** — DAX for financial KPIs, semi-additive balances, weighted household aggregation.
- **[dashboard-templates.md](dashboard-templates.md)** — bank executive, branch manager, CFO budget-vs-actual, insurance claims, underwriting dashboards.

---
*Provenance: Implements SKILL 5 (Financial Services Visualization, Layer 2) of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
