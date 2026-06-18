# Financial Services DAX Measure Library

Copy-paste-ready, syntactically valid DAX for financial-services Power BI reports. Organized so the riskiest patterns (semi-additive balances, non-additive ratios, weighted aggregation) come first.

## Contents
- Conventions and table assumptions
- Semi-additive balance measures (CRITICAL)
- Banking KPIs (net interest margin, efficiency ratio, ROA, ROE, NPL, loan-to-deposit)
- Insurance KPIs (loss ratio, expense ratio, combined ratio)
- Weighted aggregation for households and joint accounts
- Non-additive ratio measures (margins)
- Budget vs actual (accounting / GL)
- Claim lag and pipeline measures
- Time-intelligence helpers (YTD)

---

## Conventions and table assumptions

Adjust table/column names to your model. Assumed tables:
- `AccountSnapshot` — monthly account periodic snapshot; columns `Balance`, `InterestIncome`, `InterestExpense`, `Fees`, `OperatingExpense`, `Date`.
- `Date` — date dimension marked as the date table, with `Date[Date]` and `Date[Month]`.
- `GLSnapshot` — GL periodic snapshot; `Balance`, `Date`.
- `BudgetActual` — consolidated fact; `ActualAmount`, `BudgetAmount`.
- `AccountCustomerBridge` — joint-account bridge; `WeightingFactor`.
- `PremiumSnapshot` — `WrittenPremium`, `EarnedPremium`.
- `ClaimSnapshot` — accumulating snapshot; `ClaimLoss`, `DaysToSettle`, `Status`.

All ratio measures use `DIVIDE(...)` so a zero or blank denominator returns BLANK instead of an error.

---

## Semi-additive balance measures (CRITICAL)

Balances must never be summed across time. Two valid time-aggregation rules:

**Period-end balance (as-of)** — use `LASTNONBLANK` to grab the latest non-blank date in context, then evaluate the balance there:

```dax
Ending Balance =
CALCULATE (
    SUM ( AccountSnapshot[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( AccountSnapshot[Balance] ) ) )
)
```

**Average balance over time** — correct for trend lines:

```dax
Avg Balance = AVERAGE ( AccountSnapshot[Balance] )
```

Average of monthly period-end balances over a fiscal range (e.g. average monthly GL balance):

```dax
Avg GL Balance =
AVERAGEX (
    VALUES ( 'Date'[Month] ),
    CALCULATE ( SUM ( GLSnapshot[Balance] ) )
)
```

GL period-end balance (same period-end pattern):

```dax
GL Ending Balance =
CALCULATE (
    SUM ( GLSnapshot[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( GLSnapshot[Balance] ) ) )
)
```

Deposits and loans as period-end balances (use these in cards, KPIs, gauges):

```dax
Total Deposits =
CALCULATE ( [Ending Balance], AccountSnapshot[AccountCategory] = "Deposit" )

Total Loans =
CALCULATE ( [Ending Balance], AccountSnapshot[AccountCategory] = "Loan" )
```

**Anti-pattern (do NOT use):** `Bad Balance = SUM ( AccountSnapshot[Balance] )` on a date axis overcounts every prior month.

---

## Banking KPIs

Net interest margin = (interest income - interest expense) / average earning assets. A ratio, so use DIVIDE over SUMs, never an average of per-row margins:

```dax
Net Interest Margin =
DIVIDE (
    SUM ( AccountSnapshot[InterestIncome] ) - SUM ( AccountSnapshot[InterestExpense] ),
    [Avg Earning Assets]
)

Avg Earning Assets =
CALCULATE ( [Avg Balance], AccountSnapshot[EarningAssetFlag] = TRUE () )
```

Efficiency ratio = operating expense / (net interest income + fee income). Lower is better:

```dax
Efficiency Ratio =
DIVIDE (
    SUM ( AccountSnapshot[OperatingExpense] ),
    ( SUM ( AccountSnapshot[InterestIncome] ) - SUM ( AccountSnapshot[InterestExpense] ) )
        + SUM ( AccountSnapshot[Fees] )
)
```

Return on assets (ROA) = net income / average total assets:

```dax
Net Income =
( SUM ( AccountSnapshot[InterestIncome] ) - SUM ( AccountSnapshot[InterestExpense] ) )
    + SUM ( AccountSnapshot[Fees] )
    - SUM ( AccountSnapshot[OperatingExpense] )
    - SUM ( AccountSnapshot[ProvisionForLosses] )

ROA = DIVIDE ( [Net Income], [Avg Total Assets] )

Avg Total Assets =
CALCULATE ( [Avg Balance], AccountSnapshot[AccountCategory] IN { "Loan", "Investment" } )
```

Return on equity (ROE) = net income / average equity (equity from a separate `Equity` table or measure):

```dax
ROE = DIVIDE ( [Net Income], [Avg Equity] )

Avg Equity = AVERAGE ( Equity[EquityAmount] )
```

Loan-to-deposit ratio:

```dax
Loan to Deposit Ratio = DIVIDE ( [Total Loans], [Total Deposits] )
```

Non-performing loan ratio:

```dax
NPL Ratio =
DIVIDE (
    CALCULATE ( [Ending Balance], AccountSnapshot[NonPerformingFlag] = TRUE () ),
    [Total Loans]
)
```

Overdraft rate (checking subtype):

```dax
Overdraft Rate =
DIVIDE (
    CALCULATE ( COUNTROWS ( CheckingSnapshot ), CheckingSnapshot[OverdraftCount] > 0 ),
    COUNTROWS ( CheckingSnapshot )
)
```

---

## Insurance KPIs

Loss ratio = claim losses / earned premium:

```dax
Loss Ratio =
DIVIDE ( SUM ( ClaimSnapshot[ClaimLoss] ), SUM ( PremiumSnapshot[EarnedPremium] ) )
```

Expense ratio = underwriting expenses / written premium:

```dax
Expense Ratio =
DIVIDE ( SUM ( PremiumSnapshot[UnderwritingExpense] ), SUM ( PremiumSnapshot[WrittenPremium] ) )
```

Combined ratio = loss ratio + expense ratio (below 1.00 = underwriting profit):

```dax
Combined Ratio = [Loss Ratio] + [Expense Ratio]
```

Written and earned premium (additive within the premium snapshot):

```dax
Written Premium = SUM ( PremiumSnapshot[WrittenPremium] )
Earned Premium  = SUM ( PremiumSnapshot[EarnedPremium] )
```

Underwriting profit:

```dax
Underwriting Profit =
[Earned Premium]
    - SUM ( ClaimSnapshot[ClaimLoss] )
    - SUM ( PremiumSnapshot[UnderwritingExpense] )
```

---

## Weighted aggregation for households and joint accounts

Joint accounts link to multiple customers via `AccountCustomerBridge[WeightingFactor]` (factors sum to 1.00 per account). Two distinct, correct measures:

**Allocation (correctly weighted)** — for relationship-value and household totals; no double-counting:

```dax
Weighted Balance =
SUMX (
    AccountCustomerBridge,
    RELATED ( AccountSnapshot[Balance] ) * AccountCustomerBridge[WeightingFactor]
)
```

**Impact (unweighted exposure)** — for risk exposure ("total balance touching customers age 65+"); intentionally may overcount:

```dax
Exposure Balance =
SUMX ( AccountCustomerBridge, RELATED ( AccountSnapshot[Balance] ) )
```

Use `Weighted Balance` in clustered bar charts of top households; use `Exposure Balance` only for exposure analysis, never for summed reporting totals.

---

## Non-additive ratio measures (margins)

Ratios cannot be averaged. Always DIVIDE over the summed components:

```dax
Net Profit Margin =
DIVIDE ( [Net Income], SUM ( AccountSnapshot[Revenue] ) )

Total Revenue = SUM ( AccountSnapshot[Revenue] )
```

---

## Budget vs actual (accounting / GL)

Constrain to a single ledger upstream (via a filter or a pre-constrained view) to avoid double-counting.

```dax
Actual Amount = SUM ( BudgetActual[ActualAmount] )
Budget Amount = SUM ( BudgetActual[BudgetAmount] )

Variance Amount = [Actual Amount] - [Budget Amount]

Variance Pct = DIVIDE ( [Variance Amount], [Budget Amount] )

Budget Remaining = [Budget Amount] - [Actual Amount]
```

YTD actual (compute to-date totals in DAX, never store YTD as a fact):

```dax
Actual YTD = TOTALYTD ( [Actual Amount], 'Date'[Date] )
Budget YTD = TOTALYTD ( [Budget Amount], 'Date'[Date] )
```

---

## Claim lag and pipeline measures

Lag facts are precomputed in the accumulating snapshot; average them:

```dax
Avg Days To Settle = AVERAGE ( ClaimSnapshot[DaysToSettle] )

Open Claim Count =
CALCULATE ( COUNTROWS ( ClaimSnapshot ), ClaimSnapshot[Status] = "Open" )

Closed Claim Count =
CALCULATE ( COUNTROWS ( ClaimSnapshot ), ClaimSnapshot[Status] = "Closed" )

Policy Count =
CALCULATE ( COUNTROWS ( PremiumSnapshot ), PremiumSnapshot[Status] = "Active" )
```

---

## Time-intelligence helpers (YTD)

General-purpose YTD wrapper for any additive measure:

```dax
Transaction Count = COUNTROWS ( AccountTransaction )

Transaction Count YTD = TOTALYTD ( [Transaction Count], 'Date'[Date] )
```

Do not apply `TOTALYTD` directly to a semi-additive balance — use `Ending Balance` for as-of values or `Avg Balance` for trends instead.
