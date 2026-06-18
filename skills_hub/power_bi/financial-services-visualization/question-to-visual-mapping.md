# Financial Services: Question to Visualization Mapping

40+ common financial-services questions mapped to confirmed Power BI visuals, with the fact type, additivity note, and the DAX measure each needs. All visual names are confirmed against the Power BI visualization-types reference. Measures referenced by name are defined in `dax-measure-library.md`.

## Contents
- Banking: account balances (semi-additive)
- Banking: transactions
- Banking: customer segmentation (mini-dimensions)
- Banking: product profitability (supertype/subtype)
- Banking: households and joint accounts (weighted)
- Banking: risk and compliance
- Accounting / general ledger: budget vs actual
- Accounting / general ledger: hierarchies and balances
- Insurance: policies and premiums (pay-in-advance)
- Insurance: claims (accumulating snapshot)
- Insurance: underwriting and profitability

---

## Banking: account balances (semi-additive)

> Balance facts come from the monthly account periodic snapshot. They sum across accounts/products/branches but NOT across time.

**1. What is the total deposit balance right now?**
- Fact: account periodic snapshot. Semi-additive.
- Visual: **Card** (single number).
- Measure: `Ending Balance` (LASTNONBLANK). Never `SUM` across time.

**2. How have average account balances changed over time?**
- Visual: **Line chart**, month on axis.
- Measure: `Avg Balance` (AVERAGE). Do not plot `SUM(balance)`.

**3. What are total deposits vs total loans, with a target?**
- Visual: **KPI** (value = deposits, target = goal) or **Gauge**.
- Measure: `Total Deposits`, `Total Loans` (period-end), `Loan to Deposit Ratio`.

**4. Total balance by product category as of month-end?**
- Visual: **Clustered bar chart** (categories on Y for readability).
- Measure: `Ending Balance` evaluated within each product context (sum across products is valid in a single period).

**5. Balance distribution across balance bands?**
- Visual: **Column chart** (histogram-style, band on axis) or **Table**.
- Use a band definition table; group by band, COUNT accounts and `Ending Balance`.

## Banking: transactions

> Transaction counts and amounts are fully additive.

**6. Volume of transactions by type (deposit, withdrawal, transfer)?**
- Fact: account transaction. Additive.
- Visual: **Clustered column chart** or **Donut chart** (if few types).
- Measure: `Transaction Count` (SUM/COUNTROWS).

**7. Transaction trend over the past 12 months?**
- Visual: **Line chart**.
- Measure: `Transaction Count`.

**8. Transaction volume by day-of-week?**
- Visual: **Column chart** (day-of-week on axis); optionally split into multiple charts by channel.

**9. Channel mix shift over time (branch, ATM, online, mobile)?**
- Visual: **Stacked area chart** or **Ribbon chart** (ribbon highlights rank changes by period).

**10. Fee income trend vs interest income?**
- Visual: **Combo chart** (columns = fee income, line = interest income).

## Banking: customer segmentation (mini-dimensions)

> Banded mini-dimension attributes (credit-score band, income band, age band) are ideal grouping axes.

**11. How are customers distributed by age band and credit-score band?**
- Visual: **Matrix** (rows = age band, columns = credit-score band, values = customer count).

**12. Which segments hold the highest balances?**
- Visual: **Clustered bar chart**, segments ranked by `Avg Balance` or `Ending Balance`.

**13. Hierarchical segment breakdown (region to age band to income band)?**
- Visual: **Treemap** (size = customer count or balance).

**14. Relationship between credit score and balance?**
- Visual: **Scatter chart** (X = `credit_score_value` discrete fact, Y = balance). Use the discrete fact, not the band.

## Banking: product profitability (supertype/subtype)

**15. How do product types (checking, savings, mortgage, credit card) contribute to revenue?**
- Fact: supertype account snapshot. Visual: **Treemap** or **Donut chart**.
- Measure: `Total Revenue` by product type.

**16. Profitability trend by product type?**
- Visual: **Combo chart** (columns = profit, line = `Net Profit Margin`).

**17. Net interest margin by product line?**
- Visual: **Clustered bar chart** or **KPI** per line.
- Measure: `Net Interest Margin` (ratio — DIVIDE, never average).

**18. Overdraft rate by minimum-balance tier (checking deep dive)?**
- Fact: checking subtype snapshot. Visual: **Clustered column chart** or **Matrix**.
- Measure: `Overdraft Rate`.

## Banking: households and joint accounts (weighted)

> Joint accounts are linked via an account-to-customer bridge with weighting factors that sum to 1.00 per account.

**19. Total relationship value by household?**
- Visual: **Clustered bar chart**, top households ranked.
- Measure: `Weighted Balance` (SUMX over bridge with weighting_factor). Avoids double-counting joint accounts.

**20. Total balance exposure for customers age 65+ (impact, not allocation)?**
- Visual: **Card** or **Table**.
- Measure: `Exposure Balance` (ignores weights; may overcount intentionally for risk exposure).

**21. Household product penetration (how many product types per household)?**
- Visual: **Column chart** (product-count band on axis, household count as value).

## Banking: risk and compliance

**22. Which accounts exceed regulatory thresholds?**
- Visual: **Table** with conditional formatting (red flag) or **Scatter chart** (X = balance, Y = transaction volume, with threshold reference lines).

**23. Distribution of credit risk scores?**
- Visual: **Column chart** (risk-score band on axis).

**24. What factors drive loan default?**
- Visual: **Key influencers chart** (target = defaulted flag; explain by tenure, utilization, delinquency band).

**25. Non-performing loan ratio vs target?**
- Visual: **KPI** or **Gauge**.
- Measure: `NPL Ratio` (DIVIDE).

## Accounting / general ledger: budget vs actual

> Use a consolidated budget-vs-actual fact at common grain, or drill-across. Constrain to a single ledger.

**26. Variance between budget and actual by department?**
- Visual: **Clustered column chart** (budget vs actual side by side); **Waterfall chart** for the variance bridge.
- Measure: `Variance Amount`, `Variance Pct`.

**27. Budget-to-actual variance bridge (budget to favorable to unfavorable to actual)?**
- Visual: **Waterfall chart**.

**28. Which accounts have the largest budget overruns?**
- Visual: **Matrix** with conditional formatting (red = over budget), sorted by `Variance Amount`.

**29. YTD actual vs budget trend?**
- Visual: **Combo chart** (columns = monthly actual, line = budget) or **Line chart** with two series.
- Note: YTD computed in DAX via `TOTALYTD`, not stored as a fact.

**30. Budget remaining vs target?**
- Visual: **KPI** or **Card**.
- Measure: `Budget Remaining`.

## Accounting / general ledger: hierarchies and balances

**31. GL balance by account as of period-end?**
- Fact: GL periodic snapshot. Semi-additive.
- Visual: **Matrix** (account hierarchy on rows). Measure: `GL Ending Balance`.

**32. Average monthly GL balance across a fiscal year?**
- Visual: **Line chart** or **Card**. Measure: `Avg GL Balance` (AVERAGEX over months). Never SUM.

**33. Drill division to region to branch to account?**
- Visual: **Decomposition tree** or **Matrix** with drill-down enabled. Backed by an org-hierarchy bridge table for ragged hierarchies.

**34. Period activity (debits/credits) by account?**
- Fact: GL journal entry transactions. Additive.
- Visual: **Column chart** or **Table**.

## Insurance: policies and premiums (pay-in-advance)

> Premiums are pay-in-advance: track written and earned premium separately.

**35. Written premium vs earned premium over time?**
- Fact: premium periodic snapshot. Visual: **Combo chart** (columns = written, line = earned).
- Measures: `Written Premium`, `Earned Premium` (both additive within the snapshot).

**36. Policy distribution by coverage type and premium band?**
- Visual: **Matrix** (rows = coverage type, columns = premium band) or **Clustered column chart**.

**37. In-force policy count trend?**
- Fact: premium snapshot with status dimension. Visual: **Line chart**.
- Measure: `Policy Count` (count of active rows in period — semi-additive, use period status).

**38. Premium mix by line of business (auto, home, life)?**
- Visual: **Treemap** or **Donut chart**. Measure: `Written Premium` by LOB (supertype coverage).

## Insurance: claims (accumulating snapshot)

> Claim accumulating snapshot has multiple milestone dates (reported, opened, first payment, closed).

**39. How many claims are at each stage (reported to reserved to payment to closed)?**
- Visual: **Funnel chart** (stage counts) or **Column chart** by current status.

**40. Average time from claim report to payment?**
- Visual: **Card** or **Gauge**.
- Measure: `Avg Days To Settle` (averages the precomputed lag fact).

**41. Days-to-settlement trend?**
- Visual: **Line chart** (average lag by month).

**42. Current claim backlog vs target?**
- Visual: **Gauge** or **KPI**.
- Measure: `Open Claim Count`.

**43. High-value open claims with aging?**
- Visual: **Table** with conditional formatting on aging (red = oldest).

## Insurance: underwriting and profitability

**44. Loss ratio by coverage type?**
- Fact: consolidated policy/claim profit fact. Visual: **Clustered bar chart** or **Matrix**.
- Measure: `Loss Ratio` (DIVIDE claim loss / earned premium).

**45. Combined ratio vs the 100% breakeven line?**
- Visual: **KPI** or **Gauge** (target = 1.00), or **Bar chart** with a reference line.
- Measure: `Combined Ratio` (`Loss Ratio` + `Expense Ratio`).

**46. Underwriting profit by line of business?**
- Visual: **Waterfall chart** (earned premium to claim loss to expenses to underwriting profit) or **Combo chart**.
- Measure: `Underwriting Profit`.

**47. What factors drive high loss ratios?**
- Visual: **Key influencers chart** (target = loss ratio; explain by coverage, region, claimant attributes).
