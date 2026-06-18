# Late-Arriving Data Handling

Complete guide to handling facts and dimensions that arrive delayed in the data warehouse.

## Overview

Late-arriving data occurs when:
- **Late-arriving facts**: Transaction records arrive significantly after the transaction date
- **Late-arriving dimensions**: Dimensional context arrives after the fact record needs to be loaded

Both scenarios require special ETL processing to ensure correct dimensional relationships and historical accuracy.

---

## Late-Arriving Facts

### Problem Statement

**Scenario**: Order transaction from January 15 arrives on February 20.

**Challenge**: Customer dimension has Type 2 changes between transaction date and arrival date.

```
Timeline:
Jan 15: Order placed (customer segment = "Silver")
Feb 1:  Customer promoted to "Gold" segment (Type 2 change)
Feb 20: Order record arrives at data warehouse

Question: Which customer segment should the Jan 15 fact reference?
Answer: "Silver" (the segment effective on Jan 15)
```

### Solution: Historical Surrogate Key Lookup

Standard incremental processing looks up current dimension key. Late-arriving processing must look up historical key.

#### Standard Lookup (for Current Facts)

```sql
-- Gets current customer key
SELECT dc.customer_key
FROM dim.customer dc
WHERE dc.customer_natural_key = :fact_customer_id
  AND dc.current_row_indicator = 'Y';
```

#### Historical Lookup (for Late-Arriving Facts)

```sql
-- Gets customer key effective on transaction date
SELECT dc.customer_key
FROM dim.customer dc
WHERE dc.customer_natural_key = :fact_customer_id
  AND :fact_transaction_date BETWEEN dc.row_effective_date
                                  AND dc.row_end_date;
```

### Complete Late-Arriving Fact Processing

```sql
-- Step 1: Identify late-arriving facts
-- Facts where transaction date << load date
CREATE TABLE staging.late_arriving_facts AS
SELECT *
FROM staging.sales_extract
WHERE transaction_date < CURRENT_DATE - :late_threshold_days;  -- e.g., 7 days

-- Step 2: Surrogate key pipeline with historical lookup
SELECT
    f.transaction_date,
    f.customer_id,
    f.product_id,
    f.quantity,
    f.amount,
    -- Date dimension: straightforward
    dd.date_key,
    -- Product dimension: Type 1 only, use current
    dp.product_key,
    -- Customer dimension: Type 2, need historical key
    dc_historical.customer_key,
    -- Store dimension: Type 2, need historical key
    ds_historical.store_key
FROM staging.late_arriving_facts f
-- Standard lookup for date
INNER JOIN dim.date dd
  ON f.transaction_date = dd.date_actual
-- Current lookup for Type 1 dimensions
LEFT JOIN dim.product dp
  ON f.product_id = dp.product_natural_key
  AND dp.current_row_indicator = 'Y'
-- Historical lookup for Type 2 dimensions
LEFT JOIN dim.customer dc_historical
  ON f.customer_id = dc_historical.customer_natural_key
  AND f.transaction_date BETWEEN dc_historical.row_effective_date
                              AND dc_historical.row_end_date
LEFT JOIN dim.store ds_historical
  ON f.store_id = ds_historical.store_natural_key
  AND f.transaction_date BETWEEN ds_historical.row_effective_date
                              AND ds_historical.row_end_date;
```

### Edge Case: Dimension Didn't Exist Yet

**Problem**: Fact references dimension member that hadn't been created when transaction occurred.

**Example**:
- Transaction: Jan 15
- New customer first created in dimension: Jan 20
- Fact arrives: Feb 10

**Solution**: Create backdated dimension row

```sql
-- Check if dimension existed on transaction date
SELECT customer_key
FROM dim.customer
WHERE customer_natural_key = :fact_customer_id
  AND :fact_transaction_date >= row_effective_date;

-- If not found, create historical row
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    ...
    row_effective_date,     -- Backdate to transaction date
    row_end_date,
    current_row_indicator
) VALUES (
    customer_seq.NEXTVAL,
    :fact_customer_id,
    'Unknown - Late Arriving',  -- Placeholder values
    ...
    :fact_transaction_date,     -- Effective from transaction date
    :current_effective_date - 1, -- End when real dimension created
    'N'                          -- Not current
);

-- Then update current row to have correct effective date
UPDATE dim.customer
SET row_effective_date = :current_effective_date
WHERE customer_natural_key = :fact_customer_id
  AND current_row_indicator = 'Y';
```

### Impact on Semi-Additive Facts

Late-arriving facts may require adjusting subsequent balance snapshots.

**Example**: Late-arriving deposit into bank account

```
Original balance snapshots:
Jan 30: $1,000 (without late deposit)
Feb 28: $1,500 (without late deposit)

Late-arriving fact:
Jan 25: $200 deposit

Corrected balance snapshots:
Jan 30: $1,200 (adjusted +$200)
Feb 28: $1,700 (adjusted +$200)
```

**Processing**:
```sql
-- Identify snapshots occurring after late fact transaction date
UPDATE fact.account_balance
SET balance_amount = balance_amount + :late_deposit_amount,
    late_arrival_adjustment_count = late_arrival_adjustment_count + 1,
    last_update_date = CURRENT_TIMESTAMP
WHERE account_key = :account_key
  AND snapshot_date >= :late_transaction_date;
```

---

## Late-Arriving Dimensions

### Problem Statement

**Scenario**: Fact record arrives before complete dimensional context is available.

**Example**: Real-time sales system receives transaction immediately, but customer master updated hours later.

### Decision: Wait or Proceed?

#### Option 1: Wait for Dimension (Batch Processing)
```
Advantage: Guaranteed correct dimensional relationship
Disadvantage: Delays fact visibility
Use when: Latency requirements allow delay (daily batch)
```

#### Option 2: Load with Provisional Dimension (Real-Time Processing)
```
Advantage: Immediate fact visibility
Disadvantage: Temporary incorrect or incomplete dimension
Use when: Real-time requirements demand it
```

### Solution 1: Default Dimension Row

Create default "Not Yet Assigned" dimension member.

```sql
-- Create default dimension row (one-time setup)
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    customer_segment,
    customer_status,
    row_effective_date,
    row_end_date,
    current_row_indicator
) VALUES (
    -1,  -- Reserved key for unknown
    'UNKNOWN',
    'Unknown - Not Yet Available',
    'Unknown',
    'Provisional',
    DATE '1900-01-01',
    DATE '9999-12-31',
    'Y'
);

-- When fact arrives without dimension
INSERT INTO fact.sales (
    date_key,
    customer_key,    -- Use default key
    product_key,
    quantity,
    amount
) VALUES (
    :date_key,
    -1,              -- Default unknown customer
    :product_key,
    :quantity,
    :amount
);

-- Later: Update fact when real dimension arrives
-- (Destructive update - not ideal)
UPDATE fact.sales
SET customer_key = :real_customer_key
WHERE customer_key = -1
  AND date_key = :transaction_date_key
  AND amount = :amount;  -- Additional filters to find specific fact row
```

**Problem**: Requires destructive updates to fact table (expensive).

### Solution 2: Create Dummy Dimension Row (Preferred)

Assign new surrogate key with dummy attributes. Update attributes later with Type 1.

```sql
-- When fact arrives with new customer natural key
-- but customer not yet in dimension

-- Step 1: Check if already have placeholder for this natural key
SELECT customer_key
FROM dim.customer
WHERE customer_natural_key = :new_customer_id
  AND customer_status = 'Provisional';

-- Step 2: If not found, create dummy dimension row
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_name,
    customer_address,
    customer_segment,
    customer_status,
    row_effective_date,
    row_end_date,
    current_row_indicator
) VALUES (
    customer_seq.NEXTVAL,  -- New surrogate key
    :new_customer_id,      -- Real natural key
    'Pending - ' || :new_customer_id,  -- Placeholder
    'Address Not Yet Available',
    'Unknown',
    'Provisional',         -- Marks as incomplete
    CURRENT_DATE,
    DATE '9999-12-31',
    'Y'
)
RETURNING customer_key INTO :new_surrogate_key;

-- Step 3: Use this surrogate key in fact table
INSERT INTO fact.sales (
    customer_key,  -- Real surrogate key
    ...
) VALUES (
    :new_surrogate_key,  -- Not a default, unique per natural key
    ...
);

-- Step 4: Later, when real customer data arrives, Type 1 update
UPDATE dim.customer
SET customer_name = :real_name,
    customer_address = :real_address,
    customer_segment = :real_segment,
    customer_status = 'Active',  -- No longer provisional
    last_update_date = CURRENT_TIMESTAMP
WHERE customer_natural_key = :new_customer_id
  AND customer_status = 'Provisional';
```

**Advantages**:
- No fact table updates required
- Each unknown natural key gets unique surrogate key
- Correct dimensional relationship preserved
- Attributes filled in later with Type 1 overwrite

### Solution 3: Late-Arriving Type 2 Dimension Update

**Most complex scenario**: Dimension update arrives late AND is a Type 2 change.

**Example**:
```
Timeline:
Jan 10: Customer moved to new address (not yet in DW)
Jan 15: Sale occurs (loaded with old customer address)
Jan 20: Customer address update finally processed
```

**Problem**: Jan 15 sale now references wrong customer address row.

**Solution**: Retroactive Type 2 processing

```sql
-- Step 1: Insert late-arriving Type 2 dimension row
-- with effective date backdated to actual change date
INSERT INTO dim.customer (
    customer_key,
    customer_natural_key,
    customer_address,
    customer_city,
    row_effective_date,      -- Jan 10 (backdated)
    row_end_date,
    current_row_indicator
) VALUES (
    customer_seq.NEXTVAL,
    :customer_id,
    :new_address,
    :new_city,
    DATE '2024-01-10',       -- Actual change date
    DATE '9999-12-31',
    'Y'                      -- Current as of today
);

-- Step 2: Adjust effective dates on previous row
UPDATE dim.customer
SET row_end_date = DATE '2024-01-09',  -- Day before new row effective
    current_row_indicator = 'N'
WHERE customer_natural_key = :customer_id
  AND row_effective_date < DATE '2024-01-10'
  AND row_end_date = DATE '9999-12-31';

-- Step 3: Find and update facts that occurred during new dimension effective period
-- This is the expensive part - requires fact table scan
UPDATE fact.sales
SET customer_key = :new_customer_key_for_new_address
WHERE customer_key = :old_customer_key_for_old_address
  AND date_key BETWEEN :jan10_date_key AND :current_date_key;

-- Alternative Step 3: Use fact table surrogate key for insert + delete
-- Insert corrected facts
INSERT INTO fact.sales
SELECT
    fact_surrogate_key_seq.NEXTVAL,  -- New fact key
    date_key,
    :new_customer_key,               -- Corrected customer key
    product_key,
    quantity,
    amount,
    correction_flag = 'Y'
FROM fact.sales
WHERE customer_key = :old_customer_key
  AND date_key BETWEEN :jan10_date_key AND :current_date_key;

-- Delete original incorrect facts
DELETE FROM fact.sales
WHERE customer_key = :old_customer_key
  AND date_key BETWEEN :jan10_date_key AND :current_date_key;
```

**Minimize this scenario**:
- Prefer real-time dimension feeds over batch
- Implement dimension staging area to buffer dimension changes
- Alert on dimension delays

---

## Automated Late-Arriving Detection

### Fact Latency Monitoring

```sql
-- Identify facts arriving late
SELECT
    transaction_date,
    load_date,
    DATEDIFF(day, transaction_date, load_date) as days_late,
    COUNT(*) as late_fact_count
FROM staging.sales_extract
WHERE load_date > transaction_date + :acceptable_lag_days
GROUP BY transaction_date, load_date
ORDER BY days_late DESC;

-- Alert if latency exceeds threshold
```

### Dimension Latency Monitoring

```sql
-- Identify provisional dimension rows not updated
SELECT
    customer_key,
    customer_natural_key,
    customer_name,
    row_effective_date,
    DATEDIFF(day, row_effective_date, CURRENT_DATE) as days_provisional
FROM dim.customer
WHERE customer_status = 'Provisional'
  AND row_effective_date < CURRENT_DATE - :max_provisional_days
ORDER BY days_provisional DESC;

-- Alert for stale provisional records
```

---

## Handling Strategies by Latency

### Daily Latency (Standard Incremental)
**Facts arrive within 24 hours**
- Standard surrogate key lookup against current dimensions
- No special late-arriving logic needed
- Most common scenario

### Weekly Latency (Moderate Delay)
**Facts arrive within 7 days**
- Use historical surrogate key lookup (BETWEEN effective dates)
- Acceptable for most business processes
- Plan for occasional manual reconciliation

### Monthly Latency (Significant Delay)
**Facts arrive weeks to months late**
- Requires automated historical key lookup
- May need semi-additive balance adjustments
- Implement alerting for business users
- Consider separate "Late Adjustments" mart

### Unknown Latency (Extreme Delay)
**Facts can arrive at any time in history**
- Implement full late-arriving infrastructure
- Partition fact tables to minimize impact
- Provide late-arrival audit reports
- May require periodic full historical refresh

---

## Best Practices

### 1. Design for Late Arrival from Day One

Even if source system is real-time today, prepare for:
- Source system outages causing batch catchup
- Network failures requiring replay
- Data corrections requiring restatement

### 2. Use Fact Table Surrogate Keys

```sql
CREATE TABLE fact.sales (
    fact_key INTEGER PRIMARY KEY,  -- Enables insert + delete for corrections
    date_key INTEGER,
    customer_key INTEGER,
    ...
);
```

Enables update pattern:
```sql
-- Insert corrected row
INSERT ...

-- Delete old row by fact_key
DELETE WHERE fact_key = :old_fact_key;
```

### 3. Minimize Fact Table Updates

Prefer insert + delete over destructive updates:
- Maintains audit trail
- Better for recovery
- Avoids row locking issues
- Simpler to test

### 4. Partition Fact Tables by Date

```sql
CREATE TABLE fact.sales (
    ...
)
PARTITION BY RANGE (date_key) (
    PARTITION p_2024_01 VALUES LESS THAN (20240201),
    PARTITION p_2024_02 VALUES LESS THAN (20240301),
    ...
);
```

**Benefits**:
- Late facts only impact specific partition
- Can reload partition without full table lock
- Easier to identify late-arriving data

### 5. Implement Provisional Status Tracking

Track dimension completeness:

```sql
ALTER TABLE dim.customer ADD (
    data_quality_status VARCHAR(20),  -- 'Complete', 'Provisional', 'Partial'
    missing_attributes VARCHAR(500),  -- List of pending attributes
    provisional_since_date DATE
);
```

### 6. Document Late-Arrival SLAs

Define acceptable delays:
```
- Tier 1 (Critical): < 1 day
- Tier 2 (Important): < 7 days
- Tier 3 (Standard): < 30 days
- Tier 4 (Best effort): No guarantee
```

Alert when SLAs breached.

### 7. Provide Late-Arrival Transparency

```sql
-- Add metadata to facts
ALTER TABLE fact.sales ADD (
    fact_arrival_date DATE,
    fact_arrival_lag_days INTEGER,
    late_arrival_flag CHAR(1)
);

-- Users can filter on data quality
SELECT * FROM fact.sales
WHERE late_arrival_flag = 'N';  -- Only on-time data
```

### 8. Test Late-Arrival Scenarios

```sql
-- Test case: Insert fact for 30 days ago
INSERT INTO staging.sales_extract
VALUES (..., DATE '2024-01-15', ...);  -- Transaction date
-- Load date: 2024-02-15

-- Verify:
-- 1. Correct historical dimension key assigned
-- 2. Semi-additive balances adjusted
-- 3. Aggregates refreshed
-- 4. Audit trail created
```

---

## Common Mistakes to Avoid

1. **Using current dimension key for all facts**
   - Late facts get wrong dimensional context
   - Solution: Check transaction vs. load date, use historical lookup

2. **Ignoring semi-additive impact**
   - Late deposits don't adjust balances forward
   - Solution: Cascade adjustments through balance snapshots

3. **Updating facts in place**
   - Lost audit trail, locking issues
   - Solution: Insert + delete pattern with fact surrogate key

4. **Single "Unknown" dimension row**
   - Cannot distinguish different unknown natural keys
   - Solution: Unique provisional row per natural key

5. **No provisional status tracking**
   - Cannot identify incomplete dimensions
   - Solution: Add status column, monitor aging

6. **Updating fact foreign keys**
   - Expensive, error-prone, breaks audit trail
   - Solution: Create provisional dimension row, update attributes later

7. **No late-arrival alerting**
   - Issues go unnoticed until users complain
   - Solution: Monitor lag metrics, alert on SLA breaches

8. **Complex late-arrival logic in production**
   - Hard to maintain, debug, test
   - Solution: Keep simple, automate common cases, manual handle rare cases

---

## Late-Arrival Processing Checklist

**Before deploying to production**:

- [ ] Identified SLA for fact timeliness per source
- [ ] Implemented late fact detection logic
- [ ] Historical surrogate key lookup for Type 2 dimensions
- [ ] Provisional dimension row creation process
- [ ] Type 1 attribute update for late-arriving dimension details
- [ ] Semi-additive fact adjustment process (if applicable)
- [ ] Fact table surrogate key implemented
- [ ] Insert + delete pattern for fact corrections
- [ ] Late-arrival monitoring and alerting
- [ ] Tested with production-like delay scenarios
- [ ] User communication plan for late adjustments
- [ ] Documented escalation procedure for extreme delays

---

## Summary

Late-arriving data handling requires:

**For Late-Arriving Facts**:
- Historical dimension key lookup using effective dates
- Semi-additive balance adjustments
- Partition-based impact minimization

**For Late-Arriving Dimensions**:
- Provisional dimension rows with unique surrogate keys
- Type 1 updates when complete data arrives
- Avoid default "Unknown" rows that lump everything together

**Infrastructure**:
- Fact table surrogate keys
- Insert + delete pattern for corrections
- Monitoring and alerting
- Testing and documentation

Build late-arrival capability early. It's much harder to retrofit than to design in from the start.
