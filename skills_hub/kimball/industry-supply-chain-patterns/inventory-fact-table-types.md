# Inventory Fact Table Types - Deep Dive

This reference provides comprehensive guidance on the three complementary inventory fact table types: periodic snapshots, transactions, and accumulating snapshots.

## Overview

Inventory analysis requires multiple perspectives:
- **Cumulative view** - How much inventory do we have? (Periodic Snapshot)
- **Event detail** - What transactions are impacting inventory? (Transaction)
- **Pipeline tracking** - How fast is product moving through the warehouse? (Accumulating Snapshot)

No single fact table type can answer all these questions effectively. Most inventory implementations require **at least two** of these fact table types.

---

## Fact Table Type 1: Periodic Snapshot

### Business Questions Answered

- What are current inventory levels by product and location?
- What is the average inventory on hand over the past quarter?
- How many inventory turns did we achieve this year?
- What is the days supply for each product category?
- What is the total inventory value at cost? At selling price?
- How has inventory changed week-over-week?

### Grain Declaration

**Daily Snapshot:** "One row per product per store per day"
**Weekly Snapshot:** "One row per product per warehouse per week"
**Monthly Snapshot:** "One row per product per distribution center per month"

The grain statement must specify the **time period** because you cannot express it as a transaction event.

### Dimension Selection

**Required Dimensions:**
- **Date** - Snapshot date (not transaction date)
- **Product** - With inventory-specific attributes
- **Location** - Store, warehouse, or distribution center

**Optional Dimensions:**
- Inventory status (available, reserved, damaged, in-transit)
- Storage zone (refrigerated, frozen, ambient)

**NOT Applicable:**
- Promotion (associated with sales events, not inventory levels)
- Customer (inventory is pre-customer assignment)
- Transaction type (this is a snapshot, not a transaction)

### Fact Selection

#### Balance Facts (Semi-Additive)

```
Quantity on hand (units)
  - Additive: Across products, locations, inventory statuses
  - NOT Additive: Across dates
  - Aggregation across dates: AVERAGE or take ending balance

Inventory value at cost
  - Extended amount (quantity × unit cost)
  - Semi-additive (same rules as quantity on hand)

Inventory value at latest selling price
  - Extended amount (quantity × current selling price)
  - Semi-additive (same rules as quantity on hand)
```

#### Flow Facts (Fully Additive)

```
Quantity sold (during snapshot period)
  - Fully additive across all dimensions including dates
  - Rolled up from transaction data to snapshot granularity

Beginning balance quantity
  - Quantity on hand at start of period
  - Semi-additive (not additive across dates)

Ending balance quantity
  - Quantity on hand at end of period (same as quantity on hand)
  - Semi-additive (not additive across dates)

Inventory delta/change quantity
  - Difference between ending and beginning balance
  - Fully additive across all dimensions
```

### Semi-Additive Aggregation Examples

#### Correct Approach for Time-Based Averaging

```sql
-- CORRECT: Average inventory over time
-- Calculate average daily inventory for Product X in Store Y during March
SELECT
    product_key,
    store_key,
    SUM(quantity_on_hand) / COUNT(DISTINCT snapshot_date_key) as avg_daily_inventory
FROM inventory_periodic_snapshot
WHERE product_key = 12345
  AND store_key = 67
  AND snapshot_date BETWEEN '2024-03-01' AND '2024-03-31'
GROUP BY product_key, store_key;
```

#### Incorrect Approach (Using AVG Function)

```sql
-- INCORRECT: SQL AVG divides by total rows, not distinct dates
-- If querying 3 products × 4 stores × 7 dates = 84 rows
-- AVG() divides by 84, but should divide by 7 (number of dates)
SELECT
    product_category,
    region,
    AVG(quantity_on_hand) as avg_inventory  -- WRONG!
FROM inventory_periodic_snapshot
WHERE snapshot_date BETWEEN '2024-03-01' AND '2024-03-07'
GROUP BY product_category, region;
```

### Derived Metrics Calculations

#### Inventory Turns

```sql
-- Number of turns for a single day
turns_per_day = quantity_sold / quantity_on_hand

-- Number of turns for an extended period (e.g., year)
annual_turns = SUM(quantity_sold) / AVG_DAILY(quantity_on_hand)

-- Example:
SELECT
    product_key,
    SUM(quantity_sold) /
      (SUM(quantity_on_hand) / COUNT(DISTINCT snapshot_date_key)) as annual_turns
FROM inventory_periodic_snapshot
WHERE YEAR(snapshot_date) = 2024
GROUP BY product_key;
```

#### Days Supply

```sql
-- Days supply for an extended period
days_supply = final_quantity_on_hand / average_quantity_sold_per_day

-- Example: Days supply at end of quarter
SELECT
    p.product_key,
    p.product_description,
    final.quantity_on_hand /
      (SUM(daily.quantity_sold) / COUNT(DISTINCT daily.snapshot_date_key)) as days_supply
FROM inventory_periodic_snapshot final
JOIN inventory_periodic_snapshot daily
  ON final.product_key = daily.product_key
  AND final.store_key = daily.store_key
WHERE final.snapshot_date = '2024-03-31'  -- End of quarter
  AND daily.snapshot_date BETWEEN '2024-01-01' AND '2024-03-31'
GROUP BY p.product_key, p.product_description, final.quantity_on_hand;
```

### Density Management Strategies

#### Challenge: Dense Fact Tables

For retailer with:
- 60,000 products
- 100 stores
- Daily snapshots
- Result: **6 million rows per day** (60,000 × 100)

Over 3 years: **6.57 billion rows** (6M × 365 × 3)

#### Solution 1: Time-Based Rollup Strategy

**Maintain two separate fact tables with different periodicities:**

```
inventory_daily_snapshot
  - Last 60 days at daily granularity
  - 360 million rows (6M × 60 days)
  - Refreshed nightly, oldest day dropped

inventory_weekly_snapshot
  - Historical data at weekly granularity
  - Week-ending snapshots only
  - 148 weeks in 3 years (52 × 3 - 8 most recent weeks)
  - 888 million rows (6M × 148 weeks)
```

**Total storage:** 1.25 billion rows vs. 6.57 billion rows (81% reduction)

**ETL Process:**
1. Load daily snapshot each night
2. Drop snapshot older than 60 days from daily table
3. If dropped snapshot is week-ending date, insert into weekly table

#### Solution 2: Sparse Storage with Nulls

Store NULL for quantity on hand when product is out of stock (if acceptable to business):
- Reduces row count for products frequently out of stock
- Requires NULL-aware aggregation logic
- May complicate "what products are out of stock" queries

#### Solution 3: Partition Pruning

Use table partitioning by date:
- Daily partitions for recent data
- Weekly partitions for historical data
- Queries automatically prune irrelevant partitions

### Product Dimension Enhancements for Inventory

Add inventory-specific attributes to product dimension:

```
Minimum reorder quantity
Maximum stock level
Reorder point threshold
Lead time days
Storage requirement code (ambient/refrigerated/frozen)
Shelf life days
Storage cube volume
Storage weight
Hazmat indicator
```

These attributes should be **type 0 or type 1** (not type 2) unless business needs historical tracking of policy changes.

### Location Dimension Enhancements for Inventory

Add inventory-specific attributes to store/warehouse dimension:

```
Total storage square footage
Refrigerated storage square footage
Frozen storage square footage
Ambient storage square footage
Number of storage zones
Automated storage indicator
Climate controlled indicator
```

---

## Fact Table Type 2: Transaction

### Business Questions Answered

- How many receipts occurred today? By vendor?
- What is the frequency of product returns by product category?
- How many inspection failures occurred this month?
- What time of day do most shipments leave the warehouse?
- Which transaction types are most common?
- What is the error rate for picking operations?

### Grain Declaration

"One row per inventory transaction"

Each row represents a single discrete event that impacts inventory levels.

### Dimension Selection

**Required Dimensions:**
- **Date** (transaction date/time)
- **Product**
- **Location** (warehouse, store, distribution center)
- **Transaction Type** - Dimension with one row per transaction type

**Optional Dimensions:**
- Vendor (for receipts, returns to vendor)
- Customer (for shipments, customer returns)
- Employee (receiving clerk, picker, packer, shipper)
- Carrier/shipper (for inbound and outbound shipments)
- Reason code (for returns, adjustments, write-offs)

### Transaction Type Dimension Design

```
Transaction Type Dimension Attributes:

transaction_type_key (surrogate key)
transaction_type_code (operational code)
transaction_type_description
transaction_category (Receipt / Put-away / Pick / Ship / Adjust / Return)
inventory_impact_direction (Increase / Decrease / Neutral)
requires_inspection_flag
requires_approval_flag
triggers_reorder_flag
transaction_type_sort_order
```

Example rows:
```
| Key | Code | Description | Category | Impact |
|-----|------|-------------|----------|--------|
| 1 | RCV | Receive from Vendor | Receipt | Increase |
| 2 | INS | Place in Inspection Hold | Put-away | Neutral |
| 3 | REL | Release from Inspection | Put-away | Neutral |
| 4 | RTV | Return to Vendor | Return | Decrease |
| 5 | PUT | Place in Bin | Put-away | Neutral |
| 6 | PIK | Pick from Bin | Pick | Neutral |
| 7 | PKG | Package for Shipment | Pick | Neutral |
| 8 | SHP | Ship to Customer | Ship | Decrease |
| 9 | RTN | Receive Customer Return | Receipt | Increase |
| 10 | RST | Restock from Return | Put-away | Increase |
| 11 | ADJ | Cycle Count Adjustment | Adjust | Increase/Decrease |
| 12 | DMG | Remove Damaged Product | Adjust | Decrease |
```

### Fact Selection

**Primary Fact:**
```
Transaction quantity
  - Positive for receipts and restocks
  - Negative for shipments and removals
  - Neutral (0) for status changes (inspection hold, bin transfers)
  - Fully additive across all dimensions
```

**Optional Facts:**
```
Transaction extended cost (quantity × unit cost at time of transaction)
Transaction extended value (quantity × valuation basis)
Transaction weight
Transaction cube volume
```

### Handling Varied Dimensionality

**Challenge:** Different transaction types may have different applicable dimensions.

**Example:**
- Receive from Vendor: Has vendor dimension
- Ship to Customer: Has customer and shipper dimensions
- Place in Bin: Has bin/zone dimension
- Cycle Count Adjustment: Has employee dimension

**Solution 1: Single Fact Table with Optional Dimensions**

Include all possible dimensions. Use "Not Applicable" dimension rows when dimension doesn't apply to transaction type.

```
inventory_transaction_fact
  - transaction_date_key
  - product_key
  - warehouse_key
  - vendor_key (NA for non-vendor transactions)
  - customer_key (NA for non-customer transactions)
  - employee_key (NA if not tracked)
  - shipper_key (NA for non-shipment transactions)
  - transaction_type_key
  - transaction_quantity
```

**Solution 2: Separate Fact Tables by Transaction Category**

Create separate fact tables when dimensionality varies significantly:

```
warehouse_receipt_fact (from vendors)
warehouse_shipment_fact (to customers)
warehouse_putaway_fact (internal movements)
warehouse_adjustment_fact (corrections, damage)
```

### Transaction vs. Snapshot Trade-offs

**Transactions are GOOD for:**
- Detailed event analysis
- Frequency and timing analysis
- Root cause investigation
- Operational monitoring
- Audit trails

**Transactions are BAD for:**
- "What is current inventory level?" (must roll forward all transactions)
- "What was inventory last month end?" (must roll forward to specific point)
- Trend analysis over time
- Cumulative performance metrics

**Therefore:** Most inventory implementations need **BOTH** transaction and snapshot fact tables.

---

## Fact Table Type 3: Accumulating Snapshot

### Business Questions Answered

- How long does product sit in inspection on average?
- What is the average time from receipt to shipment?
- Which products move through the warehouse fastest?
- How many products are currently in each pipeline stage?
- What percentage of received products fail inspection?
- What is the average storage time by product category?

### When to Use

**Required Process Characteristics:**
- Definite **beginning** (product receipt)
- Definite **end** (product shipment or removal)
- Well-defined **intermediate milestones** (inspection, binning, picking)
- Ability to **uniquely identify** each pipeline occurrence (lot number, receipt number, serial number)

**Ideal Scenarios:**
- Tracking product lots through warehouse
- Managing serialized inventory (electronics, appliances)
- Monitoring perishable goods (food, pharmaceuticals)
- Analyzing warehouse throughput and efficiency

**NOT Suitable For:**
- Continuously replenished inventory without discrete lots
- Commingled inventory (multiple receipts mixed in bins)
- Very long-lived inventory (years in storage)

### Grain Declaration

"One row per product lot received at warehouse" or
"One row per serialized product unit" or
"One row per warehouse receipt line item"

### Multiple Milestone Date Dimensions

**Standard Warehouse Pipeline Milestones:**

```
received_date_key          (Product arrives at warehouse)
inspection_start_date_key  (Inspection begins)
inspection_end_date_key    (Inspection completes)
binned_date_key           (Product placed in storage)
picked_date_key           (Product picked for order)
packed_date_key           (Product packaged)
shipped_date_key          (Product leaves warehouse)
```

Each milestone is a **role-playing date dimension**.

**Unknown Date Handling:**
- Initial insert: Only received_date_key is known
- All future milestone date keys point to "To Be Determined" special row
- As milestones occur, update fact row with actual date keys

### Dimension Selection

**Dimensions from Receipt Stage:**
- Product
- Warehouse
- Vendor
- Receipt reason

**Dimensions from Shipment Stage:**
- Customer (ship-to)
- Destination warehouse
- Carrier/shipper
- Shipment reason

**Dimensions Throughout Pipeline:**
- Inspection result (dimension: Pass/Fail/Partial)
- Storage zone
- Final disposition (dimension: Shipped/Damaged/Returned to Vendor/Expired)

### Fact Selection

**Quantity Facts:**
```
Received quantity (initial quantity)
Inspection passed quantity
Inspection failed quantity
Returned to vendor quantity
Binned quantity
Picked quantity
Damaged in storage quantity
Shipped quantity
```

**Derived Quantity Facts:**
```
Quantity shrinkage = Received quantity - (Shipped + Damaged + Returned to vendor)
```

**Lag Facts (Days Between Milestones):**
```
Days in receiving (inspection_start_date - received_date)
Days in inspection (inspection_end_date - inspection_start_date)
Days to binning (binned_date - inspection_end_date)
Days in storage (picked_date - binned_date)
Days in picking/packing (shipped_date - picked_date)
Total cycle time (shipped_date - received_date)
```

**Milestone Completion Counters (0 or 1):**
```
Inspection completed flag
Binning completed flag
Picked completed flag
Shipped completed flag
```

### Row Evolution Example

**Day 1 - Product Received:**
```
| Receipt | Received | Inspect | Inspect | Binned | Picked | Shipped | Rcvd | Shipped | Days in | Days in | Status |
| Number  | Date     | Start   | End     | Date   | Date   | Date    | Qty  | Qty     | Inspect | Storage |        |
|---------|----------|---------|---------|--------|--------|---------|------|---------|---------|---------|--------|
| R-12345 | 1/15/24  | TBD     | TBD     | TBD    | TBD    | TBD     | 1000 | 0       | NULL    | NULL    | Open   |
```

**Day 2 - Inspection Started:**
```
| R-12345 | 1/15/24  | 1/16/24 | TBD     | TBD    | TBD    | TBD     | 1000 | 0       | NULL    | NULL    | Open   |
```

**Day 4 - Inspection Completed, Product Binned:**
```
| R-12345 | 1/15/24  | 1/16/24 | 1/18/24 | 1/18/24| TBD    | TBD     | 1000 | 0       | 2       | NULL    | Open   |
```

**Day 10 - Product Picked for Order:**
```
| R-12345 | 1/15/24  | 1/16/24 | 1/18/24 | 1/18/24| 1/24/24| TBD     | 1000 | 0       | 2       | 6       | Open   |
```

**Day 11 - Product Shipped (Pipeline Complete):**
```
| R-12345 | 1/15/24  | 1/16/24 | 1/18/24 | 1/18/24| 1/24/24| 1/25/24 | 1000 | 1000    | 2       | 6       | Closed |
```

### Update Triggers and ETL Process

**When to Update Accumulating Snapshot:**

1. **Product received** → INSERT new row with received_date, TBD for future milestones
2. **Inspection event** → UPDATE row with inspection dates
3. **Binning event** → UPDATE row with binned_date, calculate storage lag
4. **Picking event** → UPDATE row with picked_date, update storage lag
5. **Shipment event** → UPDATE row with shipped_date, final quantities, mark status = Closed

**ETL Logic:**
```
If NOT EXISTS (fact row for receipt_number):
    INSERT new row with current milestone date
    Set all future milestone dates to TBD dimension key
Else:
    UPDATE existing row
    Set milestone date that just occurred
    Recalculate lag facts
    Update status if pipeline complete
```

### Status Dimension for Pipeline State

```
Status Dimension Attributes:

status_key (surrogate key)
status_code
status_description
is_active_flag (pipeline still moving vs. complete)
completion_percentage

Example Rows:
| Key | Code | Description | Active | Completion % |
|-----|------|-------------|--------|--------------|
| 1 | RCV | Received Only | Yes | 10% |
| 2 | INS | In Inspection | Yes | 30% |
| 3 | BIN | In Storage | Yes | 50% |
| 4 | PIK | Picked for Order | Yes | 70% |
| 5 | SHP | Shipped Complete | No | 100% |
| 6 | RTV | Returned to Vendor | No | 100% |
| 7 | DMG | Damaged/Scrapped | No | 100% |
```

### Handling Type 2 Slowly Changing Dimensions

**For Active Pipelines:**
When product dimension has type 2 change (e.g., product category reassignment):
- UPDATE accumulating snapshot to reference new product dimension surrogate key
- Ensures reports show product with current attributes

**For Closed Pipelines:**
- Do NOT update for type 2 changes after pipeline complete
- Preserves historical context at time of shipment

### Accumulating Snapshot Limitations

**Not Good For:**
- Complex exceptional scenarios (multiple inspection cycles, partial shipments)
  - Use transaction fact table for detailed event tracking
- OLAP cubes
  - Updating fact rows forces dimension key changes
  - Requires extensive cube reprocessing
  - Solution: Only load accumulating snapshot to cube after pipeline complete

---

## Choosing the Right Fact Table Type(s)

### Decision Matrix

| Business Question Type | Transaction | Periodic Snapshot | Accumulating Snapshot |
|------------------------|-------------|-------------------|----------------------|
| What is current inventory level? | ✗ Very Hard | ✓ Perfect | ✗ Aggregation Required |
| What was inventory last month? | ✗ Very Hard | ✓ Perfect | ✗ Aggregation Required |
| How many receipts occurred today? | ✓ Perfect | ✗ Not Available | ✗ Not Available |
| Average inventory over quarter? | ✗ Very Hard | ✓ Perfect | ✗ Aggregation Required |
| Time from receipt to shipment? | ✗ Very Hard | ✗ Not Available | ✓ Perfect |
| Frequency of transaction types? | ✓ Perfect | ✗ Not Available | ✗ Not Available |
| Inventory turns calculation? | ○ Possible | ✓ Perfect | ✗ Not Ideal |
| Current pipeline status? | ✗ Very Hard | ✗ Not Available | ✓ Perfect |

### Recommended Combinations

**Minimum Viable Inventory DW:**
- Periodic Snapshot (for levels and trends)
- Transaction (for event detail)

**Complete Inventory DW:**
- Periodic Snapshot (for levels and trends)
- Transaction (for event detail and audit trail)
- Accumulating Snapshot (for pipeline velocity and throughput)

**Quick Win First Implementation:**
- Start with Periodic Snapshot only
- Add Transaction fact table in Phase 2
- Add Accumulating Snapshot in Phase 3 (if lot tracking feasible)

---

## Integration Patterns

### Periodic Snapshot Built from Transactions

Some organizations build periodic snapshot from transaction detail:

**Process:**
1. Start with prior period ending balance
2. Add all transactions (positive and negative) during current period
3. Calculate ending balance = beginning balance + net transactions
4. Insert ending balance into periodic snapshot fact table

**When Feasible:**
- All inventory changes captured as transactions
- No external balance adjustments
- Transaction history complete and reliable

**When NOT Feasible:**
- Operational system provides balances but not complete transaction history
- External cycle count adjustments bypass transaction system
- Simpler to extract balances directly from operational snapshot

### Accumulating Snapshot Feeding Periodic Snapshot

For finite-lived inventory (lot-tracked, serialized):

**Process:**
1. Maintain accumulating snapshot for active lots
2. When lot fully depleted (shipped, damaged, expired), archive to periodic snapshot
3. Periodic snapshot becomes historical repository
4. Accumulating snapshot remains lean (only active lots)

**Benefits:**
- Accumulating snapshot optimized for pipeline analysis (small, fast)
- Periodic snapshot optimized for long-term trend analysis
- Clear separation of active vs. historical inventory

---

## Performance Optimization Considerations

### Periodic Snapshot Optimization

**Partitioning Strategy:**
- Partition by date (monthly or quarterly)
- Sub-partition by product category (if very large)
- Enables partition pruning for date-range queries

**Indexing Strategy:**
- Clustered index on date + product + location
- Non-clustered indexes on frequently queried dimension keys
- Consider columnstore index for large historical snapshots

**Aggregation Tables:**
- Pre-aggregate weekly/monthly rollups from daily snapshots
- Pre-aggregate product category/region combinations
- Significantly faster for executive dashboards

### Transaction Fact Table Optimization

**Partitioning Strategy:**
- Partition by transaction date
- Sliding window: Drop oldest partitions, add new partitions
- Keep only last 13-24 months in main fact table

**Indexing Strategy:**
- Clustered index on transaction date/time + transaction type
- Non-clustered index on product + date
- Consider covering index for most common query patterns

**Archival Strategy:**
- After 2 years, move to archive table or data lake
- Retain summarized transaction counts in aggregation table

### Accumulating Snapshot Optimization

**Active vs. Closed Separation:**
```
inventory_pipeline_active (rows with status = Active)
  - Small table (only current open pipelines)
  - Updated frequently
  - Optimized for write performance

inventory_pipeline_history (rows with status = Closed)
  - Large table (all completed pipelines)
  - Append-only (no updates)
  - Optimized for read performance and compression
```

**Indexing Strategy:**
- Active: Clustered index on receipt_number for fast updates
- History: Clustered index on shipped_date for trend analysis

---

## Common Mistakes to Avoid

1. **Using only transactions** without periodic snapshots for balance tracking
2. **Not accounting for semi-additive nature** of balance facts in queries and reports
3. **Including promotion dimension** in inventory snapshot (promotions associated with sales, not inventory)
4. **Building accumulating snapshot for continuous replenishment** inventory (no discrete lots/serial numbers)
5. **Not using "To Be Determined" date dimension row** for unknown future milestone dates
6. **Trying to build complete balance picture from transactions alone** (too complex and slow)
7. **Loading accumulating snapshots into OLAP cubes** before pipelines are closed (causes excessive reprocessing)
8. **Not separating active vs. closed** accumulating snapshot rows (poor update performance)
9. **Over-complicated transaction type dimensions** (keep it simple with basic categorization)
10. **Forgetting to update accumulating snapshots** to current type 2 dimension keys for active pipelines

---

## Summary

**Three inventory fact table types serve complementary purposes:**

- **Periodic Snapshot:** Cumulative view of inventory levels over time (semi-additive balance facts)
- **Transaction:** Detailed event tracking of every inventory movement (fully additive)
- **Accumulating Snapshot:** Pipeline velocity and throughput analysis (updated rows, multiple milestone dates)

**Most implementations need at least two types** to provide complete inventory visibility. Start with periodic snapshot, add transactions for detail, add accumulating snapshots for pipeline analysis if lot/serial tracking is feasible.

**Integration across types** provides the most powerful inventory analytics: current levels (periodic), movement detail (transaction), and velocity metrics (accumulating).
