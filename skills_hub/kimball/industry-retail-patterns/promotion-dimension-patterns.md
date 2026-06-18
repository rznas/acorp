# Promotion (Causal) Dimension Patterns

## Overview

The promotion dimension, often called the **causal dimension**, describes factors that cause changes in product sales. It's one of the most interesting and analytically powerful dimensions in retail because it enables measurement of marketing effectiveness.

---

## Purpose of Causal Dimensions

Promotions are judged on multiple factors:

### 1. Lift Measurement
**Definition:** Sales increase during promotional period compared to baseline

**Analysis Requirements:**
- Baseline sales estimate (what sales would have been without promotion)
- Actual sales during promotion
- Lift = (Actual Sales - Baseline Sales) / Baseline Sales

**Baseline Estimation Methods:**
- Historical average (same period in prior years)
- Moving average (prior 4-8 weeks, excluding promotional periods)
- Regression models (accounting for seasonality, trends)

### 2. Time Shifting Detection
**Definition:** Sales decrease before/after promotion canceling gains during promotion

**Pattern:**
- Week before promotion: Customers delay purchases (waiting for sale)
- Promotion week: Sales spike
- Week after promotion: Sales drop (customers stocked up)
- Net effect: Sales transferred across time periods, not incremental

**Analysis:**
- Compare sales 2 weeks before, during, and 2 weeks after promotion
- Calculate net incremental sales across entire window

### 3. Cannibalization Analysis
**Definition:** Promoted products gain sales while nearby non-promoted products lose sales

**Example:**
- Promote Coca-Cola at $2.99 (down from $4.99)
- Coca-Cola sales increase 300%
- Pepsi (not promoted) sales decrease 40%
- Store brand cola sales decrease 60%
- Net category sales increase only 50% (not 300%)

**Analysis:**
- Define product category boundary (all colas, all carbonated beverages?)
- Measure sales change for ALL products in category
- Net lift = Category sales increase, not just promoted product increase

### 4. Market Growth Assessment
**Definition:** Overall category experiences net gain accounting for time-shifting and cannibalization

**Analysis:**
- Sum ALL products in category
- Include time periods before, during, and after promotion
- Subtract baseline expectations for entire category
- Result = true incremental category growth

### 5. Profitability Calculation
**Definition:** Incremental profit considering promotion costs

**Formula:**
```
Incremental Profit = (Incremental Units × (Price - Cost))
                   - Promotion Costs
                   - Cannibalization Impact

Where:
- Incremental Units = Net new units sold (after time-shifting)
- Price = Promotional price
- Cost = Product cost
- Promotion Costs = Ad costs + Display costs + Price reduction subsidy
- Cannibalization Impact = Lost profit on cannibalized products
```

---

## Promotion Dimension Design Patterns

### Pattern 1: Combined Promotion Dimension (Recommended)

**Concept:** Create one dimension row per unique combination of promotional mechanisms

**Promotional Mechanisms:**
1. **Temporary Price Reduction (TPR)** - Reduced shelf price
2. **Advertisement** - Newspaper, circular, TV, radio
3. **Display** - End aisle, feature display, floor stack
4. **Coupon** - Manufacturer coupon, store coupon

**Example Combinations:**

Over one year:
- 1,000 ads placed
- 5,000 temporary price reductions
- 1,000 end aisle displays
- 500 coupon programs

**Typical combinations:**
- TPR + Ad + Display (most stores)
- TPR + Ad only (some stores without display)
- TPR only (quiet promotion)
- Ad + Coupon (manufacturer-driven)
- Display only (seasonal merchandising)

**Result:** ~10,000 promotion dimension rows (combinations, not sum)

**Dimension Table Structure:**

| Promotion Key | Promotion Name | Price Reduction Type | Price Reduction Amt | Ad Type | Ad Media | Display Type | Display Location | Coupon Type | Coupon Value | Promo Start Date | Promo End Date |
|--------------|----------------|---------------------|--------------------:|---------|----------|--------------|------------------|-------------|-------------:|-----------------|----------------|
| 0 | No Promotion | None | 0.00 | None | None | None | None | None | 0.00 | NULL | NULL |
| 1001 | Summer Soda Blast 2013 | Percentage Off | 25.00 | Circular | Print | End Aisle | Front End Cap | None | 0.00 | 2013-06-15 | 2013-07-15 |
| 1002 | Summer Soda Blast 2013 - No Display | Percentage Off | 25.00 | Circular | Print | None | None | None | 0.00 | 2013-06-15 | 2013-07-15 |
| 1003 | Back to School Special | Dollars Off | 1.50 | Newspaper | Print | Feature Display | Main Aisle | Store Coupon | 0.50 | 2013-08-10 | 2013-09-05 |

**Advantages:**
- If mechanisms highly correlated, dimension not much larger than separate dimensions
- Efficient browsing: See how mechanisms used together
- Simpler schema (fewer tables)
- Single foreign key in fact table

**Disadvantages:**
- Dimension administration more complex (manage combinations)
- If mechanisms not correlated, dimension explodes (1000 × 5000 × 1000 = 5 billion rows - NOT realistic)

---

### Pattern 2: Separate Causal Dimensions

**Concept:** Four separate dimension tables, four foreign keys in fact table

**Dimension Tables:**
1. Price Reduction Dimension
2. Advertisement Dimension
3. Display Dimension
4. Coupon Dimension

**Example Schema:**

**Price Reduction Dimension:**
| Price Reduction Key | Reduction Type | Reduction Amount | Reduction Percent |
|--------------------|----------------|-----------------|------------------|
| 0 | No Reduction | 0.00 | 0.00 |
| 101 | Dollars Off | 1.00 | N/A |
| 102 | Dollars Off | 2.00 | N/A |
| 201 | Percentage Off | N/A | 10.00 |
| 202 | Percentage Off | N/A | 25.00 |

**Advertisement Dimension:**
| Ad Key | Ad Type | Ad Media | Ad Theme | Ad Cost |
|--------|---------|----------|----------|--------:|
| 0 | No Ad | None | None | 0.00 |
| 1 | Circular | Print | Summer Sale | 5000.00 |
| 2 | Newspaper | Print | Back to School | 2500.00 |

**Fact Table:**
```
POS Sales Fact Table
- Date Key (FK)
- Product Key (FK)
- Store Key (FK)
- Price Reduction Key (FK)
- Advertisement Key (FK)
- Display Key (FK)
- Coupon Key (FK)
- Cashier Key (FK)
- Transaction Number (DD)
- Sales Quantity
- Extended Sales Dollars
- ...
```

**Advantages:**
- Each dimension managed independently
- May be more intuitive if business thinks of mechanisms separately
- Administration simpler (manage 4 small dimensions, not 1 large combination dimension)
- If mechanisms uncorrelated, avoids combinatorial explosion

**Disadvantages:**
- Four foreign keys in fact table (wider fact table)
- Cannot browse combinations easily (requires querying fact table)
- More complex queries (join to 4 causal dimensions)

---

### Decision Criteria: Combined vs Separated

**Use Combined Promotion Dimension when:**
- Promotional mechanisms highly correlated (usually deployed together)
- Business users think of promotions as integrated campaigns
- Schema simplicity important (fewer tables)
- Combination count reasonable (< 100,000 rows)

**Use Separate Causal Dimensions when:**
- Promotional mechanisms independent (different teams manage)
- Mechanisms used in many different combinations
- Combined dimension would be too large
- Business users accustomed to analyzing mechanisms separately

**Hybrid Approach:**
- Core promotion dimension (most common combinations)
- Additional foreign keys for independent mechanisms
- Example: Promotion dimension (price + ad + display) + separate coupon dimension

---

## Essential Promotion Dimension Attributes

### Identification Attributes
- **Promotion Key (PK):** Surrogate key
- **Promotion Natural Key (NK):** Operational promotion ID
- **Promotion Name:** Descriptive name (e.g., "Summer Soda Blast 2013")
- **Promotion Description:** Full text description

### Temporal Attributes
- **Promotion Start Date:** First day of promotion
- **Promotion End Date:** Last day of promotion
- **Promotion Duration Days:** Calculated (end date - start date + 1)
- **Promotion Year-Month:** For grouping promotions by period

### Price Reduction Attributes
- **Price Reduction Type:** "Percentage Off", "Dollars Off", "Buy X Get Y", "None"
- **Price Reduction Amount:** Dollar value (e.g., $1.00 off)
- **Price Reduction Percent:** Percentage value (e.g., 25% off)
- **Reduced Price:** Actual promotional price (if known)

### Advertisement Attributes
- **Ad Type:** "Circular", "Newspaper", "TV", "Radio", "Online", "None"
- **Ad Media Type:** "Print", "Broadcast", "Digital", "None"
- **Ad Theme:** "Summer Sale", "Back to School", "Holiday", etc.
- **Ad Placement:** "Front Page", "Category Page", "Back Page"
- **Ad Size:** "Full Page", "Half Page", "Quarter Page"

### Display Attributes
- **Display Type:** "End Aisle", "Feature Display", "Floor Stack", "None"
- **Display Location:** "Front End Cap", "Main Aisle", "Checkout", "None"
- **Display Size:** "Large", "Medium", "Small"
- **Display Duration:** Days display remained in place

### Coupon Attributes
- **Coupon Type:** "Manufacturer Coupon", "Store Coupon", "Digital Coupon", "None"
- **Coupon Value:** Dollar value (e.g., $0.50 off)
- **Coupon Redemption Method:** "Physical", "Digital", "Auto-Applied"

### Promotion Scope Attributes
- **Promotion Scope:** "National", "Regional", "Store-Specific", "Customer-Specific"
- **Target Customer Segment:** "All Customers", "Loyalty Members", "New Customers"
- **Participating Stores:** "All Stores", "Select Stores", "Online Only"

### Cost Attributes (Use with Caution)
- **Estimated Promotion Cost:** Total promotional campaign cost

**WARNING:** Promotion cost attributes should be used ONLY for constraining and grouping, NOT as facts in the POS transaction fact table. Why?

**Wrong Grain:**
- POS fact table grain: One row per product on transaction
- Promotion cost grain: One cost value per entire promotion campaign

**If promotion cost appears in POS fact table:**
- Cost value repeated on every line item
- Summing across line items gives wrong total (massive over-counting)

**Correct Approach:**
- Promotion cost as dimension attribute: Filter/group promotions by cost tier
- Promotion cost in separate fact table: Grain = one row per promotion (different fact table)

---

## Handling "No Promotion" Condition

Most retail transactions involve non-promoted products. The promotion dimension MUST handle this.

### No Promotion Dimension Row

**Required Attributes:**

| Promotion Key | Promotion Name | Price Reduction Type | Ad Type | Display Type | Coupon Type |
|--------------|----------------|---------------------|---------|--------------|-------------|
| 0 | No Promotion | None | None | None | None |

**Alternative:** Surrogate key = -1 instead of 0 (preference varies)

**Why Required:**
1. **Referential Integrity:** Fact table foreign key cannot be NULL (violates RI)
2. **Join Simplicity:** NULL keys cannot be joined (special SQL syntax required)
3. **Query Consistency:** Grouping by promotion name with NULLs gives confusing results
4. **User Experience:** "No Promotion" appears in filter lists; NULL disappears or requires special handling

**Usage in Fact Table:**
Many (often majority) of fact table rows reference this "No Promotion" row:
```
Transaction 1: Product A, Promotion Key = 0 (no promotion)
Transaction 2: Product B, Promotion Key = 1001 (summer sale)
Transaction 3: Product C, Promotion Key = 0 (no promotion)
```

---

## Promotion Analysis Patterns

### Basic Promotion Effectiveness Query

**Business Question:** "What was sales lift for Summer Soda Blast promotion?"

**SQL Pattern:**
```sql
SELECT
    p.promotion_name,
    SUM(f.sales_quantity) as total_units,
    SUM(f.extended_sales_dollars) as total_revenue,
    COUNT(DISTINCT f.transaction_number) as transaction_count
FROM pos_sales_fact f
JOIN promotion_dimension p ON f.promotion_key = p.promotion_key
JOIN date_dimension d ON f.date_key = d.date_key
WHERE p.promotion_name = 'Summer Soda Blast 2013'
  AND d.calendar_date BETWEEN '2013-06-15' AND '2013-07-15'
GROUP BY p.promotion_name;
```

### Baseline Comparison

**Business Question:** "Compare promoted sales to baseline (same period last year)"

**SQL Pattern:**
```sql
-- Promoted period (2013)
SELECT
    SUM(f.sales_quantity) as promoted_units
FROM pos_sales_fact f
JOIN promotion_dimension p ON f.promotion_key = p.promotion_key
JOIN date_dimension d ON f.date_key = d.date_key
WHERE p.promotion_name = 'Summer Soda Blast 2013'
  AND d.calendar_date BETWEEN '2013-06-15' AND '2013-07-15';

-- Baseline period (2012, same dates, no promotion)
SELECT
    SUM(f.sales_quantity) as baseline_units
FROM pos_sales_fact f
JOIN promotion_dimension p ON f.promotion_key = p.promotion_key
JOIN date_dimension d ON f.date_key = d.date_key
WHERE p.promotion_key = 0  -- No promotion
  AND d.calendar_date BETWEEN '2012-06-15' AND '2012-07-15';

-- Calculate lift
-- Lift % = (Promoted Units - Baseline Units) / Baseline Units × 100
```

### Time-Shifting Analysis

**Business Question:** "Did promotion just shift sales across time periods?"

**SQL Pattern:**
```sql
SELECT
    CASE
        WHEN d.calendar_date < '2013-06-15' THEN 'Pre-Promotion'
        WHEN d.calendar_date BETWEEN '2013-06-15' AND '2013-07-15' THEN 'During Promotion'
        WHEN d.calendar_date > '2013-07-15' THEN 'Post-Promotion'
    END as promotion_period,
    SUM(f.sales_quantity) as units_sold,
    SUM(f.extended_sales_dollars) as revenue
FROM pos_sales_fact f
JOIN product_dimension pr ON f.product_key = pr.product_key
JOIN date_dimension d ON f.date_key = d.date_key
WHERE pr.brand_name = 'Coca-Cola'
  AND d.calendar_date BETWEEN '2013-06-01' AND '2013-08-01'  -- 2 weeks before/after
GROUP BY promotion_period
ORDER BY promotion_period;
```

**Interpretation:**
- If pre-promotion sales below normal → customers delayed purchases
- If post-promotion sales below normal → customers stocked up during promotion
- Net incremental sales = Sum all periods - Expected baseline for all periods

### Cannibalization Analysis

**Business Question:** "Did Coca-Cola promotion cannibalize Pepsi sales?"

**SQL Pattern:**
```sql
SELECT
    pr.brand_name,
    SUM(CASE WHEN d.calendar_date BETWEEN '2013-06-15' AND '2013-07-15'
             THEN f.sales_quantity ELSE 0 END) as promo_period_units,
    SUM(CASE WHEN d.calendar_date BETWEEN '2013-05-15' AND '2013-06-14'
             THEN f.sales_quantity ELSE 0 END) as pre_promo_period_units,
    SUM(CASE WHEN d.calendar_date BETWEEN '2013-06-15' AND '2013-07-15'
             THEN f.sales_quantity ELSE 0 END) -
    SUM(CASE WHEN d.calendar_date BETWEEN '2013-05-15' AND '2013-06-14'
             THEN f.sales_quantity ELSE 0 END) as unit_change
FROM pos_sales_fact f
JOIN product_dimension pr ON f.product_key = pr.product_key
JOIN date_dimension d ON f.date_key = d.date_key
WHERE pr.category_name = 'Carbonated Beverages'
  AND d.calendar_date BETWEEN '2013-05-15' AND '2013-07-15'
GROUP BY pr.brand_name
ORDER BY unit_change DESC;
```

**Interpretation:**
- Coca-Cola: +300% (promoted brand)
- Pepsi: -40% (cannibalized)
- Store Brand: -60% (cannibalized)
- Category Total: +50% (net incremental growth)

### Promotion ROI Analysis

**Business Question:** "What was profitability of the promotion?"

**Requires:** Profitability fact table with extended cost, promotion cost allocation

**SQL Pattern:**
```sql
SELECT
    p.promotion_name,
    SUM(f.extended_sales_dollars) as total_revenue,
    SUM(f.extended_cost_dollars) as total_cost,
    SUM(f.extended_sales_dollars - f.extended_cost_dollars) as gross_profit,
    SUM(f.allocated_promotion_cost) as promotion_cost,
    SUM(f.extended_sales_dollars - f.extended_cost_dollars) -
    SUM(f.allocated_promotion_cost) as net_profit
FROM profitability_fact f
JOIN promotion_dimension p ON f.promotion_key = p.promotion_key
WHERE p.promotion_name = 'Summer Soda Blast 2013'
GROUP BY p.promotion_name;
```

**Note:** This requires promotion costs allocated to individual line items in profitability fact table (different grain than POS sales fact).

---

## Promotion Mechanism Correlation Analysis

**Business Question:** "Which promotional mechanisms work best together?"

**Approach:** Query fact table to see mechanism combinations and their effectiveness

**If using combined promotion dimension:**
```sql
SELECT
    p.price_reduction_type,
    p.ad_type,
    p.display_type,
    COUNT(DISTINCT f.transaction_number) as transaction_count,
    SUM(f.sales_quantity) as total_units,
    AVG(f.sales_quantity) as avg_units_per_transaction
FROM pos_sales_fact f
JOIN promotion_dimension p ON f.promotion_key = p.promotion_key
WHERE p.promotion_key <> 0  -- Exclude "No Promotion"
GROUP BY p.price_reduction_type, p.ad_type, p.display_type
ORDER BY avg_units_per_transaction DESC;
```

**If using separate causal dimensions:**
```sql
SELECT
    pr.reduction_type,
    ad.ad_type,
    dp.display_type,
    COUNT(DISTINCT f.transaction_number) as transaction_count,
    SUM(f.sales_quantity) as total_units,
    AVG(f.sales_quantity) as avg_units_per_transaction
FROM pos_sales_fact f
JOIN price_reduction_dimension pr ON f.price_reduction_key = pr.price_reduction_key
JOIN advertisement_dimension ad ON f.ad_key = ad.ad_key
JOIN display_dimension dp ON f.display_key = dp.display_key
WHERE pr.price_reduction_key <> 0  -- Has some promotion
GROUP BY pr.reduction_type, ad.ad_type, dp.display_type
ORDER BY avg_units_per_transaction DESC;
```

**Results might show:**
- TPR + Ad + Display: 12.5 units per transaction (best)
- TPR + Ad only: 8.3 units per transaction
- TPR only: 5.1 units per transaction
- Ad + Display (no price reduction): 2.1 units per transaction (ineffective)

---

## Promotion Dimension Slowly Changing Dimensions

Most promotion dimension attributes are Type 0 (fixed) or Type 1 (overwrite).

**Type 0 (Retain Original) - Most Common:**
- Promotion name, dates, mechanisms defined at creation
- Never change once promotion runs

**Type 1 (Overwrite) - Occasional:**
- Estimated promotion cost might be refined after campaign
- Promotion effectiveness rating added after analysis

**Type 2 (Rare for Promotions):**
- Seldom used because promotions are time-bounded events
- Historical promotion definition already captured in dimension row

**Exception:** Ongoing/recurring promotion programs might use Type 2 for changing attributes.

---

## Advanced Promotion Patterns

### Promotion Hierarchy

Some organizations have promotion hierarchies:
```
Campaign (e.g., "Summer 2013")
  └─ Promotion (e.g., "Summer Soda Blast")
    └─ Promotion Variant (e.g., "With Display", "Without Display")
```

**Design:** Flatten hierarchy into promotion dimension (like product hierarchy)

| Promotion Key | Campaign Name | Promotion Name | Promotion Variant | ... |
|--------------|---------------|----------------|-------------------|-----|

### Multi-Product Promotions

Promotion applies to multiple products:

**Pattern 1:** Promotion dimension row + products inferred from fact table
- Promotion dimension: One row per promotion (mechanisms, dates)
- Fact table: Identifies which products sold under which promotion

**Pattern 2:** Promotion-Product bridge table (if need to pre-define promoted products)
- Promotion dimension: One row per promotion
- Product dimension: One row per product
- Promotion-Product bridge: Many-to-many relationship (which products included in promotion)

### Customer-Specific Promotions

Promotions targeted to specific customers (loyalty programs):

**Option 1:** Customer attribute in promotion dimension
- Promotion dimension attribute: "Target Customer Segment"

**Option 2:** Separate promotion-customer bridge table
- For personalized promotions (different promotion per customer)

---

## Common Mistakes to Avoid

**1. Storing promotion cost in POS transaction fact table**
- ❌ Promotion cost as fact in line-item grain table
- ✓ Promotion cost as dimension attribute (for filtering) or in separate promotion fact table

**2. NULL promotion keys in fact table**
- ❌ NULL when product not on promotion
- ✓ "No Promotion" dimension row with surrogate key 0 or -1

**3. Promotion as Type 2 SCD when it's time-bounded event**
- ❌ Creating new promotion dimension row when promotion characteristics change
- ✓ Promotion attributes fixed at creation (Type 0); use end date to bound promotion

**4. Insufficient promotion granularity**
- ❌ "Summer Sale" promotion covering 100 products with different mechanisms
- ✓ Separate promotion rows for different mechanism combinations

**5. Too much promotion granularity**
- ❌ New promotion dimension row for every store-product-date combination
- ✓ Promotion dimension describes promotional conditions; fact table associates conditions with sales

---

## Implementation Checklist

**Dimension Structure:**
- [ ] Decision made: Combined vs separate causal dimensions
- [ ] "No Promotion" row with surrogate key 0 or -1
- [ ] All promotional mechanisms captured (price, ad, display, coupon)

**Attributes:**
- [ ] Descriptive names ("Percentage Off" not "PCT", "No Promotion" not "N")
- [ ] Temporal attributes (start date, end date, duration)
- [ ] All mechanism types and values captured
- [ ] Promotion scope and targeting attributes

**Cost Handling:**
- [ ] Promotion cost NOT in POS transaction fact table (wrong grain)
- [ ] Promotion cost as dimension attribute (for filtering) if needed
- [ ] Promotion cost in separate fact table (promotion grain) if full analysis needed

**Analysis Support:**
- [ ] Sufficient attributes to support lift analysis
- [ ] Sufficient attributes to support time-shifting analysis
- [ ] Sufficient attributes to support cannibalization analysis
- [ ] Sufficient attributes to support ROI analysis

**Integration:**
- [ ] Promotion dimension conformed across sales channels (store, web, phone)
- [ ] Promotion data sourced from marketing systems
- [ ] Promotion effectiveness measured and fed back to dimension

---

## Summary

The promotion (causal) dimension is one of retail's most powerful dimensions:

1. **Enables measurement** of marketing effectiveness (lift, time-shifting, cannibalization, ROI)
2. **Combined vs separated** - choose based on mechanism correlation and business thinking
3. **"No Promotion" row required** - most transactions reference this row
4. **Promotion cost** - dimension attribute for filtering, NOT fact in transaction table
5. **Rich attributes** - capture all mechanism types, values, scope, targeting
6. **Analysis patterns** - baseline comparison, time-shifting, cannibalization, ROI
7. **Integration** - conform across all sales channels for consistent promotion analysis

Effective promotion analysis requires atomic grain POS transaction data combined with comprehensive promotion dimension attributes.
