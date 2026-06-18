# Fact Additivity Guide - Complete Reference

## Overview

Fact additivity determines how facts can be aggregated across dimensions. Understanding additivity is critical for proper fact table design and avoiding common calculation errors. This guide provides comprehensive coverage of additive, semi-additive, and non-additive facts.

## The Three Types of Additivity

### Summary Table

| Type | Definition | Example Facts | Aggregation |
|------|------------|---------------|-------------|
| **Additive** | Can sum across ALL dimensions | sales_quantity, extended_sales_amount, cost | SUM across all dimensions |
| **Semi-Additive** | Can sum across SOME dimensions, not time | account_balance, inventory_on_hand, headcount | SUM across non-time dimensions, AVG across time |
| **Non-Additive** | Cannot sum across ANY dimension | unit_price, ratios, percentages, temperatures | Store components, calculate in BI layer |

## Additive Facts - Complete Guide

### Definition
Additive facts can be summed across ALL dimensions to produce meaningful totals.

### Characteristics
```
Properties:
✓ Numeric measurements that accumulate
✓ Each fact row contributes independently to total
✓ Summation across any dimension is meaningful
✓ Most common and preferred fact type
```

### Examples of Additive Facts

**Quantity Facts**:
```
sales_quantity
- Sum across products: Total items sold
- Sum across stores: Total items sold
- Sum across dates: Total items sold
- Sum across all dimensions: Total items sold

return_quantity
shipped_quantity
received_quantity
manufactured_quantity
```

**Extended Amount Facts**:
```
extended_sales_amount = sales_quantity × unit_sales_price

Additive across all dimensions:
- Sum across products: Total revenue by store, date
- Sum across stores: Total revenue by product, date
- Sum across dates: Total revenue by product, store
- Sum across all: Total revenue period

Other extended amounts:
- extended_discount_amount
- extended_cost_amount
- extended_gross_profit_amount
```

**Duration Facts**:
```
call_duration_seconds
- Sum across calls: Total talk time
- Sum across agents: Total talk time handled
- Sum across dates: Total talk time per day

page_view_duration_seconds
session_duration_minutes
process_duration_hours
```

**Count Facts**:
```
transaction_count (constant 1 per row)
- Sum across any dimension: Total transactions

line_item_count
customer_visit_count
page_view_count
```

### Design Principles for Additive Facts

**Store Extended Amounts, Not Unit Rates**:
```
✓ Correct: Store extended_sales_amount = quantity × unit_price
❌ Wrong: Store only unit_price

Why extended amounts:
1. Fully additive across all dimensions
2. No calculation errors
3. Easy to calculate weighted averages: SUM(extended) / SUM(quantity)
4. Consistent across all BI tools

Example:
Product A: 10 units @ $5.00 = $50.00 extended
Product B: 20 units @ $3.00 = $60.00 extended

Total revenue: $50 + $60 = $110 (correct, additive)
Weighted avg price: $110 / 30 units = $3.67 (correct calculation)

If only unit prices stored:
Incorrect avg: ($5.00 + $3.00) / 2 = $4.00 (wrong! Not weighted)
```

**Store Derived Facts for Common Calculations**:
```
If frequently calculated, store in fact table:
✓ extended_gross_profit = extended_sales_amount - extended_cost_amount

Benefits:
- Eliminates calculation errors
- Ensures consistent definition
- Faster queries (no calculation needed)

Cost:
- Minimal (one additional numeric column)
- Much less than risk of calculation errors
```

**Preserve Detail Over Aggregation**:
```
Store facts at most granular level:
✓ Store individual line item amounts
❌ Don't pre-aggregate to order total (loses dimensionality)

Aggregation is query-time operation, not storage-time
```

### Validation: Is My Fact Additive?

**Test Questions**:
```
1. Does summing across products make sense?
   Example: SUM(sales_quantity) across all products = total units sold ✓

2. Does summing across locations make sense?
   Example: SUM(sales_quantity) across all stores = total units sold ✓

3. Does summing across time make sense?
   Example: SUM(sales_quantity) across all days = total units sold ✓

4. Is each fact row independent?
   Example: One sale doesn't affect another sale's quantity ✓

If all YES: Fact is additive
```

## Semi-Additive Facts - Complete Guide

### Definition
Semi-additive facts can be summed across SOME dimensions but not all. Typically, they cannot be summed across the time dimension.

### Why Semi-Additive Facts Exist

**Represent Balances/Levels at Point in Time**:
```
Characteristics:
- Snapshot of a static level or balance
- Not a flow or movement
- Represents state at a moment
- Summing across time is meaningless (double-counting)

Common in: Periodic snapshot fact tables
```

### Examples of Semi-Additive Facts

**Inventory Balances**:
```
quantity_on_hand

Additive across:
✓ Products: Sum inventory of all products = total inventory units
✓ Stores: Sum inventory across all stores = total inventory units

NOT additive across:
❌ Time: Cannot sum Monday's inventory + Tuesday's inventory
  - Monday: 50 units on hand
  - Tuesday: 50 units on hand (same 50 units!)
  - SUM across days: 50 + 50 = 100 (WRONG, double-counting)
  - Correct: Average = (50 + 50) / 2 = 50 average daily inventory
```

**Financial Account Balances**:
```
account_balance_amount

Example: Checking account
- Monday: $100 balance
- Tuesday: $100 balance
- Wednesday: $200 balance (deposit)
- Thursday: $200 balance
- Friday: $200 balance

Correct aggregations:
✓ Sum across accounts: Total balance in all accounts
❌ Sum across days: $100+$100+$200+$200+$200 = $800 (WRONG!)
✓ Average across days: ($100+$100+$200+$200+$200) / 5 = $160 average daily balance
```

**Other Semi-Additive Examples**:
```
headcount (number of employees on a date)
- Sum across departments: Total employees
- Average across dates: Average headcount

work_in_process_quantity
equipment_utilization_percent (if stored as level)
warehouse_capacity_occupied
open_order_backlog_quantity
```

### Handling Semi-Additive Facts Correctly

**Averaging Across Time**:
```
Correct approach: Calculate average over time periods

SQL example - Average daily inventory:
SELECT
    product_key,
    store_key,
    SUM(quantity_on_hand) / COUNT(DISTINCT date_key) as avg_daily_inventory
FROM inventory_snapshot
WHERE date_key BETWEEN 20240301 AND 20240331
GROUP BY product_key, store_key;

Why COUNT(DISTINCT date_key)?
- Ensures division by number of time periods, not row count
- If 100 products × 31 days = 3,100 rows
- Want to divide by 31 (days), not 3,100 (rows)
```

**Common SQL Mistake**:
```
❌ WRONG: Using SQL AVG function for semi-additive time aggregation

SELECT
    product_category,
    AVG(quantity_on_hand) as avg_inventory
FROM inventory_snapshot
WHERE date_key BETWEEN 20240301 AND 20240331
GROUP BY product_category;

Problem:
- AVG divides by row count, not date count
- Category with 100 products and Category with 10 products
- Result is biased by number of products, not an accurate time average

✓ CORRECT: Manual calculation
SELECT
    product_category,
    SUM(quantity_on_hand) / COUNT(DISTINCT date_key) as avg_daily_inventory
FROM inventory_snapshot
WHERE date_key BETWEEN 20240301 AND 20240331
GROUP BY product_category;
```

**Beginning/Ending Balance Pattern**:
```
Store three related facts:
- beginning_balance (semi-additive)
- period_activity_delta (fully additive)
- ending_balance (semi-additive)

Relationship:
ending_balance = beginning_balance + period_activity_delta

Benefits:
- Ending balance for point-in-time analysis
- Delta for change analysis (fully additive!)
- Beginning balance for reconciliation
```

**OLAP Cube Configuration**:
```
OLAP products support custom aggregation rules:
- Configure balance measures to AVERAGE across time dimension
- Configure balance measures to SUM across all other dimensions

In OLAP metadata:
- Set quantity_on_hand aggregation:
  - Time dimension: AVERAGE
  - All other dimensions: SUM

Users then query normally; cube applies correct aggregation
```

### Design Principles for Semi-Additive Facts

**Document Semi-Additivity**:
```
Critical for user understanding:
1. Clearly label semi-additive facts in metadata
2. Provide SQL examples for correct time averaging
3. Train users on semi-additive behavior
4. Include BI tool-specific guidance

Documentation example:
"quantity_on_hand is semi-additive. Sum across products and stores.
For time-based aggregation, use: SUM(quantity_on_hand) / COUNT(DISTINCT date_key)"
```

**Consider Additive Alternatives**:
```
If possible, complement semi-additive balance with additive delta:
- quantity_on_hand (semi-additive)
- quantity_change_during_period (fully additive)

Enables both balance and movement analysis
```

**Validate in Context**:
```
Test: Does your fact represent a balance/level at a point in time?
✓ YES: Likely semi-additive
  - Inventory levels, account balances, headcount, WIP

✗ NO: Should be additive
  - Quantities sold, amounts deposited, employees hired
```

## Non-Additive Facts - Complete Guide

### Definition
Non-additive facts cannot be meaningfully summed across ANY dimension.

### Why Non-Additive Facts Exist

**Represent Ratios, Rates, or Intensities**:
```
Characteristics:
- Not measurements that accumulate
- Summing produces meaningless results
- Often derived from other facts
- Averaging may or may not make sense

Typical examples:
- Unit prices (rates)
- Percentages (ratios)
- Temperatures (intensities)
- Scores/grades (ordinal values)
```

### Examples of Non-Additive Facts

**Unit Prices/Rates**:
```
unit_sales_price

Cannot sum:
❌ Product A: $5.00 unit price
❌ Product B: $3.00 unit price
❌ Sum: $5.00 + $3.00 = $8.00 (meaningless!)

Can average (carefully):
- Simple average: ($5.00 + $3.00) / 2 = $4.00
- Weighted average: SUM(extended_amount) / SUM(quantity)

Storage decision:
- Store if needed for filtering, display, or weighted average calculation
- Also store extended amounts (additive)
```

**Percentage/Ratio Facts**:
```
gross_margin_percent = (sales - cost) / sales × 100

Cannot sum:
❌ Product A: 40% margin
❌ Product B: 30% margin
❌ Sum: 40% + 30% = 70% (meaningless!)

Cannot average directly:
❌ Simple average: (40% + 30%) / 2 = 35% (WRONG!)
  - Doesn't account for sales volume differences

Correct calculation:
✓ Ratio of sums: (SUM(sales) - SUM(cost)) / SUM(sales) × 100
✓ Store numerator and denominator separately:
  - extended_gross_profit_amount (sales - cost)
  - extended_sales_amount
  - Calculate ratio in BI layer
```

**Temperature/Intensity Measurements**:
```
room_temperature_fahrenheit

Cannot sum:
❌ Room A: 72°F
❌ Room B: 68°F
❌ Sum: 140°F (meaningless!)

Can average (if business agrees):
✓ Average: (72°F + 68°F) / 2 = 70°F

But consider:
- Does average temperature make sense for business question?
- Or do they want min/max/distribution?
```

**Scores/Ratings**:
```
customer_satisfaction_score (1-10 scale)

Cannot sum meaningfully
May average: AVG(satisfaction_score)

Alternative: Store as facts and derive metrics:
- Promoters (9-10) count
- Passives (7-8) count
- Detractors (1-6) count
- Calculate Net Promoter Score = (Promoters - Detractors) / Total
```

### Handling Non-Additive Facts

**Store Component Facts, Not Derived Ratios**:
```
✓ Correct: Store numerator and denominator
- extended_gross_profit_amount
- extended_sales_amount
- Calculate gross_margin_percent in BI layer: profit / sales × 100

Benefits:
- Components are additive (can roll up correctly)
- Ratio calculation always accurate (ratio of sums, not sum of ratios)
- No risk of incorrect averaging

❌ Incorrect: Store only gross_margin_percent
- Cannot roll up accurately
- Users may incorrectly average percentages
```

**Unit Price Storage Decisions**:
```
When to store unit price:
✓ If needed for filtering ("Show products priced $10-$20")
✓ If needed for display on reports
✓ If source system provides it

Also store:
✓ extended_amount = quantity × unit_price (additive)
✓ quantity (additive)

Calculate weighted average:
price = SUM(extended_amount) / SUM(quantity)
```

**Temperature/Intensity Facts**:
```
Storage:
- Store individual measurements
- Let BI tools calculate AVG, MIN, MAX, STDDEV as needed

Question validity:
- Does "average temperature" answer business questions?
- Or do they need distribution analysis, outlier detection?
```

### Design Principles for Non-Additive Facts

**Ratio of Sums, Not Sum of Ratios**:
```
Fundamental rule:
✓ CORRECT: Calculate ratios from aggregated components
  gross_margin = SUM(gross_profit) / SUM(sales)

❌ WRONG: Aggregate pre-calculated ratios
  avg_gross_margin = AVG(gross_margin_percent)

Example showing the error:
Product A: $100 sales, $40 profit = 40% margin, 100 units sold
Product B: $900 sales, $270 profit = 30% margin, 900 units sold

Wrong avg margin: (40% + 30%) / 2 = 35%
Correct margin: ($40 + $270) / ($100 + $900) = 31%

The simple average incorrectly weights products equally
```

**Document Non-Additivity**:
```
Metadata must include:
1. Flag fact as non-additive
2. Explain why (ratio, rate, intensity)
3. Provide correct calculation formula
4. Identify component facts if applicable

Example documentation:
"gross_margin_percent is non-additive. Calculate as:
(SUM(extended_gross_profit) / SUM(extended_sales_amount)) × 100.
Do not average gross_margin_percent across rows."
```

**Consider Fact Table Exclusion**:
```
Question: Should this non-additive fact be in fact table?

Exclude if:
- Easily calculated from stored additive components
- Not used for filtering
- Rarely displayed

Include if:
- Needed for filtering/constraining
- Frequently displayed
- Calculation is complex
- Want to enforce consistent calculation

Many non-additive facts can be views on fact table
```

## Special Cases and Edge Cases

### Distinct Count Facts

**Unique Customer Count**:
```
Challenge: Count distinct customers across dimensions

Not truly additive:
- Store A: 100 unique customers
- Store B: 150 unique customers
- Cannot sum to 250 (may have overlap)

Solutions:
1. Calculate DISTINCT COUNT in BI layer from transaction detail
2. Use factless fact table with one row per customer per dimension
3. Use approximate distinct count algorithms (HyperLogLog)
```

### Percentage Facts That ARE Additive

**Market Share as Extended Value**:
```
Scenario: Each product row has % of total market

If percentage represents allocation:
✓ extended_market_share_value = total_market_value × share_percent

This is additive:
- Product A: 40% of $1M market = $400k
- Product B: 30% of $1M market = $300k
- Product C: 30% of $1M market = $300k
- Sum: $400k + $300k + $300k = $1M total market ✓
```

### Sparse Additive Facts

**Optional Facts**:
```
Facts that don't apply to all rows:
- freight_amount (only on certain order types)
- returned_quantity (only on returns)

Storage:
✓ NULL for inapplicable values (not zero)

Aggregation:
- SUM ignores NULLs (correct behavior)
- AVG ignores NULLs (correct behavior)
- COUNT counts only non-NULL values

If store zero instead of NULL:
❌ Skews averages
❌ Inflates counts
❌ Makes it harder to identify missing vs. zero
```

## Practical Decision Framework

### Choosing Fact Additivity

**Decision Tree**:
```
Question 1: Does this fact represent accumulated amounts/quantities?
├─ YES: Likely additive
│   └─ Store extended amounts (quantity × rate)
└─ NO: Go to Question 2

Question 2: Does this fact represent a balance/level at a point in time?
├─ YES: Semi-additive
│   └─ Document time averaging requirement
└─ NO: Go to Question 3

Question 3: Is this fact a ratio, rate, or intensity?
├─ YES: Non-additive
│   └─ Store components, calculate ratio in BI layer
└─ NO: Review fact definition, may be misclassified
```

### Validation Tests

**Test for Additivity**:
```
1. Create sample data (3 products, 2 stores, 3 days)
2. Calculate aggregates:
   - SUM across products
   - SUM across stores
   - SUM across days
3. Validate each aggregation makes business sense
4. If any aggregation is meaningless: Not fully additive
5. If only time aggregation fails: Semi-additive
6. If all aggregations fail: Non-additive
```

## Documentation Templates

### Fact Table Documentation Template

```
Fact: quantity_on_hand
Type: Semi-additive
Description: Snapshot of inventory quantity at end of day
Additive across: Product, Store, Warehouse
NOT additive across: Date (must average)
Correct time aggregation: SUM(quantity_on_hand) / COUNT(DISTINCT date_key)
Units: Integer count of units
Source: Inventory system daily snapshot extract
---
Fact: extended_sales_amount
Type: Additive
Description: Total sales amount for line item (quantity × unit price)
Additive across: All dimensions
Calculation: sales_quantity × unit_sales_price
Units: Dollars and cents (numeric 18,2)
Source: POS transaction system
---
Fact: gross_margin_percent
Type: Non-additive
Description: Gross margin percentage (for display/filtering only)
NOT additive across: Any dimension
Correct aggregation: (SUM(extended_gross_profit) / SUM(extended_sales_amount)) × 100
Display format: Percentage with 2 decimal places
Component facts: extended_gross_profit, extended_sales_amount
Source: Calculated in ETL
```

## Summary: Best Practices

### Design-Time Decisions

**Prefer Additive Facts**:
```
✓ Design for additivity whenever possible
✓ Store extended amounts (quantity × rate)
✓ Store derived additive facts if frequently used
✓ Avoid storing non-additive ratios if components available
```

**Handle Semi-Additive Carefully**:
```
✓ Clearly document which dimensions are non-additive
✓ Provide SQL examples for correct time averaging
✓ Configure OLAP cubes with proper aggregation rules
✓ Consider storing deltas alongside balances
```

**Minimize Non-Additive Facts**:
```
✓ Store component facts (numerator, denominator)
✓ Calculate ratios in BI layer or views
✓ Only store non-additive facts if needed for filtering/display
✓ Always document correct aggregation approach
```

### User Education

**Critical Training Topics**:
```
1. Difference between additive, semi-additive, non-additive
2. Why you can't sum balances across time
3. How to calculate time-based averages correctly
4. Ratio of sums vs. sum of ratios
5. When to use SUM vs. AVG vs. calculated ratios
```

### Metadata Requirements

**Every Fact Must Have**:
```
✓ Additivity type (additive, semi-additive, non-additive)
✓ Description of what fact measures
✓ Which dimensions are additive (for semi-additive)
✓ Correct aggregation formula
✓ Units of measure
✓ Source system and calculation
```

---

By following these principles, you ensure facts aggregate correctly, users understand how to calculate metrics properly, and the dimensional model delivers accurate, consistent results across all BI tools and use cases.
