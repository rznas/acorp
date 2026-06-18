# Product Dimension Deep Dive

## Overview

The product dimension is typically the richest and most descriptive dimension in retail dimensional models. This reference provides comprehensive guidance on designing robust product dimensions that support unpredictable analytic requirements.

---

## Dimension Characteristics

### Typical Size and Scope

**Row Count:**
- Current products in single store: 60,000 SKUs
- Enterprise product master (current + historical): 150,000 - 300,000+ rows
- Growth over time as new products introduced, old products discontinued

**Column Count:**
- 50-100+ descriptive attributes common
- Each attribute provides filtering, grouping, and labeling capability
- "More is better" - cannot predict which attributes users will need

**Source Systems:**
- Product master file from headquarters (primary source)
- Merchandising systems (hierarchy definitions, classifications)
- Supplier/vendor systems (manufacturer information)
- Marketing systems (brand positioning, product lifecycle stage)

---

## Merchandise Hierarchy Design

### Fixed Depth Hierarchies

Most retail product hierarchies have fixed, well-defined levels:

```
Level 1: Department (e.g., "Beverages")
  ├─ Level 2: Category (e.g., "Carbonated Soft Drinks")
  │  ├─ Level 3: Subcategory (e.g., "Cola")
  │  │  ├─ Level 4: Brand (e.g., "Coca-Cola")
  │  │  │  ├─ Level 5: SKU (e.g., "Coca-Cola Classic 12oz Can 12-pack")
```

**Critical Design Rule: Flatten ALL levels into single product dimension table**

**Example Product Dimension Rows:**

| Product Key | SKU Description | Brand | Category | Department | Package Type | Package Size | Flavor | ... |
|------------|----------------|-------|----------|------------|--------------|--------------|--------|-----|
| 42501 | Coca-Cola Classic 12oz Can 12pk | Coca-Cola | Carbonated Beverages | Beverages | Can | 12 oz | Cola | ... |
| 42502 | Coca-Cola Classic 12oz Can 6pk | Coca-Cola | Carbonated Beverages | Beverages | Can | 12 oz | Cola | ... |
| 42503 | Coca-Cola Diet 12oz Can 12pk | Coca-Cola | Carbonated Beverages | Beverages | Can | 12 oz | Diet Cola | ... |
| 58201 | Wonder Bread White 20oz Loaf | Wonder | Bread & Bakery | Bakery | Loaf | 20 oz | White | ... |

**Why Flatten (Not Snowflake):**

1. **Performance** - Single join to product dimension vs 5 joins through snowflaked hierarchy
2. **Usability** - Users see one product table with all attributes vs navigating complex hierarchy
3. **Bitmap indexes** - Can use bitmap indexes on low-cardinality attributes (category, department)
4. **Browsing** - Easy to see all brands within category without joining multiple tables
5. **Space savings negligible** - Repeated department names consume <1% of total warehouse space

**Space Calculation Example:**
- 300,000 SKUs
- Department description = 20 bytes
- 50 unique departments → average 6,000 repetitions per department
- Normalizing saves: 300,000 × 18 bytes = 5.4 MB
- Meanwhile, fact table = 10+ GB (space savings is rounding error!)

### Multiple Independent Hierarchies

Product dimensions often contain MULTIPLE hierarchies that are not correlated:

**Merchandise Hierarchy:**
SKU → Brand → Category → Department

**Supplier Hierarchy:**
SKU → Manufacturer → Distributor → Supplier Type

**Package Hierarchy:**
SKU → Package Size → Package Type → Package Family

**Geographic Source Hierarchy:**
SKU → Country of Origin → Region of Origin

**All hierarchies flattened in single dimension table:**

| Product Key | SKU | Brand | Category | Dept | Manufacturer | Distributor | Package Type | Package Size | Country Origin | ... |
|------------|-----|-------|----------|------|--------------|-------------|--------------|--------------|---------------|-----|

Users can drill down ANY hierarchy or mix attributes from multiple hierarchies:
- "Show sales by Department, then drill to Country of Origin" (cross-hierarchy)
- "Show sales by Category, then drill to Manufacturer" (cross-hierarchy)
- "Show sales by Brand, then drill to Package Type" (merchandise + package hierarchies)

---

## Non-Hierarchical Attributes

Not all product attributes participate in hierarchies. These are standalone descriptive attributes:

**Physical Attributes:**
- Package Type: Bag, Bottle, Box, Can, Carton, Jar
- Package Size: 8 oz, 12 oz, 16 oz, 1 lb, 5 lb
- Color: Red, Blue, Green, White, Clear
- Size: Small, Medium, Large, X-Large
- Weight: Numeric (e.g., 1.5 lbs)

**Content Attributes:**
- Flavor: Cherry, Vanilla, Chocolate, Strawberry
- Fat Content: Full Fat, Low Fat, Non-Fat
- Organic Indicator: Organic, Non-Organic
- Sugar Content: Regular, Diet, Sugar-Free
- Gluten Free Indicator: Gluten-Free, Contains Gluten

**Marketing Attributes:**
- Product Lifecycle Stage: Introduction, Growth, Maturity, Decline
- Promotional Tier: Premium, Standard, Value
- Seasonal Indicator: Seasonal, Year-Round
- Private Label Indicator: Private Label, National Brand
- New Product Indicator: New (< 90 days), Established

**Regulatory Attributes:**
- FDA Approved: Yes, No, N/A
- Alcohol Content Percent: Numeric or N/A
- Prescription Required: Yes, No, N/A
- Age Restricted: Yes, No

**Business Attributes:**
- Stock Status: In Stock, Low Stock, Out of Stock, Discontinued
- Replenishment Type: Automatic, Manual
- ABC Classification: A (high value), B (medium value), C (low value)

**Key Principle: Any attribute can be combined with any other for analysis**
- "Show organic products in bags under $5"
- "Show new seasonal products by manufacturer"
- "Show diet beverages in bottles by country of origin"

---

## Handling Embedded Meaning in Operational Codes

Operational SKU codes often contain embedded intelligence:

**Example Operational SKU Code:** `BEV-CC-12C-12PK-001`

**Embedded Meaning:**
- Characters 1-3: `BEV` = Department code (Beverages)
- Characters 5-6: `CC` = Brand code (Coca-Cola)
- Characters 8-10: `12C` = Package size and type (12 oz Can)
- Characters 12-14: `12PK` = Pack quantity (12-pack)
- Characters 16-18: `001` = Variant number

**Design Pattern:**

1. **Preserve entire operational code** as single attribute (natural key)
2. **Extract meaningful components** into separate decoded attributes

**Product Dimension Columns:**

| Column Name | Example Value | Source |
|------------|---------------|--------|
| Product Natural Key (NK) | BEV-CC-12C-12PK-001 | Operational system (preserved) |
| Department Code | BEV | Extracted from NK positions 1-3 |
| Department Description | Beverages | Decoded from department code |
| Brand Code | CC | Extracted from NK positions 5-6 |
| Brand Name | Coca-Cola | Decoded from brand code |
| Package Size Text | 12 oz | Extracted and decoded |
| Package Type Text | Can | Extracted and decoded |
| Pack Quantity | 12 | Extracted |

**Why Preserve AND Extract:**
- Preserve natural key for linkage to operational systems
- Extract meaning so users don't need to remember encoding rules
- Provide text descriptions, not codes, for filtering and labeling

---

## Numeric Values: Fact or Dimension Attribute?

Some product attributes are numeric. Where do they belong?

### Standard List Price

**If used primarily for FILTERING/GROUPING:**
→ Dimension attribute

**Example Queries:**
- "Show all products with list price between $10 and $20"
- "Group sales by price tier: Under $5, $5-$10, $10-$20, Over $20"

**Dimension Design:**
- Standard List Price (Numeric)
- Price Tier (Text): "Under $5", "$5-$10", "$10-$20", "Over $20"

**If used primarily for CALCULATION:**
→ Fact table

**Example Calculation:**
- Extended List Price = Quantity × Standard List Price

**If used for BOTH:**
→ Store in BOTH locations

**Fact Table:** Standard List Price at time of transaction (for calculation)
**Dimension Attribute:** Current Standard List Price (for filtering/grouping)

**Label to distinguish:**
- Fact: `standard_list_price` (valuation at transaction time)
- Dimension: `current_standard_list_price` (current list price)

### Other Numeric Descriptors

**Package Weight:**
- Usually dimension attribute (filtering: "Show products under 1 lb")
- Possibly fact if used in calculations (total shipment weight = sum of package weights)

**Unit Count per Package:**
- Dimension attribute (filtering: "Show all 12-packs")

**Standard Cost:**
- Typically fact (used to calculate extended cost)
- Might also be dimension attribute as "current standard cost" for product profiling

---

## Slowly Changing Dimensions for Products

Product attributes change at different rates. Define SCD strategy per attribute type.

### SCD Type 0 (Retain Original) - Rare for Products

**Attributes that never change:**
- Product introduction date
- Original product classification

### SCD Type 1 (Overwrite) - Common for Products

**Attributes where history not needed:**
- Product description typo corrections
- Non-material attribute corrections
- Current stock status
- ABC classification

**Implementation:**
- Overwrite attribute value in existing dimension row
- All history rewritten with new value
- Use when business doesn't care about historical attribute value

### SCD Type 2 (Add New Row) - Critical for Products

**Attributes where history IS needed:**
- **Price changes** - Need to know product's price at time of sale
- **Supplier changes** - Need to track which supplier provided product when
- **Package size changes** - Product reformulated from 16 oz to 15 oz
- **Brand changes** - Product rebrand ed from one brand to another
- **Category reclassification** - Product moved from one category to another

**Implementation:**
- Create new dimension row with new surrogate key
- Preserve old row with old surrogate key
- Add effective/expiration dates and current row indicator
- Fact rows reference appropriate product surrogate key based on transaction date

**Example: Product Price Change**

Before price change:

| Product Key | SKU NK | Description | Brand | Standard Price | Effective Date | Expiration Date | Current Row |
|------------|--------|-------------|-------|----------------|----------------|-----------------|-------------|
| 42501 | BEV-CC-001 | Coke 12oz 12pk | Coca-Cola | $4.99 | 2010-01-01 | 2013-06-30 | No |
| 42502 | BEV-CC-001 | Coke 12oz 12pk | Coca-Cola | $5.49 | 2013-07-01 | 9999-12-31 | Yes |

**Sales fact rows from June 2013 use surrogate key 42501 ($4.99 price)**
**Sales fact rows from July 2013+ use surrogate key 42502 ($5.49 price)**

### SCD Type 3 (Add New Attribute Column)

**Use case:** Track one previous value

**Example:** Product rebrand, but want to report sales under both old and new brand

| Product Key | SKU NK | Current Brand | Previous Brand | Brand Change Date |
|------------|--------|---------------|----------------|-------------------|
| 42501 | BEV-CC-001 | Coca-Cola Zero Sugar | Coca-Cola Zero | 2017-08-01 |

**Enables queries:**
- "Show sales under current brand name" (all history)
- "Show sales under previous brand name" (only applicable period)

### Recommended SCD Strategy by Attribute Type

| Attribute Type | SCD Strategy | Rationale |
|---------------|--------------|-----------|
| Product Natural Key | Type 0 | Never changes (immutable identifier) |
| Product Description | Type 1 | Minor corrections, no business impact |
| Brand | Type 2 | Critical for time-accurate brand performance analysis |
| Category/Department | Type 2 | Product reclassifications need history |
| Package Type/Size | Type 2 | Product reformulations need tracking |
| Standard List Price | Type 2 | Critical for price analysis |
| Manufacturer/Supplier | Type 2 | Track supplier changes over time |
| Organic Indicator | Type 2 | Certification status changes |
| Stock Status | Type 1 | Current status only, no history |
| ABC Classification | Type 1 | Recalculated periodically, no history |
| Product Lifecycle Stage | Type 1 | Current stage only |

---

## Product Dimension Row Lifecycle

### Product Introduction

**Event:** New product added to product master

**Dimension Update:**
1. Assign new surrogate key (e.g., 99001)
2. Populate all attributes from product master
3. Set effective date = introduction date
4. Set expiration date = 9999-12-31
5. Set current row indicator = Yes

### Product Attribute Change (Type 2)

**Event:** Product price changes from $4.99 to $5.49

**Dimension Update:**
1. Old row:
   - Set expiration date = day before change
   - Set current row indicator = No
2. New row:
   - Assign new surrogate key (e.g., 99002)
   - Copy all attributes from old row
   - Update changed attribute (standard price = $5.49)
   - Set effective date = change date
   - Set expiration date = 9999-12-31
   - Set current row indicator = Yes

### Product Discontinuation

**Event:** Product no longer sold

**Dimension Update (Option 1 - Preferred):**
- Keep row in dimension (retain for historical analysis)
- Set stock status = "Discontinued"
- Keep current row indicator = Yes
- Never delete dimension rows

**Dimension Update (Option 2):**
- Set expiration date = discontinuation date
- Set current row indicator = No
- Mark as discontinued in product lifecycle stage

---

## Browsing Product Dimension

Dimension tables should support efficient browsing - constrain on some attributes, view distinct values of others.

**Example Browsing Queries:**

**Browse brands within a category:**
```sql
SELECT DISTINCT brand_name
FROM product_dimension
WHERE category_name = 'Carbonated Beverages'
ORDER BY brand_name;
```

**Browse package types for organic products under $10:**
```sql
SELECT DISTINCT package_type, COUNT(*) as product_count
FROM product_dimension
WHERE organic_indicator = 'Organic'
  AND current_standard_price < 10.00
  AND current_row_indicator = 'Yes'
GROUP BY package_type
ORDER BY product_count DESC;
```

**Browse attributes across hierarchies:**
```sql
SELECT department_name, country_of_origin, COUNT(*) as sku_count
FROM product_dimension
WHERE current_row_indicator = 'Yes'
GROUP BY department_name, country_of_origin
ORDER BY department_name, sku_count DESC;
```

**Snowflaking defeats browsing:**
If category is in separate table, brand in another table, package type in yet another:
- Browsing brands within category requires join to category table
- Browsing package types for brands in categories requires 3-table join
- Complex SQL, poor performance, unusable for business users

---

## Product Dimension Advanced Patterns

### Product Variants

Some retailers have product "variants" - same base product in different colors/sizes:

**Option 1: Separate SKU per variant**
- Each color/size combination = separate dimension row
- Color and Size as dimension attributes
- Simple, clean design

**Option 2: Base product + variant attributes**
- Base product dimension + variant bridge table (for many-to-many)
- More complex, use only if variants truly multivalued

### Product Bundles

Product sold as bundle (e.g., "Shampoo + Conditioner Bundle"):

**Option 1: Bundle as separate SKU**
- Bundle SKU in product dimension
- Bundle components tracked separately in bill-of-materials (different business process)

**Option 2: Bridge table for bundle components**
- Product dimension contains bundle SKU
- Bridge table links bundle SKU to component SKUs with quantity

### Product Genealogy

Products with genealogy relationships (e.g., version 1.0 → version 2.0):

**Solution: Genealogy Attributes**
- Predecessor SKU (foreign key to product dimension)
- Successor SKU (foreign key to product dimension)
- Enables queries: "Show sales of product and all successors"

---

## Implementation Checklist

**Structure:**
- [ ] One row per SKU (current + historical products)
- [ ] Surrogate key (4-byte integer) as primary key
- [ ] Natural key (operational SKU code) preserved
- [ ] Durable supernatural key if natural key not reliable

**Hierarchies:**
- [ ] ALL merchandise hierarchy levels flattened (SKU, Brand, Category, Department, etc.)
- [ ] Multiple independent hierarchies flattened (merchandise, supplier, package, etc.)
- [ ] No snowflaking of any hierarchy levels

**Attributes:**
- [ ] 50+ descriptive attributes (more is better)
- [ ] Non-hierarchical attributes included (flavor, package, organic, etc.)
- [ ] Operational code meaning extracted (department code → department name)
- [ ] Text values for all indicators ("Organic" not "Y", "Gluten-Free" not "1")
- [ ] Numeric descriptors (price, weight) included as attributes if used for filtering

**Slowly Changing Dimensions:**
- [ ] SCD strategy defined for each attribute
- [ ] Type 2 SCD for price, supplier, category (time-accurate analysis)
- [ ] Type 1 SCD for descriptions, status (overwrite)
- [ ] Effective date, expiration date, current row indicator columns
- [ ] Never delete dimension rows (keep discontinued products for history)

**Data Quality:**
- [ ] Unknown/Not Applicable rows for each attribute (no nulls)
- [ ] Consistent attribute values (standardized spelling, capitalization)
- [ ] Valid hierarchy rollups (all SKUs assigned to valid brand/category/department)

**Usability:**
- [ ] Attribute names clear and business-friendly
- [ ] Attribute values self-documenting (no cryptic codes)
- [ ] Support efficient browsing (single table, no complex joins)

---

## Summary

The product dimension is the most descriptive and complex dimension in retail models:

1. **Flatten ALL hierarchies** into single dimension table - no snowflaking
2. **Include 50+ attributes** - cannot predict which attributes users will need
3. **Support multiple independent hierarchies** - merchandise, supplier, package
4. **Extract embedded meaning** from operational codes into descriptive attributes
5. **Use Type 2 SCD** for time-accurate price, supplier, category analysis
6. **Enable browsing** - constrain on attributes, view distinct values of others
7. **Never delete rows** - keep discontinued products for historical reporting

A robust product dimension with rich attributes translates directly into robust analytic capabilities for business users.
