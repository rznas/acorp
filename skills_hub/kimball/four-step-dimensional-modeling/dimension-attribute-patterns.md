# Dimension Attribute Patterns

## Purpose

This guide provides comprehensive patterns for designing rich, robust dimension table attributes. Well-designed dimension attributes are the "soul" of the data warehouse—they enable powerful filtering, grouping, and labeling that make dimensional models valuable to business users.

## When to Use This Guide

Use this guide when:
- Designing detailed dimension table attributes (Step 3 of four-step process)
- Deciding which attributes to include in dimensions
- Determining how to structure hierarchies and descriptive attributes
- Handling special attribute types (flags, codes, numeric values, dates)

## Core Principle

**A disproportionate amount of effort should go into dimension table design** because dimensions are the entry points and descriptive labels that enable business analysis. Invest time making dimensions as descriptive and complete as possible.

---

## Attribute Pattern Categories

### 1. Descriptive Text Attributes
### 2. Hierarchical Attributes
### 3. Indicators and Flags
### 4. Numeric Descriptors
### 5. Operational Codes
### 6. Date Attributes
### 7. Degenerate Dimensions
### 8. Special Attributes

---

## Pattern 1: Descriptive Text Attributes

### Description

The most common and valuable dimension attributes—verbose, meaningful text that describes entities.

### Characteristics

- Text data type (VARCHAR)
- 20-100 characters typical length
- Human-readable values
- Used for filtering, grouping, and report labels
- Multiple attributes provide different perspectives

### Examples

**Product Dimension:**
```
Product Description: "Campbell's Condensed Chicken Noodle Soup 10.75 oz"
Brand Description: "Campbell's"
Category Description: "Soups"
Department Description: "Grocery"
Package Type Description: "Can"
Size Description: "10.75 oz"
Flavor Description: "Chicken Noodle"
Dietary Description: "Regular"
```

**Store Dimension:**
```
Store Name: "Northgate Mall Store"
Store Manager Name: "Sarah Johnson"
District Name: "Northwest District"
Region Name: "Western Region"
City: "Seattle"
State: "Washington"
Store Format Description: "Superstore"
```

**Customer Dimension:**
```
Customer Full Name: "John Michael Smith"
Customer First Name: "John"
Customer Last Name: "Smith"
Customer Gender: "Male" (not "M")
Marital Status: "Married" (not "M")
Income Range: "$50,000 - $74,999" (not code "3")
Education Level: "Bachelor's Degree" (not "BS")
```

### Best Practices

**DO:**
- Use full, spelled-out descriptions, not abbreviations
- Make attribute names and values self-explanatory
- Include both long and short versions if needed (Month Name: "January", Month Abbreviation: "Jan")
- Populate with "Unknown" or "Not Applicable" instead of NULL
- Decode all operational codes into meaningful text

**DON'T:**
- Use cryptic abbreviations (use "Description" not "Desc")
- Store only codes without decoded values
- Use inconsistent naming across similar attributes
- Leave attributes NULL if data is missing

### When to Use

- Always—descriptive text attributes are the foundation of dimension tables
- Any time you need to describe "who, what, where, when, why, how"
- For any attribute users will filter on or group by

---

## Pattern 2: Hierarchical Attributes

### Description

Attributes that form many-to-one rollup paths, enabling drill-down analysis.

### Characteristics

- Multiple levels from detail to summary
- Fixed depth (predictable levels) or variable depth (ragged)
- Each level is a separate attribute in the dimension table
- Flattened (denormalized) in a single dimension table
- Multiple hierarchies can coexist in same dimension

### Fixed Depth Hierarchy Examples

**Product Merchandise Hierarchy:**
```
SKU Description (lowest level)
  ↓ (many SKUs to one Brand)
Brand Description
  ↓ (many Brands to one Category)
Category Description
  ↓ (many Categories to one Department)
Department Description (highest level)

Example Row:
SKU: "Campbell's Condensed Chicken Noodle Soup 10.75oz"
Brand: "Campbell's"
Category: "Soups"
Department: "Grocery"
```

**Geographic Hierarchy:**
```
Store Name (lowest)
  ↓
City
  ↓
County
  ↓
State
  ↓
Region (highest)

Example Row:
Store: "Northgate Mall Store"
City: "Seattle"
County: "King County"
State: "Washington"
Region: "Pacific Northwest"
```

**Organizational Hierarchy:**
```
Employee Name (lowest)
  ↓
Manager Name
  ↓
Director Name
  ↓
VP Name
  ↓
Executive VP Name (highest)

Example Row:
Employee: "Jane Doe"
Manager: "Bob Smith"
Director: "Carol White"
VP: "David Lee"
EVP: "Emily Chen"
```

**Date/Time Hierarchy:**
```
Day (lowest)
  ↓
Week
  ↓
Month
  ↓
Quarter
  ↓
Year (highest)

Example Row:
Date: "2023-06-15"
Day of Week: "Thursday"
Week Ending: "2023-06-17"
Month: "June"
Quarter: "Q2"
Year: "2023"
```

### Multiple Hierarchies in Single Dimension

**Store Dimension with Two Hierarchies:**

**Geographic Hierarchy:**
Store → City → State → Country → Region

**Organizational Hierarchy:**
Store → District → Division → Company

```
Example Row:
Store Name: "Downtown Seattle Store"

Geographic Attributes:
City: "Seattle"
State: "Washington"
Country: "United States"
Geographic Region: "Pacific Northwest"

Organizational Attributes:
District Name: "Seattle District"
Division Name: "Western Division"
Company Name: "National Grocery Corp"
```

Both hierarchies coexist peacefully in the same dimension table.

### Best Practices

**DO:**
- Flatten all fixed-depth hierarchy levels as separate attributes
- Use consistent naming: [Level] Description (Brand Description, Category Description)
- Include all levels even if some values repeat (denormalize completely)
- Support multiple independent hierarchies in same dimension
- Ensure each level is truly many-to-one to next level

**DON'T:**
- Normalize hierarchies into separate tables (snowflaking)
- Skip intermediate levels
- Mix hierarchies with correlated attributes into separate dimensions
- Create overly deep hierarchies (>6 levels becomes unwieldy)

### Variable Depth (Ragged) Hierarchies

When hierarchy depth varies (org charts, bill of materials), use bridge tables—covered in advanced dimension patterns.

### When to Use

- Product/service catalogs with classification hierarchies
- Geographic rollups (store, city, state, region)
- Organizational structures (employee to manager to director)
- Time hierarchies (day, week, month, quarter, year)
- Any natural many-to-one rollup relationship

---

## Pattern 3: Indicators and Flags

### Description

Binary or low-cardinality attributes that describe characteristics or conditions.

### Characteristics

- Two to five possible values typically
- Used for filtering and segmentation
- Should be stored as text, not Y/N or 0/1
- Meaningful values improve report readability

### Examples

**Product Attributes:**
```
❌ BAD:  Organic Flag: "Y" / "N"
✅ GOOD: Organic Indicator: "Organic" / "Non-Organic"

❌ BAD:  Gluten Free: TRUE / FALSE
✅ GOOD: Gluten Free Indicator: "Gluten Free" / "Contains Gluten"

❌ BAD:  Active: 1 / 0
✅ GOOD: Product Status: "Active" / "Discontinued"
```

**Customer Attributes:**
```
✅ GOOD: Loyalty Member Indicator: "Loyalty Member" / "Non-Member"
✅ GOOD: Email Opt-In Indicator: "Opted In" / "Opted Out" / "Unknown"
✅ GOOD: Credit Risk Level: "Low Risk" / "Medium Risk" / "High Risk"
```

**Date Attributes:**
```
✅ GOOD: Holiday Indicator: "Holiday" / "Non-Holiday"
✅ GOOD: Weekday Indicator: "Weekday" / "Weekend"
✅ GOOD: Last Day of Month Indicator: "Last Day" / "Not Last Day"
✅ GOOD: Fiscal Month End Indicator: "Month End" / "Not Month End"
```

**Store Attributes:**
```
✅ GOOD: 24 Hour Store Indicator: "24 Hour" / "Limited Hours"
✅ GOOD: Drive-Through Indicator: "Drive-Through" / "No Drive-Through"
```

### Best Practices

**DO:**
- Use full text values: "Organic" not "Y"
- Make values self-explanatory on reports
- Use consistent terminology across similar indicators
- Consider 3-5 value indicators: "High / Medium / Low" instead of just "High / Not High"
- Store "Unknown" or "Not Applicable" instead of NULL

**DON'T:**
- Use Y/N, 1/0, T/F, or other cryptic codes
- Assume users will decode indicators in BI tool
- Use NULL for missing values
- Create too many two-value indicators (consider consolidation)

### When to Use

- Binary characteristics (Organic vs Non-Organic)
- Status indicators (Active vs Inactive)
- Categorizations with few values (Risk Level: Low/Medium/High)
- Special dates or events (Holiday, Weekend, Month End)

---

## Pattern 4: Numeric Descriptors

### Description

Numeric values used primarily for filtering and grouping, not calculations.

### Characteristics

- Numeric data type but used like dimension attributes
- Relatively stable values
- Used for constraints, groupings, bands
- May exist in both dimension and fact tables

### Examples

**Product Dimension:**
```
Standard List Price: 2.99
Package Weight Ounces: 10.75
Shelf Life Days: 730
Units Per Case: 24
Reorder Point Quantity: 100
```

**Store Dimension:**
```
Selling Square Feet: 45000
Parking Spaces: 250
Checkout Lanes: 12
Year Opened: 2015
Distance to Warehouse Miles: 125
```

**Customer Dimension:**
```
Years as Customer: 5
Lifetime Purchase Count: 127
Credit Score: 720
Household Size: 4
Age: 42
```

### Dimension vs Fact Decision

**Store in Dimension When:**
- Used primarily for filtering/grouping
- Value is relatively stable
- Value is attribute of the entity, not measurement event
- Example: Product Standard List Price, Store Square Footage

**Store in Fact When:**
- Used primarily for calculations
- Value changes with each measurement event
- Value is result of business process
- Example: Unit Price on Transaction, Quantity Sold

**Store in Both When:**
- Serves both purposes
- Dimension: Current or typical value for filtering
- Fact: Transaction-time value for calculation
- Example:
  - Product Dimension: Current Standard Price (for filtering/grouping)
  - Sales Fact: Transaction Unit Price (for revenue calculation)

### Best Practices

**DO:**
- Include units in attribute name: Square_Feet not just Square
- Use appropriate precision (no false precision)
- Support banding/grouping in BI layer or with additional attributes
- Document whether value is current or historical

**DON'T:**
- Use numeric descriptors for calculations across fact rows (put in fact table)
- Include highly volatile numbers (change with every transaction)
- Assume users understand units without labels

### When to Use

- Physical measurements (size, weight, capacity)
- Counts (number of items, locations, people)
- Stable financial values (list prices, credit limits)
- Time periods (age, tenure, duration)
- Scores and ratings

---

## Pattern 5: Operational Codes

### Description

Codes from source systems, both preserved and decoded.

### Characteristics

- Alphanumeric identifiers from operational systems
- May contain embedded meaning
- Should be decoded into text descriptions
- Original code preserved as separate attribute

### Examples

**Product Codes:**
```
Product Code (Natural Key): "SKU-12345-A-10"
  Decoded into:
  Product Description: "Campbell's Chicken Noodle Soup 10.75oz"
  Brand Code: "12345"
  Brand Description: "Campbell's"
  Package Type Code: "A"
  Package Type Description: "Can"
  Size Code: "10"
  Size Description: "10.75 oz"
```

**Customer Account Codes:**
```
Account Number (Natural Key): "CUST-2023-WA-00123"
  Preserved as-is, plus:
  Account Type: "Individual"
  Year Opened: "2023"
  State: "Washington"
  Sequential Number: "00123"
```

**Store Codes:**
```
Store Code (Natural Key): "ST-NW-SEA-001"
  Decoded into:
  Store Type: "Standard Store"
  Region Code: "NW"
  Region Description: "Northwest"
  City Code: "SEA"
  City: "Seattle"
  Store Number: "001"
```

### Multi-Part Operational Codes

When codes have embedded meaning:

**Preserve Complete Code:**
```
Product Code: "ABC-123-XL-RED"
```

**Break into Component Attributes:**
```
Manufacturer Code: "ABC"
Manufacturer Description: "Acme Manufacturing"
Style Number: "123"
Size Code: "XL"
Size Description: "Extra Large"
Color Code: "RED"
Color Description: "Red"
```

### Best Practices

**DO:**
- Preserve original operational code as natural key
- Decode all parts into separate descriptive attributes
- Store both code and description versions (Brand Code and Brand Description)
- Handle codes from multiple source systems with source prefix
- Document code structure and meanings

**DON'T:**
- Store only codes without descriptions
- Expect users to understand code meanings
- Rely on codes for user-facing reports
- Lose operational code (needed for source system linkage)

### When to Use

- Source system identifiers (account numbers, product SKUs)
- Codes with embedded business meaning
- Operational classifications (status codes, type codes)
- Multi-source scenarios requiring source identification

---

## Pattern 6: Date Attributes in Dimensions

### Description

Date values stored as dimension attributes (not foreign keys to date dimension).

### Characteristics

- Date data type or foreign key to date dimension
- Represents dates "about" the entity, not business process events
- Used for filtering and grouping by date characteristics
- May need outrigger to date dimension for non-standard calendar attributes

### Examples

**Product Dimension:**
```
Product Introduction Date: 2023-01-15
Product Discontinue Date: NULL (or 9999-12-31 if active)
Last Price Change Date: 2023-06-30
```

**Store Dimension:**
```
Store Open Date: 2015-03-20
Last Remodel Date: 2022-09-15
Next Scheduled Remodel Date: 2025-06-01
Lease Expiration Date: 2030-12-31
```

**Customer Dimension:**
```
Customer Since Date: 2020-05-10
Last Purchase Date: 2023-11-28
Birthday: 1985-07-22
```

**Employee Dimension:**
```
Hire Date: 2018-03-15
Termination Date: NULL
Last Promotion Date: 2022-08-01
Performance Review Due Date: 2024-03-15
```

### Outrigger Pattern for Advanced Filtering

If users need to filter/group by fiscal periods, holidays, or other non-standard calendar attributes:

**Create Date Dimension Outrigger:**
```
Store Dimension:
  Store Key (PK)
  Store Number (NK)
  Store Name
  Store Open Date (date type)
  Store Open Date Key (FK) → Date Dimension

This enables:
  "Show me stores opened in Q4"
  "Show me stores opened on a holiday"
  "Show me stores opened in fiscal period 8"
```

### Best Practices

**DO:**
- Use date data type for simple filtering (year, month, day-of-week)
- Use date dimension foreign key (outrigger) for complex filtering (fiscal periods, holidays)
- Label date attributes clearly (distinguish from fact table date)
- Handle NULL dates appropriately (Unknown Date, Not Applicable, 9999-12-31 for "forever")

**DON'T:**
- Confuse dimension dates with fact table dates (different purposes)
- Create date dimension foreign key unless complex filtering needed
- Store dates as strings (use proper date type)

### When to Use

- Lifecycle dates (open, close, start, end)
- Milestone dates (last promotion, last review)
- Recurring dates (birthday, anniversary)
- Any date that describes the entity itself

---

## Pattern 7: Degenerate Dimensions

### Description

Dimension attributes stored directly in the fact table without a corresponding dimension table.

### Characteristics

- High-cardinality operational identifiers
- No descriptive attributes beyond the identifier itself
- Stored in fact table, not dimension table
- Often transaction/document control numbers
- Part of fact table primary key

### Examples

**Retail Sales:**
```
POS Transaction Number: "TXN-20230615-00123"
  - No dimension table; sits in fact table
  - Groups line items into shopping cart
  - Enables market basket analysis
```

**Order Management:**
```
Order Number: "ORD-2023-00456"
Purchase Order Number: "PO-2023-00789"
  - Stored in fact table
  - No additional attributes to justify dimension table
  - Provides linkage back to operational system
```

**Healthcare:**
```
Claim Number: "CLM-2023-123456"
Patient Visit ID: "VST-20230615-001"
  - Degenerate dimensions in claim fact table
  - No separate dimension tables needed
```

**Telecommunications:**
```
Call ID: "CALL-20230615-1234567"
  - Degenerate dimension in call detail fact table
```

### When NOT Degenerate

If there ARE descriptive attributes beyond the identifier, create a proper dimension table:

**Order Dimension (Not Degenerate):**
```
Order Number (NK)
Order Type: "Web" / "Phone" / "In-Store"
Order Source: "Mobile App" / "Desktop Website"
Order Priority: "Standard" / "Express" / "Rush"
Payment Terms: "Net 30" / "COD" / "Credit Card"
```

This justifies a dimension table because descriptive attributes exist.

### Best Practices

**DO:**
- Use degenerate dimensions for transaction control numbers
- Include in fact table primary key where appropriate
- Use for grouping related fact rows (line items in order)
- Preserve operational identifiers for source system linkage

**DON'T:**
- Create empty dimension tables just to avoid degenerate dimensions
- Confuse with dimension surrogate keys (degenerate dimensions are natural keys)
- Use for low-cardinality attributes (create junk dimension instead)

### When to Use

- Transaction numbers (POS transaction, order number, invoice number)
- Document numbers (bill of lading, claim number, policy number)
- High-cardinality identifiers with no associated attributes
- Grouping keys for line items in parent-child relationships

---

## Pattern 8: Special Attributes

### SCD Tracking Attributes

When using Type 2 Slowly Changing Dimensions:

```
Row Effective Date: 2023-01-15
Row Expiration Date: 9999-12-31 (current row) or 2023-06-30 (expired row)
Current Row Indicator: "Current" / "Expired"
Row Change Reason: "Address Change" / "Name Change" / "Attribute Update"
```

### Audit and Lineage Attributes

```
Source System Code: "SAP" / "CRM" / "Legacy"
Record Create Timestamp: 2023-06-15 10:30:00
Record Update Timestamp: 2023-11-20 14:45:00
ETL Batch ID: 20231120-001
Data Quality Score: 95.5
```

### Role-Playing Attributes

Same dimension plays different roles:

```
Date Dimension (role-playing):
  - Order Date
  - Ship Date
  - Delivery Date
  - Invoice Date

Employee Dimension (role-playing):
  - Sales Rep
  - Account Manager
  - Support Contact
```

Implemented via views or aliases:
```sql
CREATE VIEW Order_Date AS SELECT * FROM Date_Dimension;
CREATE VIEW Ship_Date AS SELECT * FROM Date_Dimension;
```

### Junk Dimension Attributes

Low-cardinality flags consolidated into single dimension:

```
Transaction Profile Dimension:
  - Payment Method: "Cash" / "Credit" / "Debit" / "Check"
  - Gift Wrap Indicator: "Gift Wrapped" / "Not Wrapped"
  - Gift Receipt Indicator: "Gift Receipt" / "Standard Receipt"
  - Tax Exempt Indicator: "Tax Exempt" / "Taxable"

Instead of 4 separate tiny dimensions, consolidate into one.
```

---

## Attribute Selection Guidelines

### Questions to Ask When Identifying Attributes

1. **Will users filter on this?**
   - "Show me organic products"
   - "Show me holiday sales"
   - "Show me customers in Seattle"

2. **Will users group by this?**
   - "Sales by brand"
   - "Sales by store district"
   - "Sales by customer age range"

3. **Will this appear as a report label?**
   - Column headers, row headers, page headers
   - Makes reports self-explanatory

4. **Does this take on a single value at the fact grain?**
   - If no, it's not a valid dimension attribute at this grain
   - May need different grain or separate fact table

5. **Is this available in the source system?**
   - If yes, document source table/column
   - If no, document transformation or derivation rule

6. **Does this provide meaningful business value?**
   - Don't include attributes just because they exist
   - Focus on what supports decision-making

### Comprehensive Attribute Checklist

For each dimension, consider these attribute categories:

**Identity Attributes:**
- [ ] Surrogate key (primary key)
- [ ] Natural key from source system
- [ ] Durable supernatural key (if needed)
- [ ] Alternate identifiers

**Descriptive Attributes:**
- [ ] Primary description (name, title, label)
- [ ] Long description
- [ ] Short description or abbreviation
- [ ] Code and decoded values

**Hierarchical Attributes:**
- [ ] All levels of fixed hierarchies (flattened)
- [ ] Multiple independent hierarchies (if applicable)

**Classification Attributes:**
- [ ] Categories and subcategories
- [ ] Types and subtypes
- [ ] Status and state

**Indicator Attributes:**
- [ ] Binary indicators (active/inactive, yes/no)
- [ ] Multi-value indicators (low/medium/high)

**Numeric Descriptors:**
- [ ] Physical measurements (size, weight, capacity)
- [ ] Counts (quantity, occurrences)
- [ ] Stable financial values (list price, credit limit)

**Date Attributes:**
- [ ] Lifecycle dates (start, end, open, close)
- [ ] Milestone dates (last update, next review)

**Audit Attributes:**
- [ ] Source system identifier
- [ ] Record timestamps
- [ ] Data quality indicators
- [ ] SCD tracking columns (effective date, expiration date, current flag)

---

## Common Mistakes to Avoid

### Mistake 1: Insufficient Descriptive Attributes

**Wrong:**
```
Product Dimension:
  Product Key
  Product Code
  Category Code
```

**Right:**
```
Product Dimension:
  Product Key
  Product Code (NK)
  Product Description
  Product Long Description
  Brand Description
  Category Description
  Department Description
  Package Type Description
  Size Description
  Flavor Description
  ... (30-50+ attributes)
```

### Mistake 2: Storing Only Codes Without Descriptions

**Wrong:**
```
Store Dimension:
  Store Key
  Store Number
  Format: "SS" / "NM" / "DS"
  Status: "A" / "I" / "P"
```

**Right:**
```
Store Dimension:
  Store Key
  Store Number (NK)
  Store Format Code: "SS"
  Store Format Description: "Super Store"
  Store Status Code: "A"
  Store Status Description: "Active"
```

### Mistake 3: Normalizing Hierarchies (Snowflaking)

**Wrong:**
```
Product Dimension → Brand Dimension → Category Dimension → Department Dimension
(Four separate tables with joins)
```

**Right:**
```
Product Dimension:
  Product Description
  Brand Description
  Category Description
  Department Description
(All in one denormalized table)
```

### Mistake 4: Not Handling NULLs Properly

**Wrong:**
```
Customer Dimension:
  Gender: NULL (for unknown)
  Income Range: NULL (for not provided)
```

**Right:**
```
Customer Dimension:
  Gender: "Unknown"
  Income Range: "Not Provided"
```

### Mistake 5: Using Cryptic Indicators

**Wrong:**
```
Product Dimension:
  Organic: "Y" / "N"
  Active: 1 / 0
  Premium: TRUE / FALSE
```

**Right:**
```
Product Dimension:
  Organic Indicator: "Organic" / "Non-Organic"
  Product Status: "Active" / "Discontinued"
  Price Tier: "Premium" / "Standard" / "Value"
```

---

## Summary

Rich, descriptive dimension attributes are essential for successful dimensional models:

**Key Principles:**
1. **Verbose is better than cryptic** - Use full text descriptions
2. **Denormalize hierarchies** - Flatten many-to-one rollups
3. **Decode all codes** - Make values self-explanatory
4. **Eliminate NULLs** - Use "Unknown" or "Not Applicable"
5. **Include many attributes** - 30-50+ attributes per dimension is common

**Attribute Categories:**
- Descriptive text (foundation of dimensions)
- Hierarchical (enable drill-down)
- Indicators/flags (meaningful text values)
- Numeric descriptors (for filtering/grouping)
- Operational codes (preserved and decoded)
- Dates (entity lifecycle dates)
- Degenerate (transaction control numbers)
- Special (SCD, audit, role-playing, junk)

**Questions to Guide Attribute Selection:**
- Will users filter on this?
- Will users group by this?
- Will this appear as a report label?
- Does it take single value at fact grain?
- Does it provide meaningful business value?

Dimension attributes are the primary drivers of the user's BI experience. Invest time making them as complete, descriptive, and user-friendly as possible.
