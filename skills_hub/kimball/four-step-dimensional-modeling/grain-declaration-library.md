# Grain Declaration Library

## Purpose

This library provides a comprehensive collection of grain statement examples across different industries, business processes, and fact table types. Use these examples as patterns for declaring grain in your own dimensional models.

## Why Grain Matters

The grain is the single most important decision in dimensional modeling. It:
- Defines exactly what one fact table row represents
- Determines which dimensions and facts are permissible
- Serves as a "binding contract" for the entire design
- Prevents circular discussions and rogue facts
- Ensures design team consensus

**Remember**: Express grain in business terms, not as a list of dimension keys.

---

## Grain Declaration Patterns

### Pattern 1: Transaction Fact Table Grains

**Format**: "One row per [transaction event] [optional qualifier]"

**Characteristics:**
- Most common fact table type
- Represents discrete business events
- Typically sparse (rows only where events occur)
- Highly dimensional
- Facts usually additive

### Pattern 2: Periodic Snapshot Fact Table Grains

**Format**: "One row per [entity] per [time period]"

**Characteristics:**
- Summarizes status over standard periods (day, week, month)
- Uniformly dense (rows exist even with no activity)
- Often contains semi-additive facts (balances)
- Good for trend analysis

### Pattern 3: Accumulating Snapshot Fact Table Grains

**Format**: "One row per [process lifecycle entity]" OR "One row per [workflow instance]"

**Characteristics:**
- Tracks definable process with beginning, milestones, and end
- Rows updated as process progresses (unique among fact table types)
- Contains multiple date dimensions (one per milestone)
- Includes lag/duration facts

---

## Transaction Fact Table Grain Examples

### Retail and E-Commerce

**Retail POS Sales**
```
✓ GOOD: One row per product on a POS sales transaction
         (assuming same product scanned multiple times is consolidated into one line item)

✓ GOOD: One row per scan of a product at the POS register
         (if POS system records every individual scan separately)

✓ GOOD: One row per line item on a customer's shopping cart receipt

✗ BAD:  One row per date key, product key, store key, and promotion key
         (This lists dimension keys, not business grain)

✗ BAD:  One row per sales transaction
         (Too high level; can't analyze individual products sold)
```

**E-Commerce Clickstream**
```
✓ GOOD: One row per page view event for a website visitor

✓ GOOD: One row per click event on a web page

✓ GOOD: One row per visitor session on the website
```

**E-Commerce Orders**
```
✓ GOOD: One row per product line item on a web order

✓ GOOD: One row per e-commerce order header
         (if only interested in order-level metrics, not product details)
```

### Financial Services

**Banking Transactions**
```
✓ GOOD: One row per account transaction (deposit, withdrawal, transfer, fee)

✓ GOOD: One row per ATM transaction

✓ GOOD: One row per debit card purchase transaction

✓ GOOD: One row per check cleared against an account
```

**Insurance Claims**
```
✓ GOOD: One row per claim transaction (open, reserve adjustment, payment, subrogation, closure)

✓ GOOD: One row per claim payment disbursement

✓ GOOD: One row per claim reserve change event
```

**Investment Trades**
```
✓ GOOD: One row per securities trade execution

✓ GOOD: One row per trade order placement
         (different grain from execution; may need both fact tables)
```

### Supply Chain and Manufacturing

**Purchase Orders**
```
✓ GOOD: One row per line item on a purchase order

✓ GOOD: One row per purchase order header
         (if line-item detail not needed)
```

**Warehouse Shipping**
```
✓ GOOD: One row per line item on a bill of lading

✓ GOOD: One row per shipment package

✓ GOOD: One row per individual item picked from warehouse location
```

**Manufacturing Production**
```
✓ GOOD: One row per production run of a product at a manufacturing line

✓ GOOD: One row per unit produced (if tracking individual units)

✓ GOOD: One row per quality inspection event
```

### Healthcare

**Claims and Billing**
```
✓ GOOD: One row per line item on a doctor's bill (claim)

✓ GOOD: One row per medical procedure performed

✓ GOOD: One row per prescription filled at a pharmacy
```

**Electronic Medical Records**
```
✓ GOOD: One row per patient encounter (office visit, hospital admission)

✓ GOOD: One row per diagnostic test performed

✓ GOOD: One row per medication administration event
```

### Telecommunications

**Call Detail Records**
```
✓ GOOD: One row per phone call (completed call)

✓ GOOD: One row per call attempt (including failed/blocked calls)

✓ GOOD: One row per text message sent

✓ GOOD: One row per data usage session
```

### Transportation

**Airlines**
```
✓ GOOD: One row per individual passenger flight segment
         (one city pair on one flight on one date)

✓ GOOD: One row per boarding pass scanned at gate

✓ GOOD: One row per baggage item checked
```

**Shipping and Logistics**
```
✓ GOOD: One row per package delivery event

✓ GOOD: One row per truck trip segment (one origin-destination pair)
```

### Hospitality

**Hotel Reservations**
```
✓ GOOD: One row per room night booked (one room for one night)

✓ GOOD: One row per hotel reservation (multi-night stay as one row)
         (different grain; may need both)

✓ GOOD: One row per guest check-in event
```

### Education

**Student Registration**
```
✓ GOOD: One row per student enrolled in a class section in a term

✓ GOOD: One row per class session attended by a student

✓ GOOD: One row per assignment submitted by a student
```

### Human Resources

**Payroll**
```
✓ GOOD: One row per employee per pay period

✓ GOOD: One row per paycheck line item (earnings, deductions, taxes)
```

### Media and Entertainment

**Video Streaming**
```
✓ GOOD: One row per video viewing session by a subscriber

✓ GOOD: One row per video stream quality change event

✓ GOOD: One row per video pause/resume event
```

---

## Periodic Snapshot Fact Table Grain Examples

### Pattern: "One row per [entity] per [time period]"

### Retail and E-Commerce

**Inventory Levels**
```
✓ GOOD: One row per product in each warehouse at the end of each day

✓ GOOD: One row per product SKU per store per day

✓ GOOD: One row per product per distribution center at the end of each week
```

**Daily Sales Rollup** (if transaction detail not needed)
```
✓ GOOD: One row per product per store per day (daily sales summary)

✗ BAD:  One row per product on a POS transaction
         (This is transaction grain, not periodic snapshot)
```

### Financial Services

**Account Balances**
```
✓ GOOD: One row per bank account per month (end-of-month balance snapshot)

✓ GOOD: One row per bank account per day (daily balance snapshot)

✓ GOOD: One row per investment portfolio per month

✓ GOOD: One row per loan account per month (outstanding balance snapshot)
```

**Insurance Policy Status**
```
✓ GOOD: One row per insurance policy per month (monthly premium snapshot)

✓ GOOD: One row per covered item (vehicle, property) per coverage period
```

### Human Resources

**Employee Headcount**
```
✓ GOOD: One row per employee per month (monthly headcount snapshot)

✓ GOOD: One row per department per month (departmental headcount summary)
```

### Accounting

**General Ledger Balances**
```
✓ GOOD: One row per account per month (end-of-month balance)

✓ GOOD: One row per account per fiscal period (period-end balance)
```

**Budgets**
```
✓ GOOD: One row per budget line item per fiscal period

✗ BAD:  One row per budget (not clear what time period)
```

### Manufacturing

**Work-in-Progress Inventory**
```
✓ GOOD: One row per production order per day (WIP snapshot)

✓ GOOD: One row per manufacturing line per shift (production output summary)
```

### Healthcare

**Patient Census**
```
✓ GOOD: One row per hospital bed per day (occupancy snapshot)

✓ GOOD: One row per patient per day (patient status snapshot)
```

---

## Accumulating Snapshot Fact Table Grain Examples

### Pattern: "One row per [process instance/lifecycle entity]"

### Retail and E-Commerce

**Order Fulfillment Pipeline**
```
✓ GOOD: One row per order line item from placement through delivery

✓ GOOD: One row per customer order from placement through fulfillment
         (if tracking order-level metrics, not line item details)
```

### Supply Chain

**Purchase Order Fulfillment**
```
✓ GOOD: One row per purchase order line item from requisition through receipt

✓ GOOD: One row per shipment from origin through delivery
```

### Insurance

**Claim Processing Lifecycle**
```
✓ GOOD: One row per insurance claim from initial report through final closure

✓ GOOD: One row per claim coverage item (for complex claims with multiple coverages)
```

### Education

**Student Application Pipeline**
```
✓ GOOD: One row per student application from submission through enrollment decision

✓ GOOD: One row per student enrolled in a course from registration through final grade
```

### Healthcare

**Patient Treatment Episode**
```
✓ GOOD: One row per patient treatment episode from admission through discharge

✓ GOOD: One row per surgery from pre-op through recovery
```

### Manufacturing

**Production Work Order**
```
✓ GOOD: One row per work order from initiation through completion

✓ GOOD: One row per product unit from start of assembly through final inspection
```

### Services

**Customer Service Case**
```
✓ GOOD: One row per support ticket from opening through resolution

✓ GOOD: One row per warranty claim from filing through disposition
```

### Financial Services

**Loan Origination Process**
```
✓ GOOD: One row per loan application from submission through funding (or rejection)

✓ GOOD: One row per mortgage from application through closing
```

---

## Special Case Grains

### Factless Fact Tables

**Event Coverage (Promotion Coverage)**
```
✓ GOOD: One row per product on promotion in a store on a day
         (regardless of whether product sold)

Purpose: Enables analysis of "what didn't happen" by comparing to actual sales
```

**Event Attendance (Class Attendance)**
```
✓ GOOD: One row per student enrolled in a class for a term
         (regardless of whether student attended any particular session)

Purpose: Enables analysis of no-shows by comparing to actual attendance transactions
```

**Eligibility/Coverage**
```
✓ GOOD: One row per employee eligible for a benefit in a coverage period
         (regardless of whether employee utilized benefit)
```

### Consolidated Fact Tables

**Budget vs. Actual**
```
✓ GOOD: One row per account per fiscal period combining budget and actual amounts

Note: Combines periodic snapshot (budget) with transaction rollup (actual) at same grain
```

**Multi-Source Profitability**
```
✓ GOOD: One row per product per store per day combining sales, costs, and allocated expenses

Note: Combines facts from multiple source processes at consistent grain
```

---

## Grain Declaration Best Practices

### DO: Express Grain in Business Terms

**Good Examples:**
- "One row per product on a customer's sales receipt"
- "One row per bank account each month"
- "One row per insurance claim from initial report through closure"

**Bad Examples:**
- "One row per date key, product key, and store key" (lists keys, not business grain)
- "Daily product sales by store" (ambiguous - is it atomic transactions or daily rollup?)
- "Product dimension and date dimension" (confuses dimensions with grain)

### DO: Be Specific About Transaction Granularity

**Specify consolidation rules when relevant:**
- "One row per product on a POS transaction, with multiple scans of same product consolidated into single line item"
- "One row per individual product scan at the register, with no consolidation"

**Clarify header vs. line item:**
- "One row per order line item" (line item grain)
- "One row per order header" (order grain)

### DO: Specify Time Period for Snapshots

**Good Examples:**
- "One row per account per month" (not "one row per account")
- "One row per employee per pay period" (not "one row per employee")
- "One row per product per store per day" (not "one row per product per store")

### DO: Identify the Lifecycle Entity for Accumulating Snapshots

**Good Examples:**
- "One row per order from placement through delivery"
- "One row per claim from report through closure"
- "One row per student application from submission through decision"

### DO: Get Team Consensus on Grain

**Questions to validate grain:**
- Can everyone on team explain what one row represents?
- Do proposed dimensions take on single values at this grain?
- Are proposed facts true to this grain?
- Does grain match source system's lowest level of detail?
- Does grain enable unpredictable user queries?

### DON'T: Mix Grains in One Fact Table

**Wrong:**
```
✗ BAD: Fact table contains both:
       - Daily product sales summaries (periodic snapshot grain)
       - Individual POS transactions (transaction grain)
```

**Right:**
```
✓ GOOD: Separate fact tables:
        - Retail Sales Transaction Fact (transaction grain)
        - Daily Product Sales Summary Fact (periodic snapshot grain)
```

### DON'T: Start with Aggregated Grain

**Wrong:**
```
✗ BAD: "One row per product per month" as initial design
       (Can't answer daily questions later)
```

**Right:**
```
✓ GOOD: "One row per product on a POS transaction" (atomic)
        Then create separate aggregate table if needed for performance
```

### DON'T: Confuse Grain with Primary Key

**Wrong:**
```
✗ BAD: "The grain is Date_Key + Product_Key + Store_Key"
       (This is the primary key structure, not the business grain)
```

**Right:**
```
✓ GOOD: "One row per product sold in a store on a day"
        Then derive: Primary key is (Date_Key, Product_Key, Store_Key)
```

---

## Grain Testing Questions

Use these questions to validate your grain declaration:

### Question 1: Can you explain the grain to a business user?
- If you can't explain it in simple business terms, it's not clear enough
- Business users should immediately understand what one row represents

### Question 2: Does every proposed dimension take on a single value at this grain?
- If a dimension causes multiple rows, it violates the grain
- Either remove the dimension or change the grain

### Question 3: Is every proposed fact true to the grain?
- Manager's salary doesn't belong in retail sales grain
- Promotion cost doesn't belong in individual transaction grain

### Question 4: Is this the most atomic grain captured by the source system?
- If yes, you're on solid ground
- If no, why are you aggregating? (You're limiting future flexibility)

### Question 5: Could this grain handle unpredictable future questions?
- Atomic grain: "Yes, can slice any way users want"
- Summarized grain: "Only if users want these specific dimensions"

### Question 6: Do different grains exist in the design?
- If yes, they belong in separate fact tables
- Mixing grains is the #1 dimensional modeling error

---

## Grain Evolution Examples

Sometimes grain needs to evolve as you learn more. This is OK if you return to step 2 and update steps 3 and 4.

### Example 1: Retail Sales Grain Evolution

**Initial grain (too high level):**
```
"One row per sales transaction per store per day"
Problem: Can't analyze individual products sold
```

**Revised grain (better):**
```
"One row per product on a POS sales transaction"
Result: Can analyze products, still can roll up to transaction or day
```

### Example 2: Order Fulfillment Grain Evolution

**Initial grain:**
```
"One row per order"
Problem: Can't analyze individual line items
```

**Revised grain:**
```
"One row per order line item from placement through delivery"
Result: Can analyze line items, can still aggregate to order level
```

### Example 3: Inventory Grain Evolution

**Initial grain (transaction only):**
```
"One row per inventory movement transaction"
Problem: Hard to answer "what is current inventory level?"
```

**Solution (add second fact table):**
```
Keep transaction fact table AND add:
"One row per product per warehouse per day" (periodic snapshot)
Result: Both transaction detail and snapshot status available
```

---

## Summary

The grain declaration is the foundation of dimensional modeling. A clear, business-focused grain statement:

- Creates immediate team consensus
- Determines valid dimensions and facts
- Prevents design errors and rework
- Enables maximum analytic flexibility

**Key Principles:**
1. Express grain in business terms, not dimension keys
2. Start with atomic grain (lowest level of source detail)
3. Get team consensus before proceeding to dimensions and facts
4. Use separate fact tables for different grains
5. Be willing to revisit grain if steps 3 or 4 reveal issues

**Grain Patterns:**
- **Transaction**: "One row per [event]"
- **Periodic Snapshot**: "One row per [entity] per [time period]"
- **Accumulating Snapshot**: "One row per [process instance]"

Use this library as a reference when declaring grain for your dimensional models. Find similar business processes and adapt the grain patterns to your specific needs.
