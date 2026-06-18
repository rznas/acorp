---
name: industry-financial-patterns
description: Apply dimensional modeling patterns for financial services industries including banking, accounting/general ledger, and insurance. Covers account balances (semi-additive facts), heterogeneous products with supertype/subtype schemas, budget vs actual consolidation, ragged organizational hierarchies, household dimensions, mini-dimensions for large customer bases, and pay-in-advance revenue recognition. Use when designing data warehouses for banks, insurance companies, accounting systems, or any financial institution with diverse product portfolios and complex customer relationships.
---

# Industry-Specific Patterns: Financial Services

## Overview

Financial services industries—including **banking**, **accounting/general ledger**, and **insurance**—share common dimensional modeling challenges despite their operational differences. This skill provides comprehensive patterns for modeling financial data across these sectors.

**Key Challenges in Financial Services:**
- **Heterogeneous products** with widely varying attributes and facts (checking vs mortgages vs insurance policies)
- **Large dimension tables** (millions of customers/accounts/policies)
- **Pay-in-advance revenue** requiring both written and earned metrics
- **Complex customer relationships** (households, multiple account holders, beneficiaries)
- **Semi-additive balance facts** that cannot sum across time
- **Ragged organizational hierarchies** of indeterminate depth
- **Regulatory requirements** for historical accuracy and audit trails

**When to Use This Skill:**
- Designing DW/BI systems for retail banks, credit unions, investment firms
- Modeling general ledger, budgeting, and financial statement data
- Building insurance data warehouses (property/casualty, life, health)
- Handling any business with heterogeneous product portfolios sold to same customer base
- Dealing with account-based or policy-based business models

## Core Financial Services Patterns

### Pattern 1: Accounting & General Ledger

**Business Process:** General ledger periodic snapshot and journal entry transactions

<business_context>
The general ledger is the foundation financial system tying together subledgers for purchasing, payables, and receivables. Financial analysts need both summary period-end views and detailed transaction drill-down capabilities.
</business_context>

#### 1.1 General Ledger Periodic Snapshot

**Grain:** One row per general ledger account per accounting period

**Dimensions:**
- **Accounting Period** (month/fiscal period dimension)
- **Account** (chart of accounts with account type, rollup hierarchy)
- **Organization** (cost center with geographic + organizational hierarchies)
- **Ledger/Book** (multiple sets of books for tax, regulatory reporting)
- **Currency** (if multi-currency consolidation required)

**Facts:**
- **Balance Amount** (semi-additive—average across time, sum across accounts)
- **Period Activity** (debits and credits for the period—fully additive)

**Critical Design Decisions:**

<design_pattern type="semi-additive-facts">
**Semi-Additive Balance Facts:**
- Balance amounts do NOT add across time periods
- Must average balances by dividing sum by number of periods
- Include balance because recalculating from beginning of time is impractical
- Most BI tools and SQL can handle semi-additive aggregation correctly

```sql
-- CORRECT: Average balance across months
SELECT
  org.department,
  AVG(balance_amount) as avg_monthly_balance
FROM gl_periodic_snapshot f
JOIN organization org ON f.org_key = org.org_key
WHERE accounting_period BETWEEN '2024-01' AND '2024-12'
GROUP BY org.department;

-- INCORRECT: Summing balances across time
SELECT SUM(balance_amount) -- WRONG! Overcounts balances
FROM gl_periodic_snapshot
WHERE accounting_period BETWEEN '2024-01' AND '2024-12';
```
</design_pattern>

<design_pattern type="ledger-dimension-warning">
**Ledger Dimension Warning:**
The ledger dimension enables multiple ledgers (Final Approved, Tax, GAAP, IFRS) in same fact table. However, **every query MUST constrain to a single ledger value** or results will double-count.

**Best Practice:** Release separate views with ledger pre-constrained:
- `gl_snapshot_final` (WHERE ledger = 'Final Approved')
- `gl_snapshot_tax` (WHERE ledger = 'Tax Ledger')

This prevents users from accidentally querying across ledgers.
</design_pattern>

<design_pattern type="year-to-date-facts">
**Avoid Year-to-Date Facts in Fact Table:**
- Do NOT store QTD, YTD, or other to-date totals as facts
- These violate grain (not true to single period)
- Cause nonsensical overcounting when summarized arbitrarily
- Calculate in BI layer instead: `SUM(balance) OVER (PARTITION BY account ORDER BY period)`
</design_pattern>

#### 1.2 General Ledger Journal Entry Transactions

**Grain:** One row per journal entry line item

**Dimensions:**
- **Posting Date** (daily grain)
- **Effective Accounting Date** (role-playing date for accrual accounting)
- **Account**, **Organization**, **Ledger** (reused from periodic snapshot)
- **Journal Entry Type** (adjusting, closing, reversing, etc.)

**Degenerate Dimensions:**
- **Journal Entry Number** (groups related line items)

**Facts:**
- **Debit Amount** / **Credit Amount** (or single amount with debit/credit indicator)

**Purpose:** Enables drill-down from periodic snapshot anomalies to underlying transaction detail.

#### 1.3 Budgeting Process Chain

Financial organizations have a **series of events** in budgeting: budget creation → commitments → payments.

**Grain:** Net change of budget line item by organization and GL account by effective month

<budgeting_chain>
**Three Fact Tables in Chain:**

1. **Budget Fact Table**
   - Dimensions: Effective Month, Organization, Budget Line Item, GL Account
   - Facts: Budget Amount (net change)
   - Captures original budget + all modifications during year

2. **Commitments Fact Table**
   - Same dimensions + Commitment Document, Commitment Party
   - Facts: Committed Amount
   - Tracks purchase orders, work orders, contracts

3. **Payments Fact Table**
   - Same dimensions + Payment Type, Payee
   - Facts: Payment Amount
   - Actual cash transfers

**Grain Insight: Net Change, Not Snapshot**
- Each row represents CHANGES to budget in a given month
- Original $200K budget → row in January with $200K
- June adjustment +$40K → row in June with $40K
- October reduction -$25K → row in October with -$25K
- To get current budget: SUM all rows from beginning to present date
</budgeting_chain>

**Budget vs Actual Analysis:** Use drill-across or consolidated fact table (covered in Pattern 4).

#### 1.4 Ragged Organizational Hierarchies

Organizations often have **ragged hierarchies of indeterminate depth** (divisions → departments → cost centers with varying levels).

<hierarchy_solution>
**Solution: Bridge Table for Ragged Hierarchies**

Instead of recursive pointers in dimension table, create **organization map bridge table**:

**Bridge Table Grain:** Each path from parent to all children below that parent

| parent_org_key | child_org_key | ownership_pct | highest_parent_flag | lowest_child_flag |
|----------------|---------------|---------------|---------------------|-------------------|
| 1              | 1             | 1.00          | Y                   | N                 |
| 1              | 2             | 1.00          | Y                   | N                 |
| 1              | 3             | 1.00          | Y                   | Y                 |
| 2              | 2             | 1.00          | Y                   | N                 |
| 2              | 4             | 1.00          | Y                   | N                 |

**Query Pattern:**
```sql
-- Get budget for entire tree under Organization #1
SELECT SUM(f.budget_amount)
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization d ON b.parent_org_key = d.org_key
WHERE d.org_key = 1;  -- Constrain dimension to single parent

-- Get budget for leaf nodes only
WHERE d.org_key = 1 AND b.lowest_child_flag = 'Y';
```

**Advantages:**
- Traverse entire tree in single query without recursion
- Support multiple rollup structures (swap bridge tables)
- Handle shared ownership (50% to Div A, 50% to Div B)
- Track time-varying hierarchies (add effective/expiration dates to bridge)
- SCD Type 2 changes don't ripple through entire tree
</hierarchy_solution>

See `ragged-hierarchy-bridge-tables.md` for complete implementation patterns.

---

### Pattern 2: Banking & Financial Services

**Business Process:** Account balances (monthly snapshot) and account transactions

<business_context>
Retail banks offer heterogeneous products (checking, savings, mortgages, credit cards) to the same customer base. Business users need both enterprise-wide customer relationship views and line-of-business deep dives.
</business_context>

#### 2.1 Dimension Triage: Avoid "Too Few Dimensions"

**Anti-Pattern:** Collapsing everything into account dimension because "each account has only one household, branch, and product."

<dimension_triage>
**Proper Dimensional Model:**
- **Month** (periodic snapshot grain)
- **Account** (unique account identifier)
- **Product** (checking, savings, mortgage, credit card hierarchy)
- **Branch** (facility/location)
- **Primary Customer** (account holder)
- **Household** (economic unit grouping multiple accounts/customers)
- **Account Status** (active, inactive, new, closed, dormant)

**Why Separate Dimensions?**
1. **Performance:** Smaller entry points (product/branch) vs 10M-row account dimension
2. **Usability:** Mirror natural business thinking (users ask "show me by product")
3. **SCD Management:** Product/branch change independently; avoid bloating account dimension
4. **Many-to-Many:** Products and branches have M:N relationship

**Rule of Thumb:** Most dimensional models have 5-20 dimensions. If you have < 5, investigate:
- Causal dimensions (promotion, contract, weather)
- Multiple date dimensions (transaction date, effective date, due date)
- Degenerate dimensions (order number, invoice number)
- Role-playing dimensions (ship-to, bill-to, sold-to customer)
- Status dimensions (account status, application status)
- Audit dimensions (data lineage, ETL metadata)
- Junk dimensions (flags, indicators, yes/no attributes)
</dimension_triage>

#### 2.2 Household Dimension

**Purpose:** Analyze entire economic unit, not just individual accounts. Enable cross-sell/up-sell strategies.

**Challenges:**
- One household → multiple accounts (John's checking + Mary's savings + joint mortgage)
- One account → multiple account holders (joint checking)
- Relationships change over time (divorce, remarriage, address moves)

**Design Decision:** Treat account and household as **separate dimensions** with relationship captured in fact table.

**Household Attributes:**
- Household income range
- Own vs rent home
- Presence of children (Y/N)
- Retiree household (Y/N)
- Household demographic segment
- Household behavior scores

**Relationship Management:** Many households change monthly; capture relationship in monthly fact table row (not type 2 SCD on 10M-row account dimension).

#### 2.3 Multivalued Dimensions: Multiple Account Holders

**Problem:** Joint checking account with John and Mary. Customer cannot be account attribute (violates grain). Customer cannot be fact table dimension (violates grain).

**Solution: Account-to-Customer Bridge Table**

<bridge_table type="multivalued">
| account_key | customer_key | weighting_factor | effective_date | expiration_date |
|-------------|--------------|------------------|----------------|-----------------|
| 12345       | 111          | 0.50             | 2020-01-01     | 9999-12-31      |
| 12345       | 222          | 0.50             | 2020-01-01     | 9999-12-31      |

**Weighting Factors:**
- Sum to 1.00 for each account
- Allocate additive facts across account holders
- Enable two types of queries:

1. **Correctly Weighted Report:** Apply weights to get accurate totals
   ```sql
   SELECT c.customer_name, SUM(f.balance * b.weighting_factor) as allocated_balance
   FROM account_snapshot f
   JOIN account_customer_bridge b ON f.account_key = b.account_key
   JOIN customer c ON b.customer_key = c.customer_key
   GROUP BY c.customer_name;
   ```

2. **Impact Report:** Ignore weights to understand customer exposure (may overcount)
   ```sql
   SELECT c.demographic_segment, SUM(f.balance) as total_exposure
   FROM account_snapshot f
   JOIN account_customer_bridge b ON f.account_key = b.account_key
   JOIN customer c ON b.customer_key = c.customer_key
   GROUP BY c.demographic_segment;
   -- Useful for "What's total balance for customers age 65+?" (impact, not allocation)
   ```
</bridge_table>

**Alternative:** If each customer has unique identifier at transaction level (like unique credit card numbers for John and Mary), use customer directly as fact table dimension—no bridge needed.

#### 2.4 Mini-Dimensions for Large, Rapidly Changing Attributes

**Challenge:** 20M accounts × monthly credit bureau updates = unsustainable type 2 SCD growth

**Solution: Type 4 Mini-Dimensions**

<mini_dimensions>
**Break off volatile attributes into separate dimensions:**

**Account Snapshot Fact Table has multiple mini-dimension keys:**
- `account_key` (static attributes: account number, open date)
- `credit_bureau_mini_dim_key` (monthly updates: credit score band, delinquency status)
- `demographics_mini_dim_key` (annual updates: income band, age band, zip code)
- `behavior_mini_dim_key` (monthly updates: profitability score band, retention score band)

**Mini-Dimension Design:**
- Band continuous values: Income $31,257 → "$30K-$35K" band
- Pre-populate all combinations: 10 income bands × 8 age bands × 5 scores = 400 rows
- Reuse existing rows when attributes change (just update fact table foreign key)
- Keep row counts manageable (hundreds to low thousands)

**Three Storage Locations for Same Attribute:**
1. **Banded in mini-dimension:** "$30K-$35K" for routine reporting/filtering
2. **Discrete in fact table:** $31,257 as fact for data mining/precise analysis
3. **Current in main dimension:** Type 1 overwrite for "current state" queries

Label distinctly: `income_range` (mini-dim), `income_amount` (fact), `current_income_range` (dimension).
</mini_dimensions>

See `mini-dimensions-banking.md` for complete patterns and ETL strategies.

#### 2.5 Supertype/Subtype for Heterogeneous Products

**Dilemma:** Checking accounts have minimum balance, overdraft limit, online banking flags. Mortgages have interest rate, term, points, escrow amounts. Credit cards have APR, credit limit, rewards program. **Hundreds of product-specific attributes/facts** that don't apply across product types.

<supertype_subtype>
**Solution: Supertype + Subtype Schemas**

**1. Supertype Schema (Enterprise View)**
- **Fact Table:** `account_snapshot_supertype`
  - All accounts across all product types
  - Common facts only: balance, transaction_count, interest_paid, fees_charged
  - Common dimensions: month, account, product (generic), branch, customer, household

- **Product Dimension:** Generic attributes applicable to ALL products
  - Product name, product type, product category
  - Does NOT include checking-specific or mortgage-specific attributes

**2. Subtype Schemas (Line-of-Business Views)**
- **Checking Fact Table:** `account_snapshot_checking`
  - Checking accounts only (shrunken subset of supertype)
  - Includes supertype facts PLUS checking-specific facts
  - Checking-specific: overdraft_count, online_banking_logins, check_count

- **Checking Account Dimension:**
  - Same surrogate keys as supertype (conformed)
  - Includes generic attributes PLUS checking-specific attributes
  - Checking-specific: minimum_balance_requirement, overdraft_limit, free_checks_flag

- **Mortgage Fact Table:** `account_snapshot_mortgage`
- **Mortgage Account Dimension:** interest_rate, term_months, escrow_balance, points_paid

**Key Design Principles:**
- Same account → same surrogate key in supertype and subtype dimensions
- Subtype dimensions are shrunken conformed dimensions (subset of rows)
- Subtype fact tables contain supertype facts (avoid joins across super/sub)
- Users never see all tables—either enterprise view (supertype) or LOB view (one subtype)

**When to Use:**
- Cross-product customer analysis → supertype schema
- Deep-dive checking performance analysis → checking subtype schema
- Profit by product type → supertype schema
- Overdraft pattern analysis → checking subtype schema
</supertype_subtype>

See `heterogeneous-products-supertype-subtype.md` for detailed examples and variation patterns.

---

### Pattern 3: Insurance

**Business Process:** Policy transactions, premium snapshots, claim transactions, claim snapshots

<business_context>
Property/casualty insurance companies issue policies with coverages on covered items (house, car), collect premiums, and process claims. Business requires both transactional detail and lifecycle snapshots.
</business_context>

#### 3.1 Policy Transaction Fact Table

**Grain:** One row per policy transaction (create policy, alter coverage, rate, underwrite, cancel)

**Dimensions:**
- **Transaction Date** (when entered in system)
- **Effective Date** (when legally takes effect—role-playing date)
- **Policyholder** (insured party with SCD Type 1/2/3 for different attributes)
- **Coverage** (product—fire, flood, collision, liability)
- **Covered Item** (specific house, car, property)
- **Employee** (agent, underwriter)
- **Policy Transaction Type** (create, alter, cancel with reason)

**Degenerate Dimension:**
- **Policy Number** (if all policy attributes extracted to dimensions)

**Facts:**
- **Transaction Amount** (meaning depends on transaction type)

**Typical of Transaction Schemas:**
- Fact table mostly keys (highly dimensional at atomic grain)
- Minimal facts (often just one or two measurements)
- Enables extreme detail analysis and behavioral pattern discovery

#### 3.2 Premium Periodic Snapshot

**Grain:** One row per coverage on covered item on policy per month

**Why Snapshot Needed:** Cannot determine policy value or revenue by traversing transactions (too complex, business rules for revenue recognition are intricate).

<pay_in_advance>
**Pay-in-Advance Revenue Recognition:**

Insurance premiums are paid upfront for future coverage. Revenue is **earned over time** as coverage is provided.

**Two Premium Metrics Required:**

1. **Written Premium:** Amount contracted/paid
   - $600 annual policy written in January → written premium = $600 in Jan, $0 in Feb-Dec

2. **Earned Premium:** Revenue recognized as service delivered
   - $600 annual policy → earned premium = $50/month ($600 ÷ 12)
   - Policy canceled March 31 → earned $50 in Jan, Feb, Mar; written premium = -$450 in Mar

**Both metrics in same fact table:**
```sql
-- January row
written_premium_amount = 600
earned_premium_amount = 50

-- February row
written_premium_amount = 0
earned_premium_amount = 50

-- March row (canceled)
written_premium_amount = -450  -- Refund unearned portion
earned_premium_amount = 50     -- Earned through cancellation date
```

**Also applies to:**
- Magazine subscriptions (paid annual, earned monthly)
- Extended warranty contracts
- SaaS annual subscriptions
- Prepaid service contracts
</pay_in_advance>

**Conformed Dimensions:** Reuse policyholder, covered item, coverage. Replace daily date with month. Replace transaction type with **status dimension** (new, active, canceled, lapsed).

#### 3.3 Claim Transaction Fact Table

**Grain:** One row per claim task transaction

**Claim Task Types:**
- Open/reopen/close claim
- Set/reset/close reserve
- Adjuster inspection/interview
- Open/close lawsuit
- Make/receive payment
- Set/receive salvage estimate/payment
- Subrogate claim

**Dimensions:**
- **Transaction Date**, **Effective Date** (role-playing)
- **Claim** (claim identifier, claim type)
- **Claimant** (party making claim—often "dirty" dimension with duplicates)
- **Third-Party Payee** (doctor, lawyer, body shop—also "dirty")
- **Coverage**, **Covered Item**, **Policyholder** (reused from policy schemas)
- **Employee** (adjuster, claims processor—important for payment authorization)
- **Claim Transaction Type**
- **Claim Profile Junk Dimension** (low-cardinality claim attributes: report method, catastrophic event flag, fraud indicator)

**Facts:**
- **Transaction Amount** (reserve amount, payment amount, salvage amount)

**Dirty Dimensions:** Claimants and payees may deliberately obscure identity across claims (fraud detection use case).

#### 3.4 Claim Accumulating Snapshot

**Grain:** One row per claim (created at claim open, updated throughout lifecycle, finalized at close)

**Purpose:** Difficult to derive claim-to-date performance from transaction detail. Snapshot accumulates key metrics and tracks pipeline milestones.

**Multiple Date Dimensions (Milestones):**
- Claim Reported Date
- Claim Opened Date
- First Payment Date
- Adjuster Assigned Date
- Lawsuit Filed Date
- Claim Closed Date
- Claim Reopened Date (if applicable)

**Lag/Duration Facts:**
- Days from report to first payment
- Days from open to close
- Days claim was open
- (Calculate as: `first_payment_date - claim_reported_date` or more sophisticated workday logic)

**Accumulated Facts:**
- Total reserve amount
- Total payments to date
- Total salvage recovered
- Claim count (always 1 for grain verification)

**Status Dimension:** Open, closed, reopened, denied, in litigation

**Updated Throughout Lifecycle:**
- Open claim → insert row with opened date, null payment/closed dates
- First payment → update row with first payment date and amount
- Close claim → update row with closed date and final totals
- Reopen → update row with reopened date

See `accumulating-snapshot-complex-workflows.md` for patterns when workflow has 20+ unpredictable milestones.

#### 3.5 Consolidated Policy/Claim Profit Fact Table

**Purpose:** Combine premium revenue and claim loss for profit analysis. Common drill-across requirement.

**Grain:** Lowest common dimensionality—typically month, coverage, covered item

**Dimensions (Reduced Set):**
- Month
- Coverage
- Covered Item
- Policyholder (if grain permits)

**Facts:**
- Earned premium amount
- Written premium amount
- Claim loss amount
- Claim count
- Gross profit (earned premium - claim loss)

**Best Practice:** Build consolidated fact table AFTER atomic fact tables are in production. Consolidation sacrifices dimensionality for convenience.

---

### Pattern 4: Consolidated Fact Tables

**Definition:** Fact tables combining metrics from multiple business processes at common grain.

<consolidated_facts>
**When to Use:**
- Extremely common drill-across analyses (budget vs actual, premium vs claims)
- Performance optimization for frequent cross-process queries
- Executive dashboards requiring integrated metrics

**Example: Actual vs Budget Consolidated Fact Table (from Accounting)**

**Grain:** One row per GL account per organization per accounting period

**Dimensions:**
- Accounting Period
- Organization
- Account

**Facts:**
- Actual Amount (from GL periodic snapshot)
- Budget Amount (from budget fact table)
- Variance Amount (calculated: actual - budget)
- Variance Percent (calculated: variance / budget)

**Multi-Currency Handling:**
- Actual amount in local currency
- Actual amount in corporate currency (at effective rate)
- Actual amount in corporate currency (at planned rate)
- Enables variance analysis isolating FX impact from operational performance

**Critical Warning:**
- Consolidated facts must live at **common grain**
- Cannot force-fit different granularities with artificial facts/dimensions
- Retain atomic fact tables for full dimensionality/grain flexibility
- Consolidation = "least common denominator" dimensionality compromise
</consolidated_facts>

---

## Advanced Techniques

### Dynamic Value Banding of Facts

**Use Case:** Users want flexible balance ranges in reports, not fixed mini-dimension bands.

<value_banding>
**Report Example:**
| Balance Range    | Account Count | Total Balance |
|------------------|---------------|---------------|
| $0 - $1,000      | 45,234        | $23,456,789   |
| $1,001 - $5,000  | 23,456        | $67,890,123   |
| $5,001 - $10,000 | 12,345        | $89,012,345   |
| $10,001 and up   | 8,901         | $234,567,890  |

**Solution: Band Definition Table**

| band_group     | band_range_name  | lower_bound | upper_bound | sort_order |
|----------------|------------------|-------------|-------------|------------|
| Standard       | $0 - $1,000      | 0           | 1000        | 1          |
| Standard       | $1,001 - $5,000  | 1001        | 5000        | 2          |
| High_Net_Worth | $250K - $500K    | 250000      | 500000      | 1          |

**Query Pattern:**
```sql
SELECT
  b.band_range_name,
  COUNT(*) as account_count,
  SUM(f.balance_amount) as total_balance
FROM account_snapshot f
JOIN band_definition b
  ON f.balance_amount BETWEEN b.lower_bound AND b.upper_bound
  AND b.band_group = 'Standard'
WHERE f.snapshot_month = '2024-12'
GROUP BY b.band_range_name, b.sort_order
ORDER BY b.sort_order;
```

**Performance:** May require index on balance fact. Columnar databases excel at this pattern.
</value_banding>

---

## Financial Services Fact Table Types

### Summary Table

| Fact Table Type           | Grain                                    | When to Use                                  | Examples                            |
|---------------------------|------------------------------------------|----------------------------------------------|-------------------------------------|
| **Transaction**           | One row per event                        | Detailed behavior, audit trail               | GL journal entries, policy transactions, claim tasks |
| **Periodic Snapshot**     | One row per entity per period            | Semi-additive balances, period performance   | GL account balances, account balances, premium revenue |
| **Accumulating Snapshot** | One row per process instance             | Pipeline tracking, lifecycle metrics         | Budget→commitment→payment, policy lifecycle, claim lifecycle |
| **Consolidated**          | Common grain across processes            | Frequent drill-across, integrated metrics    | Budget vs actual, premium vs claims |

---

## Common Mistakes to Avoid

<mistakes>
1. **Storing YTD/QTD Totals in Fact Tables**
   - Violates grain, causes overcounting
   - Calculate in BI layer with window functions

2. **Using Operational Keys as Surrogate Keys**
   - Including effective date in dimension key
   - Use simple integer surrogate keys (1 to N)

3. **Snowflaking Fixed-Depth Hierarchies**
   - Keep account type → category → class hierarchy flattened in single dimension
   - Exception: Ragged hierarchies of indeterminate depth (use bridge table)

4. **Failing to Track Dimension Changes**
   - Type 1 only is rarely sufficient
   - Use Type 2 for critical attributes (ZIP code impacts pricing)
   - Use mini-dimensions for rapidly changing attributes in large dimensions

5. **Forcing Supertype/Subtype with Nulls**
   - Don't put all product-specific facts in single fact table (Swiss cheese with nulls)
   - Don't put all product-specific attributes in single dimension
   - Use separate subtype fact tables and dimensions

6. **Not Conforming Across Financial Systems**
   - "Revenue" in GL must match "Revenue" in budgeting
   - If definitions differ, use different names (net revenue, gross revenue, statutory revenue)
   - Conform account, organization, and period dimensions religiously

7. **Querying Across Multiple Ledgers**
   - Ledger dimension can cause double-counting
   - Pre-constrain with views (one view per ledger)

8. **Ignoring Semi-Additive Nature of Balances**
   - Balance facts don't sum across time
   - Use AVG aggregation or explicit period-weighting logic
</mistakes>

---

## Implementation Checklist

### Accounting / General Ledger
- [ ] GL periodic snapshot with semi-additive balance handling
- [ ] GL journal entry transaction fact table for drill-down
- [ ] Account dimension with chart of accounts hierarchy
- [ ] Organization dimension (consider ragged hierarchy bridge table if needed)
- [ ] Multiple fiscal calendar handling (outrigger or separate dimensions)
- [ ] Budget fact table with net change grain
- [ ] Commitments and payments fact tables (optional budgeting chain)
- [ ] Consolidated budget vs actual fact table
- [ ] Views pre-constraining ledger dimension to single ledger
- [ ] Multi-currency handling (local + corporate currency facts)

### Banking
- [ ] Account balance periodic snapshot (monthly grain)
- [ ] Account transaction fact table (daily/real-time grain)
- [ ] Dimension triage complete (5-20 dimensions, not 2)
- [ ] Product dimension (generic hierarchy)
- [ ] Account dimension (stable attributes only)
- [ ] Household dimension (economic unit grouping)
- [ ] Account status dimension
- [ ] Credit bureau mini-dimension (monthly updates)
- [ ] Demographics mini-dimension (annual updates)
- [ ] Behavior score mini-dimension (monthly/quarterly updates)
- [ ] Account-to-customer bridge table for joint accounts
- [ ] Supertype account snapshot fact table (cross-product metrics)
- [ ] Subtype fact tables by product type (checking, mortgage, credit card)
- [ ] Subtype account dimensions by product type (product-specific attributes)

### Insurance
- [ ] Policy transaction fact table (atomic transaction grain)
- [ ] Premium periodic snapshot (monthly coverage/item grain)
- [ ] Claim transaction fact table (atomic claim task grain)
- [ ] Claim accumulating snapshot (one row per claim lifecycle)
- [ ] Policy accumulating snapshot (optional—one row per policy lifecycle)
- [ ] Policyholder dimension with appropriate SCD strategies
- [ ] Coverage dimension (products)
- [ ] Covered item dimension (or mini-dimension if large)
- [ ] Claimant dimension (handle "dirty" duplicates)
- [ ] Third-party payee dimension
- [ ] Policy status dimension, claim status dimension
- [ ] Claim profile junk dimension (flags, indicators)
- [ ] Written premium vs earned premium facts in snapshot
- [ ] Supertype coverage/item dimensions (generic attributes)
- [ ] Subtype coverage/item dimensions (auto, home, life-specific)
- [ ] Consolidated policy/claim profit fact table
- [ ] Multiple insured drivers bridge table (auto insurance)

### Cross-Industry
- [ ] Conformed date/month dimensions
- [ ] Conformed customer/policyholder dimensions across schemas
- [ ] Conformed product/coverage dimensions
- [ ] Audit dimensions for data lineage (transaction and snapshot tables)
- [ ] All facts with identical definitions conform (same name, same logic)
- [ ] All dimensions with overlapping content conform (same keys, same attributes)

---

## References

- **Chapter 7:** Accounting (ragged hierarchies, budgeting, consolidated facts)
- **Chapter 10:** Financial Services (supertype/subtype, mini-dimensions, household)
- **Chapter 16:** Insurance (policy/claim transactions and snapshots)
- **Related Skills:**
  - `four-step-dimensional-modeling` - Foundation process
  - `dimension-table-design-patterns` - SCD techniques, mini-dimensions
  - `fact-table-design-patterns` - Transaction, periodic, accumulating snapshots
  - `conformed-dimension-management` - Enterprise integration

---

## Quick Decision Trees

### Should I Use Supertype/Subtype?

```
Does your organization offer heterogeneous products?
├─ YES: Do the products have significantly different facts?
│   ├─ YES: Use supertype fact + subtype facts by product type
│   └─ NO: Single fact table with supertype product dimension + subtype product dimensions
└─ NO: Standard single fact table and dimension approach
```

### How Should I Handle This Dimension Attribute?

```
Is the attribute rapidly changing? (monthly updates)
├─ YES: Is the dimension very large? (> 1M rows)
│   ├─ YES: Use mini-dimension (Type 4 SCD)
│   └─ NO: Use Type 2 SCD (if history needed) or Type 1 (if not)
└─ NO: Use Type 2 SCD (history) or Type 1 (current state only)
```

### Which Fact Table Type for This Financial Process?

```
What is the business process measurement pattern?
├─ Individual events (payments, journal entries, claims) → Transaction Fact Table
├─ Regular period status (month-end balances, premiums) → Periodic Snapshot
├─ Process with defined start/end (claim lifecycle, policy lifecycle) → Accumulating Snapshot
└─ Comparing metrics across processes (budget vs actual) → Consolidated Fact Table
```
