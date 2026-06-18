# Type 2 SCD History Reconstruction

## Overview

Type 2 Slowly Changing Dimensions track historical changes by adding new rows to the dimension table. During historic load, you must **reconstruct** this history from historical sources to provide accurate context for historical facts.

## Why History Reconstruction Matters

**Business Expectation:**
If business users identify an attribute as important for tracking history, they want that history **going back in time**, not just from the data warehouse implementation date forward.

**Example Scenario:**
- Product "Widget A" was in category "Electronics" from 2020-2022
- Product "Widget A" moved to category "Smart Home" in 2023
- Sales from 2021 should report under "Electronics" category
- Sales from 2024 should report under "Smart Home" category

Without history reconstruction, all historical sales would incorrectly show under current category.

## Challenges

<challenges>
**1. Source Data Availability**
- Historical snapshots may not exist
- Source systems often keep only current state
- Archives may be incomplete or inconsistent

**2. Reconstruction Difficulty**
- Sometimes difficult to reconstruct
- Sometimes completely impossible
- Requires historical snapshots, archives, or transaction logs

**3. Attribute Selection**
- Not all attributes need history tracking
- Prioritize based on business analytical needs
- Resource constraints limit what can be reconstructed

**4. Data Quality**
- Historical data may have quality issues
- Missing values in archives
- Inconsistent formats over time
</challenges>

## History Source Options

<history_sources>
**Option 1: Operational System Archives**
- **Best case scenario**
- Periodic snapshots of master files
- Often taken for backup purposes
- May exist on tape or archive storage

**Example:**
```
customer_master_2020_01_31.dat
customer_master_2020_02_28.dat
customer_master_2020_03_31.dat
...
customer_master_2024_12_31.dat
```

**Option 2: Transaction Logs**
- Database transaction logs with before/after values
- Change data capture (CDC) logs
- Audit tables tracking attribute changes

**Example Log Entry:**
```
Timestamp: 2023-03-15 14:32:01
Table: CUSTOMER
Operation: UPDATE
Customer_ID: 12345
Changed_Column: customer_segment
Old_Value: 'Bronze'
New_Value: 'Silver'
```

**Option 3: Historical Snapshots in Separate Tables**
- Some source systems maintain history tables
- Effective-dated records
- Archive tables with timestamp columns

**Option 4: Derived from Fact Transactions**
- Infer dimension changes from transaction patterns
- Customer status changes based on purchase behavior
- Product category changes from sales system

**Example:**
- Customer made first purchase over $10K on 2023-06-15
- Infer customer_segment changed from 'Standard' to 'Premium' on that date

**Option 5: External Historical Data**
- Third-party data providers
- Historical address databases
- Industry classification changes
- Government data (e.g., zip code changes)

**Option 6: No History Available**
- **Reality check**: Sometimes history cannot be reconstructed
- **Options:**
  - Set all historical rows to current values (Type 1 treatment for history)
  - Use "Unknown Historical Value" placeholder
  - Document limitation and set business expectations
</history_sources>

## Reconstruction Process

<reconstruction_process>
**Step 1: Gather Historical Snapshots**

Collect all available historical snapshots of the dimension:

```sql
-- Extract monthly snapshots from archives
SELECT customer_id, customer_name, customer_segment, customer_status,
       '2020-01-31' as snapshot_date
FROM archive.customer_master_202001
UNION ALL
SELECT customer_id, customer_name, customer_segment, customer_status,
       '2020-02-29' as snapshot_date
FROM archive.customer_master_202002
UNION ALL
-- ... continue for all snapshots
SELECT customer_id, customer_name, customer_segment, customer_status,
       '2024-12-31' as snapshot_date
FROM archive.customer_master_202412;
```

**Step 2: Identify Type 2 Attributes**

Know which attributes require history tracking:

```python
# Configuration
type2_attributes = ['customer_segment', 'customer_status', 'credit_limit']
type1_attributes = ['customer_name', 'customer_phone', 'customer_email']
```

**Step 3: Sort and Compare Snapshots**

For each natural key, sort snapshots chronologically and compare:

```python
# Pseudocode
for each customer_id in snapshots:
    customer_snapshots = get_snapshots(customer_id).sort_by(snapshot_date)

    previous_snapshot = None
    dimension_rows = []

    for snapshot in customer_snapshots:
        if previous_snapshot is None:
            # First snapshot - create initial dimension row
            create_dimension_row(snapshot, effective_date=snapshot.date)

        else:
            # Compare Type 2 attributes
            if type2_changed(previous_snapshot, snapshot, type2_attributes):
                # Type 2 change detected - create new dimension row
                create_dimension_row(snapshot, effective_date=snapshot.date)

            else:
                # No Type 2 change - update Type 1 attributes in current row
                update_type1_attributes(current_row, snapshot, type1_attributes)

        previous_snapshot = snapshot
```

**Step 4: Assign Surrogate Keys**

Each dimension row (including historical versions) gets unique surrogate key:

```python
surrogate_key = 1
for dimension_row in all_dimension_rows:
    dimension_row.surrogate_key = surrogate_key
    surrogate_key += 1
```

**Step 5: Set Effective and End Dates**

Critical step: Ensure no gaps in time series:

```python
# Sort dimension rows by natural key and effective date
dimension_rows.sort_by(natural_key, effective_date)

for i, row in enumerate(dimension_rows):
    if next_row_exists(i + 1) and same_natural_key(row, dimension_rows[i+1]):
        # Set end date to day before next row's effective date
        row.row_end_date = dimension_rows[i+1].row_effective_date - 1 day
        row.current_flag = 'N'
    else:
        # This is the current (most recent) row
        row.row_end_date = '9999-12-31'  # Far future date
        row.current_flag = 'Y'
```

**Step 6: Handle Gaps in Snapshot History**

If snapshots are monthly but changes occurred mid-month:

**Conservative Approach:**
- Use snapshot date as effective date
- Accept that change date is approximate (within one month)

**Aggressive Approach:**
- Use transaction facts to pinpoint change date
- Cross-reference with fact transactions to find exact date

**Example:**
```sql
-- Find first transaction showing new customer segment
SELECT min(transaction_date) as probable_change_date
FROM fact_sales
WHERE customer_natural_key = '12345'
  AND customer_segment_at_transaction = 'Silver'
  AND transaction_date > '2023-02-28'  -- After previous snapshot
  AND transaction_date <= '2023-03-31'; -- Before next snapshot
```
</reconstruction_process>

## Implementation Techniques

<implementation>
**Not Well Suited for Standard SQL**

Type 2 history reconstruction requires row-by-row comparison with state management—not SQL's strength.

**Recommended Approaches:**

**1. Database Cursor (PL/SQL, T-SQL)**

```sql
-- PL/SQL example
DECLARE
    CURSOR snapshot_cursor IS
        SELECT customer_id, snapshot_date, customer_segment, customer_status
        FROM customer_snapshots
        ORDER BY customer_id, snapshot_date;

    v_prev_customer_id NUMBER := NULL;
    v_prev_segment VARCHAR2(50) := NULL;
    v_prev_status VARCHAR2(20) := NULL;
    v_surrogate_key NUMBER := 1;

BEGIN
    FOR rec IN snapshot_cursor LOOP
        IF v_prev_customer_id IS NULL OR v_prev_customer_id != rec.customer_id THEN
            -- First snapshot for this customer
            INSERT INTO dim_customer VALUES (
                v_surrogate_key, rec.customer_id, rec.customer_segment,
                rec.customer_status, rec.snapshot_date, '9999-12-31', 'Y'
            );
            v_surrogate_key := v_surrogate_key + 1;

        ELSIF v_prev_segment != rec.customer_segment OR v_prev_status != rec.customer_status THEN
            -- Type 2 change detected
            -- Update previous row's end date
            UPDATE dim_customer
            SET row_end_date = rec.snapshot_date - 1,
                current_flag = 'N'
            WHERE customer_natural_key = rec.customer_id
              AND current_flag = 'Y';

            -- Insert new row
            INSERT INTO dim_customer VALUES (
                v_surrogate_key, rec.customer_id, rec.customer_segment,
                rec.customer_status, rec.snapshot_date, '9999-12-31', 'Y'
            );
            v_surrogate_key := v_surrogate_key + 1;
        END IF;

        v_prev_customer_id := rec.customer_id;
        v_prev_segment := rec.customer_segment;
        v_prev_status := rec.customer_status;
    END LOOP;
END;
```

**2. Procedural Language Script (Python, Java, C#)**

```python
import pandas as pd
from datetime import datetime, timedelta

def reconstruct_type2_history(snapshots_df, type2_columns):
    """
    Reconstruct Type 2 SCD history from historical snapshots.

    Parameters:
        snapshots_df: DataFrame with columns [natural_key, snapshot_date, ...attributes]
        type2_columns: List of column names that are Type 2 tracked

    Returns:
        DataFrame with dimension rows including surrogate keys and effective dates
    """
    dimension_rows = []
    surrogate_key = 1

    # Sort by natural key and snapshot date
    snapshots_df = snapshots_df.sort_values(['natural_key', 'snapshot_date'])

    # Group by natural key
    for natural_key, group in snapshots_df.groupby('natural_key'):
        previous_row = None

        for _, snapshot in group.iterrows():
            if previous_row is None:
                # First snapshot for this natural key
                dimension_row = create_dimension_row(
                    surrogate_key, natural_key, snapshot,
                    effective_date=snapshot['snapshot_date']
                )
                dimension_rows.append(dimension_row)
                surrogate_key += 1

            else:
                # Check if Type 2 attributes changed
                type2_changed = False
                for col in type2_columns:
                    if previous_row[col] != snapshot[col]:
                        type2_changed = True
                        break

                if type2_changed:
                    # Update previous row's end date
                    dimension_rows[-1]['row_end_date'] = snapshot['snapshot_date'] - timedelta(days=1)
                    dimension_rows[-1]['current_flag'] = 'N'

                    # Create new dimension row
                    dimension_row = create_dimension_row(
                        surrogate_key, natural_key, snapshot,
                        effective_date=snapshot['snapshot_date']
                    )
                    dimension_rows.append(dimension_row)
                    surrogate_key += 1

            previous_row = snapshot

    return pd.DataFrame(dimension_rows)

def create_dimension_row(surrogate_key, natural_key, snapshot, effective_date):
    """Create a dimension row dictionary."""
    return {
        'surrogate_key': surrogate_key,
        'natural_key': natural_key,
        'row_effective_date': effective_date,
        'row_end_date': datetime(9999, 12, 31),  # Far future
        'current_flag': 'Y',
        **{col: snapshot[col] for col in snapshot.index if col not in ['natural_key', 'snapshot_date']}
    }
```

**3. ETL Tool Script Component**

Most ETL tools support scripting transformations:
- Informatica: Expression Transformation with variables
- SSIS: Script Component (C#/VB.NET)
- Talend: tJavaRow component
- DataStage: Transformer stage with stage variables

**Example Logic in ETL Tool:**

```
Input: Sorted by natural_key, snapshot_date
Stage Variables:
    - prev_natural_key
    - prev_type2_hash (hash of Type 2 attributes)
    - current_surrogate_key

For each row:
    current_type2_hash = hash(type2_attributes)

    if (natural_key != prev_natural_key) {
        // New natural key - first historical row
        action = "INSERT"
        effective_date = snapshot_date
        end_date = 9999-12-31
        current_flag = 'Y'
        current_surrogate_key++
    }
    else if (current_type2_hash != prev_type2_hash) {
        // Type 2 change detected
        // Output UPDATE for previous row
        output_update(prev_surrogate_key, end_date = snapshot_date - 1, current_flag = 'N')

        // Output INSERT for new row
        action = "INSERT"
        effective_date = snapshot_date
        end_date = 9999-12-31
        current_flag = 'Y'
        current_surrogate_key++
    }
    else {
        // No Type 2 change - skip
        action = "SKIP"
    }

    prev_natural_key = natural_key
    prev_type2_hash = current_type2_hash
```
</implementation>

## Handling Edge Cases

<edge_cases>
**1. First Snapshot Not at Beginning of Time**

If earliest snapshot is 2022-01-31 but facts exist from 2020:

**Option A:** Create initial dimension row with earliest known values, effective from beginning of fact data
```python
initial_row.row_effective_date = '2020-01-01'  # First fact date
```

**Option B:** Create "Unknown Historical Value" dimension row for pre-snapshot period
```python
initial_row.customer_segment = 'Unknown - Pre-History'
initial_row.row_effective_date = '2020-01-01'
initial_row.row_end_date = '2022-01-30'

# Then first real snapshot becomes effective 2022-01-31
```

**2. Snapshot Frequency Doesn't Match Fact Granularity**

Monthly snapshots but daily facts:

**Reality:** Cannot determine exact day of change within month
**Approach:** Use snapshot date as change date (within one month accuracy acceptable)
**Documentation:** Inform business users of this limitation

**3. Type 2 Attribute Changes Multiple Times in Snapshot Period**

Monthly snapshots miss mid-month changes:

**Detection:** Review source system transaction logs if available
**Refinement:** Use fact transactions to infer intermediate changes
**Practical:** Often not worth the complexity; monthly accuracy sufficient

**4. Dimension Member Deleted and Re-Added**

Customer 12345 exists in Jan, deleted in Jun, re-added in Dec:

**Approach:** Treat as separate dimension member "lifespans"
```python
# First lifespan
row1: customer_id=12345, effective='2024-01-01', end='2024-05-31', current='N'

# Second lifespan (assign new surrogate key)
row2: customer_id=12345, effective='2024-12-01', end='9999-12-31', current='Y'
```

**5. Missing Snapshots in Sequence**

Snapshots for Jan, Feb, Apr, May (March missing):

**Conservative:** Assume no change during March
**Approach:** Carry forward Feb snapshot's values through missing period
</edge_cases>

## Validation and Testing

<validation>
**Validate No Gaps in Time Series:**

```sql
-- Find gaps in effective date coverage
SELECT d1.natural_key,
       d1.row_end_date as gap_start,
       d2.row_effective_date as gap_end,
       d2.row_effective_date - d1.row_end_date as gap_days
FROM dim_customer d1
JOIN dim_customer d2
    ON d1.natural_key = d2.natural_key
    AND d2.row_effective_date = (
        SELECT min(row_effective_date)
        FROM dim_customer d3
        WHERE d3.natural_key = d1.natural_key
          AND d3.row_effective_date > d1.row_end_date
    )
WHERE d2.row_effective_date - d1.row_end_date > 1  -- Gap larger than 1 day
ORDER BY d1.natural_key, gap_start;
```

**Validate Exactly One Current Row per Natural Key:**

```sql
-- Should return zero rows
SELECT natural_key, count(*) as current_row_count
FROM dim_customer
WHERE current_flag = 'Y'
GROUP BY natural_key
HAVING count(*) > 1;
```

**Validate No Overlapping Effective Date Ranges:**

```sql
-- Find overlapping date ranges for same natural key
SELECT d1.natural_key, d1.surrogate_key, d2.surrogate_key
FROM dim_customer d1
JOIN dim_customer d2
    ON d1.natural_key = d2.natural_key
    AND d1.surrogate_key != d2.surrogate_key
WHERE d1.row_effective_date <= d2.row_end_date
  AND d1.row_end_date >= d2.row_effective_date;
```

**Validate History Reconstruction Count:**

```sql
-- Compare total dimension rows to original snapshot rows
SELECT 'Snapshots' as source, count(*) as row_count
FROM customer_snapshots
UNION ALL
SELECT 'Dimension' as source, count(*) as row_count
FROM dim_customer;

-- Dimension count should be <= snapshot count (consolidates unchanged rows)
```

**Spot Check Historical Accuracy:**

```sql
-- For specific customer, show history timeline
SELECT surrogate_key, natural_key, customer_segment, customer_status,
       row_effective_date, row_end_date, current_flag
FROM dim_customer
WHERE natural_key = '12345'
ORDER BY row_effective_date;

-- Manually verify against known business events
```
</validation>

## Performance Considerations

<performance>
**Snapshot Volume:**
- 1M customers × 60 monthly snapshots = 60M snapshot rows to process
- Requires efficient sorting and comparison

**Optimization Techniques:**

1. **Pre-Sort Snapshots**
   ```sql
   CREATE INDEX idx_snapshots ON customer_snapshots(customer_id, snapshot_date);
   ```

2. **Hash Type 2 Attributes**
   ```python
   # Compare hash instead of multiple columns
   type2_hash = hashlib.md5('|'.join([str(row[col]) for col in type2_columns])).hexdigest()
   ```

3. **Process in Batches**
   ```python
   # Process 10K natural keys at a time
   for batch in natural_key_batches:
       reconstruct_batch(batch)
       commit_batch()
   ```

4. **Parallel Processing**
   ```python
   # Partition by natural_key ranges
   # Process partitions in parallel (each is independent)
   partition1: natural_key 1-100000
   partition2: natural_key 100001-200000
   ...
   ```

5. **Incremental State Management**
   ```python
   # For very large dimensions, checkpoint progress
   checkpoint_every = 10000
   if rows_processed % checkpoint_every == 0:
       save_checkpoint(current_natural_key, current_surrogate_key)
   ```
</performance>

## Business Communication

<business_communication>
**Set Expectations:**

"We can reconstruct customer segment history from monthly snapshots. The effective dates will be accurate to within one month—we'll know the segment changed sometime during that month, but not the exact day."

**Document Limitations:**

| Attribute | History Available | Accuracy | Source |
|-----------|------------------|----------|---------|
| Customer Segment | Yes | Monthly | Archive snapshots |
| Customer Status | Yes | Monthly | Archive snapshots |
| Credit Limit | Yes | Daily | Transaction log |
| Customer Email | No | Current only | No historical source |

**Alternative Approaches:**

If history reconstruction is impossible or too costly:
1. **Type 1 treatment**: Load current values for all historical periods (acceptable for some attributes)
2. **Document limitation**: "Customer email shown is current value; historical emails not available"
3. **Phase 2 enhancement**: Add history tracking going forward, accept limitation for past
</business_communication>

## Summary

Type 2 history reconstruction is **challenging but essential** for dimensional model accuracy:

- Requires historical snapshots or transaction logs
- Not suited for standard SQL—use procedural code
- Carefully manage effective and end dates (no gaps!)
- Validate thoroughly before loading facts
- Set business expectations where history is unavailable
- Document limitations and data lineage

Success requires balancing **analytical accuracy** with **practical constraints** of available historical data.
