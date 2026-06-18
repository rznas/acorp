# Multivalued Dimension Bridge Tables - Complete Reference

This reference provides comprehensive guidance on handling multivalued dimensions using bridge table patterns, with emphasis on healthcare diagnoses, education instructors, and other service industry scenarios.

---

## The Multivalued Dimension Problem

### Standard Dimensional Model Assumption

**Normal case:** Each dimension takes on **single value** in context of fact table row.

Example: Patient procedure
- One patient
- One physician
- One facility
- One procedure code
- One date

**Multivalued challenge:** Dimension naturally takes on **multiple values** for single fact event.

Examples:
- Patient has 3-20 simultaneous diagnoses during procedure
- Course taught by 2-3 co-instructors
- Surgical team includes 5-12 physicians and staff
- Product sold with 3-4 promotional offers
- Flight includes 150+ passengers

### Why NOT Multiple Foreign Keys?

**Tempting but problematic approach:**

```
-- DON'T DO THIS for open-ended multivalued dimensions
Procedure Fact Table:
- diagnosis_1_key
- diagnosis_2_key
- diagnosis_3_key
- diagnosis_4_key
...
- diagnosis_20_key??
```

**Problems:**

1. **Unknown maximum:** How many diagnosis slots? 3? 10? 20?
2. **Slots don't represent roles:** Unlike role-playing dimensions (admitting physician vs attending physician), diagnoses don't fit defined roles
3. **Query complexity:** BI tool doesn't know which slot to constrain
   ```sql
   -- User wants "procedures with diabetes diagnosis"
   -- Must check ALL slots!
   WHERE diagnosis_1_key = @diabetes OR
         diagnosis_2_key = @diabetes OR
         diagnosis_3_key = @diabetes OR ...
   ```
4. **Inefficient:** Many NULL slots for patients with few diagnoses
5. **Inflexible:** ALTER TABLE when maximum changes

### The Bridge Table Solution

**Pattern:** Many-to-many relationship between fact table and multivalued dimension

```
Fact Table → Bridge Table → Dimension
(one)         (many)         (one)
```

---

## Bridge Table Structure

### Basic Bridge Table Schema

```
Fact Table:
- fact_key (PK)
- dimension_group_key (FK to bridge table)
- [other dimension FKs]
- [facts]

Bridge Table:
- dimension_group_key (PK, part 1)
- dimension_key (PK, part 2, FK to dimension table)
- [optional: weighting_factor]
- [optional: primary_indicator]

Dimension Table:
- dimension_key (PK)
- dimension_attributes
```

**Composite Primary Key:** (dimension_group_key, dimension_key) uniquely identifies each bridge row.

### Example: Patient Diagnoses

```
Procedure Fact Table:
- procedure_fact_key (PK)
- diagnosis_group_key (FK to diagnosis group bridge)
- patient_key
- physician_key
- procedure_key
- facility_key
- procedure_date_key
- billed_amount
- units

Diagnosis Group Bridge Table:
- diagnosis_group_key (PK, part 1)
- diagnosis_key (PK, part 2, FK to diagnosis dimension)

Diagnosis Dimension:
- diagnosis_key (PK)
- ICD_code
- diagnosis_description
- diagnosis_category
- chronic_indicator
```

**How it works:**

Patient with 3 diagnoses:
1. Fact table row has diagnosis_group_key = 42
2. Bridge table has 3 rows with diagnosis_group_key = 42
3. Each bridge row points to different diagnosis_key (e.g., 101, 205, 387)

---

## Creating and Managing Diagnosis Groups

### Approach 1: Unique Group Per Fact (Don't Do This)

```
-- Creates new diagnosis group for every patient encounter
Patient A, Procedure 1: Group 1 → [Diabetes, Hypertension, CHF]
Patient B, Procedure 1: Group 2 → [Diabetes, Hypertension, CHF]  -- DUPLICATE!
Patient A, Procedure 2: Group 3 → [Diabetes, Hypertension, CHF]  -- DUPLICATE!
```

**Problems:**
- Astronomical number of groups
- Many identical groups (waste of storage)
- Difficult to manage

### Approach 2: Portfolio of Reusable Groups (Recommended)

```
-- Reuse existing groups when diagnosis set matches

Group 1 → [Diabetes, Hypertension, CHF]

Patient A, Procedure 1: Uses Group 1
Patient B, Procedure 1: Uses Group 1  -- REUSE!
Patient A, Procedure 2: Uses Group 1  -- REUSE!
Patient C, Procedure 1: Group 2 → [Diabetes, CHF]  -- Different set, new group
```

**ETL Logic:**

```python
def assign_diagnosis_group(diagnosis_keys_list):
    # Sort diagnosis keys for consistent matching
    sorted_keys = sorted(diagnosis_keys_list)

    # Lookup existing group with this exact set of diagnoses
    existing_group = lookup_diagnosis_group(sorted_keys)

    if existing_group:
        return existing_group.group_key  # REUSE
    else:
        # Create new group
        new_group_key = create_new_diagnosis_group()
        for diagnosis_key in sorted_keys:
            insert_bridge_row(new_group_key, diagnosis_key)
        return new_group_key
```

**Benefits:**
- Reduced storage (groups reused across patients/procedures)
- Easier management
- Faster queries (fewer unique groups)

**Group Key Assignment:**
- Use surrogate key generator (database sequence)
- Store group creation date for audit
- Track group usage frequency for optimization

---

## Weighting Factors in Bridge Tables

### The Weighting Factor Concept

**Purpose:** Allocate fact table metrics across multivalued dimension members to prevent over-counting.

**Extended Bridge Table:**

```
Bridge Table:
- dimension_group_key (PK, part 1)
- dimension_key (PK, part 2)
- weighting_factor (DECIMAL, sum to 1.0 per group)
```

**Example with Weights:**

```
Product Bundle: "Home Office Package"
- Desk (weight 0.50)
- Chair (weight 0.30)
- Lamp (weight 0.20)
Total weight: 1.00

If package sold for $500:
- Desk allocated revenue: $500 × 0.50 = $250
- Chair allocated revenue: $500 × 0.30 = $150
- Lamp allocated revenue: $500 × 0.20 = $100
```

### SQL with Weighting Factors

**Query:** Total revenue by product, weighted allocation

```sql
SELECT
    p.product_name,
    SUM(f.revenue * b.weighting_factor) AS allocated_revenue
FROM
    sales_fact f
    JOIN product_group_bridge b ON f.product_group_key = b.product_group_key
    JOIN product_dim p ON b.product_key = p.product_key
GROUP BY
    p.product_name
ORDER BY
    allocated_revenue DESC;
```

**Result:**
- Each product gets portion of revenue based on weight
- Prevents over-counting when summing across all products
- Total allocated revenue equals total fact table revenue

### When to Use Weighting Factors

**Use weighting factors when:**

✅ **Clear business rules** for allocation exist
- Revenue split by contract percentage
- Cost allocation by activity-based costing percentages
- Work effort distribution across team members

✅ **Realistic weights** can be determined
- Product bundle pricing (manufacturer knows component values)
- Project staffing allocations (project manager assigns percentages)

✅ **Business community agrees** on allocation method
- Finance approves cost allocation logic
- Sales agrees on commission splitting rules

**Examples where weighting works:**

1. **Employee project assignments:**
   ```
   Employee A works on 3 projects:
   - Project X: 50% time
   - Project Y: 30% time
   - Project Z: 20% time
   ```

2. **Multi-fund investment allocations:**
   ```
   Customer portfolio includes:
   - Stock Fund A: 60% of portfolio value
   - Bond Fund B: 30% of portfolio value
   - Cash Fund C: 10% of portfolio value
   ```

### When NOT to Use Weighting Factors

**Omit weighting factors when:**

❌ **No agreement** on how to weight
- "How much does diabetes diagnosis contribute to surgery cost vs hypertension?"
- Impossible to determine realistic allocation

❌ **No enthusiasm** from business community
- Finance doesn't want allocated costs (wants actuals)
- Analysts prefer impact analysis over allocated metrics

❌ **Allocation would be arbitrary**
- Equal weighting (1/N) provides no real value
- Guessing percentages undermines data credibility

**Examples where weighting doesn't work:**

1. **Patient diagnoses:** Impossible to allocate procedure cost across 12 simultaneous diagnoses
2. **Course instructors:** Arbitrary to split student credit hours across co-teachers
3. **Surgical team:** Can't reasonably allocate surgery cost across 8 team members

---

## Queries Without Weighting Factors

### Impact Analysis Approach

When bridge tables omit weighting factors, analysis focuses on **impact questions**:

**Example: Healthcare Diagnoses**

```sql
-- Question: "What is total billed amount for procedures involving diabetes diagnosis?"

SELECT
    d.diagnosis_description,
    COUNT(DISTINCT f.procedure_fact_key) AS procedure_count,
    SUM(f.billed_amount) AS total_billed_amount
FROM
    procedure_fact f
    JOIN diagnosis_group_bridge b ON f.diagnosis_group_key = b.diagnosis_group_key
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
WHERE
    d.ICD_code = 'E11.9'  -- Type 2 Diabetes
GROUP BY
    d.diagnosis_description;
```

**Result interpretation:**

- `total_billed_amount` represents **procedures where diabetes was involved** as one of potentially many diagnoses
- **Over-counting:** Same procedure counted multiple times if analyzing multiple diagnoses
- **Impact metric:** Shows scope of diabetes-related procedures, not allocated cost

**Acceptable for impact questions:**
- "How many procedures involved congestive heart failure?"
- "Which diagnoses appear most frequently in high-cost procedures?"
- "What is total billed amount for procedures with at least one cancer diagnosis?"

**NOT acceptable for allocation:**
- "What is diabetes's share of procedure costs?" (Can't answer without weights)

### Primary Indicator as Alternative

**Bridge table with primary indicator:**

```
Diagnosis Group Bridge Table:
- diagnosis_group_key
- diagnosis_key
- primary_diagnosis_indicator (Y/N or 1/0)
```

**Use case:** Identify one "most important" diagnosis even if weights unknown.

```sql
-- Allocate 100% of cost to primary diagnosis only

SELECT
    d.diagnosis_description,
    SUM(f.billed_amount) AS allocated_billed_amount
FROM
    procedure_fact f
    JOIN diagnosis_group_bridge b ON f.diagnosis_group_key = b.diagnosis_group_key
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
WHERE
    b.primary_diagnosis_indicator = 'Y'  -- Only primary diagnosis
GROUP BY
    d.diagnosis_description;
```

**Benefits:**
- Avoids over-counting
- Business rule simple (primary = 100%, others = 0%)
- Medically meaningful (admitting diagnosis, primary reason for visit)

**Limitation:**
- Doesn't reflect true multi-factor nature of medical conditions
- May understate impact of secondary diagnoses

---

## Alternative: Diagnosis Group Dimension

### The Extra Table Approach

Some modeling tools insist on proper FK-to-PK relationships. Bridge table's many-to-many join violates this.

**Solution:** Insert diagnosis group dimension between fact and bridge.

```
Fact Table → Diagnosis Group Dimension → Diagnosis Group Bridge → Diagnosis Dimension
```

**Schema:**

```
Procedure Fact Table:
- diagnosis_group_key (FK to diagnosis group dimension)
- [other dimensions and facts]

Diagnosis Group Dimension:
- diagnosis_group_key (PK)
- diagnosis_group_label (e.g., "Kimball Syndrome")
- diagnosis_count (number of diagnoses in group)
- group_created_date

Diagnosis Group Bridge Table:
- diagnosis_group_key (FK to diagnosis group dimension)
- diagnosis_key (FK to diagnosis dimension)

Diagnosis Dimension:
- diagnosis_key (PK)
- [diagnosis attributes]
```

**Benefits:**

- Conventional many-to-one joins from fact table to diagnosis group dimension
- Many-to-one join from bridge table to diagnosis group dimension
- Modeling tools happy with FK-to-PK relationships

**Optional diagnosis group dimension attributes:**

- **Group labels:** Named clusters of related diagnoses
  - Example: "Diabetes Complications Cluster" for [Diabetes, Neuropathy, Retinopathy]
- **Group metadata:** Creation date, last used date, usage frequency
- **Group statistics:** Diagnosis count, average billed amount for procedures with this group

**Typically no new information unless:**
- Cluster labels exist (Kimball Syndrome, Metabolic Syndrome, etc.)
- Group-level aggregations needed for performance

---

## Evolving Diagnosis Groups (Temporal Bridge Tables)

### Use Case: Inpatient Hospital Stays

**Challenge:** Diagnosis group evolves during multi-day hospital stay.

- **Day 1 admission:** [Chest pain, shortness of breath]
- **Day 3 tests completed:** [Myocardial infarction, coronary artery disease, diabetes]
- **Day 7 discharge:** [MI, CAD, diabetes, pneumonia]

### Temporal Bridge Table with Date Stamps

```
Diagnosis Group Bridge Table:
- diagnosis_group_key
- diagnosis_key
- effective_date
- expiration_date
- [optional: primary_diagnosis_indicator]

Composite PK: (diagnosis_group_key, diagnosis_key, effective_date)
```

**Example rows for Group 55 (Patient A's hospital stay):**

| diagnosis_group_key | diagnosis_key | effective_date | expiration_date | Diagnosis |
|---------------------|---------------|----------------|-----------------|-----------|
| 55 | 101 | 2024-01-10 | 2024-01-12 | Chest pain |
| 55 | 205 | 2024-01-10 | 2024-01-12 | Shortness of breath |
| 55 | 310 | 2024-01-13 | 9999-12-31 | Myocardial infarction |
| 55 | 315 | 2024-01-13 | 9999-12-31 | Coronary artery disease |
| 55 | 387 | 2024-01-13 | 9999-12-31 | Diabetes Type 2 |
| 55 | 420 | 2024-01-17 | 9999-12-31 | Pneumonia |

**Query: Diagnoses effective on specific date**

```sql
-- What diagnoses were active on Jan 15, 2024?

SELECT
    d.diagnosis_description
FROM
    diagnosis_group_bridge b
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
WHERE
    b.diagnosis_group_key = 55
    AND '2024-01-15' >= b.effective_date
    AND '2024-01-15' < b.expiration_date;

-- Result: MI, CAD, Diabetes (NOT chest pain, shortness of breath, pneumonia)
```

**ETL Complexity:**

- More complex updates to bridge table
- Must manage date ranges carefully
- Avoid overlapping date ranges for same diagnosis in same group
- Use 9999-12-31 as "current" expiration date
- Update expiration date when diagnosis removed from group

**Trade-off:**
- ✅ Accurate temporal tracking of evolving diagnoses
- ❌ Significantly more complex ETL processing
- ❌ More complex BI queries

**When to use:** Hospital inpatient scenarios where diagnosis evolution is critical for outcomes analysis.

**When to skip:** Outpatient visits or ER encounters where diagnosis group is static.

---

## SQL Query Patterns with Bridge Tables

### Pattern 1: Impact Analysis (No Weights)

```sql
-- Total procedures and billed amount by diagnosis

SELECT
    d.diagnosis_category,
    d.diagnosis_description,
    COUNT(DISTINCT f.procedure_fact_key) AS procedure_count,
    SUM(f.billed_amount) AS total_billed_amount
FROM
    procedure_fact f
    JOIN diagnosis_group_bridge b ON f.diagnosis_group_key = b.diagnosis_group_key
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
GROUP BY
    d.diagnosis_category,
    d.diagnosis_description
ORDER BY
    total_billed_amount DESC;
```

**Over-counting:** If procedure has 3 diagnoses, same billed_amount counted 3 times.

**Acceptable for:** Understanding scope and impact of diagnoses on procedure volume and charges.

### Pattern 2: Allocated Analysis (With Weights)

```sql
-- Total revenue by product with weighting factors

SELECT
    p.product_category,
    p.product_name,
    SUM(f.revenue * b.weighting_factor) AS allocated_revenue,
    COUNT(DISTINCT f.sales_fact_key) AS transaction_count
FROM
    sales_fact f
    JOIN product_group_bridge b ON f.product_group_key = b.product_group_key
    JOIN product_dim p ON b.product_key = p.product_key
GROUP BY
    p.product_category,
    p.product_name
ORDER BY
    allocated_revenue DESC;
```

**No over-counting:** Weighting factors ensure allocated_revenue sums to total fact table revenue.

### Pattern 3: Primary Member Only

```sql
-- Allocate 100% to primary diagnosis

SELECT
    d.diagnosis_description,
    COUNT(DISTINCT f.procedure_fact_key) AS procedure_count,
    SUM(f.billed_amount) AS total_billed_amount
FROM
    procedure_fact f
    JOIN diagnosis_group_bridge b ON f.diagnosis_group_key = b.diagnosis_group_key
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
WHERE
    b.primary_diagnosis_indicator = 1  -- Primary only
GROUP BY
    d.diagnosis_description;
```

**No over-counting:** Each procedure counted once (primary diagnosis only).

**Trade-off:** Ignores impact of secondary diagnoses.

### Pattern 4: Diagnosis Co-Occurrence

```sql
-- Find diagnoses that frequently appear together

SELECT
    d1.diagnosis_description AS diagnosis_1,
    d2.diagnosis_description AS diagnosis_2,
    COUNT(DISTINCT b1.diagnosis_group_key) AS co_occurrence_count
FROM
    diagnosis_group_bridge b1
    JOIN diagnosis_group_bridge b2 ON b1.diagnosis_group_key = b2.diagnosis_group_key
    JOIN diagnosis_dim d1 ON b1.diagnosis_key = d1.diagnosis_key
    JOIN diagnosis_dim d2 ON b2.diagnosis_key = d2.diagnosis_key
WHERE
    b1.diagnosis_key < b2.diagnosis_key  -- Avoid duplicates and self-joins
GROUP BY
    d1.diagnosis_description,
    d2.diagnosis_description
HAVING
    COUNT(DISTINCT b1.diagnosis_group_key) > 100  -- Frequently co-occurring
ORDER BY
    co_occurrence_count DESC;
```

**Use case:** Identify diagnosis patterns, comorbidities, syndrome clustering.

### Pattern 5: Constrain to Specific Diagnosis Combination

```sql
-- Find procedures where patient had BOTH diabetes AND hypertension

SELECT
    f.procedure_fact_key,
    f.procedure_date_key,
    f.billed_amount
FROM
    procedure_fact f
WHERE
    f.diagnosis_group_key IN (
        -- Groups containing diabetes
        SELECT b1.diagnosis_group_key
        FROM diagnosis_group_bridge b1
        JOIN diagnosis_dim d1 ON b1.diagnosis_key = d1.diagnosis_key
        WHERE d1.ICD_code LIKE 'E11%'  -- Diabetes Type 2
    )
    AND f.diagnosis_group_key IN (
        -- Groups containing hypertension
        SELECT b2.diagnosis_group_key
        FROM diagnosis_group_bridge b2
        JOIN diagnosis_dim d2 ON b2.diagnosis_key = d2.diagnosis_key
        WHERE d2.ICD_code LIKE 'I10%'  -- Essential hypertension
    );
```

**Use case:** Analyze procedures for specific diagnosis combinations (comorbidities).

### Pattern 6: Exclude Specific Diagnosis

```sql
-- Procedures WITHOUT cancer diagnosis

SELECT
    f.procedure_fact_key,
    f.billed_amount
FROM
    procedure_fact f
WHERE
    f.diagnosis_group_key NOT IN (
        SELECT b.diagnosis_group_key
        FROM diagnosis_group_bridge b
        JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
        WHERE d.diagnosis_category = 'Cancer'
    );
```

---

## Bridge Table Performance Optimization

### Index Strategy

**Diagnosis Group Bridge Table Indexes:**

```sql
-- Composite index for FK from fact table
CREATE INDEX idx_bridge_group ON diagnosis_group_bridge(diagnosis_group_key);

-- Composite index for FK to dimension
CREATE INDEX idx_bridge_diagnosis ON diagnosis_group_bridge(diagnosis_key);

-- Covering index for common queries
CREATE INDEX idx_bridge_cover ON diagnosis_group_bridge(diagnosis_group_key, diagnosis_key, weighting_factor);
```

**Query Optimizer Benefits:**
- Fast lookups from fact table diagnosis_group_key
- Fast lookups to dimension diagnosis_key
- Covering index avoids table access for weight-based queries

### Materialized Bridge Groups

For frequently queried diagnosis combinations, consider materializing:

```sql
CREATE TABLE frequent_diagnosis_combos AS
SELECT
    b.diagnosis_group_key,
    LISTAGG(d.diagnosis_description, '; ') AS diagnosis_list,
    COUNT(*) AS diagnosis_count
FROM
    diagnosis_group_bridge b
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key
GROUP BY
    b.diagnosis_group_key;
```

**Use in queries:**

```sql
SELECT
    fdc.diagnosis_list,
    SUM(f.billed_amount) AS total_billed
FROM
    procedure_fact f
    JOIN frequent_diagnosis_combos fdc ON f.diagnosis_group_key = fdc.diagnosis_group_key
GROUP BY
    fdc.diagnosis_list;
```

**Benefits:**
- Avoids bridge table join for simple listing queries
- Faster BI tool label generation

**Maintain via ETL:** Refresh when bridge table updated.

### Bridge Table Partitioning

For very large bridge tables:

```sql
-- Partition by diagnosis_group_key range
CREATE TABLE diagnosis_group_bridge (
    diagnosis_group_key INT,
    diagnosis_key INT,
    ...
)
PARTITION BY RANGE (diagnosis_group_key) (
    PARTITION p1 VALUES LESS THAN (1000000),
    PARTITION p2 VALUES LESS THAN (2000000),
    PARTITION p3 VALUES LESS THAN (3000000),
    ...
);
```

**Benefit:** Partition pruning when queries filter on diagnosis_group_key.

---

## BI Tool Considerations

### Reporting Application Challenges

**Challenge 1: SQL Complexity**

Business users struggle with bridge table SQL. BI tools must abstract complexity.

**Solution:** Create BI semantic layer views:

```sql
-- View: Procedure with all diagnoses (flattened)
CREATE VIEW vw_procedure_diagnoses AS
SELECT
    f.procedure_fact_key,
    f.patient_key,
    f.procedure_date_key,
    f.billed_amount,
    d.diagnosis_key,
    d.diagnosis_description,
    d.diagnosis_category
FROM
    procedure_fact f
    JOIN diagnosis_group_bridge b ON f.diagnosis_group_key = b.diagnosis_group_key
    JOIN diagnosis_dim d ON b.diagnosis_key = d.diagnosis_key;
```

Business users query view directly, BI tool handles grouping.

**Challenge 2: Over-Counting**

Users forget that summing billed_amount across diagnoses over-counts.

**Solutions:**

1. **Documentation:** Clearly label impact metrics as "procedures involving diagnosis X"
2. **BI Tool Calculation:** Create calculated measure that divides by diagnosis count
   ```
   Allocated_Billed_Amount = SUM(billed_amount) / COUNT(DISTINCT diagnosis_key)
   ```
3. **Aggregate Tables:** Pre-calculate allocated or weighted amounts

### OLAP Cube Handling

**Multidimensional databases more tolerant** of multivalued dimensions.

**Approach:** Create dimension with "Diagnosis 1", "Diagnosis 2", ... members

```
Diagnosis Dimension in Cube:
- Diagnosis 1
- Diagnosis 2
- Diagnosis 3
- ...
- All Diagnoses (calculated member = sum of all)
```

**Cube automatically:**
- Handles sparsity (minimizes storage overhead)
- Provides flexible slicing by any diagnosis
- Supports "All Diagnoses" aggregate without over-counting

**Limitation:** Fixed maximum number of diagnosis slots in cube design.

---

## Common Mistakes and Solutions

### Mistake 1: Creating Unique Group Per Fact Row

**Problem:** Massive, unmanageable bridge table with duplicate groups.

**Solution:** Implement group reuse logic in ETL (portfolio approach).

### Mistake 2: Forcing Weighting Factors When Inappropriate

**Problem:** Arbitrary weights (equal allocation, guesses) undermine credibility.

**Solution:** Omit weighting factors; use impact analysis instead.

### Mistake 3: Not Using Surrogate Keys

**Problem:** Using composite natural keys for diagnosis_group_key.

**Solution:** Use surrogate key (integer sequence) for diagnosis_group_key.

### Mistake 4: Forgetting Composite Primary Key

**Problem:** Only indexing diagnosis_group_key, not (diagnosis_group_key, diagnosis_key).

**Solution:** Define composite PK or unique constraint on both columns.

### Mistake 5: Overusing Bridge Tables

**Problem:** Creating bridge table for every dimension relationship.

**Solution:** Use bridge tables ONLY for truly multivalued dimensions, not fixed roles.

### Mistake 6: Not Explaining Over-Counting to Users

**Problem:** Business users surprised by inflated totals when summing across diagnoses.

**Solution:** Education, documentation, calculated measures that adjust for over-counting.

---

## ETL Bridge Table Administration

### Group Creation and Maintenance

**Initial Load:**

```python
def initial_load_diagnosis_groups():
    # Extract all unique diagnosis sets from source
    unique_diagnosis_sets = extract_unique_diagnosis_combinations()

    for diagnosis_set in unique_diagnosis_sets:
        sorted_diagnoses = sorted(diagnosis_set)
        group_key = generate_surrogate_key()

        # Insert into diagnosis group dimension (if using)
        insert_diagnosis_group_dimension(group_key, len(sorted_diagnoses))

        # Insert bridge rows
        for diagnosis_key in sorted_diagnoses:
            insert_bridge_row(group_key, diagnosis_key)
```

**Incremental Updates:**

```python
def assign_diagnosis_group_incremental(diagnosis_keys):
    sorted_keys = sorted(diagnosis_keys)

    # Lookup existing group
    existing_group = lookup_group_by_diagnosis_set(sorted_keys)

    if existing_group:
        return existing_group.group_key
    else:
        # Create new group
        new_group_key = generate_surrogate_key()
        for diag_key in sorted_keys:
            insert_bridge_row(new_group_key, diag_key)
        return new_group_key
```

### Group Lookup Hash Table

For performance, maintain hash table of diagnosis sets:

```python
# In-memory hash table (or database table)
diagnosis_group_hash = {
    hash([101, 205, 387]): 1,  # Group 1
    hash([101, 387]): 2,       # Group 2
    hash([205, 310, 420]): 3,  # Group 3
    ...
}

def lookup_group_by_diagnosis_set(sorted_diagnosis_keys):
    set_hash = hash(tuple(sorted_diagnosis_keys))
    return diagnosis_group_hash.get(set_hash)
```

### Orphaned Group Cleanup

Periodically remove unused groups:

```sql
-- Find diagnosis groups not referenced by any fact table rows
DELETE FROM diagnosis_group_bridge
WHERE diagnosis_group_key IN (
    SELECT b.diagnosis_group_key
    FROM diagnosis_group_bridge b
    LEFT JOIN procedure_fact f ON b.diagnosis_group_key = f.diagnosis_group_key
    WHERE f.diagnosis_group_key IS NULL
);
```

**Frequency:** Monthly or quarterly during maintenance window.

---

## Summary

**Bridge tables handle multivalued dimensions elegantly:**

✅ **Flexible:** Supports any number of values per fact event
✅ **Efficient:** Avoids NULL-filled fact table columns
✅ **Reusable:** Portfolio approach reduces bridge table size
✅ **Queryable:** Standard SQL joins to multivalued dimension

**Weighting factors are optional:**
- Use when clear allocation rules exist
- Omit when allocation is arbitrary or no business agreement
- Primary indicator is simpler alternative

**Common applications:**
- Healthcare: Patient diagnoses, physician teams
- Education: Course instructors, research investigators
- Retail: Promotional offers, product bundles
- Many-to-many relationships requiring dimensional analysis

**See Chapter 19 ETL Subsystems for:**
- Detailed bridge table ETL processing
- Group creation and maintenance patterns
- Performance optimization techniques
