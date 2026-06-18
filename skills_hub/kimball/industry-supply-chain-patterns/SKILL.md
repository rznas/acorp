---
name: industry-supply-chain-patterns
description: Apply dimensional modeling patterns for supply chain processes including inventory management (periodic/transaction/accumulating snapshots), procurement transactions, and order fulfillment pipelines. Use when designing data warehouses for manufacturing, distribution, retail inventory, purchasing, or order management systems.
---

# Industry-Specific Patterns - Supply Chain

## Overview

Supply chain dimensional modeling addresses three critical business processes that form the backbone of most product-based organizations:

1. **Inventory Management** - Tracking product levels, movement velocity, and storage costs
2. **Procurement** - Managing vendor relationships, purchase orders, receipts, and payments
3. **Order Management** - Processing orders, shipments, invoices, and the complete order fulfillment pipeline

These processes are interconnected through the value chain and share conformed dimensions (product, vendor, customer, date), making them ideal candidates for integrated dimensional models using the enterprise data warehouse bus architecture.

## When to Use This Skill

Apply this skill when you need to:

- **Design inventory dimensional models** for warehouses, stores, or distribution centers
- **Model procurement processes** for purchase requisitions, POs, receipts, and vendor payments
- **Create order management schemas** for orders, shipments, and invoicing
- **Analyze product movement** through the supply chain pipeline
- **Track vendor performance** including fill rates, on-time delivery, and pricing compliance
- **Measure order fulfillment velocity** from order to shipment
- **Calculate profitability** at the invoice line level including activity-based costs
- **Support multi-currency** and multi-unit-of-measure reporting

**Lifecycle Stage:** Dimensional Modeling
**Industry Applicability:** Manufacturing, distribution, retail, wholesale, e-commerce

---

## Process 1: Inventory Management

Inventory is typically modeled using **three complementary fact table types**, each providing a different analytical perspective.

### Inventory Periodic Snapshot

**Best For:** Long-running, continuously replenished inventory scenarios
**Grain:** One row per product per location per time period (day/week/month)

<grain_declaration>
"Daily snapshot of product inventory levels by store" or
"Weekly snapshot of product inventory levels by warehouse"
</grain_declaration>

#### Four-Step Design Process

**Step 1: Select Business Process**
Inventory snapshotting at regular intervals (daily, weekly, monthly)

**Step 2: Declare the Grain**
One row per product per location per snapshot period

**Step 3: Identify Dimensions**
- Date (snapshot date)
- Product (with inventory-specific attributes like minimum reorder quantity, storage requirements)
- Location (store, warehouse, or distribution center)

**Step 4: Identify Facts**

<facts_categories>
**Balance Facts (Semi-Additive):**
- Quantity on hand (semi-additive - NOT additive across dates)
- Inventory value at cost
- Inventory value at latest selling price

**Flow Facts (Fully Additive):**
- Quantity sold (during snapshot period)
- Beginning balance quantity
- Ending balance quantity
- Inventory change/delta

**Derived Metrics:**
- Number of inventory turns = Total quantity sold / Average quantity on hand
- Days supply = Final quantity on hand / Average quantity sold
</facts_categories>

#### Semi-Additive Facts Critical Concept

<semi_additive_explanation>
**Why Inventory Balances Are Semi-Additive:**

Inventory levels and all balance/level measures (financial account balances, room temperatures, etc.) are:
- **Additive** across products and locations (can sum across these dimensions)
- **NOT additive** across dates (cannot sum balances across time periods)

**Example:** Your checking account has $50 on Monday, $50 on Tuesday, and $100 on Wednesday. You CANNOT add these to get $200 as your balance. The correct aggregation across dates is **averaging**.

**SQL Challenge:** You cannot use `AVG()` function directly because it divides by all rows received (products × locations × dates), not just the number of time periods. Must divide `SUM(quantity_on_hand)` by the count of distinct dates.

**OLAP Solution:** OLAP cubes allow definition of aggregation rules per measure, making semi-additive measures easier to handle.
</semi_additive_explanation>

#### Design Considerations

**Density Management:**
- Periodic snapshots create **dense fact tables** (row for every product/location/period combination)
- For retailer with 60,000 products × 100 stores × daily snapshots = 6 million rows per day
- Consider **time-based rollup strategy**: Keep 60 days daily + historical weekly snapshots in separate tables

### Inventory Transaction Fact Table

**Best For:** Analyzing frequency and timing of specific inventory movements
**Grain:** One row per inventory transaction

<inventory_transaction_types>
Common warehouse inventory transactions:
- Receive product from vendor
- Place product into inspection hold
- Release product from inspection hold
- Return product to vendor (failed inspection)
- Place product in bin
- Pick product from bin
- Package product for shipment
- Ship product to customer
- Receive product from customer (return)
- Return product to inventory from customer return
- Remove product from inventory (damage, obsolescence)
</inventory_transaction_types>

**Dimensions:**
- Transaction date
- Product
- Warehouse/location
- Vendor (if applicable)
- Transaction type (dimension with one row per transaction type)

**Facts:**
- Transaction quantity (positive or negative impact on inventory)

**Limitation:** Impractical as the sole basis for inventory analysis. Reconstructing exact inventory position by rolling forward all transactions is too cumbersome for broad analytic questions.

<design_principle>
**Remember:** There's more to life than transactions alone. Some form of snapshot table to give a cumulative view of a process often complements a transaction fact table.
</design_principle>

### Inventory Accumulating Snapshot

**Best For:** Tracking product lots/serial numbers through warehouse pipeline
**Grain:** One row per product lot or serial number
**Requirements:** Must be able to distinguish products from different shipments (lot number, serial number, or shipment identifier)

**Pipeline Milestones Captured as Multiple Dates:**
- Received date
- Inspection completed date
- Bin placement date
- Pick date
- Package date
- Ship date

**Dimensions:**
- Multiple role-playing date dimensions (one per milestone)
- Product
- Warehouse
- Vendor
- Destination/customer

**Facts:**
- Received quantity
- Inspected quantity
- Rejected quantity
- Shipped quantity
- Days in inspection (lag calculation)
- Days in storage (lag calculation)
- Days receiving to shipping (total pipeline lag)

**Key Characteristic:** **Fact table rows are updated repeatedly** as product moves through milestones until completely depleted from warehouse.

---

## Process 2: Procurement

Procurement encompasses purchase requisitions, purchase orders, receipts, and vendor payments.

### Single vs. Multiple Transaction Fact Tables Decision

When faced with multiple transaction types, evaluate:

<decision_criteria>
**Use MULTIPLE fact tables when:**
- Users view activities as separate, distinct business processes
- Different source systems capture metrics with unique granularities
- Transaction types have varied dimensionality
- Separate control numbers exist for each step (strong indicator)

**Use SINGLE fact table when:**
- All transactions are part of a single unified business process
- Consistent dimensionality across all transaction types
- Users want integrated view more than separate process views
</decision_criteria>

**Recommended Approach for Procurement:** Multiple fact tables (purchase orders, shipping notices, warehouse receipts, vendor payments)

### Procurement Bus Matrix

The enterprise data warehouse bus matrix should show:

| Business Process | Date | Product | Vendor | Contract Terms | Warehouse | Employee | Customer | Payment Terms |
|-----------------|------|---------|--------|----------------|-----------|----------|----------|---------------|
| Purchase Requisitions | Requisition Date | ✓ | ✓ | ✓ | ✓ (requesting) | Requisitioner | | |
| Purchase Orders | PO Date | ✓ | ✓ | ✓ | ✓ (receiving) | Purchasing Agent | | |
| Shipping Notices | Ship Date | ✓ | ✓ | | ✓ (destination) | | | |
| Warehouse Receipts | Receipt Date | ✓ | ✓ | | ✓ | Receiving Clerk | | |
| Vendor Payments | Payment Date | ✓ | ✓ | ✓ | | Accounts Payable | | ✓ (payment terms) |

### Multiple Fact Tables Design

**Benefits:**
- Richer, more descriptive dimensions and attributes
- Avoids generalized labeling (transaction date, employee)
- Each fact table inherits dimensions from previous pipeline steps
- Simpler ETL (separate sources → separate fact tables vs. complex integration)

**Procurement Pipeline Accumulating Snapshot:**
- One row per purchase order line item
- Updated as product moves from requisition → PO → shipment → receipt → payment
- Multiple milestone dates (requisition, PO, ship, receipt, payment)
- Lag facts (days requisition to PO, days PO to receipt, days receipt to payment)

### Slowly Changing Dimensions in Procurement

Procurement models extensively use **vendor dimensions** which frequently need type 2 SCD tracking for:
- Vendor address changes (ship-from location)
- Vendor performance tier changes
- Preferred vendor status changes
- Contract term modifications

Refer to supporting file: `slowly-changing-dimension-guide.md` for complete SCD techniques (Types 0-7).

---

## Process 3: Order Management

Order management spans order entry, shipment processing, and invoicing.

### Order Transaction Fact Table

**Grain:** One row per order line item

**Dimensions:**
- Order date (role-playing date dimension)
- Requested ship date (role-playing date dimension)
- Product
- Customer (ship-to location)
- Sales representative
- Deal/contract terms
- Order indicator (junk dimension for flags/indicators)

**Degenerate Dimensions:**
- Order number (groups line items, links to operational system)
- Line number (part of primary key)

**Facts:**
- Order quantity
- Extended gross order amount
- Extended discount amount
- Extended net order amount

### Role-Playing Dimensions

<role_playing_pattern>
**Definition:** A single physical dimension table simultaneously appears in multiple roles in the same fact table.

**Implementation:**
1. Build and administer a **single physical date dimension table**
2. Create **views or aliases** for each role with uniquely labeled columns
3. Each view/alias appears as independent dimension to BI tools

**Example:**
```sql
CREATE VIEW order_date
  (order_date_key, order_day_of_week, order_month, ...)
  AS SELECT date_key, day_of_week, month, ... FROM date;

CREATE VIEW req_ship_date
  (req_ship_date_key, req_ship_day_of_week, req_ship_month, ...)
  AS SELECT date_key, day_of_week, month, ... FROM date;
```

**Critical:** Unique column labeling prevents confusion when both dates appear in reports.

**Bus Matrix Documentation:** Indicate multiple roles within a single cell: "Date (order, req ship, actual ship, invoice)"
</role_playing_pattern>

### Customer Dimension Patterns

**Grain:** One row per discrete ship-to location

**Multiple Independent Hierarchies:**
1. **Geographic Hierarchy:** Ship-to Address → City → County → State → Region → Country
2. **Organizational Hierarchy:** Ship-to → Bill-to → Parent Corporation
3. **Sales Organization Hierarchy:** Ship-to → Sales Rep → Sales Team → Region

<customer_dimension_design>
**Flattening Multiple Hierarchies:**
All hierarchies are embedded as attributes in the single customer dimension. Each customer row has well-defined values for physical geography AND organizational affiliation, even though hierarchies roll up differently.

**Ship-to/Bill-to Relationship:**
- **Simple case (many-to-one):** Include bill-to attributes in customer dimension
- **Rare exceptions (few many-to-many):** Create separate rows for each ship-to/bill-to combination
- **Robust many-to-many:** Use separate ship-to and bill-to dimensions linked by fact table
</customer_dimension_design>

### Junk Dimensions for Transaction Indicators

<junk_dimension_pattern>
**Purpose:** Group low-cardinality flags and indicators into a single dimension

**When to Use:**
- Multiple miscellaneous flags/indicators with small value ranges
- Don't want to clutter fact table with textual descriptors
- Don't want to create 15 separate tiny dimensions

**Sizing Calculation:**
- 10 two-value indicators = maximum 1,024 rows (2^10)
- 5 three-value indicators = maximum 243 rows (3^5)
- 5 hundred-value indicators = 100 million rows (100^5) - **TOO LARGE, use separate dimensions**

**ETL Approach:**
- **Low combinations:** Pre-build all possible combinations
- **High theoretical combinations:** Build rows on-the-fly as encountered in data

**Business Labeling:** Call it "Transaction Profile Dimension" or "Order Indicator Dimension" (not "junk")
</junk_dimension_pattern>

### Header/Line Item Patterns

<header_line_best_practices>
**Recommended Pattern:**
1. Identify fact table grain at **most atomic level** (line item, not header)
2. **Allocate** header-level facts down to line items whenever possible
3. **Distribute** header descriptive attributes into analytic dimensions (customer, date, deal)
4. Keep order number as **degenerate dimension** in fact table

**Facts at Different Granularity:**
- **Preferred:** Allocate shipping charges, taxes to line items (enables slicing by all dimensions including product)
- **Allocation basis:** Gross order amount, weight, volume (get business consensus via data governance)
- **Alternative:** Separate header-grain fact table (loses ability to analyze by product)

**Patterns to AVOID:**
1. **Order header as dimension** - Creates massive dimension, poor performance
2. **Header facts on first/last line only** - Causes incomplete results when lines filtered
3. **Special product key for header facts** - Requires "decoder ring" logic, poor legibility
</header_line_best_practices>

### Invoice Transaction Fact Table with Profitability

**Grain:** One row per invoice line item
**When to Implement:** Often the first DW/BI project - combines customers, products, revenues, and costs

<invoice_profitability_model>
**P&L Fact Structure (top to bottom):**

**Revenue Facts:**
- Extended gross amount (quantity × list price)
- Extended allowance amount (off-invoice allowances)
- Extended discount amount (volume/payment term discounts)
- **Extended net amount** (gross - allowances - discounts)

**Cost Facts (Activity-Based):**
- Extended fixed manufacturing cost
- Extended variable manufacturing cost
- Extended storage cost
- Extended distribution cost

**Bottom Line:**
- **Contribution amount** (net revenue - all costs)

**Critical Success Factor:** All facts must be **extended amounts** (unit rates × quantity) to ensure additivity. Rarely useful to query unit price from single row; when averaging, sum extended prices then divide by sum of quantities.
</invoice_profitability_model>

**Dimensions:**
- Invoice date (role-playing)
- Requested ship date (role-playing)
- Actual ship date (role-playing)
- Product
- Customer (ship-to)
- Deal/contract
- Warehouse (ship-from)
- Shipper (carrier and method)
- Service level (junk dimension or separate)

**Service Level Patterns:**
Multiple representations of service performance:

1. **Multiple Event Dates:** Order date, requested ship date, promised ship date, actual ship date
2. **Quantitative Metrics:** On-time counter (0/1), days early/late (lag calculation)
3. **Qualitative Descriptors:** "On Time", "1-5 Days Late", "6+ Days Late", "Early"

### Multiple Currencies

<multi_currency_pattern>
**Requirement:** Track orders in local transaction currency AND standard corporate currency

**Implementation:**
- Dual fact columns for each metric (local + standard)
- Currency dimension identifies local currency type
- Conversion performed during ETL using business-defined rate (moment of transaction, end of day, end of month)

**Example Facts:**
- Quantity shipped
- Extended gross amount (local currency)
- Extended gross amount (USD)
- Extended net amount (local currency)
- Extended net amount (USD)
- Local currency code (FK to currency dimension)

**Advanced Requirements:**
- Add regional currency facts for regional rollups
- Separate currency conversion fact table with daily exchange rates for ad-hoc conversions
- Multiple rate types (spot, end of month, end of quarter close)
</multi_currency_pattern>

### Multiple Units of Measure

<multi_uom_pattern>
**Business Requirement:** Manufacturing wants pallets/cases, Sales wants retail units/consumer units

**Physical Design:**
- Store base facts in one unit of measure (e.g., shipping cases)
- Store conversion factors as facts (not in product dimension)
- 10 quantity facts × 5 units of measure = store 10 base + 4 conversion factors (14 columns)

**Logical Presentation:**
- Create view(s) that multiply base facts by conversion factors
- Present all 50 calculated facts (10 × 5) with unique labels
- Users see "Order Quantity (Pallets)", "Order Quantity (Consumer Units)" etc.
- No risk of incorrect manual calculations
- Handles time-varying conversion factors correctly

**Why Store Factors in Fact Table:**
- Guarantees correct usage (intra-row calculation is fast and reliable)
- Captures conversion factor values at moment of transaction
- Reduces pressure on product dimension for type 2 changes
- Factors behave more like facts than dimension attributes when they evolve over time
</multi_uom_pattern>

### Audit Dimension for Metadata Context

<audit_dimension_pattern>
**Purpose:** Expose ETL back-room metadata to business users for data quality transparency

**Sample Attributes:**
- Data quality indicator (High/Medium/Low confidence)
- Out of bounds indicator (Yes/No - were any measures anomalous?)
- Amount adjusted flag (Yes/No - were any amounts modified during ETL?)
- Cost allocation version (which logic version calculated costs?)
- Currency conversion version (which rate table was used?)

**Business Value:**
Enables queries like "Show me today's revenue report and flag any rows with out-of-bounds measures" or "What percentage of our profit calculations used estimated vs. actual costs?"

**ETL Source:**
- Error event table (data quality indicators)
- ETL status tables (version numbers, environmental variables)

**Instrumented Reporting:**
Add audit dimension attributes to standard reports to expose data confidence and processing context.
</audit_dimension_pattern>

---

## Process 4: Order Fulfillment Pipeline (Accumulating Snapshot)

### When to Use Accumulating Snapshots

<accumulating_snapshot_criteria>
**Required Characteristics:**
- Process has definite **beginning** and **end**
- Standard set of **intermediate milestones** are well-defined
- Business wants to analyze **workflow** or **pipeline** performance
- Need to measure **velocity** and identify **bottlenecks**

**Ideal For:**
- Order fulfillment (order → manufacture → ship → invoice)
- Manufacturing processes with defined stages
- Product with serial numbers or lot numbers
- Lab specimens moving through testing pipeline

**NOT Suitable For:**
- Long-lived processes that never end (bank accounts → use periodic snapshot)
- Continuous flow processes without discrete milestones
</accumulating_snapshot_criteria>

### Order Fulfillment Accumulating Snapshot Design

**Grain:** One row per order line item

**Multiple Milestone Date Dimensions:**
- Order date
- Manufacturing release date
- Manufacturing completed date
- Finished goods date
- Ship date
- Invoice date
- Payment received date

**Implementation:** Each date is a separate foreign key to role-playing date dimensions. Unknown dates initially reference "To Be Determined" dimension row.

**Dimensions (inherited from all pipeline stages):**
- Product
- Customer
- Sales rep
- Deal
- Warehouse
- Shipper

**Facts:**
- Quantity ordered
- Quantity manufactured
- Quantity shipped
- Quantity invoiced
- Order gross amount
- Invoice net amount
- Total costs
- Contribution amount

**Lag Facts (Velocity Metrics):**
- Days order to manufacturing release
- Days manufacturing release to completion
- Days manufacturing to finished goods
- Days finished goods to shipment
- Days order to shipment (total order cycle time)
- Days shipment to payment

<lag_calculation_intelligence>
**Workday Lag Calculations:**
Rather than simple date differences, ETL should calculate lags that account for:
- Weekends
- Holidays
- Plant shutdowns
- Business operating hours

For short-lived, closely monitored processes, use timestamp-based lags measured in hours or minutes.
</lag_calculation_intelligence>

### Updating Accumulating Snapshot Rows

<accumulating_update_pattern>
**Fundamental Difference from Other Fact Tables:**
- **Transactions:** Load once, never update
- **Periodic Snapshots:** Load layered rows, preserve all snapshots
- **Accumulating Snapshots:** Load once, update repeatedly as milestones occur

**Update Workflow:**
1. Order received → Insert row with order date, unknown dates for all future milestones
2. Manufacturing released → Update row to add release date, update lag facts
3. Manufacturing completed → Update row to add completion date, update lag facts
4. Product shipped → Update row to add ship date, final quantity, update lag facts
5. Customer invoiced → Update row to add invoice date, invoice amounts, update contribution
6. Payment received → Update row to add payment date, close the pipeline row

**Type 2 Dimension Handling:**
For active pipelines, update to reference most current surrogate dimension keys. Once pipeline complete, typically don't revisit for future type 2 changes.
</accumulating_update_pattern>

### Complementary Fact Table Types

<complementary_perspectives>
**Complete Pipeline Analysis Requires Multiple Fact Table Types:**

**Transaction Fact Tables:**
- Show amount of product **flowing through** each pipeline spigot
- One row per event (order, shipment, invoice)
- Best for detailed event analysis

**Periodic Snapshot Fact Tables:**
- Show amount of product **sitting in** the pipeline (backorders, finished goods inventory)
- Dense, predictable time series
- Best for trend analysis and cumulative views

**Accumulating Snapshot Fact Tables:**
- Show **current state** and **velocity** of each order through pipeline
- Updated as milestones occur
- Best for workflow analysis and bottleneck identification

**Integration Pattern:**
Incrementally build monthly periodic snapshot by adding daily accumulating snapshot deltas, then archive as new monthly snapshot on last day of month.
</complementary_perspectives>

---

## Common Design Mistakes to Avoid

<anti_patterns>
1. **Relying on transactions alone** without periodic or accumulating snapshots
2. **Normalizing the fact table** with measurement type dimension (except for sparse metrics or OLAP-primary deployment)
3. **Treating order header as monolithic dimension** instead of distributing attributes to analytic dimensions
4. **Not allocating header facts** to line items (creates unusable separate header fact table)
5. **Mixing fact granularities** in a single fact table (header + line facts together)
6. **Storing conversion factors in dimensions** instead of fact table (currency, units of measure)
7. **Using timed extracts** for change data capture (unreliable, can miss changes)
8. **Ignoring semi-additive nature** of balance facts (inventory, account balances)
9. **Creating separate dimensions for hierarchy levels** instead of flattening hierarchies
10. **Not using surrogate keys** for type 2 slowly changing dimensions
</anti_patterns>

---

## Integration Across the Supply Chain Value Chain

<value_chain_integration>
**Enterprise Data Warehouse Bus Architecture:**

The supply chain processes share conformed dimensions enabling drill-across analysis:

**Shared Conformed Dimensions:**
- **Product:** Links inventory, procurement, orders, invoices
- **Date:** Connects all time-series analysis across processes
- **Vendor:** Joins procurement with inventory receipts
- **Customer:** Connects orders with invoices and shipments
- **Warehouse/Location:** Integrates inventory with shipments

**Drill-Across Query Example:**
"Show me product velocity by comparing:
- Procurement receipt quantity (from procurement fact)
- Warehouse inventory levels (from inventory periodic snapshot)
- Order quantity (from order fact)
- Shipped quantity (from invoice fact)
...all grouped by product and month"

This query joins separate fact tables using conformed product and date dimensions.
</value_chain_integration>

---

## Implementation Checklist

<implementation_checklist>
**Inventory Models:**
- [ ] Determine snapshot frequency and time-based rollup strategy
- [ ] Identify inventory transaction types requiring tracking
- [ ] Assess feasibility of lot/serial number tracking for accumulating snapshot
- [ ] Establish semi-additive fact handling approach (relational vs. OLAP)
- [ ] Define inventory velocity and days supply calculation rules

**Procurement Models:**
- [ ] Map procurement value chain and identify separate processes
- [ ] Draft bus matrix with grain and dimensionality for each process
- [ ] Determine single vs. multiple fact table approach
- [ ] Define SCD strategies for vendor dimension attributes
- [ ] Establish vendor performance metrics and calculations

**Order Management Models:**
- [ ] Identify all order-related transaction types
- [ ] Define role-playing date requirements (order, request, promise, actual)
- [ ] Design customer dimension with multiple hierarchies
- [ ] Determine header fact allocation rules (get business consensus)
- [ ] Specify junk dimension contents for order indicators
- [ ] Define service level performance metrics (quantitative + qualitative)

**Invoice/Profitability Models:**
- [ ] Assess availability of activity-based cost data at line item level
- [ ] Establish P&L fact structure and calculation rules
- [ ] Define multi-currency requirements and conversion rate business rules
- [ ] Identify multiple units of measure requirements and conversion factors
- [ ] Design audit dimension for data quality transparency
- [ ] Determine profitability allocation schemes (freight, overhead, etc.)

**Pipeline/Accumulating Snapshots:**
- [ ] Map complete order fulfillment pipeline with all milestones
- [ ] Define lag calculation intelligence (workdays, holidays, hours vs. days)
- [ ] Establish accumulating snapshot update trigger points
- [ ] Design complementary transaction and periodic snapshot fact tables
- [ ] Determine type 2 dimension update policy for active vs. completed pipelines
</implementation_checklist>

---

## Supporting Reference Files

- `inventory-fact-table-types.md` - Deep dive on transaction, periodic snapshot, and accumulating snapshot patterns for inventory
- `slowly-changing-dimension-guide.md` - Complete reference for SCD types 0-7 techniques
- `multi-currency-implementation.md` - Detailed patterns for handling multiple currencies and exchange rates
- `profitability-model-design.md` - Complete P&L fact table design with activity-based costing integration

---

## Key Takeaways

1. **Three fact table types** (transaction, periodic snapshot, accumulating snapshot) provide complementary views of supply chain processes
2. **Semi-additive facts** (inventory balances) require special aggregation rules - cannot sum across time
3. **Allocate header facts** to line items whenever possible to enable full dimensional analysis
4. **Role-playing dimensions** enable multiple date/time perspectives in single fact table
5. **Multiple currencies and units of measure** stored as fact pairs + conversion factors, exposed via views
6. **Junk dimensions** efficiently handle miscellaneous flags without cluttering fact table
7. **Accumulating snapshots** track pipeline velocity with multiple milestone dates and lag calculations
8. **Profitability models** require activity-based costing and careful allocation consensus
9. **Conformed dimensions** across supply chain processes enable powerful drill-across integration
10. **Header/line relationship** handled by distributing header attributes to dimensions, using degenerate dimensions for order number

---

**Related Skills:**
- `four-step-dimensional-modeling` - Foundation process for all dimensional designs
- `fact-table-design-patterns` - Complete reference for all fact table types
- `conformed-dimension-management` - Integration across business processes
- `etl-system-architecture-design` - ETL subsystems for inventory and procurement data
