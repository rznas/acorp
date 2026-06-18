# Real-Time Partition Design Patterns

## Overview

This reference provides detailed design patterns for implementing real-time partitions in dimensional data warehouses. Real-time partitions extend static fact tables to include continuously loaded recent data while maintaining query performance and dimensional design integrity.

---

## Core Partition Architecture

### Physical Partition vs Logical Union

<partition_types>

**Physical Database Partition (Preferred)**

The fact table is partitioned by date, with the most recent partition designated as the real-time partition:

```sql
CREATE TABLE sales_fact (
    date_key INTEGER NOT NULL,
    product_key INTEGER NOT NULL,
    store_key INTEGER NOT NULL,
    customer_key INTEGER,
    -- Additional dimension keys
    sales_amount DECIMAL(10,2),
    sales_quantity INTEGER
    -- Additional facts
)
PARTITION BY RANGE (date_key) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    PARTITION p_2024_02 VALUES LESS THAN (20240301),
    -- Historical partitions...
    PARTITION p_2024_12 VALUES LESS THAN (20250101),
    PARTITION p_realtime VALUES LESS THAN (MAXVALUE)  -- Real-time partition
);
```

**Advantages**:
- Single table, seamless queries
- Partition pruning automatically applied
- Partition swapping for efficient daily migration
- Different indexing strategies per partition possible
- Database manages partition boundaries

**Logical Union View (Alternative)**

Separate physical tables with a UNION ALL view:

```sql
CREATE TABLE sales_fact_static (
    -- Same structure as above
);

CREATE TABLE sales_fact_realtime (
    -- Identical structure
);

CREATE VIEW sales_fact_complete AS
SELECT * FROM sales_fact_static
UNION ALL
SELECT * FROM sales_fact_realtime;
```

**Advantages**:
- Simpler to implement on databases without partitioning
- Complete isolation between static and real-time
- Easy to apply different table properties (clustering, distribution)

**Disadvantages**:
- BI tools must query view, not base table
- No automatic partition pruning
- UNION ALL overhead on every query

</partition_types>

### Indexing Strategy

<indexing_strategy>

**Static Partitions (Historical Data)**

Heavily indexed for query performance:

```sql
-- Primary key
CREATE UNIQUE INDEX sales_fact_pk ON sales_fact_static
    (date_key, product_key, store_key, transaction_number);

-- Foreign keys for dimension joins
CREATE INDEX sales_fact_product_fk ON sales_fact_static (product_key);
CREATE INDEX sales_fact_store_fk ON sales_fact_static (store_key);
CREATE INDEX sales_fact_customer_fk ON sales_fact_static (customer_key);

-- Common query patterns
CREATE INDEX sales_fact_date_product ON sales_fact_static (date_key, product_key);
```

**Real-Time Partition (Today's Data)**

Minimally indexed or unindexed:

```sql
-- Option 1: Completely unindexed
-- No indexes at all; rely on in-memory scan

-- Option 2: Minimal indexing (only if required by RDBMS)
CREATE INDEX sales_rt_date ON sales_fact_realtime (date_key);
```

**Rationale**:
- Indexes slow down INSERT/UPDATE operations
- Real-time partition size small enough for in-memory scan
- Partition pinned in memory (see below)
- Query latency from scan acceptable given small size

</indexing_strategy>

### Memory Pinning

<memory_pinning>

Pin real-time partition in database buffer cache to enable fast queries despite lack of indexes:

**Oracle**:
```sql
ALTER TABLE sales_fact_realtime STORAGE (BUFFER_POOL KEEP);
```

**SQL Server**:
```sql
-- Use in-memory OLTP (Hekaton)
CREATE TABLE sales_fact_realtime (
    -- columns
) WITH (MEMORY_OPTIMIZED = ON, DURABILITY = SCHEMA_AND_DATA);
```

**PostgreSQL**:
```sql
-- Use pg_prewarm extension
SELECT pg_prewarm('sales_fact_realtime');
```

**Sizing considerations**:
- 10M transactions/day × 40 bytes/row = 400 MB
- 15M accounts × 32 bytes/row = 480 MB
- Keep partition size under 1-2 GB for memory pinning feasibility

</memory_pinning>

---

## Transaction Fact Table Patterns

### Pattern 1: Retail Sales Transaction

**Grain**: One row per product on a POS transaction

**Static fact table**:
- Contains all transactions from beginning of history to yesterday
- Heavily indexed
- Aggregates built for common queries
- Size: Large (years of data)

**Real-time partition**:
- Contains only today's transactions
- Unindexed or minimally indexed
- Pinned in memory
- Size: Manageable (single day)

**Daily migration process**:

```sql
-- Step 1: Append yesterday's real-time data to static table
INSERT INTO sales_fact_static
SELECT * FROM sales_fact_realtime
WHERE date_key = YESTERDAY;

-- Step 2: Build indexes on yesterday's data (if using range partitioning)
ALTER INDEX sales_fact_product_fk REBUILD PARTITION p_yesterday;

-- Step 3: Delete yesterday's data from real-time partition
DELETE FROM sales_fact_realtime
WHERE date_key = YESTERDAY;

-- Step 4: Update aggregates to include yesterday
INSERT INTO sales_monthly_agg
SELECT date_key, product_key, SUM(sales_amount), SUM(sales_quantity)
FROM sales_fact_static
WHERE date_key = YESTERDAY
GROUP BY date_key, product_key;
```

**Using physical partitioning (preferred)**:

```sql
-- Partition exchange: instant migration
ALTER TABLE sales_fact
EXCHANGE PARTITION p_realtime
WITH TABLE sales_fact_yesterday_staging;

-- Create new empty partition for today
ALTER TABLE sales_fact
ADD PARTITION p_realtime VALUES LESS THAN (MAXVALUE);
```

### Pattern 2: Web Clickstream Events

**Grain**: One row per page view or event

**Characteristics**:
- Very high volume (millions per hour)
- Continuous arrival throughout day
- Users query recent sessions (today + last 7 days)

**Real-time partition design**:

```sql
CREATE TABLE clickstream_fact_realtime (
    event_timestamp TIMESTAMP NOT NULL,
    date_key INTEGER NOT NULL,
    time_key INTEGER NOT NULL,
    session_key BIGINT NOT NULL,
    page_key INTEGER,
    event_key INTEGER,
    visitor_key BIGINT,
    referral_key INTEGER,
    duration_seconds INTEGER,
    bytes_transferred INTEGER
)
PARTITION BY RANGE (event_timestamp) (
    PARTITION p_hour_00 VALUES LESS THAN ('2024-12-15 01:00:00'),
    PARTITION p_hour_01 VALUES LESS THAN ('2024-12-15 02:00:00'),
    -- Partitions for each hour of today
    PARTITION p_current VALUES LESS THAN (MAXVALUE)
);
```

**Hourly migration**:
- Migrate completed hours to static table
- Keep only current hour in real-time partition
- Reduces real-time partition size, improves load and query performance

### Pattern 3: Call Center Transactions

**Grain**: One row per customer call

**Special requirements**:
- Calls may span midnight (start before midnight, end after)
- Need to query both in-progress and completed calls

**Design approach**:

```sql
-- Static fact table: Completed calls only
CREATE TABLE call_fact_static (
    call_key BIGINT PRIMARY KEY,
    start_date_key INTEGER,
    end_date_key INTEGER,
    customer_key INTEGER,
    agent_key INTEGER,
    -- Facts
    call_duration_seconds INTEGER,
    calls_transferred INTEGER
);

-- Real-time partition: In-progress + today's completed calls
CREATE TABLE call_fact_realtime (
    call_key BIGINT PRIMARY KEY,
    start_date_key INTEGER,
    end_date_key INTEGER,  -- NULL for in-progress
    customer_key INTEGER,
    agent_key INTEGER,
    call_duration_seconds INTEGER,  -- Updated as call progresses
    calls_transferred INTEGER,
    is_complete BOOLEAN DEFAULT FALSE
);
```

**Daily migration**:
```sql
-- Move completed calls to static table
INSERT INTO call_fact_static
SELECT call_key, start_date_key, end_date_key, customer_key, agent_key,
       call_duration_seconds, calls_transferred
FROM call_fact_realtime
WHERE is_complete = TRUE
  AND end_date_key < TODAY;

-- Delete migrated calls
DELETE FROM call_fact_realtime
WHERE is_complete = TRUE
  AND end_date_key < TODAY;

-- In-progress calls remain for completion
```

---

## Periodic Snapshot Fact Table Patterns

### Pattern 1: Account Balance Snapshot

**Grain**: One row per account per month

**Static fact table**:
- Contains all completed months from history
- One row per account per month
- Rows never updated after month closes

**Real-time partition**:
- Contains current developing month only
- Same rows updated continuously as balances change
- At month end, migrated to static table

**Design**:

```sql
CREATE TABLE account_balance_fact (
    month_key INTEGER NOT NULL,
    account_key BIGINT NOT NULL,
    -- Dimension keys
    customer_key INTEGER,
    product_key INTEGER,
    branch_key INTEGER,
    -- Semi-additive facts (balances)
    beginning_balance DECIMAL(15,2),
    ending_balance DECIMAL(15,2),
    average_daily_balance DECIMAL(15,2),
    -- Additive facts (flows)
    deposits_amount DECIMAL(15,2),
    withdrawals_amount DECIMAL(15,2),
    fees_amount DECIMAL(15,2),
    PRIMARY KEY (month_key, account_key)
)
PARTITION BY RANGE (month_key) (
    PARTITION p_2024_01 VALUES LESS THAN (202402),
    PARTITION p_2024_02 VALUES LESS THAN (202403),
    -- Historical partitions
    PARTITION p_realtime VALUES LESS THAN (MAXVALUE)
);
```

**Continuous update process** (real-time partition):

```sql
-- Update existing account row as new transactions arrive
UPDATE account_balance_fact
SET ending_balance = ending_balance + :transaction_amount,
    average_daily_balance = /* recalculated */,
    deposits_amount = deposits_amount + :deposit_amount
WHERE month_key = CURRENT_MONTH
  AND account_key = :account_key;

-- Insert new account if first transaction this month
INSERT INTO account_balance_fact (month_key, account_key, ...)
VALUES (CURRENT_MONTH, :account_key, ...)
ON CONFLICT (month_key, account_key) DO UPDATE
SET ending_balance = ending_balance + :transaction_amount,
    -- update other facts
```

**End-of-month migration**:

```sql
-- On last day of month, finalize calculations
UPDATE account_balance_fact
SET average_daily_balance = total_daily_balances / days_in_month
WHERE month_key = CURRENT_MONTH;

-- Partition becomes static (no further updates)
-- New real-time partition created for next month automatically
```

### Pattern 2: Inventory Snapshot

**Grain**: One row per product per warehouse per day

**Design**:

```sql
CREATE TABLE inventory_fact (
    date_key INTEGER NOT NULL,
    product_key INTEGER NOT NULL,
    warehouse_key INTEGER NOT NULL,
    -- Semi-additive facts
    quantity_on_hand INTEGER,
    quantity_allocated INTEGER,
    quantity_available INTEGER,
    -- Additive facts
    quantity_received INTEGER,
    quantity_shipped INTEGER,
    PRIMARY KEY (date_key, product_key, warehouse_key)
)
PARTITION BY RANGE (date_key) (
    PARTITION p_2024_12_01 VALUES LESS THAN (20241202),
    -- Daily partitions
    PARTITION p_realtime VALUES LESS THAN (MAXVALUE)
);
```

**Intra-day update pattern**:

```sql
-- Update inventory snapshot as transactions occur
UPDATE inventory_fact
SET quantity_on_hand = quantity_on_hand - :shipped_qty,
    quantity_shipped = quantity_shipped + :shipped_qty,
    quantity_available = quantity_on_hand - quantity_allocated
WHERE date_key = TODAY
  AND product_key = :product_key
  AND warehouse_key = :warehouse_key;
```

**End-of-day finalization**:

```sql
-- No migration needed—today's partition becomes static
-- New partition created for tomorrow
-- Yesterday's partition can now be indexed
```

---

## Accumulating Snapshot: Not Recommended

### Why Accumulating Snapshots Don't Use Real-Time Partitions

**Accumulating snapshot characteristics**:
- One row per process instance (order, claim, project)
- Multiple date columns (order_date, ship_date, delivery_date)
- Rows updated as process progresses through milestones
- No single partition key (rows span many dates)

**Problems with partitioning**:
1. **No clear partition boundary**: Which partition contains an order placed Dec 1 but shipped Dec 15?
2. **Updates cross partitions**: When ship_date is updated, which partition?
3. **Cannot migrate daily**: Orders remain open for days/weeks/months

**Recommended approach**:
- Use transaction fact table for real-time events (order placed, shipment created)
- Build accumulating snapshot in batch process (nightly or weekly)
- Query transaction facts for real-time, accumulating snapshot for lifecycle analysis

---

## Query Performance Optimization

### Union View Query Patterns

When using logical union (separate static and real-time tables):

```sql
-- BAD: Forces scan of entire history
SELECT product_key, SUM(sales_amount)
FROM sales_fact_complete
GROUP BY product_key;

-- GOOD: Explicitly partition-prune
SELECT product_key, SUM(sales_amount)
FROM (
    SELECT product_key, sales_amount
    FROM sales_fact_static
    WHERE date_key >= 20241201  -- Last 30 days
    UNION ALL
    SELECT product_key, sales_amount
    FROM sales_fact_realtime
) combined
GROUP BY product_key;
```

### Materialized Views Combining Static + Real-Time

```sql
-- Materialized aggregate refreshed nightly
CREATE MATERIALIZED VIEW sales_monthly_agg AS
SELECT TRUNC(date_key, 'MM') AS month_key,
       product_key,
       SUM(sales_amount) AS total_sales
FROM sales_fact_static
GROUP BY TRUNC(date_key, 'MM'), product_key;

-- Real-time additive query
SELECT month_key, product_key,
       total_sales + COALESCE(today_sales, 0) AS total_sales_current
FROM sales_monthly_agg
LEFT JOIN (
    SELECT TRUNC(date_key, 'MM') AS month_key,
           product_key,
           SUM(sales_amount) AS today_sales
    FROM sales_fact_realtime
    GROUP BY TRUNC(date_key, 'MM'), product_key
) rt USING (month_key, product_key);
```

### BI Tool Configuration

**Configure BI tool to query correctly**:

```sql
-- Define "Complete Sales" subject area
-- Physical SQL generated by BI tool:
SELECT d.date, p.product_name, SUM(f.sales_amount)
FROM sales_fact_complete f  -- Union view or partitioned table
JOIN date_dim d ON f.date_key = d.date_key
JOIN product_dim p ON f.product_key = p.product_key
WHERE d.date >= ADD_MONTHS(CURRENT_DATE, -3)  -- Auto partition prune
GROUP BY d.date, p.product_name;
```

---

## Platform-Specific Implementations

### Oracle

```sql
-- Interval partitioning (auto-creates daily partitions)
CREATE TABLE sales_fact (
    date_key INTEGER,
    -- columns
)
PARTITION BY RANGE (date_key)
INTERVAL (1)  -- New partition per day
(
    PARTITION p_initial VALUES LESS THAN (20240101)
);

-- Real-time partition: Most recent partition is always real-time
-- Alter indexing on partition as it ages:
ALTER INDEX sales_fact_pk REBUILD PARTITION p_20241214 NOLOGGING PARALLEL 4;
```

### SQL Server

```sql
-- Partitioned table with sliding window
CREATE PARTITION FUNCTION pf_sales_daily (INT)
AS RANGE RIGHT FOR VALUES (20240101, 20240102, ..., 20241214);

CREATE PARTITION SCHEME ps_sales_daily
AS PARTITION pf_sales_daily ALL TO ([PRIMARY]);

CREATE TABLE sales_fact (
    date_key INT,
    -- columns
) ON ps_sales_daily(date_key);

-- Real-time partition: Use in-memory table
CREATE TABLE sales_fact_realtime (
    -- columns
) WITH (MEMORY_OPTIMIZED = ON);
```

### PostgreSQL

```sql
-- Declarative partitioning (10+)
CREATE TABLE sales_fact (
    date_key INTEGER,
    -- columns
) PARTITION BY RANGE (date_key);

-- Create partitions
CREATE TABLE sales_fact_202401 PARTITION OF sales_fact
    FOR VALUES FROM (20240101) TO (20240201);

-- Real-time partition (unbounded)
CREATE TABLE sales_fact_realtime PARTITION OF sales_fact
    FOR VALUES FROM (20241215) TO (UNBOUNDED);
```

### Amazon Redshift

```sql
-- Distribution and sort keys
CREATE TABLE sales_fact_static (
    date_key INTEGER,
    product_key INTEGER,
    -- columns
)
DISTKEY(product_key)
SORTKEY(date_key, product_key);

CREATE TABLE sales_fact_realtime (
    -- Same structure
)
DISTKEY(product_key)
SORTKEY(date_key);  -- Lighter sort key for real-time
```

### Snowflake

```sql
-- Automatic clustering
CREATE TABLE sales_fact (
    date_key INTEGER,
    -- columns
)
CLUSTER BY (date_key);

-- Partition by date using search optimization
ALTER TABLE sales_fact ADD SEARCH OPTIMIZATION ON EQUALITY(date_key);
```

---

## Monitoring and Maintenance

### Real-Time Partition Size Monitoring

```sql
-- Monitor real-time partition growth
SELECT partition_name,
       num_rows,
       blocks * block_size / 1024 / 1024 AS size_mb,
       last_analyzed
FROM user_tab_partitions
WHERE table_name = 'SALES_FACT'
  AND partition_name = 'P_REALTIME';

-- Alert if partition exceeds memory budget
```

### Load Performance Monitoring

```sql
-- Track insert rate (rows/second)
SELECT TRUNC(insert_timestamp, 'MI') AS minute,
       COUNT(*) AS rows_inserted
FROM sales_fact_realtime
WHERE insert_timestamp >= CURRENT_DATE
GROUP BY TRUNC(insert_timestamp, 'MI')
ORDER BY minute DESC;
```

### Query Performance Monitoring

```sql
-- Identify queries scanning both static + real-time
SELECT sql_text, executions, elapsed_time, buffer_gets
FROM v$sql
WHERE sql_text LIKE '%sales_fact%'
  AND executions > 10
ORDER BY elapsed_time DESC;
```

---

## Summary

Real-time partitions enable low-latency dimensional data warehousing by separating continuously loaded recent data from static historical data. Transaction fact tables use daily partitions with trickle-feed inserts. Periodic snapshot fact tables use rolling period partitions with continuous updates. Minimal indexing and memory pinning enable simultaneous fast loading and querying. Physical database partitioning is preferred over logical union views. Daily migration moves real-time partition data to static partitions where full indexing and aggregation occur.
