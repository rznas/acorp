# Drill-Across Query Patterns and Examples

## Overview

Drilling across fact tables is one of the most powerful capabilities of a dimensional data warehouse built on conformed dimensions. This reference provides detailed patterns, SQL examples, and best practices for implementing drill-across queries in BI applications.

## Fundamental Drill-Across Concept

**Definition**: Drill-across queries combine performance metrics from two or more fact tables that share one or more conformed dimensions.

**How It Works**:
1. Query each fact table separately with identical dimension constraints
2. Each query groups by the same conformed dimension attribute(s)
3. Results are joined together using FULL OUTER JOIN on the conformed attribute(s)
4. Combined result set enables cross-process analysis

## Basic Drill-Across Pattern

### Scenario: Compare Sales and Inventory for Products

**Fact Table 1: Retail Sales** (grain = one row per product per POS transaction)
- Dimensions: Date, Product, Store, Promotion, POS Transaction Number (DD)
- Facts: Sales Quantity, Sales Dollars, Discount Dollars

**Fact Table 2: Store Inventory Snapshot** (grain = one row per product per store per day)
- Dimensions: Date, Product, Store
- Facts: Quantity On Hand, Quantity Sold, Value at Cost

**Shared Conformed Dimensions**: Date, Product, Store

### SQL Implementation

```sql
-- Query 1: Sales by Product and Store for January 2013
SELECT
    p.product_name,
    s.store_name,
    SUM(sf.sales_dollars) as total_sales
FROM
    sales_facts sf
    JOIN product p ON sf.product_key = p.product_key
    JOIN store s ON sf.store_key = s.store_key
    JOIN date d ON sf.date_key = d.date_key
WHERE
    d.month_name = 'January'
    AND d.year = 2013
GROUP BY
    p.product_name,
    s.store_name;

-- Query 2: Average Inventory by Product and Store for January 2013
SELECT
    p.product_name,
    s.store_name,
    AVG(if.quantity_on_hand) as avg_inventory
FROM
    inventory_facts if
    JOIN product p ON if.product_key = p.product_key
    JOIN store s ON if.store_key = s.store_key
    JOIN date d ON if.date_key = d.date_key
WHERE
    d.month_name = 'January'
    AND d.year = 2013
GROUP BY
    p.product_name,
    s.store_name;

-- Application Layer: Outer Join Results
-- (Typically performed by BI tool, not written manually)

SELECT
    COALESCE(sales.product_name, inventory.product_name) as product_name,
    COALESCE(sales.store_name, inventory.store_name) as store_name,
    sales.total_sales,
    inventory.avg_inventory,
    -- Cross-fact calculation in application
    CASE
        WHEN inventory.avg_inventory > 0
        THEN sales.total_sales / inventory.avg_inventory
        ELSE NULL
    END as sales_per_unit_inventory
FROM
    (Query 1 results) sales
    FULL OUTER JOIN (Query 2 results) inventory
        ON sales.product_name = inventory.product_name
        AND sales.store_name = inventory.store_name;
```

**Result**:
```
Product Name      | Store Name  | Total Sales | Avg Inventory | Sales/Inventory
------------------|-------------|-------------|---------------|------------------
Cheerios 18oz     | Store #001  | $45,231     | 1,250         | 36.18
Cheerios 18oz     | Store #025  | $38,890     | NULL          | NULL
Tide 64oz         | Store #001  | $89,445     | 890           | 100.50
Bananas (lb)      | Store #025  | NULL        | 2,340         | NULL
```

## Multi-Process Drill-Across Pattern

### Scenario: Analyze Complete Product Value Chain

**Business Question**: "For each product brand, show me purchase order value, warehouse receipts, store deliveries, and retail sales for Q1 2013"

**Fact Tables Involved**:
1. **Purchase Orders** (Procurement)
2. **Warehouse Receipts** (Inbound Logistics)
3. **Store Deliveries** (Distribution)
4. **Retail Sales** (Consumer Sales)

**Shared Conformed Dimension**: Product (at Brand level via shrunken rollup)

### Implementation Approach

```sql
-- Query 1: PO Value by Brand
SELECT
    p.brand,
    SUM(po.order_quantity * po.unit_cost) as po_value
FROM
    purchase_order_facts po
    JOIN product p ON po.product_key = p.product_key
    JOIN date d ON po.po_date_key = d.date_key
WHERE
    d.year = 2013 AND d.quarter = 1
GROUP BY p.brand;

-- Query 2: Warehouse Receipts by Brand
SELECT
    p.brand,
    SUM(wr.received_quantity * wr.unit_cost) as receipt_value
FROM
    warehouse_receipt_facts wr
    JOIN product p ON wr.product_key = p.product_key
    JOIN date d ON wr.receipt_date_key = d.date_key
WHERE
    d.year = 2013 AND d.quarter = 1
GROUP BY p.brand;

-- Query 3: Store Deliveries by Brand
SELECT
    p.brand,
    SUM(sd.delivery_quantity * sd.unit_cost) as delivery_value
FROM
    store_delivery_facts sd
    JOIN product p ON sd.product_key = p.product_key
    JOIN date d ON sd.delivery_date_key = d.date_key
WHERE
    d.year = 2013 AND d.quarter = 1
GROUP BY p.brand;

-- Query 4: Retail Sales by Brand
SELECT
    p.brand,
    SUM(sf.sales_dollars) as sales_value
FROM
    sales_facts sf
    JOIN product p ON sf.product_key = p.product_key
    JOIN date d ON sf.date_key = d.date_key
WHERE
    d.year = 2013 AND d.quarter = 1
GROUP BY p.brand;

-- Application combines via repeated FULL OUTER JOINs on p.brand
```

**Result Enables Value Chain Analysis**:
```
Brand        | PO Value   | Receipt Value | Delivery Value | Sales Value | Margin
-------------|------------|---------------|----------------|-------------|--------
Cheerios     | $1,250,000 | $1,248,500    | $1,247,800     | $1,890,000  | 51.4%
Tide         | $890,000   | $889,200      | $888,600       | $1,456,000  | 63.8%
Private Lbl  | $445,000   | $446,100      | $445,900       | $623,000    | 39.7%
```

## Drill-Across with Different Grains

### Scenario: Daily Sales vs. Monthly Budget

**Challenge**: Sales fact table has daily grain; Budget fact table has monthly grain

**Solution**: Roll sales up to monthly level OR expand budget down to daily level

**Approach 1: Roll Sales to Month** (Preferred)

```sql
-- Query 1: Actual Sales by Product/Month
SELECT
    p.product_category,
    d.year_month,  -- YYYY-MM format
    SUM(sf.sales_dollars) as actual_sales
FROM
    sales_facts sf
    JOIN product p ON sf.product_key = p.product_key
    JOIN date d ON sf.date_key = d.date_key
WHERE
    d.year = 2013
GROUP BY
    p.product_category,
    d.year_month;

-- Query 2: Budget by Product/Month
SELECT
    p.product_category,
    d.year_month,
    SUM(bf.budget_dollars) as budget_sales
FROM
    budget_facts bf
    JOIN product p ON bf.product_key = p.product_key
    JOIN month_date d ON bf.month_date_key = d.month_date_key
WHERE
    d.year = 2013
GROUP BY
    p.product_category,
    d.year_month;

-- Join on product_category and year_month
```

**Approach 2: Expand Budget to Daily** (Less Common)

Requires budget allocation logic (e.g., divide monthly budget by days in month), which may introduce inaccuracies.

## Handling Non-Additive Facts in Drill-Across

### Scenario: Inventory Balances (Semi-Additive) Across Locations

**Challenge**: Inventory balances can be summed across products and stores but NOT across dates

**Solution**: Use AVG for time aggregation, SUM for dimensional aggregation

```sql
-- Query 1: Average Daily Inventory by Product Category (Jan 2013)
SELECT
    p.product_category,
    AVG(if.quantity_on_hand) as avg_daily_inventory
FROM
    inventory_facts if
    JOIN product p ON if.product_key = p.product_key
    JOIN date d ON if.date_key = d.date_key
WHERE
    d.month_name = 'January' AND d.year = 2013
GROUP BY
    p.product_category;

-- Query 2: Sales by Product Category (Jan 2013)
SELECT
    p.product_category,
    SUM(sf.sales_quantity) as total_sales_qty
FROM
    sales_facts sf
    JOIN product p ON sf.product_key = p.product_key
    JOIN date d ON sf.date_key = d.date_key
WHERE
    d.month_name = 'January' AND d.year = 2013
GROUP BY
    p.product_category;

-- Application calculates: Inventory Turns = total_sales_qty / avg_daily_inventory
```

**Important**: SQL AVG() function divides by row count, not date count. Correct calculation requires:
- Sum inventory across all rows
- Divide by distinct count of dates

## Drill-Across with Conformed Facts

### Scenario: Revenue from Multiple Sources

**Business Question**: "What is total enterprise revenue by customer segment?"

**Fact Tables**:
1. **Product Sales** (retail channel)
2. **Service Revenue** (service channel)
3. **Subscription Revenue** (recurring channel)

**Conformed Fact**: Revenue (must have identical definition and currency)

**Conformed Dimension**: Customer (at Segment level via shrunken rollup)

```sql
-- Query 1: Product Sales Revenue
SELECT
    c.customer_segment,
    SUM(sf.revenue_dollars) as product_revenue
FROM
    sales_facts sf
    JOIN customer c ON sf.customer_key = c.customer_key
GROUP BY c.customer_segment;

-- Query 2: Service Revenue
SELECT
    c.customer_segment,
    SUM(srv.revenue_dollars) as service_revenue
FROM
    service_facts srv
    JOIN customer c ON srv.customer_key = c.customer_key
GROUP BY c.customer_segment;

-- Query 3: Subscription Revenue
SELECT
    c.customer_segment,
    SUM(sub.revenue_dollars) as subscription_revenue
FROM
    subscription_facts sub
    JOIN customer c ON sub.customer_key = c.customer_key
GROUP BY c.customer_segment;

-- Application joins and sums: total_revenue = product + service + subscription
```

**Critical**: All "revenue_dollars" facts must:
- Use identical calculation formulas
- Be expressed in same currency
- Have identical fiscal definitions (gross vs. net)
- Be additive at the customer segment level

## BI Tool Implementation Patterns

### Pattern 1: Temporary Table Join (Database-Side)

```sql
-- BI tool creates temp tables for each query
CREATE TEMP TABLE sales_temp AS (Query 1);
CREATE TEMP TABLE inventory_temp AS (Query 2);

-- Then joins temp tables
SELECT
    COALESCE(s.product_name, i.product_name) as product,
    s.sales, i.inventory
FROM sales_temp s
FULL OUTER JOIN inventory_temp i
    ON s.product_name = i.product_name;

-- Drop temp tables
DROP TABLE sales_temp;
DROP TABLE inventory_temp;
```

### Pattern 2: Application Server Join

1. BI tool executes Query 1, retrieves result set into memory
2. BI tool executes Query 2, retrieves result set into memory
3. Application server performs in-memory join
4. Combined result set rendered in report

**Advantage**: No database temp table overhead
**Disadvantage**: Memory constraints for large result sets

### Pattern 3: Report-Layer Join

Some BI tools join results directly in the reporting layer using report variables and matching logic.

## Best Practices for Drill-Across Design

<best_practices>
1. **Consistent Filtering**: Apply identical dimension constraints across all queries
   - Same date range, same product filter, same geographic scope

2. **Use FULL OUTER JOIN**: Ensures all rows from all fact tables appear in result
   - INNER JOIN would exclude products that only appear in one fact table
   - LEFT/RIGHT JOIN would bias toward one fact table

3. **Attribute Label Consistency**: Conformed dimension attributes must have:
   - Identical column names (product_name not product_desc)
   - Identical data values ("Brand A" not "BRAND A" or "Brand_A")
   - Identical data types (VARCHAR not CHAR)

4. **Cross-Fact Calculations in Application**: Ratios and percentages combining facts from different tables must be calculated AFTER drill-across join
   - Cannot pre-calculate in individual fact table queries

5. **Performance Optimization**:
   - Constrain queries as much as possible before drill-across join
   - Use aggregates if available for common drill-across patterns
   - Consider materialized views for frequently-used drill-across combinations

6. **Handle NULLs Appropriately**:
   - Use COALESCE for dimension attributes in FULL OUTER JOIN
   - Decide on NULL handling for missing facts (display NULL, 0, or "N/A")

7. **Validate Conformance**: Before deploying drill-across capabilities:
   - Verify dimension attribute conformance (names, values, definitions)
   - Test edge cases (products in one fact table but not others)
   - Validate cross-fact calculation formulas
</best_practices>

## Common Drill-Across Mistakes

<anti_patterns>
1. **Using Non-Conformed Dimensions**: Attempting to drill across using dimensions that aren't standardized
   - Result: Join fails or produces incorrect matches

2. **Inconsistent Filters**: Applying different constraints to different fact table queries
   - Result: Comparing apples to oranges

3. **Using INNER JOIN**: Only including rows that exist in all fact tables
   - Result: Missing data from products/customers that don't appear in all processes

4. **Attempting to Join on Surrogate Keys**: Joining on dimension surrogate keys instead of natural/business keys
   - Result: No matches because surrogate keys differ across fact tables

5. **Pre-calculating Ratios**: Computing ratios within individual fact table queries instead of after drill-across
   - Result: Incorrect ratio values (sum of ratios ≠ ratio of sums)

6. **Incompatible Fact Definitions**: Combining facts with different business rules or currencies
   - Result: Meaningless comparisons and incorrect totals

7. **Ignoring Grain Differences**: Attempting to drill across fact tables at incompatible grains without aggregation
   - Result: Duplicate counting or incorrect joins
</anti_patterns>

## Summary

Drill-across queries are the payoff for investing in conformed dimensions. They enable integrated, cross-process analysis that would be impossible with standalone departmental databases. Successful drill-across implementation requires:

- Rigorous adherence to conformed dimension standards
- BI tool support for multi-pass queries and result joining
- Careful handling of grain differences and non-additive facts
- Cross-fact calculations performed in the application layer after joining
- Comprehensive testing to validate conformance and calculation logic

When properly designed and implemented, drill-across capabilities unlock the full analytical power of the enterprise data warehouse bus architecture.
