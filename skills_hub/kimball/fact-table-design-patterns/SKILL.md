---
name: fact-table-design-patterns
description: Design transaction, periodic snapshot, and accumulating snapshot fact tables for dimensional models. Use when defining fact table grain, selecting fact table type, handling fact additivity, or designing metrics for any business process measurement event.
---

# Fact Table Design Patterns

## Overview

Every dimensional model centers on a fact table representing business process measurements. There are exactly three fundamental fact table types, and choosing the right type is critical to effective dimensional modeling. This skill guides you through selecting and designing the appropriate fact table pattern.

## The Three Fact Table Types

Understanding which fact table type to use is the foundation of dimensional modeling:

1. **Transaction Fact Tables** - Capture individual business events at a point in time
2. **Periodic Snapshot Fact Tables** - Capture cumulative performance at regular intervals
3. **Accumulating Snapshot Fact Tables** - Track evolving status through a defined workflow pipeline

Each type serves a distinct analytic purpose. Many business processes benefit from 2-3 complementary fact tables.

---

## When to Use Each Fact Table Type

### Transaction Fact Tables - Use When:
- Business process generates discrete events (orders, sales, calls, transactions)
- Each event occurs at a specific moment in time
- Need to analyze individual event details with maximum dimensionality
- Events are unpredictable and sparse
- **Grain example**: "One row per line item on a POS transaction"

### Periodic Snapshot Fact Tables - Use When:
- Need cumulative performance view at regular intervals
- Measurements represent balances or levels at a point in time
- Transaction details are too granular for trend analysis
- Want predictable, dense coverage across time periods
- **Grain example**: "One row per product per store per day for inventory levels"

### Accumulating Snapshot Fact Tables - Use When:
- Process has definite beginning, end, and standard milestones
- Need to track workflow or pipeline status as it evolves
- Want to measure time lags between process stages
- Analyzing throughput, velocity, or cycle times
- **Grain example**: "One row per order line, updated as order moves through fulfillment pipeline"

---

## Step 1: Select the Appropriate Fact Table Type

<selection_framework>

Ask these questions to determine fact table type:

**Question 1**: Does the business process generate discrete events or maintain ongoing status?
- Discrete events → Transaction fact table
- Ongoing status → Snapshot fact table (periodic or accumulating)

**Question 2**: If snapshot, does the status represent regular intervals or a workflow pipeline?
- Regular intervals (daily, weekly, monthly) → Periodic snapshot
- Workflow with milestones → Accumulating snapshot

**Question 3**: Can you answer all business questions with a single fact table type?
- If no → Design complementary fact tables
- Example: Inventory needs both transaction (receipts/shipments) AND periodic snapshot (daily balances)

</selection_framework>

---

## Step 2: Design Transaction Fact Tables

Transaction fact tables are the most common pattern.

### Characteristics
- Grain: One row per transaction or transaction line item
- Loading: Insert only, rows never updated after posting
- Density: Sparse and unpredictable
- Size: Often billions of rows
- Dimensionality: Highly dimensional (10-20 dimensions typical)

### Design Steps

1. **Declare atomic grain** - Most granular level captured by source system
   - Example: "One row per product scanned on a POS transaction"
   - NOT: "Daily sales by product" (too aggregated)

2. **Identify dimensions** - All "who, what, where, when, why, how" at this grain
   - Date, product, store, promotion, customer, cashier, payment method
   - Include degenerate dimensions (transaction numbers)

3. **Identify facts** - Measurements at declared grain
   - Quantities, extended amounts (quantity × unit price/cost)
   - Prefer additive facts
   - Store derived facts if used frequently

4. **Validate additivity** - Ensure facts sum correctly across all dimensions

### Transaction Fact Examples
```
POS Retail Sales (Chapter 3):
- Grain: One row per product per POS transaction
- Dimensions: Date, product, store, promotion, cashier, payment method, transaction number (DD)
- Facts: sales_quantity, extended_sales_amount, extended_discount_amount, extended_cost_amount

Order Line Items (Chapter 6):
- Grain: One row per product per order
- Dimensions: Order date, requested ship date, product, customer, sales rep, deal, order number (DD)
- Facts: order_quantity, extended_gross_amount, extended_discount_amount, extended_net_amount
```

**See [transaction-fact-patterns.md](transaction-fact-patterns.md) for detailed guidance.**

---

## Step 3: Design Periodic Snapshot Fact Tables

Periodic snapshots provide regular, predictable views of cumulative performance.

### Characteristics
- Grain: Often expressed as combination of dimensions (one row per product per store per day)
- Loading: Regular snapshots stacked consecutively
- Density: Dense and predictable
- Size: Calculated rows based on dimensions × time periods
- Dimensionality: Usually fewer dimensions than transaction tables

### Design Steps

1. **Declare snapshot grain** - Specify period and dimensional scope
   - Example: "One row per product per warehouse per day"
   - Period examples: Daily, weekly, monthly, quarterly

2. **Identify dimensions** - Natural dimensions at snapshot grain
   - Typically NO promotion or causal dimensions (those apply to movements)
   - Example: Date, product, warehouse (NOT vendor, transaction type)

3. **Select snapshot facts** - Balance metrics plus activity summaries
   - Balances/levels: quantity_on_hand, account_balance
   - Period activity: quantity_sold, quantity_received
   - Valuations: inventory_value_at_cost, inventory_value_at_retail

4. **Handle semi-additive facts** - Special treatment for balance metrics
   - Balances are semi-additive (additive across some dimensions, not date)
   - Average balances across time, don't sum them

### Periodic Snapshot Examples
```
Store Inventory (Chapter 4):
- Grain: One row per product per store per day
- Dimensions: Date, product, store
- Facts: quantity_on_hand (semi-additive), quantity_sold, extended_cost_value, extended_retail_value

Monthly Account Balances (Chapter 7):
- Grain: One row per account per month
- Dimensions: Month, account, organization
- Facts: ending_balance (semi-additive), period_activity, beginning_balance
```

**See [periodic-snapshot-patterns.md](periodic-snapshot-patterns.md) for detailed guidance.**

---

## Step 4: Design Accumulating Snapshot Fact Tables

Accumulating snapshots track finite processes with standard milestones.

### Characteristics
- Grain: One row per distinct item/order/workflow instance
- Loading: Insert initial row, UPDATE as process progresses
- Density: One row per pipeline item
- Size: Based on active and completed pipeline items
- Dimensionality: Multiple date dimensions for each milestone

### Design Steps

1. **Identify process milestones** - Key events in workflow
   - Example order fulfillment: Order received, released to manufacturing, produced, shipped, invoiced
   - Typically 3-7 major milestones

2. **Create date dimension for each milestone** - Use role-playing dates
   - order_date, manufacturing_release_date, production_complete_date, ship_date, invoice_date
   - Use surrogate key for "Unknown" or "To Be Determined" dates

3. **Identify other dimensions** - Standard dimensions at overall process grain
   - Product, customer, warehouse (NOT changing across milestones)
   - May include status dimension that updates

4. **Define milestone facts** - Quantities and amounts at each stage
   - quantity_ordered, quantity_produced, quantity_shipped

5. **Calculate lag metrics** - Elapsed time between milestones
   - order_to_ship_lag_days, manufacturing_lag_days
   - ETL should calculate based on business rules (workdays, not just date difference)

6. **Add milestone counters** - 0/1 flags for completion tracking
   - manufacturing_complete_flag, shipped_flag

### Accumulating Snapshot Example
```
Order Fulfillment Pipeline (Chapter 6):
- Grain: One row per order line item (updated as order progresses)
- Dimensions (dates): Order date, manufacturing release date, production date, ship date, invoice date
- Dimensions (other): Product, customer, warehouse, sales rep, deal, order number (DD)
- Facts: quantity_ordered, quantity_produced, quantity_shipped, order_to_ship_lag_days
- Updated: As each milestone is reached
```

**See [accumulating-snapshot-patterns.md](accumulating-snapshot-patterns.md) for detailed guidance.**

---

## Step 5: Understand Fact Additivity

Fact additivity determines how facts can be summed across dimensions.

### Additive Facts
Facts that can be summed across ALL dimensions.
- **Examples**: sales_quantity, extended_sales_amount, extended_cost
- **Preferred**: Design for additivity whenever possible
- **Rule**: Always store extended amounts (quantity × unit rate), not unit rates

### Semi-Additive Facts
Facts additive across SOME dimensions but not all (typically not time).
- **Examples**: account_balance, inventory_quantity_on_hand, headcount
- **Reason**: Represent snapshots of a level at a point in time
- **Solution**: Average across time dimension, sum across others
- **Common in**: Periodic snapshots

### Non-Additive Facts
Facts that cannot be summed across any dimension.
- **Examples**: unit_price, ratios, percentages, temperatures
- **Storage**: Store numerator and denominator instead
- **Calculation**: Ratio of sums, not sum of ratios
- **Example**: Gross margin = SUM(gross_profit) / SUM(revenue), NOT AVG(gross_margin)

**Warning**: SQL AVG function divides by row count, not time period count. For time-based averages of semi-additive facts, manually calculate: SUM(balance) / COUNT(DISTINCT date)

**See [fact-additivity-guide.md](fact-additivity-guide.md) for complete coverage.**

---

## Step 6: Apply Advanced Fact Table Patterns

### Multiple Currencies
Store facts in multiple currency contexts:
```
Facts per row:
- extended_local_currency_amount + local_currency_key (dimension FK)
- extended_corporate_currency_amount (e.g., USD)
- extended_regional_currency_amount (optional)

Alternative: Currency conversion fact table for ad-hoc conversions
```

### Multiple Units of Measure
Store base facts plus conversion factors in same row:
```
Physical storage:
- quantity_shipped (base unit, e.g., cases)
- case_to_pallet_factor
- case_to_retail_unit_factor
- case_to_eaches_factor

Present via view:
- quantity_shipped_pallets (calculated)
- quantity_shipped_retail_units (calculated)
- quantity_shipped_eaches (calculated)
```

### Header/Line Item Facts
When facts exist at different grains (order header vs. line):
- **Preferred**: Allocate header facts down to line item grain
- **Benefit**: Enables slicing by ALL dimensions including product
- **Alternative**: Separate fact tables if allocation impossible
- **Warning**: Never mix grains in same fact table

### Derived Facts
Facts calculated from other facts in same row:
```
Example: gross_profit = extended_sales_amount - extended_cost_amount

Store if:
- Frequently used by business users
- Want consistent calculation across all BI tools
- Calculation is non-trivial

Cost: Minimal storage increase vs. risk of calculation errors
```

### Null Fact Values
- **Leave as NULL** for proper handling in aggregate functions (SUM, AVG, COUNT)
- Do NOT substitute zero (skews calculations)
- Exception: Data mining tools may require different treatment

---

## Common Fact Table Patterns to Avoid

### ❌ Fact Table Normalization
Don't normalize facts into separate rows with measurement type dimension:
```
Bad Pattern:
- Grain: One row per measurement type per transaction line
- Fact table grows 5× if you have 5 fact types
- SQL computations between facts become extremely difficult
```

### ❌ Centipede Fact Tables
Don't create dozens of dimension foreign keys:
```
Problem: Fact table with 30+ dimensions
- Often indicates correlated dimensions that should be combined
- Normalized hierarchy levels treated as separate dimensions
- Hurts usability and performance

Solution: Combine correlated dimensions into single denormalized dimension
```

### ❌ Header as Dimension
Don't model transaction header as dimension table:
```
Problem: Order header dimension with all header attributes
- Dimension as large as fact table
- Defeats dimensional model benefits
- Difficult to analyze header characteristics

Solution: Decompose header attributes into proper analytic dimensions
```

### ❌ Mixed Granularity
Don't combine facts from different grains in same fact table:
```
Problem: Order line facts mixed with order header facts
- Some facts repeat on every line (overcounting)
- Impossible to analyze header facts by line-level dimensions

Solution: Allocate to common grain OR create separate fact tables
```

---

## Validation Checklist

Before finalizing fact table design:

**Grain Statement**
- [ ] Grain clearly stated in business terms
- [ ] Grain represents single measurement event
- [ ] All team members agree on grain

**Fact Table Type**
- [ ] Type (transaction/periodic/accumulating) matches business process
- [ ] Considered whether complementary fact tables needed

**Dimensions**
- [ ] All relevant dimensions at declared grain
- [ ] No dimensions violate grain by causing additional rows
- [ ] Degenerate dimensions identified for transaction identifiers
- [ ] Role-playing dimensions handled via views/aliases

**Facts**
- [ ] All facts true to declared grain
- [ ] Facts expressed as extended amounts (additive) where possible
- [ ] Additivity classification documented for each fact
- [ ] Derived facts stored if frequently calculated
- [ ] Multiple currencies/units of measure handled correctly

**Keys**
- [ ] Dimension foreign keys use surrogate keys (not natural keys)
- [ ] Primary key defined (subset of FKs + degenerate dimensions)
- [ ] No null foreign keys (use special dimension rows)

---

## Quick Reference: Fact Table Type Comparison

| Characteristic | Transaction | Periodic Snapshot | Accumulating Snapshot |
|----------------|-------------|-------------------|----------------------|
| **Grain** | One row per event | One row per period | One row per lifecycle |
| **Date dimension** | Event date | Snapshot period date | Multiple milestone dates |
| **Fact table load** | Insert | Insert snapshots | Insert + UPDATE |
| **Fact table updates** | None | None | Updated repeatedly |
| **Date span** | Single point in time | Single point in time | Multiple dates |
| **Example facts** | Quantity, amounts | Balances, period totals | Pipeline quantities, lags |
| **Typical use** | Detailed event analysis | Trend analysis | Workflow/pipeline analysis |
| **Density** | Sparse | Dense | One row per item |

---

## Integration with Bus Matrix

On enterprise data warehouse bus matrix:
- Each business process row typically spawns one or more fact tables
- Fact table type depends on measurement characteristics
- All fact tables in a row share common conformed dimensions
- Different fact table types for same process are complementary, not competitive

---

## Examples Across Industries

**Retail**: POS sales (transaction) + Store inventory (periodic snapshot)
**Manufacturing**: Production runs (transaction) + Work-in-process (accumulating snapshot)
**Financial Services**: Account transactions (transaction) + Monthly balances (periodic snapshot)
**Healthcare**: Patient visits (transaction) + Claims pipeline (accumulating snapshot)
**Education**: Course registrations (factless transaction) + Student enrollment (periodic snapshot)

---

## Next Steps After Designing Fact Table

1. Design dimension tables with robust attributes
2. Implement slowly changing dimension strategy
3. Design physical schema (indexes, partitions, aggregates)
4. Develop ETL processes for fact table population
5. Create BI applications leveraging dimensional model

---

## Additional Resources

For deeper coverage of specific topics:
- **Transaction details**: [transaction-fact-patterns.md](transaction-fact-patterns.md)
- **Periodic snapshot details**: [periodic-snapshot-patterns.md](periodic-snapshot-patterns.md)
- **Accumulating snapshot details**: [accumulating-snapshot-patterns.md](accumulating-snapshot-patterns.md)
- **Fact additivity**: [fact-additivity-guide.md](fact-additivity-guide.md)

For related dimensional modeling topics, see other skills in this series.
