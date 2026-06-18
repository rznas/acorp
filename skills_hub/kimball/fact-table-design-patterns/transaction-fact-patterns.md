# Transaction Fact Table Patterns - Detailed Guide

## Overview

Transaction fact tables are the most fundamental view of business operations. They capture individual events at a specific point in time with maximum dimensional detail. This document provides comprehensive guidance on designing robust transaction fact tables.

## Fundamental Characteristics

### Defining Traits
- **Grain**: One row per transaction or transaction line item
- **Time representation**: Single event date (point in time)
- **Loading pattern**: Insert only, never updated after posting
- **Density**: Sparse and unpredictable
- **Size**: Often the largest tables (billions/trillions of rows)
- **Dimensionality**: Highly dimensional (typically 10-20 dimensions)

### Why Transaction Fact Tables Matter
- Provide maximum analytic flexibility
- Enable drill-down to individual event details
- Support ad-hoc analysis in unpredictable ways
- Foundation for aggregated views and snapshots
- Most naturally dimensional data in organization

## Detailed Design Process

### 1. Declaring Atomic Grain

The grain is the single most important decision in fact table design.

**Grain Declaration Format**:
"One row per [business event] in the context of [dimensional scope]"

**Examples**:
- "One row per product scanned on a POS transaction"
- "One row per line item on a customer order"
- "One row per call detail record in telecommunications"
- "One row per individual boarding pass scanned at airport gate"
- "One row per test result for a lab specimen"

**Common Grain Mistakes**:
```
Too aggregated:
❌ "Daily product sales by store"
✓ "Individual product scan on POS transaction"

Mixing grains:
❌ "Sales and returns" (different events)
✓ Separate fact tables for sales and returns

Unclear grain:
❌ "Customer transactions"
✓ "One row per line item on a customer order"
```

**Grain Validation Questions**:
1. Can you describe a single fact table row in business terms?
2. Does every team member agree on what one row represents?
3. Is this the most atomic level captured by the source system?
4. Would further decomposition provide meaningful additional detail?

### 2. Identifying Dimensions

Dimensions fall naturally from the grain declaration.

**Dimension Discovery Questions**:
- **Who**: Customers, employees, sales reps, cashiers
- **What**: Products, services, line items
- **Where**: Stores, warehouses, facilities, locations
- **When**: Transaction date, requested date, promised date
- **Why**: Promotions, deals, contracts, campaigns
- **How**: Payment methods, delivery methods, channels

**Standard Dimension Patterns**:

**Date Dimension** (Always present):
- Primary: Transaction date (when event occurred)
- Secondary: Requested date, ship date (role-playing dimensions)
- Use views/aliases for multiple date roles

**Product Dimension**:
- Individual SKU level (most granular)
- Merchandise hierarchies flattened
- Multiple independent hierarchies supported
- 50-100+ attributes typical

**Customer/Location Dimension**:
- Ship-to location or customer identifier
- Multiple geographic hierarchies
- Organizational hierarchies
- Demographic attributes

**Causal Dimensions** (Why event occurred):
- Promotions, deals, contracts
- Combination of correlated factors
- Example: Price reduction + ad + display = single promotion row

**Degenerate Dimensions**:
- Transaction control numbers without dimension table
- Order numbers, invoice numbers, ticket numbers
- Sit in fact table without join
- Enable grouping of line items
- Often part of primary key

### 3. Selecting Transaction Facts

**Fact Selection Criteria**:
1. Must be true to declared grain
2. Prefer additive measurements
3. Store extended amounts (not unit rates)
4. Include derived facts if frequently calculated

**Core Transaction Facts**:

**Quantity Facts**:
```
sales_quantity
return_quantity
order_quantity
shipped_quantity
```

**Extended Amount Facts** (Quantity × Unit Rate):
```
extended_sales_amount = sales_quantity × unit_price
extended_discount_amount = sales_quantity × unit_discount
extended_cost_amount = sales_quantity × unit_cost
extended_gross_profit = extended_sales_amount - extended_cost_amount
```

**Why Extended Amounts?**:
- Fully additive across all dimensions
- Eliminates calculation errors
- Consistent across all BI tools
- Supports ratio calculations (sum of extended amounts / sum of quantities)

**Derived Facts Storage Decision**:
```
Store gross_profit physically if:
✓ Frequently used by many users
✓ Calculation is standard and agreed upon
✓ Want to eliminate calculation errors
✓ All BI tools reference it

Cost is minimal:
- Single additional numeric column
- Risk of incorrect calculation >> storage cost
```

### 4. Handling Non-Additive Facts

**Unit Price/Rate Facts**:
- Non-additive (cannot sum meaningfully)
- Store if needed for filtering/display
- Never use for totaling
- Calculate weighted averages: SUM(extended_amount) / SUM(quantity)

**Percentage/Ratio Facts**:
- Store numerator and denominator separately
- Calculate ratio in BI layer
- Formula: Ratio of sums, NOT sum of ratios
- Example: Margin = SUM(profit) / SUM(revenue), not AVG(margin)

**Temperature/Intensity Measurements**:
- Non-additive static levels
- Can average carefully if business agrees
- Usually not appropriate for summing

## Advanced Transaction Patterns

### Factless Fact Tables

Factless fact tables capture event relationships without numeric measurements.

**Use Cases**:
- Coverage/eligibility tracking
- Attendance/enrollment recording
- Event occurrence without measures

**Pattern 1: Coverage/Promotion**:
```
Schema: Promotion Coverage
Grain: One row per product on promotion per store per day
Dimensions: Date, product, store, promotion
Facts: promotion_count (constant 1 for counting)

Purpose: Track what was promoted regardless of sales
Analysis: What products were promoted but didn't sell? (set difference with sales)
```

**Pattern 2: Enrollment/Attendance**:
```
Schema: Course Registration
Grain: One row per student per course per term
Dimensions: Date, student, course, instructor, classroom
Facts: registration_count (constant 1)

Purpose: Track registrations independent of grades
Analysis: Class sizes, student course loads, instructor assignments
```

**Pattern 3: Sales Rep Coverage**:
```
Schema: Customer Coverage
Grain: One row per sales rep assignment per customer per time period
Dimensions: Assignment date, expiration date, sales rep, customer
Facts: coverage_count (constant 1)

Purpose: Complete map of rep-to-customer assignments over time
Analysis: Identify assignments with no resulting sales activity
```

### Junk Dimensions for Transaction Indicators

**When to Use Junk Dimensions**:
- Multiple low-cardinality flags/indicators (2-10 values each)
- Flags are transaction-specific
- Don't want clutter in fact table
- Indicators take on every combination

**Design Approach**:
```
Scenario: Order transaction has 6 boolean flags
- Order type (3 values): Inbound, Outbound, Return
- Priority (2 values): Normal, Rush
- Gift wrap (2 values): Yes, No
- Insurance (2 values): Yes, No
- Signature required (2 values): Yes, No
- Weekend delivery (2 values): Yes, No

Junk dimension rows: 3 × 2 × 2 × 2 × 2 × 2 = 96 possible combinations

Build junk dimension:
Option A: Pre-populate all 96 combinations
Option B: Add rows as combinations encountered (if low occurrence)
```

**Junk Dimension Benefits**:
- Single FK in fact table vs. 6 separate dimension FKs
- Useful framework for filtering/grouping
- Reduces fact table width
- Eliminates multiple tiny dimensions

**When NOT to Use**:
```
Bad for junk dimension:
- 5 indicators with 100 values each = 10 billion combinations
- Solution: Create separate dimensions for uncorrelated, high-cardinality attributes
```

### Multiple Currencies in Transactions

**Dual Currency Pattern** (Most Common):
```
For each monetary fact, store:
- extended_local_currency_amount
- extended_corporate_currency_amount (e.g., USD)

Add dimension:
- local_currency_key (FK to currency dimension)

Conversion:
- Applied at transaction moment using ETL business rules
- Corporate currency amounts fully additive
- Local currency amounts additive only within single currency
```

**Multi-Currency with Conversion Table**:
```
Use when:
- Need to support any-to-any currency conversion
- Conversion rates change after transaction posting
- Multiple rate types (daily, month-end, quarter-end)

Design:
- Transaction fact table stores base currency
- Currency conversion fact table: daily rates between currency pairs
- BI application joins to conversion table for ad-hoc conversions
```

### Header/Line Item Handling

**Preferred Pattern: Allocate to Line Grain**:
```
Problem: Shipping charge applies to order header, not individual lines

Solution: Allocate header facts down to line items
- Get organizational agreement on allocation rules
- Example: Allocate freight by line item gross amount
- All facts now at common grain
- Can analyze by product dimension

Fact table includes:
- extended_line_gross_amount (line level)
- allocated_freight_amount (header allocated to line)
- allocated_order_discount_amount (header allocated to line)
```

**Alternative: Separate Fact Tables** (If allocation impossible):
```
Order Line Fact Table:
- Grain: One row per line item
- Facts: Line-level amounts
- Dimensions: Include product

Order Header Fact Table:
- Grain: One row per order
- Facts: Header-level amounts
- Dimensions: Exclude product

Warning: Cannot analyze header facts by product
```

**Patterns to Absolutely Avoid**:
```
❌ Repeat unallocated header fact on every line (overcounting)
❌ Store header fact only on first/last line (undercounting if filtered)
❌ Use special product key (99999) for header facts (decoder ring required)
```

### Degenerate Dimension Handling

**Standard Usage**:
```
Degenerate dimensions for:
- Order numbers, invoice numbers, ticket numbers
- Transaction control numbers from operational systems
- POS transaction receipts, bill-of-lading numbers

Characteristics:
- Stored directly in fact table (no dimension table join)
- No descriptive attributes remain after other dimensions created
- Often part of fact table primary key
- Enable grouping (e.g., all line items on an order)
```

**When Degenerate Becomes Real Dimension**:
```
If header has attributes that don't fit elsewhere:
- Create proper dimension table
- Assign surrogate key
- No longer degenerate

But challenge assumptions:
- Does header info really belong in another analytic dimension?
- Example: Order date → Date dimension, not order header dimension
```

**Surrogate Keys for Degenerate Dimensions**:
```
Assign surrogate key if:
✓ Transaction numbers not unique across locations
✓ Transaction numbers get recycled/reused
✓ Transaction number is bulky (24-character alphanumeric)
✓ Need to drill-across on transaction number (tool limitation)

Creates small dimension table with:
- Surrogate key (PK)
- Natural transaction number
- System source identifier (if from multiple systems)
```

## Transaction Fact Table Sizing

### Row Count Estimation

**Method 1: Source System Counts**:
- Query operational database for daily/monthly transaction volumes
- Account for seasonality and growth

**Method 2: Business Volume Calculation**:
```
Example: Retail grocery chain
Annual revenue: $4 billion
Average item price: $2.00
Estimated annual transactions: $4B / $2 = 2 billion rows/year

Daily load: 2B / 365 = ~5.5 million rows/day
```

**Method 3: Per-Event Multiplication**:
```
Example: Order management
Orders per day: 50,000
Average lines per order: 4
Daily rows: 50,000 × 4 = 200,000 rows/day
Annual rows: 200,000 × 365 = 73 million rows/year
```

### Storage Estimation

**Fact Table Row Size**:
```
Example: POS transaction line item
- 7 dimension FKs × 4 bytes = 28 bytes
- 1 degenerate dimension × 20 bytes = 20 bytes
- 4 facts × 8 bytes = 32 bytes
- Total row size: 80 bytes

Annual storage (5.5M rows/day):
- Rows: 5.5M × 365 = 2 billion
- Storage: 2B × 80 bytes = 160 GB
- With indexes, overhead: ~300-400 GB
```

## Transaction Performance Optimization

### Partitioning Strategy

**Date-Based Partitioning** (Most Common):
```
Partition on date key (YYYYMMDD format)
- Monthly partitions for most businesses
- Weekly partitions for very high volume
- Daily partitions if needed for extremely high volume

Benefits:
- Query pruning (scan only relevant partitions)
- Fast data loading (load to current partition)
- Easy archiving (drop old partitions)
- Efficient backup/recovery
```

### Indexing Strategy

**Bitmap Indexes** (For low-cardinality dimensions):
```
Good candidates:
- Date key (even though high cardinality, commonly filtered)
- Store key (100-1000 distinct values)
- Product category (50-100 values)
- Promotion key (1,000s of values)

Benefits:
- Efficient for AND/OR combinations
- Low storage overhead
- Fast bitmap operations
```

**B-tree Indexes** (For high-cardinality):
```
Consider for:
- Degenerate dimensions (transaction numbers)
- Very high cardinality dimensions if frequently filtered alone

Use sparingly:
- Each index adds load time
- Only index frequently filtered columns
```

## Real-World Examples

### Retail POS Sales
```
Grain: One row per product per POS transaction
Dimensions: Date, product, store, promotion, cashier, payment method, transaction# (DD)
Facts: sales_quantity, extended_sales_amount, extended_discount_amount, extended_cost_amount
Size: 2 billion rows/year for large grocery chain
```

### E-commerce Clickstream
```
Grain: One row per page view event
Dimensions: Date, time of day, session, page, referrer, customer
Facts: page_view_count (1), duration_seconds
Size: Billions of rows for major e-commerce sites
```

### Telecommunications Call Detail
```
Grain: One row per phone call
Dimensions: Call date/time, calling number, called number, cell tower, service plan
Facts: call_duration_seconds, call_charge_amount
Size: Hundreds of millions to billions per day for carriers
```

### Financial Services Transactions
```
Grain: One row per account transaction
Dimensions: Transaction date, account, transaction type, branch, teller
Facts: transaction_amount, beginning_balance, ending_balance
Size: Millions to billions depending on institution size
```

## Validation Checklist

Before finalizing transaction fact table:

**Grain**:
- [ ] Grain stated in clear business terms
- [ ] Grain is most atomic level available
- [ ] Grain agreed upon by all stakeholders

**Dimensions**:
- [ ] All "who, what, where, when, why, how" identified
- [ ] Date dimension included
- [ ] Degenerate dimensions for transaction IDs
- [ ] No dimensions violate grain

**Facts**:
- [ ] All facts true to grain
- [ ] Extended amounts used (not unit rates)
- [ ] Additivity documented for each fact
- [ ] Derived facts stored if frequently used
- [ ] Non-additive facts handled appropriately

**Performance**:
- [ ] Row count estimated
- [ ] Storage sizing calculated
- [ ] Partitioning strategy defined
- [ ] Index strategy planned

**Integration**:
- [ ] Conforms to bus matrix
- [ ] Uses conformed dimensions
- [ ] Complements other fact tables if needed
