# Financial Services Dashboard Templates

Five audience-specific dashboard layouts. Each lists the visuals (all confirmed Power BI visual types), the measures they use (defined in `dax-measure-library.md`), and the additivity caution that applies. Lay out top-left = most important for Western reading order; keep under ~20 visuals per page.

## Contents
- Bank Executive Dashboard
- Branch Manager Dashboard
- CFO Budget-vs-Actual Dashboard
- Insurance Claims Manager Dashboard
- Insurance Underwriting Dashboard

---

## Bank Executive Dashboard

Audience: C-suite. Goal: enterprise health at a glance across all product lines (supertype view).

| Position | Visual | Content | Measures |
|----------|--------|---------|----------|
| Top row | **Card** / **KPI** (x4) | Total deposits, total loans, net interest margin, NPL ratio | `Total Deposits`, `Total Loans`, `Net Interest Margin`, `NPL Ratio` |
| Middle left | **Line chart** | Average balance trend by account type | `Avg Balance` |
| Middle right | **Waterfall chart** | Profit bridge: interest income to interest expense to provisions to operating costs to net income | components of `Net Income` |
| Bottom left | **Treemap** | Customer segments by profitability (supertype) | `Net Income` by segment |
| Bottom right | **Filled map (choropleth)** | Branch performance by geography | `Net Income` or `Ending Balance` |
| Left sidebar | **Slicer** (x3) | Date range, region, product type | — |

Additivity caution: deposit/loan cards use `Total Deposits` / `Total Loans` (period-end), NOT `SUM`. The balance line chart uses `Avg Balance`. Net interest margin and NPL are ratios via `DIVIDE`.

---

## Branch Manager Dashboard

Audience: branch operations. Goal: this branch's daily performance and book of business.

| Position | Visual | Content | Measures |
|----------|--------|---------|----------|
| Top row | **KPI** / **Card** | Branch deposits vs target, new accounts MTD, transaction count | `Total Deposits`, `Transaction Count` |
| Top right | **Gauge** | Branch balance vs target reserve | `Ending Balance` (target = goal) |
| Middle | **Matrix** | Product performance: rows = product, values = balance, account count, fees | `Ending Balance`, `Fees` |
| Lower left | **Column chart** | Transaction volume by type | `Transaction Count` |
| Lower right | **Clustered bar chart** | Top households by relationship value (weighted) | `Weighted Balance` |
| Bottom | **Table** w/ conditional formatting | Accounts breaching thresholds (red flag) | `Ending Balance`, threshold logic |
| Sidebar | **Slicer** | Date range, customer segment band | — |

Additivity caution: top-households bar uses `Weighted Balance` (SUMX with weighting factor) to avoid double-counting joint accounts.

---

## CFO Budget-vs-Actual Dashboard

Audience: CFO / FP&A. Goal: accounting/GL variance analysis. Pre-constrain to a single ledger.

| Position | Visual | Content | Measures |
|----------|--------|---------|----------|
| Top row | **Card** / **KPI** | YTD revenue, YTD expenses, variance %, budget remaining | `Actual YTD`, `Variance Pct`, `Budget Remaining` |
| Upper middle | **Waterfall chart** | Budget to favorable variances to unfavorable variances to actual | `Variance Amount` |
| Center | **Matrix** w/ conditional formatting | Departments x (budget, actual, variance %), red = over budget | `Budget Amount`, `Actual Amount`, `Variance Pct` |
| Lower left | **Combo chart** | Monthly actual (columns) vs budget (line) | `Actual Amount`, `Budget Amount` |
| Lower right | **Decomposition tree** | Drill division to department to account to line item | `Variance Amount` |
| Sidebar | **Slicer** | Fiscal period, ledger, organization | — |

Additivity caution: YTD figures use `TOTALYTD`, never a stored YTD fact. Ledger slicer must select exactly one ledger to prevent double-counting. GL balances (if shown) use `GL Ending Balance` or `Avg GL Balance`.

---

## Insurance Claims Manager Dashboard

Audience: claims operations. Goal: pipeline throughput and settlement speed from the claim accumulating snapshot.

| Position | Visual | Content | Measures |
|----------|--------|---------|----------|
| Top left | **Gauge** / **KPI** | Current claim backlog vs target | `Open Claim Count` |
| Top right | **Card** | Average days to settlement | `Avg Days To Settle` |
| Center | **Funnel chart** | Claim progression: reported to reserved to payment to closed | stage counts |
| Lower left | **Line chart** | Average days-to-settlement trend | `Avg Days To Settle` by month |
| Lower right | **Clustered column chart** | Claim counts by coverage type | `Open Claim Count` / `Closed Claim Count` |
| Bottom | **Table** w/ conditional formatting | High-value open claims, aging color scale | `ClaimLoss`, aging |
| Sidebar | **Slicer** | Date range, coverage, status | — |

Additivity caution: lag metrics average the precomputed `DaysToSettle` fact. Open-claim counts use period status, not cumulative sums.

---

## Insurance Underwriting Dashboard

Audience: underwriters / actuarial. Goal: profitability and risk of the book (consolidated policy/claim profit fact).

| Position | Visual | Content | Measures |
|----------|--------|---------|----------|
| Top row | **KPI** / **Card** | Loss ratio, expense ratio, combined ratio (target 1.00), written premium | `Loss Ratio`, `Expense Ratio`, `Combined Ratio`, `Written Premium` |
| Upper middle | **Combo chart** | Written premium (columns) vs earned premium (line) over time | `Written Premium`, `Earned Premium` |
| Center left | **Clustered bar chart** | Loss ratio by coverage type | `Loss Ratio` |
| Center right | **Waterfall chart** | Underwriting profit: earned premium to claim loss to expenses to profit | components of `Underwriting Profit` |
| Lower | **Key influencers chart** | What drives high loss ratios (explain by coverage, region, claimant attributes) | `Loss Ratio` |
| Sidebar | **Slicer** | Date range, line of business, coverage | — |

Additivity caution: written and earned premium are tracked as separate facts (pay-in-advance revenue recognition); never substitute one for the other. All ratios use `DIVIDE`.
