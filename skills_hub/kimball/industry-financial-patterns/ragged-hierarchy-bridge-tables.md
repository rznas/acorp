# Ragged Hierarchy Bridge Tables

## Complete Guide to Modeling Variable Depth Organizational Hierarchies

Financial organizations often have **ragged hierarchies of indeterminate depth** for cost centers, organizational units, and budget structures. Unlike fixed-depth hierarchies (product → brand → category), ragged hierarchies have varying levels that cannot be predetermined.

---

## The Problem: Fixed Position Hierarchies Don't Work

**Example Organization Tree:**
```
Enterprise (Level 1)
├── Division A (Level 2)
│   ├── Dept 1 (Level 3)
│   │   ├── Cost Center A (Level 4)
│   │   └── Cost Center B (Level 4)
│   └── Dept 2 (Level 3)
├── Division B (Level 2)
│   └── Cost Center C (Level 3)  ← Only 3 levels!
└── Cost Center D (Level 2)  ← Only 2 levels!
```

**Why Fixed Position Fails:**
- Number of levels varies (2 to 4 in this example)
- Cannot create meaningful attribute names (Level-1, Level-2 are meaningless to users)
- Business reorganizations constantly change depth
- Some nodes skip levels entirely

---

## Solution: Organization Map Bridge Table

### Bridge Table Structure

**Grain:** One row for each parent-to-child path in the organizational tree

| parent_org_key | child_org_key | ownership_pct | highest_parent_flag | lowest_child_flag | levels_from_parent |
|----------------|---------------|---------------|---------------------|-------------------|--------------------|
| 1              | 1             | 1.00          | Y                   | N                 | 0                  |
| 1              | 2             | 1.00          | Y                   | N                 | 1                  |
| 1              | 3             | 1.00          | Y                   | N                 | 2                  |
| 1              | 4             | 1.00          | Y                   | Y                 | 2                  |
| 1              | 5             | 1.00          | Y                   | Y                 | 3                  |
| 2              | 2             | 1.00          | Y                   | N                 | 0                  |
| 2              | 3             | 1.00          | Y                   | N                 | 1                  |
| 2              | 4             | 1.00          | Y                   | Y                 | 1                  |
| 3              | 3             | 1.00          | N                   | N                 | 0                  |
| 3              | 5             | 1.00          | N                   | Y                 | 1                  |
| 4              | 4             | 1.00          | N                   | Y                 | 0                  |
| 5              | 5             | 1.00          | N                   | Y                 | 0                  |

**Key Attributes:**
- `parent_org_key`: Starting point of path (FK to organization dimension)
- `child_org_key`: Ending point of path (FK to organization dimension)
- `ownership_pct`: Weighting factor (1.00 for full ownership, < 1.00 for shared)
- `highest_parent_flag`: 'Y' if path starts at top of tree
- `lowest_child_flag`: 'Y' if path ends at leaf node (no children)
- `levels_from_parent`: Distance in hierarchy (optional, useful for reporting)

**Important:** Include self-referencing rows (parent_org_key = child_org_key) for every node.

---

## Query Patterns

### Pattern 1: Sum All Children Under a Parent

**Business Question:** "What's the total budget for Division A and everything under it?"

```sql
-- Get entire subtree under Organization #2 (Division A)
SELECT
  d.org_name,
  SUM(f.budget_amount) as total_budget
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization d ON b.parent_org_key = d.org_key
WHERE d.org_key = 2  -- Division A
  AND d.org_key = 2  -- Must constrain dimension to single parent
GROUP BY d.org_name;
```

**How It Works:**
- Joining dimension (parent_org_key) to fact (via child_org_key) traverses tree
- Constraining dimension to org_key = 2 selects all paths starting at Division A
- Single query fetches all descendants without recursion

### Pattern 2: Leaf Nodes Only

**Business Question:** "What's the budget for the lowest-level cost centers under Division A?"

```sql
SELECT
  c.org_name as cost_center,
  SUM(f.budget_amount) as budget
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization d ON b.parent_org_key = d.org_key
JOIN organization c ON b.child_org_key = c.org_key
WHERE d.org_key = 2  -- Division A
  AND b.lowest_child_flag = 'Y'  -- Leaf nodes only
GROUP BY c.org_name;
```

### Pattern 3: Top-Level Summary

**Business Question:** "Show me budget by top-level divisions only."

```sql
SELECT
  p.org_name as division,
  SUM(f.budget_amount) as total_budget
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization p ON b.parent_org_key = p.org_key
WHERE b.highest_parent_flag = 'Y'  -- Paths from top of tree
  AND b.levels_from_parent = 1     -- Only immediate children
GROUP BY p.org_name;
```

### Pattern 4: Avoiding Overcounting with Multiple Constraints

**Warning:** Constraining on attribute values (not single org_key) can cause overcounting.

**Incorrect Query (Overcounts):**
```sql
-- WRONG: California includes both Los Angeles and San Francisco divisions
SELECT SUM(budget_amount)
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization d ON b.parent_org_key = d.org_key
WHERE d.state = 'California';
-- Counts children multiple times if multiple CA parents exist
```

**Correct Query:**
```sql
SELECT SUM(f.budget_amount)
FROM budget_fact f
WHERE f.org_key IN (
  SELECT DISTINCT b.child_org_key
  FROM organization d
  JOIN org_bridge b ON d.org_key = b.parent_org_key
  WHERE d.state = 'California'
);
```

---

## Shared Ownership / Partial Allocation

**Business Scenario:** Cost Center 5 is 60% owned by Division A, 40% owned by Division B.

**Extended Bridge Table:**
| parent_org_key | child_org_key | ownership_pct | highest_parent_flag | lowest_child_flag |
|----------------|---------------|---------------|---------------------|-------------------|
| 1              | 5             | 1.00          | Y                   | Y                 |
| 2              | 5             | 0.60          | Y                   | Y                 |
| 3              | 5             | 0.60          | N                   | Y                 |
| 6              | 5             | 0.40          | Y                   | Y                 |
| 5              | 5             | 1.00          | N                   | Y                 |

**Query with Allocation:**
```sql
-- Budget allocated by ownership percentage
SELECT
  d.org_name as division,
  SUM(f.budget_amount * b.ownership_pct) as allocated_budget
FROM budget_fact f
JOIN org_bridge b ON f.org_key = b.child_org_key
JOIN organization d ON b.parent_org_key = d.org_key
WHERE b.highest_parent_flag = 'Y'
  AND b.levels_from_parent = 1
GROUP BY d.org_name;
```

**Result:**
- Division A sees $60K for Cost Center 5's $100K budget
- Division B sees $40K

---

## Time-Varying Ragged Hierarchies

**Challenge:** Organizations restructure. Cost centers move between divisions over time.

**Solution: Add Effective Date Columns**

| parent_org_key | child_org_key | ownership_pct | begin_effective_date | end_effective_date | highest_parent_flag | lowest_child_flag |
|----------------|---------------|---------------|----------------------|--------------------|---------------------|-------------------|
| 2              | 5             | 1.00          | 2020-01-01           | 2023-12-31         | Y                   | Y                 |
| 6              | 5             | 1.00          | 2024-01-01           | 9999-12-31         | Y                   | Y                 |

**Query Pattern (Point-in-Time):**
```sql
-- Budget structure as of June 2023
SELECT
  d.org_name,
  SUM(f.budget_amount) as budget
FROM budget_fact f
JOIN org_bridge b
  ON f.org_key = b.child_org_key
  AND f.accounting_period BETWEEN b.begin_effective_date AND b.end_effective_date
JOIN organization d ON b.parent_org_key = d.org_key
WHERE d.org_key = 2
  AND f.accounting_period = '2023-06'
  AND '2023-06-30' BETWEEN b.begin_effective_date AND b.end_effective_date
GROUP BY d.org_name;
```

**Critical Rule:** Always constrain to single date/time when using time-varying bridge table. Failing to do so fetches multiple inconsistent paths.

---

## Maintaining the Bridge Table

### Building Bridge Table: Recursive SQL

**Oracle Example (CONNECT BY):**
```sql
INSERT INTO org_bridge (parent_org_key, child_org_key, ownership_pct)
SELECT
  CONNECT_BY_ROOT org_key as parent_org_key,
  org_key as child_org_key,
  1.0 as ownership_pct
FROM organization
CONNECT BY PRIOR org_key = parent_org_key
START WITH parent_org_key IS NULL;  -- Top of tree
```

**SQL Server Example (Recursive CTE):**
```sql
WITH org_paths AS (
  -- Anchor: Start with all nodes pointing to themselves
  SELECT org_key as parent_org_key, org_key as child_org_key, 0 as depth
  FROM organization

  UNION ALL

  -- Recursive: Add parent-to-grandchild paths
  SELECT p.parent_org_key, o.org_key as child_org_key, p.depth + 1
  FROM org_paths p
  JOIN organization o ON p.child_org_key = o.parent_org_key
  WHERE p.depth < 10  -- Safety limit
)
INSERT INTO org_bridge (parent_org_key, child_org_key, ownership_pct)
SELECT parent_org_key, child_org_key, 1.0
FROM org_paths;
```

**PostgreSQL Example (Recursive CTE with ownership):**
```sql
WITH RECURSIVE org_hierarchy AS (
  SELECT
    org_key,
    org_key as ancestor_key,
    1.0 as ownership,
    0 as levels
  FROM organization

  UNION ALL

  SELECT
    o.org_key,
    oh.ancestor_key,
    o.ownership_allocation * oh.ownership as ownership,
    oh.levels + 1
  FROM organization o
  JOIN org_hierarchy oh ON o.parent_org_key = oh.org_key
  WHERE oh.levels < 20
)
INSERT INTO org_bridge
  (child_org_key, parent_org_key, ownership_pct, levels_from_parent)
SELECT org_key, ancestor_key, ownership, levels
FROM org_hierarchy;
```

### Adding Metadata Flags

**After initial bridge creation:**
```sql
-- Mark highest parent paths
UPDATE org_bridge b
SET highest_parent_flag = 'Y'
WHERE parent_org_key IN (
  SELECT org_key FROM organization WHERE parent_org_key IS NULL
);

-- Mark lowest child paths
UPDATE org_bridge b
SET lowest_child_flag = 'Y'
WHERE child_org_key NOT IN (
  SELECT DISTINCT parent_org_key FROM organization WHERE parent_org_key IS NOT NULL
);
```

### Modifying Hierarchy: Moving Nodes

**Business Change:** Move Cost Centers 4, 5, 6 from Division A (org_key=2) to Division C (org_key=9).

**Static Bridge (Current Structure Only):**
```sql
-- 1. Delete old paths into nodes 4, 5, 6
DELETE FROM org_bridge
WHERE child_org_key IN (4, 5, 6)
  AND parent_org_key NOT IN (4, 5, 6);

-- 2. Insert new paths from higher-level parents to moved nodes
INSERT INTO org_bridge (parent_org_key, child_org_key, ownership_pct)
SELECT parent_org_key, 4 as child_org_key, 1.0
FROM org_bridge
WHERE parent_org_key IN (1, 7, 9);  -- Enterprise, Division X, Division C

INSERT INTO org_bridge (parent_org_key, child_org_key, ownership_pct)
SELECT parent_org_key, 5 as child_org_key, 1.0
FROM org_bridge
WHERE parent_org_key IN (1, 7, 9);

INSERT INTO org_bridge (parent_org_key, child_org_key, ownership_pct)
SELECT parent_org_key, 6 as child_org_key, 1.0
FROM org_bridge
WHERE parent_org_key IN (1, 7, 9);
```

**Time-Varying Bridge (Historical Tracking):**
```sql
-- 1. Close out old paths
UPDATE org_bridge
SET end_effective_date = '2024-12-31'
WHERE child_org_key IN (4, 5, 6)
  AND parent_org_key NOT IN (4, 5, 6)
  AND end_effective_date = '9999-12-31';

-- 2. Insert new paths with new effective date
INSERT INTO org_bridge
  (parent_org_key, child_org_key, ownership_pct, begin_effective_date, end_effective_date)
SELECT parent_org_key, 4, 1.0, '2025-01-01', '9999-12-31'
FROM org_bridge
WHERE parent_org_key IN (1, 7, 9)
  AND child_org_key = 9  -- Get all ancestors of new parent
  AND end_effective_date = '9999-12-31';
-- Repeat for child_org_key 5, 6
```

**Advantage:** Only paths directly involved are modified. Other paths untouched. Compare to alternative schemes where entire tree must be relabeled.

---

## Alternative Approaches (Not Recommended)

### Alternative 1: Pathstring Attribute

**Approach:** Store hierarchical path in dimension attribute.

| org_key | org_name  | pathstring |
|---------|-----------|------------|
| 1       | Enterprise| A+         |
| 2       | Division A| AA+        |
| 3       | Dept 1    | AAA+       |
| 4       | CC A      | AAAA.      |
| 5       | CC B      | AAAB.      |

**Query with Wildcards:**
```sql
-- Get Division A and all children
SELECT * FROM organization
WHERE pathstring LIKE 'AA%';

-- Get only leaf nodes
WHERE pathstring LIKE '%.';

-- Get topmost node
WHERE pathstring LIKE '?+';  -- Single char wild card
```

**Disadvantages:**
- Relabeling cascade when node inserted (all siblings to right must shift)
- Hierarchy definition embedded in dimension (cannot swap rollup structures)
- Limited to single hierarchy
- Pathstring length limits depth

### Alternative 2: Modified Preordered Tree Traversal

**Approach:** Number tree sequentially, left to right, top to bottom. Each node has left/right numbers encompassing all children.

| org_key | org_name  | left_num | right_num |
|---------|-----------|----------|-----------|
| 1       | Enterprise| 1        | 26        |
| 2       | Division A| 2        | 15        |
| 3       | Dept 1    | 3        | 8         |
| 4       | CC A      | 4        | 5         |

**Query Pattern:**
```sql
-- All nodes under Division A
SELECT * FROM organization
WHERE left_num BETWEEN 2 AND 15;

-- Leaf nodes (left and right differ by 1)
WHERE right_num = left_num + 1;
```

**Disadvantages:**
- Entire tree must be renumbered when any node added/moved
- Catastrophic maintenance burden
- Cannot support multiple simultaneous hierarchies

---

## Bridge Table Advantages Summary

**Compared to Recursive Pointers or Pathstrings:**

✅ **Flexibility**
- Support multiple rollup structures (swap bridge tables)
- Alternative hierarchies for same organization set

✅ **Shared Ownership**
- Model matrix organizations (cost center reports to multiple divisions)
- Proportional allocation via ownership_pct

✅ **Time Variance**
- Track historical organizational structures
- Answer "What did the org chart look like in Q2 2022?"

✅ **SCD Compatibility**
- Type 2 dimension changes don't ripple through tree
- Only affected paths updated, not entire structure

✅ **Modification Impact**
- Limited impact when moving nodes (only involved paths change)
- Compare to pathstring/preorder schemes requiring full tree relabeling

✅ **Query Performance**
- Single join traverses tree without recursion
- Flags enable leaf-only, top-level, or specific-depth queries

**When to Use Bridge Table:**
- Hierarchies with > 3 levels
- Indeterminate or varying depth
- Frequent reorganizations
- Multiple rollup perspectives required
- Shared ownership / matrix organizations
- Historical structure tracking needed

**When Fixed Position Hierarchy Suffices:**
- Stable 2-3 level hierarchy (product → brand → category)
- Depth never varies
- Reorganizations extremely rare
- Single rollup structure

---

## Bridge Table ETL Considerations

**Full Refresh vs Incremental:**
- Small organizations (< 10K nodes): Full refresh nightly acceptable
- Large organizations: Incremental update only changed subtrees

**Validation Queries:**
```sql
-- Check: Every node has self-referencing row
SELECT org_key
FROM organization
WHERE org_key NOT IN (
  SELECT child_org_key FROM org_bridge WHERE parent_org_key = child_org_key
);

-- Check: Ownership percentages sum to expected totals
SELECT child_org_key, SUM(ownership_pct)
FROM org_bridge
WHERE parent_org_key <> child_org_key
GROUP BY child_org_key
HAVING SUM(ownership_pct) <> 1.0;  -- Should be 1.0 if no shared ownership

-- Check: No orphan nodes (except top parent)
SELECT org_key
FROM organization
WHERE org_key NOT IN (SELECT child_org_key FROM org_bridge)
  AND parent_org_key IS NOT NULL;
```

**Index Recommendations:**
```sql
CREATE INDEX idx_bridge_parent ON org_bridge(parent_org_key, child_org_key);
CREATE INDEX idx_bridge_child ON org_bridge(child_org_key, parent_org_key);
CREATE INDEX idx_bridge_dates ON org_bridge(begin_effective_date, end_effective_date);
```

---

## Real-World Example: Global Bank

**Hierarchy Depth Variation:**
- North America Division → Region → Branch → Cost Center (4 levels)
- Europe Division → Country → Branch (3 levels)
- APAC Division → Country → Region → City → Branch → Cost Center (6 levels)

**Multiple Rollups:**
- Geographic rollup (for regional P&L)
- Product rollup (for product line profitability)
- Customer segment rollup (for CRM)

**Bridge Table Enables:**
- Single query for "total budget across all APAC branches" despite 6-level depth
- Swap geographic_bridge with product_bridge for different view
- Track 2023 reorganization when 15 branches moved from Europe to APAC

**Without Bridge Table:**
- Would need complex recursive CTEs (performance issues)
- Or separate reporting tables for each rollup (maintenance nightmare)
- Historical reorganization tracking nearly impossible
