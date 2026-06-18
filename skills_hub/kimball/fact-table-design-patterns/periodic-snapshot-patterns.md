# Periodic Snapshot Fact Table Patterns - Detailed Guide

## Overview

Periodic snapshot fact tables provide cumulative performance views at regular, predictable intervals. They stack consecutive snapshots over time, creating layers of performance data. This document provides comprehensive guidance on when and how to design periodic snapshots.

## Fundamental Characteristics

### Defining Traits
- **Grain**: Combination of dimensions at specific time interval (one row per product per store per day)
- **Time representation**: Regular intervals (daily, weekly, monthly)
- **Loading pattern**: Insert new snapshot layers, preserve prior snapshots
- **Density**: Dense and predictable coverage
- **Size**: Calculated as dimensions × time periods
- **Dimensionality**: Fewer dimensions than transactions (no causal factors)

### Why Periodic Snapshots Matter
- Provide regular, predictable longitudinal performance trends
- Enable cumulative views difficult from transactions alone
- Capture static levels/balances at points in time
- Easier performance queries than rolling up transactions
- Complement transaction detail with aggregate perspective

## When to Use Periodic Snapshots

### Strong Indicators for Periodic Snapshots

**Need for Cumulative Performance**:
- Business wants regular trend reporting (daily, weekly, monthly)
- Transactions too granular for management reporting
- Need to see performance at consistent time intervals

**Balance/Level Measurements**:
- Inventory quantities on hand
- Financial account balances
- Work-in-process quantities
- Headcount levels
- Equipment utilization rates

**Transaction Complexity**:
```
Problem: 50 transaction types affect inventory
- Receipts, adjustments, cycle counts, transfers, shipments, returns, damages, etc.
- Rolling transactions forward is extremely complex

Solution: Periodic snapshot with daily inventory levels
- Source system already maintains balances
- Extract calculated balance daily
- Avoid complex transaction interpretation logic
```

**Performance Requirements**:
```
Transactions: 2 billion rows
Query: "Show daily sales trend for last quarter by product category"

Without snapshot: Aggregate 2B rows on-the-fly
With daily snapshot: Scan ~90 days of pre-aggregated data

Snapshot dramatically improves query performance
```

### When NOT to Use Periodic Snapshots

```
❌ Single events don't repeat regularly
   → Use transaction fact table

❌ Need transaction-level detail
   → Use transaction fact table (snapshot doesn't replace it)

❌ Process has distinct workflow pipeline
   → Use accumulating snapshot instead

❌ No meaningful balance/level to capture
   → Periodic snapshot may not add value
```

## Detailed Design Process

### 1. Declaring Snapshot Grain

Unlike transactions, snapshot grain combines dimensions and time period.

**Grain Declaration Format**:
"One row per [dimension 1] per [dimension 2] ... per [time period]"

**Examples**:
```
Store Inventory:
"One row per product per store per day"

Financial Account Balances:
"One row per account per month"

Student Enrollment:
"One row per student per course per term"

Website Traffic:
"One row per page per day"

Warehouse Capacity:
"One row per warehouse per week"
```

**Time Period Selection**:
- Daily: Most common, provides flexibility
- Weekly: Reduces volume for less volatile metrics
- Monthly: Financial reporting, low-frequency measurements
- Mixed: Daily for recent, weekly for historical (separate fact tables)

**Grain Validation**:
```
Can you fill in the blank?
"On [date], for [dimension context], the [measure] was [value]"

Example:
"On March 15, 2024, for Product 12345 in Store 88, the quantity on hand was 47 units"
```

### 2. Identifying Snapshot Dimensions

Snapshot dimensions are more limited than transaction dimensions.

**Standard Snapshot Dimensions**:

**Date/Period Dimension** (Always):
- Daily snapshots: Regular date dimension
- Weekly snapshots: Week dimension
- Monthly snapshots: Month dimension
- Grain must match snapshot frequency

**Subject Dimensions**:
- Product (for inventory)
- Account (for financial balances)
- Student (for enrollment)
- Page (for web traffic)
- Facility (for utilization)

**Location/Organization Dimensions**:
- Store, warehouse, branch
- Department, business unit
- Geographic region

**Exclude Causal Dimensions**:
```
❌ Promotion (promotions cause movements, not relevant to balance)
❌ Sales rep (reps create transactions, not relevant to balance)
❌ Transaction type (types cause changes, not relevant to snapshot state)

These belong in transaction fact tables, not snapshots
```

**Dimension Validation Question**:
"Does this dimension make sense when asking 'What was the balance for [dimension] on [date]?'"

### 3. Selecting Snapshot Facts

Snapshot facts fall into two categories: balances and period activity.

**Balance/Level Facts** (Primary):
```
Characteristics:
- Represent state/level at snapshot moment
- Semi-additive (not additive across time)
- Core purpose of snapshot

Examples:
- quantity_on_hand
- account_balance
- active_student_count
- warehouse_capacity_cubic_feet
- inventory_value_at_cost
```

**Period Activity Facts** (Secondary):
```
Characteristics:
- Summarize activity during period
- Fully additive
- Provide context for balance changes

Examples:
- quantity_sold (during day)
- quantity_received (during day)
- deposits_amount (during month)
- withdrawals_amount (during month)
- page_views_count (during day)
```

**Derived Facts**:
```
Store for user convenience:
- extended_cost_value = quantity_on_hand × unit_cost
- extended_retail_value = quantity_on_hand × unit_retail_price
- inventory_turns = quantity_sold / quantity_on_hand (careful: daily vs. period)

Calculate in view if preferred:
- Pro: Reduces storage
- Con: Computation on every query
```

### 4. Handling Semi-Additive Facts

Semi-additive facts are the hallmark of periodic snapshots.

**Understanding Semi-Additivity**:
```
Inventory Example:
Monday quantity_on_hand: 50
Tuesday quantity_on_hand: 50
Wednesday quantity_on_hand: 100 (received shipment)
Thursday quantity_on_hand: 100
Friday quantity_on_hand: 100

Correct weekly aggregations:
✓ SUM across products: 50 + 75 + 25 = 150 total units in all products
✓ SUM across stores: 50 + 30 + 20 = 100 total units in all stores
❌ SUM across dates: 50+50+100+100+100 = 400 (meaningless!)
✓ AVG across dates: (50+50+100+100+100)/5 = 80 average daily inventory
```

**SQL Challenges with Semi-Additive Facts**:
```sql
-- WRONG: SQL AVG divides by row count, not date count
SELECT
    product_category,
    AVG(quantity_on_hand) as avg_inventory
FROM inventory_snapshot
WHERE date_key BETWEEN 20240301 AND 20240331
GROUP BY product_category;

-- Problem: If Category A has 100 products and Category B has 10 products
-- AVG calculation is biased by number of products, not dates

-- CORRECT: Manual calculation
SELECT
    product_category,
    SUM(quantity_on_hand) / COUNT(DISTINCT date_key) as avg_daily_inventory
FROM inventory_snapshot
WHERE date_key BETWEEN 20240301 AND 20240331
GROUP BY product_category;
```

**OLAP Cube Solution**:
- OLAP products support custom aggregation rules
- Configure balance measures to average across time dimension
- Sum across all other dimensions
- Simplifies semi-additive handling for users

**Documentation**:
- Clearly label which facts are semi-additive
- Provide SQL examples for correct averaging
- Train users on semi-additive behavior

### 5. Managing Snapshot Density

**Density Challenge**:
```
Example: Retail store inventory
- 60,000 products
- 100 stores
- Daily snapshots
- Daily rows: 60,000 × 100 = 6 million rows/day
- Annual rows: 6M × 365 = 2.19 billion rows
- Row size: ~14 bytes (3 FKs, 1 fact)
- Annual storage: ~31 GB (manageable)
```

**Density Management Strategies**:

**Strategy 1: Zero Suppression** (Use Carefully):
```
Option: Omit fact rows when quantity = 0

Pros:
- Reduces fact table size if many zeros
- Works for true out-of-stock situations

Cons:
- Must distinguish NULL (no data) from 0 (explicit zero)
- Queries become complex ("show all products with zero inventory")
- Risk: Is missing row out-of-stock or data quality issue?

Recommendation: Only if zeros are rare and explicit zero tracking not needed
```

**Strategy 2: Mixed Periodicity**:
```
Design:
- Daily snapshots for recent period (last 60 days)
- Weekly snapshots for historical data

Implementation:
- Separate fact tables (different grains)
- Union in BI layer if needed
- Reduces rows: 365 daily → 60 daily + 52 weekly = 112 snapshots/year

Example savings:
- 365-day approach: 2.19B rows
- 60+weekly approach: 360M + 312M = 672M rows (~70% reduction)
```

**Strategy 3: Conditional Detail**:
```
Full detail where needed:
- Current products: Daily snapshots
- Discontinued products: Weekly or monthly snapshots

Challenges:
- Product status changes over time
- Complex ETL logic
- User confusion about grain differences
```

**Strategy 4: Aggregate Snapshots**:
```
Complement atomic snapshots with aggregates:
- Daily product/store snapshot (atomic)
- Daily category/region snapshot (aggregate)
- Monthly product/store snapshot (aggregate)

Users select appropriate granularity for query
```

## Advanced Snapshot Patterns

### Beginning and Ending Balances with Deltas

**Extended Balance Pattern**:
```
Facts per row:
- beginning_balance (semi-additive)
- period_activity_delta (fully additive)
- ending_balance (semi-additive)

Relationship:
ending_balance = beginning_balance + period_activity_delta

Benefits:
- Ending balance enables state-at-moment analysis
- Delta enables change analysis
- Beginning balance provides reconciliation
```

**Example: Financial Account**:
```
Facts:
- beginning_balance
- deposits_amount (delta)
- withdrawals_amount (delta)
- interest_earned_amount (delta)
- fees_charged_amount (delta)
- ending_balance

Validation:
ending_balance = beginning_balance + deposits - withdrawals + interest - fees
```

### Enhanced Metrics for Velocity Analysis

**Inventory Velocity Metrics**:
```
Facts enabling inventory turn calculations:
- quantity_on_hand (semi-additive)
- quantity_sold (additive period activity)
- quantity_received (additive period activity)
- extended_cost_value (semi-additive)
- extended_retail_value (semi-additive)

Calculated metrics:
- Daily turns = quantity_sold / quantity_on_hand
- Annual turns = SUM(quantity_sold) / AVG(quantity_on_hand) over year
- Days supply = quantity_on_hand / AVG(quantity_sold per day)
```

### Multiple Snapshot Types for Same Process

**Complementary Inventory Snapshots**:
```
Snapshot 1: Daily Store Inventory
- Grain: Product × Store × Day
- Purpose: Detailed inventory position
- Volume: High

Snapshot 2: Weekly Distribution Center Inventory
- Grain: Product × DC × Week
- Purpose: Replenishment planning
- Volume: Lower

Snapshot 3: Monthly Financial Inventory
- Grain: Category × Region × Month
- Purpose: Financial reporting
- Volume: Lowest
```

**Design Decision**:
- Separate fact tables (different grains)
- Share conformed dimensions
- Each serves distinct business purpose

## Snapshot Loading Patterns

### Standard Load Process

**Daily Snapshot ETL**:
```
1. Extract from source:
   - Current balances/levels as of snapshot date
   - Period activity for snapshot period

2. Transform:
   - Map to dimension surrogate keys
   - Calculate derived facts
   - Apply business rules

3. Load:
   - Insert new snapshot layer
   - Never update prior snapshots
   - Partition by snapshot date for performance

4. Validate:
   - Row count matches expected (dimensions product)
   - Balance totals reconcile to control reports
   - No NULL foreign keys
```

### Late-Arriving Facts

**Challenge**:
```
Problem: Snapshot loaded for March 15, then late transactions discovered
- Late transactions affect March 15 balance
- March 15 snapshot already published to users

Options:
1. Restate March 15 snapshot (update fact rows)
2. Leave snapshot as-is (reflects data known at time)
3. Version snapshots (keep both original and restated)
```

**Recommendation**:
- Operational systems should provide point-in-time balances
- DW/BI extracts system's calculated balance
- Don't recalculate balances in DW/BI
- If restatement needed, update snapshot and document

### Snapshot Retention Strategy

**Tiered Retention**:
```
Tier 1: Last 13 months
- Granularity: Daily
- Purpose: Operational reporting, recent trends
- Location: High-performance storage

Tier 2: 14-36 months
- Granularity: Weekly (roll up daily)
- Purpose: Historical comparisons
- Location: Standard storage

Tier 3: 37+ months
- Granularity: Monthly (roll up weekly)
- Purpose: Long-term trending, regulatory
- Location: Archive storage
```

## Real-World Examples

### Store Inventory Snapshot
```
Source: Chapter 4 - Retail Inventory
Grain: One row per product per store per day
Dimensions: Date, product, store
Facts:
- quantity_on_hand (semi-additive)
- quantity_sold (additive)
- extended_cost_value (semi-additive)
- extended_retail_value (semi-additive)

Business value:
- Daily inventory positions
- Out-of-stock analysis
- Inventory turn calculations
- Shrinkage detection
```

### Financial Account Balances
```
Grain: One row per account per month-end
Dimensions: Month, account, customer, branch, account type
Facts:
- beginning_balance (semi-additive)
- deposits_amount (additive)
- withdrawals_amount (additive)
- interest_amount (additive)
- fees_amount (additive)
- ending_balance (semi-additive)

Business value:
- Monthly statements
- Average daily balance calculations
- Account activity trends
- Customer relationship profitability
```

### Student Enrollment Snapshot
```
Grain: One row per student per term
Dimensions: Term, student, program, enrollment status
Facts:
- enrolled_credit_hours (additive across students)
- cumulative_gpa (non-additive, must weight by credit hours)
- credits_completed_to_date (additive across students)

Business value:
- Enrollment trends over time
- Student progression tracking
- Program size analysis
```

### Web Traffic Daily Snapshot
```
Grain: One row per page per day
Dimensions: Date, page, site section
Facts:
- unique_visitors (additive across pages, not across days)
- page_views (fully additive)
- avg_time_on_page_seconds (non-additive)
- bounce_rate_percent (non-additive)

Business value:
- Daily traffic patterns
- Page performance trends
- Content effectiveness
```

### Warehouse Capacity Snapshot
```
Grain: One row per warehouse per week
Dimensions: Week, warehouse, product category
Facts:
- available_capacity_cubic_feet (semi-additive)
- utilized_capacity_cubic_feet (semi-additive)
- utilization_percent (non-additive, calculate as utilized/available)

Business value:
- Capacity planning
- Utilization trends
- Expansion decision support
```

## Snapshot Performance Optimization

### Partitioning Strategy
```
Partition by snapshot date:
- Monthly partitions for daily snapshots
- Yearly partitions for weekly/monthly snapshots

Benefits:
- Query pruning (only scan relevant time periods)
- Fast loading (load to current partition)
- Easy archiving (drop/archive old partitions)
```

### Aggregate Snapshots
```
Build aggregate snapshots from atomic:
Daily Product/Store → Daily Category/Region
Daily Product/Store → Weekly Product/Store
Daily Product/Store → Monthly Product/Store

Aggregate snapshots:
- Dramatically faster query performance
- 1000× reduction in rows scanned
- Pre-joined dimension hierarchies
- Still drill to atomic detail when needed
```

### Indexing
```
Dimension foreign keys:
- Bitmap indexes for low-to-moderate cardinality
- Concatenated indexes for common filter combinations

Skip indexing:
- Date key if partitioned by date
- Facts (never filter on facts)
```

## Validation Checklist

Before finalizing periodic snapshot:

**Grain**:
- [ ] Grain clearly specifies dimensions + time period
- [ ] Time period matches business reporting needs
- [ ] Grain density calculated and acceptable

**Dimensions**:
- [ ] Date/period dimension matches snapshot frequency
- [ ] Subject dimensions at appropriate level
- [ ] No causal dimensions included
- [ ] Dimensions conform to bus matrix

**Facts**:
- [ ] Balance/level facts identified as semi-additive
- [ ] Period activity facts included where relevant
- [ ] Derived facts calculated or stored appropriately
- [ ] Fact additivity clearly documented

**ETL**:
- [ ] Source system provides point-in-time balances
- [ ] Load process inserts snapshots, doesn't update
- [ ] Partition strategy defined
- [ ] Retention/archiving strategy planned

**Complementary Fact Tables**:
- [ ] Identified transaction fact table for detail
- [ ] Determined if accumulating snapshot also needed
- [ ] Documented relationship between fact tables

**Performance**:
- [ ] Row count estimated
- [ ] Storage calculated
- [ ] Aggregate strategy considered
- [ ] Query performance validated
