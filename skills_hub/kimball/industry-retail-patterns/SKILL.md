---
name: industry-retail-patterns
description: Apply retail and e-commerce dimensional modeling patterns for POS transactions, product hierarchies, promotions, customer behavior, inventory, and clickstream data. Use when designing data warehouses for retail stores, grocery chains, e-commerce websites, or any business selling products directly to consumers.
---

# Industry-Specific Patterns: Retail

## When to Use This Skill

Use this skill when designing dimensional models for retail and e-commerce businesses, including:
- **Grocery stores and supermarkets** - POS transactions, product hierarchies, promotional analysis
- **Department stores and specialty retail** - Multi-channel sales, customer shopping behavior
- **E-commerce and web retailers** - Online sales, clickstream integration, customer sessions
- **Any consumer-facing retail** - Direct product sales to end consumers

Apply these patterns during the **Dimensional Modeling** phase after completing business requirements gathering and bus matrix design.

## Overview

Retail businesses share common dimensional modeling patterns despite varying scales and channels. The retail industry was instrumental in developing many foundational dimensional modeling techniques, particularly around transaction-level granularity, product hierarchies, and causal analysis of promotions.

This skill covers:
1. Point-of-sale (POS) transaction modeling
2. Product dimension design with hierarchies
3. Promotion/causal dimension patterns
4. Customer shopping behavior analysis
5. Retail-specific date and time handling
6. Clickstream integration for e-commerce
7. Multi-channel profitability analysis

---

## Core Retail Dimensional Model

### Business Process: POS Retail Sales

The foundational retail dimensional model captures individual product sales at the cash register.

**Grain Declaration:**
"One row per product line item on a POS transaction"

**Why This Grain:**
- Provides maximum analytic flexibility
- Highly dimensional (many descriptive attributes available)
- Supports unpredictable slicing: by day-of-week, promotion type, product size, store location, etc.
- Atomic data can be summarized "every which way" by users
- Cannot answer detailed questions from pre-aggregated data

### Fact Table: POS Retail Sales

**Dimensions:**
- Date (transaction date)
- Product (SKU-level)
- Store (location)
- Promotion (causal conditions)
- Cashier (employee processing transaction)
- Payment Method (if single method per transaction)
- POS Transaction Number (degenerate dimension)

**Facts:**
- Sales Quantity (units sold) - Additive
- Regular Unit Price - Non-additive
- Discount Unit Price - Non-additive
- Net Unit Price - Non-additive
- Extended Sales Dollars (quantity × net price) - Additive
- Extended Discount Dollars - Additive
- Extended Cost Dollars - Additive
- Gross Profit Dollars (sales - cost) - Additive, derived
- Gross Margin Percent (profit / sales) - Non-additive ratio

**Critical Design Principles:**
- Store derived facts (like gross profit) physically to ensure consistent calculation
- Use surrogate keys for all dimension foreign keys
- Include "Not Applicable" rows in dimensions (e.g., "No Promotion", "No Cashier" for self-checkout)
- POS transaction number is degenerate dimension (no dimension table, just FK in fact)

---

## Retail-Specific Dimension Patterns

### 1. Product Dimension

The product dimension is typically the most descriptive dimension in retail models.

**Key Characteristics:**
- One row per SKU (stock keeping unit)
- Sourced from product master file at headquarters
- Can contain 60,000-300,000+ rows (current + historical products)
- Rich with 50+ descriptive attributes

**Flattened Merchandise Hierarchy:**
```
SKU (e.g., "12 oz Diet Coke Can")
  → Brand (e.g., "Coca-Cola")
    → Category (e.g., "Carbonated Beverages")
      → Department (e.g., "Beverages")
```

**Critical Pattern: Flatten the Hierarchy**
- Store ALL hierarchy levels (SKU, brand, category, department) as separate columns in the product dimension
- Do NOT normalize into separate brand/category/department tables (snowflaking)
- Accept attribute value repetition (department "Beverages" appears in 6,000 SKU rows)
- Disk space savings from normalization are negligible (<1% of total warehouse)
- Denormalization enables efficient browsing and bitmap indexes

**Essential Attributes:**
- **Natural key (NK):** Operational SKU code
- **Surrogate key (PK):** Warehouse-assigned integer
- **Descriptions:** SKU description, brand name, category name, department name
- **Hierarchy levels:** All levels from atomic SKU to highest rollup
- **Non-hierarchical attributes:** Package type, package size, flavor, fat content, organic indicator
- **Embedded meaning extraction:** If operational code contains meaning (e.g., characters 5-9 = manufacturer), extract as separate attributes
- **Numeric descriptors:** Standard list price, package weight, unit count
- **Indicators as text:** Use "Organic" / "Non-Organic" NOT "Y/N" or "1/0"

**Multiple Hierarchies:**
Product dimensions often contain multiple independent hierarchies:
- Merchandise hierarchy: SKU → Brand → Category → Department
- Supplier hierarchy: SKU → Manufacturer → Supplier Type
- Package hierarchy: SKU → Package Size → Package Type

**Numeric Values - Fact or Attribute?**
For values like "standard list price":
- **As dimension attribute** if primarily used for filtering/grouping (e.g., "Show all products with list price $10-$20")
- **As fact** if used in calculations (e.g., extended list price = quantity × list price)
- **Both locations** if used for both purposes (label attribute as "Current Standard Price" to distinguish)

**Reference:** See `product-dimension-deep-dive.md` for product dimension design patterns, hierarchies, and SCD strategies.

---

### 2. Store Dimension

The store dimension describes every retail location.

**Characteristics:**
- One row per store
- May require assembling from multiple operational sources (POS system, real estate dept, HR)
- Typically hundreds to thousands of rows

**Multiple Hierarchies:**

**Geographic Hierarchy:**
```
Store → City-State → County → State/Province → Country
```

**Organizational Hierarchy:**
```
Store → District → Region → Division
```

**Key Attributes:**
- Store Number (NK), Store Name
- Street Address, City-State, County, State, ZIP, Country
- District, Region (organizational rollup)
- Store Manager Name
- Floor Plan Type, Square Footage
- First Open Date, Last Remodel Date
- Photo Processing Type, Bakery Type, Pharmacy Indicator
- Selling Square Footage (numeric descriptor, but belongs in dimension not fact)

**Date Attributes in Dimensions:**
Dates like "First Open Date" and "Last Remodel Date":
- Can be simple date type columns if only used for filtering on standard calendar attributes
- Should be foreign keys to date dimension outriggers if users want to filter/group by fiscal period, holiday status, etc.

**Pattern: Role-Playing Date Dimensions**
```sql
CREATE VIEW first_open_date AS
  SELECT day_number AS first_open_day_number,
         month AS first_open_month,
         fiscal_period AS first_open_fiscal_period
  FROM date;
```

---

### 3. Promotion (Causal) Dimension

The promotion dimension is often the most interesting and complex retail dimension because it describes factors that cause changes in product sales.

**Purpose:**
Enable analysis of promotion effectiveness:
- Did promoted products experience sales lift?
- Was there time-shifting (sales before/after promotion)?
- Was there cannibalization (non-promoted products lost sales)?
- What was the net profitability of the promotion?

**Design Decision: Combined vs Separated Causal Mechanisms**

**Option A: Combined Promotion Dimension (Recommended)**
Create one row per unique combination of causal conditions:
- Temporary Price Reduction (TPR) - Yes/No, Amount
- Advertisement - Newspaper/Insert/None, Ad Type
- Display - End Aisle/Feature/None, Display Type
- Coupon - Cents Off/Percent Off/None, Coupon Value

Over a year: 1,000 ads + 5,000 price reductions + 1,000 displays = ~10,000 combined promotion condition rows

**Advantages:**
- If causal mechanisms are highly correlated, single dimension not much larger than separate dimensions
- Efficient browsing to see how mechanisms are used together
- Simpler schema for users

**Option B: Separate Causal Dimensions**
Four separate dimension tables: Price Reduction, Advertisement, Display, Coupon

**Advantages:**
- May be more intuitive if business thinks of mechanisms separately
- Easier to administer if managed by different teams

**Essential Attributes:**
- Promotion Name (e.g., "Summer Soda Blast 2013")
- Price Reduction Type, Price Reduction Amount
- Ad Type, Media Type, Ad Theme
- Display Type, Display Location
- Coupon Type, Coupon Value
- Promotion Begin Date, Promotion End Date
- Promotion Cost (WARNING: Only for constraining/grouping, NOT in sales fact table - wrong grain)

**Handling "No Promotion" Condition:**
- Create dimension row with surrogate key 0 or -1
- Populate with "No Promotion" descriptive values
- Ensures referential integrity (no nulls in fact table foreign keys)
- Many transactions will reference this row

**Reference:** See `promotion-dimension-patterns.md` for detailed promotion analysis techniques, lift calculation, and multi-mechanism design patterns.

---

### 4. Date Dimension for Retail

Retail businesses have specific date dimension requirements.

**Standard Attributes:**
- Day Number in Calendar Month (1-31) - Compare same day each month
- Day of Week Name (Monday-Sunday) - Compare Monday vs Sunday sales
- Week Ending Date, Week Number in Year
- Month Name, Month Number in Year (1-12), Year-Month (YYYY-MM)
- Calendar Quarter (Q1-Q4), Year-Quarter (YYYY-Q1)
- Calendar Year
- Fiscal Period, Fiscal Quarter, Fiscal Year (if different from calendar)

**Retail-Specific Attributes:**
- **Holiday Indicator:** "Holiday" / "Non-Holiday" (NOT "Y/N")
- **Weekday Indicator:** "Weekday" / "Weekend"
- **Payday Indicator:** "Payday" / "Non-Payday" (if relevant)
- **Shopping Season:** "Back to School" / "Holiday" / "Regular"
- **Promotional Period Indicator:** Links to major promotional campaigns

**Why Text Values for Indicators:**
These values become report labels and filter values. "Holiday" vs "Non-Holiday" is self-documenting; "1" vs "0" requires users to remember encoding.

**Time-of-Day Handling:**
- **Do NOT add time-of-day to date dimension** (would create 1,440 rows per day for minute grain = 10M+ rows)
- **If need to filter/group by time periods:** Create separate time-of-day dimension (1,440 rows for minute grain, with attributes: hour, shift, day-part grouping)
- **If only need time for calculations:** Store as date/time fact in fact table

---

### 5. Degenerate Dimensions in Retail

**POS Transaction Number:**
- Operational ticket/receipt number from POS system
- Groups together all products in a single shopping cart
- No corresponding dimension table (all interesting attributes extracted to other dimensions)
- Sits alone in fact table as degenerate dimension (DD)
- Part of fact table primary key: (Transaction Number + Product Key)
- Enables linking back to operational source system

**When to Assign Surrogate Key to Degenerate Dimension:**
Consider assigning surrogate key if:
- Transaction numbers not unique across stores
- Transaction numbers wrap/recycle over time
- Transaction control number is bulky (24-byte alphanumeric)
- BI tool requires dimension table to drill across

---

## Advanced Retail Patterns

### Handling Multiple Payment Methods

**Simple Case:** One payment method per transaction
- Add Payment Method dimension with attributes: Method Name, Type (Cash/Credit/Debit)

**Complex Case:** Multiple payment methods per transaction
- Do NOT alter grain to "one row per payment method per product" - unnatural grain
- **Option 1:** Separate fact table with grain "one row per transaction" with payment methods as separate facts
- **Option 2:** Separate fact table with grain "one row per payment method per transaction" with Payment Method dimension

### Frequent Shopper Programs

**Pattern: Add Customer Dimension**
When implementing loyalty/frequent shopper programs:
- Add Customer (Shopper) dimension foreign key to existing fact table
- Default historical rows to "Prior to Frequent Shopper Program" dimension row
- Include "Frequent Shopper Not Identified" row for non-enrolled shoppers
- Atomic grain enables this extension without impacting existing BI applications

**Customer Dimension Attributes:**
- Demographics: Age band, gender, income range, household size
- Geographic: Residential address, ZIP, city, county
- Behavioral: Loyalty tier, enrollment date, preferred store
- Preferences: Communication preferences, product affinities

### Factless Fact Table: Promotion Coverage

**Business Question:** "What products were on promotion but did not sell?"

**Problem:** Sales fact table only records products that sold. No rows for SKUs on promotion with zero sales.

**Solution:** Promotion Coverage Fact Table

**Grain:** "One row per product on promotion in a store each day (or week)"

**Dimensions:**
- Date, Product, Store, Promotion

**Facts:**
- Promotion Count (dummy fact = constant value 1, facilitates BI tool counting)

**Usage:**
Two-step query:
1. Query promotion coverage table → Universe of promoted products
2. Query sales fact table → Products that sold
3. Set difference = Promoted but didn't sell

OLAP cubes handle this more naturally with explicit cells for non-events.

---

## E-Commerce and Clickstream Integration

### Clickstream Unique Dimensions

E-commerce businesses add four unique dimensions not found in traditional retail:

**1. Page Dimension**
- Grain: Individual page (function/type, not every dynamic instance)
- Attributes: Page name, page type, page category, content category, site section
- Collaboration needed: Web developers embed descriptive codes in log files

**2. Event Dimension**
- Grain: Type of page event
- Values: Open Page, Refresh Page, Click Link, Enter Data, Submit Form

**3. Session Dimension**
- Grain: Classification of visitor session outcome
- Attributes: Local context (e.g., "Requesting Product Info"), overall context (e.g., "Ordering Product"), success status, customer status labels

**4. Referral Dimension**
- Grain: How customer arrived at page
- Attributes: Referrer type (Search Engine/Partner/Direct/Bookmark), referrer URL, search specification

### Clickstream Fact Tables

**Fact Table 1: Session-Level Fact Table**
**Grain:** "One row per completed customer session"

**Dimensions:** Date (GMT), Date (Local Wall Clock), Time-of-Day, Customer, Page (entry page), Session, Referrer

**Facts:** Session seconds, pages visited, orders placed, units ordered, order dollars

**Design Notes:**
- Two date/time roles: GMT for synchronization across servers, local wall clock for visitor perspective
- Entry page captured (how did customer "hop on the bus"?)
- More manageable size (20M sessions vs 100M page events per day)

**Fact Table 2: Page Event-Level Fact Table**
**Grain:** "One row per page event in each customer session"

**Dimensions:** Date (GMT), Date (Local Wall Clock), Time-of-Day, Customer, Page, Event, Session, Session ID (DD), Product, Referrer, Promotion, Step (3 roles)

**Facts:** Page seconds, units ordered, order dollars

**Design Notes:**
- Session ID degenerate dimension groups page events within session
- Step dimension (3 roles): Overall session step, purchase subsession step, abandonment subsession step
- Revenue facts (units/dollars) mostly null except on purchase event - but critical to tie behavior to revenue

**Reference:** See `clickstream-integration-patterns.md` for detailed clickstream dimension design, session identification challenges, and Google Analytics integration.

---

## Multi-Channel Profitability

### Extending Sales to Profitability Analysis

**Grain:** "One row per product line item on a sales ticket" (same as sales transaction grain)

**Channels:** Store sales, telesales, web sales

**Dimensions:** Date, Time, Customer, Channel, Product, Promotion, Ticket Number (DD)

**Profit & Loss Fact Structure:**
```
Units Sold
Gross Revenue (list price × quantity)
- Manufacturer Allowance
- Marketing Promotion Allowance
- Markdown
= Net Revenue
- Product Cost (manufacturing or acquisition)
- Product Storage Cost
= Gross Profit
- Outbound Shipping Cost
- Customer Service Cost
- Website System Cost (allocated)
- Marketing Cost (allocated)
- Overhead Cost (allocated)
= Net Profit
```

**Calculated Facts:**
- Store net revenue, gross profit, net profit as physical columns (not just views) if users access physical table
- Ensures consistent calculation across all users/applications

**Cost Allocation Challenges:**
- Some costs available at granular level (e.g., actual shipping cost per line item)
- Some costs only available as aggregates (e.g., national annual average)
- Website costs: Allocate by pages devoted to product line, pages visited, or actual purchases
- Document allocation rules and improve over time as better cost data becomes available

**Analysis Power:**
With P&L in dimensional framework, answer:
- How profitable is each channel? Why?
- How profitable are customer segments? Why?
- How profitable is each product line? Why?
- Which promotions work well on web but not in stores? Why?

**Reference:** See `retail-profitability-model.md` for complete P&L fact table design, cost allocation strategies, and cross-channel profitability analysis.

---

## Common Retail Queries and Patterns

### Drilling Down
"Nothing more than adding row header attributes from dimension tables"

**Example:** Sales by Department
- Drill down by Brand → Add Brand attribute to row header
- Drill down by Fat Content → Add Fat Content attribute (not in hierarchy!)
- Can drill down across multiple hierarchies and non-hierarchical attributes

### Drilling Across
Query multiple fact tables (e.g., Sales + Inventory + Clickstream) using conformed dimensions

**Example:** Compare web sessions to actual purchases
- Constrain both fact tables by same date, product, customer
- Combine results in report using conformed dimension attributes as row headers

### Market Basket Analysis
Use POS Transaction Number (degenerate dimension) to group products purchased together

**Example:** "Customers who bought diapers also bought..."
- Constrain fact table to transactions containing diapers
- Group remaining products in those transactions
- Rank by frequency

---

## Retail Schema Extensibility

Atomic grain enables graceful schema evolution:

**Adding New Dimensions:**
Example: Frequent Shopper dimension added years after initial deployment
- Add Customer foreign key column to fact table
- Populate historical rows with "Prior to Program" surrogate key
- Populate non-enrolled shoppers with "Not Identified" surrogate key
- No impact to existing BI applications

**Adding New Dimension Attributes:**
Example: New product attribute discovered
- Add column to product dimension
- Populate historical rows with "Not Available"
- Existing applications unaffected

**Adding New Facts:**
Example: New cost metric becomes available
- Add fact column to fact table
- Populate historical rows with null if metric not available historically
- Document availability date in metadata

**Cannot Support:** Adding dimensions that don't apply at atomic grain
- If grain is "daily store sales summary", cannot add customer dimension
- Atomic grain maximizes future extensibility

---

## Common Mistakes to Avoid

**1. Declaring wrong grain**
- ❌ "We'll store daily product sales by store"
- ✓ "One row per product on POS transaction" (atomic grain)
- Atomic grain provides maximum flexibility for unpredictable queries

**2. Snowflaking product hierarchy**
- ❌ Separate tables for Brand, Category, Department to "save space"
- ✓ Flatten all hierarchy levels into product dimension
- Dimension table space savings trivial (<1% total warehouse), but usability/performance costs enormous

**3. Creating centipede fact table**
- ❌ Separate foreign keys for SKU, Brand, Category, Department, Store, District, Region in fact table
- ✓ One product FK, one store FK (hierarchies flattened in dimensions)
- Centipede design explodes fact table size and creates unmanageable joins

**4. Using natural keys instead of surrogates**
- ❌ Product natural key as fact table foreign key
- ✓ Surrogate keys for all dimension foreign keys
- Buffers warehouse from operational changes, enables SCD, improves performance

**5. Null foreign keys in fact table**
- ❌ Null promotion key when product not on promotion
- ✓ "No Promotion" dimension row with surrogate key 0 or -1
- Maintains referential integrity, eliminates join complications

**6. Cryptic indicator values**
- ❌ Holiday indicator: "Y" / "N"
- ✓ Holiday indicator: "Holiday" / "Non-Holiday"
- Dimension attributes become report labels - make them self-documenting

**7. Premature aggregation**
- ❌ Store only weekly product sales to "save space"
- ✓ Store transaction line detail, build aggregates on top
- Users can summarize atomic data, cannot drill into summarized data

**8. Mixing grains in fact table**
- ❌ Some rows are transaction line items, some rows are transaction headers
- ✓ Separate fact tables for different grains
- Each fact table = single declared grain statement

---

## Implementation Checklist

**Product Dimension:**
- [ ] One row per SKU (current + historical)
- [ ] All hierarchy levels flattened (SKU, Brand, Category, Department)
- [ ] Non-hierarchical attributes included (package, flavor, organic, etc.)
- [ ] Operational codes preserved + meaning extracted
- [ ] Numeric descriptors treated as attributes (standard price, weight)
- [ ] Surrogate key assigned, natural key preserved
- [ ] SCD strategy defined for each attribute type

**Store Dimension:**
- [ ] Multiple hierarchies flattened (geographic + organizational)
- [ ] Descriptive attributes not codes ("Large Format" not "LF")
- [ ] Date attributes handled (outrigger if need non-standard calendar grouping)
- [ ] Current store attributes + historical stores retained

**Promotion Dimension:**
- [ ] Combined causal mechanisms or separate dimensions (business preference)
- [ ] "No Promotion" row with surrogate key for non-promoted sales
- [ ] Promotion cost excluded from transaction fact table (wrong grain)
- [ ] Descriptive attributes for all mechanisms (price reduction, ad, display, coupon)

**Date Dimension:**
- [ ] 10-20 years of rows (past history + future)
- [ ] Retail-specific attributes (holiday, weekday, season)
- [ ] Text values for indicators ("Holiday" not "H")
- [ ] Fiscal periods if different from calendar

**Fact Table:**
- [ ] Grain declared: "One row per product line item on POS transaction"
- [ ] All dimensions at declared grain
- [ ] POS transaction number as degenerate dimension
- [ ] Additive facts: quantity, extended dollars, cost
- [ ] Non-additive facts: unit prices, gross margin %
- [ ] Derived facts stored physically (gross profit)
- [ ] Null metrics left null (not zero) for proper aggregation

**Extensibility:**
- [ ] Customer dimension placeholder for future frequent shopper program
- [ ] Atomic grain supports unpredictable future requirements
- [ ] Conformed dimensions enable integration across business processes

---

## Integration with Other Skills

**Prerequisites:**
- SKILL 1: Business Requirements Gathering - Understand retail business processes
- SKILL 2: Enterprise Bus Matrix Design - Identify retail business processes (POS sales, inventory, orders)
- SKILL 3: Four-Step Dimensional Modeling - Apply systematic design process

**Builds Upon:**
- SKILL 4: Dimension Table Design Patterns - SCD for product changes, hierarchies
- SKILL 5: Fact Table Design Patterns - Transaction fact tables, additivity
- SKILL 6: Conformed Dimension Management - Product, customer, date across retail processes

**Supports:**
- SKILL 12: BI Application Design - Standard retail reports (sales by dept, promotion effectiveness)
- SKILL 20: Program Management - Retail DW/BI rollout strategies

---

## Summary

Retail dimensional modeling patterns apply broadly across consumer-facing businesses. Key takeaways:

1. **Atomic grain** (POS transaction line item) provides maximum analytic flexibility
2. **Flatten hierarchies** in dimension tables - denormalization is essential for performance and usability
3. **Rich dimension attributes** enable unpredictable slicing - 50+ product attributes typical
4. **Promotion dimension** captures causal factors for sales analysis
5. **Degenerate dimensions** (transaction number) group related fact rows
6. **Factless fact tables** answer "what didn't happen" questions
7. **Clickstream dimensions** (page, session, event, referral) extend traditional retail to e-commerce
8. **Atomic grain enables extensibility** - add dimensions (customer), attributes, facts without breaking existing apps

The retail case study introduced many fundamental dimensional modeling patterns used across all industries: transaction fact tables, multiple hierarchies, causal dimensions, degenerate dimensions, and the critical importance of atomic grain.
