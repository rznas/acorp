# Slowly Changing Dimension Complete Reference Guide

This reference provides comprehensive guidance on all SCD techniques (Types 0-7) with specific applications to supply chain dimensions.

## Overview

Dimension table attributes are relatively static, but they **do change over time**. Examples in supply chain contexts:

- **Vendor relocates** to new address
- **Product category** is restructured by merchandising
- **Customer** moves to different sales territory
- **Warehouse** adds refrigerated storage capacity
- **Vendor** achieves preferred vendor status

The business's data governance representatives must determine the appropriate change-handling strategy for each dimension attribute. **Never assume** the business doesn't care about tracking changes just because they didn't mention it.

---

## SCD Type Summary

| Type | Name | Action | Preserves History | Shows Current | Complexity |
|------|------|--------|-------------------|---------------|------------|
| 0 | Retain Original | Never change | ✓ Original only | ✗ | Very Low |
| 1 | Overwrite | Update in place | ✗ | ✓ | Low |
| 2 | Add New Row | Insert new version | ✓ | ✓ (with filter) | Medium |
| 3 | Add New Attribute | Add "prior" column | Limited (1-2 changes) | ✓ | Low |
| 4 | Mini-Dimension | Separate dimension for volatile attrs | ✓ (via fact table) | ✓ | High |
| 5 | Type 4 + Type 1 Outrigger | Mini-dim + current pointer | ✓ | ✓ (dual access) | High |
| 6 | Type 1 + Type 2 + Type 3 | Historic + current columns in type 2 | ✓ | ✓ (dual columns) | High |
| 7 | Dual Type 1 and Type 2 | Two dimensions, dual FK | ✓ | ✓ (dual dimensions) | High |

---

## SCD Type 0: Retain Original

### Definition

The attribute value **never changes** in the data warehouse. Facts are always grouped by the original value.

### When to Use

- Attributes explicitly labeled "original" (e.g., customer original credit score, vendor original performance tier)
- Birth dates, hire dates, account opening dates
- Most attributes in date dimension (day of week for 1/1/2024 will always be Monday)
- Durable natural keys (assuming they truly never get reassigned)

### Supply Chain Examples

**Product Dimension:**
- Product introduction date
- Original product line
- Original manufacturer
- First cost basis

**Vendor Dimension:**
- Vendor first contract date
- Original vendor category
- Initial credit limit

**Customer Dimension:**
- Customer account opening date
- Original sales territory
- First order date

### Implementation

No special ETL logic required. Simply do not update the attribute value.

```sql
-- Product dimension with type 0 attributes
CREATE TABLE product_dim (
    product_key INT PRIMARY KEY,  -- Surrogate key
    sku VARCHAR(20),              -- Natural key (type 0 - never reassigned)
    product_description VARCHAR(100),
    introduction_date DATE,       -- Type 0 - never changes
    original_product_line VARCHAR(50),  -- Type 0
    current_product_line VARCHAR(50),   -- Type 1 or 2
    ...
);
```

### Considerations

- Type 0 attributes provide baseline for "before and after" comparisons
- Combination of "original" (type 0) and "current" (type 1) attributes enables change analysis

---

## SCD Type 1: Overwrite

### Definition

**Overwrite** the old attribute value with the new value. The attribute always reflects the most recent assignment. **No history is preserved.**

### When to Use

- Attribute change is a correction of a data entry error
- No analytic value in keeping the old description
- Rapid changes where history tracking creates excessive dimension rows
- Business explicitly confirms historical tracking not needed

### Supply Chain Examples

**Product Dimension:**
- Spelling correction in product description
- Updated supplier phone number
- Current selling price (if not tracked historically)

**Vendor Dimension:**
- Vendor contact person name
- Vendor phone/email (unless tracking for audit)
- Billing address (if only current address matters)

**Customer Dimension:**
- Customer phone number
- Email address
- Mailing preferences

### Implementation

```sql
-- ETL: Detect change and overwrite
UPDATE product_dim
SET product_line = 'Strategy'  -- New value
WHERE sku = 'INTEL-KID-123';   -- Natural key lookup
```

**Before Change:**
```
| product_key | sku | description | product_line | ... |
|-------------|-----|-------------|--------------|-----|
| 12345 | INTEL-KID-123 | IntelliKidz Software | Education | ... |
```

**After Type 1 Change:**
```
| product_key | sku | description | product_line | ... |
|-------------|-----|-------------|--------------|-----|
| 12345 | INTEL-KID-123 | IntelliKidz Software | Strategy | ... |
```

### Critical Implications

**Fact Table Impact:**
- No changes to fact table keys
- Historical facts now appear grouped under new attribute value
- Report results change before vs. after the type 1 change

**Aggregation Table Impact:**
- Pre-existing aggregations based on the changed attribute **must be rebuilt**
- Failure to rebuild causes mismatch between atomic and aggregated data

**OLAP Cube Impact:**
- If attribute is hierarchical, cube **must be reprocessed**
- Cube aggregations must be recalculated

### Example Impact on Reporting

**Fact Data:**
- January sales: $500 for IntelliKidz
- February sales: $100 for IntelliKidz

**Query: Sales by Product Line**

*Before Type 1 Change (product_line = Education):*
```
| Product Line | Sales |
|--------------|-------|
| Education | $600 |
```

*After Type 1 Change (product_line = Strategy):*
```
| Product Line | Sales |
|--------------|-------|
| Strategy | $600 |
```

**Same query, different results!** Users running reports on Jan 31 vs. Feb 1 get different answers.

### Best Practices

- Use sparingly and only with explicit business approval
- Document which attributes are type 1 in metadata
- Consider type 2 as safer default if uncertain about business rules
- Remember: **Once you overwrite, historical value is lost forever**

---

## SCD Type 2: Add New Row

### Definition

When an attribute changes, **insert a new dimension row** with a new surrogate key. The new row reflects the new attribute profile. **This is the primary workhorse technique** for accurately tracking history.

### When to Use

- Business requires accurate historical tracking of attribute changes
- Attribute changes are significant and infrequent
- Need to partition history to analyze "before vs. after" change impact
- Uncertain about business requirements (type 2 is the safe default)

### Supply Chain Examples

**Product Dimension:**
- Product category restructuring
- Product line reassignment
- Package size changes
- Manufacturing location changes

**Vendor Dimension:**
- Vendor performance tier changes (Preferred → Standard)
- Vendor category reclassification
- Geographic region reassignment

**Customer Dimension:**
- Sales territory changes
- Customer credit tier changes
- Customer organizational hierarchy changes

### Implementation with Administrative Columns

**Required Administrative Columns:**
- `row_effective_date` - Date this profile version became valid
- `row_expiration_date` - Date this profile version became invalid
- `current_row_indicator` - Flag to identify current profile (Y/N or True/False)

```sql
CREATE TABLE product_dim (
    product_key INT PRIMARY KEY,      -- Surrogate key (new for each version)
    sku VARCHAR(20),                  -- Natural key (same across versions)
    product_description VARCHAR(100),
    product_line VARCHAR(50),         -- Type 2 attribute
    department VARCHAR(50),           -- Type 2 attribute
    row_effective_date DATE,
    row_expiration_date DATE,
    current_row_indicator CHAR(1)
);
```

**Before Change:**
```
| product_key | sku | description | product_line | dept | effective_date | expiration_date | current |
|-------------|-----|-------------|--------------|------|----------------|-----------------|---------|
| 12345 | INTEL-KID-123 | IntelliKidz | Education | Software | 2012-01-01 | 9999-12-31 | Y |
```

**After Type 2 Change (Feb 1, 2013 - moved to Strategy department):**
```
| product_key | sku | description | product_line | dept | effective_date | expiration_date | current |
|-------------|-----|-------------|--------------|------|----------------|-----------------|---------|
| 12345 | INTEL-KID-123 | IntelliKidz | Education | Software | 2012-01-01 | 2013-01-31 | N |
| 25984 | INTEL-KID-123 | IntelliKidz | Strategy | Software | 2013-02-01 | 9999-12-31 | Y |
```

### ETL Process

```sql
-- Step 1: Expire the current row
UPDATE product_dim
SET row_expiration_date = '2013-01-31',
    current_row_indicator = 'N'
WHERE sku = 'INTEL-KID-123'
  AND current_row_indicator = 'Y';

-- Step 2: Insert new row with new surrogate key
INSERT INTO product_dim (
    product_key, sku, product_description, product_line, department,
    row_effective_date, row_expiration_date, current_row_indicator
)
VALUES (
    25984, 'INTEL-KID-123', 'IntelliKidz Software', 'Strategy', 'Software',
    '2013-02-01', '9999-12-31', 'Y'
);
```

### Fact Table Impact

**Critical: Fact table is UNTOUCHED by type 2 changes.**

Facts created before Feb 1, 2013 continue to reference `product_key = 12345` (Education).
Facts created after Feb 1, 2013 reference `product_key = 25984` (Strategy).

**This perfectly partitions history** to reflect the change.

**Query Example:**

*January + February Sales by Department:*
```sql
SELECT
    p.department,
    SUM(f.sales_amount) as total_sales
FROM sales_fact f
JOIN product_dim p ON f.product_key = p.product_key
WHERE f.order_date BETWEEN '2013-01-01' AND '2013-02-28'
GROUP BY p.department;

Result:
| Department | Total Sales |
|------------|-------------|
| Education | $500 |  -- January sales with product_key 12345
| Strategy | $100 |   -- February sales with product_key 25984
```

### Effective/Expiration Date Usage

**In ETL System:**
Needed when loading historical fact data. ETL must determine which product dimension version was valid at the time of the fact event.

```sql
-- ETL: Find correct product dimension key for historical fact dated 2013-01-15
SELECT product_key
FROM product_dim
WHERE sku = 'INTEL-KID-123'
  AND '2013-01-15' BETWEEN row_effective_date AND row_expiration_date;

Returns: 12345 (the Education version)
```

**In BI Queries:**
**NOT typically needed.** The fact table already contains the correct surrogate key. Constraining on effective/expiration dates is unnecessary and causes confusion.

**Exception:** Supporting "as of" reporting (see Type 7 section).

### Expiration Date Business Rule

**Gap or No Gap?**

**Option 1: No Gap Between Rows (Recommended)**
- Old row expiration = 1 day before new row effective
- Enables `BETWEEN` logic: `fact_date BETWEEN effective_date AND expiration_date`

**Option 2: Exact Boundary**
- Old row expiration = New row effective date (exactly equal)
- Requires different logic: `fact_date >= effective_date AND fact_date < expiration_date`
- Cannot use `BETWEEN`

**Default Future Expiration Date:**
Use `9999-12-31` (not NULL) to avoid complications with `BETWEEN` logic and NULLs.

### Current Row Indicator

**Purpose:** Quickly filter to only current dimension profiles

```sql
-- Get current product list
SELECT sku, product_description, product_line
FROM product_dim
WHERE current_row_indicator = 'Y';
```

**Alternative:** Could use `row_expiration_date = '9999-12-31'` but explicit flag is clearer.

### Natural Key as Glue

The natural key (SKU) holds together all type 2 versions of the same product.

**Counting Products Correctly:**
```sql
-- WRONG: Count surrogate keys
SELECT COUNT(DISTINCT product_key) as product_count
FROM product_dim;
-- Result: 2 (counts both versions of IntelliKidz)

-- CORRECT: Count natural keys
SELECT COUNT(DISTINCT sku) as product_count
FROM product_dim;
-- Result: 1 (correctly identifies single product)
```

### Mixing Type 1 and Type 2 in Same Dimension

**Scenario:** Product dimension has both type 1 and type 2 attributes.

**Example:** Product introduction date is corrected (type 1) after a type 2 change occurred.

**Before Correction:**
```
| product_key | sku | product_line | intro_date | effective_date | current |
|-------------|-----|--------------|------------|----------------|---------|
| 12345 | INTEL-KID-123 | Education | 2012-03-15 | 2012-01-01 | N |
| 25984 | INTEL-KID-123 | Strategy | 2012-03-15 | 2013-02-01 | Y |
```

**After Type 1 Correction (intro_date should be 2012-01-15):**
```
| product_key | sku | product_line | intro_date | effective_date | current |
|-------------|-----|--------------|------------|----------------|---------|
| 12345 | INTEL-KID-123 | Education | 2012-01-15 | 2012-01-01 | N |
| 25984 | INTEL-KID-123 | Strategy | 2012-01-15 | 2013-02-01 | Y |
```

**Business Rule:** Type 1 correction updates **all type 2 versions** of the natural key.

**Data governance must define these rules!**

### Why Type 2 Requires Surrogate Keys

**Cannot use natural key as fact table FK:**
- Multiple dimension rows exist for same natural key
- Fact table would have ambiguous joins

**Cannot use natural key + version digits:**
- Vulnerable to all the operational key problems (reuse, format changes, smart keys)
- Appending effective date is equally problematic

**Solution:** Single-column surrogate key uniquely identifies each version.

### Advantages of Type 2

- **Perfect history partitioning** - Automatically segments facts by attribute value in effect
- **No aggregation rebuilding** required (unlike type 1)
- **No OLAP reprocessing** required for hierarchical attributes (unlike type 1)
- **Safest default** when requirements are unclear
- **Can simulate type 1** with views showing only current rows (cannot do reverse)

### Type 2 Performance Considerations

- Dimension table grows with each change
- For large dimensions with frequent changes, consider type 4 (mini-dimension)
- Indexes on natural key become less selective (multiple rows per natural key)

---

## SCD Type 3: Add New Attribute

### Definition

Add a new column to capture the **prior value** alongside the **current value**. Enables viewing facts by either current or previous attribute values simultaneously.

### When to Use

- Need to support **alternate realities** (view facts by current OR prior structure)
- Predictable, significant changes affecting many rows (reorganizations)
- Need to track only 1-2 historical values (not unlimited history)
- Business wants to analyze performance "as if" the change never occurred

### Supply Chain Examples

**Product Dimension:**
- Product line reorganization: current_product_line + prior_product_line
- Department restructuring: current_department + 2023_department + 2022_department

**Vendor Dimension:**
- Vendor category reassignment: current_category + prior_category
- Geographic territory reorganization: current_region + prior_region

**Customer Dimension:**
- Sales territory reorganization: current_territory + prior_territory
- Account manager reassignment: current_account_mgr + prior_account_mgr

### When NOT to Use

- Unpredictable changes at different times for different rows (customer home state changes)
- Need to track unlimited history
- Changes are infrequent enough for type 2

### Implementation

**Before Reorganization (All products in Education dept):**
```
| product_key | sku | description | department | prior_department |
|-------------|-----|-------------|------------|------------------|
| 12345 | INTEL-KID-123 | IntelliKidz | Education | NULL |
```

**After Reorganization (Feb 1, 2013 - moved to Strategy):**
```
| product_key | sku | description | department | prior_department |
|-------------|-----|-------------|------------|------------------|
| 12345 | INTEL-KID-123 | IntelliKidz | Strategy | Education |
```

### ETL Process

```sql
-- Step 1: Alter table to add prior column (one-time)
ALTER TABLE product_dim ADD COLUMN prior_department VARCHAR(50);

-- Step 2: On reorganization, populate prior with current values
UPDATE product_dim
SET prior_department = department;

-- Step 3: Overwrite current column with new values
UPDATE product_dim
SET department = 'Strategy'
WHERE sku = 'INTEL-KID-123';
```

### Fact Table Impact

**No changes to fact table.** All facts reference same `product_key = 12345`.

### Querying Both Views

**Current Department View:**
```sql
SELECT
    p.department,
    SUM(f.sales_amount) as total_sales
FROM sales_fact f
JOIN product_dim p ON f.product_key = p.product_key
GROUP BY p.department;

Result:
| Department | Total Sales |
|------------|-------------|
| Strategy | $600 |  -- All sales (Jan + Feb) grouped by current dept
```

**Prior Department View:**
```sql
SELECT
    p.prior_department,
    SUM(f.sales_amount) as total_sales
FROM sales_fact f
JOIN product_dim p ON f.product_key = p.product_key
GROUP BY p.prior_department;

Result:
| Prior Department | Total Sales |
|------------------|-------------|
| Education | $600 |  -- All sales (Jan + Feb) grouped by prior dept
```

**Both Views Simultaneously:**
Enables "What if we hadn't reorganized?" analysis.

### Multiple Type 3 Attributes (Predictable Rhythm)

**Scenario:** Product line recategorized every January.

```
| product_key | sku | current_product_line | 2024_product_line | 2023_product_line | 2022_product_line |
|-------------|-----|----------------------|-------------------|-------------------|-------------------|
| 12345 | ABC-123 | Games | Games | Education | Education |
| 67890 | XYZ-789 | STEM | Education | Education | Sports |
```

**Annual ETL Process:**
1. Alter table to add new column: `2024_product_line`
2. Populate new column with current values: `2024_product_line = current_product_line`
3. Overwrite current with new assignments: `current_product_line = <new values>`

**Business users can analyze by:**
- Current structure
- 2024 structure
- 2023 structure
- 2022 structure

**Column Naming:** Use year or descriptive labels like "pre_merger_department" and "post_merger_department"

### Type 3 OLAP Cube Impact

If hierarchical attribute is type 3, cube likely needs reprocessing when:
- New prior column is added
- Current column is updated

---

## SCD Type 4: Mini-Dimension

### Definition

Split frequently changing or frequently analyzed attributes into a **separate dimension table** (mini-dimension). The mini-dimension FK is included in the fact table, enabling change tracking without exploding the main dimension.

### When to Use

- **Large dimension** (millions of rows) with **frequently changing attributes**
- **Frequently analyzed volatile attributes** causing query performance issues
- Type 2 would create excessive dimension rows
- Attributes can be discretized into value bands

### Supply Chain Examples

**Customer Demographics Mini-Dimension:**
- Age band (18-24, 25-34, 35-44, etc.)
- Income band ($0-30K, $30-60K, $60-100K, $100K+)
- Purchase frequency score (Low/Medium/High)
- Credit tier (A/B/C/D)

**Product Performance Mini-Dimension:**
- Sales velocity tier (Slow/Medium/Fast)
- Margin tier (Low/Standard/Premium)
- Inventory turns tier (Low/Medium/High)
- Lifecycle stage (Introduction/Growth/Mature/Decline)

**Vendor Performance Mini-Dimension:**
- On-time delivery tier (Below Standard/Standard/Excellent)
- Quality score band (1-2/3-4/5-6/7-8/9-10)
- Fill rate tier (< 90% / 90-95% / 95-98% / >98%)

### Why Separate from Main Dimension

**Problem with Type 2 for Volatile Attributes:**

Customer dimension: 5 million customers
Customer income changes annually → 5 million new dimension rows per year
Customer age changes every birthday → Potentially 5 million new rows per year

**Solution:** Mini-dimension decouples volatile attributes.

### Value Banding Requirement

**Cannot store precise continuous values** in mini-dimension or it defeats the purpose.

**Bad (Defeats Purpose):**
```
| demo_key | exact_age | exact_income | purchase_frequency |
|----------|-----------|--------------|---------------------|
| 1 | 25 | $47,382.15 | 14 |
| 2 | 25 | $47,500.00 | 14 |
| 3 | 26 | $47,382.15 | 15 |
```
Result: Nearly as many mini-dimension rows as customer rows!

**Good (Enables Grouping):**
```
| demo_key | age_band | income_band | purchase_freq_tier |
|----------|----------|-------------|--------------------|
| 1 | 25-34 | $40-60K | Medium |
| 142 | 25-34 | $20-40K | Low |
| 573 | 35-44 | $40-60K | Medium |
```
Result: Small, manageable mini-dimension (hundreds to low thousands of rows).

### Mini-Dimension Structure

```sql
CREATE TABLE customer_demographics_dim (
    demographics_key INT PRIMARY KEY,     -- Surrogate key
    age_band VARCHAR(20),                -- "18-24", "25-34", "35-44", etc.
    income_band VARCHAR(20),             -- "$0-30K", "$30-60K", etc.
    purchase_frequency_tier VARCHAR(20), -- "Low", "Medium", "High"
    credit_tier CHAR(1)                  -- "A", "B", "C", "D"
);

-- Sample rows
| demographics_key | age_band | income_band | purchase_freq_tier | credit_tier |
|------------------|----------|-------------|--------------------|-------------|
| 1 | 25-34 | $20-40K | Low | B |
| 2 | 25-34 | $20-40K | Medium | B |
| 3 | 25-34 | $20-40K | High | B |
| ...142 | 25-34 | $40-60K | Low | A |
| ...573 | 35-44 | $40-60K | Medium | A |
```

**Sizing Calculation:**
- 8 age bands × 6 income bands × 3 purchase frequency tiers × 4 credit tiers = **576 rows**
- vs. 5 million customer rows

### Fact Table Integration

**Each fact table row includes demographics FK:**

```sql
CREATE TABLE sales_fact (
    order_date_key INT,
    customer_key INT,              -- FK to customer_dim (5M rows)
    demographics_key INT,          -- FK to customer_demographics_dim (576 rows)
    product_key INT,
    sales_amount DECIMAL(10,2),
    ...
);
```

**Benefits:**
- **Performance:** Small demographics dimension is fast for filtering/grouping
- **Change Tracking:** Demographics key changes in fact table over time capture profile changes
- **Avoids main dimension:** Can analyze demographics without touching huge customer dimension

### Change Tracking via Fact Table

**January Sales Fact (Customer John Smith is 25 years old, income $25K):**
```
| order_date | customer_key | demographics_key | product_key | sales_amount |
|------------|--------------|------------------|-------------|--------------|
| 2024-01-15 | 987654 | 1 | 12345 | $150 |
```

Demographics key 1 = Age 25-34, Income $20-40K, Low purchase frequency

**March Sales Fact (John had birthday, now 26, same profile band):**
```
| 2024-03-10 | 987654 | 1 | 67890 | $200 |
```

Demographics key still 1 (birthday doesn't change age band)

**June Sales Fact (John got raise to $45K, moves to new income band):**
```
| 2024-06-20 | 987654 | 142 | 11111 | $300 |
```

Demographics key now 142 = Age 25-34, Income $40-60K, Low purchase frequency

**Fact table automatically tracks demographic changes without touching customer dimension!**

### Value Band Granularity Trade-offs

**Coarse Bands (Fewer Rows):**
```
Age: Under 35 / 35-55 / Over 55
Income: Under $50K / $50-100K / Over $100K
```
Pros: Very small mini-dimension (e.g., 27 combinations = 3 × 3 × 3)
Cons: Loss of analytical precision

**Fine Bands (More Rows):**
```
Age: 18-21, 22-24, 25-29, 30-34, 35-39, 40-44, 45-49, 50-54, 55-59, 60+
Income: $0-20K, $20-30K, $30-40K, $40-50K, $50-75K, $75-100K, $100K+
```
Pros: Better analytical precision
Cons: Larger mini-dimension (e.g., 10 × 7 = 70 age/income combinations before other attributes)

**Guideline:** Keep total mini-dimension rows under **100,000**. If exceeds, use multiple mini-dimensions or coarsen bands.

### Storing Precise Values in Fact Table

If users absolutely need access to precise values (e.g., exact credit score), include as fact:

```sql
CREATE TABLE sales_fact (
    order_date_key INT,
    customer_key INT,
    demographics_key INT,          -- Banded demographics
    product_key INT,
    sales_amount DECIMAL(10,2),
    customer_credit_score INT,     -- Precise value as fact
    customer_income DECIMAL(10,2), -- Precise value as fact
    ...
);
```

**Trade-off:** Fact table becomes wider, but supports both banded analysis (via mini-dimension) and precise filtering (via fact columns).

### Mini-Dimension Limitations

- **Value banding** is permanent; changing bands later is difficult
- **Not suitable for** attributes requiring precise values (phone numbers, addresses)
- **Adds complexity** - users see two ways to access demographic data

---

## (Continued in next message due to length...)
