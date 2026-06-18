# BI Application Template Specification Examples

## Overview

This reference provides concrete examples of BI application specifications following the standard template format described in the main skill. Each example demonstrates how to translate business requirements into detailed BI application specifications that developers can implement.

---

## Example 1: Product Sales Performance Dashboard

### Business Purpose

**Objective**: Enable store managers and merchandising analysts to monitor daily product sales performance, identify top/bottom performers, and track promotion effectiveness

**Users**:
- Primary: Store Managers (100 users)
- Secondary: Category Managers (15 users), Regional Managers (10 users)

**Frequency**: Daily (morning review), ad-hoc throughout the day

### Data Sources

**Fact Tables**:
- `retail_sales_facts` (grain = one row per product per POS transaction)

**Conformed Dimensions**:
- `date` - Daily grain
- `product` - SKU grain with brand/category/department hierarchy
- `store` - Store grain with district/region hierarchy
- `promotion` - Promotion condition grain

**Grain**: Aggregated to selected dimensional level (default: Daily × Brand × Store)

### Report Layout

#### Filters/Parameters

| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| Date Range | Date Picker | Last 90 days | Yesterday |
| Product Level | Radio Button | SKU / Brand / Category / Department | Brand |
| Geographic Level | Radio Button | Store / District / Region | Store |
| Promotion Filter | Multi-Select | All Promotions / Promoted Only / Non-Promoted | All Promotions |
| Metric Sort | Dropdown | Sales Dollars / Sales Units / Gross Profit / Margin % | Sales Dollars |

#### Row Headers (Dimensions)
- Product [at selected level]
- Store [at selected level]
- Date [daily, weekly, or monthly based on date range]

#### Column Headers (Metrics)
- Sales Dollars
- Sales Units
- Gross Profit Dollars
- Gross Margin %
- Discount Dollars
- Discount %

#### Sort Order
- Default: Descending by selected metric
- User can interactively re-sort by clicking column headers

#### Breaks/Subtotals
- Subtotal by Product Level (e.g., sum all brands in category)
- Subtotal by Geographic Level (e.g., sum all stores in district)
- Grand Total at bottom

### Calculations

#### Derived Facts

**Gross Profit Dollars**:
```
Gross Profit = SUM(sales_dollars) - SUM(cost_dollars)
```

**Gross Margin %** (Non-Additive Ratio):
```
Gross Margin % = (SUM(sales_dollars) - SUM(cost_dollars)) / SUM(sales_dollars) * 100

Note: Calculate as ratio of sums, NOT sum of ratios
```

**Discount %**:
```
Discount % = SUM(discount_dollars) / (SUM(sales_dollars) + SUM(discount_dollars)) * 100
```

#### Aggregation Rules
- Sales Dollars: SUM (additive across all dimensions)
- Sales Units: SUM (additive across all dimensions)
- Cost Dollars: SUM (additive across all dimensions)
- Margin %: Calculated ratio (non-additive, computed at display level)

#### Business Rules
- Exclude returns (sales_dollars < 0)
- Include only completed transactions (transaction_status = 'Complete')
- Cost dollars from `standard_cost` at time of sale

### Interactivity

#### Drill-Down Paths

**From Product Perspective**:
- Department → Category → Brand → SKU

**From Geographic Perspective**:
- Region → District → Store

**From Time Perspective**:
- Year → Quarter → Month → Week → Day

**Cross-Hierarchy Drilling**: Users can add any product attribute (e.g., Package Type) even if not in merchandise hierarchy

#### Drill-Across
Yes - Users can drill across to:
- `inventory_facts` to see average inventory levels for selected products/stores/dates
- `promotion_facts` (factless) to see promotion coverage

#### Filters
Users can interactively filter on any dimension attribute:
- Product: Brand, Category, Department, Package Type, Size
- Store: District, Region, Store Type, Square Footage Range
- Date: Day of Week, Holiday Indicator, Fiscal Period
- Promotion: Promotion Type, Ad Type, Display Type, Coupon Indicator

#### Export
- Excel (.xlsx)
- CSV
- PDF

### Performance

**Expected Row Count**:
- Typical: 500-2,000 rows (brands × stores for single day)
- Maximum: 50,000 rows (all SKUs × all stores for 7 days)

**Aggregates Leveraged**:
- `daily_sales_by_brand_store_agg` (pre-aggregated daily brand/store level)
- Falls back to atomic `retail_sales_facts` for SKU-level or non-aggregated dimension combinations

**Response Time Target**:
- Interactive queries (< 1000 rows): < 5 seconds
- Large queries (> 10,000 rows): < 30 seconds

---

## Example 2: Customer Segmentation Analysis Template

### Business Purpose

**Objective**: Enable marketing analysts to understand customer purchase behavior by demographic and behavioral segments to optimize targeting strategies

**Users**:
- Primary: Marketing Analysts (8 users)
- Secondary: Marketing Directors (3 users)

**Frequency**: Weekly for campaign planning, monthly for strategic review

### Data Sources

**Fact Tables**:
- `retail_sales_facts` (grain = one row per product per POS transaction)

**Conformed Dimensions**:
- `customer` - Individual customer grain with demographic and behavioral attributes
- `date` - Daily grain
- `product` - SKU grain with category hierarchy
- `store` - Store grain

**Grain**: Aggregated to Customer Segment × Product Category × Time Period

### Report Layout

#### Filters/Parameters

| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| Time Period | Dropdown | Last Week / Last Month / Last Quarter / Last Year | Last Quarter |
| Customer Segment | Multi-Select | All / High-Value / Medium-Value / Low-Value / New / At-Risk / Lapsed | All |
| Product Category | Multi-Select | All Categories / [list of categories] | All Categories |
| Minimum Purchase Count | Numeric Input | 0-999 | 1 |

#### Row Headers
- Customer Segment
- Product Category (if not filtered to single category)

#### Column Headers (Metrics)
- Customer Count
- Total Purchases (Transaction Count)
- Total Sales Dollars
- Average Transaction Value
- Average Items per Transaction
- Purchase Frequency (Transactions per Customer)
- Recency (Days Since Last Purchase - Average)

#### Sort Order
- Default: Descending by Total Sales Dollars
- Segment order: High-Value → Medium-Value → Low-Value → New → At-Risk → Lapsed

#### Breaks/Subtotals
- Subtotal by Customer Segment
- Grand Total at bottom

### Calculations

#### Derived Facts

**Customer Count** (Distinct Count):
```
Customer Count = COUNT(DISTINCT customer_key)
```

**Total Purchases** (Transaction Count):
```
Total Purchases = COUNT(DISTINCT pos_transaction_number)
```

**Average Transaction Value**:
```
Avg Transaction Value = SUM(sales_dollars) / COUNT(DISTINCT pos_transaction_number)
```

**Average Items per Transaction**:
```
Avg Items per Transaction = SUM(sales_quantity) / COUNT(DISTINCT pos_transaction_number)
```

**Purchase Frequency**:
```
Purchase Frequency = COUNT(DISTINCT pos_transaction_number) / COUNT(DISTINCT customer_key)
```

**Average Recency** (Days Since Last Purchase):
```
Avg Recency = AVG(current_date - MAX(transaction_date) for each customer)

Implementation: Requires customer-level calculation, then average
```

#### Aggregation Rules
- All metrics are aggregated to segment level
- Distinct counts prevent double-counting customers or transactions
- Averages calculated as ratio of sums (not sum of averages)

#### Business Rules
- Customer Segment assignments from `customer` dimension based on:
  - High-Value: Lifetime value > $5,000
  - Medium-Value: Lifetime value $1,000-$4,999
  - Low-Value: Lifetime value < $1,000
  - New: First purchase within last 90 days
  - At-Risk: No purchase in 90-180 days (was active)
  - Lapsed: No purchase in 180+ days

### Interactivity

#### Drill-Down Paths

**From Customer Segment**:
- Segment → Individual Customers (customer name)

**From Product**:
- Category → Brand → SKU

**From Time**:
- Quarter → Month → Week

**Cross-Attribute Drilling**: Add demographic attributes (Age Range, Income Level, Geographic Region)

#### Drill-Across
Yes - Users can drill across to:
- `customer_satisfaction_facts` to see NPS scores by segment
- `marketing_campaign_facts` to see campaign response rates by segment

#### Filters
Users can interactively filter on:
- Customer: Age Range, Income Level, ZIP Code, Loyalty Program Status
- Product: Brand, Package Type, Price Range
- Date: Month, Quarter, Year, Fiscal Period

#### Export
- Excel (.xlsx) with embedded segment definitions
- CSV
- PowerPoint (chart only)

### Performance

**Expected Row Count**:
- Typical: 6-20 rows (customer segments × 1-3 product categories)
- Maximum: 300 rows (all segments × all categories × monthly breakdown)

**Aggregates Leveraged**:
- `customer_segment_summary_agg` (monthly pre-aggregated segment metrics)
- Atomic `retail_sales_facts` for custom segment definitions or sub-segment analysis

**Response Time Target**:
- Standard queries: < 10 seconds
- Custom segmentation (require atomic access): < 60 seconds

---

## Example 3: Inventory Turnover and Days Supply Report

### Business Purpose

**Objective**: Enable inventory managers to monitor product inventory levels, identify slow-moving and fast-moving SKUs, and optimize reorder quantities

**Users**:
- Primary: Inventory Managers (5 users)
- Secondary: Category Managers (15 users), Store Managers (100 users)

**Frequency**: Daily for critical SKUs, weekly for full inventory review

### Data Sources

**Fact Tables**:
- `inventory_facts` (grain = one row per product per store per day)
- `sales_facts` (grain = one row per product per POS transaction) - for drill-across

**Conformed Dimensions**:
- `date` - Daily grain
- `product` - SKU grain
- `store` - Store grain

**Grain**: Daily snapshot aggregated to selected product/store level

### Report Layout

#### Filters/Parameters

| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| Snapshot Date | Date Picker | Last 60 days | Yesterday |
| Product Level | Radio Button | SKU / Brand / Category | Category |
| Store Level | Radio Button | Store / District / Region | District |
| Alert Threshold | Multi-Select | All / Overstocked / Understocked / Optimal | All |

**Alert Thresholds**:
- Overstocked: Days Supply > 60
- Understocked: Days Supply < 7
- Optimal: Days Supply 7-60

#### Row Headers
- Product [at selected level]
- Store [at selected level]

#### Column Headers (Metrics)
- Quantity On Hand (EOD)
- Quantity On Hand Value at Cost
- Days Supply
- Inventory Turns (Annualized)
- Quantity Sold (Last 30 Days)
- Quantity Sold (Same Period Last Year)
- Alert Status

#### Sort Order
- Default: Descending by Days Supply (shows slow movers first)
- Color-coded alert status: Red (Overstocked), Yellow (Understocked), Green (Optimal)

#### Breaks/Subtotals
- Subtotal by Product Level
- Subtotal by Store Level
- Grand Total at bottom

### Calculations

#### Derived Facts

**Quantity On Hand** (Semi-Additive):
```
Quantity On Hand = Snapshot value for selected date

Note: Additive across products and stores, NOT across dates
For date ranges, use AVG(quantity_on_hand) over date range
```

**Quantity On Hand Value at Cost**:
```
QOH Value = quantity_on_hand * standard_unit_cost
```

**Days Supply**:
```
Days Supply = quantity_on_hand / AVG(daily_sales_quantity_last_30_days)

Where: daily_sales_quantity_last_30_days from sales_facts
```

**Inventory Turns (Annualized)**:
```
Inventory Turns = (SUM(sales_quantity_last_365_days) / AVG(quantity_on_hand_last_365_days))

Note: Requires drill-across to sales_facts
```

**Quantity Sold (Last 30 Days)**:
```
Qty Sold L30D = SUM(sales_quantity) from sales_facts for last 30 days

Note: Requires drill-across to sales_facts
```

**Alert Status** (Derived Attribute):
```
Alert Status =
  CASE
    WHEN Days Supply > 60 THEN 'Overstocked'
    WHEN Days Supply < 7 THEN 'Understocked'
    ELSE 'Optimal'
  END
```

#### Aggregation Rules
- Quantity On Hand: SUM across products/stores; AVG across dates
- Days Supply: Calculate at display level (cannot pre-aggregate)
- Inventory Turns: Calculate at display level using summed sales and averaged inventory

#### Business Rules
- Use EOD (End of Day) inventory snapshot values
- Exclude discontinued products (product_status = 'Active')
- Sales quantity excludes returns (sales_quantity > 0)
- Standard unit cost from product dimension current value

### Interactivity

#### Drill-Down Paths

**From Product**:
- Category → Brand → SKU

**From Store**:
- Region → District → Store

**From Alert Status**:
- Filter to show only Overstocked or Understocked items

#### Drill-Across
Yes - Critical for this report:
- Drill across to `sales_facts` to get quantity sold metrics
- Drill across to `purchase_order_facts` to see pending receipts

#### Filters
Users can interactively filter on:
- Product: Brand, Category, Supplier, Package Type
- Store: District, Region, Store Type
- Date: Single date or rolling date range
- Alert Status: Overstocked / Understocked / Optimal

#### Export
- Excel (.xlsx) with conditional formatting for alerts
- CSV
- PDF

### Performance

**Expected Row Count**:
- Typical: 200-500 rows (categories × districts for single day)
- Maximum: 6 million rows (all SKUs × all stores for single day - not recommended for UI)

**Aggregates Leveraged**:
- `inventory_category_district_daily_agg` for category/district level
- Atomic `inventory_facts` required for SKU-level analysis
- `sales_last_30_days_agg` for recent sales velocity

**Response Time Target**:
- Aggregated queries (category/district): < 10 seconds
- SKU-level queries (limited scope): < 30 seconds
- Full SKU inventory (all stores): Export to file, 5-10 minutes

### Special Considerations

**Drill-Across Complexity**:
This report requires careful drill-across design because:
1. Inventory facts are daily snapshots (semi-additive)
2. Sales facts are transaction grain (fully additive)
3. Days Supply and Turns calculations require combining both fact tables

**Implementation Approach**:
1. Query inventory_facts for QOH snapshot at selected date/level
2. Query sales_facts for sales quantity over last 30/365 days at same level
3. Join results on product/store conformed dimensions
4. Calculate Days Supply and Turns in BI application layer

**Performance Optimization**:
- Pre-calculate last 30-day sales velocity in ETL (store as additional fact in inventory_facts)
- Use partitioned fact tables (by date) to limit scan range
- Limit SKU-level analysis to specific product categories or stores

---

## Example 4: Promotion Effectiveness Scorecard

### Business Purpose

**Objective**: Enable merchandising and marketing teams to evaluate promotion performance, calculate ROI, and compare promotion types

**Users**:
- Primary: Marketing Analysts (8 users), Category Managers (15 users)
- Secondary: CMO (1 user), VP Merchandising (1 user)

**Frequency**: Weekly during active promotions, monthly for historical analysis

### Data Sources

**Fact Tables**:
- `sales_facts` (grain = one row per product per POS transaction)
- `promotion_coverage_facts` (factless - grain = one row per product on promotion per store per day)

**Conformed Dimensions**:
- `date` - Daily grain
- `product` - SKU grain
- `store` - Store grain
- `promotion` - Promotion condition grain with promotion type, ad type, display type, coupon attributes

**Grain**: Aggregated to Promotion × Product Category × Time Period

### Report Layout

#### Filters/Parameters

| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| Promotion Period | Date Range | Last 90 days | Last 30 days |
| Promotion Type | Multi-Select | All / Price Reduction / Ad Only / Display Only / Ad+Display / Coupon | All |
| Product Category | Multi-Select | All / [list of categories] | All |
| Compare to Baseline | Checkbox | Yes / No | Yes |

#### Row Headers
- Promotion Description
- Promotion Type
- Product Category
- Promotion Start Date
- Promotion End Date

#### Column Headers (Metrics)
- Promoted Sales Units
- Promoted Sales Dollars
- Baseline Sales Units (Expected without promotion)
- Baseline Sales Dollars
- Incremental Sales Units (Lift)
- Incremental Sales Dollars (Lift)
- Promotion Cost
- Promotion ROI %
- Promoted Products Count
- Products Sold on Promotion (% of coverage)

#### Sort Order
- Default: Descending by Promotion ROI %
- Group by Promotion Type

#### Breaks/Subtotals
- Subtotal by Promotion Type
- Grand Total at bottom

### Calculations

#### Derived Facts

**Promoted Sales**:
```
Promoted Sales Dollars = SUM(sales_dollars) WHERE promotion_key <> 0 (not "No Promotion")
Promoted Sales Units = SUM(sales_quantity) WHERE promotion_key <> 0
```

**Baseline Sales** (Estimated without promotion):
```
Baseline Sales = AVG(sales_dollars for same products/stores/day-of-week in prior 8 weeks
                     WHERE promotion_key = 0)

Implementation: Requires ETL pre-calculation or complex SQL with date arithmetic
```

**Incremental Sales (Lift)**:
```
Incremental Sales Dollars = Promoted Sales Dollars - Baseline Sales Dollars
Incremental Sales Units = Promoted Sales Units - Baseline Sales Units
```

**Promotion Cost**:
```
Promotion Cost = (discount_dollars + ad_cost + display_cost + coupon_cost)

Where:
  - discount_dollars: SUM from sales_facts
  - ad_cost, display_cost, coupon_cost: from promotion dimension
```

**Promotion ROI %**:
```
Promotion ROI % = ((Incremental Sales Dollars - Promotion Cost) / Promotion Cost) * 100

Note: Can be negative if promotion was unprofitable
```

**Promoted Products Count**:
```
Promoted Products Count = COUNT(DISTINCT product_key) from promotion_coverage_facts
```

**Products Sold on Promotion**:
```
Products Sold % = (COUNT(DISTINCT product_key with sales) /
                   COUNT(DISTINCT product_key in coverage)) * 100
```

#### Aggregation Rules
- All dollar and unit metrics: SUM
- ROI %: Calculated at display level (non-additive)
- Distinct counts for products

#### Business Rules
- Baseline estimation uses 8-week lookback, excluding weeks with promotions
- Promotion cost allocated to product category based on promoted sales mix
- Exclude "No Promotion" rows (promotion_key = 0 or -1)
- Compare identical day-of-week (e.g., current Tuesday vs. prior 8 Tuesdays) for baseline

### Interactivity

#### Drill-Down Paths

**From Promotion**:
- Promotion → Product Category → Brand → SKU (shows which products drove lift)

**From Time**:
- Promotion Period → Week → Day (shows daily lift pattern)

**From Store**:
- All Stores → District → Store (shows geographic promotion effectiveness)

#### Drill-Across
Yes:
- Drill to `promotion_coverage_facts` to see which products WERE promoted but DIDN'T sell
- Calculate "promotion sell-through %" using set difference

#### Filters
Users can interactively filter on:
- Promotion: Type, Discount %, Ad Type, Display Type
- Product: Brand, Category, Price Range
- Store: District, Region, Store Type
- Date: Week, Month, Fiscal Period

#### Export
- Excel (.xlsx) with promotion analysis formulas preserved
- PowerPoint (summary charts for executive presentation)
- CSV

### Performance

**Expected Row Count**:
- Typical: 50-150 rows (active promotions × categories for selected period)
- Maximum: 2,000 rows (all promotions × all categories × monthly)

**Aggregates Leveraged**:
- `promotion_sales_summary_agg` (pre-aggregated promotion × category × week)
- `baseline_sales_estimate_agg` (pre-calculated baselines to avoid complex real-time calculation)
- Atomic `sales_facts` for drill-down to brand/SKU level

**Response Time Target**:
- Summary promotion scorecard: < 10 seconds
- Detailed promotion drill-down: < 20 seconds

### Special Considerations

**Baseline Calculation Complexity**:
- Baseline estimation is business-critical but computationally expensive
- Best practice: ETL pre-calculates baseline sales and stores as additional facts
- Alternative: Maintain separate `baseline_facts` table for comparison

**Set Difference for Sell-Through**:
- Requires querying both sales_facts and promotion_coverage_facts
- Products in coverage but not sales = products that didn't sell despite promotion
- Highlights ineffective promotional efforts

---

## Summary

These specification examples demonstrate:

1. **Comprehensive Documentation**: Each template fully specifies business purpose, data sources, layout, calculations, interactivity, and performance expectations

2. **Dimensional Model Leverage**: All examples show how to translate dimensional model components (fact grains, dimension attributes, conformed dimensions) into BI application design

3. **Calculation Clarity**: Explicit formulas for derived facts, aggregation rules, and business logic ensure consistent implementation

4. **User Experience Focus**: Parameter options, drill paths, filtering, and export capabilities designed for specific user personas and use cases

5. **Performance Awareness**: Realistic row count estimates, aggregate strategies, and response time targets guide physical design and optimization

**Using These Examples**:
- Adapt specification format to your organization's standards
- Customize parameters and metrics to match business requirements
- Work with business users to validate specifications before development
- Use specifications as communication tool between business, BI developers, and DBAs

These detailed specifications bridge the gap between business requirements and technical implementation, ensuring BI applications deliver intended business value.
