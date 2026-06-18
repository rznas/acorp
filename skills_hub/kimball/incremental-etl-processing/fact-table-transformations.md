# Fact Table Transformations for Incremental Loading

Detailed reference for transforming and preparing fact data during incremental ETL processing.

## Overview

Fact table transformations convert extracted source data into clean, validated, grain-consistent measurements ready for loading into dimensional models. These transformations apply equally to both historic and incremental loads, but incremental processing adds complexity around updates and late-arriving data.

---

## Transformation Categories

### 1. Data Type Conversions
### 2. NULL Value Handling
### 3. Grain Validation and Filtering
### 4. Derived Fact Calculation
### 5. Multiple Units of Measure
### 6. Currency Conversion
### 7. Allocations and Apportionments
### 8. Fact Aggregation
### 9. Error Correction Processing

---

## 1. Data Type Conversions

### Numeric Conversions

```sql
-- Convert string to numeric, handling errors
SELECT
    order_id,
    CAST(quantity AS INTEGER) as quantity,
    -- Safe conversion with default for errors
    CASE
        WHEN amount ~ '^[0-9]+\.?[0-9]*$' THEN CAST(amount AS DECIMAL(10,2))
        ELSE 0.00
    END as amount,
    -- Remove currency symbols before converting
    CAST(REPLACE(REPLACE(discount, '$', ''), ',', '') AS DECIMAL(10,2)) as discount
FROM staging.sales_extract;
```

### Date/Time Conversions

```sql
-- Standardize various date formats
SELECT
    order_id,
    -- Parse common formats
    CASE
        WHEN order_date ~ '^\d{4}-\d{2}-\d{2}$' THEN TO_DATE(order_date, 'YYYY-MM-DD')
        WHEN order_date ~ '^\d{2}/\d{2}/\d{4}$' THEN TO_DATE(order_date, 'MM/DD/YYYY')
        WHEN order_date ~ '^\d{2}-\w{3}-\d{4}$' THEN TO_DATE(order_date, 'DD-MON-YYYY')
        ELSE NULL
    END as order_date_clean,
    -- Extract time component
    CASE
        WHEN order_timestamp ~ '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$'
        THEN TO_TIMESTAMP(order_timestamp, 'YYYY-MM-DD HH24:MI:SS')
        ELSE CURRENT_TIMESTAMP  -- Default if parsing fails
    END as order_timestamp_clean
FROM staging.sales_extract;
```

### String Standardization

```sql
-- Clean and standardize text fields
SELECT
    order_id,
    -- Trim whitespace
    TRIM(customer_name) as customer_name,
    -- Uppercase for consistency
    UPPER(TRIM(state_code)) as state_code,
    -- Remove special characters
    REGEXP_REPLACE(phone_number, '[^0-9]', '') as phone_number_clean,
    -- Standardize case
    INITCAP(TRIM(city_name)) as city_name
FROM staging.sales_extract;
```

---

## 2. NULL Value Handling

### NULL Detection and Replacement

```sql
-- Identify and handle various NULL representations
SELECT
    order_id,
    -- Replace string 'NULL' with actual NULL
    CASE
        WHEN UPPER(TRIM(quantity)) IN ('NULL', 'N/A', 'NONE', '') THEN NULL
        ELSE CAST(quantity AS INTEGER)
    END as quantity,
    -- Replace NULL with default value
    COALESCE(discount_amount, 0.00) as discount_amount,
    -- Replace NULL with derived value
    COALESCE(
        net_amount,
        gross_amount - COALESCE(discount_amount, 0.00)
    ) as net_amount
FROM staging.sales_extract;
```

### Mandatory Field Validation

```sql
-- Screen for required fields
SELECT
    order_id,
    quantity,
    amount,
    CASE
        WHEN order_id IS NULL THEN 'Missing order_id'
        WHEN quantity IS NULL THEN 'Missing quantity'
        WHEN amount IS NULL THEN 'Missing amount'
        WHEN customer_id IS NULL THEN 'Missing customer_id'
        ELSE 'OK'
    END as validation_status
FROM staging.sales_extract;

-- Filter valid records and write errors to error schema
INSERT INTO staging.sales_validated
SELECT * FROM staging.sales_extract
WHERE order_id IS NOT NULL
  AND quantity IS NOT NULL
  AND amount IS NOT NULL
  AND customer_id IS NOT NULL;

-- Log validation errors
INSERT INTO etl.error_event_fact (
    error_date_key,
    error_timestamp,
    screen_key,
    table_name,
    record_id,
    error_message
)
SELECT
    :current_date_key,
    CURRENT_TIMESTAMP,
    :null_check_screen_key,
    'sales_extract',
    order_id,
    validation_status
FROM staging.sales_extract
WHERE validation_status <> 'OK';
```

---

## 3. Grain Validation and Filtering

### Ensure Single Grain

**Problem**: Source extract contains mixed grains (order header + order lines).

```sql
-- Extract only line-level grain
SELECT
    order_id,
    line_number,        -- Grain indicator
    product_id,
    quantity,
    line_amount
FROM staging.order_extract
WHERE line_number IS NOT NULL  -- Filter out header rows
  AND line_number > 0;

-- Alternatively: Filter out summary rows
SELECT *
FROM staging.sales_extract
WHERE record_type = 'DETAIL'  -- Exclude 'SUMMARY', 'HEADER'
  AND product_id <> 'TOTAL';  -- Exclude total lines
```

### Deduplication

**Problem**: Source contains duplicate records.

```sql
-- Identify duplicates
SELECT
    order_id,
    line_number,
    COUNT(*) as duplicate_count
FROM staging.sales_extract
GROUP BY order_id, line_number
HAVING COUNT(*) > 1;

-- Remove duplicates using ROW_NUMBER
WITH ranked_records AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY order_id, line_number
               ORDER BY extract_timestamp DESC  -- Keep most recent
           ) as rn
    FROM staging.sales_extract
)
SELECT * FROM ranked_records
WHERE rn = 1;

-- Alternative: Use DISTINCT if records are truly identical
SELECT DISTINCT
    order_id,
    line_number,
    product_id,
    quantity,
    amount
FROM staging.sales_extract;
```

### Fact Row Filtering

```sql
-- Business rule: Exclude cancelled orders
DELETE FROM staging.sales_fact
WHERE order_status = 'CANCELLED';

-- Business rule: Exclude $0 transactions (unless meaningful)
DELETE FROM staging.sales_fact
WHERE amount = 0
  AND quantity = 0;

-- Business rule: Exclude test transactions
DELETE FROM staging.sales_fact
WHERE customer_id IN (SELECT customer_id FROM dim.customer WHERE test_account_flag = 'Y');

-- Business rule: Minimum transaction threshold
DELETE FROM staging.sales_fact
WHERE amount < 0.01;
```

---

## 4. Derived Fact Calculation

### Simple Calculations

```sql
-- Calculate derived facts from base facts
SELECT
    order_id,
    line_number,
    quantity,
    unit_price,
    -- Derived: extended amount
    quantity * unit_price as extended_amount,
    -- Derived: discount amount
    (quantity * unit_price) * (discount_percent / 100.0) as discount_amount,
    -- Derived: net amount
    (quantity * unit_price) * (1 - discount_percent / 100.0) as net_amount,
    -- Derived: margin
    ((quantity * unit_price) - (quantity * unit_cost)) as margin_amount,
    -- Derived: margin percent
    CASE
        WHEN unit_price > 0 THEN
            ((unit_price - unit_cost) / unit_price) * 100.0
        ELSE 0
    END as margin_percent
FROM staging.sales_extract;
```

### Profit Calculations

```sql
-- Join to get product cost for profit calculation
SELECT
    s.order_id,
    s.product_id,
    s.quantity,
    s.revenue_amount,
    p.standard_cost,
    -- Derived: cost of goods sold
    s.quantity * p.standard_cost as cogs_amount,
    -- Derived: gross profit
    s.revenue_amount - (s.quantity * p.standard_cost) as gross_profit_amount,
    -- Derived: gross profit margin
    CASE
        WHEN s.revenue_amount > 0 THEN
            ((s.revenue_amount - (s.quantity * p.standard_cost)) / s.revenue_amount) * 100.0
        ELSE 0
    END as gross_profit_margin_percent
FROM staging.sales_fact s
LEFT JOIN dim.product p
  ON s.product_id = p.product_natural_key
  AND p.current_row_indicator = 'Y';
```

### Time-Based Calculations

```sql
-- Calculate duration facts
SELECT
    order_id,
    order_date,
    ship_date,
    delivery_date,
    -- Derived: days to ship
    ship_date - order_date as days_to_ship,
    -- Derived: days in transit
    delivery_date - ship_date as days_in_transit,
    -- Derived: total fulfillment days
    delivery_date - order_date as total_fulfillment_days,
    -- Derived: on-time delivery flag
    CASE
        WHEN delivery_date <= promised_delivery_date THEN 'Y'
        ELSE 'N'
    END as on_time_delivery_flag
FROM staging.order_fact;
```

---

## 5. Multiple Units of Measure

Store facts in multiple measurement units for analytical flexibility.

```sql
-- Support multiple UOM for same measurement
SELECT
    sales_transaction_id,
    product_id,
    -- Quantity in source UOM
    quantity_sold as quantity_selling_uom,
    selling_unit_of_measure,
    -- Convert to standard UOM (e.g., 'EACH')
    CASE selling_unit_of_measure
        WHEN 'CASE' THEN quantity_sold * 12  -- Case = 12 each
        WHEN 'PALLET' THEN quantity_sold * 144  -- Pallet = 144 each
        ELSE quantity_sold
    END as quantity_each,
    -- Convert to weight UOM (from product master)
    quantity_sold * p.unit_weight_kg as quantity_kilograms,
    -- Convert to volume UOM
    quantity_sold * p.unit_volume_liters as quantity_liters
FROM staging.sales_extract s
LEFT JOIN dim.product p
  ON s.product_id = p.product_natural_key
  AND p.current_row_indicator = 'Y';
```

**Best practice**: Store facts in both:
- Original transaction UOM (for auditability)
- Standard UOM (for aggregation across products)

---

## 6. Currency Conversion

### Multi-Currency Facts

```sql
-- Store amounts in transaction currency AND standard currency
SELECT
    sales_transaction_id,
    transaction_date,
    -- Original transaction currency
    transaction_currency_code,
    revenue_amount as revenue_local_currency,
    -- Convert to standard currency (e.g., USD)
    revenue_amount * er.exchange_rate as revenue_usd,
    -- Include exchange rate used for auditability
    er.exchange_rate as usd_exchange_rate,
    er.effective_date as exchange_rate_date
FROM staging.sales_extract s
LEFT JOIN dim.exchange_rate er
  ON s.transaction_currency_code = er.from_currency_code
  AND er.to_currency_code = 'USD'
  AND s.transaction_date = er.effective_date;
```

### Historical Exchange Rates

```sql
-- Use exchange rate effective on transaction date
-- NOT the exchange rate on ETL processing date
SELECT
    s.transaction_date,
    s.revenue_amount,
    s.transaction_currency_code,
    er.exchange_rate,
    s.revenue_amount * er.exchange_rate as revenue_usd
FROM staging.sales_extract s
LEFT JOIN dim.exchange_rate er
  ON s.transaction_currency_code = er.from_currency_code
  AND s.transaction_date BETWEEN er.effective_start_date
                              AND er.effective_end_date
WHERE er.to_currency_code = 'USD';
```

### Handling Missing Exchange Rates

```sql
-- Fallback logic for missing exchange rates
SELECT
    s.revenue_amount,
    s.transaction_currency_code,
    s.transaction_date,
    -- Try exact date match
    COALESCE(
        er_exact.exchange_rate,
        -- Fallback: Previous week average
        er_week.avg_exchange_rate,
        -- Fallback: Previous month average
        er_month.avg_exchange_rate,
        -- Fallback: Flag for manual review
        NULL
    ) as exchange_rate,
    CASE
        WHEN er_exact.exchange_rate IS NOT NULL THEN 'Exact'
        WHEN er_week.avg_exchange_rate IS NOT NULL THEN 'Week Average'
        WHEN er_month.avg_exchange_rate IS NOT NULL THEN 'Month Average'
        ELSE 'Missing - Manual Review Required'
    END as exchange_rate_source
FROM staging.sales_extract s
LEFT JOIN dim.exchange_rate er_exact
  ON s.transaction_currency_code = er_exact.from_currency_code
  AND s.transaction_date = er_exact.effective_date
LEFT JOIN (
    SELECT from_currency_code,
           AVG(exchange_rate) as avg_exchange_rate,
           DATE_TRUNC('week', effective_date) as week_start
    FROM dim.exchange_rate
    WHERE to_currency_code = 'USD'
    GROUP BY from_currency_code, DATE_TRUNC('week', effective_date)
) er_week
  ON s.transaction_currency_code = er_week.from_currency_code
  AND DATE_TRUNC('week', s.transaction_date) = er_week.week_start
LEFT JOIN (
    SELECT from_currency_code,
           AVG(exchange_rate) as avg_exchange_rate,
           DATE_TRUNC('month', effective_date) as month_start
    FROM dim.exchange_rate
    WHERE to_currency_code = 'USD'
    GROUP BY from_currency_code, DATE_TRUNC('month', effective_date)
) er_month
  ON s.transaction_currency_code = er_month.from_currency_code
  AND DATE_TRUNC('month', s.transaction_date) = er_month.month_start;
```

---

## 7. Allocations and Apportionments

### Allocating Header Amounts to Lines

**Problem**: Shipping charge at order header level, need to allocate to line items.

```sql
-- Calculate line proportion of order total
WITH line_proportions AS (
    SELECT
        order_id,
        line_number,
        line_amount,
        SUM(line_amount) OVER (PARTITION BY order_id) as order_total,
        line_amount / NULLIF(SUM(line_amount) OVER (PARTITION BY order_id), 0) as line_proportion
    FROM staging.order_lines
)
SELECT
    lp.order_id,
    lp.line_number,
    lp.line_amount,
    h.shipping_amount,
    -- Allocate shipping proportionally
    h.shipping_amount * lp.line_proportion as allocated_shipping_amount,
    -- Total line amount including allocated shipping
    lp.line_amount + (h.shipping_amount * lp.line_proportion) as total_line_amount
FROM line_proportions lp
INNER JOIN staging.order_header h
  ON lp.order_id = h.order_id;
```

### Time-Based Allocation

**Problem**: Monthly fee needs to be allocated to daily snapshots.

```sql
-- Allocate monthly amount across days
SELECT
    account_id,
    snapshot_date,
    monthly_fee,
    days_in_month,
    -- Daily allocation
    monthly_fee / days_in_month as daily_allocated_fee
FROM staging.account_monthly_fee f
CROSS JOIN (
    SELECT snapshot_date,
           DATE_PART('day', DATE_TRUNC('month', snapshot_date) + INTERVAL '1 month' - INTERVAL '1 day') as days_in_month
    FROM dim.date
    WHERE calendar_month = :processing_month
) d;
```

---

## 8. Fact Aggregation

### Pre-Aggregating for Performance

```sql
-- Source provides transaction-level detail
-- Business users query daily summaries
-- Pre-aggregate to daily grain

INSERT INTO fact.daily_sales_summary
SELECT
    TRUNC(transaction_timestamp) as transaction_date_key,
    store_id,
    product_id,
    COUNT(*) as transaction_count,
    SUM(quantity) as total_quantity,
    SUM(revenue_amount) as total_revenue,
    SUM(cost_amount) as total_cost,
    AVG(unit_price) as average_unit_price,
    MIN(transaction_timestamp) as first_transaction_time,
    MAX(transaction_timestamp) as last_transaction_time
FROM staging.transaction_detail
GROUP BY TRUNC(transaction_timestamp), store_id, product_id;
```

### Aggregating During Incremental Load

```sql
-- For periodic snapshot: aggregate detail to period level
INSERT INTO fact.monthly_product_sales
SELECT
    DATE_TRUNC('month', transaction_date) as month_start_date,
    product_id,
    customer_segment,
    SUM(quantity) as total_quantity_sold,
    SUM(revenue_amount) as total_revenue,
    COUNT(DISTINCT customer_id) as unique_customer_count,
    COUNT(DISTINCT order_id) as order_count
FROM staging.sales_detail
WHERE transaction_date >= :period_start_date
  AND transaction_date < :period_end_date
GROUP BY DATE_TRUNC('month', transaction_date), product_id, customer_segment;
```

---

## 9. Error Correction Processing

### Identifying Corrections vs. New Transactions

```sql
-- Differentiate corrections from new data
SELECT
    transaction_id,
    transaction_date,
    amount,
    correction_flag,
    original_transaction_id,
    CASE
        WHEN correction_flag = 'Y' THEN 'Correction'
        WHEN original_transaction_id IS NOT NULL THEN 'Reversal'
        ELSE 'New Transaction'
    END as transaction_type
FROM staging.sales_extract;
```

### Processing Corrections (Insert + Delete Pattern)

```sql
-- Step 1: Identify corrections
CREATE TEMP TABLE corrections AS
SELECT
    original_transaction_id,
    corrected_amount,
    correction_reason
FROM staging.sales_extract
WHERE correction_flag = 'Y';

-- Step 2: Insert corrected facts
INSERT INTO fact.sales (
    fact_surrogate_key,
    date_key,
    customer_key,
    product_key,
    amount,
    correction_flag,
    correction_reason
)
SELECT
    fact_key_seq.NEXTVAL,
    f_original.date_key,
    f_original.customer_key,
    f_original.product_key,
    c.corrected_amount,        -- Corrected value
    'Y',
    c.correction_reason
FROM fact.sales f_original
INNER JOIN corrections c
  ON f_original.transaction_id = c.original_transaction_id;

-- Step 3: Delete original incorrect facts
DELETE FROM fact.sales
WHERE transaction_id IN (SELECT original_transaction_id FROM corrections);

-- Alternative Step 3: Mark as superseded instead of deleting
UPDATE fact.sales
SET superseded_flag = 'Y',
    superseded_date = CURRENT_DATE
WHERE transaction_id IN (SELECT original_transaction_id FROM corrections);
```

### Processing Reversals

```sql
-- Reversal: offset original transaction with negative amount
INSERT INTO fact.sales (
    date_key,
    customer_key,
    product_key,
    quantity,
    amount,
    reversal_flag,
    original_transaction_id
)
SELECT
    :reversal_date_key,        -- Date of reversal
    customer_key,
    product_key,
    -1 * quantity,             -- Negative quantity
    -1 * amount,               -- Negative amount
    'Y',
    transaction_id             -- Reference to original
FROM fact.sales
WHERE transaction_id = :transaction_to_reverse;
```

---

## Transformation Pipeline Best Practices

### 1. Layer Transformations

```
Extract → Bronze (raw) → Silver (cleansed) → Gold (business-ready)

Bronze: Exact copy of source (staging.sales_extract)
Silver: Data types fixed, NULLs handled (staging.sales_cleansed)
Gold: Business logic applied, ready for load (staging.sales_fact_ready)
```

### 2. Stage Intermediate Results

```sql
-- Stage after each major transformation
CREATE TABLE staging.sales_bronze AS
SELECT * FROM source_system.sales;

CREATE TABLE staging.sales_silver AS
SELECT
    -- Type conversions, NULL handling
FROM staging.sales_bronze;

CREATE TABLE staging.sales_gold AS
SELECT
    -- Business logic, derived facts
FROM staging.sales_silver;
```

**Why**: Easier debugging, can restart from any stage, audit trail.

### 3. Validate After Each Stage

```sql
-- Validation checkpoints
SELECT 'Bronze' as stage, COUNT(*) as row_count FROM staging.sales_bronze
UNION ALL
SELECT 'Silver', COUNT(*) FROM staging.sales_silver
UNION ALL
SELECT 'Gold', COUNT(*) FROM staging.sales_gold;

-- Expect: Bronze >= Silver >= Gold (filtering occurs)
```

### 4. Maintain Audit Columns

```sql
-- Track transformation lineage
ALTER TABLE staging.sales_gold ADD (
    source_system VARCHAR(50),
    extract_timestamp TIMESTAMP,
    transform_timestamp TIMESTAMP,
    etl_batch_id INTEGER
);

UPDATE staging.sales_gold
SET source_system = 'SAP_ERP',
    extract_timestamp = :extract_time,
    transform_timestamp = CURRENT_TIMESTAMP,
    etl_batch_id = :batch_id;
```

### 5. Handle Division by Zero

```sql
-- Safe division
SELECT
    revenue,
    cost,
    -- Avoid division by zero
    CASE
        WHEN revenue > 0 THEN ((revenue - cost) / revenue) * 100
        ELSE 0
    END as profit_margin_percent,
    -- Alternative: NULLIF
    (revenue - cost) / NULLIF(revenue, 0) as profit_margin_decimal
FROM staging.sales_fact;
```

### 6. Document Business Rules

```sql
-- Embed business rule documentation
SELECT
    order_id,
    -- Business rule #BR-127: Exclude cancelled orders from revenue
    CASE
        WHEN order_status = 'CANCELLED' THEN 0
        ELSE order_amount
    END as revenue_amount,
    -- Business rule #BR-128: Returns are negative revenue
    CASE
        WHEN order_type = 'RETURN' THEN -1 * ABS(order_amount)
        ELSE order_amount
    END as net_revenue_amount
FROM staging.order_fact;
```

---

## Common Transformation Mistakes

1. **Mixing grain**: Combining header and line facts in same table
2. **Losing precision**: Using INTEGER for currency amounts
3. **Hard-coded derivations**: Embedding business logic without documentation
4. **Missing NULL checks**: Calculations fail when operands are NULL
5. **Division by zero**: No safeguards in percentage calculations
6. **Currency confusion**: Mixing local and standard currency without labels
7. **Unvalidated conversions**: Assuming string-to-number conversions always succeed
8. **No audit trail**: Cannot trace transformed value back to source
9. **Destructive updates**: Overwriting facts instead of insert + delete
10. **Over-aggregation**: Aggregating too early, losing required detail

---

## Transformation Testing Checklist

- [ ] Data type conversions handle all source variations
- [ ] NULL values replaced or flagged appropriately
- [ ] Grain is consistent (no mixed header/line data)
- [ ] Duplicates identified and removed
- [ ] Derived facts calculated correctly
- [ ] Multiple UOM conversions verified against product master
- [ ] Currency conversions use correct historical rates
- [ ] Allocations sum to total (no rounding errors)
- [ ] Corrections processed with insert + delete pattern
- [ ] Reversals create negative offsetting facts
- [ ] Transformations documented with business rule IDs
- [ ] Audit columns populated (source, timestamp, batch ID)
- [ ] Staged intermediate results for debugging
- [ ] Row counts validated at each stage
- [ ] Edge cases tested (zero, negative, NULL, extreme values)

---

## Summary

Fact table transformations prepare raw source data for dimensional modeling by:

1. **Standardizing formats** - Data types, dates, strings
2. **Handling missing data** - NULLs, defaults, validation
3. **Ensuring grain** - Filtering, deduplication
4. **Calculating derived facts** - Extensions, margins, durations
5. **Supporting analytics** - Multiple UOM, currency conversion
6. **Distributing amounts** - Allocations, apportionments
7. **Optimizing queries** - Pre-aggregation where appropriate
8. **Correcting errors** - Insert + delete, reversals

Apply these transformations consistently in both historic and incremental loads. Stage intermediate results, validate at each step, and maintain comprehensive audit trails.
