# Accumulating Snapshot Fact Table Patterns - Detailed Guide

## Overview

Accumulating snapshot fact tables track finite processes with a definite beginning, definite end, and identifiable milestones in between. Unlike other fact tables, accumulating snapshots are repeatedly updated as a process progresses. This document provides comprehensive guidance on when and how to design accumulating snapshots.

## Fundamental Characteristics

### Defining Traits
- **Grain**: One row per distinct item/order/workflow instance through entire lifecycle
- **Time representation**: Multiple date dimensions for each milestone
- **Loading pattern**: Insert initial row, UPDATE as process progresses
- **Density**: One row per pipeline item (active and completed)
- **Size**: Based on pipeline volume (orders, cases, shipments, etc.)
- **Dimensionality**: Multiple date roles plus standard dimensions

### Why Accumulating Snapshots Matter
- Enable pipeline/workflow analysis from end-to-end
- Track updated status of in-flight processes
- Calculate time lags between milestones
- Identify bottlenecks and inefficiencies
- Measure process velocity and throughput
- Complement transaction and periodic snapshot perspectives

## When to Use Accumulating Snapshots

### Strong Indicators for Accumulating Snapshots

**Definite Workflow Pipeline**:
```
Process characteristics:
✓ Clear start event (order received, case opened, patient admitted)
✓ Clear end event (order shipped, case closed, patient discharged)
✓ Standard intermediate milestones (3-7 typical)
✓ Finite lifecycle (hours to months, not years)

Examples:
- Order fulfillment (order → manufacture → ship → invoice → payment)
- Claims processing (claim filed → reviewed → approved → paid)
- Manufacturing batch (ingredients → mixing → baking → packaging → shipping)
- Patient episode (admission → diagnosis → treatment → discharge)
- Loan application (application → review → approval → funding)
```

**Process Velocity Questions**:
```
Business users ask:
- "How long does it take from order to shipment?"
- "What's the average time in each process stage?"
- "Where are bottlenecks in our pipeline?"
- "How many orders are stuck in manufacturing?"
- "What's the status of order #12345?"

These questions demand accumulating snapshot
```

**Uniquely Identified Items**:
```
Best when tracking:
✓ Individual serial numbers (vehicles, equipment)
✓ Lot/batch numbers (manufacturing, pharmaceuticals)
✓ Order numbers (discrete orders)
✓ Case numbers (customer service, legal)
✓ Patient IDs/visit IDs (healthcare)

Enables tracking individual item through entire pipeline
```

### When NOT to Use Accumulating Snapshots

```
❌ Long-lived processes without clear end
   → Example: Bank accounts, student enrollment (use periodic snapshots)

❌ No standard milestones
   → Example: Ad-hoc processes with varying steps (use transactions)

❌ Simple two-event process
   → Example: Order and shipment only (may not justify complexity)

❌ Need detailed transaction history
   → Accumulating snapshot shows latest state, not all events (use transaction table)

❌ OLAP cube deployment without update support
   → Updates force extensive cube reprocessing (problematic for some tools)
```

## Detailed Design Process

### 1. Identifying Process Milestones

Milestones are the key events tracked in the accumulating snapshot.

**Milestone Selection Criteria**:
```
Include milestones that are:
✓ Standard steps that most items pass through
✓ Business-critical measurement points
✓ Meaningful for velocity/lag analysis
✓ Captured in source systems with dates/timestamps

Typical milestone count: 3-7
- Too few: Doesn't capture enough pipeline detail
- Too many: Complexity outweighs benefit
```

**Example: Order Fulfillment Pipeline**:
```
Milestones:
1. Order received date
2. Order released to manufacturing date
3. Manufacturing complete date
4. Product shipped date
5. Invoice sent date
6. Payment received date

Each milestone becomes a date dimension foreign key
```

**Example: Claims Processing Pipeline**:
```
Milestones:
1. Claim received date
2. Initial review complete date
3. Adjudication complete date
4. Approval/denial date
5. Payment processed date
```

**Example: Manufacturing Batch**:
```
Milestones:
1. Batch started date
2. Mixing complete date
3. Cooking complete date
4. Quality inspection date
5. Packaging complete date
6. Batch shipped date
```

### 2. Declaring Accumulating Snapshot Grain

**Grain Declaration Format**:
"One row per [item/order/case/batch], updated as it progresses through [process name]"

**Examples**:
```
Order Fulfillment:
"One row per order line item, updated as order moves through fulfillment pipeline"

Claims Processing:
"One row per insurance claim, updated as claim moves through adjudication process"

Manufacturing:
"One row per product lot, updated as lot progresses through manufacturing stages"

Patient Care:
"One row per patient episode, updated from admission through discharge"
```

**Grain Considerations**:
```
Question: Header or line item grain?

Order fulfillment example:
Option A: One row per order (header grain)
- Can track order-level milestones
- Loses product detail

Option B: One row per order line item (line grain)
- Tracks product-level movement
- Multiple rows per order
- Preferred if products move independently

Choose grain matching how business tracks pipeline items
```

### 3. Designing Multiple Date Dimensions

Multiple dates are the signature of accumulating snapshots.

**Date Dimension Roles**:
```
Create role-playing date dimension for each milestone:
- order_date (when order received)
- manufacturing_release_date (when released to floor)
- production_complete_date (when manufacturing finished)
- ship_date (when shipped to customer)
- invoice_date (when invoice sent)

Implementation:
- Single physical date dimension table
- Multiple foreign keys in fact table
- Views/aliases for each role with unique column names
```

**Handling Unknown Dates**:
```
Challenge: Many dates unknown when row initially loaded

Solution: Surrogate key for special date dimension row
- Date key: -1 or 99991231 (far future date)
- Date description: "Unknown", "To Be Determined", "Not Yet Occurred"
- Substitute this key for unknown milestone dates

Example initial row (when order received):
- order_date_key: 20240315 (known)
- manufacturing_release_date_key: -1 (unknown)
- production_complete_date_key: -1 (unknown)
- ship_date_key: -1 (unknown)
- invoice_date_key: -1 (unknown)
```

**Date Column Naming**:
```
Make role clear in column names:

Clear:
- order_received_date_key
- mfg_release_date_key
- production_complete_date_key
- shipment_date_key

Unclear:
- date_key_1
- date_key_2
- date_key_3
```

### 4. Identifying Standard Dimensions

Beyond multiple dates, include standard analytic dimensions.

**Standard Dimension Categories**:

**Subject Dimensions**:
```
What's moving through pipeline:
- Product (for order fulfillment, manufacturing)
- Claim type (for claims processing)
- Patient (for healthcare episodes)
- Case type (for customer service)
```

**Actor Dimensions**:
```
Who is involved:
- Customer (for orders)
- Policyholder (for claims)
- Sales rep (for orders)
- Adjuster (for claims)
```

**Location Dimensions**:
```
Where process occurs:
- Warehouse (for order fulfillment)
- Manufacturing plant (for production)
- Service center (for claims)
- Hospital/clinic (for patient care)
```

**Other Context Dimensions**:
```
Additional context:
- Deal/contract (for orders)
- Policy (for claims)
- Service level agreement (for orders)
```

**Dynamic Dimensions**:
```
Status dimension:
- Current pipeline status
- Updated as process progresses
- Example values: In Manufacturing, Awaiting Parts, Shipped, Complete

Consider: Do you need status dimension or can you derive status from dates?
```

### 5. Selecting Accumulating Snapshot Facts

Facts in accumulating snapshots fall into several categories.

**Milestone Quantity Facts**:
```
Quantities at each major milestone:
- quantity_ordered
- quantity_released_to_manufacturing
- quantity_produced
- quantity_shipped
- quantity_invoiced

Purpose:
- Track yield (quantity changes through pipeline)
- Identify shrinkage/waste
- Reconcile input to output
```

**Milestone Amount Facts**:
```
Dollar amounts at milestones:
- extended_order_amount
- extended_cost_amount
- extended_invoice_amount
- extended_payment_amount

Purpose:
- Financial tracking
- Profitability analysis (especially with costs at different stages)
```

**Lag/Duration Facts**:
```
Time elapsed between milestones:
- order_to_manufacturing_lag_days
- manufacturing_to_ship_lag_days
- order_to_invoice_lag_days
- invoice_to_payment_lag_days

Calculation:
- Raw difference: ship_date - order_date
- Workday difference: Business days between dates (excludes weekends/holidays)
- Hours/minutes: For short-lived processes

Best practice: Calculate in ETL, store in fact table
- Don't make users calculate date differences
- Enables proper averaging
```

**Milestone Completion Flags**:
```
0/1 indicators for milestone completion:
- manufacturing_complete_flag (0 = not complete, 1 = complete)
- shipped_flag
- invoiced_flag
- paid_flag

Purpose:
- Count completions
- Filter to incomplete items
- Track completion rates
```

**Status Indicator Facts**:
```
Current state facts:
- is_active_flag (0 = completed, 1 = still in pipeline)
- is_on_time_flag (based on SLA)
- exception_count (number of problems encountered)
```

### 6. Managing Fact Row Updates

Accumulating snapshots uniquely require updates.

**Update Pattern**:
```
Initial load (when order received):
INSERT INTO order_fulfillment_snapshot
- order_date_key = 20240315
- manufacturing_release_date_key = -1 (unknown)
- production_complete_date_key = -1 (unknown)
- ship_date_key = -1 (unknown)
- quantity_ordered = 100
- All lag facts = NULL
- manufacturing_complete_flag = 0

First update (when released to manufacturing):
UPDATE order_fulfillment_snapshot
SET manufacturing_release_date_key = 20240318,
    order_to_manufacturing_lag_days = 3
WHERE order_number = '12345'

Second update (when production complete):
UPDATE order_fulfillment_snapshot
SET production_complete_date_key = 20240322,
    manufacturing_to_ship_lag_days = 4,
    quantity_produced = 98,
    manufacturing_complete_flag = 1
WHERE order_number = '12345'

And so on...
```

**ETL Change Detection**:
```
Identify which rows need updates:
1. Compare operational system to DW snapshot
2. Detect new milestone dates
3. Update corresponding fact row
4. Recalculate affected lag metrics
5. Update completion flags

Frequency:
- Often daily for most pipelines
- More frequent for high-priority or fast-moving processes
```

**Completed Pipeline Items**:
```
Question: Continue updating completed items?

Typical approach:
- Once pipeline complete (all milestones reached), stop updating
- Rare exceptions: Payment received after completion, returns

Benefit:
- Reduces update volume
- Most updates focus on active pipeline items
```

### 7. Handling Type 2 Dimension Changes

**Challenge with Updates and SCDs**:
```
Accumulating snapshot references current dimension keys
When dimension has type 2 change, should snapshot be updated?

Example:
- Order placed March 15 for Product surrogate key 5428
- March 20: Product attribute changes (type 2), new surrogate key 5429
- Order still in pipeline
- Should order fact row update to key 5429?
```

**Recommended Approach**:
```
For ACTIVE pipeline items:
✓ Update to current dimension surrogate key
- Reflects current product attributes
- Users want to see current state

For COMPLETED pipeline items:
✗ Do NOT update dimension keys
- Preserves historical context
- Completed items are stable

Implementation:
UPDATE order_fulfillment_snapshot
SET product_key = <new_current_key>
WHERE is_active_flag = 1
  AND product_key = <old_key>
```

## Advanced Accumulating Snapshot Patterns

### Alternative Milestone Date Handling

**Actual vs. Planned Dates**:
```
Store both planned and actual milestone dates:
- planned_ship_date_key
- actual_ship_date_key
- planned_delivery_date_key
- actual_delivery_date_key

Benefits:
- Compare plan vs. actual
- Calculate variance
- On-time performance metrics

Additional facts:
- ship_date_variance_days = actual_ship_date - planned_ship_date
- on_time_ship_flag = CASE WHEN actual <= planned THEN 1 ELSE 0 END
```

### Timestamp Precision for Short Processes

**Sub-Day Granularity**:
```
For fast-moving processes (hours/minutes):
- Use timestamp dimensions or store timestamps directly
- Calculate lag in hours or minutes

Example: Call center case resolution
- case_opened_timestamp
- first_response_timestamp
- escalation_timestamp
- resolution_timestamp
- case_closed_timestamp

Lag facts:
- time_to_first_response_minutes
- time_to_resolution_hours
```

### Snapshot with Complementary Transactions

**Combined Perspective**:
```
Accumulating snapshot: Overall pipeline status
Transaction fact table: Detailed event history

Example: Order fulfillment
- Accumulating snapshot: Current order status, end-to-end lags
- Transaction fact table: Every inventory transaction affecting order

Benefits:
- Snapshot: Fast pipeline analysis
- Transactions: Detailed event drill-down
- Complement each other
```

### Handling Pipeline Exceptions

**Non-Standard Paths**:
```
Challenge: Some items skip milestones or follow alternate paths

Options:
1. NULL or special date key for skipped milestones
   - Example: Order ships direct (skips manufacturing)
   - manufacturing_release_date_key = -2 ("Not Applicable")

2. Separate fact tables for exceptional paths
   - Standard path: Full accumulating snapshot
   - Exception path: Simplified snapshot
   - Decision: How common are exceptions?

3. Status dimension to indicate path taken
   - Attribute: fulfillment_path ("Standard", "Direct Ship", "Drop Ship")
```

## Real-World Examples

### Order Fulfillment Pipeline
```
Source: Chapter 6 - Order Management
Grain: One row per order line item, updated through fulfillment lifecycle

Dimensions (dates):
- order_date
- manufacturing_release_date
- production_complete_date
- ship_date
- invoice_date
- payment_received_date

Dimensions (other):
- product, customer, warehouse, sales_rep, deal
- order_number (degenerate dimension)

Facts:
- quantity_ordered, quantity_produced, quantity_shipped
- extended_order_amount, extended_invoice_amount
- order_to_ship_lag_days, ship_to_payment_lag_days
- shipped_flag, invoiced_flag, paid_flag

Update frequency: Daily
Typical lifecycle: Days to weeks
```

### Insurance Claims Adjudication
```
Grain: One row per claim, updated through adjudication process

Dimensions (dates):
- claim_received_date
- initial_review_complete_date
- investigation_complete_date
- adjudication_complete_date
- payment_date

Dimensions (other):
- policy, policyholder, claim_type, adjuster, status

Facts:
- claim_amount_requested
- claim_amount_approved
- claim_amount_paid
- received_to_approval_lag_days
- approval_to_payment_lag_days
- claim_approved_flag, claim_paid_flag

Business value:
- Claims processing efficiency
- Adjuster performance
- Bottleneck identification
```

### Manufacturing Batch Production
```
Grain: One row per production batch/lot, updated through manufacturing

Dimensions (dates):
- batch_start_date
- mixing_complete_date
- cooking_complete_date
- quality_inspection_date
- packaging_complete_date
- batch_ship_date

Dimensions (other):
- product, plant, production_line, shift, lot_number (DD)

Facts:
- batch_quantity_started
- batch_quantity_completed
- batch_quantity_passed_inspection
- batch_quantity_shipped
- mixing_duration_hours
- cooking_duration_hours
- total_cycle_time_hours
- quality_passed_flag

Business value:
- Manufacturing throughput
- Yield analysis (started vs. shipped)
- Cycle time optimization
- Quality pass rates
```

### Patient Care Episode
```
Grain: One row per patient episode, updated from admission to discharge

Dimensions (dates):
- admission_date
- diagnosis_date
- treatment_start_date
- treatment_complete_date
- discharge_date

Dimensions (other):
- patient, attending_physician, diagnosis, hospital, department
- episode_number (DD)

Facts:
- admission_to_diagnosis_lag_hours
- diagnosis_to_treatment_lag_hours
- treatment_duration_days
- total_length_of_stay_days
- estimated_cost_amount
- actual_cost_amount
- readmitted_30day_flag

Business value:
- Episode length of stay analysis
- Care delivery efficiency
- Cost management
- Readmission tracking
```

## Accumulating Snapshot Challenges

### OLAP Cube Deployment

**Challenge**:
```
Problem: Accumulating snapshots update both facts AND dimension foreign keys
- Dimension key changes in fact row
- OLAP cube aggregations depend on dimension relationships
- Updates may require extensive cube reprocessing

Impact:
- Daily updates could require full cube rebuild
- Impractical for large accumulating snapshots
```

**Solutions**:
```
Option 1: Load snapshot only when pipeline complete
- No updates after initial load
- Loses "current status" capability
- Acceptable if only analyzing completed items

Option 2: Incremental cube processing
- Some OLAP tools support incremental updates
- Partition cube by completion status
- Only reprocess active partition

Option 3: Use relational star schema, not OLAP
- Relational handles updates naturally
- May sacrifice some query performance
```

### Long-Running Pipelines

**Challenge**:
```
Pipeline items active for months/years:
- Accumulating snapshot stays in "updating" mode
- Fact table size grows but most rows incomplete
- Example: Construction projects, R&D initiatives, PhD programs
```

**Consideration**:
```
Question: Is accumulating snapshot appropriate?

Long pipelines may be better served by:
- Periodic snapshots (monthly project status)
- Transaction fact tables (project events)
- Milestone fact tables (separate row per milestone reached)

Accumulating snapshots work best for:
- Finite lifecycles (hours to months, not years)
- Standard milestone sequences
- High volume of similar items
```

### Source System Constraints

**Challenge**:
```
Accumulating snapshot requires:
✓ Ability to uniquely identify pipeline item
✓ Access to milestone dates as they occur
✓ Ability to detect which items updated since last ETL

If source system doesn't track milestone dates:
- Cannot populate accumulating snapshot accurately
- May need to infer from transactions (complex)
- Consider whether accumulating snapshot is feasible
```

## Performance Optimization

### Partitioning Strategy
```
Partition by completion status or completion date:
- Active partition: Items still in pipeline (frequently updated)
- Completed partitions: Finished items (rarely/never updated)

Benefits:
- Updates isolated to active partition
- Query pruning based on completion
- Archive old completed partitions
```

### Indexing
```
Primary key: Typically the item identifier
- Order number + line number
- Claim number
- Batch/lot number

Secondary indexes:
- Bitmap indexes on completion flags
- Indexes on frequently filtered dimensions
- Avoid indexing frequently updated columns
```

### Update Optimization
```
Minimize update scope:
- Only update changed columns
- WHERE clause on item identifier (indexed)
- Batch updates when possible

Example:
UPDATE order_fulfillment_snapshot
SET ship_date_key = ?,
    order_to_ship_lag_days = ?,
    shipped_flag = 1
WHERE order_number = ?
  AND line_number = ?
```

## Validation Checklist

Before finalizing accumulating snapshot:

**Process Fit**:
- [ ] Process has definite beginning and end
- [ ] Standard milestone sequence identified (3-7 typical)
- [ ] Finite lifecycle duration (hours to months)
- [ ] Items uniquely identified in source systems

**Grain**:
- [ ] Grain clearly stated (one row per item/order/case)
- [ ] Item grain appropriate for tracking
- [ ] Update pattern understood by team

**Milestone Dates**:
- [ ] All milestone dates identified
- [ ] Role-playing date dimensions designed
- [ ] Unknown date handling specified
- [ ] Source system provides milestone dates

**Dimensions**:
- [ ] Standard dimensions identified
- [ ] Type 2 SCD update strategy defined
- [ ] Status dimension considered

**Facts**:
- [ ] Milestone quantities/amounts identified
- [ ] Lag calculations specified (workdays vs. calendar days)
- [ ] Completion flags designed
- [ ] Yield metrics (if applicable) defined

**ETL**:
- [ ] Initial load process designed
- [ ] Update detection mechanism defined
- [ ] Update frequency determined
- [ ] Completed item handling specified

**Deployment**:
- [ ] OLAP compatibility considered
- [ ] Partitioning strategy defined
- [ ] Performance testing planned
- [ ] User training on update behavior planned

**Complementary Tables**:
- [ ] Transaction table for event detail considered
- [ ] Periodic snapshot for trend analysis considered
- [ ] Relationship between fact tables documented
