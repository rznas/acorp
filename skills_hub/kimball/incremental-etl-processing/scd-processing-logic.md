# SCD Processing Logic for Incremental Loads

Complete reference for implementing slowly changing dimension updates during incremental ETL processing.

## Overview

During incremental dimension processing, you must determine if each incoming row is:
1. A new dimension member (insert)
2. An unchanged dimension member (no action)
3. A changed dimension member requiring Type 1, Type 2, or Type 3 handling

This document provides detailed logic flows and SQL examples for each scenario.

---

## Processing Flow

```
Incoming Dimension Row
    |
    ├─→ Lookup on Natural Key
    |       |
    |       ├─→ Not Found: NEW MEMBER
    |       |       └─→ Insert with new surrogate key
    |       |
    |       └─→ Found: Check for Changes
    |               |
    |               ├─→ No Changes: SKIP (no action needed)
    |               |
    |               └─→ Changes Detected
    |                       |
    |                       ├─→ Type 1 Attributes Changed
    |                       |       └─→ Overwrite ALL rows for this natural key
    |                       |
    |                       ├─→ Type 2 Attributes Changed
    |                       |       └─→ Expire current row, insert new row
    |                       |
    |                       └─→ Type 3 Attributes Changed
    |                               └─→ Update columns (shift old→previous, new→current)
```

---

## Type 1: Overwrite Processing

**When**: Attribute change is a correction or does not require history tracking.

**Examples**:
- Corrected misspelling in customer name
- Updated email address
- Refreshed phone number
- Fixed categorization error

### Implementation Logic

```sql
-- Step 1: Identify Type 1 changes
-- Compare Type 1 attributes only
SELECT s.customer_id,
       s.customer_email,
       s.customer_phone,
       s.customer_name
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE (s.customer_email <> d.customer_email
   OR s.customer_phone <> d.customer_phone
   OR s.customer_name <> d.customer_name);

-- Step 2: Update ALL rows for this natural key
-- Important: Update all versions, not just current
UPDATE dim.customer
SET customer_email = :new_email,
    customer_phone = :new_phone,
    customer_name = :new_name,
    last_update_date = CURRENT_TIMESTAMP
WHERE customer_natural_key = :natural_key;
```

### Critical Points

**Update all rows**: Type 1 means "this was always true"
```sql
-- CORRECT: Updates all historical versions
UPDATE dim.customer
SET customer_category = 'Gold'
WHERE customer_natural_key = 'C12345';

-- WRONG: Only updates current row
UPDATE dim.customer
SET customer_category = 'Gold'
WHERE customer_natural_key = 'C12345'
  AND current_row_indicator = 'Y';
```

**Impact on aggregates**: Type 1 changes invalidate aggregates built on changed attributes.

```sql
-- If customer_category changed (Type 1), must rebuild aggregates
-- Dimension manager must notify fact provider
DELETE FROM agg.sales_by_customer_category;
-- Rebuild from atomic facts
```

**Do not change surrogate keys**: Type 1 updates overwrite values, not keys.

---

## Type 2: Add New Row Processing

**When**: Attribute change requires historical tracking.

**Examples**:
- Customer moved to new address
- Product changed category
- Employee promoted to new department
- Account upgraded to premium tier

### Implementation Logic

```sql
-- Step 1: Identify Type 2 changes
SELECT s.customer_id,
       s.customer_address,
       s.customer_city,
       s.customer_state,
       s.customer_segment
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE (s.customer_address <> d.customer_address
   OR s.customer_city <> d.customer_city
   OR s.customer_state <> d.customer_state
   OR s.customer_segment <> d.customer_segment);

-- Step 2: Expire the current row
UPDATE dim.customer
SET row_end_date = CURRENT_DATE - 1,  -- Yesterday
    current_row_indicator = 'N',
    row_end_datetime = CURRENT_TIMESTAMP
WHERE customer_natural_key = :natural_key
  AND current_row_indicator = 'Y';

-- Step 3: Insert new row with new surrogate key
INSERT INTO dim.customer (
    customer_key,              -- New surrogate key
    customer_natural_key,      -- Same natural key
    customer_name,             -- Copy from previous or new
    customer_email,            -- Type 1 attributes from current values
    customer_phone,
    customer_address,          -- New Type 2 value
    customer_city,             -- New Type 2 value
    customer_state,            -- New Type 2 value
    customer_segment,          -- New Type 2 value
    row_effective_date,        -- Today
    row_effective_datetime,
    row_end_date,              -- Far future
    row_end_datetime,
    current_row_indicator,     -- 'Y'
    change_reason,             -- Optional description
    source_system_id
) VALUES (
    customer_seq.NEXTVAL,      -- Generate new key
    :natural_key,
    :customer_name,            -- Most current value
    :customer_email,
    :customer_phone,
    :new_address,              -- Updated values
    :new_city,
    :new_state,
    :new_segment,
    CURRENT_DATE,              -- Effective today
    CURRENT_TIMESTAMP,
    DATE '9999-12-31',         -- Maximum date
    TIMESTAMP '9999-12-31 23:59:59',
    'Y',
    'Address Change',          -- Why row was created
    :source_system
);

-- Step 4: Update surrogate key map for fact table lookups
UPDATE etl.customer_key_map
SET current_surrogate_key = :new_surrogate_key,
    last_update_date = CURRENT_TIMESTAMP
WHERE natural_key = :natural_key;
```

### Five Essential Housekeeping Columns

```sql
-- Add these columns to Type 2 dimensions
ALTER TABLE dim.customer ADD (
    row_effective_date        DATE NOT NULL,
    row_effective_datetime    TIMESTAMP,
    row_end_date             DATE NOT NULL,  -- Never NULL!
    row_end_datetime         TIMESTAMP,
    current_row_indicator    CHAR(1) NOT NULL,  -- 'Y' or 'N'
    change_reason            VARCHAR(100)
);
```

**Why row_end_date should never be NULL**:
```sql
-- This query fails if row_end_date is NULL
SELECT * FROM dim.customer
WHERE '2024-01-15' BETWEEN row_effective_date AND row_end_date;

-- Use maximum date instead
-- row_end_date = DATE '9999-12-31' for current rows
```

### Closing Date Gap Requirements

Ensure no gaps between consecutive dimension versions:

```sql
-- CORRECT: No gap
-- Old row: row_end_date = 2024-03-14
-- New row: row_effective_date = 2024-03-15

-- WRONG: One day gap
-- Old row: row_end_date = 2024-03-14
-- New row: row_effective_date = 2024-03-16  -- Missing 3/15!
```

**For day-level granularity**:
```sql
SET row_end_date = CURRENT_DATE - 1
```

**For second-level granularity**:
```sql
-- End timestamp = exactly the begin timestamp of next row
SET row_end_datetime = :new_row_begin_datetime
```

### Handling Mixed Type 1 + Type 2 Changes

When same row has both Type 1 and Type 2 changes:

```sql
-- Scenario: Name (Type 1) AND address (Type 2) both changed

-- Step 1: Update ALL rows for Type 1 attributes first
UPDATE dim.customer
SET customer_name = :corrected_name
WHERE customer_natural_key = :natural_key;

-- Step 2: Then handle Type 2 (expire + insert)
UPDATE dim.customer
SET row_end_date = CURRENT_DATE - 1,
    current_row_indicator = 'N'
WHERE customer_natural_key = :natural_key
  AND current_row_indicator = 'Y';

INSERT INTO dim.customer (...)
VALUES (..., :corrected_name, :new_address, ...);
```

---

## Type 3: Add New Attribute Processing

**When**: Need to compare both old and new values in queries.

**Example**: Sales region reorganization - want to analyze by both old and new regions.

### Schema Change Required

```sql
-- Add columns to track previous value
ALTER TABLE dim.customer ADD (
    customer_region_current VARCHAR(50),
    customer_region_previous VARCHAR(50),
    region_change_date DATE
);
```

### Implementation Logic

```sql
-- Step 1: Detect change in Type 3 attribute
SELECT s.customer_id,
       s.customer_region as new_region,
       d.customer_region as current_region
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
WHERE s.customer_region <> d.customer_region;

-- Step 2: Shift values and update
UPDATE dim.customer
SET customer_region_previous = customer_region_current,  -- Save old
    customer_region_current = :new_region,               -- Set new
    region_change_date = CURRENT_DATE
WHERE customer_natural_key = :natural_key;
```

### Type 3 Limitations

- Only tracks one previous value (not full history)
- Requires schema change for each new alternate reality
- Not scalable for frequent changes
- Better to use Type 2 for most historical tracking

**When Type 3 is appropriate**:
- Known finite change (e.g., organizational restructuring)
- Need to compare both perspectives in same query
- Change happens rarely

---

## Advanced SCD Types (Hybrid Techniques)

### Type 4: Mini-Dimension

**When**: Rapidly changing attributes in large dimension (million+ rows).

**Example**: Customer demographics that change frequently.

```sql
-- Customer dimension (stable attributes)
CREATE TABLE dim.customer (
    customer_key INTEGER PRIMARY KEY,
    customer_natural_key VARCHAR(50),
    customer_name VARCHAR(100),
    customer_birth_date DATE
    -- No rapidly changing demographics here
);

-- Customer demographics mini-dimension
CREATE TABLE dim.customer_demographics (
    demographics_key INTEGER PRIMARY KEY,
    income_range VARCHAR(20),
    credit_score_range VARCHAR(20),
    number_of_children INTEGER
);

-- Fact table references BOTH
CREATE TABLE fact.sales (
    date_key INTEGER,
    customer_key INTEGER,          -- Stable customer
    demographics_key INTEGER,      -- Demographics at time of sale
    product_key INTEGER,
    quantity INTEGER,
    amount DECIMAL(10,2)
);
```

**Processing**:
```sql
-- New demographics combination encountered
-- Insert into mini-dimension
INSERT INTO dim.customer_demographics (
    demographics_key,
    income_range,
    credit_score_range,
    number_of_children
) VALUES (
    demographics_seq.NEXTVAL,
    :income_range,
    :credit_score,
    :num_children
);

-- Use mini-dimension key in fact table
```

### Type 5: Mini-Dimension + Type 1 Outrigger

**When**: Want current demographics on customer dimension AND historical on facts.

```sql
-- Add Type 1 reference to current demographics
ALTER TABLE dim.customer ADD (
    current_demographics_key INTEGER  -- Type 1: overwritten
);

-- Overwrite when demographics change
UPDATE dim.customer
SET current_demographics_key = :new_demographics_key
WHERE customer_key = :customer_key;

-- Fact table still has historical demographics_key
```

### Type 6: Type 2 + Type 1 Attributes

**When**: Need both historical tracking AND current value on same row.

```sql
-- Dimension has both versions of attribute
CREATE TABLE dim.product (
    product_key INTEGER PRIMARY KEY,
    product_natural_key VARCHAR(50),
    product_name VARCHAR(100),
    product_category_current VARCHAR(50),  -- Type 1: always current
    product_category_historical VARCHAR(50),-- Type 2: as-was
    row_effective_date DATE,
    row_end_date DATE,
    current_row_indicator CHAR(1)
);

-- When category changes
-- Step 1: Update current category in ALL rows (Type 1)
UPDATE dim.product
SET product_category_current = :new_category
WHERE product_natural_key = :natural_key;

-- Step 2: Create new Type 2 row
-- (expire old + insert new with historical category)
```

### Type 7: Dual Type 1 and Type 2 Dimensions

**When**: Support both historical and current perspective queries easily.

```sql
-- Fact table has TWO foreign keys to same dimension
CREATE TABLE fact.sales (
    date_key INTEGER,
    customer_key INTEGER,           -- Type 2: historical
    customer_durable_key INTEGER,   -- Type 1: current
    product_key INTEGER,
    quantity INTEGER,
    amount DECIMAL(10,2)
);

-- customer_key: joins to row effective at transaction time
-- customer_durable_key: joins to current row (via natural key)

-- Dimension includes durable key
CREATE TABLE dim.customer (
    customer_key INTEGER PRIMARY KEY,              -- Surrogate (changes)
    customer_durable_key VARCHAR(50) NOT NULL,    -- Natural (constant)
    customer_name VARCHAR(100),
    row_effective_date DATE,
    current_row_indicator CHAR(1)
);
```

---

## Change Detection Optimization Techniques

### Hash Comparison for Wide Dimensions

```sql
-- Pre-compute hashes in dimension table
ALTER TABLE dim.product ADD (
    hash_type1_attributes CHAR(32),
    hash_type2_attributes CHAR(32)
);

-- During dimension load, compute hash on Type 1 attributes
UPDATE dim.product
SET hash_type1_attributes = MD5(CONCAT(
        COALESCE(product_name, ''),
        COALESCE(product_description, ''),
        COALESCE(manufacturer, '')
    ))
WHERE current_row_indicator = 'Y';

-- Similarly for Type 2 attributes
UPDATE dim.product
SET hash_type2_attributes = MD5(CONCAT(
        COALESCE(product_category, ''),
        COALESCE(product_subcategory, ''),
        COALESCE(product_status, '')
    ))
WHERE current_row_indicator = 'Y';

-- Fast change detection
SELECT s.*
FROM staging.product_extract s
INNER JOIN dim.product d
  ON s.product_id = d.product_natural_key
  AND d.current_row_indicator = 'Y'
WHERE MD5(CONCAT(s.category, s.subcategory, s.status)) <> d.hash_type2_attributes;
```

### DBMS-Specific Change Detection

```sql
-- SQL Server EXCEPT operator
SELECT * FROM staging.customer_extract
EXCEPT
SELECT customer_natural_key, customer_name, customer_address, customer_city
FROM dim.customer
WHERE current_row_indicator = 'Y';

-- Oracle MINUS operator
SELECT * FROM staging.customer_extract
MINUS
SELECT customer_natural_key, customer_name, customer_address, customer_city
FROM dim.customer
WHERE current_row_indicator = 'Y';
```

---

## Error Handling and Edge Cases

### Scenario 1: Multiple Changes Same Day

```sql
-- If dimension extract contains multiple changes to same natural key
-- Process only the most recent version

WITH ranked_changes AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id
               ORDER BY extract_timestamp DESC
           ) as rn
    FROM staging.customer_extract
)
SELECT * FROM ranked_changes WHERE rn = 1;
```

### Scenario 2: Missing Natural Key Data

```sql
-- Handle NULL or blank natural keys
-- Create specific "Unknown" row per missing value pattern

IF :natural_key IS NULL THEN
    -- Use special "Unknown - NULL" dimension member
    :surrogate_key := -1;  -- Reserved for NULL natural key
ELSIF :natural_key = '' THEN
    :surrogate_key := -2;  -- Reserved for blank natural key
ELSE
    -- Normal lookup/insert logic
END IF;
```

### Scenario 3: Restatement of History

**Requirement**: Retroactively change historical dimension value (rare).

```sql
-- Find the dimension row effective on specific historical date
SELECT customer_key
FROM dim.customer
WHERE customer_natural_key = :natural_key
  AND :historical_date BETWEEN row_effective_date AND row_end_date;

-- Update that specific row (changes history!)
UPDATE dim.customer
SET customer_segment = :corrected_value,
    restatement_flag = 'Y',
    restatement_date = CURRENT_DATE,
    restatement_reason = :reason
WHERE customer_key = :historical_customer_key;

-- WARNING: Changes historical facts' dimensional context
-- May require aggregate rebuilds
-- Document heavily for compliance
```

---

## Best Practices Summary

1. **Always use surrogate keys** - Never use natural keys as dimension primary keys
2. **Never NULL in row_end_date** - Use maximum date (9999-12-31) for current rows
3. **Update ALL rows for Type 1** - Ensures consistency across history
4. **Maintain key map table** - Speeds up fact table lookups
5. **No gaps in effective dates** - Prevents query errors and data ambiguity
6. **Document SCD policy per attribute** - Make explicit in data model
7. **Test with production-like volumes** - SCD processing can be slow
8. **Monitor dimension growth** - Type 2 can balloon table size
9. **Archive superseded rows** - Consider purging very old Type 2 versions
10. **Version dimension definitions** - Notify downstream aggregates of changes

---

## SQL Template: Complete SCD Processing

```sql
-- Complete SCD processing for incremental dimension load

-- Input: staging.customer_extract
-- Output: dim.customer (updated)

-- Step 1: Identify new members
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    customer_address,
    customer_segment,
    row_effective_date,
    row_end_date,
    current_row_indicator
)
SELECT
    customer_seq.NEXTVAL,
    s.customer_id,
    s.customer_name,
    s.customer_address,
    s.customer_segment,
    CURRENT_DATE,
    DATE '9999-12-31',
    'Y'
FROM staging.customer_extract s
LEFT JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE d.customer_key IS NULL;

-- Step 2: Identify and process Type 1 changes
UPDATE dim.customer d
SET (customer_name, customer_email) = (
        SELECT s.customer_name, s.customer_email
        FROM staging.customer_extract s
        WHERE s.customer_id = d.customer_natural_key
    )
WHERE d.customer_natural_key IN (
    SELECT s.customer_id
    FROM staging.customer_extract s
    INNER JOIN dim.customer d2
      ON s.customer_id = d2.customer_natural_key
      AND d2.current_row_indicator = 'Y'
    WHERE s.customer_name <> d2.customer_name
       OR s.customer_email <> d2.customer_email
);

-- Step 3: Expire rows with Type 2 changes
UPDATE dim.customer d
SET row_end_date = CURRENT_DATE - 1,
    current_row_indicator = 'N'
WHERE d.current_row_indicator = 'Y'
  AND EXISTS (
      SELECT 1
      FROM staging.customer_extract s
      WHERE s.customer_id = d.customer_natural_key
        AND (s.customer_address <> d.customer_address
         OR s.customer_segment <> d.customer_segment)
  );

-- Step 4: Insert new Type 2 rows
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    customer_email,
    customer_address,
    customer_segment,
    row_effective_date,
    row_end_date,
    current_row_indicator
)
SELECT
    customer_seq.NEXTVAL,
    s.customer_id,
    s.customer_name,    -- Most current value
    s.customer_email,   -- Most current value
    s.customer_address, -- Changed value
    s.customer_segment, -- Changed value
    CURRENT_DATE,
    DATE '9999-12-31',
    'Y'
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'N'   -- Just expired
  AND d.row_end_date = CURRENT_DATE - 1;  -- Today

-- Step 5: Validate no duplicates
SELECT customer_natural_key, COUNT(*)
FROM dim.customer
WHERE current_row_indicator = 'Y'
GROUP BY customer_natural_key
HAVING COUNT(*) > 1;
-- Should return 0 rows
```

---

This reference provides the complete technical foundation for implementing SCD logic in incremental dimension processing. Combine with change data capture (CDC) and dimension transformation logic for complete dimension ETL solution.
