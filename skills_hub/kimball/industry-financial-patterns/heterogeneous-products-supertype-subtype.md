# Heterogeneous Products: Supertype and Subtype Schemas

## Complete Guide to Modeling Diverse Product Portfolios

Organizations offering **heterogeneous products**—products with vastly different attributes and metrics—to the same customer base face a dimensional modeling dilemma. This pattern is ubiquitous in financial services (banking, insurance) but applies to any multi-line-of-business organization.

---

## The Problem

**Business Scenario:** Retail bank offers:
- **Checking accounts:** Minimum balance, overdraft limit, check count, online banking usage
- **Savings accounts:** Interest rate, compounding frequency, withdrawal limits
- **Mortgages:** Interest rate, term, points, escrow balance, loan-to-value ratio
- **Credit cards:** APR, credit limit, rewards program, cash advance limit
- **Certificates of deposit:** Maturity date, early withdrawal penalty, compounding frequency

**Challenges:**

1. **Attribute Incompatibility**
   - Checking has 50 specific attributes; mortgages have 40 different specific attributes
   - Almost no overlap beyond generic "account number" and "balance"
   - Including all attributes in single dimension → 300-column table, 95% nulls per row (Swiss cheese)

2. **Fact Incompatibility**
   - Checking: transaction count, overdraft count, service charge amount
   - Mortgage: principal paid, interest paid, escrow payment, prepayment amount
   - Credit card: purchase count, cash advance amount, interest charge, rewards earned
   - Including all facts in single table → hundreds of facts, mostly null

3. **Dual Business Requirements**
   - **Enterprise view:** "Show me total customer relationship value across ALL product types"
   - **Line-of-business view:** "Deep-dive into checking account overdraft patterns"

**Anti-Pattern: Forcing Everything into Single Schema**
```
account_dimension (BAD):
- account_key
- account_number
- minimum_balance          ← Only for checking
- overdraft_limit          ← Only for checking
- interest_rate            ← For savings, mortgages, credit cards (but different meanings!)
- maturity_date            ← Only for CDs and mortgages
- rewards_program          ← Only for credit cards
- ... 200 more attributes with 95% nulls
```

**Why This Fails:**
- Meaningless attribute values for most rows
- Users confused about which attributes apply to which products
- Query performance degraded (scanning null-filled columns)
- ETL complexity (conditional logic for which attributes to populate)
- Impossible to enforce data quality constraints

---

## Solution: Supertype and Subtype Schemas

**Principle:** Separate **common cross-product analysis** (supertype) from **line-of-business detail** (subtype).

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  SUPERTYPE SCHEMA (Enterprise View)                         │
│  ┌────────────────────┐        ┌────────────────────────┐  │
│  │ Account Dimension  │───────>│ Account Snapshot Fact  │  │
│  │ (Generic attrs)    │        │ (Common facts)         │  │
│  │ - account_key      │        │ - balance              │  │
│  │ - account_number   │        │ - transaction_count    │  │
│  │ - account_type     │        │ - fees_charged         │  │
│  │ - open_date        │        │ - interest_paid        │  │
│  └────────────────────┘        └────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  SUBTYPE SCHEMA: Checking (Line-of-Business View)           │
│  ┌────────────────────┐        ┌────────────────────────┐  │
│  │ Checking Account   │───────>│ Checking Snapshot Fact │  │
│  │ Dimension          │        │                        │  │
│  │ - account_key (FK) │        │ - balance (supertype)  │  │
│  │ - minimum_balance  │        │ - fees_charged (super) │  │
│  │ - overdraft_limit  │        │ - overdraft_count (sub)│  │
│  │ - online_banking   │        │ - check_count (sub)    │  │
│  └────────────────────┘        └────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  SUBTYPE SCHEMA: Mortgage (Line-of-Business View)           │
│  ┌────────────────────┐        ┌────────────────────────┐  │
│  │ Mortgage Account   │───────>│ Mortgage Snapshot Fact │  │
│  │ Dimension          │        │                        │  │
│  │ - account_key (FK) │        │ - balance (supertype)  │  │
│  │ - interest_rate    │        │ - fees_charged (super) │  │
│  │ - term_months      │        │ - principal_paid (sub) │  │
│  │ - escrow_balance   │        │ - escrow_payment (sub) │  │
│  └────────────────────┘        └────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Detailed Design

### 1. Supertype Schema (Enterprise View)

**Purpose:** Integrated view of ALL accounts across all product types for customer relationship analysis.

#### Supertype Account Dimension

**Grain:** One row per account (all account types)

**Attributes (Common to ALL product types):**
- account_key (surrogate key)
- account_number (natural key)
- account_type (checking, savings, mortgage, credit card, CD)
- account_category (deposit, loan, investment)
- account_status (active, inactive, closed)
- open_date
- close_date
- primary_customer_name (current—type 1 or could be outrigger)

**What's NOT Included:**
- Product-specific attributes (minimum balance, interest rate, etc.)
- Attributes that mean different things by product type

**Row Count:** All accounts (10 million if bank has 10M accounts)

#### Supertype Product Dimension

**Grain:** One row per product offering (all product types)

**Attributes:**
- product_key
- product_name ("Free Checking", "Premier Savings", "30-Year Fixed Mortgage")
- product_type (checking, savings, mortgage, credit card, CD)
- product_category (deposit accounts, loan accounts, credit accounts)

**What's NOT Included:**
- Product-specific attributes (overdraft policy, prepayment terms, rewards structure)

#### Supertype Account Snapshot Fact Table

**Grain:** One row per account per month (all account types)

**Facts (Common to ALL products):**
- balance_amount (primary balance—universal across all account types)
- transaction_count (activity measure)
- interest_paid (if applicable, else 0)
- fees_charged (if applicable, else 0)

**What's NOT Included:**
- Product-specific facts (overdraft_count, principal_paid, rewards_earned)

**Row Count:** 10M accounts × 60 months = 600M rows

**Use Cases:**
- "What's total customer relationship value by household?"
- "Show me product mix by demographic segment"
- "Trend total deposit balances over time"
- "Customer retention rate across all products"

---

### 2. Subtype Schemas (Line-of-Business Views)

**Purpose:** Deep-dive analysis with product-specific attributes and facts.

#### Subtype Checking Account Dimension

**Grain:** One row per **checking account only** (shrunken subset)

**Attributes:**
- account_key (same surrogate key as supertype!)
- account_number, account_type, open_date, close_date (duplicated from supertype)
- **minimum_balance_requirement** (checking-specific)
- **overdraft_limit** (checking-specific)
- **overdraft_protection_flag** (checking-specific)
- **free_checks_flag** (checking-specific)
- **online_banking_enabled** (checking-specific)
- **mobile_deposit_enabled** (checking-specific)
- **debit_card_type** (checking-specific)

**Row Count:** ~3M rows (if 3M checking accounts out of 10M total)

**Critical: Key Conformance**
- Account with account_key=12345 in supertype has SAME account_key=12345 in checking subtype
- Shrunken conformed dimension (subset of rows, consistent keys)

#### Subtype Checking Snapshot Fact Table

**Grain:** One row per checking account per month

**Facts:**
- balance_amount (duplicated from supertype for convenience)
- transaction_count (duplicated)
- interest_paid (duplicated)
- fees_charged (duplicated)
- **overdraft_count** (checking-specific)
- **check_count** (checking-specific)
- **debit_transaction_count** (checking-specific)
- **atm_withdrawal_count** (checking-specific)
- **online_banking_login_count** (checking-specific)
- **service_charge_waived_flag** (checking-specific)

**Why Duplicate Supertype Facts?**
- Avoid joins between supertype and subtype fact tables
- Users working in checking analysis don't want to query two fact tables
- Storage cost minimal compared to usability benefit

**Row Count:** 3M accounts × 60 months = 180M rows

**Use Cases:**
- "Overdraft frequency by minimum balance tier"
- "Online banking adoption rate by account vintage"
- "Average check usage by customer age"

#### Additional Subtype Schemas

**Mortgage Account Dimension:**
- interest_rate, term_months, loan_amount, property_value, loan_to_value_ratio
- escrow_required_flag, prepayment_penalty_flag, arm_vs_fixed
- origination_date, first_payment_date, maturity_date

**Mortgage Snapshot Fact:**
- principal_balance, interest_paid, principal_paid, escrow_balance
- prepayment_amount, late_payment_count, refinance_flag

**Credit Card Account Dimension:**
- credit_limit, apr_rate, cash_advance_apr, balance_transfer_apr
- rewards_program, annual_fee_amount, foreign_transaction_fee_pct

**Credit Card Snapshot Fact:**
- purchase_amount, cash_advance_amount, interest_charge_amount
- rewards_points_earned, payment_amount, minimum_payment_due
- overlimit_flag, late_payment_flag

**CD Account Dimension:**
- interest_rate, term_months, maturity_date, compounding_frequency
- early_withdrawal_penalty_pct, renewal_option

**CD Snapshot Fact:**
- principal_balance, interest_earned, early_withdrawal_penalty_paid

---

## Design Principles

### Principle 1: Same Keys Across Supertype and Subtypes

**Account 99999 (30-year fixed mortgage) appears in:**
- Supertype account dimension (account_key = 99999)
- Mortgage account dimension (account_key = 99999—SAME KEY)
- NOT in checking account dimension (not a checking account)

**Benefits:**
- Can join supertype and subtype if needed (rare, but possible)
- Consistent identity across schemas
- Simplifies ETL (assign key once, reuse everywhere)

### Principle 2: Subtypes are Shrunken Conformed Dimensions

**Shrunken = Subset of Rows**
- Supertype account dimension: 10M rows (all accounts)
- Checking account dimension: 3M rows (only checking accounts)
- Mortgage account dimension: 1M rows (only mortgage accounts)

**Conformed = Consistent Structure and Keys**
- Same account_key values
- Overlapping attributes have identical definitions

### Principle 3: Duplicate Supertype Facts in Subtype Fact Tables

**Why?**
- Users analyzing checking don't want to join to supertype fact table
- Query performance (avoid large fact-to-fact join)
- Simplicity (one fact table per business question)

**Objection: "But that's data duplication!"**
- Response: Fact table storage is cheap, user productivity expensive
- Subtype fact table partitioned by product type anyway (no cross-product mixing)
- Incremental storage minimal (supertype facts are small % of total)

### Principle 4: Users See One Schema at a Time

**User Perspective:**

**Enterprise Analyst (Supertype):**
- Sees: account, product, customer, household, supertype_account_snapshot
- Asks: "Total balances by product category by region"

**Checking Product Manager (Subtype):**
- Sees: checking_account, customer, household, checking_account_snapshot
- Asks: "Overdraft rate by online banking usage"
- Does NOT see mortgage, CD, or credit card schemas

**DBA Perspective:**
- Sees all schemas
- Creates views/security to expose correct schema to each user group

---

## Variation: Common Facts, Heterogeneous Attributes

**Scenario:** Facts are consistent across product types, but attributes vary wildly.

**Example:** Bank solicitation process (same metrics regardless of product)

**Fact Table:** `new_account_solicitation`
- Grain: One row per solicitation event
- Facts: solicitation_count (always 1), response_flag, account_opened_flag, days_to_response
- These facts do NOT vary by product type

**Dimension Strategy:**
- **Supertype product dimension:** Generic attributes for cross-product analysis
- **Subtype product dimensions:** Product-specific attributes (checking_product, mortgage_product)
- **Application Decision:** Use supertype product for cross-product queries; use subtype product for product-specific attribute filtering

**Fact Table Structure:**
```sql
new_account_solicitation_fact:
- solicitation_date_key
- customer_key
- product_key  ← Can join to supertype OR subtype product dimension
- solicitation_count
- response_flag
```

**Query Examples:**

*Cross-Product Analysis (Supertype):*
```sql
SELECT p.product_category, COUNT(*) as solicitation_count
FROM new_account_solicitation_fact f
JOIN product_supertype p ON f.product_key = p.product_key
GROUP BY p.product_category;
```

*Checking-Specific Analysis (Subtype):*
```sql
SELECT cp.minimum_balance_tier, COUNT(*) as solicitation_count
FROM new_account_solicitation_fact f
JOIN checking_product cp ON f.product_key = cp.product_key
WHERE cp.product_type = 'Checking'
GROUP BY cp.minimum_balance_tier;
```

**Key Point:** One fact table, multiple product dimension options.

---

## When to Use Supertype/Subtype

### Use Supertype/Subtype When:

✅ **Heterogeneous product attributes**
- Product types have > 50% different attributes
- Attributes have different meanings by product (interest_rate means different things for savings vs mortgage)

✅ **Heterogeneous facts**
- Different product types measure different things
- Combining facts creates > 50% null values per row

✅ **Dual business requirements**
- Enterprise view needed (cross-product customer relationship)
- Line-of-business view needed (deep product-specific analysis)

✅ **Multiple lines of business**
- Products from different divisions (insurance: auto, home, life)
- Each LOB has separate management, reporting, and analytics

### Don't Use Supertype/Subtype When:

❌ **Facts are common across products**
- All products measure same things
- Just attributes vary → use single fact table + supertype/subtype dimensions

❌ **Products are similar**
- Variations are minor (small checking vs large checking)
- Handle with product attributes, not separate schemas

❌ **No enterprise-wide analysis requirement**
- Each product line analyzes in isolation
- No cross-product customer relationship analysis

❌ **Small number of products**
- 2-3 product types with < 20 unique attributes each
- Can handle with nullable attributes in single dimension

---

## ETL Considerations

### Loading Supertype Schemas

```sql
-- Load supertype account dimension (all accounts)
INSERT INTO account_dimension_supertype
  (account_key, account_number, account_type, open_date, ...)
SELECT
  account_key,
  account_number,
  account_type,
  open_date,
  ...
FROM staging.all_accounts;

-- Load supertype fact table (all accounts)
INSERT INTO account_snapshot_fact_supertype
  (month_key, account_key, product_key, balance_amount, ...)
SELECT
  month_key,
  account_key,
  product_key,
  balance_amount,
  ...
FROM staging.account_monthly_snapshot;
```

### Loading Subtype Schemas

```sql
-- Load checking account dimension (checking accounts only)
INSERT INTO checking_account_dimension
  (account_key, account_number, account_type, minimum_balance, overdraft_limit, ...)
SELECT
  a.account_key,
  a.account_number,
  a.account_type,
  c.minimum_balance,
  c.overdraft_limit,
  ...
FROM staging.all_accounts a
JOIN staging.checking_account_details c ON a.account_id = c.account_id
WHERE a.account_type = 'Checking';

-- Load checking fact table (checking accounts only)
INSERT INTO checking_account_snapshot_fact
  (month_key, account_key, balance_amount, overdraft_count, check_count, ...)
SELECT
  s.month_key,
  s.account_key,
  s.balance_amount,  ← Supertype fact
  c.overdraft_count, ← Subtype fact
  c.check_count,     ← Subtype fact
  ...
FROM staging.account_monthly_snapshot s
JOIN staging.checking_monthly_details c ON s.account_id = c.account_id
WHERE s.account_type = 'Checking';
```

**Key ETL Principles:**
- Same account → same account_key in supertype and subtype
- Supertype loads ALL accounts; subtype loads FILTERED accounts
- Subtype fact tables include supertype facts (duplicated)
- Single source data extracted once, written to multiple targets

---

## Real-World Examples

### Example 1: Insurance Company

**Products:** Auto, homeowners, umbrella, life, health insurance

**Supertype Schema:**
- **Policy dimension:** Policy number, policy status, effective date, expiration date
- **Coverage dimension:** Coverage type, coverage category
- **Policy snapshot fact:** Written premium, earned premium, policy count

**Subtype Schemas:**
- **Auto policy dimension:** Vehicle make/model/year, driver count, mileage, garage location
- **Auto coverage dimension:** Liability limits, deductible, collision vs comprehensive
- **Auto snapshot fact:** Written premium, earned premium, vehicle count, driver count

- **Homeowners policy dimension:** Property value, construction type, year built, security system
- **Homeowners coverage dimension:** Replacement cost vs actual cash value, deductible
- **Homeowners snapshot fact:** Written premium, earned premium, property value

**Business Value:**
- Enterprise view: "Customer lifetime value across all policies"
- Auto LOB view: "Premium by vehicle type and driver age"

### Example 2: Technology Company

**Products:** Hardware, software licenses, professional services, SaaS subscriptions

**Supertype Schema:**
- **Product dimension:** Product name, product type, product family
- **Sales fact:** Quantity sold, revenue amount, discount amount

**Subtype Schemas:**
- **Hardware product dimension:** Warranty period, weight, power requirements, SKU
- **Software product dimension:** License type, seat count, upgrade eligibility, version
- **Services product dimension:** Hourly vs fixed-price, skill level required, certification
- **SaaS product dimension:** Subscription term, user tier, API call limits

**Business Value:**
- Enterprise view: "Total revenue by customer segment"
- Software LOB view: "License renewals by version and seat count"

### Example 3: Healthcare System

**Products:** Inpatient, outpatient, emergency, surgery, imaging services

**Supertype Schema:**
- **Encounter dimension:** Encounter type, admission date, discharge date
- **Encounter fact:** Charge amount, payment amount, patient count

**Subtype Schemas:**
- **Inpatient encounter dimension:** Admission source, discharge disposition, length of stay, ICU days
- **Surgery encounter dimension:** Procedure type, anesthesia type, surgeon, OR minutes
- **Imaging encounter dimension:** Modality (CT, MRI, X-ray), body part, contrast used

**Business Value:**
- Enterprise view: "Total patient encounters by payer"
- Surgery LOB view: "OR utilization by procedure type and surgeon"

---

## Common Mistakes

### Mistake 1: Forcing All Products into Single Schema

**Anti-Pattern:** 500-column dimension with 90% nulls per row

**Symptom:** Queries slow, ETL complex, users confused about which attributes apply

**Fix:** Recognize heterogeneity; implement supertype/subtype

### Mistake 2: Different Keys for Same Entity

**Anti-Pattern:** Account_key=123 in supertype, account_key=456 in checking subtype

**Symptom:** Cannot correlate same account across schemas

**Fix:** Use same surrogate key in supertype and subtype dimensions

### Mistake 3: Not Duplicating Supertype Facts in Subtype

**Anti-Pattern:** Subtype fact table only has product-specific facts

**Symptom:** Users must join supertype and subtype fact tables (performance nightmare)

**Fix:** Duplicate common facts in subtype fact tables

### Mistake 4: Subtypes Not Shrunken

**Anti-Pattern:** Checking dimension has 10M rows (same as supertype)

**Symptom:** Dimension includes non-checking accounts with nulls for checking attributes

**Fix:** Filter subtype dimension to ONLY applicable product types

### Mistake 5: Creating Hundreds of Subtype Schemas

**Anti-Pattern:** Separate schema for every minor product variation

**Symptom:** Explosion of tables, maintenance burden, user confusion

**Fix:** Group similar products (all checking variations → single checking subtype)

---

## Performance Considerations

### Partitioning

**Subtype fact tables naturally partition:**
```sql
-- Physically separate checking, mortgage, credit card fact partitions
CREATE TABLE account_snapshot_fact_supertype
PARTITION BY LIST (account_type) (
  PARTITION checking VALUES ('Checking'),
  PARTITION savings VALUES ('Savings'),
  PARTITION mortgage VALUES ('Mortgage'),
  PARTITION credit_card VALUES ('Credit Card')
);
```

**Benefits:**
- Subtype queries only scan relevant partition
- Partition pruning automatic (WHERE account_type = 'Checking')

### Indexing

**Supertype fact table:**
- Index on month_key, account_key, product_key (standard fact table indexing)

**Subtype fact tables:**
- Same indexes as supertype
- Additional indexes on subtype-specific dimensions if needed

### Aggregate Tables

**Supertype aggregates:**
- Monthly rollup to product category (cross-product executives)

**Subtype aggregates:**
- Checking aggregate by minimum balance tier
- Mortgage aggregate by interest rate band

**Each LOB can optimize aggregates independently**

---

## Migration Strategy

**Phase 1: Supertype Only**
- Build enterprise view first
- Deliver cross-product customer analytics
- Validate dimensional model and ETL

**Phase 2: Add One Subtype**
- Choose highest-priority LOB (e.g., checking)
- Build subtype dimension and fact table
- Validate with LOB business users

**Phase 3: Expand to Additional Subtypes**
- Add mortgage, credit card, CD schemas iteratively
- Reuse ETL patterns from first subtype
- Each subtype is independent (low-risk additions)

**This incremental approach:**
- Delivers value quickly (supertype for enterprise view)
- Proves architecture before full build-out
- Allows learning from first subtype before scaling
