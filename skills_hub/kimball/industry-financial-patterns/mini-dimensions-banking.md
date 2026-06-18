# Mini-Dimensions in Banking and Financial Services

## Complete Guide to Type 4 Mini-Dimensions for Large Customer/Account Dimensions

Financial institutions face unique dimensional modeling challenges: **millions of customers/accounts** with **rapidly changing attributes** (monthly credit scores, quarterly demographics, weekly behavior scores). This combination creates a dimensional modeling crisis that mini-dimensions elegantly solve.

---

## The Problem: Monster Dimensions with Rapid Change

### Scenario: Large Retail Bank

**Customer Dimension:**
- **20 million customers** (large dimension)
- **Monthly credit bureau updates** (credit score, delinquency status, credit utilization)
- **Annual demographic updates** (income, age, home ownership status)
- **Quarterly behavior scores** (profitability score, retention risk, attrition propensity)

**SCD Type 2 Disaster:**
- Credit scores update monthly for 5M customers → 5M new dimension rows per month
- 12 months × 5M = **60M new rows per year** just for credit score changes
- 5 years → 300M customer dimension rows (from original 20M)
- Dimension explodes to 15× original size

**Symptoms:**
- Customer dimension table bloated (300M rows)
- Queries slow (scanning massive dimension)
- ETL slow (type 2 change detection on 20M rows)
- Fact table foreign keys non-selective (many surrogate keys per customer)
- BI tool performance degraded (dimension browsing sluggish)

---

## Solution: Type 4 Mini-Dimensions

**Principle:** Split rapidly changing, frequently analyzed attributes into separate mini-dimensions directly linked to fact table.

### Architecture

**Before (Type 2 Monster):**
```
Customer Dimension (300M rows):
- customer_key (surrogate)
- customer_id (natural key)
- name, address, city, state, zip
- credit_score ← Changes monthly
- delinquency_status ← Changes monthly
- income_range ← Changes annually
- age_band ← Changes annually
- profitability_score ← Changes quarterly
- retention_risk ← Changes quarterly
```

**After (Mini-Dimensions):**
```
Customer Dimension (22M rows—stable):
- customer_key
- customer_id
- name, address, city, state, zip
- (Stable attributes only)

Credit Bureau Mini-Dimension (400 rows):
- credit_bureau_key
- credit_score_band
- delinquency_status
- credit_utilization_band
- bankruptcy_flag

Demographics Mini-Dimension (800 rows):
- demographics_key
- income_range
- age_band
- homeowner_flag
- marital_status

Behavior Score Mini-Dimension (200 rows):
- behavior_score_key
- profitability_score_band
- retention_risk_band
- attrition_propensity_band

Account Snapshot Fact Table:
- month_key
- account_key
- customer_key ← Stable dimension
- credit_bureau_key ← Monthly updates
- demographics_key ← Annual updates
- behavior_score_key ← Quarterly updates
- balance_amount
- ...
```

**Result:**
- Customer dimension stabilized at 22M rows (20M + normal churn)
- Mini-dimensions tiny (400, 800, 200 rows—pre-populated combinations)
- Fact table has 4 dimension foreign keys instead of 1, but each update just changes FK value
- No dimension row insertions for most attribute changes

---

## Mini-Dimension Design Patterns

### Pattern 1: Banding Continuous Values

**Challenge:** Credit scores range from 300 to 850 (550 unique values). Creating 550-row mini-dimension defeats the purpose.

**Solution: Band into ranges**

| credit_bureau_key | credit_score_band | delinquency_status | credit_utilization_band |
|-------------------|-------------------|--------------------|-------------------------|
| 1                 | 300-579 (Poor)    | Current            | 0-30%                   |
| 2                 | 300-579 (Poor)    | Current            | 31-50%                  |
| 3                 | 300-579 (Poor)    | Current            | 51-100%                 |
| 4                 | 300-579 (Poor)    | 30 Days Late       | 0-30%                   |
| ...               | ...               | ...                | ...                     |
| 400               | 800-850 (Excellent) | Current          | 0-30%                   |

**Banding Strategy:**
- Credit score: 5 bands (Poor, Fair, Good, Very Good, Excellent)
- Delinquency: 5 statuses (Current, 30 Days, 60 Days, 90+ Days, Charge-off)
- Credit utilization: 4 bands (0-30%, 31-50%, 51-75%, 76-100%)
- 5 × 5 × 4 = 100 combinations (pre-populate all)

**Trade-off:**
- Lose precision (can't filter to "credit score = 720", only "700-749 band")
- Gain usability (users think in bands anyway: "show me good credit customers")
- Gain performance (100-row mini-dimension vs 550-row)

### Pattern 2: Pre-Populating All Combinations

**Approach:** Generate all possible attribute combinations upfront.

```sql
-- Pre-populate credit bureau mini-dimension
INSERT INTO credit_bureau_mini_dim
  (credit_bureau_key, credit_score_band, delinquency_status, credit_utilization_band)
SELECT
  ROW_NUMBER() OVER (ORDER BY cs.band, ds.status, cu.band) as credit_bureau_key,
  cs.band as credit_score_band,
  ds.status as delinquency_status,
  cu.band as credit_utilization_band
FROM
  (SELECT '300-579 (Poor)' as band UNION ALL
   SELECT '580-669 (Fair)' UNION ALL
   SELECT '670-739 (Good)' UNION ALL
   SELECT '740-799 (Very Good)' UNION ALL
   SELECT '800-850 (Excellent)') cs
CROSS JOIN
  (SELECT 'Current' as status UNION ALL
   SELECT '30 Days Late' UNION ALL
   SELECT '60 Days Late' UNION ALL
   SELECT '90+ Days Late' UNION ALL
   SELECT 'Charge-off') ds
CROSS JOIN
  (SELECT '0-30%' as band UNION ALL
   SELECT '31-50%' UNION ALL
   SELECT '51-75%' UNION ALL
   SELECT '76-100%') cu;
```

**Result:** 100 rows pre-loaded. ETL just looks up existing key.

**Benefits:**
- No mini-dimension inserts during fact load (just lookups)
- Consistent keys across months (score 720 → band "740-799" → key always same)
- Fact table updates just change FK (no dimension row creation)

**When Pre-Population Works:**
- Discrete attributes with < 20 values each
- Reasonable number of combinations (< 10,000)
- Bands are business-defined and stable

**When Pre-Population Fails:**
- Continuous attributes (can't band income into every possible value)
- Too many combinations (10 attributes × 10 values each = 10 billion combinations)

### Pattern 3: Three Storage Locations for Same Attribute

**Business Scenario:** Credit score is critical for risk analytics, customer segmentation, and data mining.

**Three Storage Locations:**

1. **Banded in Mini-Dimension** (for routine reporting)
   - credit_bureau_mini_dim.credit_score_band = "670-739 (Good)"
   - Used by 95% of users/reports
   - Enables fast filtering: "WHERE credit_score_band = 'Good'"

2. **Discrete in Fact Table** (for precise analysis)
   - account_snapshot_fact.credit_score_value = 712
   - Used by data scientists, advanced analysts
   - Enables precise calculations: "AVG(credit_score_value)", data mining models

3. **Current in Main Dimension** (for "current state" queries)
   - customer_dimension.current_credit_score = 712 (Type 1 overwrite)
   - Used for operational reports: "current customer profile"
   - Always shows most recent value

**Labeling Discipline:**
- `credit_score_band` (mini-dimension attribute)
- `credit_score_value` (fact table—numeric fact)
- `current_credit_score` (customer dimension—type 1 attribute)

**Benefits:**
- Routine reporting fast (mini-dimension bands)
- Advanced analytics possible (discrete fact values)
- Current state accessible (dimension attribute)

**ETL Flow:**
```sql
-- Monthly account snapshot load
INSERT INTO account_snapshot_fact
  (month_key, account_key, customer_key, credit_bureau_key, credit_score_value, ...)
SELECT
  202412 as month_key,
  a.account_key,
  c.customer_key,
  -- Lookup banded mini-dimension key
  (SELECT cb.credit_bureau_key
   FROM credit_bureau_mini_dim cb
   WHERE s.credit_score BETWEEN cb.score_min AND cb.score_max
     AND s.delinquency_status = cb.delinquency_status
     AND ...) as credit_bureau_key,
  -- Store discrete value as fact
  s.credit_score as credit_score_value,
  ...
FROM staging.account_snapshot s
JOIN account a ON s.account_id = a.account_id
JOIN customer c ON a.customer_id = c.customer_id;

-- Update current credit score in customer dimension (Type 1)
UPDATE customer c
SET current_credit_score = s.credit_score
FROM staging.latest_credit_scores s
WHERE c.customer_id = s.customer_id;
```

---

## Multiple Mini-Dimensions in Single Fact Table

**Typical Banking Snapshot:**

```sql
CREATE TABLE account_snapshot_fact (
  snapshot_month_key        INT NOT NULL,
  account_key               INT NOT NULL,
  customer_key              INT NOT NULL,
  product_key               INT NOT NULL,
  branch_key                INT NOT NULL,
  household_key             INT NOT NULL,

  -- Mini-Dimension Keys (Type 4)
  credit_bureau_key         INT NOT NULL,  -- Monthly updates
  demographics_key          INT NOT NULL,  -- Annual updates
  behavior_score_key        INT NOT NULL,  -- Quarterly updates
  account_status_key        INT NOT NULL,  -- As-needed updates

  -- Facts
  balance_amount            DECIMAL(18,2),
  transaction_count         INT,
  fee_amount                DECIMAL(18,2),

  -- Discrete values for precise analysis
  credit_score_value        INT,
  income_amount             DECIMAL(18,2),
  profitability_score_value INT,

  PRIMARY KEY (snapshot_month_key, account_key)
);
```

**Update Patterns:**

| Month   | Account | Customer Key | Credit Bureau Key | Demographics Key | Behavior Score Key |
|---------|---------|--------------|-------------------|------------------|--------------------|
| 2024-01 | 12345   | 99999        | 42                | 15               | 88                 |
| 2024-02 | 12345   | 99999        | 42                | 15               | 88                 |
| 2024-03 | 12345   | 99999        | 48                | 15               | 88                 |
| 2024-04 | 12345   | 99999        | 48                | 18               | 92                 |

**What Changed:**
- March: Credit score band changed (42→48) due to monthly update
- April: Income band changed (15→18) due to annual update + behavior score (88→92) quarterly

**Key Insight:** Only foreign key values change; no dimension row creation.

---

## Mini-Dimension Attribute Selection

### Good Candidates for Mini-Dimensions

✅ **Rapidly Changing Attributes**
- Updated monthly, weekly, or daily
- Type 2 SCD would cause dimension explosion

✅ **Frequently Analyzed Attributes**
- Used in 50%+ of queries for filtering/grouping
- Performance-critical for BI applications

✅ **Browsable Attributes**
- Users want to filter/group by these attributes
- Values are discrete or can be banded meaningfully

✅ **Correlated Attribute Groups**
- Attributes that change together (credit score + delinquency status)
- Natural grouping reduces mini-dimension count

**Banking Examples:**
- Credit bureau data (score, delinquency, utilization)
- Demographics (income, age, education, homeowner status)
- Behavior scores (profitability, retention, attrition, lifetime value)
- Product usage (transaction frequency, channel preference, service adoption)

### Poor Candidates for Mini-Dimensions

❌ **Slowly Changing Attributes**
- Update annually or less frequently
- Type 2 SCD on main dimension acceptable

❌ **Rarely Analyzed Attributes**
- Used in < 5% of queries
- Leave in main dimension, handle as Type 1

❌ **High-Cardinality Attributes**
- Too many unique values to pre-populate
- Cannot band meaningfully (customer name, SSN)

❌ **Uncorrelated Singleton Attributes**
- Single attribute in mini-dimension defeats purpose
- Would create too many mini-dimensions (one per attribute = cluttered fact table)

**Anti-Pattern:**
```
account_snapshot_fact:
- account_key
- customer_key
- income_mini_dim_key      ← Just income (bad!)
- age_mini_dim_key         ← Just age (bad!)
- education_mini_dim_key   ← Just education (bad!)
- credit_score_mini_dim_key ← Just credit score (bad!)
```

**Better:** Group into demographics mini-dimension with all four attributes.

---

## Mini-Dimension ETL Patterns

### ETL Pattern 1: Lookup Pre-Populated Keys

**Scenario:** Credit bureau mini-dimension pre-populated with all combinations.

```sql
-- Stage incoming credit scores
CREATE TABLE staging.credit_scores (
  customer_id       VARCHAR(20),
  snapshot_date     DATE,
  credit_score      INT,
  delinquency_days  INT,
  credit_utilization DECIMAL(5,2)
);

-- ETL: Map to mini-dimension keys
INSERT INTO account_snapshot_fact
  (snapshot_month_key, account_key, customer_key, credit_bureau_key, ...)
SELECT
  dm.month_key,
  a.account_key,
  c.customer_key,
  -- Lookup credit bureau mini-dimension key
  cb.credit_bureau_key,
  ...
FROM staging.credit_scores s
JOIN customer c ON s.customer_id = c.customer_id
JOIN account a ON c.customer_key = a.customer_key
JOIN date_dimension dm ON s.snapshot_date = dm.calendar_date
JOIN credit_bureau_mini_dim cb
  ON (CASE
        WHEN s.credit_score BETWEEN 300 AND 579 THEN '300-579 (Poor)'
        WHEN s.credit_score BETWEEN 580 AND 669 THEN '580-669 (Fair)'
        WHEN s.credit_score BETWEEN 670 AND 739 THEN '670-739 (Good)'
        WHEN s.credit_score BETWEEN 740 AND 799 THEN '740-799 (Very Good)'
        WHEN s.credit_score BETWEEN 800 AND 850 THEN '800-850 (Excellent)'
      END) = cb.credit_score_band
  AND (CASE
        WHEN s.delinquency_days = 0 THEN 'Current'
        WHEN s.delinquency_days BETWEEN 1 AND 29 THEN '30 Days Late'
        WHEN s.delinquency_days BETWEEN 30 AND 59 THEN '60 Days Late'
        WHEN s.delinquency_days BETWEEN 60 AND 89 THEN '90+ Days Late'
        ELSE 'Charge-off'
      END) = cb.delinquency_status
  AND (CASE
        WHEN s.credit_utilization BETWEEN 0 AND 30 THEN '0-30%'
        WHEN s.credit_utilization BETWEEN 31 AND 50 THEN '31-50%'
        WHEN s.credit_utilization BETWEEN 51 AND 75 THEN '51-75%'
        ELSE '76-100%'
      END) = cb.credit_utilization_band;
```

**Optimization:** Create mapping function to simplify:
```sql
CREATE FUNCTION get_credit_bureau_key(
  p_credit_score INT,
  p_delinquency_days INT,
  p_credit_utilization DECIMAL
) RETURNS INT
AS
BEGIN
  RETURN (
    SELECT credit_bureau_key
    FROM credit_bureau_mini_dim
    WHERE credit_score_band = get_credit_band(p_credit_score)
      AND delinquency_status = get_delinquency_status(p_delinquency_days)
      AND credit_utilization_band = get_utilization_band(p_credit_utilization)
  );
END;
```

### ETL Pattern 2: Incremental Mini-Dimension Updates

**Scenario:** Cannot pre-populate all combinations (too many).

```sql
-- Check if combination exists
SELECT demographics_key
FROM demographics_mini_dim
WHERE income_range = '50K-75K'
  AND age_band = '35-44'
  AND homeowner_flag = 'Y'
  AND marital_status = 'Married';

-- If not exists, insert new row
INSERT INTO demographics_mini_dim
  (demographics_key, income_range, age_band, homeowner_flag, marital_status)
SELECT
  COALESCE(MAX(demographics_key), 0) + 1,
  '50K-75K',
  '35-44',
  'Y',
  'Married'
FROM demographics_mini_dim
WHERE NOT EXISTS (
  SELECT 1 FROM demographics_mini_dim
  WHERE income_range = '50K-75K'
    AND age_band = '35-44'
    AND homeowner_flag = 'Y'
    AND marital_status = 'Married'
);
```

**Alternative: Merge/Upsert Pattern**
```sql
MERGE INTO demographics_mini_dim d
USING (
  SELECT DISTINCT
    income_range, age_band, homeowner_flag, marital_status
  FROM staging.customer_demographics
) s
ON (d.income_range = s.income_range
    AND d.age_band = s.age_band
    AND d.homeowner_flag = s.homeowner_flag
    AND d.marital_status = s.marital_status)
WHEN NOT MATCHED THEN
  INSERT (demographics_key, income_range, age_band, homeowner_flag, marital_status)
  VALUES (demographics_key_seq.NEXTVAL, s.income_range, s.age_band,
          s.homeowner_flag, s.marital_status);
```

---

## Query Patterns with Mini-Dimensions

### Query Pattern 1: Filtering on Mini-Dimension Attributes

**Business Question:** "Show me average balance for customers with good credit in high-income bracket."

```sql
SELECT
  d.income_range,
  cb.credit_score_band,
  AVG(f.balance_amount) as avg_balance,
  COUNT(DISTINCT f.account_key) as account_count
FROM account_snapshot_fact f
JOIN credit_bureau_mini_dim cb ON f.credit_bureau_key = cb.credit_bureau_key
JOIN demographics_mini_dim d ON f.demographics_key = d.demographics_key
WHERE f.snapshot_month_key = 202412
  AND cb.credit_score_band IN ('670-739 (Good)', '740-799 (Very Good)', '800-850 (Excellent)')
  AND d.income_range IN ('$75K-$100K', '$100K-$150K', '$150K+')
GROUP BY d.income_range, cb.credit_score_band;
```

**Performance:** Mini-dimensions small (hundreds of rows), joins fast, highly selective filtering.

### Query Pattern 2: Trend Analysis with Changing Attributes

**Business Question:** "How did average balance change for customers as their credit scores improved?"

```sql
SELECT
  f.snapshot_month_key,
  cb.credit_score_band,
  AVG(f.balance_amount) as avg_balance
FROM account_snapshot_fact f
JOIN credit_bureau_mini_dim cb ON f.credit_bureau_key = cb.credit_bureau_key
WHERE f.account_key = 12345  -- Specific account
  AND f.snapshot_month_key BETWEEN 202401 AND 202412
GROUP BY f.snapshot_month_key, cb.credit_score_band
ORDER BY f.snapshot_month_key;
```

**Result:**
| Month   | Credit Score Band     | Avg Balance |
|---------|-----------------------|-------------|
| 2024-01 | 580-669 (Fair)        | $1,200      |
| 2024-02 | 580-669 (Fair)        | $1,350      |
| 2024-03 | 670-739 (Good)        | $1,500      |
| 2024-04 | 670-739 (Good)        | $1,650      |

**Insight:** Credit score improvement (Fair→Good) correlated with balance growth.

### Query Pattern 3: Cohort Analysis

**Business Question:** "Compare balance growth for customers who started with poor credit vs excellent credit."

```sql
WITH starting_credit AS (
  SELECT
    f.account_key,
    cb.credit_score_band as starting_band
  FROM account_snapshot_fact f
  JOIN credit_bureau_mini_dim cb ON f.credit_bureau_key = cb.credit_bureau_key
  WHERE f.snapshot_month_key = 202401  -- Starting month
)
SELECT
  sc.starting_band,
  f.snapshot_month_key,
  AVG(f.balance_amount) as avg_balance
FROM account_snapshot_fact f
JOIN starting_credit sc ON f.account_key = sc.account_key
WHERE f.snapshot_month_key BETWEEN 202401 AND 202412
GROUP BY sc.starting_band, f.snapshot_month_key
ORDER BY sc.starting_band, f.snapshot_month_key;
```

**Mini-Dimension Benefit:** Historical credit bands preserved in fact table (each month's row has that month's credit_bureau_key).

---

## Advanced Patterns

### Pattern: Mini-Dimension in Bridge Table

**Scenario:** Account-to-customer bridge table (joint accounts) with rapidly changing customer demographics.

**Problem:** Customer demographics change monthly. Bridge table would bloat with type 2 SCD.

**Solution:** Add demographics mini-dimension key to bridge table.

```sql
CREATE TABLE account_customer_bridge (
  account_key           INT,
  customer_key          INT,
  demographics_key      INT,  -- Mini-dimension in bridge!
  weighting_factor      DECIMAL(5,2),
  effective_date        DATE,
  expiration_date       DATE,
  PRIMARY KEY (account_key, customer_key, demographics_key, effective_date)
);
```

**Query with Mini-Dimension Bridge:**
```sql
-- Impact report: Accounts exposed to high-income customers
SELECT
  d.income_range,
  COUNT(DISTINCT f.account_key) as account_count,
  SUM(f.balance_amount) as total_exposure
FROM account_snapshot_fact f
JOIN account_customer_bridge b
  ON f.account_key = b.account_key
  AND f.snapshot_month_key BETWEEN b.effective_date AND b.expiration_date
JOIN demographics_mini_dim d ON b.demographics_key = d.demographics_key
WHERE d.income_range IN ('$100K-$150K', '$150K+')
GROUP BY d.income_range;
```

**Benefit:** Bridge table row count stabilized (only new rows when demographics band changes, not every monthly update).

---

## Business Value

### Quantified Benefits: Real Bank Example

**Before Mini-Dimensions:**
- Customer dimension: 300M rows
- Dimension query response: 45 seconds
- Fact table load: 8 hours (type 2 surrogate key lookup on 300M rows)
- Storage: 450 GB (customer dimension table)

**After Mini-Dimensions:**
- Customer dimension: 22M rows (stable)
- Credit bureau mini-dim: 400 rows
- Demographics mini-dim: 800 rows
- Behavior mini-dim: 200 rows
- Dimension query response: 2 seconds
- Fact table load: 90 minutes (lookup on 22M + tiny mini-dims)
- Storage: 35 GB (customer dimension) + 1 MB (mini-dims)

**ROI:**
- 95% reduction in customer dimension size
- 22× faster dimension queries
- 5× faster fact table loads
- Enabled monthly credit score tracking (previously impossible)

---

## Common Mistakes

### Mistake 1: One Attribute Per Mini-Dimension

**Anti-Pattern:** 10 mini-dimensions, each with single attribute

**Symptom:** Fact table has 15 foreign keys, query complexity exploded

**Fix:** Group correlated attributes (credit score + delinquency + utilization = single mini-dim)

### Mistake 2: Not Pre-Populating

**Anti-Pattern:** Insert new mini-dimension rows during fact load

**Symptom:** Inconsistent keys across months (score 720 → different key each month)

**Fix:** Pre-populate all combinations, ensure deterministic key assignment

### Mistake 3: Too Fine-Grained Banding

**Anti-Pattern:** Credit score bands every 10 points (300-309, 310-319, ...)

**Symptom:** Mini-dimension has 56 credit bands × other attributes = thousands of rows

**Fix:** Use business-meaningful bands aligned with decision thresholds

### Mistake 4: Forgetting Discrete Values in Fact

**Anti-Pattern:** Only banded mini-dimension, no discrete fact values

**Symptom:** Data scientists cannot build models (need precise values)

**Fix:** Store discrete values as facts for advanced analytics

### Mistake 5: Not Validating Referential Integrity

**Anti-Pattern:** Fact table has mini-dimension key with no matching mini-dim row

**Symptom:** Queries lose rows, business metrics wrong

**Fix:** Ensure all mini-dimension keys populated before fact load; use NOT NULL constraints
