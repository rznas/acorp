---
name: physical-database-design-optimization
description: Translate logical dimensional models into optimized physical database structures with partitioning, indexing, aggregates, and performance tuning. Use after dimensional models are designed, before database implementation.
---

# Physical Database Design and Optimization

## When to Use This Skill

**Lifecycle Phase:** Physical Design

**Use this skill when:**
- Dimensional models have been logically designed and validated
- You need to create physical database structures (DDL)
- Preparing for database implementation before ETL development
- Optimizing existing dimensional models for performance
- Deciding between relational star schemas and OLAP cubes
- Planning storage, indexing, and partitioning strategies
- Designing aggregate tables for query performance

**Prerequisites:**
- Completed dimensional models (fact and dimension tables logically designed)
- Understanding of business query patterns and performance requirements
- Knowledge of target database platform capabilities
- Estimated data volumes (current and projected growth)

## Overview

Physical database design transforms logical dimensional models into optimized physical structures. While dimensional modeling focuses on simplicity and business understandability, physical design focuses on performance, scalability, and manageability.

The primary goal is to deliver fast query performance and efficient ETL processing while maintaining the dimensional model's simplicity for business users.

## Core Physical Design Decisions

### 1. Platform Selection: Star Schema vs OLAP Cube

**Star Schema (Relational Database)**
- Stores dimensional data in relational tables with star-like structure
- Fact table at center, dimension tables surrounding it
- Industry standard: widely supported by BI tools
- Advantages:
  - Handles detailed atomic data extremely well
  - Easier backup and recovery
  - More portable across different vendors
  - Industry standard SQL for querying
  - Better for very large data volumes (billions of rows)

**OLAP Cube (Multidimensional Database)**
- Stores data in specialized multidimensional structures
- Pre-calculates aggregations for fast query response
- Advanced analytical functions beyond SQL
- Advantages:
  - Superior query performance for aggregations
  - Drill-down/drill-up without new queries
  - More sophisticated security (detail vs summary access)
  - Better support for ragged hierarchies

**Recommended Approach:**
- Load atomic data into relational star schema
- Optionally populate OLAP cubes from star schema for specific use cases
- Star schema serves as stable foundation and source of truth
- See `star-vs-olap-considerations.md` for detailed comparison

### 2. Table Structure Design

#### Fact Table Physical Design

**Surrogate Key Primary Key**
```sql
-- Fact table with surrogate key PK
CREATE TABLE fact_sales (
    sales_key        BIGINT NOT NULL PRIMARY KEY,
    date_key         INTEGER NOT NULL,
    product_key      INTEGER NOT NULL,
    store_key        INTEGER NOT NULL,
    promotion_key    INTEGER NOT NULL,
    customer_key     INTEGER NOT NULL,
    quantity_sold    INTEGER,
    sales_amount     DECIMAL(10,2),
    cost_amount      DECIMAL(10,2)
);
```

**Benefits of surrogate PK:**
- Enables single-column recovery and restart
- Simplifies bulk load error handling
- Supports partition management
- Provides reliable sequencing for incremental loads

**Composite Key Considerations**
```sql
-- Alternative: Natural composite key from dimensions
CREATE TABLE fact_sales (
    date_key         INTEGER NOT NULL,
    product_key      INTEGER NOT NULL,
    store_key        INTEGER NOT NULL,
    promotion_key    INTEGER NOT NULL,
    customer_key     INTEGER NOT NULL,
    quantity_sold    INTEGER,
    sales_amount     DECIMAL(10,2),
    cost_amount      DECIMAL(10,2),
    PRIMARY KEY (date_key, product_key, store_key, customer_key, promotion_key)
);
```

**When to use composite keys:**
- Small to medium fact tables
- Strong uniqueness guarantee from source system
- Database optimizer benefits from multi-column statistics

#### Dimension Table Physical Design

**Surrogate Key Assignment**
```sql
CREATE TABLE dim_product (
    product_key              INTEGER NOT NULL PRIMARY KEY,
    product_id               VARCHAR(20) NOT NULL,  -- Natural key
    product_name             VARCHAR(100),
    brand_name               VARCHAR(50),
    category_name            VARCHAR(50),
    subcategory_name         VARCHAR(50),
    package_type             VARCHAR(30),
    package_size             VARCHAR(20),
    -- SCD Type 2 tracking columns
    row_effective_date       DATE NOT NULL,
    row_expiration_date      DATE NOT NULL,
    current_flag             CHAR(1) NOT NULL,
    -- Audit columns
    row_insert_datetime      TIMESTAMP NOT NULL,
    row_update_datetime      TIMESTAMP
);

-- Sequence for surrogate key generation
CREATE SEQUENCE dim_product_seq START WITH 1;
```

**Key design principles:**
- Surrogate keys are integers for efficiency
- Natural keys preserved as dimension attributes
- Denormalized hierarchies (no snowflaking)
- NULL values replaced with meaningful defaults
- Row tracking for SCD Type 2 dimensions

### 3. Indexing Strategy

Indexing is critical for both query performance and ETL efficiency.

#### Dimension Table Indexes

**Minimum required indexes:**
```sql
-- Primary key index (clustered or unique)
ALTER TABLE dim_product ADD PRIMARY KEY (product_key);

-- Natural key index for surrogate key lookup
CREATE UNIQUE INDEX idx_product_natural ON dim_product(product_id, row_effective_date);

-- Current row index for typical queries
CREATE INDEX idx_product_current ON dim_product(current_flag, row_expiration_date);
```

**Additional indexes based on usage:**
```sql
-- Frequently filtered attributes
CREATE INDEX idx_product_brand ON dim_product(brand_name);
CREATE INDEX idx_product_category ON dim_product(category_name);

-- Bitmap indexes for low-cardinality attributes (Oracle, other platforms)
CREATE BITMAP INDEX idx_product_pkgtype ON dim_product(package_type);
```

#### Fact Table Indexes

**Compound indexes on foreign keys:**
```sql
-- Date is almost always in WHERE clause - put it first
CREATE INDEX idx_sales_date_product ON fact_sales(date_key, product_key);
CREATE INDEX idx_sales_date_store ON fact_sales(date_key, store_key);
CREATE INDEX idx_sales_date_customer ON fact_sales(date_key, customer_key);

-- Other common query patterns
CREATE INDEX idx_sales_product_date ON fact_sales(product_key, date_key);
CREATE INDEX idx_sales_store_date ON fact_sales(store_key, date_key);
```

**Bitmap indexes for low-cardinality dimensions:**
```sql
-- Promotion has relatively few values
CREATE BITMAP INDEX idx_sales_promotion ON fact_sales(promotion_key);
```

**Index design principles:**
- Date dimension key almost always appears first in compound indexes
- Create indexes supporting common query patterns
- Balance query performance vs ETL load time
- Drop indexes during bulk loads, rebuild after
- Monitor query plans to identify missing indexes

See `indexing-best-practices.md` for comprehensive indexing guidance.

### 4. Table Partitioning

Partitioning divides large tables into manageable chunks for performance and maintenance.

#### Date-Based Partitioning (Most Common)

**Monthly partitioning example:**
```sql
-- Fact table partitioned by month
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
    -- ... continue for each month
    PARTITION p_max VALUES LESS THAN (MAXVALUE)
);
```

**Benefits:**
- Partition elimination improves query performance
- Easier to drop old partitions than delete rows
- Parallel load into different partitions
- Faster index rebuilds on partitions
- Simplified backup/recovery strategies

**Partitioning strategies by fact table type:**
- **Transaction facts:** Partition by transaction date (daily, weekly, or monthly)
- **Periodic snapshots:** Partition by snapshot date (monthly or quarterly)
- **Accumulating snapshots:** Partition by most recent milestone date

#### Other Partitioning Schemes

**Range-list composite partitioning:**
```sql
-- Partition by date, subpartition by region
PARTITION BY RANGE (date_key)
SUBPARTITION BY LIST (region_key) (
    PARTITION p_2023_01 VALUES LESS THAN (20230201) (
        SUBPARTITION p_2023_01_east VALUES (1, 2, 3),
        SUBPARTITION p_2023_01_west VALUES (4, 5, 6)
    )
);
```

**When to use:**
- Queries frequently filter by multiple attributes
- Geographic or organizational segregation requirements
- Regulatory data retention policies vary by region

See `partitioning-strategies.md` for detailed implementation patterns.

### 5. Aggregate Table Design

Aggregate tables pre-calculate summaries for fast query performance.

#### When Aggregates Are Needed

**Indicators you need aggregates:**
- Queries scanning millions of atomic fact rows
- Repeated queries at summary levels (monthly, by category, etc.)
- User complaints about slow report performance
- Dashboard queries timing out

**When aggregates may not be needed:**
- Modern columnar databases with excellent compression
- In-memory databases or appliances
- Fact tables under 50-100 million rows
- OLAP cubes already providing aggregations

#### Aggregate Table Design Pattern

**Example aggregate: Monthly product category sales**
```sql
CREATE TABLE fact_sales_monthly_category (
    month_key           INTEGER NOT NULL,
    category_key        INTEGER NOT NULL,
    -- Removed: product_key, store_key, customer_key, promotion_key, daily date
    quantity_sold       INTEGER,
    sales_amount        DECIMAL(12,2),
    cost_amount         DECIMAL(12,2),
    -- Aggregated metrics
    transaction_count   INTEGER,
    avg_basket_size     DECIMAL(10,2),
    PRIMARY KEY (month_key, category_key)
);
```

**Aggregate design principles:**
- Remove dimensions that don't participate in aggregation
- Shrunken dimensions replace detailed dimensions (month vs day, category vs product)
- Include all additive facts from base table
- Add derived facts (counts, averages) that improve usability
- Maintain same naming conventions as base fact table
- Build aggregates from base facts, not from other aggregates

**Aggregate navigation:**
- BI tool automatically selects appropriate aggregate (preferred)
- Database materialized views with query rewrite
- Manual query routing logic (least preferred)

### 6. Compression and Columnar Storage

Modern databases offer compression and columnar storage for dimensional models.

**Row-oriented storage (traditional):**
- Entire row stored contiguously
- Good for OLTP and single-row retrieval
- Typical for dimension tables

**Columnar storage:**
- Each column stored separately
- Excellent compression (10-40x typical)
- Dramatically faster for analytic queries
- Ideal for fact tables
- Available in: Redshift, Snowflake, BigQuery, Synapse, Vertica, columnar indexes in SQL Server/Oracle

**Compression techniques:**
- **Dictionary encoding:** Perfect for dimension foreign keys
- **Run-length encoding:** Efficient for sorted data
- **Delta encoding:** Good for sequential values
- **Bit-packing:** Compresses integer ranges

**Implementation:**
```sql
-- Columnar table (Redshift example)
CREATE TABLE fact_sales (
    -- columns
)
DISTSTYLE KEY
DISTKEY (date_key)
SORTKEY (date_key, product_key)
ENCODE AUTO;

-- Compression on specific columns (SQL Server)
CREATE TABLE fact_sales (
    date_key INT NOT NULL,
    sales_amount DECIMAL(10,2)
)
WITH (DATA_COMPRESSION = PAGE);
```

### 7. Referential Integrity Constraints

**Recommendation:** Define but typically disable RI constraints in production.

```sql
-- Define foreign keys for documentation and tooling
ALTER TABLE fact_sales
    ADD CONSTRAINT fk_sales_date
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key);

-- Disable for bulk loads
ALTER TABLE fact_sales NOCHECK CONSTRAINT fk_sales_date;

-- Re-enable with NOCHECK (trust existing data)
ALTER TABLE fact_sales CHECK CONSTRAINT fk_sales_date;
```

**Why disable in production:**
- ETL system manages RI through surrogate key pipeline
- Constraints add overhead during loads
- Fact table bulk loads dramatically slower with RI enabled
- Constraints still provide metadata for BI tools

**When to enforce RI:**
- Development and test environments
- Small data volumes
- Strong business requirement for database-enforced integrity

### 8. Naming Conventions

Consistent naming is critical for usability and maintainability.

**Standard naming conventions:**
```
Tables:
  - fact_[business_process]        (fact_sales, fact_orders)
  - dim_[dimension_name]           (dim_product, dim_customer)
  - agg_[fact]_[grain]_[dims]      (agg_sales_monthly_category)

Keys:
  - [table]_key                    (product_key, date_key, sales_key)
  - [natural_key]_id               (product_id, customer_id)

Columns:
  - descriptive_business_names     (sales_amount, not amt or slsamt)
  - consistent_suffixes            (_date, _amount, _quantity, _count)

Constraints:
  - pk_[table]                     (pk_fact_sales)
  - fk_[table]_[dimension]         (fk_sales_product)
  - idx_[table]_[columns]          (idx_sales_date_product)
```

**Principles:**
- Use full words, avoid abbreviations
- Be consistent across all tables
- Use business-friendly terminology
- Standardize suffixes and prefixes

### 9. Security and Access Control

**Database-level security:**
```sql
-- Create read-only roles for business users
CREATE ROLE bi_analyst_role;
GRANT SELECT ON fact_sales TO bi_analyst_role;
GRANT SELECT ON dim_product TO bi_analyst_role;
GRANT SELECT ON dim_customer TO bi_analyst_role;

-- Limited access to sensitive dimensions
CREATE ROLE finance_role;
GRANT SELECT ON fact_gl TO finance_role;
GRANT SELECT ON dim_account TO finance_role;
```

**Row-level security (where supported):**
```sql
-- Restrict customer dimension by sales region
CREATE POLICY customer_region_policy ON dim_customer
FOR SELECT
USING (region_key IN (
    SELECT region_key FROM user_region_access
    WHERE user_name = CURRENT_USER
));
```

**Column-level security:**
```sql
-- Mask sensitive columns for non-privileged users
CREATE VIEW dim_customer_public AS
SELECT
    customer_key,
    customer_name,
    -- Redact sensitive data
    '***-**-****' AS social_security_number,
    city_name,
    state_code
FROM dim_customer;

GRANT SELECT ON dim_customer_public TO bi_analyst_role;
```

## Physical Design Process

### Step-by-Step Implementation

**1. Gather physical requirements**
- Data volumes (current and 3-year projection)
- Query patterns (batch reports vs ad-hoc vs dashboards)
- Load windows and availability requirements
- Retention and archiving policies
- Performance SLAs (query response time targets)

**2. Select database platform and architecture**
- Relational database vs OLAP cube
- On-premises vs cloud
- Appliance, columnar, in-memory, or traditional RDBMS
- Consider: cost, scalability, skills, vendor support

**3. Create DDL scripts for dimension tables**
- Define surrogate keys with sequences
- Include all attributes (denormalized)
- Add SCD tracking columns (Type 2 dimensions)
- Replace NULLs with default values
- Create primary key and natural key indexes

**4. Create DDL scripts for fact tables**
- Define foreign keys to all dimensions
- Include all facts at declared grain
- Add surrogate key if needed
- Define partitioning scheme (typically by date)
- Create initial index set

**5. Define partitioning strategy**
- Partition fact tables by date (most common)
- Create partition maintenance procedures
- Plan for partition swapping during loads
- Document partition retention policy

**6. Design indexing strategy**
- Create compound indexes on frequently joined dimensions
- Balance query performance vs load performance
- Plan to drop/disable during bulk loads
- Document index rebuild procedures

**7. Plan aggregate tables**
- Identify common query patterns needing pre-aggregation
- Design aggregate fact tables at multiple grains
- Create aggregate maintenance processes
- Configure aggregate navigation

**8. Configure compression and storage**
- Enable columnar storage for fact tables (if available)
- Configure compression on large text columns
- Set tablespace and storage parameters
- Plan for growth and capacity

**9. Implement security model**
- Create database roles for user groups
- Grant appropriate permissions
- Implement row/column-level security if needed
- Document security policies

**10. Create physical database documentation**
- Complete DDL scripts
- ER diagrams showing physical structure
- Indexing rationale and query patterns
- Partitioning scheme and maintenance procedures
- Performance baseline metrics

## Common Mistakes to Avoid

**1. Premature optimization**
- Don't create aggregates before understanding actual query patterns
- Start with simple indexing, monitor, then optimize

**2. Snowflaking dimensions**
- Keep dimensions denormalized even in physical design
- Snowflaking destroys simplicity and performance

**3. Over-indexing**
- Too many indexes slow down ETL dramatically
- Create indexes based on actual query patterns, not speculation

**4. Ignoring partitioning**
- Large fact tables (>100M rows) need partitioning
- Partitioning is much harder to add retroactively

**5. Enforcing RI constraints in production**
- Constraint validation during bulk loads kills performance
- ETL surrogate key pipeline is sufficient

**6. Inconsistent naming**
- Mixed naming conventions confuse users and developers
- Establish and enforce standards from the beginning

**7. Not planning for growth**
- Design for 3-year projected volumes, not current state
- Consider partition scheme, compression, and scaling strategy

**8. Ignoring database-specific features**
- Each platform has unique optimization capabilities
- Leverage columnar storage, compression, in-memory options

## Validation Checklist

Before finalizing physical design:

**Completeness:**
- [ ] All dimension tables have DDL with surrogate keys
- [ ] All fact tables have DDL with proper grain
- [ ] Partitioning scheme defined for large fact tables
- [ ] Primary and foreign keys documented
- [ ] Indexing strategy documented for each table
- [ ] Naming conventions consistently applied

**Performance:**
- [ ] Expected data volumes documented with growth projections
- [ ] Query patterns analyzed and documented
- [ ] Partitioning aligned with most common query filters
- [ ] Indexes support identified query patterns
- [ ] Aggregate requirements identified
- [ ] Compression strategy defined for large tables

**Manageability:**
- [ ] Partition maintenance procedures documented
- [ ] Index rebuild procedures documented
- [ ] Backup and recovery strategy defined
- [ ] Archiving and purging strategy defined
- [ ] Capacity planning completed

**Security:**
- [ ] Database roles defined
- [ ] Permissions granted appropriately
- [ ] Sensitive data protected
- [ ] Compliance requirements addressed

**Documentation:**
- [ ] Complete DDL scripts version-controlled
- [ ] Physical ER diagrams created
- [ ] Partitioning scheme documented
- [ ] Index rationale documented
- [ ] Security model documented

## Integration with Other Skills

**Prerequisites:**
- **Four-Step Dimensional Modeling** - Logical models must be complete
- **Dimension Table Design Patterns** - SCD strategies drive physical design
- **Fact Table Design Patterns** - Grain and fact types influence partitioning

**Next Steps:**
- **ETL System Architecture Design** - Physical design drives ETL approach
- **Historic Data Loading Strategy** - Bulk load optimizations depend on physical design
- **Incremental ETL Processing** - Index strategy affects incremental performance

## Quick Reference

**Decision Matrix:**

| Requirement | Recommendation |
|-------------|----------------|
| Fact table > 100M rows | Partition by date (monthly or daily) |
| Slowly changing dimensions | Add row_effective_date, row_expiration_date, current_flag |
| Query response > 5 seconds | Investigate aggregates or columnar storage |
| Mixed OLTP/OLAP workload | Separate operational from analytic databases |
| Regulatory retention | Partition by date for easy purging |
| Complex analytics | Consider OLAP cube fed from star schema |
| Very large dimensions (>10M rows) | Mini-dimensions, indexing on current_flag |

**Platform-Specific Optimizations:**
- **Oracle:** Partition by range, bitmap indexes, materialized views
- **SQL Server:** Columnstore indexes, partition switching
- **Redshift:** Distribution keys, sort keys, columnar encoding
- **Snowflake:** Clustering keys, automatic compression
- **BigQuery:** Partitioned tables, clustered tables

## Summary

Physical database design transforms elegant logical dimensional models into high-performance physical structures. The key is balancing:
- **Simplicity** for business users and BI tools
- **Performance** for queries and ETL processes
- **Scalability** for growing data volumes
- **Manageability** for operations and maintenance

Start with a solid foundation (proper keys, partitioning, basic indexes), monitor actual usage, and optimize incrementally based on real-world performance data. Avoid premature optimization while planning for known scale requirements.

The dimensional model's symmetry and simplicity survive physical implementation, ensuring business users continue to benefit from intuitive, fast access to enterprise data.
