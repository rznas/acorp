# Bulk Loading Optimization Techniques

## Overview

Historic data loads involve massive data volumes—sometimes **thousands of times larger** than daily incremental loads. Bulk loading optimization is critical for completing historic loads within acceptable timeframes.

This guide covers database-specific techniques for maximizing load performance during one-time historic data loads.

## Core Principles

<core_principles>
**1. Optimize for Throughput, Not Availability**
- Loading into offline tables (not yet in production)
- Multi-day load times are tolerable
- Focus: Maximum rows/second throughput

**2. Minimize Database Overhead**
- Disable transaction logging
- Drop/disable indexes during load
- Disable referential integrity constraints
- Reduce checkpointing frequency

**3. Leverage Parallelism**
- Partition tables and load partitions in parallel
- Parallel index creation
- Multiple independent load streams

**4. Prepare Data for Loading**
- Pre-sort by primary index column
- Pre-aggregate or pre-join in staging area
- Clean and transform before bulk load step
</core_principles>

## Bulk Loading Technologies

<bulk_load_technologies>
**Database Native Bulk Loaders:**

**SQL Server:**
- `BULK INSERT` statement
- `bcp` utility (command line)
- SSIS Bulk Insert Task

**Oracle:**
- SQL*Loader (sqlldr)
- External Tables with `PARALLEL` hint
- Oracle Data Pump (impdp)

**PostgreSQL:**
- `COPY` command
- `pg_bulkload` extension

**MySQL:**
- `LOAD DATA INFILE` statement
- `mysqlimport` utility

**Snowflake:**
- `COPY INTO` with staged files
- Snowpipe for continuous loading

**Redshift:**
- `COPY` command from S3/DynamoDB
- Parallel slice loading

**BigQuery:**
- `bq load` command
- Streaming inserts (not ideal for historic bulk)

**General ETL Tools:**
- Informatica: Bulk loader transformations
- SSIS: SQL Server Destination (fast load mode)
- Talend: tBulkExec components
- DataStage: Bulk loader stages
</bulk_load_technologies>

## Pre-Load Optimization

### 1. Table Structure Preparation

<table_preparation>
**Disable Foreign Key Constraints:**

```sql
-- SQL Server
ALTER TABLE fact_sales NOCHECK CONSTRAINT fk_fact_date;
ALTER TABLE fact_sales NOCHECK CONSTRAINT fk_fact_product;
ALTER TABLE fact_sales NOCHECK CONSTRAINT fk_fact_store;
ALTER TABLE fact_sales NOCHECK CONSTRAINT fk_fact_customer;

-- Oracle
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_date;
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_product;

-- PostgreSQL
ALTER TABLE fact_sales DISABLE TRIGGER ALL;
```

**Drop or Disable Indexes:**

```sql
-- SQL Server
ALTER INDEX idx_fact_sales_date ON fact_sales DISABLE;
ALTER INDEX idx_fact_sales_product ON fact_sales DISABLE;

-- Oracle - drop indexes (will rebuild later)
DROP INDEX idx_fact_sales_date;
DROP INDEX idx_fact_sales_product;

-- PostgreSQL
DROP INDEX IF EXISTS idx_fact_sales_date;
```

**Why Drop Indexes?**
- Index maintenance during insert is **extremely expensive**
- Can slow load by 10x or more
- Rebuilding indexes after load is much faster
- Clustered index exception: May need to keep for ordered insert

**Truncate Staging Tables:**

```sql
-- Most efficient way to empty a table
TRUNCATE TABLE staging.fact_sales_load;
-- Faster than DELETE FROM; deallocates space
```

**Partition Preparation:**

```sql
-- Oracle: Ensure target partitions exist
ALTER TABLE fact_sales ADD PARTITION p_202401
    VALUES LESS THAN (TO_DATE('2024-02-01', 'YYYY-MM-DD'));

-- Ensure partitions are empty (for fast load mode)
-- Most bulk loaders require empty partition for optimal performance
```
</table_preparation>

### 2. Data Preparation in Staging Area

<data_staging_prep>
**Pre-Sort Data:**

Sorting data by the primary index column before bulk load dramatically improves performance:

```sql
-- Create sorted staging table
CREATE TABLE staging.fact_sales_sorted AS
SELECT *
FROM staging.fact_sales_transformed
ORDER BY date_key, product_key;  -- Sort by clustered index columns

-- Or write pre-sorted file
SELECT *
FROM staging.fact_sales_transformed
ORDER BY date_key, product_key
INTO OUTFILE '/data/fact_sales_sorted.dat';
```

**Benefits:**
- Faster index creation (data already in index order)
- Reduced fragmentation
- Better compression (similar values adjacent)
- Improved query performance post-load

**Pre-Aggregate or Pre-Join:**

Perform complex transformations in staging area, not during bulk load:

```sql
-- Pre-compute derived facts in staging
UPDATE staging.fact_sales_load
SET net_profit = revenue - cost,
    margin_pct = (revenue - cost) / NULLIF(revenue, 0) * 100;

-- Pre-join to get surrogate keys (if not using lookup stream)
-- Then bulk load contains only surrogate keys
```

**Validate Data Quality:**

Catch errors before bulk load:

```sql
-- Find and fix RI violations before load
SELECT DISTINCT f.product_natural_key
FROM staging.fact_sales_load f
LEFT JOIN dim_product p ON f.product_natural_key = p.product_natural_key
WHERE p.product_key IS NULL;

-- Prevents bulk load failures
```
</data_staging_prep>

### 3. Database Configuration

<database_config>
**Increase Memory Allocation:**

```sql
-- SQL Server: Set memory for bulk operations
EXEC sp_configure 'max server memory', 32768;  -- 32 GB
RECONFIGURE;

-- Oracle: Increase SGA for bulk loads
ALTER SYSTEM SET sga_max_size=32G SCOPE=SPFILE;
ALTER SYSTEM SET pga_aggregate_target=16G SCOPE=SPFILE;
-- Restart instance
```

**Adjust Transaction Log Settings:**

```sql
-- SQL Server: Set recovery model to SIMPLE during load
ALTER DATABASE AdventureWorksDW SET RECOVERY SIMPLE;
-- After load, return to FULL for production

-- Oracle: Disable archive logging (use with caution!)
ALTER DATABASE NOARCHIVELOG;
-- After load, re-enable
ALTER DATABASE ARCHIVELOG;
```

**Set Parallelism:**

```sql
-- SQL Server: Configure max degree of parallelism
EXEC sp_configure 'max degree of parallelism', 8;  -- Use 8 threads
RECONFIGURE;

-- Oracle: Set parallel execution parameters
ALTER SESSION SET parallel_degree_policy=MANUAL;
ALTER SESSION ENABLE PARALLEL DML;
```

**Increase Checkpoint Interval:**

```sql
-- SQL Server: Reduce checkpoint frequency during bulk load
EXEC sp_configure 'recovery interval', 60;  -- 60 minutes
RECONFIGURE;
```
</database_config>

## Bulk Load Execution

### Database-Specific Examples

<bulk_load_examples>
**SQL Server - BULK INSERT:**

```sql
BULK INSERT fact_sales
FROM '/data/fact_sales_sorted.dat'
WITH (
    DATAFILETYPE = 'native',        -- Binary format (fastest)
    TABLOCK,                         -- Table lock (enables fast load)
    ROWS_PER_BATCH = 1000000,       -- Batch size
    ORDER (date_key, product_key),  -- Pre-sorted
    CHECK_CONSTRAINTS,               -- Validate constraints (optional)
    FIRE_TRIGGERS = OFF              -- Don't fire triggers
);
```

**SQL Server - bcp Utility:**

```bash
# Export from staging
bcp "SELECT * FROM staging.fact_sales_sorted ORDER BY date_key" queryout fact_sales.dat -n -S server -d database -T

# Import to target
bcp fact_sales in fact_sales.dat -n -S server -d database -T -b 1000000 -h "TABLOCK,ORDER(date_key)"
```

**Oracle - SQL*Loader:**

```
# Control file: fact_sales.ctl
LOAD DATA
INFILE 'fact_sales.dat'
APPEND INTO TABLE fact_sales
FIELDS TERMINATED BY '|'
TRAILING NULLCOLS
(
    date_key,
    product_key,
    store_key,
    customer_key,
    quantity,
    revenue,
    cost
)
```

```bash
# Execute SQL*Loader with parallel direct path
sqlldr userid=dwuser/password control=fact_sales.ctl direct=true parallel=true rows=1000000
```

**Oracle - External Table:**

```sql
-- Create external table definition
CREATE TABLE fact_sales_external (
    date_key NUMBER,
    product_key NUMBER,
    store_key NUMBER,
    customer_key NUMBER,
    quantity NUMBER,
    revenue NUMBER(10,2),
    cost NUMBER(10,2)
)
ORGANIZATION EXTERNAL (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY data_dir
    ACCESS PARAMETERS (
        RECORDS DELIMITED BY NEWLINE
        FIELDS TERMINATED BY '|'
        MISSING FIELD VALUES ARE NULL
    )
    LOCATION ('fact_sales_part1.dat', 'fact_sales_part2.dat')
)
PARALLEL 8
REJECT LIMIT UNLIMITED;

-- Load from external table
INSERT /*+ APPEND PARALLEL(8) */ INTO fact_sales
SELECT * FROM fact_sales_external;
COMMIT;
```

**PostgreSQL - COPY:**

```sql
-- Load from file
COPY fact_sales (date_key, product_key, store_key, customer_key, quantity, revenue, cost)
FROM '/data/fact_sales_sorted.dat'
WITH (FORMAT CSV, DELIMITER '|', HEADER false);

-- Or from program output
COPY fact_sales FROM PROGRAM 'zcat /data/fact_sales.dat.gz'
WITH (FORMAT CSV, DELIMITER '|');
```

**Snowflake - COPY INTO:**

```sql
-- Stage file first
PUT file:///data/fact_sales.csv @my_stage;

-- Bulk load
COPY INTO fact_sales
FROM @my_stage/fact_sales.csv
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = '|' SKIP_HEADER = 1)
ON_ERROR = 'CONTINUE'  -- Skip error rows, log to table
PURGE = TRUE;          -- Remove file after successful load
```

**Redshift - COPY from S3:**

```sql
COPY fact_sales
FROM 's3://mybucket/fact_sales/'
IAM_ROLE 'arn:aws:iam::account:role/MyRedshiftRole'
FORMAT AS PARQUET
-- or FORMAT AS CSV DELIMITER '|'
COMPUPDATE OFF        -- Skip compression analysis (faster)
STATUPDATE OFF        -- Skip statistics update (faster)
MAXERROR 100;         -- Allow some errors
```
</bulk_load_examples>

### Batch Size Optimization

<batch_size>
**Why Batch Size Matters:**
- Too small: Excessive overhead from commits
- Too large: Memory pressure, long rollback if failure

**Finding Optimal Batch Size:**

```python
import time

batch_sizes = [10000, 50000, 100000, 500000, 1000000]

for batch_size in batch_sizes:
    start = time.time()

    # Load with this batch size
    load_data(batch_size=batch_size)

    elapsed = time.time() - start
    rows_per_second = total_rows / elapsed

    print(f"Batch size: {batch_size}, Rows/sec: {rows_per_second:.0f}")
```

**Typical Optimal Ranges:**
- Small rows (< 100 bytes): 500K - 1M rows per batch
- Medium rows (100-500 bytes): 100K - 500K rows per batch
- Large rows (> 500 bytes): 10K - 100K rows per batch

**Database-Specific Defaults:**
- SQL Server: 0 (entire file in one batch) or 1M rows
- Oracle SQL*Loader: 64 (increase to 100K-1M for large loads)
- PostgreSQL COPY: All rows in single transaction (default)
</batch_size>

## Parallel Loading Techniques

### 1. Partition-Based Parallelism

<partition_parallel>
**Strategy:** Load each partition independently in parallel.

**Prerequisites:**
- Table partitioned (typically by date)
- Partition key present in source data
- Sufficient I/O bandwidth and CPU cores

**Example - Parallel Partition Load:**

```bash
#!/bin/bash
# load_all_partitions.sh

# Function to load one partition
load_partition() {
    partition_date=$1

    echo "Loading partition ${partition_date}..."

    sqlplus dwuser/password <<EOF
    ALTER SESSION ENABLE PARALLEL DML;

    INSERT /*+ APPEND PARALLEL(8) */ INTO fact_sales
    SELECT * FROM staging.fact_sales_sorted
    WHERE date_key >= TO_NUMBER(TO_CHAR(DATE '${partition_date}', 'YYYYMMDD'))
      AND date_key < TO_NUMBER(TO_CHAR(DATE '${partition_date}' + INTERVAL '1' MONTH, 'YYYYMMDD'));

    COMMIT;
    EXIT;
EOF

    echo "Partition ${partition_date} complete."
}

# Export function for parallel execution
export -f load_partition

# Load 12 months in parallel (4 at a time)
parallel -j 4 load_partition ::: \
    2024-01-01 2024-02-01 2024-03-01 2024-04-01 \
    2024-05-01 2024-06-01 2024-07-01 2024-08-01 \
    2024-09-01 2024-10-01 2024-11-01 2024-12-01

echo "All partitions loaded."
```

**Concurrency Guidelines:**
- **CPU-bound**: Parallel jobs ≤ CPU cores
- **I/O-bound**: Parallel jobs ≤ independent I/O channels
- **Memory**: Ensure adequate RAM for all parallel streams
- **Typical**: 2-8 parallel partition loads

**Benefit:** Can reduce 12-hour load to 3 hours with 4-way parallelism
</partition_parallel>

### 2. File-Based Parallelism

<file_parallel>
**Strategy:** Split source data into multiple files, load in parallel.

**File Split:**

```bash
# Split large file into 1GB chunks
split -b 1G -d fact_sales.dat fact_sales_part

# Results in:
# fact_sales_part00
# fact_sales_part01
# fact_sales_part02
# ...
```

**Parallel Load Script:**

```bash
#!/bin/bash

for file in fact_sales_part*; do
    (
        echo "Loading ${file}..."
        sqlcmd -S server -d database -Q "
            BULK INSERT fact_sales
            FROM '/data/${file}'
            WITH (TABLOCK, ROWS_PER_BATCH=1000000)
        "
        echo "${file} complete."
    ) &
done

# Wait for all background jobs
wait

echo "All files loaded."
```
</file_parallel>

### 3. Database-Parallel Features

<db_parallel_features>
**Oracle Parallel DML:**

```sql
ALTER SESSION ENABLE PARALLEL DML;
ALTER SESSION SET parallel_degree_policy=AUTO;

-- Oracle automatically parallelizes this INSERT
INSERT /*+ APPEND PARALLEL */ INTO fact_sales
SELECT * FROM staging.fact_sales_sorted;

COMMIT;
```

**SQL Server Parallel Insert:**

```sql
-- Enable parallelism for bulk insert
INSERT INTO fact_sales WITH (TABLOCK)
SELECT * FROM staging.fact_sales_sorted
OPTION (MAXDOP 8);  -- Use up to 8 threads
```

**PostgreSQL Parallel Workers:**

```sql
-- Set parallel workers for session
SET max_parallel_workers_per_gather = 4;

-- PostgreSQL will parallelize large sequential scans
INSERT INTO fact_sales
SELECT * FROM staging.fact_sales_sorted;
```
</db_parallel_features>

## Post-Load Optimization

### 1. Index Rebuilding

<index_rebuild>
**Sequential Index Creation:**

```sql
-- SQL Server
ALTER INDEX idx_fact_sales_date ON fact_sales REBUILD
    WITH (SORT_IN_TEMPDB = ON, ONLINE = OFF);

ALTER INDEX idx_fact_sales_product ON fact_sales REBUILD
    WITH (SORT_IN_TEMPDB = ON, ONLINE = OFF);
```

**Parallel Index Creation:**

```sql
-- Oracle - parallel index creation
CREATE INDEX idx_fact_sales_date ON fact_sales(date_key)
    PARALLEL 8 NOLOGGING;

CREATE INDEX idx_fact_sales_product ON fact_sales(product_key)
    PARALLEL 8 NOLOGGING;

-- After creation, set to non-parallel for maintenance
ALTER INDEX idx_fact_sales_date NOPARALLEL;
ALTER INDEX idx_fact_sales_product NOPARALLEL;
```

**Concurrent Index Creation (PostgreSQL):**

```bash
# Create multiple indexes in parallel (background processes)
psql -d dwdb -c "CREATE INDEX idx_fact_sales_date ON fact_sales(date_key)" &
psql -d dwdb -c "CREATE INDEX idx_fact_sales_product ON fact_sales(product_key)" &
psql -d dwdb -c "CREATE INDEX idx_fact_sales_store ON fact_sales(store_key)" &

wait
```

**Index Build Performance Tips:**
- **SORT_IN_TEMPDB**: Use tempdb for sort operations (faster if tempdb on fast I/O)
- **NOLOGGING** (Oracle): Disable logging during index creation
- **PARALLEL**: Use parallel degree appropriate for CPU cores
- **FILLFACTOR**: Set to 100 for read-only historic data (no free space needed)

```sql
-- SQL Server - 100% fillfactor for historic partitions
CREATE INDEX idx_fact_sales_date ON fact_sales(date_key)
    WITH (FILLFACTOR = 100);
```
</index_rebuild>

### 2. Statistics Update

<statistics_update>
**Why Update Statistics?**
- Query optimizer needs accurate statistics for execution plans
- Bulk loads bypass normal statistic update mechanisms
- Stale statistics → poor query performance

**SQL Server:**

```sql
-- Update statistics with full scan
UPDATE STATISTICS fact_sales WITH FULLSCAN;

-- Or per-index
UPDATE STATISTICS fact_sales idx_fact_sales_date WITH FULLSCAN;
```

**Oracle:**

```sql
-- Gather table statistics
BEGIN
    DBMS_STATS.GATHER_TABLE_STATS(
        ownname => 'DWUSER',
        tabname => 'FACT_SALES',
        estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
        degree => 8,  -- Parallel
        cascade => TRUE,  -- Include indexes
        method_opt => 'FOR ALL COLUMNS SIZE AUTO'
    );
END;
/
```

**PostgreSQL:**

```sql
-- Analyze table
ANALYZE fact_sales;

-- Or analyze entire database
ANALYZE;
```

**Redshift:**

```sql
-- Analyze table for query optimization
ANALYZE fact_sales;

-- Or analyze specific columns
ANALYZE fact_sales (date_key, product_key, revenue);
```
</statistics_update>

### 3. Enable Constraints

<enable_constraints>
**Re-Enable Foreign Keys:**

```sql
-- SQL Server - enable with check (validates all rows)
ALTER TABLE fact_sales WITH CHECK CHECK CONSTRAINT fk_fact_date;
ALTER TABLE fact_sales WITH CHECK CHECK CONSTRAINT fk_fact_product;

-- If confident in data integrity, enable without check (faster)
ALTER TABLE fact_sales WITH NOCHECK CHECK CONSTRAINT fk_fact_date;
-- Note: constraint enabled but existing data not validated
```

**Oracle:**

```sql
-- Enable constraint with validation
ALTER TABLE fact_sales ENABLE CONSTRAINT fk_fact_date;

-- Enable without validation (if pre-validated)
ALTER TABLE fact_sales ENABLE NOVALIDATE CONSTRAINT fk_fact_date;
```

**Best Practice:**
- **Pre-validate** RI in staging area before bulk load
- Enable constraints **without validation** after load (faster)
- Trust pre-load validation rather than post-load check
</enable_constraints>

### 4. Partition Maintenance

<partition_maintenance>
**Merge Daily Partitions to Monthly (if appropriate):**

```sql
-- Oracle - merge daily partitions into monthly
ALTER TABLE fact_sales MERGE PARTITIONS p_20240101 TO p_20240131
    INTO PARTITION p_202401;
```

**Compress Historic Partitions:**

```sql
-- SQL Server - enable page compression on old partitions
ALTER TABLE fact_sales REBUILD PARTITION = 1
    WITH (DATA_COMPRESSION = PAGE);

-- Oracle - enable compression
ALTER TABLE fact_sales MODIFY PARTITION p_202401 COMPRESS FOR QUERY HIGH;
```

**Benefits:**
- Reduced storage footprint
- Improved query performance (fewer I/Os)
- Lower backup/restore times
</partition_maintenance>

## Performance Monitoring

<performance_monitoring>
**Key Metrics to Track:**

**1. Throughput (Rows/Second):**

```python
import time

total_rows = 100000000  # 100M rows
start_time = time.time()

# ... bulk load process ...

elapsed = time.time() - start_time
throughput = total_rows / elapsed

print(f"Loaded {total_rows:,} rows in {elapsed:.1f} seconds")
print(f"Throughput: {throughput:,.0f} rows/second")
```

**2. I/O Wait Times:**

```sql
-- SQL Server - check I/O waits
SELECT wait_type, wait_time_ms, waiting_tasks_count
FROM sys.dm_os_wait_stats
WHERE wait_type LIKE 'PAGEIOLATCH%'
ORDER BY wait_time_ms DESC;
```

**3. CPU Utilization:**

```bash
# Linux - monitor CPU during load
vmstat 5

# Check for CPU saturation (high %user or %system)
```

**4. Disk I/O:**

```bash
# Linux - monitor disk I/O
iostat -x 5

# Check for I/O saturation (high %util, await)
```

**5. Memory Pressure:**

```sql
-- SQL Server - check memory grants
SELECT
    session_id,
    requested_memory_kb,
    granted_memory_kb,
    used_memory_kb
FROM sys.dm_exec_query_memory_grants
WHERE session_id = @@SPID;
```

**Bottleneck Identification:**

| Symptom | Likely Bottleneck | Solution |
|---------|------------------|----------|
| High CPU, low I/O wait | CPU bound | Increase parallelism, optimize transformations |
| High I/O wait, low CPU | I/O bound | Faster storage, partition across disks, reduce logging |
| Memory grants waiting | Memory pressure | Reduce batch size, add RAM, reduce parallelism |
| Network wait times | Network throughput | Use compression, stage files locally, parallel streams |
</performance_monitoring>

## Troubleshooting Common Issues

<troubleshooting>
**Problem: Bulk Load Slower Than Expected**

**Check:**
1. Are indexes disabled? If not, disable and retry.
2. Is data pre-sorted? Unsorted data slows index creation.
3. Is transaction logging minimal? Check recovery model/logging settings.
4. Is parallelism enabled? Verify parallel execution is occurring.

**Problem: Load Fails with Out of Memory**

**Solutions:**
- Reduce batch size
- Reduce degree of parallelism
- Increase database memory allocation
- Split load into smaller chunks

**Problem: Referential Integrity Violations After Load**

**Prevention:**
- Pre-validate RI in staging area before bulk load
- Use surrogate key pipeline that guarantees RI
- Never skip dimension loading before facts

**Problem: Index Creation Takes Too Long**

**Solutions:**
- Ensure data is pre-sorted by index key
- Use parallel index creation
- Use NOLOGGING (Oracle) or SORT_IN_TEMPDB (SQL Server)
- Consider creating only critical indexes initially

**Problem: Statistics Not Updated After Load**

**Symptoms:**
- Queries using poor execution plans
- Table scans instead of index seeks

**Solution:**
```sql
UPDATE STATISTICS table_name WITH FULLSCAN;
```
</troubleshooting>

## Performance Benchmarks

<benchmarks>
**Realistic Expectations:**

| Row Size | Rows | Data Size | Expected Throughput | Estimated Time |
|----------|------|-----------|-------------------|----------------|
| 40 bytes | 100M | 4 GB | 500K rows/sec | 3.3 minutes |
| 40 bytes | 1B | 40 GB | 500K rows/sec | 33 minutes |
| 200 bytes | 100M | 20 GB | 200K rows/sec | 8.3 minutes |
| 200 bytes | 1B | 200 GB | 100K rows/sec | 2.8 hours |

**Factors Affecting Performance:**
- Disk I/O speed (SSD vs HDD: 10-100x difference)
- Number of indexes (each index adds overhead)
- Network latency (local vs remote load)
- Degree of parallelism
- Data compression
- Table partitioning

**Real-World Example:**

```
Project: Retail Sales Data Warehouse
Fact Table: 3 billion rows, 120 bytes/row = 360 GB
Configuration:
    - Oracle 19c
    - 16-core server
    - SSD storage
    - Partitioned by month (36 partitions)

Approach: Parallel partition load (8 partitions at a time)

Results:
    - Per-partition load: 12 minutes average
    - Total elapsed (8-way parallel): 54 minutes
    - Throughput: 925K rows/second
    - Index creation: 90 minutes (parallel)
    - Total load time: 2.4 hours
```
</benchmarks>

## Summary

Bulk loading optimization is **critical for historic data loads**:

1. **Pre-load preparation**: Disable constraints and indexes
2. **Use native bulk loaders**: Much faster than standard INSERT
3. **Pre-sort data**: Dramatically speeds index creation
4. **Leverage parallelism**: Partition-based or file-based
5. **Optimize batch size**: Test to find optimal throughput
6. **Post-load tasks**: Rebuild indexes, update statistics, enable constraints
7. **Monitor performance**: Track throughput and identify bottlenecks

With proper optimization, you can achieve **hundreds of thousands to millions of rows per second** throughput, turning what could be week-long loads into hours.

**Key Takeaway:** The difference between a poorly optimized and well-optimized bulk load can be **100x or more** in performance.
