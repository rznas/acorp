---
name: historic-data-loading-strategy
description: Design and execute the one-time initial load of historical data into dimension and fact tables. Use when performing the first bulk load of a new data warehouse or business process dimensional model, before incremental ETL begins. Covers dimension population (Type 1, Type 2 SCD history reconstruction, static dimensions), fact table bulk loading, surrogate key assignment, referential integrity validation, and performance optimization techniques for large-volume historic loads.
---

# Historic Data Loading Strategy

## When to Use This Skill

Use this skill when you need to perform the **one-time initial load** of historical data into a new data warehouse or business process dimensional model. This occurs:

- After dimensional models have been designed and physically implemented
- Before incremental ETL processes begin
- When populating a new business process (new row in the bus matrix)
- When rebuilding a data warehouse with historical data

**Prerequisites:**
- Logical dimensional model complete (dimensions and facts designed)
- Physical database structures created (tables, partitions)
- Source-to-target mappings documented
- Data profiling completed
- ETL specification document prepared

**Note:** Historic load ETL processes are typically separate from incremental load processes, though some ETL tool functionality can be reused between them.

## Overview

The historic load is a **bulk, one-time operation** focused on loading large volumes of historical data. Key characteristics:

- **Volume**: Often thousands of times larger than daily incremental loads
- **Timing**: Can take several days; not constrained by nightly load windows
- **Offline**: Loading into tables not yet in production use
- **Manual intervention**: Can pause to examine data and fix issues
- **Performance priority**: Optimize for throughput, not availability

**Load Sequence:**
1. **Dimension tables first** (simplest to most complex)
2. **Fact tables second** (after all dimensions are loaded)

## Step 5: Populate Dimension Tables with Historic Data

Start with the simplest dimension tables and progressively tackle more complex ones.

### Load Sequence for Dimensions

<load_sequence>
**Order 1: Static Dimensions**
- Date dimension (pre-built, typically in spreadsheet)
- Other static/constructed dimensions (budget scenarios, etc.)
- Require business sign-off

**Order 2: Type 1 Only Dimensions**
- All attributes managed as overwrites
- Extract current values from source system
- Simplest to load

**Order 3: Type 2 SCD Dimensions**
- Contains one or more Type 2 (history-tracked) attributes
- Requires history reconstruction
- Most complex dimension loading
</load_sequence>

### Dimension Transformations

All dimensions require transformations before loading.

#### 1. Simple Data Transformations

<transformations>
**Data Type Conversion**
- Use ETL tool functions for type conversions
- Tedious but straightforward

**NULL Replacement**
- **Critical**: Replace NULLs with default values in dimension tables
- NULLs cause problems in queries (comparison issues, Unknown results)
- Example defaults: "Unknown", "Not Applicable", "TBD", 0

**Production Code Decoding**
- Look up text equivalents for production codes
- Store lookup tables in staging database
- Include default handling for missing codes

**Example:**
```sql
-- Replace NULLs with defaults
COALESCE(product_description, 'Unknown Product')
COALESCE(category, 'Unassigned')
COALESCE(customer_phone, 'Not Provided')
```
</transformations>

#### 2. Combine from Separate Sources

<multi_source_consolidation>
When dimensions derive from multiple sources (e.g., customer data from multiple lines of business):

**Challenges:**
- No universal key across sources
- Name/address variations and duplicates
- Requires deduplication and consolidation

**Approach:**
1. **Parse** names and addresses into component pieces
2. **Use fuzzy logic** for matching:
   - Account for misspellings, typos
   - Handle alternative spellings (IBM vs I.B.M. vs International Business Machines)
3. **Multiple passes** with progressively looser matching rules
4. **MDM integration**: Leverage Master Data Management systems if available

**One-Time vs Ongoing:**
- Large one-time consolidation project for existing customers
- Ongoing incremental deduplication for new records
</multi_source_consolidation>

#### 3. Validate Hierarchical Relationships

<hierarchy_validation>
**Requirement:** Hierarchical rollups must be **perfectly clean**.

**Many-to-One Relationships:**
Product → Product Model → Subcategory → Category

**Validation Approach:**

**SQL Verification:**
```sql
-- Find violations: products with multiple models
SELECT Product_SKU,
       count(*) as Row_Count,
       count(DISTINCT Product_Model) as Model_Count
FROM StagingDatabase.Product
GROUP BY Product_SKU
HAVING count(DISTINCT Product_Model) > 1;
```

**Snowflake Staging (Optional):**
- Load into normalized (snowflaked) structure in staging area
- Database enforces foreign key constraints
- Prevents loading data that violates many-to-one relationships
- **Pre-verify** hierarchies before attempting load

**Source System Quality:**
- **Best case**: Source system normalizes hierarchy levels with FK constraints
- **Risky case**: Hierarchies from spreadsheets or denormalized sources
- **Action**: Either clean up or acknowledge it's not a true hierarchy

**Guidelines:**
- Pre-verify relationships before loading
- Don't rely solely on database constraints at load time
- Document any hierarchy exceptions or business rules
</hierarchy_validation>

#### 4. Dimension Surrogate Key Assignment

<surrogate_key_assignment>
After dimension data is clean with one row per unique dimension member:

**Purpose:**
- Assign integer surrogate keys (independent of natural keys)
- Maintain key map table (production natural key ↔ surrogate key)
- Use key map during fact table processing

**Implementation (RDBMS Sequence):**

**Step 1: Create sequence (one-time)**
```sql
CREATE SEQUENCE dim_product_seq CACHE=1000;
-- Choose appropriate cache level for performance
```

**Step 2: Populate key map table**
```sql
INSERT INTO dim_product_key_map (production_key_id, product_key)
SELECT production_key_id, dim_product_seq.NEXT
FROM dim_product_extract_table;
```

**Step 3: Join to dimension data and load**
```sql
INSERT INTO dim_product (product_key, product_sku, product_name, ...)
SELECT km.product_key, e.product_sku, e.product_name, ...
FROM dim_product_extract_table e
JOIN dim_product_key_map km ON e.production_key_id = km.production_key_id;
```

**Key Characteristics:**
- Sequential integers (1, 2, 3, ...)
- Independent of source system keys
- Enables handling of source system key changes
- Required for Type 2 SCD (multiple rows per natural key)
</surrogate_key_assignment>

### Dimension Table Loading

<dimension_loading>
**Use Bulk Loading Utilities:**
- Even for small first dimension tables, use database bulk/fast-loading utilities
- Set pattern for all subsequent loads
- Most major databases support bulk loading APIs or extended SQL

**Bulk Load Parameters:**

**1. Turn Off Logging**
- Transaction logging adds significant overhead
- Not valuable for DW table loads
- Design ETL with recoverability/restart points instead

**2. Bulk Load in Fast Mode**
- Database engines have stringent conditions for "fast path"
- If conditions not met, load doesn't fail—just uses slower path
- Check documentation for fast mode requirements

**3. Presort the File**
- Sort in order of primary index
- Significantly speeds up indexing
- Reduces fragmentation

**4. Transform with Caution**
- Some loaders support conversions, calculations, string manipulation
- **Warning**: May switch loader out of high-speed mode
- **Recommendation**: Perform transformations in ETL tool, not loader

**5. Truncate Table Before Full Refresh**
- `TRUNCATE TABLE` is most efficient for deleting all rows
- Commonly used for staging tables at start of ETL processing
- Faster than `DELETE FROM table`

**Example:**
```sql
-- Clean staging table
TRUNCATE TABLE staging.dim_product;

-- Bulk load
BULK INSERT dim_product
FROM 'product_dimension.dat'
WITH (
    TABLOCK,           -- Fast load mode
    ROWS_PER_BATCH = 100000,
    ORDER (product_key)  -- Presorted
);
```
</dimension_loading>

### Load Type 2 Dimension Table History

<type2_history_reconstruction>
**Challenge:** Re-create historical changes for Type 2 attributes.

**Business Requirement:**
- If an attribute tracks history, business wants that history going back in time
- Not just from data warehouse implementation date forward
- Often difficult or impossible to reconstruct

**History Sources:**
1. **Operational archives** (if they exist)
2. **Historical snapshots** of master files
3. **Transaction logs** with before/after values
4. **External historical data** (e.g., customer address changes)
5. **Derived from facts** (e.g., customer status changes inferred from transaction patterns)

**Implementation Approach:**

**Not Well Suited for Standard SQL:**
- Use database cursor constructs, OR
- Use procedural language (Visual Basic, C, Java, Python), OR
- Use ETL tool script processing

**Process:**
1. **Extract** all historical snapshots of dimension member
2. **Sort** by effective date
3. **Compare** sequential rows to identify changes
4. **For each change in Type 2 attribute:**
   - Create new dimension row
   - Assign new surrogate key
   - Set row_effective_date
   - Set row_end_date (or leave as "current" for latest)
   - Set current_flag appropriately

**Set Row End Dates (Final Pass):**
- Ensure no gaps in time series
- **Daily granularity**: End date = day before next row's begin date
- **Timestamp granularity**: End date = exact begin timestamp of next row

**Example Logic:**
```python
# Pseudocode for Type 2 history reconstruction
for each natural_key in source_data:
    snapshots = get_historical_snapshots(natural_key)
    snapshots.sort_by(effective_date)

    previous_row = None
    for snapshot in snapshots:
        if previous_row is None or type2_attributes_changed(previous_row, snapshot):
            new_surrogate_key = get_next_surrogate_key()

            dimension_row = {
                'surrogate_key': new_surrogate_key,
                'natural_key': natural_key,
                'attributes': snapshot.attributes,
                'row_effective_date': snapshot.effective_date,
                'row_end_date': '9999-12-31',  # Current row marker
                'current_flag': 'Y'
            }

            if previous_row is not None:
                # Update previous row's end date
                previous_row.row_end_date = snapshot.effective_date - 1 day
                previous_row.current_flag = 'N'

            insert_dimension_row(dimension_row)
            previous_row = dimension_row
```

**Row End Date Best Practices:**
- **Current rows**: Use far-future date (9999-12-31) not NULL
- **NULL comparison issues**: Relational databases may error or return Unknown
- **Gap prevention**: Critical for correct Type 2 lookups
</type2_history_reconstruction>

### Populate Date and Static Dimensions

<static_dimensions>
**Date Dimension:**
- **Requirement**: Every data warehouse needs a date dimension
- **Granularity**: One row per day (most common)
- **Span**: From oldest fact transaction to future dates (3-5 years ahead)
- **Creation**: Usually built by hand in spreadsheet
- **Columns**: Day, month, quarter, year, day_of_week, fiscal_period, holiday_flag, etc.

**Other Static Dimensions:**
- Budget scenario dimension (Actual, Budget, Forecast)
- Version dimensions
- Reason code dimensions
- Any dimension without source system data

**Governance:**
- Business data governance representatives must sign off
- These are "constructed" dimensions (not sourced from systems)
- Require business approval of values and definitions

**Example Date Dimension Creation:**
```sql
-- Generate 10 years of dates
INSERT INTO dim_date (date_key, full_date, day_of_week, month_name, ...)
SELECT
    CAST(FORMAT(date_value, 'yyyyMMdd') AS INT) as date_key,
    date_value as full_date,
    DATENAME(weekday, date_value) as day_of_week,
    DATENAME(month, date_value) as month_name,
    ...
FROM generate_series('2020-01-01', '2029-12-31', INTERVAL '1 day') as date_value;
```
</static_dimensions>

## Step 6: Perform the Fact Table Historic Load

Load fact tables **after all dimension tables** are populated.

### Historic Fact Table Characteristics

<historic_fact_overview>
**Differences from Incremental Load:**
- **Volume**: Sometimes thousands of times bigger than daily loads
- **Target**: Loading into table not yet in production
- **Timing**: Multi-day loads are tolerable
- **Intervention**: Can pause and examine data
- **Performance**: Optimize for bulk loading throughput

**Primary Concern:** Sheer data volume

**Advantage:** Offline loading (no user impact)
</historic_fact_overview>

### Historic Fact Table Extracts

<fact_extraction>
**Scope Definition:**
- Identify records within extraction parameters
- **Filter out** operational records not useful for analytics
- Many transaction systems keep operational data irrelevant to business analysis

**Audit Statistics Collection:**
- Capture during extract step (efficient point in pipeline)
- Calculate subtotals, totals, row counts
- **Purpose**: Cross-check integrity between source and warehouse

**Audit Statistics Examples:**
- Total row count
- Sum of revenue by month
- Sum of revenue by product category
- Count of transactions by store
- These tie back to operational reports and forward to warehouse load results

**Credibility Requirement:**
- Warehouse must tie back to source system
- Explains differences when perfect tie-back is impossible:
  - Business rules applied in warehouse but not in source
  - Source system errors being corrected
  - Timing differences

**Example Extract with Audit Stats:**
```sql
-- Extract with audit stat capture
INSERT INTO staging.fact_sales_extract
SELECT transaction_id, transaction_date, store_id, product_id,
       quantity, revenue, cost
FROM source_system.transactions
WHERE transaction_date BETWEEN '2020-01-01' AND '2024-12-31'
  AND transaction_status = 'COMPLETED'
  AND revenue > 0;

-- Capture audit statistics
INSERT INTO audit_stats (load_date, stat_name, stat_value)
VALUES
    (CURRENT_DATE, 'extract_row_count', (SELECT count(*) FROM staging.fact_sales_extract)),
    (CURRENT_DATE, 'extract_revenue_sum', (SELECT sum(revenue) FROM staging.fact_sales_extract)),
    (CURRENT_DATE, 'extract_date_range_start', (SELECT min(transaction_date) FROM staging.fact_sales_extract)),
    (CURRENT_DATE, 'extract_date_range_end', (SELECT max(transaction_date) FROM staging.fact_sales_extract));
```
</fact_extraction>

### Fact Table Transformations

<fact_transformations>
**General Principle:**
- Fact data is usually **relatively clean** (comes from operational transaction systems)
- ETL developers spend more time on dimension content
- Facts require **modest transformation** compared to dimensions

**Common Transformations:**

**1. Null Fact Values**
- Source systems may use special values (e.g., -1) to represent NULL
- **Replace special values** with true NULL for numeric measures
- **NULL behavior**: Does "right thing" in SUM() and AVG() calculations
- **Exception**: Foreign key columns (dimension references) must be NOT NULL

**2. Grain Consistency**
- **Requirement**: All facts must be expressed at same grain
- **Remove** any facts representing totals/aggregates at different grains
- Example: No year totals in daily fact table
- Move aggregate-grain facts to appropriate aggregate tables

**3. Derived Facts**
- Pre-compute derived facts if:
  - Users query physical table directly (not through view)
  - Need to index on derived value
  - Performance requires pre-calculation
- Calculate in view/OLAP cube if:
  - Users access through view only
  - Calculation is simple
  - Want to ensure correct calculation every time

**Example Derived Fact:**
```sql
-- Derived fact: net_profit = revenue - cost
INSERT INTO fact_sales (date_key, product_key, revenue, cost, net_profit, ...)
SELECT
    date_key,
    product_key,
    revenue,
    cost,
    revenue - cost as net_profit,  -- Derived fact
    ...
FROM staging.fact_sales_prepared;
```

**4. Multiple Units of Measure**
- Handle in view or OLAP cube if possible
- Pre-compute if users need to filter or index on alternate UOM

**Next Step:** Surrogate Key Pipeline
</fact_transformations>

### Surrogate Key Pipeline for Historic Facts

<surrogate_key_pipeline>
**Purpose:** Exchange natural keys for surrogate keys while maintaining referential integrity.

**Pipeline Position:**
- **Final operation** before loading target fact table
- All other cleaning, transformation, processing already complete
- Incoming fact data looks like target fact table except still has natural keys

**Prerequisite:**
- **All dimension processing must be complete**
- New dimension members loaded
- Type 2 changes processed
- Surrogate keys available in key map tables

**Referential Integrity Requirement:**
- **Never** allow fact row with dimension key that doesn't exist
- **Never** have NULL values in foreign key columns
- **Guarantee** RI between fact and dimension tables

**Historic Load RI Validation:**

**Manual Pre-Check (Historic Load Advantage):**
1. Confirm each natural key in fact extract exists in dimension tables
2. Pause historic load to investigate violations
3. Fix dimension table OR redesign fact extract to filter spurious rows

**Example RI Validation:**
```sql
-- Check for product keys in facts but not in dimension
SELECT DISTINCT f.product_natural_key
FROM staging.fact_sales_extract f
LEFT JOIN dim_product p ON f.product_natural_key = p.product_natural_key
WHERE p.product_key IS NULL;

-- If violations found, either:
-- Option 1: Add missing products to dimension
-- Option 2: Filter out invalid fact rows
```

**Lookup Approaches for Historic Load:**

**Approach 1: Big Join Query (Relational Database)**
```sql
-- Single query joining fact staging to all dimensions
SELECT
    dd.date_key,
    dp.product_key,
    ds.store_key,
    dc.customer_key,
    f.quantity,
    f.revenue,
    f.cost
FROM staging.fact_sales_extract f
JOIN dim_date dd
    ON f.transaction_date = dd.full_date
JOIN dim_product p
    ON f.product_natural_key = p.product_natural_key
    AND p.current_flag = 'Y'  -- Type 1 dimension
JOIN dim_store s
    ON f.store_natural_key = s.store_natural_key
    AND f.transaction_date BETWEEN s.row_effective_date AND s.row_end_date  -- Type 2
JOIN dim_customer c
    ON f.customer_natural_key = c.customer_natural_key
    AND f.transaction_date BETWEEN c.row_effective_date AND c.row_end_date;  -- Type 2
```

**Benefits:**
- Leverages relational database power
- Parallel lookups across all dimensions
- Simplifies Type 2 lookup with BETWEEN clause on effective/end dates

**Considerations:**
- Works well for moderate volumes (< hundreds of GB)
- Type 2 joins create greatest demand
- Ensure effective_date and end_date columns are indexed
- May be too slow for very large historic loads (hundreds of GB to TB range)

**Approach 2: Separate Type 1 and Type 2 Lookups**
- Use big join for all Type 1 dimensions (one pass)
- Handle Type 2 dimensions separately (with BETWEEN logic)
- Reduces complexity of single massive join

**Approach 3: ETL Tool Lookup Operator**
- Use ETL tool's built-in lookup functionality
- Tool handles caching and optimization
- Processes row-by-row or in micro-batches
- Good for very large volumes

**Type 2 Dimension Lookup - Critical Detail:**
```sql
-- Must use BETWEEN to get correct historical version
WHERE fact_transaction_date BETWEEN dim.row_effective_date AND dim.row_end_date
```

**Result:**
- All natural keys replaced with surrogate keys
- Fact row guaranteed to have RI with dimension tables
- Ready to load into target fact table
</surrogate_key_pipeline>

### Assign Audit Dimension Key

<audit_dimension>
**Purpose:** Track data lineage and quality metadata for each fact row.

**Audit Dimension Content:**
- Master ETL version number
- Load date/time
- Source system identifier
- Business rule version numbers (e.g., profit allocation logic)
- Data quality indicators:
  - "Quality Checks Passed"
  - "Quality Problems Encountered"
  - Specific quality issue codes

**Size:** Usually quite small (hundreds to thousands of rows)

**Evolution:** Start simple, add detail over time

**Pipeline Position:**
- Add audit key immediately after OR immediately before surrogate key pipeline
- Usually minimal performance impact

**Example:**
```sql
-- Create audit record for this load
INSERT INTO dim_audit (etl_version, load_date, source_system, quality_status)
VALUES ('2.1.5', CURRENT_DATE, 'SAP_PROD', 'Passed')
RETURNING audit_key INTO v_audit_key;

-- Add audit key to fact rows
UPDATE staging.fact_sales_ready
SET audit_key = v_audit_key;
```
</audit_dimension>

### Fact Table Loading

<fact_table_loading>
**Primary Concern:** Load performance with large volumes.

**Performance Techniques:**

**1. Use Bulk Loader**
- Fastest loading method available for database
- Extended SQL (BULK INSERT) or API

**2. Set Batch Size**
- Database technologies support fast loading with specified batch size
- Experiment to find ideal size for row size and memory configuration
- Typical values: 10,000 / 100,000 / 1,000,000 rows per batch

**3. Load into Partitioned Table**
- Best way to improve historic load performance
- Load multiple partitions in parallel

**Partitioned Table Load Steps:**

```sql
-- STEP 1: Disable foreign key constraints
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_sales_date;
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_sales_product;
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_sales_store;
ALTER TABLE fact_sales DISABLE CONSTRAINT fk_fact_sales_customer;

-- STEP 2: Drop or disable indexes
ALTER INDEX idx_fact_sales_date DISABLE;
ALTER INDEX idx_fact_sales_product DISABLE;
ALTER INDEX idx_fact_sales_store DISABLE;

-- STEP 3: Load data using fast-loading techniques
BULK INSERT fact_sales
FROM 'fact_sales_data.dat'
WITH (
    TABLOCK,
    ROWS_PER_BATCH = 1000000,
    ORDER (date_key)  -- Presorted by partition key
);

-- STEP 4: Create or enable fact table indexes
ALTER INDEX idx_fact_sales_date REBUILD;
ALTER INDEX idx_fact_sales_product REBUILD;
ALTER INDEX idx_fact_sales_store REBUILD;

-- STEP 5: Stitch together partitions (if necessary)
-- Database-specific partition management commands

-- STEP 6: Confirm dimension surrogate key indexes exist
CREATE UNIQUE INDEX uk_dim_product ON dim_product(product_key);
CREATE UNIQUE INDEX uk_dim_store ON dim_store(store_key);

-- STEP 7: Enable foreign key constraints
ALTER TABLE fact_sales ENABLE CONSTRAINT fk_fact_sales_date;
ALTER TABLE fact_sales ENABLE CONSTRAINT fk_fact_sales_product;
ALTER TABLE fact_sales ENABLE CONSTRAINT fk_fact_sales_store;
ALTER TABLE fact_sales ENABLE CONSTRAINT fk_fact_sales_customer;
```

**Parallel Partition Loading:**
- If fact table partitioned by date (common practice)
- Load multiple date partitions simultaneously
- Each partition load is independent
- Significant performance improvement for large historic loads

**Example Parallel Load:**
```bash
# Load 12 months in parallel (4 concurrent jobs)
load_partition.sh 2024-01 &
load_partition.sh 2024-02 &
load_partition.sh 2024-03 &
load_partition.sh 2024-04 &
wait

load_partition.sh 2024-05 &
load_partition.sh 2024-06 &
# ... continue for all partitions
```
</fact_table_loading>

## Validation and Quality Checks

After historic load completion, perform comprehensive validation.

<validation>
**Row Count Reconciliation:**
```sql
-- Compare staging to production
SELECT 'Staging' as source, count(*) as row_count
FROM staging.fact_sales_extract
UNION ALL
SELECT 'Production' as source, count(*) as row_count
FROM fact_sales;
```

**Audit Statistic Verification:**
```sql
-- Verify revenue totals match
SELECT sum(revenue) FROM staging.fact_sales_extract;  -- From extract
SELECT sum(revenue) FROM fact_sales;                  -- From loaded table
```

**Referential Integrity Verification:**
```sql
-- Should return zero rows
SELECT count(*)
FROM fact_sales f
LEFT JOIN dim_product p ON f.product_key = p.product_key
WHERE p.product_key IS NULL;
```

**Date Range Verification:**
```sql
SELECT min(date_key), max(date_key) FROM fact_sales;
```

**Dimension Coverage Check:**
```sql
-- Verify all dimension members referenced
SELECT d.product_key, d.product_name, count(f.product_key) as fact_count
FROM dim_product d
LEFT JOIN fact_sales f ON d.product_key = f.product_key
WHERE d.current_flag = 'Y'
GROUP BY d.product_key, d.product_name
ORDER BY fact_count;
```

**Deliverables:**
- Load validation report with all checks documented
- Audit statistics comparison (source vs warehouse)
- Row count reconciliation
- Data quality scorecards
- Performance benchmarks (rows/second, elapsed time)
</validation>

## Common Mistakes to Avoid

<common_mistakes>
**1. Loading Facts Before Dimensions**
- Results in referential integrity violations
- Must load dimensions first

**2. Not Validating Hierarchies**
- Dirty hierarchies cause reporting errors
- Pre-validate many-to-one relationships

**3. Skipping Audit Statistics**
- Lost opportunity to establish credibility
- Capture during extract for efficiency

**4. Using Incremental Load Logic for Historic**
- Historic and incremental have different requirements
- Historic can be less automated, more optimized for bulk

**5. Not Using Bulk Load Utilities**
- Standard INSERT statements are too slow
- Always use database fast-loading features

**6. Loading with Indexes Enabled**
- Dramatically slows load performance
- Disable/drop indexes, rebuild after load

**7. Failing to Reconstruct Type 2 History**
- Business expects historical accuracy
- Plan for history reconstruction or set expectations

**8. Not Pre-Sorting Data**
- Pre-sorting by index columns speeds indexing
- Reduces fragmentation

**9. Inadequate Testing with Production Volumes**
- Test with realistic data volumes
- Identify performance issues before production

**10. No Restart/Recovery Strategy**
- Multi-day loads can fail
- Design checkpoint/restart capability
</common_mistakes>

## Performance Considerations

<performance_tips>
**Volume Estimates:**
- Calculate expected row counts for each dimension and fact
- Estimate data size (rows × row width)
- Plan for 3-5 year growth

**Load Time Estimates:**
- Benchmark with sample data
- Calculate rows/second throughput
- Plan load schedule with buffer time

**Partition Strategy:**
- Partition large fact tables (typically by date)
- Enables parallel loading
- Improves future incremental load and query performance

**Staging Database:**
- Use staging database on fast storage
- Adequate space for extracts, transformations, key maps
- Sufficient memory for sorting and joining

**Parallel Processing:**
- Load independent dimensions in parallel
- Load fact partitions in parallel
- Balance resource utilization (CPU, I/O, memory, network)

**Database Configuration:**
- Increase memory allocation for bulk operations
- Adjust transaction log settings
- Configure appropriate degree of parallelism
</performance_tips>

## Deliverables

<deliverables>
After completing historic load:

**1. Historic Load ETL Processes**
- Dimension load programs (one per dimension)
- Fact load programs (one per fact table)
- Surrogate key assignment processes
- Documented and version-controlled

**2. Load Volume Estimates**
- Actual row counts loaded
- Data sizes (GB/TB)
- Comparison to estimates

**3. Performance Benchmarks**
- Load elapsed times
- Throughput (rows/second)
- Resource utilization metrics

**4. Load Validation Reports**
- Row count reconciliation
- Audit statistics comparison
- RI validation results
- Data quality scorecard

**5. Key Map Tables**
- Natural key to surrogate key mappings
- Maintained in staging database
- Used for incremental processing

**6. Documentation**
- Historic load procedures
- Known issues and resolutions
- Lessons learned
- Recommendations for next business process
</deliverables>

## Transition to Incremental Processing

After successful historic load:

<transition>
**1. Production Release**
- Move fact/dimension tables to production environment
- Enable query access for users
- Activate BI applications

**2. Switch to Incremental ETL**
- Different logic (inserts, updates, deletes)
- Must be fully automated
- Runs within nightly load window
- See **Skill 10: Incremental ETL Processing** for details

**3. Maintain Key Maps**
- Key map tables bridge historic and incremental processing
- Continue using for ongoing surrogate key lookups

**4. Monitor Performance**
- Historic load performance not indicative of incremental
- Benchmark incremental loads separately
- Optimize for daily volumes and time windows
</transition>

## Summary

Historic data loading is a **one-time, bulk operation** that differs significantly from ongoing incremental processing:

- **Load dimensions first**, simplest to most complex
- **Validate hierarchies** and assign surrogate keys
- **Reconstruct Type 2 history** where business requires it
- **Load facts second**, after all dimensions populated
- **Use surrogate key pipeline** to ensure referential integrity
- **Leverage bulk loading techniques** for performance
- **Validate thoroughly** with audit statistics and RI checks
- **Document performance benchmarks** for future reference

Success requires careful planning, adequate time allocation, and comprehensive validation before transitioning to incremental processing.

---

**Next Steps:**
- After historic load: Implement incremental ETL (Skill 10)
- Reference: ETL System Architecture Design (Skill 8)
- Related: Data Profiling and Quality Assessment (Skill 7)

**Source:** Chapter 20, Steps 5-6 (Kimball Data Warehouse Toolkit)
