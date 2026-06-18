# Partitioning Strategies for Dimensional Models

## Overview

Table partitioning divides large tables into smaller, more manageable pieces called partitions. Each partition can be independently managed, queried, loaded, and maintained while appearing as a single logical table to users and applications.

Partitioning is one of the most important physical design decisions for enterprise data warehouses, dramatically improving performance, manageability, and availability.

## Why Partition?

### Performance Benefits

**1. Partition elimination (partition pruning)**
```sql
-- Query with date filter only scans relevant partitions
SELECT SUM(sales_amount)
FROM fact_sales
WHERE date_key BETWEEN 20240101 AND 20240131;

-- Without partitioning: scans entire table (billions of rows)
-- With monthly partitioning: scans only January 2024 partition
```

**Impact:** 10-100x query performance improvement for date-ranged queries.

**2. Parallel partition operations**
- Multiple partitions processed simultaneously
- Parallel index builds across partitions
- Parallel aggregation queries
- Parallel ETL loads into different partitions

**3. Improved join performance**
- Partition-wise joins when both tables partitioned identically
- Database can join corresponding partitions independently
- Reduces memory and CPU requirements

### Manageability Benefits

**1. Fast data loading**
- Load into specific partition only
- Swap staged partition into production table
- Partition exchange eliminates data movement

**2. Efficient data purging**
```sql
-- Drop oldest partition instead of DELETE (instant vs hours)
ALTER TABLE fact_sales DROP PARTITION p_2020_01;
```

**3. Partition-level operations**
- Rebuild indexes on single partition
- Gather statistics per partition
- Backup/restore individual partitions
- Move partitions between tablespaces

**4. Reduced downtime**
- Maintenance on one partition doesn't lock others
- Users query available partitions during maintenance
- Rolling partition maintenance windows

## When to Partition

### Partition Fact Tables When:
- ✓ Table size > 10 GB or 50 million rows
- ✓ Historical data retention > 1 year
- ✓ Queries frequently filter by date or other attribute
- ✓ Regular data loading (daily, weekly, monthly)
- ✓ Periodic data archival/purging required

### Do NOT Partition When:
- ✗ Table size < 5 GB
- ✗ Queries rarely filter by partitioning column
- ✗ Ad-hoc queries without predictable patterns
- ✗ Database platform doesn't support partitioning

### Dimension Tables
**Generally do NOT partition dimensions:**
- Most dimensions small enough (< 10M rows) that partitioning unnecessary
- Dimension queries typically filter by attributes, not dates
- Partitioning adds complexity with minimal benefit

**Exception:** Very large dimensions (> 10M rows)
- Partition by current_flag (current vs historical)
- Most queries filter for current members only
- Historical partition rarely accessed

## Partitioning Schemes

### 1. Range Partitioning (Most Common)

Partition by value ranges, typically dates.

**Monthly partitioning (most common for fact tables):**
```sql
CREATE TABLE fact_sales (
    sales_key        BIGINT NOT NULL,
    date_key         INTEGER NOT NULL,
    product_key      INTEGER NOT NULL,
    -- ... other columns
)
PARTITION BY RANGE (date_key) (
    PARTITION p_2023_01 VALUES LESS THAN (20230201),
    PARTITION p_2023_02 VALUES LESS THAN (20230301),
    PARTITION p_2023_03 VALUES LESS THAN (20230401),
    PARTITION p_2023_04 VALUES LESS THAN (20230501),
    -- ... continue monthly
    PARTITION p_2024_12 VALUES LESS THAN (20250101),
    PARTITION p_future VALUES LESS THAN (MAXVALUE)
);
```

**Daily partitioning (high-volume transaction systems):**
```sql
PARTITION BY RANGE (date_key) (
    PARTITION p_20240101 VALUES LESS THAN (20240102),
    PARTITION p_20240102 VALUES LESS THAN (20240103),
    PARTITION p_20240103 VALUES LESS THAN (20240104),
    -- ... daily partitions
);
```

**Weekly partitioning:**
```sql
PARTITION BY RANGE (date_key) (
    PARTITION p_2024_w01 VALUES LESS THAN (20240108),
    PARTITION p_2024_w02 VALUES LESS THAN (20240115),
    -- ... weekly partitions
);
```

**Quarterly or yearly partitioning (long retention, lower volumes):**
```sql
PARTITION BY RANGE (date_key) (
    PARTITION p_2023_q1 VALUES LESS THAN (20230401),
    PARTITION p_2023_q2 VALUES LESS THAN (20230701),
    PARTITION p_2023_q3 VALUES LESS THAN (20231001),
    PARTITION p_2023_q4 VALUES LESS THAN (20240101),
    PARTITION p_2024_q1 VALUES LESS THAN (20240401),
    -- ... quarterly partitions
);
```

**Choosing partition granularity:**
| Daily Volume | Retention | Recommended Granularity |
|--------------|-----------|------------------------|
| < 1M rows    | 2-5 years | Monthly |
| 1M - 10M rows | 2-5 years | Weekly or Monthly |
| 10M - 100M rows | 2-5 years | Daily or Weekly |
| > 100M rows  | 2-5 years | Daily |

**Trade-offs:**
- **Finer granularity (daily):** Better partition elimination, easier to manage current data, more partitions to maintain
- **Coarser granularity (monthly):** Fewer partitions to manage, less overhead, less precise partition elimination

### 2. List Partitioning

Partition by discrete value lists (rare for dimensional models).

```sql
-- Partition by region code
CREATE TABLE fact_sales (
    -- columns
)
PARTITION BY LIST (region_code) (
    PARTITION p_east VALUES IN ('NY', 'NJ', 'CT', 'MA'),
    PARTITION p_west VALUES IN ('CA', 'OR', 'WA'),
    PARTITION p_central VALUES IN ('IL', 'TX', 'OH'),
    PARTITION p_other VALUES IN (DEFAULT)
);
```

**Use cases:**
- Geographic separation requirements
- Regulatory data segregation (GDPR, data residency)
- Organizationally separated data
- Low-cardinality dimension with clear boundaries

**Challenges:**
- Partition key must be in fact table (not in dimension)
- Requires denormalization or adding region_code to fact
- Dimension changes (region reassignments) create issues

### 3. Hash Partitioning

Database distributes rows across partitions using hash function.

```sql
-- Hash partition by customer_key
CREATE TABLE fact_sales (
    -- columns
)
PARTITION BY HASH (customer_key)
PARTITIONS 16;
```

**Benefits:**
- Even distribution across partitions
- Good for parallel processing
- No partition key range management

**Drawbacks:**
- No partition elimination for queries
- Cannot drop/archive specific partitions
- Rare in data warehouses (better for OLTP)

**Use in dimensional models:**
- Very limited - typically not recommended
- Use only for extreme parallelism requirements
- Consider database-native sharding instead

### 4. Composite Partitioning

Combine multiple partitioning schemes.

**Range-List Composite:**
```sql
-- Partition by date, subpartition by region
CREATE TABLE fact_sales (
    -- columns
)
PARTITION BY RANGE (date_key)
SUBPARTITION BY LIST (region_code) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201) (
        SUBPARTITION p_2024_01_east VALUES ('NY', 'NJ', 'CT', 'MA'),
        SUBPARTITION p_2024_01_west VALUES ('CA', 'OR', 'WA'),
        SUBPARTITION p_2024_01_other VALUES (DEFAULT)
    ),
    PARTITION p_2024_02 VALUES LESS THAN (20240301) (
        SUBPARTITION p_2024_02_east VALUES ('NY', 'NJ', 'CT', 'MA'),
        SUBPARTITION p_2024_02_west VALUES ('CA', 'OR', 'WA'),
        SUBPARTITION p_2024_02_other VALUES (DEFAULT)
    )
    -- ... continue
);
```

**Range-Hash Composite:**
```sql
-- Partition by date, subpartition by hash for parallelism
PARTITION BY RANGE (date_key)
SUBPARTITION BY HASH (customer_key) SUBPARTITIONS 4 (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    PARTITION p_2024_02 VALUES LESS THAN (20240301)
    -- ... each partition has 4 hash subpartitions
);
```

**Use cases:**
- Very large partitions (> 100M rows per partition)
- Need both date-based management and parallelism
- Regulatory requirements for geographic separation

**Complexity warning:**
- Significantly more complex to manage
- More partitions = more metadata overhead
- Only justified for extremely large fact tables

## Partition Maintenance

### Adding Future Partitions

**Manual approach:**
```sql
-- Add next month's partition before month begins
ALTER TABLE fact_sales
ADD PARTITION p_2024_02 VALUES LESS THAN (20240301);
```

**Automated approach:**
```sql
-- Stored procedure to add partitions
CREATE PROCEDURE add_monthly_partitions(@months_ahead INT)
AS
BEGIN
    DECLARE @partition_date DATE;
    DECLARE @partition_name VARCHAR(20);
    DECLARE @upper_bound DATE;

    SET @partition_date = DATEADD(MONTH, @months_ahead, GETDATE());
    SET @partition_name = 'p_' + FORMAT(@partition_date, 'yyyy_MM');
    SET @upper_bound = DATEADD(MONTH, 1, DATEADD(DAY, 1-DAY(@partition_date), @partition_date));

    -- Generate and execute ALTER TABLE statement
    -- ... implementation details
END;
```

**Best practice:**
- Create partitions 3-6 months in advance
- Automate partition creation
- Monitor partition usage
- Maintain "overflow" partition (MAXVALUE) for safety

### Dropping Old Partitions

**Archive before dropping:**
```sql
-- 1. Export partition to archive storage
CREATE TABLE archive.fact_sales_2020_01
AS SELECT * FROM fact_sales PARTITION (p_2020_01);

-- 2. Verify archive
SELECT COUNT(*) FROM archive.fact_sales_2020_01;

-- 3. Drop partition from production
ALTER TABLE fact_sales DROP PARTITION p_2020_01;
```

**Instant partition drop (Oracle, PostgreSQL):**
```sql
-- Drop partition without DELETE operation
ALTER TABLE fact_sales DROP PARTITION p_2020_01;
-- Instant operation, no undo/redo generated
```

**Retention policies:**
- Document retention requirements (legal, regulatory, business)
- Automate monthly/quarterly archive and purge jobs
- Maintain archive indexes and documentation
- Test restore procedures from archives

### Partition Splitting and Merging

**Split partition (daily to weekly → daily partitions):**
```sql
-- Split a weekly partition into daily partitions
ALTER TABLE fact_sales
SPLIT PARTITION p_2024_w01
AT (20240102, 20240103, 20240104, 20240105, 20240106, 20240107)
INTO (
    PARTITION p_20240101,
    PARTITION p_20240102,
    PARTITION p_20240103,
    PARTITION p_20240104,
    PARTITION p_20240105,
    PARTITION p_20240106,
    PARTITION p_20240107
);
```

**Merge partitions (daily → monthly):**
```sql
-- Merge daily partitions into monthly partition
ALTER TABLE fact_sales
MERGE PARTITIONS p_20240101, p_20240102, ..., p_20240131
INTO PARTITION p_2024_01;
```

**Use cases:**
- Change partition granularity over time
- Recent data daily, older data monthly
- Optimize partition count (too many = overhead)

### Partition Exchange/Swap

Fastest way to load large data volumes into partitioned tables.

**Process:**
```sql
-- 1. Create staging table matching partition structure
CREATE TABLE staging_sales_202401 (
    -- same structure as fact_sales
);

-- 2. Bulk load into staging table (no indexes, constraints)
INSERT INTO staging_sales_202401 SELECT ...;

-- 3. Validate data
-- Check row counts, data quality, etc.

-- 4. Exchange staging table with empty partition (instant)
ALTER TABLE fact_sales
EXCHANGE PARTITION p_2024_01
WITH TABLE staging_sales_202401;

-- 5. Drop staging table
DROP TABLE staging_sales_202401;
```

**Benefits:**
- Instant operation (metadata swap, no data movement)
- Staging table can be indexed/validated independently
- Rollback capability (swap back if issues found)
- Zero downtime for users querying other partitions

**Requirements:**
- Partition must exist and be empty (or truncated first)
- Staging table structure must match exactly
- Staging table data must satisfy partition constraints

## Partitioning by Fact Table Type

### Transaction Fact Tables

**Recommended: Range partition by transaction date**
```sql
PARTITION BY RANGE (date_key) (
    -- Monthly or daily partitions
);
```

**Characteristics:**
- Append-only (no updates to old partitions)
- Queries typically filter by date range
- Easy to archive old partitions

### Periodic Snapshot Fact Tables

**Recommended: Range partition by snapshot date**
```sql
PARTITION BY RANGE (snapshot_date_key) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    PARTITION p_2024_02 VALUES LESS THAN (20240301),
    -- Monthly partitions
);
```

**Characteristics:**
- Current month partition updated daily (rebuild current month)
- Historical partitions read-only after month ends
- Monthly partitions align with snapshot period

### Accumulating Snapshot Fact Tables

**Recommended: Range partition by most recent milestone date**
```sql
-- Partition by last_update_date or final_milestone_date
PARTITION BY RANGE (shipment_date_key) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    -- or by workflow_status
);
```

**Challenges:**
- Rows update as process progresses (may move between partitions)
- Consider partition by workflow status instead of date
- Smaller tables, partitioning may not be needed

**Alternative: Don't partition**
- Accumulating snapshots typically smaller
- Updates complicate partition management
- May not justify partitioning complexity

## Platform-Specific Partitioning

### Oracle
- Most mature partitioning capabilities
- Range, list, hash, composite partitioning
- Partition exchange, interval partitioning (automatic)
- Local and global indexes

### SQL Server
- Range partitioning using partition functions
- Partition switching (exchange)
- Sliding window pattern for automatic partition management

### PostgreSQL
- Declarative partitioning (10+)
- Range, list, hash partitioning
- Partition-wise joins and aggregates

### MySQL
- Range, list, hash, key partitioning
- Limited partition management features
- Partition pruning requires explicit PARTITION clause

### Cloud Platforms

**Redshift:**
- No explicit partitioning (uses dist keys and sort keys)
- Sort keys provide partition-like benefits

**Snowflake:**
- Automatic micro-partitioning (no manual partition management)
- Clustering keys optimize micro-partition pruning

**BigQuery:**
- Partitioned tables by date
- Clustered tables for additional sorting
- Automatic partition management

## Monitoring and Optimization

### Partition Statistics

**Monitor partition sizes:**
```sql
-- Oracle
SELECT
    table_name,
    partition_name,
    num_rows,
    blocks,
    avg_row_len
FROM user_tab_partitions
WHERE table_name = 'FACT_SALES'
ORDER BY partition_name;

-- SQL Server
SELECT
    OBJECT_NAME(p.object_id) AS table_name,
    p.partition_number,
    p.rows,
    au.total_pages
FROM sys.partitions p
JOIN sys.allocation_units au ON p.partition_id = au.container_id
WHERE OBJECT_NAME(p.object_id) = 'fact_sales'
ORDER BY p.partition_number;
```

**Identify partition skew:**
- Partitions should be roughly equal size
- Large skew indicates poor partition key choice
- Extreme skew = hot partition performance problem

### Query Plan Analysis

**Verify partition elimination:**
```sql
-- Check execution plan for partition pruning
EXPLAIN
SELECT SUM(sales_amount)
FROM fact_sales
WHERE date_key BETWEEN 20240101 AND 20240131;

-- Look for: "Partition Range Scan" or "Partitions accessed: p_2024_01"
```

**Diagnose partition access issues:**
- Query should access minimal partitions
- Full partition scan = missing partition key in WHERE clause
- Multiple partition access = verify date filter present

## Partitioning Checklist

**Design phase:**
- [ ] Identify fact tables requiring partitioning (>10GB, >50M rows)
- [ ] Choose partitioning key (almost always date)
- [ ] Select partition granularity (daily, weekly, monthly)
- [ ] Plan for future partition creation (automation)
- [ ] Document retention and archival policies

**Implementation:**
- [ ] Create partitioned table DDL
- [ ] Create local indexes on partitions
- [ ] Test partition exchange/swap procedures
- [ ] Create partition management scripts
- [ ] Document partition naming conventions

**Operations:**
- [ ] Automate future partition creation
- [ ] Schedule partition archival/purge jobs
- [ ] Monitor partition sizes and balance
- [ ] Verify partition elimination in query plans
- [ ] Maintain partition metadata and statistics

## Summary

**Key principles:**
1. **Partition large fact tables by date** - Most common and effective strategy
2. **Choose appropriate granularity** - Balance manageability vs partition count
3. **Automate partition management** - Create, archive, purge automatically
4. **Leverage partition exchange** - Fastest bulk loading technique
5. **Monitor partition effectiveness** - Verify partition elimination in queries

**Benefits that justify partitioning effort:**
- 10-100x query performance improvement through partition elimination
- Instant data archival/purging (drop partition vs DELETE)
- Parallel operations across partitions
- Improved availability during maintenance

Partitioning is one of the highest-value physical design techniques for enterprise data warehouses. Invest time upfront to design, implement, and automate partition management for long-term scalability and performance benefits.
