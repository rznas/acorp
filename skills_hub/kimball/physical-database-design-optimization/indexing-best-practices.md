# Indexing Best Practices for Dimensional Models

## Overview

Indexing is one of the most critical physical design decisions affecting both query performance and ETL efficiency. This guide provides comprehensive indexing strategies for dimension and fact tables.

## Fundamental Indexing Principles

### The Golden Rule
**Index to support query patterns, not to cover all possible queries.**

- Monitor actual query patterns before creating indexes
- Don't speculate on what users "might" query
- Start minimal, add indexes based on observed need
- Balance query speed against ETL load time

### Index Cost-Benefit Analysis

**Benefits:**
- Faster query response time
- Reduced I/O for selective queries
- Enable partition elimination
- Support constraint enforcement

**Costs:**
- Slower INSERT operations
- Additional storage space (20-30% of table size per index)
- Index maintenance overhead during updates
- More memory required for index caching

## Dimension Table Indexing

### Minimum Required Indexes

Every dimension table needs at least these indexes:

**1. Primary Key Index (Surrogate Key)**
```sql
-- Usually created automatically with PRIMARY KEY constraint
ALTER TABLE dim_product ADD PRIMARY KEY (product_key);

-- Or explicit unique index
CREATE UNIQUE INDEX pk_product ON dim_product(product_key);
```

**Purpose:**
- Enforces uniqueness
- Supports joins from fact tables
- BI tools require PK for proper joins

**2. Natural Key Index (Business Key)**
```sql
-- For non-Type 2 dimensions
CREATE UNIQUE INDEX idx_product_natural ON dim_product(product_id);

-- For Type 2 dimensions with history
CREATE UNIQUE INDEX idx_product_natural
ON dim_product(product_id, row_effective_date);
```

**Purpose:**
- Supports surrogate key lookup during ETL
- Enables source system key to warehouse key translation
- Critical for incremental dimension processing

**3. Current Row Index (Type 2 dimensions only)**
```sql
-- Supports filtering to current dimension members
CREATE INDEX idx_product_current
ON dim_product(current_flag, row_expiration_date)
WHERE current_flag = 'Y';

-- Alternative using expiration date
CREATE INDEX idx_product_current
ON dim_product(row_expiration_date)
WHERE row_expiration_date = '9999-12-31';
```

**Purpose:**
- Fast lookup of current dimension members
- Supports surrogate key pipeline for current-only lookups
- Improves dimension browsing queries

### Optional Dimension Indexes

Create additional indexes based on observed query patterns:

**4. Frequently Filtered Attributes**
```sql
-- If users frequently filter by brand
CREATE INDEX idx_product_brand ON dim_product(brand_name);

-- If users frequently filter by category
CREATE INDEX idx_product_category ON dim_product(category_name);

-- Multiple attribute filter
CREATE INDEX idx_product_brand_category
ON dim_product(brand_name, category_name);
```

**When to create:**
- Attribute appears in WHERE clause in >20% of queries
- Attribute has moderate to high cardinality (100+ distinct values)
- Index selectivity >5% (eliminates >95% of rows)

**5. Hierarchy Rollup Attributes**
```sql
-- Support hierarchy navigation
CREATE INDEX idx_product_hierarchy
ON dim_product(category_name, subcategory_name, brand_name);
```

**When to create:**
- Users frequently drill down/up hierarchies
- Reports group by hierarchy levels

**6. Bitmap Indexes (Oracle, Sybase, etc.)**
```sql
-- Low cardinality attributes (< 100 distinct values)
CREATE BITMAP INDEX idx_product_status ON dim_product(product_status);
CREATE BITMAP INDEX idx_product_type ON dim_product(product_type);
CREATE BITMAP INDEX idx_product_flag ON dim_product(discontinued_flag);
```

**When to use:**
- Database platform supports bitmap indexes
- Attribute has low cardinality (< 100 distinct values)
- Queries combine multiple low-cardinality filters (WHERE status='Active' AND type='Premium')
- Read-heavy workload (bitmap indexes slow down updates)

### Dimension Indexing by Size

**Small dimensions (< 10,000 rows):**
- Primary key index only
- Database may scan entire table faster than using indexes
- Additional indexes provide minimal benefit

**Medium dimensions (10,000 - 1,000,000 rows):**
- Primary key, natural key, current row indexes
- 2-3 additional indexes on frequently filtered attributes

**Large dimensions (> 1,000,000 rows):**
- All standard indexes
- Consider partitioning dimension by current_flag
- Index maintenance becomes significant during ETL
- May need to disable/rebuild indexes during bulk updates

## Fact Table Indexing

Fact table indexing is more complex due to:
- Large data volumes (billions of rows)
- Many foreign key columns
- Variety of query patterns

### Clustered Index Strategy

**Most databases allow one clustered index per table:**

**Option 1: Clustered on date key (most common)**
```sql
-- Physical row order by date
CREATE CLUSTERED INDEX idx_sales_date ON fact_sales(date_key);
```

**Benefits:**
- Date almost always in WHERE clause
- Range queries on date very efficient (last 90 days, etc.)
- Supports time-series queries
- Aligns with partitioning strategy

**Option 2: Clustered on surrogate key**
```sql
-- Physical row order by fact table surrogate key
CREATE CLUSTERED INDEX idx_sales_pk ON fact_sales(sales_key);
```

**Benefits:**
- Simplifies ETL restart/recovery
- Fast single-row lookups by surrogate key
- May improve compression (similar values clustered)

**Option 3: Clustered on composite key**
```sql
-- Physical row order by multiple dimensions
CREATE CLUSTERED INDEX idx_sales_composite
ON fact_sales(date_key, product_key, store_key);
```

**Benefits:**
- Optimizes most common query pattern
- Best compression if dimensions correlated
- Supports covering index scenarios

**Recommendation:** Use date_key as clustered index for most fact tables.

### Non-Clustered Compound Indexes

Create compound (multi-column) indexes to support common join patterns:

**Date-first compound indexes (highest priority):**
```sql
-- Date + Product (product analysis over time)
CREATE INDEX idx_sales_date_product ON fact_sales(date_key, product_key);

-- Date + Store (store analysis over time)
CREATE INDEX idx_sales_date_store ON fact_sales(date_key, store_key);

-- Date + Customer (customer analysis over time)
CREATE INDEX idx_sales_date_customer ON fact_sales(date_key, customer_key);
```

**Why date first:**
- Date almost always appears in WHERE clause (to limit query range)
- Database can use date portion even if other dimensions not in query
- Index selectivity improved by starting with date

**Dimension-first compound indexes (lower priority):**
```sql
-- Product + Date (specific product history)
CREATE INDEX idx_sales_product_date ON fact_sales(product_key, date_key);

-- Customer + Date (specific customer history)
CREATE INDEX idx_sales_customer_date ON fact_sales(customer_key, date_key);
```

**When to create:**
- Query pattern frequently filters by specific dimension member
- Looking up history for single customer/product/store common use case

**Three-column compound indexes:**
```sql
-- Date + Product + Store (product-store analysis)
CREATE INDEX idx_sales_date_prod_store
ON fact_sales(date_key, product_key, store_key);
```

**When to create:**
- Very specific common query pattern
- Storage and maintenance cost justified by frequent use

### Covering Indexes

Include frequently queried facts in index to avoid table lookup:

```sql
-- Index includes facts as INCLUDE columns (SQL Server, PostgreSQL)
CREATE INDEX idx_sales_date_product
ON fact_sales(date_key, product_key)
INCLUDE (quantity_sold, sales_amount, cost_amount);
```

**Benefits:**
- Query satisfied entirely from index (no table lookup required)
- Dramatically faster query performance

**Costs:**
- Much larger index size
- Slower index maintenance during loads

**When to use:**
- Specific report queries consuming significant resources
- Small number of facts included
- Query runs frequently (daily dashboards, etc.)

### Bitmap Indexes on Fact Tables

```sql
-- Low-cardinality dimension foreign keys
CREATE BITMAP INDEX idx_sales_promo ON fact_sales(promotion_key);
CREATE BITMAP INDEX idx_sales_channel ON fact_sales(channel_key);
CREATE BITMAP INDEX idx_sales_payment ON fact_sales(payment_type_key);
```

**When to use:**
- Database supports bitmap indexes (Oracle, Sybase)
- Dimension has low cardinality (< 100 distinct values)
- Read-only or infrequent updates (bitmap indexes expensive to maintain)
- Queries commonly filter by these dimensions

**Excellent for:**
- Promotion dimension (typically 10-50 promotions active)
- Payment type (Cash, Credit, Debit, etc.)
- Order status (Pending, Approved, Shipped, etc.)

### Partitioned Fact Table Indexing

**Local indexes (one index per partition):**
```sql
-- Create local indexes on partitioned table
CREATE INDEX idx_sales_product ON fact_sales(product_key) LOCAL;
```

**Benefits:**
- Partition independence (drop partition drops its index)
- Parallel index maintenance
- Faster index rebuild per partition

**Costs:**
- Must search all partition indexes unless date in query

**Global indexes (single index across all partitions):**
```sql
-- Create global index spanning partitions
CREATE INDEX idx_sales_product ON fact_sales(product_key) GLOBAL;
```

**Benefits:**
- Single index lookup regardless of date filter
- Better for queries without date filter

**Costs:**
- Partition operations (drop, swap) invalidate entire global index
- No parallel maintenance

**Recommendation:** Use local indexes for partitioned fact tables, especially if date appears in most queries.

## Index Maintenance During ETL

### Strategy for Bulk Loads

**Historic Load Process:**
```sql
-- 1. Drop indexes before bulk load
DROP INDEX idx_sales_date_product ON fact_sales;
DROP INDEX idx_sales_date_store ON fact_sales;
-- ... drop all non-PK indexes

-- 2. Bulk load data (fast without index maintenance)
-- INSERT INTO fact_sales SELECT ...

-- 3. Rebuild indexes after load
CREATE INDEX idx_sales_date_product ON fact_sales(date_key, product_key);
CREATE INDEX idx_sales_date_store ON fact_sales(date_key, store_key);
-- ... rebuild all indexes
```

**Benefits:**
- 3-10x faster bulk load
- Fresh index statistics after rebuild
- Optimal index structure (no fragmentation)

**Incremental Load Process:**
```sql
-- Option 1: Keep indexes enabled (small daily loads)
-- INSERT INTO fact_sales SELECT ... (indexes maintained automatically)

-- Option 2: Disable indexes, load, rebuild (larger incremental loads)
ALTER INDEX idx_sales_date_product ON fact_sales DISABLE;
-- INSERT INTO fact_sales SELECT ...
ALTER INDEX idx_sales_date_product ON fact_sales REBUILD;
```

**Decision criteria:**
- Incremental load < 1% of table size → Keep indexes enabled
- Incremental load 1-10% of table size → Test both approaches
- Incremental load > 10% of table size → Disable/rebuild indexes

### Partition-Level Index Maintenance

**Best practice for partitioned fact tables:**
```sql
-- 1. Load into empty partition with indexes disabled
ALTER TABLE fact_sales TRUNCATE PARTITION p_2024_01;

-- 2. Bulk load into partition
INSERT INTO fact_sales SELECT * FROM staging_sales WHERE date_key BETWEEN 20240101 AND 20240131;

-- 3. Rebuild indexes on that partition only
ALTER INDEX idx_sales_date_product ON fact_sales REBUILD PARTITION p_2024_01;
ALTER INDEX idx_sales_date_store ON fact_sales REBUILD PARTITION p_2024_01;
```

**Benefits:**
- Index rebuild time proportional to partition size, not entire table
- Other partitions remain queryable during rebuild
- Parallel partition index rebuilds possible

## Index Monitoring and Tuning

### Identify Missing Indexes

**SQL Server:**
```sql
-- Find missing index suggestions
SELECT
    migs.avg_user_impact,
    migs.avg_total_user_cost,
    migs.user_seeks,
    mid.statement,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_details mid
JOIN sys.dm_db_missing_index_groups mig ON mid.index_handle = mig.index_handle
JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
WHERE migs.avg_user_impact > 50
ORDER BY migs.avg_user_impact DESC;
```

**Oracle:**
```sql
-- Use SQL Tuning Advisor
EXEC DBMS_SQLTUNE.CREATE_TUNING_TASK(sql_id => '&sql_id');
```

### Identify Unused Indexes

**SQL Server:**
```sql
-- Find unused indexes (no seeks, scans, or lookups)
SELECT
    OBJECT_NAME(s.object_id) AS table_name,
    i.name AS index_name,
    s.user_seeks,
    s.user_scans,
    s.user_lookups,
    s.user_updates
FROM sys.dm_db_index_usage_stats s
JOIN sys.indexes i ON s.object_id = i.object_id AND s.index_id = i.index_id
WHERE s.database_id = DB_ID()
  AND OBJECTPROPERTY(s.object_id, 'IsUserTable') = 1
  AND s.user_seeks = 0
  AND s.user_scans = 0
  AND s.user_lookups = 0
ORDER BY s.user_updates DESC;
```

**PostgreSQL:**
```sql
-- Find unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Action:** Drop unused indexes that only add ETL overhead.

### Monitor Index Fragmentation

**SQL Server:**
```sql
-- Check index fragmentation
SELECT
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 30
  AND ips.page_count > 1000
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

**Recommendations:**
- Fragmentation < 10%: No action needed
- Fragmentation 10-30%: REORGANIZE index
- Fragmentation > 30%: REBUILD index

## Indexing Strategies by Database Platform

### SQL Server
- Clustered columnstore index for large fact tables
- Nonclustered columnstore as alternative to aggregates
- Include columns for covering indexes
- Filtered indexes for current row queries

### Oracle
- Bitmap indexes for low-cardinality dimensions
- Partitioned indexes (local or global)
- Index-organized tables (IOT) for small dimensions
- Materialized views for aggregates

### PostgreSQL
- Partial indexes (WHERE clause) for current rows
- BRIN indexes for large sequential data
- GIN/GiST indexes for array/JSON attributes

### Cloud Platforms
- **Redshift:** Sortkeys and dist keys instead of traditional indexes
- **Snowflake:** Automatic micro-partitions, no manual indexes
- **BigQuery:** Clustered tables, no indexes needed

## Quick Reference Checklist

**Every dimension table needs:**
- [ ] Primary key (surrogate key) index
- [ ] Natural key index (business key)
- [ ] Current row index (for Type 2 dimensions)

**Every fact table needs:**
- [ ] Clustered index (typically on date_key)
- [ ] 2-4 compound indexes on date + frequently joined dimensions
- [ ] Consider bitmap indexes on low-cardinality dimensions

**Index maintenance:**
- [ ] Drop/disable indexes during bulk loads
- [ ] Rebuild indexes after bulk loads
- [ ] Monitor index usage and fragmentation
- [ ] Remove unused indexes

**Performance monitoring:**
- [ ] Track query execution plans
- [ ] Identify missing index opportunities
- [ ] Find and remove unused indexes
- [ ] Monitor index fragmentation and rebuild as needed

## Summary

Effective indexing requires balancing:
- **Query performance:** Indexes speed up queries
- **ETL performance:** Indexes slow down loads
- **Storage cost:** Each index consumes disk space
- **Maintenance overhead:** Indexes require rebuilding

Start with minimal essential indexes, monitor actual query patterns, and add indexes incrementally based on demonstrated need. The best index strategy emerges from production usage data, not speculation.
