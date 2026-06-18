---
name: incremental-etl-processing
description: Design automated incremental ETL processes for ongoing daily/regular updates after the initial historic load. Use when implementing change data capture, dimension incremental updates, fact table incremental loads, late-arriving data handling, and aggregate maintenance for production data warehouses.
---

# Incremental ETL Processing

## When to Use This Skill

Use this skill after the historic data load is complete and you need to implement ongoing, automated ETL processes that run daily or more frequently. This skill covers:
- Identifying changed source data (change data capture)
- Incremental dimension processing
- Incremental fact table loading
- Handling late-arriving facts and dimensions
- Aggregate table maintenance
- Performance optimization

## Overview

Incremental ETL processing differs fundamentally from historic loading. Historic loads are one-time, can be paused for investigation, and primarily perform bulk inserts. Incremental processing must be fully automated, handle inserts/updates/deletes, and run reliably without human intervention within strict time windows.

The biggest challenge is identifying which source records have changed since the last ETL run (change data capture). Once changed data is identified, similar transformation logic applies as for historic data.

---

## Step 1: Implement Change Data Capture (CDC)

Change data capture isolates source data that has changed since the last update, avoiding full table scans and unnecessary processing.

### CDC Strategy Selection

Choose the CDC approach based on source system capabilities and constraints. Each method has trade-offs:

#### Option 1: Audit Columns (Recommended when available)
**When**: Source system maintains audit columns populated by database triggers

```sql
-- Extract changed records using audit columns
SELECT *
FROM source_table
WHERE last_modified_date >= :last_etl_run_date
```

**Requirements**:
- Audit columns must be reliably populated by database triggers (not application code)
- Must capture deleted records through separate mechanism
- Check for NULL values in audit columns (indicates unreliable implementation)

**Caution**: If DBAs allow back-end scripts to modify data without updating audit columns, this approach is unreliable.

#### Option 2: Full Diff Compare
**When**: Small tables or when audit columns unavailable

**Process**:
1. Keep full snapshot of previous day's data in staging
2. Compare today's extract record-by-record with yesterday's snapshot
3. Use CRC/checksum algorithms to speed comparison

```sql
-- Efficient diff using CRC checksum
-- Step 1: Add checksum to today's data
SELECT *,
       CHECKSUM(*) as row_checksum
FROM source_table_today;

-- Step 2: Compare with yesterday
SELECT t.*
FROM source_table_today t
LEFT JOIN source_table_yesterday y
  ON t.natural_key = y.natural_key
WHERE t.row_checksum <> y.row_checksum
   OR y.natural_key IS NULL;  -- new records
```

**Trade-offs**:
- Guaranteed to find all changes
- Resource-intensive for large tables
- Prefer running comparison on source machine to avoid full data transfer

#### Option 3: Database Log Scraping
**When**: No other options available (last resort)

**Process**:
- Extract transactions from database redo/transaction log
- Parse log entries for relevant table changes

**Risks**:
- DBAs may purge logs during emergencies, losing transactions
- Complex implementation
- Requires deep database-specific knowledge
- May not capture all change types

#### Option 4: Message Queue Monitoring
**When**: Source system uses message-based architecture

**Process**:
- Monitor message queue for transactions
- Capture insert/update/delete messages

**Risks**:
- No replay capability if connection lost
- Requires real-time processing architecture
- Missed messages mean data loss

#### Option 5: Timed Extracts (Use with extreme caution)

```sql
-- PROBLEMATIC: Do not use this approach
SELECT * FROM source_table
WHERE create_date = SYSDATE - 1;
```

**Why this fails**:
- Duplicate loads if ETL process restarts mid-failure
- Missed data if ETL skips a day
- Requires manual intervention to fix data
- No reliable recovery mechanism

**Use only if**: Combined with explicit transaction control and idempotent processing.

### CDC Implementation Best Practices

1. **Perform CDC as early as possible** - Before data transfer to ETL environment
2. **Tag changes with reason codes** - Distinguish error corrections from true updates
3. **Capture all change types** - Inserts, updates, and deletes
4. **Support compliance tracking** - Add metadata for audit trails
5. **Extract deleted records separately** - Query deleted IDs from source before they vanish

---

## Step 2: Incremental Dimension Processing

Process dimension changes daily using automated SCD logic.

### Dimension Extract Strategy

**Best**: Extract only changed rows from source

```sql
-- For sources with reliable audit columns
SELECT * FROM customer_master
WHERE last_update_date >= :last_etl_timestamp;
```

**Fallback**: Extract full dimension snapshot
- Simpler to implement
- Let ETL determine changes
- Acceptable for small/medium dimensions (< 1M rows)
- For large dimensions, use CDC techniques from Step 1

### Identify New and Changed Rows

#### Finding New Dimension Members

```sql
-- Lookup incoming natural keys against existing dimension
-- New members fail the lookup
SELECT s.*
FROM staging.customer_extract s
LEFT JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE d.customer_key IS NULL;
```

**For new members**:
1. Assign new surrogate key
2. Set row_effective_date = processing date (usually yesterday)
3. Set row_end_date = 12/31/9999 (or maximum date value)
4. Set current_row_indicator = 'Y'
5. Insert into dimension table

#### Finding Changed Dimension Members

**Method 1: Column-by-column comparison**

```sql
-- Compare each attribute
SELECT s.*
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE s.customer_name <> d.customer_name
   OR s.customer_address <> d.customer_address
   OR s.customer_city <> d.customer_city
   ... -- compare all columns
```

**Method 2: Hash comparison (faster for wide dimensions)**

```sql
-- Pre-compute hashes for fast comparison
-- Add hash columns to dimension: hash_type1, hash_type2

-- Compute hash on incoming data
SELECT *,
       MD5(CONCAT(customer_name, customer_address, customer_city)) as hash_type1,
       MD5(CONCAT(customer_segment, customer_status)) as hash_type2
FROM staging.customer_extract;

-- Compare hashes
SELECT s.*
FROM staging.customer_extract s
INNER JOIN dim.customer d
  ON s.customer_id = d.customer_natural_key
  AND d.current_row_indicator = 'Y'
WHERE s.hash_type1 <> d.hash_type1
   OR s.hash_type2 <> d.hash_type2;
```

**Method 3: Use DBMS-specific syntax**

```sql
-- Some databases support EXCEPT or MINUS
SELECT * FROM staging.customer_extract
EXCEPT
SELECT * FROM dim.customer WHERE current_row_indicator = 'Y';
```

### Apply SCD Changes

See [scd-processing-logic.md](scd-processing-logic.md) for complete dimension update logic.

**Type 1 (Overwrite)**: Update existing row(s)
```sql
-- Overwrite ALL versions for this natural key
UPDATE dim.customer
SET customer_address = :new_address,
    customer_city = :new_city,
    last_update_date = CURRENT_TIMESTAMP
WHERE customer_natural_key = :natural_key;
```

**Type 2 (Add New Row)**: Create new version
```sql
-- Step 1: Expire current row
UPDATE dim.customer
SET row_end_date = CURRENT_DATE - 1,
    current_row_indicator = 'N'
WHERE customer_natural_key = :natural_key
  AND current_row_indicator = 'Y';

-- Step 2: Insert new row with new surrogate key
INSERT INTO dim.customer (
    customer_key,  -- new surrogate key from sequence
    customer_natural_key,
    customer_name,
    ...
    row_effective_date,
    row_end_date,
    current_row_indicator
) VALUES (
    customer_seq.NEXTVAL,
    :natural_key,
    :new_name,
    ...
    CURRENT_DATE,
    DATE '9999-12-31',
    'Y'
);
```

**Important**: Avoid NULL values in row_end_date. Use maximum date (9999-12-31) for current rows to prevent query errors when filtering on dates.

### Dimension Deletion Handling

**Do NOT delete dimension rows** when source records are deleted.

**Why**: Historical fact data still references these dimensions.

**Instead**: Mark as inactive
```sql
UPDATE dim.customer
SET customer_status = 'Inactive',
    status_reason = 'Deleted from source system',
    last_update_date = CURRENT_TIMESTAMP
WHERE customer_natural_key = :deleted_key;
```

---

## Step 3: Incremental Fact Table Processing

Process new and updated fact rows automatically each ETL cycle.

### Extract and Quality Checkpoint

**Step 3a: Extract changed facts**

Use CDC techniques from Step 1 to identify changed fact records:

```sql
-- Transaction systems: extract new transactions
SELECT * FROM sales_transactions
WHERE transaction_date >= :last_etl_date;

-- For updates/corrections: use audit columns
SELECT * FROM sales_transactions
WHERE last_modified_date >= :last_etl_timestamp;
```

**Step 3b: Archive untransformed data**

Write extracted data to staging before any transformations:

```sql
-- Archive for audit trail
INSERT INTO staging.sales_extract_archive
SELECT *, :etl_run_id, CURRENT_TIMESTAMP
FROM staging.sales_extract;
```

**Purpose**:
- Audit trail for compliance
- Restart point after data quality failures
- Re-processable source if logic changes

**Step 3c: Compute data quality metrics**

```sql
-- Example: Count nulls, out-of-range values
SELECT
    COUNT(*) as total_records,
    SUM(CASE WHEN quantity IS NULL THEN 1 ELSE 0 END) as null_quantity,
    SUM(CASE WHEN quantity < 0 THEN 1 ELSE 0 END) as negative_quantity,
    SUM(CASE WHEN amount < 0 THEN 1 ELSE 0 END) as negative_amount
FROM staging.sales_extract;
```

### Fact Table Transformations

Apply same transformation logic as historic load:
- NULL handling
- Grain validation
- Derived facts calculation
- Multiple units of measure conversion

See [fact-table-transformations.md](fact-table-transformations.md) for details.

### Surrogate Key Pipeline

Replace natural keys with dimension surrogate keys. **All dimensions must be updated first**.

#### Standard Incremental Lookup

```sql
-- Lookup current dimension keys
SELECT
    f.*,
    dc.customer_key,
    dp.product_key,
    dd.date_key,
    ds.store_key
FROM staging.sales_fact_transformed f
LEFT JOIN dim.customer dc
  ON f.customer_id = dc.customer_natural_key
  AND dc.current_row_indicator = 'Y'
LEFT JOIN dim.product dp
  ON f.product_id = dp.product_natural_key
  AND dp.current_row_indicator = 'Y'
INNER JOIN dim.date dd
  ON f.transaction_date = dd.date_actual
LEFT JOIN dim.store ds
  ON f.store_id = ds.store_natural_key
  AND ds.current_row_indicator = 'Y';
```

**Key point**: For Type 2 dimensions, lookup against current_row_indicator = 'Y' or similar filter.

#### Handling Referential Integrity Violations

**Automated error handling required**. Choose appropriate strategy:

**Strategy 1: Create dummy dimension row on-the-fly (Recommended for financial systems)**

```sql
-- If customer_key lookup fails, create placeholder
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    customer_status,
    row_effective_date,
    row_end_date,
    current_row_indicator
) VALUES (
    customer_seq.NEXTVAL,
    :unknown_natural_key,
    'Unknown - To Be Determined',
    'Pending',
    CURRENT_DATE,
    DATE '9999-12-31',
    'Y'
)
RETURNING customer_key INTO :new_surrogate_key;

-- Return this key to pipeline
```

**Strategy 2: Map to single "Unknown" member per dimension**
- Simpler but loses information
- All unknown values lumped together
- Cannot distinguish between different unknown natural keys

**Strategy 3: Throw away error rows**
- Only if missing dimension means irrelevant data
- Dangerous for financial/compliance systems

**Strategy 4: Write to suspense file**
- Requires manual intervention
- Database integrity questionable until resolved
- Not recommended for production

**Strategy 5: Halt the load**
- Unacceptable for production
- Requires manual restart
- Only use during development/testing

### Late-Arriving Facts

Facts that arrive significantly delayed require special handling.

**Scenario**: Transaction from 30 days ago arrives today, but customer dimension has changed via Type 2.

**Solution**: Lookup dimension key as of fact transaction date

```sql
-- Find dimension version effective when fact occurred
SELECT dc.customer_key
FROM dim.customer dc
WHERE dc.customer_natural_key = :fact_customer_id
  AND :fact_transaction_date BETWEEN dc.row_effective_date
                                  AND dc.row_end_date;
```

**Process**:
1. Identify late-arriving facts (transaction_date << load_date)
2. Use BETWEEN logic on effective dates for Type 2 dimensions
3. For Type 1 dimensions, current value is fine
4. May need to adjust semi-additive balances in subsequent snapshots

See [late-arriving-data-handling.md](late-arriving-data-handling.md) for detailed patterns.

### Incremental Load to Fact Table

#### Transaction Fact Tables
- **Primary operation**: INSERT new rows
- Use bulk load techniques if loading into empty partition
- Add audit_key and sequential fact_surrogate_key

```sql
-- Bulk insert with partitioning
INSERT INTO fact.sales PARTITION (p_2024_12)
SELECT
    fact_surrogate_key_seq.NEXTVAL as fact_key,
    date_key,
    customer_key,
    product_key,
    store_key,
    quantity,
    amount,
    cost,
    :audit_dimension_key as audit_key
FROM staging.sales_ready_to_load
WHERE transaction_date BETWEEN DATE '2024-12-01' AND DATE '2024-12-31';
```

**For error corrections**: Insert new row + delete old row (don't update)

```sql
-- Step 1: Insert corrected row
INSERT INTO fact.sales VALUES (...);

-- Step 2: Delete original row using fact surrogate key
DELETE FROM fact.sales
WHERE fact_key = :original_fact_key;
```

#### Periodic Snapshot Fact Tables
- **Primary operation**: INSERT new period + UPDATE current period
- Load monthly snapshots at month-end
- Optionally maintain "hot rolling" current month

```sql
-- Update current month-to-date balances
UPDATE fact.account_balance
SET balance_amount = :new_balance,
    last_update_date = CURRENT_DATE
WHERE account_key = :account_key
  AND month_key = :current_month_key;

-- At month-end: freeze current month, start new month
INSERT INTO fact.account_balance
SELECT account_key,
       :new_month_key,
       0 as balance_amount,  -- will be updated daily
       ...
FROM dim.account
WHERE account_status = 'Active';
```

#### Accumulating Snapshot Fact Tables
- **Primary operation**: UPDATE existing rows as process progresses
- Add new rows only when new process instance begins

```sql
-- Update order row as it progresses through pipeline
UPDATE fact.order_fulfillment
SET ship_date_key = :ship_date_key,
    shipped_quantity = :shipped_qty,
    days_to_ship = :ship_date - order_date,
    fulfillment_status = 'Shipped',
    last_update_date = CURRENT_TIMESTAMP
WHERE order_key = :order_key;
```

**Caution**: Repeated updates cause variable row length growth. Periodically drop and reload for performance.

---

## Step 4: Aggregate Table Maintenance

Update aggregate tables incrementally to avoid full rebuilds.

### Simple Incremental Aggregates

For aggregates without date roll-ups or Type 1 attributes:

```sql
-- Incrementally add today's data to aggregate
INSERT INTO agg.sales_by_product_month
SELECT
    product_key,
    month_key,
    SUM(quantity) as total_quantity,
    SUM(amount) as total_amount
FROM fact.sales
WHERE date_key IN (SELECT date_key FROM dim.date WHERE calendar_date = CURRENT_DATE - 1)
GROUP BY product_key, month_key;
```

### Date Rollup Aggregates

Current month aggregate must be updated/rebuilt:

```sql
-- Drop and rebuild current month aggregate
DELETE FROM agg.sales_by_product_month
WHERE month_key = :current_month_key;

INSERT INTO agg.sales_by_product_month
SELECT
    product_key,
    :current_month_key as month_key,
    SUM(quantity) as total_quantity,
    SUM(amount) as total_amount
FROM fact.sales
WHERE date_key IN (
    SELECT date_key FROM dim.date
    WHERE month_key = :current_month_key
)
GROUP BY product_key;
```

### Type 1 Dimension Changes Impact

**Critical**: Type 1 overwrites invalidate aggregates built on that attribute.

**Process**:
1. Dimension manager notifies fact provider of Type 1 change
2. Fact provider identifies affected aggregates
3. Drop and rebuild affected aggregates from atomic data

```sql
-- If product.category changed (Type 1)
-- Must rebuild all aggregates using product category

DELETE FROM agg.sales_by_category_month;

INSERT INTO agg.sales_by_category_month
SELECT
    p.product_category,
    d.month_key,
    SUM(f.quantity) as total_quantity,
    SUM(f.amount) as total_amount
FROM fact.sales f
INNER JOIN dim.product p ON f.product_key = p.product_key
INNER JOIN dim.date d ON f.date_key = d.date_key
GROUP BY p.product_category, d.month_key;
```

**Best practice**: Version number in conformed dimensions signals aggregate rebuild required.

---

## Step 5: Speed Up Incremental Load Cycle

Techniques to shorten ETL processing window:

### 1. Load More Frequently
- Move from monthly → weekly → daily → intraday
- Each load processes 1/30th the data volume
- Reduces load window requirements

### 2. Daytime Pre-Processing
- Stage and cleanse data throughout the day
- Perform heavy transformations incrementally
- After midnight: final quality checks + surrogate key assignment + load

### 3. Parallel Processing

**Job-level parallelism**:
```
Parallel Stream 1: Customer dimension + Sales facts
Parallel Stream 2: Product dimension + Inventory facts
Parallel Stream 3: Location dimension + Logistics facts
```

**Task-level parallelism**:
- Database parallel index creation
- Parallel partition loads
- ETL tool parallel processing streams

**Caution**: Avoid resource contention. Don't parallelize steps that compete for same I/O or CPU.

### 4. Parallel Structures

**Option A: Partition swapping**
```
1. Load into offline partition
2. Validate data quality
3. Swap offline partition into active fact table (seconds)
4. Drop old partition or archive
```

**Option B: Clustered servers**
```
Server 1: Handles queries
Server 2: Loads new data
Swap roles with minimal downtime (disk attachment swap)
```

### 5. Optimize Physical Database

- **Partition fact tables by date** - Load only into current partition
- **Disable logging during loads** - Transaction logging adds overhead
- **Minimize indexes during load** - Drop non-essential indexes, rebuild after
- **Use bulk load APIs** - Bypass SQL overhead
- **Presort data** - Match clustering index order
- **Increase batch commit size** - Reduce transaction overhead

See the ETL System Architecture Design skill for comprehensive performance optimization.

---

## Common Mistakes to Avoid

1. **Using timed extracts without safeguards** - Leads to duplicate or missing data
2. **Not staging untransformed extracts** - Cannot restart or audit processing
3. **Halting on referential integrity violations** - Requires manual intervention
4. **Deleting dimension rows** - Breaks historical fact relationships
5. **Not handling late-arriving data** - Incorrect dimensional context
6. **Updating facts in place** - Use insert + delete for audit trail
7. **Full aggregate rebuilds** - Use incremental updates when possible
8. **Ignoring Type 1 impact on aggregates** - Results in incorrect aggregate values
9. **Not testing recovery procedures** - Failures will happen in production
10. **Insufficient job logging** - Cannot diagnose failures or optimize performance

---

## Validation Checklist

Before deploying incremental ETL to production:

**CDC Validation**:
- [ ] CDC mechanism tested for inserts, updates, and deletes
- [ ] Deleted records captured correctly
- [ ] No false positives (unchanged records flagged as changed)
- [ ] No false negatives (changed records missed)

**Dimension Processing**:
- [ ] New members inserted with correct surrogate keys
- [ ] Type 1 changes update all rows for natural key
- [ ] Type 2 changes create new row and expire old row
- [ ] Effective dates set correctly
- [ ] Current row indicators maintained accurately

**Fact Processing**:
- [ ] Surrogate key lookups return correct keys
- [ ] RI violations handled automatically without halting
- [ ] Late-arriving facts mapped to correct dimension versions
- [ ] Audit keys populated on all rows
- [ ] Partitioning strategy working correctly

**Aggregates**:
- [ ] Incremental updates produce same results as full rebuild
- [ ] Type 1 changes trigger aggregate rebuilds
- [ ] Current month aggregates reflect latest data

**Performance**:
- [ ] Load completes within processing window
- [ ] No resource contention between parallel jobs
- [ ] Adequate logging without excessive overhead

**Recovery**:
- [ ] Can restart from failure point
- [ ] Can back out partial load
- [ ] Archive data available for reprocessing

---

## Quick Reference: Incremental vs Historic Load

| Aspect | Historic Load | Incremental Load |
|--------|---------------|------------------|
| Frequency | One-time | Daily/continuous |
| Automation | Can be manual | Must be fully automated |
| Primary operation | INSERT | INSERT + UPDATE + DELETE |
| Error handling | Can pause/investigate | Must handle automatically |
| Volume | Full history | Delta only |
| Target table | Usually empty | In production use |
| Fast-load techniques | Always use | Use for partition loads |
| Performance priority | Throughput | Window compliance |

---

## Related Skills

- **ETL System Architecture Design** - Overall ETL subsystem framework
- **Historic Data Loading Strategy** - Initial bulk load procedures
- **Data Profiling and Quality Assessment** - Quality screen design
- **Dimension Table Design Patterns** - SCD implementation details
- **Fact Table Design Patterns** - Transaction/snapshot/accumulating patterns

---

## Summary

Incremental ETL is the production heartbeat of the data warehouse. Success requires:
1. Reliable change data capture mechanism
2. Automated SCD processing for dimensions
3. Robust surrogate key pipeline with automated error handling
4. Proper late-arriving data support
5. Intelligent aggregate maintenance
6. Comprehensive logging and recovery procedures

Start with simple, reliable processes. Add complexity (real-time, complex error handling) only when business requirements justify it.
