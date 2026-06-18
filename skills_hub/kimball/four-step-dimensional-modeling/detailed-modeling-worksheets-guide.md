# Detailed Modeling Worksheets Guide

## Purpose

This guide provides practical templates and detailed guidance for documenting dimensional models during the detailed design phase. Use these worksheets to capture table-by-table, column-by-column specifications that will serve as blueprints for ETL development and communicate design decisions to stakeholders.

## When to Use This Guide

Use this guide when:
- Moving from high-level bubble charts to detailed dimension and fact table specifications
- Documenting design decisions during collaborative modeling sessions
- Creating source-to-target mapping documentation for ETL developers
- Communicating dimensional model details to IT and business reviewers

## Dimension Table Worksheet Template

### Essential Components

Every dimension table worksheet must include:

**1. Table-Level Information**
- **Table Name**: Logical name (e.g., `Product Dimension`, `Date Dimension`)
- **Physical Table Name**: Actual database table name (e.g., `DIM_PRODUCT`, `DIM_DATE`)
- **Description**: Business purpose and content (2-3 sentences)
- **Source Systems**: Primary and secondary source systems
- **Estimated Row Count**: Current and projected (1 year, 3 years)
- **Update Frequency**: Daily, weekly, monthly, real-time
- **Primary SCD Type**: Predominant slowly changing dimension strategy

**2. Column-Level Information** (for each attribute)

| Column Name | Description | Sample Values | Data Type | Length | Source System | Source Column | SCD Type | Null? | Business Rules |
|-------------|-------------|---------------|-----------|--------|---------------|---------------|----------|-------|----------------|

**Column Descriptions:**

- **Column Name**: Logical attribute name (business-friendly)
- **Description**: Business definition and purpose
- **Sample Values**: 3-5 representative values from source or desired output
- **Data Type**: VARCHAR, INTEGER, DATE, DECIMAL, etc.
- **Length**: For character fields (e.g., VARCHAR(50))
- **Source System**: Which operational system provides this data
- **Source Column**: Actual source table.column reference
- **SCD Type**: 0, 1, 2, 3, 4, 5, 6, or 7 (see SCD type definitions below)
- **Null?**: Yes/No (avoid nulls in dimensions; use "Unknown" or "Not Applicable")
- **Business Rules**: Transformations, derivations, or special handling

### Example: Product Dimension Worksheet

**Table-Level Information:**
- **Table Name**: Product Dimension
- **Physical Table Name**: DIM_PRODUCT
- **Description**: Contains all products (SKUs) sold across all retail stores, including current and historical products. Includes complete merchandise hierarchy from SKU through brand, category, and department. Updated nightly from product master system.
- **Source Systems**: Primary: Product Master (SAP); Secondary: Merchandising Planning System
- **Estimated Row Count**: Current: 300,000 rows; 1-year: 350,000; 3-year: 450,000
- **Update Frequency**: Daily (nightly batch)
- **Primary SCD Type**: Type 2 for most attributes; Type 1 for some

**Column-Level Specifications:**

| Column Name | Description | Sample Values | Data Type | Length | Source | Source Column | SCD Type | Null? | Business Rules |
|-------------|-------------|---------------|-----------|--------|--------|---------------|----------|-------|----------------|
| Product Key (PK) | Surrogate key | 1, 2, 3, ... | INTEGER | - | DW/BI | Generated | N/A | No | Sequentially assigned |
| Product Code (NK) | Natural key from source | SKU-12345, SKU-67890 | VARCHAR | 20 | SAP | PRODUCT.PROD_ID | N/A | No | Preserve original format |
| Product Description | Full product name | Campbell's Chicken Noodle Soup 10.75oz | VARCHAR | 100 | SAP | PRODUCT.PROD_DESC | Type 2 | No | Decode from source |
| SKU Number | Stock keeping unit | 1234567890123 | VARCHAR | 15 | SAP | PRODUCT.SKU_NBR | Type 0 | No | Barcode/UPC |
| Brand Description | Product brand | Campbell's, Progresso, Private Label | VARCHAR | 50 | SAP | BRAND.BRAND_NAME | Type 2 | No | Join to brand table |
| Category Description | Merchandise category | Soups, Canned Goods, Frozen Foods | VARCHAR | 50 | SAP | CATEGORY.CAT_NAME | Type 2 | No | Join to category table |
| Department Description | Store department | Grocery, Frozen, Dairy | VARCHAR | 50 | SAP | DEPT.DEPT_NAME | Type 1 | No | Join to department table |
| Package Type Description | Packaging | Can, Bottle, Box, Bag | VARCHAR | 20 | SAP | PRODUCT.PKG_TYPE | Type 2 | No | Decode PKG_TYPE code |
| Package Size | Size with unit | 10.75 oz, 16 oz, 1 lb | VARCHAR | 20 | SAP | PRODUCT.PKG_SIZE | Type 2 | No | Concatenate size + unit |
| Organic Indicator | Organic certification | Organic, Non-Organic | VARCHAR | 15 | SAP | PRODUCT.ORGANIC_FLG | Type 2 | No | Decode Y/N to text |
| Gluten Free Indicator | Gluten free status | Gluten Free, Contains Gluten | VARCHAR | 20 | SAP | PRODUCT.GF_FLG | Type 2 | No | Decode Y/N to text |
| Current Price | Current retail price | 2.99, 3.49, 1.99 | DECIMAL | 10,2 | SAP | PRODUCT.CURR_PRICE | Type 1 | No | Most recent price |
| Row Effective Date | SCD Type 2 start date | 2023-01-15 | DATE | - | DW/BI | Generated | N/A | No | ETL process date |
| Row Expiration Date | SCD Type 2 end date | 2023-06-30, 9999-12-31 | DATE | - | DW/BI | Generated | N/A | No | 9999-12-31 for current |
| Current Row Indicator | Is current version? | Current, Expired | VARCHAR | 10 | DW/BI | Generated | N/A | No | Decode Y/N to text |

**Special Considerations:**
- Department uses Type 1 because business wants all historical sales to reflect current department structure
- Price stored as Type 1 attribute representing current price; historical prices tracked separately in fact table
- Multiple hierarchies: Merchandise (SKU→Brand→Category→Department) and Package (Size, Type)
- Include "Unknown Product" row with Product Key = -1 for late-arriving dimension handling

---

## Fact Table Worksheet Template

### Essential Components

Every fact table worksheet must include:

**1. Table-Level Information**
- **Table Name**: Logical name (e.g., `Retail Sales Fact Table`)
- **Physical Table Name**: Actual database table name (e.g., `FACT_RETAIL_SALES`)
- **Fact Table Type**: Transaction, Periodic Snapshot, or Accumulating Snapshot
- **Grain Statement**: Clear business declaration of what one row represents
- **Description**: Business purpose and measurement events captured
- **Source Systems**: Primary and secondary source systems
- **Estimated Row Count**: Current and projected (1 year, 3 years)
- **Load Frequency**: Daily, hourly, real-time, monthly
- **Historical Depth**: How many years of history retained
- **Partitioning Strategy**: By date, by store, composite

**2. Foreign Key Columns**

| Foreign Key Column | References | Null? | Default for Unknown |
|--------------------|------------|-------|---------------------|

**3. Degenerate Dimensions** (if any)

| Degenerate Dimension | Description | Sample Values | Data Type | Source |
|----------------------|-------------|---------------|-----------|--------|

**4. Fact Columns**

| Fact Name | Description | Sample Values | Additivity | Data Type | Precision | Source System | Source Column | Calculation/Derivation | Null Handling |
|-----------|-------------|---------------|------------|-----------|-----------|---------------|---------------|------------------------|---------------|

**Fact Column Descriptions:**

- **Fact Name**: Logical metric name
- **Description**: Business definition
- **Sample Values**: Typical numeric values
- **Additivity**: Additive, Semi-Additive, or Non-Additive
- **Data Type**: DECIMAL, INTEGER, FLOAT
- **Precision**: e.g., DECIMAL(15,2) for dollars
- **Source System**: Where measurement originates
- **Source Column**: Actual source field
- **Calculation/Derivation**: Formula if derived
- **Null Handling**: Leave null or substitute zero/default

### Example: Retail Sales Fact Table Worksheet

**Table-Level Information:**
- **Table Name**: Retail Sales Fact Table
- **Physical Table Name**: FACT_RETAIL_SALES
- **Fact Table Type**: Transaction Fact Table
- **Grain Statement**: One row per individual product on a POS transaction (one line item per shopping cart transaction)
- **Description**: Captures all retail sales transactions at the individual product level as scanned at POS registers. Enables analysis of product sales performance by store, date, promotion, and customer. Source is POS system transaction log.
- **Source Systems**: Primary: POS Transaction System; Secondary: Product Master, Promotion System
- **Estimated Row Count**: Current: 2 billion rows/year; 1-year: 2.5B; 3-year: 8B cumulative
- **Load Frequency**: Daily (nightly batch load of previous day's transactions)
- **Historical Depth**: 7 years online; older data archived
- **Partitioning Strategy**: Range partitioned by Date Key (monthly partitions)

**Foreign Key Columns:**

| Foreign Key Column | References | Null? | Default for Unknown |
|--------------------|------------|-------|---------------------|
| Date Key (FK) | DIM_DATE.Date_Key | No | 0 (Unknown Date row) |
| Product Key (FK) | DIM_PRODUCT.Product_Key | No | -1 (Unknown Product row) |
| Store Key (FK) | DIM_STORE.Store_Key | No | -1 (Unknown Store row) |
| Promotion Key (FK) | DIM_PROMOTION.Promotion_Key | No | 0 (No Promotion row) |
| Cashier Key (FK) | DIM_CASHIER.Cashier_Key | No | 0 (No Cashier/Self-Service row) |
| Payment Method Key (FK) | DIM_PAYMENT_METHOD.Payment_Key | No | -1 (Unknown Payment row) |

**Degenerate Dimensions:**

| Degenerate Dimension | Description | Sample Values | Data Type | Source |
|----------------------|-------------|---------------|-----------|--------|
| POS Transaction Number | Unique transaction identifier from POS system | TXN-2023011500001, TXN-2023011500002 | VARCHAR(20) | POS.TRANS_ID |

**Fact Columns:**

| Fact Name | Description | Sample Values | Additivity | Data Type | Precision | Source | Source Column | Calculation | Null Handling |
|-----------|-------------|---------------|------------|-----------|-----------|--------|---------------|-------------|---------------|
| Sales Quantity | Number of units sold | 1, 2, 5, 10 | Additive | INTEGER | - | POS | LINE_ITEM.QTY | None | Leave null if missing |
| Unit Regular Price | Regular price per unit before discounts | 2.99, 3.49, 1.99 | Non-Additive | DECIMAL | 10,2 | POS | LINE_ITEM.REG_PRICE | None | Leave null |
| Unit Discount Amount | Discount per unit | 0.00, 0.50, 1.00 | Non-Additive | DECIMAL | 10,2 | POS | LINE_ITEM.DISC_AMT | None | Default to 0.00 |
| Unit Net Price | Net price per unit after discount | 2.99, 2.49, 1.99 | Non-Additive | DECIMAL | 10,2 | POS | LINE_ITEM.NET_PRICE | REG_PRICE - DISC_AMT | Leave null |
| Extended Discount Amount | Total discount for line item | 0.00, 1.00, 5.00 | Additive | DECIMAL | 15,2 | Derived | - | Qty * Unit_Disc_Amt | Default to 0.00 |
| Extended Sales Amount | Total sales revenue for line item | 2.99, 4.98, 19.95 | Additive | DECIMAL | 15,2 | POS | LINE_ITEM.EXT_AMT | Qty * Unit_Net_Price | Leave null |
| Extended Cost Amount | Total product cost for line item | 1.50, 2.25, 12.00 | Additive | DECIMAL | 15,2 | Product Master | - | Qty * Product.Std_Cost | Leave null |
| Gross Profit Amount | Profit for line item (revenue - cost) | 1.49, 2.73, 7.95 | Additive | DECIMAL | 15,2 | Derived | - | Ext_Sales - Ext_Cost | Leave null |

**Special Considerations:**
- **Grain Notes**: Assumes POS system consolidates multiple scans of same product into single line item. If not, grain would be "one row per scan of a product."
- **Non-Additive Facts**: Unit prices stored for reporting individual transaction details but should not be summed. Use extended amounts for aggregations.
- **Derived Facts**: Extended amounts and gross profit are derived but stored physically for consistency and performance
- **Null Facts**: Leave nulls in fact columns (not foreign keys) so aggregate functions work correctly
- **Referential Integrity**: All foreign keys must exist in dimension tables; use Unknown/Not Applicable dimension rows, never nulls

---

## SCD Type Quick Reference

Include this reference in dimension worksheets:

| SCD Type | Name | Description | When to Use |
|----------|------|-------------|-------------|
| Type 0 | Retain Original | Never changes; original value preserved | Immutable attributes (birth date, original hire date, SKU) |
| Type 1 | Overwrite | Overwrites old value with new; no history | Corrections, attributes where history not needed |
| Type 2 | Add New Row | Creates new dimension row; tracks full history | Most common; use when history is important |
| Type 3 | Add New Attribute | Adds column for previous value; limited history | Track only one prior value (e.g., prior territory) |
| Type 4 | Add Mini-Dimension | Rapidly changing attributes in separate table | Rapidly changing demographics, scores |
| Type 5 | Mini-Dimension + Type 1 Outrigger | Hybrid of Type 4 + Type 1 | Complex mini-dimension scenarios |
| Type 6 | Type 1 + Type 2 + Type 3 | Combination approach | Mix of current, historical, and prior value |
| Type 7 | Dual Type 1 and Type 2 | Current and historical views | Need both current and point-in-time views |

---

## Worksheet Development Process

**Step 1: Start with Date Dimension**
- Easiest dimension to build consensus
- Creates early success and team momentum
- Establishes worksheet format and conventions

**Step 2: Tackle Other Simple Dimensions**
- Store dimension, payment method, cashier
- Build team confidence with straightforward designs
- Refine documentation standards

**Step 3: Address Complex Core Dimensions**
- Product, customer, promotion
- Allow adequate time for discussion
- Involve data stewards for naming and definitions

**Step 4: Design Fact Tables Last**
- After all dimensions are understood
- Grain clarity makes fact selection straightforward
- Document additivity for every fact

**Step 5: Review and Validate**
- Walk through worksheets with business users
- Verify with source system experts
- Update based on feedback

---

## Documentation Best Practices

### Naming Conventions

**Table Names:**
- Use business-friendly names in worksheets (e.g., "Product Dimension")
- Physical names can use prefixes (e.g., DIM_PRODUCT, FACT_SALES)
- Be consistent across all tables

**Column Names:**
- Avoid abbreviations unless universally understood
- Use full words: "Description" not "Desc", "Indicator" not "Ind"
- Suffix with data classification: "_Key" for keys, "_Date" for dates, "_Amount" for currency
- Make column names unique across dimensions when possible to avoid confusion

**Example Naming Standards:**
- Primary Keys: `[Dimension]_Key` (e.g., Product_Key, Store_Key)
- Foreign Keys: Same as referenced primary key
- Natural Keys: `[Dimension]_Code` or `[Dimension]_Number` (e.g., Product_Code, Store_Number)
- Descriptive Attributes: `[Entity]_[Attribute]_Description` (e.g., Brand_Description, Category_Description)
- Indicators/Flags: `[Attribute]_Indicator` (e.g., Organic_Indicator, Current_Row_Indicator)
- Amounts: `[Metric]_Amount` (e.g., Sales_Amount, Discount_Amount, Cost_Amount)
- Quantities: `[Metric]_Quantity` (e.g., Sales_Quantity, Order_Quantity)

### Sample Values

**Why Important:**
- Validates understanding between IT and business
- Catches data quality issues early
- Documents expected domain values
- Guides ETL transformation logic

**Best Practices:**
- Provide 3-5 representative values for each attribute
- Show decoded text values, not operational codes
- Include edge cases (minimum, maximum, typical)
- Document "Unknown" and "Not Applicable" special rows

### Business Rules Documentation

**Capture:**
- Source-to-target transformations (e.g., "Decode Y/N to 'Yes'/'No'")
- Derived calculations (e.g., "Gross Profit = Sales - Cost")
- Data quality rules (e.g., "Default to 'Unknown' if source is null")
- Conformed dimension agreements (e.g., "Use enterprise customer dimension")
- Special handling (e.g., "Current row has expiration date = 9999-12-31")

**Example Business Rules:**

| Attribute | Business Rule |
|-----------|---------------|
| Product Description | Concatenate PRODUCT.SHORT_DESC + " " + PRODUCT.SIZE_DESC; trim to 100 characters if needed |
| Organic Indicator | Decode PRODUCT.ORGANIC_FLG: 'Y' → 'Organic', 'N' → 'Non-Organic', NULL → 'Unknown' |
| Department Description | Join to DEPARTMENT table; use Type 1 SCD so all history reflects current department structure per business requirement dated 2023-03-15 |
| Row Expiration Date | Set to 9999-12-31 for current row; set to (new row effective date - 1 day) when creating new Type 2 row |

---

## Issues Tracking Log

Maintain a separate issues log document linked to each worksheet:

**Issues Log Template:**

| Issue ID | Date Raised | Table | Column | Issue Description | Priority | Assigned To | Status | Resolution | Date Resolved |
|----------|-------------|-------|--------|-------------------|----------|-------------|--------|------------|---------------|

**Example Issues:**

| Issue ID | Date | Table | Column | Description | Priority | Assigned | Status | Resolution | Resolved |
|----------|------|-------|--------|-------------|----------|----------|--------|------------|----------|
| DIM-001 | 2023-01-10 | Product | Brand Description | Source brand table missing for 5% of products | High | J.Smith | Closed | Use 'Unknown Brand' for missing; ETL data quality report to track | 2023-01-15 |
| DIM-002 | 2023-01-12 | Product | Organic Indicator | Business wants to track history of organic certification changes | Medium | M.Johnson | Closed | Changed from Type 1 to Type 2 SCD | 2023-01-18 |
| FACT-001 | 2023-01-20 | Sales | Extended Cost Amount | Product cost not available at transaction time | High | K.Lee | Closed | Use current cost from product master; add footnote that historical cost not available | 2023-01-25 |

**Issue Priorities:**
- **Critical**: Blocks design progress; must resolve immediately
- **High**: Significant impact on design; resolve within 1-2 days
- **Medium**: Important but workaround exists; resolve within 1 week
- **Low**: Nice to have; resolve before implementation

---

## Review Meeting Checklists

### IT Review Checklist

**Before the meeting:**
- [ ] Provide dimensional modeling education materials
- [ ] Send worksheets and bus matrix 2-3 days in advance
- [ ] Prepare presentation with bubble charts and key details
- [ ] Compile list of open issues

**During the meeting:**
- [ ] Review bus matrix for enterprise context
- [ ] Walk through high-level bubble charts
- [ ] Review dimension worksheets table-by-table
- [ ] Review fact table worksheets
- [ ] Discuss open issues for each table
- [ ] Capture new issues and recommendations
- [ ] Assign action items

**After the meeting:**
- [ ] Update worksheets based on feedback
- [ ] Resolve or assign all new issues
- [ ] Distribute meeting notes and updated documents

### Business User Review Checklist

**Before the meeting:**
- [ ] Prepare business-focused presentation (avoid technical jargon)
- [ ] Create sample reports showing how model answers business questions
- [ ] Send high-level bubble charts in advance
- [ ] Prepare hierarchy diagrams for key dimensions

**During the meeting:**
- [ ] Review bus matrix as data roadmap
- [ ] Present high-level bubble charts
- [ ] Show hierarchical drill paths within dimensions
- [ ] Demonstrate sample queries against requirements
- [ ] Review critical dimension attributes (customer, product)
- [ ] Validate business terminology and definitions
- [ ] Capture feedback and requests

**After the meeting:**
- [ ] Update column names based on business terminology
- [ ] Adjust attribute definitions for clarity
- [ ] Add newly identified attributes or facts
- [ ] Schedule follow-up if needed

---

## Digital Templates

Worksheet templates in Excel format are available at the Kimball Group website (www.kimballgroup.com) under Tools and Utilities. These templates include:

- Dimension table worksheet with pre-formatted columns
- Fact table worksheet with pre-formatted columns
- Issues tracking log template
- Bus matrix template with detailed fact/dimension annotations

---

## Common Mistakes to Avoid

### Documentation Mistakes

1. **Insufficient Sample Values**: Listing only one sample value doesn't reveal data patterns or edge cases
2. **Missing Business Rules**: Undocumented transformation logic leads to ETL errors and inconsistencies
3. **Vague Descriptions**: Generic descriptions like "Product description" don't help users understand content
4. **Ignoring SCD Strategy**: Not specifying SCD type for each dimension attribute causes confusion during ETL design
5. **Null Handling Not Specified**: Ambiguity about null handling leads to inconsistent ETL implementations
6. **Physical Names Only**: Using only database table/column names makes worksheets incomprehensible to business users
7. **No Source Documentation**: Missing source system and column references makes ETL development difficult

### Process Mistakes

1. **Skipping Date Dimension First**: Starting with complex dimensions reduces early team momentum
2. **Designing Facts Before Dimensions**: Fact design is easier after dimension design is complete
3. **No Issues Log**: Trying to track issues in email or memory leads to forgotten decisions
4. **Insufficient Review Cycles**: One review is rarely enough; plan for 2-3 review iterations
5. **Working in Isolation**: Modeling without business representatives produces data-driven, not requirement-driven, designs
6. **Tool Over-Emphasis**: Jumping to modeling tools before consensus on content slows iteration
7. **Postponing Tough Decisions**: Marking issues as "TBD" indefinitely prevents design completion

---

## Deliverables Checklist

Before considering dimensional modeling complete, verify:

### High-Level Deliverables
- [ ] Updated enterprise bus matrix
- [ ] Bubble chart diagram for each fact table
- [ ] Hierarchy diagrams for complex dimensions

### Detailed Deliverables
- [ ] Dimension worksheet for each dimension table
- [ ] Fact table worksheet for each fact table
- [ ] Issues log with all issues resolved or explicitly deferred
- [ ] Naming conventions document
- [ ] SCD strategy summary by dimension

### Review Documentation
- [ ] IT review meeting notes
- [ ] Business user review meeting notes
- [ ] Sign-off from key stakeholders

### Handoff to Next Phase
- [ ] Source-to-target mapping document (from worksheets)
- [ ] Data quality assessment results
- [ ] Physical design recommendations

---

## Summary

Detailed modeling worksheets are the critical bridge between high-level dimensional design and physical implementation. They serve as:

- **Communication tool**: Translate technical designs to business stakeholders
- **ETL blueprint**: Provide source-to-target specifications for ETL developers
- **Design validation**: Document design decisions and trade-offs for future reference
- **Quality assurance**: Capture business rules to ensure consistent implementation

Invest adequate time in thorough worksheet documentation. The clarity and completeness of these worksheets directly determines the success of downstream ETL development and the quality of the resulting dimensional model.

The worksheets should be living documents that evolve as you learn more about requirements and source data. Plan for multiple iterations and regular updates based on design session discoveries and review feedback.
