---
name: four-step-dimensional-modeling
description: Apply the systematic four-step dimensional modeling process to design robust, user-friendly fact and dimension tables at atomic grain for any business process. Use when translating business requirements into dimensional models for each row of the enterprise bus matrix.
---

# Four-Step Dimensional Modeling Process

## When to Use This Skill

Use this skill when designing dimensional models for each business process in your data warehouse. This systematic process ensures you create dimensionally sound, atomic-grained models that are:
- User-friendly and intuitive for business analysts
- Flexible enough to answer unpredictable questions
- Consistent across the enterprise
- Based on both business requirements and data realities

**Lifecycle Stage**: Dimensional Modeling phase
**Prerequisites**: Completed business requirements gathering and enterprise bus matrix design
**Input**: One row from the bus matrix (one business process)
**Output**: High-level bubble chart and detailed dimension/fact table specifications

---

## The Four-Step Process

The dimensional modeling process follows four critical steps in sequence. These steps must be completed in order, though you may need to revisit earlier steps as you learn more.

### Step 1: Select the Business Process

**Objective**: Identify which business process to model by combining business requirements with source data availability.

**What is a Business Process?**
A business process is a low-level operational activity performed by an organization, such as:
- Taking orders
- Invoicing customers
- Receiving payments
- Handling service calls
- Processing insurance claims
- Registering students
- Making retail sales

**Key Characteristics**:
- Expressed as **action verbs** (not departments or functions)
- Supported by an operational system
- Generates or captures **key performance metrics**
- Triggered by an input, results in output metrics
- Often part of a series where one process's outputs become another's inputs

**Important Distinctions**:
- Business processes ≠ Business departments (avoid departmental silos)
- Business processes ≠ Strategic initiatives (break initiatives down into processes)
- One business process = One row on the bus matrix = One or more related fact tables

**Critical Decision**:
For your first project, select the business process that is both:
1. **Most critical** to business users
2. **Most feasible** (considering data availability, quality, and organizational readiness)

**Example**: In a retail grocery chain, management wants to understand consumer purchases captured by the POS system. The business process is: **POS retail sales transactions**.

---

### Step 2: Declare the Grain

**Objective**: Specify exactly what a single fact table row represents.

**Why This Step is Critical**:
- The grain is the **foundation** of your entire design
- It determines which dimensions and facts are permissible
- Unclear grain = circular discussions + rogue facts + flawed design
- This is the **most common design error** in dimensional modeling

**Golden Rule**: Start with atomic grain data—the most detailed level captured by the business process.

**Why Atomic Grain?**:
1. **Maximum dimensionality**: The more detailed the data, the more dimensions you can support
2. **Maximum flexibility**: Users can roll up atomic data in any way; you cannot create details from summaries
3. **Unpredictable queries**: Users will slice data in ways you cannot anticipate
4. **Future-proof**: Atomic data accommodates new requirements without redesign

**Grain Declaration Format**:
Express the grain in **business terms**, not as a list of dimensions:

**Good Examples**:
- "One row per scan of an individual product on a customer's sales transaction"
- "One row per line item on a doctor's bill"
- "One row per individual boarding pass scanned at an airport gate"
- "One row per daily snapshot of inventory levels for each item in a warehouse"
- "One row per bank account each month"

**Bad Example**: "One row per date key, product key, store key, promotion key..."

**Common Mistake**: Skipping this step or declaring grain too late in the process

**Process**:
1. Consider business requirements (what users need to analyze)
2. Consider source data realities (what the operational system captures)
3. Write a clear grain statement in business language
4. Get team consensus on the grain
5. Be prepared to revisit if steps 3 or 4 reveal issues

---

### Step 3: Identify the Dimensions

**Objective**: Determine which dimensions provide context to the measurement events captured in the fact table.

**Guiding Question**: "How do business people describe the data resulting from the business process measurement events?"

**The "W" Questions**:
Dimensions typically answer: Who, What, Where, When, Why, and How

**Grain-Driven Approach**:
With a clear grain statement, dimensions usually fall out naturally:
- If the grain is clear, dimensions are straightforward to identify
- Dimensions take on **single values** in the context of each measurement
- If a proposed dimension creates additional fact rows, it violates the grain

**Test for Validity**:
A dimension is valid if it:
1. Takes on only ONE value per fact table row (given the declared grain)
2. Provides descriptive context that business users need for analysis
3. Does not force you to change the grain statement

**Example** (Retail POS):
For grain: "One row per product on a POS transaction"

Primary dimensions (from grain):
- Date (when the sale occurred)
- Product (what was sold)
- Store (where the sale occurred)

Additional dimensions (single-valued at this grain):
- Promotion (under what promotion conditions)
- Cashier (who handled the transaction)
- Payment Method (how customer paid)
- POS Transaction Number (degenerate dimension)

**Critical Principle**:
> A careful grain statement determines the primary dimensionality. You add more dimensions only if they take on a single value under each combination of the primary dimensions.

**Process**:
1. Start with dimensions that fall directly from the grain statement
2. Ask: "What other single-valued descriptors apply at this grain?"
3. Challenge each proposed dimension: "Does this cause additional rows?"
4. If a dimension violates grain, either reject it or revisit the grain
5. Don't get lost in dimension details yet—complete step 4 first

---

### Step 4: Identify the Facts

**Objective**: Determine which numeric measurements will populate the fact table.

**Guiding Question**: "What is the process measuring?"

**Grain Anchor**:
Facts must be **true to the grain**—every fact must make sense at the declared granularity.

**Fact Categories**:

**1. Additive Facts** (Most valuable):
- Can be summed across **all** dimensions
- Examples: Sales quantity, extended sales amount, extended cost amount
- Most flexible for analysis

**2. Semi-Additive Facts**:
- Can be summed across **some** dimensions but not all
- Example: Account balance (additive across accounts, NOT additive across time)
- Typically balance or level metrics

**3. Non-Additive Facts**:
- Cannot be summed along any dimension
- Examples: Unit price, percentages, ratios
- For ratios: Store the **numerator and denominator** as separate additive facts
- Calculate the ratio in the BI tool (ratio of sums, not sum of ratios)

**Derived Facts**:
- Calculated from other facts in the same row
- Example: Gross profit = Extended sales amount - Extended cost amount
- **Recommendation**: Store derived facts physically for:
  - Consistency across all users and tools
  - Performance (pre-calculated)
  - Reducing user error
- Can be stored in a view if all access is controlled through the view

**Facts to Avoid**:
- Facts that don't align with the grain
- Manager's salary in a retail sales fact table
- Promotional cost in individual POS transactions (wrong grain—belongs in promotion fact table)

**Example** (Retail POS):
For grain: "One row per product on a POS transaction"

**Additive facts**:
- Sales quantity
- Extended discount dollar amount
- Extended sales dollar amount
- Extended cost dollar amount
- Gross profit amount (derived, but additive)

**Non-additive facts**:
- Unit regular price
- Unit discount price
- Unit net price
- Gross margin percentage (calculated from gross profit / sales amount)

**Process**:
1. List measurements captured by the source system
2. Verify each fact is true to the grain
3. Categorize facts (additive, semi-additive, non-additive)
4. Decide which derived facts to store physically
5. Identify numerator/denominator pairs for non-additive ratios
6. Challenge any fact that doesn't fit the grain—may indicate grain error

---

## Collaborative Modeling Workshops

Dimensional models should **never** be designed in isolation. The process requires active collaboration between:

**Core Team Members**:
- **Data modeler** (facilitator, owns deliverables)
- **Business representatives** (subject matter experts, analysts who use source data)
- **Source system experts** (understand data realities, transformations)
- **Data governance representatives** (especially for conformed dimensions)

**Optional Participants**:
- Physical DBA
- ETL team members
- Business data stewards

**Why Collaboration Matters**:
- Business reps understand requirements and success criteria
- Source system experts know data idiosyncrasies
- Cross-functional input creates richer, more complete models
- Consensus building ensures organizational buy-in
- Reduces rework due to misunderstandings

**Workshop Approach**:
- Schedule 2-3 hour sessions, 3-4 days per week
- Avoid full-day sessions (participants have other responsibilities)
- Allow time between sessions for:
  - Data profiling and research
  - Updating documentation
  - Resolving open issues
  - Confirming requirements

**Typical Timeline**: 3-4 weeks for a single business process dimensional model

---

## High-Level Bubble Chart

Before diving into details, create a **high-level dimensional model diagram**.

**Purpose**:
- Graphically represents dimension and fact tables for the business process
- Clearly identifies the grain and associated dimensions
- Communicates scope to non-technical audiences
- Ensures team consensus before detailed design

**Contents**:
- Fact table (center box with grain statement)
- Dimension tables (surrounding boxes)
- Relationships (lines connecting dimensions to fact)

**Bubble Chart Best Practices**:
- Start from the bus matrix row
- Express grain in business terms on the diagram
- Arrange dimensions consistently across related bubble charts
- Use one page per fact table
- Keep it simple and visual

**When to Create**:
After completing steps 1 and 2 (business process selection and grain declaration) and having initial thoughts on dimensions (step 3).

**Example Format**:
```
           [Date Dimension]
                  |
    [Product]----[FACT TABLE]----[Store]
                  |
           [Promotion Dimension]

Grain: One row per product on a POS transaction
```

---

## Detailed Dimensional Design

After consensus on the bubble chart, drill into table and column details.

**Sequence**:
1. Start with **dimension tables** (easier, build team momentum)
2. Begin with simple dimensions like Date
3. Move to more complex dimensions
4. End with **fact tables**

**For Each Dimension Table**:
- Attribute name and description
- Sample values and domain
- Source system and source column
- Transformation rules
- Data quality concerns
- Slowly changing dimension (SCD) type per attribute
- Hierarchies and relationships

**For Each Fact Table**:
- Fact name and description
- Additivity (additive, semi-additive, non-additive)
- Source calculation or derivation
- Grain confirmation
- Units of measure
- Null handling

**Documentation Tool**:
- Start with **spreadsheets** (fast, flexible during iteration)
- Move to **modeling tools** when design firms up
- Use data profiling tools throughout to verify source data

**Key Deliverables**:
- High-level bubble chart diagram
- Detailed dimension table worksheets
- Detailed fact table worksheets
- Issues log (definitions, transformation rules, data quality challenges)
- Updated bus matrix (if new dimensions or fact tables discovered)

---

## Design Validation Principles

Throughout the four-step process, validate your design against these principles:

**Grain Principles**:
- ✓ Grain statement is clear and expressed in business terms
- ✓ All team members agree on the grain
- ✓ Grain is atomic (lowest level captured by source)
- ✓ Each fact table has exactly one grain
- ✓ Different grains are in separate fact tables

**Dimension Principles**:
- ✓ Each dimension takes on single value given the grain
- ✓ Dimensions answer who, what, where, when, why, how
- ✓ Dimension tables are denormalized (flat)
- ✓ Descriptive attributes are text (not codes)
- ✓ No nulls in dimension foreign keys (use "Unknown" or "Not Applicable" rows)

**Fact Principles**:
- ✓ All facts are true to the grain
- ✓ Facts are numeric and (mostly) additive
- ✓ Grain is not violated by any fact
- ✓ Non-additive facts store numerator and denominator
- ✓ No nulls in fact table foreign keys

**Integration Principles**:
- ✓ Model aligns with bus matrix
- ✓ Conformed dimensions are used where appropriate
- ✓ Fact table connects to enterprise architecture

---

## Common Pitfalls to Avoid

**Grain-Related**:
- ❌ Skipping grain declaration
- ❌ Declaring grain as list of dimension keys instead of business terms
- ❌ Mixing multiple grains in one fact table
- ❌ Starting with summary grain instead of atomic
- ❌ Not revisiting grain when dimensions/facts don't fit

**Dimension-Related**:
- ❌ Normalizing (snowflaking) dimension tables
- ❌ Using natural keys instead of surrogate keys
- ❌ Allowing null foreign keys in fact table
- ❌ Storing codes instead of decoded text values
- ❌ Including dimensions that violate grain

**Fact-Related**:
- ❌ Including facts at different grains in same table
- ❌ Storing only derived facts without components
- ❌ Using non-additive facts without storing numerators/denominators
- ❌ Including descriptive attributes in fact table

**Process-Related**:
- ❌ Designing in isolation without business input
- ❌ Being data-driven only (ignoring requirements)
- ❌ Being requirement-driven only (ignoring data realities)
- ❌ Not documenting the model thoroughly

---

## Integration with Bus Matrix

The four-step process translates **one row** of the bus matrix into dimensional models:

**Before Four-Step Process**:
- Bus matrix shows: Business processes (rows) × Conformed dimensions (columns)
- One marked cell indicates this dimension applies to this process

**After Four-Step Process**:
- High-level bubble chart: Visual representation of that bus matrix row
- Detailed model: Full specification of dimensions and facts
- Updated matrix: May reveal new dimensions or fact tables not originally anticipated

**Conformed Dimension Considerations**:
- When bus matrix indicates a conformed dimension, you must use the enterprise-standard version
- Work with data stewards to ensure consistency
- May need governance committee decisions on naming and definitions

---

## Estimation and Sizing

After completing the four-step process, estimate fact table size:

**Approach 1**: Ask source system experts about transaction volume

**Approach 2**: Calculate from business metrics
```
Annual rows = Annual revenue / Average item price
```

**Example**:
- Annual revenue: $4 billion
- Average item price: $2.00
- Estimated rows: 2 billion per year

**Why Size Matters**:
- Validates design reasonableness
- Informs physical design decisions
- Guides performance optimization strategies
- Helps plan infrastructure capacity

---

## Review and Validation

After detailed design, conduct reviews with:

**1. IT Review** (DBAs, source system experts, ETL team):
- Verify source data availability
- Validate transformation feasibility
- Confirm grain and technical details
- Provide dimensional modeling education if needed

**2. Core User Review** (analytical business users):
- Validate business definitions
- Confirm model meets requirements
- Test against sample questions
- Verify attribute names and values make sense

**3. Broader Business User Review**:
- Educate on how to use the model
- Demonstrate query capabilities
- Show how model answers business questions
- Build enthusiasm and buy-in

**Review Materials**:
- Bus matrix (enterprise context)
- High-level bubble charts (entity-level overview)
- Dimension hierarchy diagrams (drill paths)
- Sample queries and results
- Critical dimension/fact details

---

## Process Flow Summary

```
1. SELECT BUSINESS PROCESS
   ↓ (from bus matrix row)

2. DECLARE THE GRAIN
   ↓ (in business terms, atomic level)

3. IDENTIFY DIMENSIONS
   ↓ (who, what, where, when, why, how)

4. IDENTIFY FACTS
   ↓ (measurements true to grain)

→ CREATE BUBBLE CHART
→ DEVELOP DETAILED DESIGN
→ REVIEW AND VALIDATE
→ FINALIZE DOCUMENTATION
```

**Iterative Nature**:
- This appears linear but is highly iterative
- Step 3 may cause you to revisit step 2
- Step 4 may reveal grain issues
- Always return to earlier steps when needed
- Multiple passes create increasingly detailed and robust design

---

## Key Success Factors

1. **Always start with atomic grain** – Resist temptation to summarize
2. **Get the grain right first** – Everything else depends on it
3. **Involve business users** – Collaborative design is essential
4. **Express grain in business terms** – Not as a list of keys
5. **One fact table grain** – Never mix grains
6. **Dimensions are single-valued** – At the declared grain
7. **Facts are true to grain** – No exceptions
8. **Document thoroughly** – Clear specifications enable successful ETL
9. **Iterate as needed** – Be willing to revisit earlier steps
10. **Validate constantly** – Against requirements and data realities

---

## Next Steps After Four-Step Process

Once the dimensional model is designed:

1. **Physical Design**: Translate logical model to physical database (SKILL 11)
2. **Dimension Table Detail**: Apply advanced patterns like SCDs (SKILL 4)
3. **Fact Table Detail**: Apply patterns for different fact table types (SKILL 5)
4. **ETL Design**: Plan how to populate the model (SKILL 8)
5. **BI Application Design**: Build reports and dashboards (SKILL 12)

The four-step process provides the **foundation**. The detailed design patterns provide the **refinement**.

---

## Quick Reference Card

| Step | Key Question | Output | Common Mistake |
|------|-------------|--------|----------------|
| 1. Business Process | What operational activity? | Process name from bus matrix | Choosing department vs. process |
| 2. Declare Grain | What does one row represent? | Grain statement in business terms | Listing dimension keys instead |
| 3. Dimensions | Who/what/where/when/why/how? | List of dimension tables | Adding multi-valued dimensions |
| 4. Facts | What are we measuring? | List of facts with additivity | Including facts at wrong grain |

**Remember**: Grain is the foundation. Get it right, and everything else falls into place.
