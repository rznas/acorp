---
name: data-profiling-quality-assessment
description: Understand source data realities through strategic and tactical data profiling, establish data quality frameworks with quality screens (column, structure, business rule), implement error event tracking, and build audit dimensions for continuous quality monitoring. Use before ETL development to assess data sources and establish quality baselines, then continuously during operations to monitor and improve data quality.
---

# Data Profiling and Quality Assessment

## When to Use This Skill

Use this skill in two critical phases:

1. **Pre-ETL Strategic Assessment** (Project Planning): Immediately after identifying a candidate data source during requirements analysis - perform light profiling to make go/no-go decisions before committing resources
2. **Tactical Deep Profiling** (ETL Design): During data modeling and ETL system design - squeeze out data quality problems and establish comprehensive quality screens
3. **Ongoing Operations**: Continuously monitor data quality through error event tracking and audit dimensions during incremental ETL processing

This skill applies throughout the DW/BI lifecycle but is most critical before ETL development begins and during production monitoring.

## Core Principles

**"Seemingly small data quality issues are important indications of broken business processes."** - Michael Hammer

Data profiling is the technical analysis of data to describe its content, consistency, and structure. Quality assessment establishes systematic mechanisms to measure, track, and improve data quality over time.

### The Two Roles of Data Profiling

**Strategic Profiling**: Light assessment for go/no-go decisions
- Performed immediately after identifying candidate data sources
- Determines suitability for inclusion in data warehouse
- Early disqualification earns respect and saves project resources
- Late revelation of unsuitable data can derail entire DW/BI initiative

**Tactical Profiling**: Lengthy effort to identify transformation requirements
- Begins during data modeling process
- Extends into ETL system design process
- Squeezes out problems before ETL development
- Forms requirements for data quality processing

## Step-by-Step Process

### Phase 1: Strategic Data Profiling (Go/No-Go Assessment)

<strategic_assessment>

**Objective**: Determine if the data source supports the business requirements

1. **Identify Data Source Candidate**
   - Confirm source identified during business requirements analysis
   - Obtain access credentials and permissions
   - Document source system contacts

2. **Perform Light Profiling**
   ```sql
   -- Check for critical fields existence
   SELECT COUNT(*) as total_rows FROM source_table;

   -- Verify key fields are populated
   SELECT
     COUNT(*) as total,
     COUNT(key_field) as populated,
     COUNT(*) - COUNT(key_field) as nulls
   FROM source_table;

   -- Check date ranges
   SELECT MIN(date_field), MAX(date_field)
   FROM source_table;
   ```

3. **Assess Critical Suitability Factors**
   - Does the source contain the required data elements?
   - Is the data at the right grain (level of detail)?
   - Are key fields reliably populated (< 5% NULL rate)?
   - Does the date range cover the required history?
   - Are there any show-stopping data quality issues?

4. **Make Go/No-Go Decision**
   - GO: Proceed to tactical profiling
   - NO-GO: Document reasons, inform stakeholders, seek alternatives
   - CONDITIONAL: Negotiate source system improvements before proceeding

</strategic_assessment>

### Phase 2: Tactical Data Profiling (Detailed Analysis)

<tactical_profiling>

**Objective**: Understand transformation requirements and quality challenges in detail

**Column-Level Profiling**:

For EVERY column in source tables:

```sql
-- Numeric columns
SELECT
  MIN(column_name) as min_value,
  MAX(column_name) as max_value,
  AVG(column_name) as avg_value,
  COUNT(DISTINCT column_name) as distinct_count,
  COUNT(*) - COUNT(column_name) as null_count
FROM source_table;

-- Text columns
SELECT
  column_name,
  COUNT(*) as frequency,
  LENGTH(column_name) as string_length
FROM source_table
GROUP BY column_name
ORDER BY frequency DESC
LIMIT 100;

-- Date columns
SELECT
  MIN(date_column) as earliest,
  MAX(date_column) as latest,
  COUNT(DISTINCT date_column) as unique_dates,
  SUM(CASE WHEN date_column IS NULL THEN 1 ELSE 0 END) as nulls,
  SUM(CASE WHEN date_column > CURRENT_DATE THEN 1 ELSE 0 END) as future_dates
FROM source_table;
```

**Relationship Analysis**:

```sql
-- Verify hierarchies (e.g., Product SKU -> Category -> Department)
SELECT
  product_sku,
  COUNT(DISTINCT category) as category_count,
  COUNT(DISTINCT department) as dept_count
FROM product_source
GROUP BY product_sku
HAVING COUNT(DISTINCT category) > 1
   OR COUNT(DISTINCT department) > 1;

-- Check foreign key relationships
SELECT s.*
FROM source_table s
LEFT JOIN reference_table r ON s.ref_key = r.ref_key
WHERE r.ref_key IS NULL;
```

**Profiling Tool Advantages**:
- Invest in data profiling tool rather than hand-coding all queries
- Tools enable exploring many relationships with simple UI gestures
- Much more productive than manual SQL queries
- Can discover patterns and anomalies automatically

**Document Findings**:
- Columns with high NULL rates (> 10%)
- Fields outside expected ranges
- Unexpected domain values
- Broken hierarchical relationships
- Missing foreign key references
- Format inconsistencies
- Character encoding issues

</tactical_profiling>

### Phase 3: Design Data Quality Screens

<quality_screens>

The heart of the ETL data quality architecture is a set of **quality screens** - diagnostic filters in the data flow pipeline.

**How Quality Screens Work**:
- Each screen is a test against the data
- If test succeeds → no action, no side effects
- If test fails → drop error event row to error event schema + choose response

**Three Types of Quality Screens**:

#### 1. Column Screens

Test data within a single column:

```sql
-- NULL value check
CREATE SCREEN check_customer_name
IF customer_name IS NULL THEN
  LOG_ERROR('Missing customer name', table, record_id, column)
  TAG_ROW(audit_dimension, 'Name missing')
END IF

-- Range check
CREATE SCREEN check_discount_range
IF discount_percent < 0 OR discount_percent > 100 THEN
  LOG_ERROR('Discount out of range', value, min, max)
  TAG_ROW(audit_dimension, 'Invalid discount')
END IF

-- Format check
CREATE SCREEN check_ssn_format
IF social_security_num NOT LIKE '___-__-____' THEN
  LOG_ERROR('SSN format violation', value, expected_format)
  TAG_ROW(audit_dimension, 'SSN format error')
END IF
```

Examples:
- Unexpected NULL values
- Values outside prescribed range
- Values not adhering to required format
- Invalid domain values

#### 2. Structure Screens

Test relationships across columns:

```sql
-- Hierarchy validation
CREATE SCREEN check_geography_hierarchy
SELECT city, state, COUNT(DISTINCT country) as country_count
FROM customer_source
GROUP BY city, state
HAVING COUNT(DISTINCT country) > 1;
-- Cities should map to exactly one state/country combination

-- Foreign key check
CREATE SCREEN check_product_reference
SELECT f.product_id
FROM fact_sales f
LEFT JOIN dim_product p ON f.product_id = p.product_id
WHERE p.product_id IS NULL;

-- Postal address validation
CREATE SCREEN check_address_validity
IF NOT valid_postal_address(street, city, state, zip) THEN
  LOG_ERROR('Invalid postal address', full_address)
  TAG_ROW(audit_dimension, 'Address validation failed')
END IF
```

Examples:
- Hierarchy violations (many-to-one relationships broken)
- Primary key/foreign key referential integrity failures
- Invalid postal addresses
- Cross-column validation rules

#### 3. Business Rule Screens

Test complex, domain-specific rules:

```sql
-- Complex time-dependent business rule
CREATE SCREEN check_platinum_status
IF customer_status = 'Platinum' AND (
  membership_years < 5 OR
  lifetime_miles < 2000000
) THEN
  LOG_ERROR('Platinum status violation', customer_id, years, miles)
  TAG_ROW(audit_dimension, 'Invalid platinum status')
END IF

-- Aggregate threshold check
CREATE SCREEN check_mri_anomaly
SELECT diagnosis_code, COUNT(*) as mri_count
FROM medical_orders
WHERE procedure_type = 'MRI'
  AND diagnosis_severity = 'Minor'
GROUP BY diagnosis_code
HAVING COUNT(*) > threshold;
-- Flags statistically improbable patterns
```

Examples:
- Customer profile business rules
- Statistical threshold violations
- Time-dependent validations
- Cross-table business logic

</quality_screens>

### Phase 4: Respond to Quality Events

<quality_responses>

When a quality screen throws an error, choose one of three responses:

**Option 1: Halt the Process** ❌ LEAST PREFERRED
- Requires manual intervention
- Must diagnose problem
- Restart/resume/abort job
- Use only for critical show-stopping errors

**Option 2: Suspense File** ⚠️ PROBLEMATIC
- Sends offending records to holding area
- Unclear when/if records will be fixed and reintroduced
- Database integrity questionable until records restored
- Avoid for minor data transgressions

**Option 3: Tag and Continue** ✅ BEST PRACTICE
- Tag data with error condition
- Allow processing to continue
- Fact table: Tag with audit dimension
- Dimension table: Tag with audit dimension or error values
- Preserves referential integrity
- Enables reporting on data quality

**Tagging Examples**:

Fact table tagging via audit dimension:
```sql
-- Audit dimension row for quality event
INSERT INTO dim_audit VALUES (
  audit_key,
  'Discount out of range',
  data_quality_score,
  etl_version,
  load_date,
  -- ... other audit attributes
);

-- Fact row references audit dimension
INSERT INTO fact_sales VALUES (
  date_key,
  product_key,
  customer_key,
  audit_key,  -- ← References audit dimension with error info
  quantity,
  amount
);
```

Dimension tagging with error values:
```sql
-- Use descriptive error values
customer_name = 'Missing - Record 12345'
customer_category = 'Not Yet Assigned 157'
-- Prevents lumping together in aggregates
```

</quality_responses>

### Phase 5: Implement Error Event Schema

<error_event_schema>

**Purpose**: Record every error event thrown by quality screens across entire ETL pipeline

**Schema Design**:

```
ERROR_EVENT_FACT (Main Table)
- error_event_key (PK, surrogate)
- calendar_date_key (FK to date dimension)
- batch_key (FK to batch dimension)
- screen_key (FK to screen dimension)
- error_date_time (timestamp, exact time)
- error_count (typically 1)

ERROR_EVENT_DETAIL_FACT (Lower Grain Table)
- error_event_key (FK to main table)
- table_name
- record_id
- field_name
- error_condition
- error_detail_date_time (for aggregate threshold errors)
```

**Dimension Tables**:

BATCH_DIMENSION:
- batch_key (PK)
- batch_name
- batch_type ('Daily Load', 'Historic Load', etc.)
- processing_step

SCREEN_DIMENSION:
- screen_key (PK)
- screen_name
- screen_criterion (description of test)
- screen_code_location (where implemented)
- error_response ('Halt', 'Suspend', 'Tag')

**Error Event Fact Table Grain**: One row per error thrown by any quality screen

**Error Event Detail Fact Table Grain**: One row per field in a specific record participating in an error

**Usage**:
- Complex structure/business rule errors → one row in main table, many rows in detail table
- Simple column errors → one row in main table, one row in detail table
- Enables complete reconstruction of multi-field, multi-record errors

**Every Quality Screen Must**:
- Populate these tables when throwing errors
- Include enough detail to diagnose root cause
- Capture context (batch, timestamp, table, record, field)

</error_event_schema>

### Phase 6: Build Audit Dimension

<audit_dimension>

**Purpose**: Attach metadata context to fact table rows at the moment of creation

**Audit Dimension Structure**:

```
DIM_AUDIT
- audit_key (PK, surrogate)
- load_date
- load_date_time
- extract_date_time
- data_quality_score (0-100)
- data_quality_category ('Perfect', 'Good', 'Warning', 'Error')
- error_flag_1 (e.g., 'Discount out of range')
- error_flag_2 (e.g., 'Missing customer address')
- error_flag_3
- ... (add flags for common errors)
- etl_version
- source_system
- source_file_name
- record_count
- sum_of_facts (for validation)
```

**How It Works**:

Scenario 1 - Perfect Load:
- Load runs with no errors flagged
- Generate ONE audit dimension row
- Attach to ALL fact rows loaded today
- All categories, scores, versions identical

Scenario 2 - Load with Errors:
- Some fact rows trigger "discount out of range" error
- Generate TWO audit dimension rows:
  - Row 1: Perfect quality (for clean records)
  - Row 2: Discount error flagged (for problem records)
- Each fact row references appropriate audit dimension row

**Metadata Elevated to Real Data**:
- Enables querying by data quality
- Reports can filter/group by quality scores
- Track quality trends over time
- Identify systematic quality issues

**Attachment to Fact Tables**:

```sql
CREATE TABLE fact_shipments (
  date_key INTEGER,
  product_key INTEGER,
  customer_key INTEGER,
  audit_key INTEGER,  -- ← Audit dimension foreign key
  quantity DECIMAL,
  amount DECIMAL,
  discount DECIMAL
);
```

</audit_dimension>

### Phase 7: Establish Data Quality Culture

<quality_culture>

**The Real Problem**: Technical solutions alone will not fix data quality

**Root Cause**: Data quality issues indicate broken business processes (Michael Hammer)

**Nine-Step Quality Culture Template**:

1. **Declare High-Level Commitment**
   - Executive sponsorship for data quality initiatives
   - Quality metrics included in organizational goals
   - Resources allocated to quality improvement

2. **Drive Process Reengineering**
   - Executive-level analysis of root causes
   - Fix broken business processes, not just symptoms
   - Address why bad data enters systems

3. **Improve Data Entry Environment**
   - Invest in better data entry systems
   - Provide complete context and choices to data entry staff
   - Remove roadblocks that encourage workarounds

4. **Improve Application Integration**
   - Reduce duplicate data entry across systems
   - Establish master data management
   - Create consistent data standards

5. **Change How Processes Work**
   - Reengineer workflows to prevent errors
   - Build validation into business processes
   - Empower data creators to ensure quality

6. **Promote End-to-End Team Awareness**
   - Show data entry staff who uses their data
   - Explain business impact of quality data
   - Create visibility into downstream usage

7. **Promote Interdepartmental Cooperation**
   - Break down data silos
   - Establish data governance across departments
   - Create shared accountability

8. **Publicly Celebrate Excellence**
   - Recognize and reward quality efforts
   - Share success stories
   - Make quality a point of pride

9. **Continuously Measure and Improve**
   - Track quality metrics over time
   - Report quality dashboards to executives
   - Establish continuous improvement programs

**Example**: Large Drugstore Chain Buyers' Assistants
- PROBLEM: Assistants judged on speed (items entered per hour), not quality
- PROBLEM: No awareness of who uses the data
- PROBLEM: Incomplete/unreliable source data from suppliers
- SOLUTION: Invest in better data entry systems with complete context
- SOLUTION: Assure assistants their work affects many decision makers
- SOLUTION: Publicly praise and reward diligent efforts
- SOLUTION: Establish end-to-end awareness and team appreciation

</quality_culture>

## Data Quality Architecture Goals

Establish comprehensive framework supporting:

1. **Early Diagnosis and Triage**
   - Identify quality issues before they enter warehouse
   - Make go/no-go decisions based on evidence
   - Set realistic expectations with business sponsors

2. **Source System Requirements**
   - Document quality issues discovered during profiling
   - Request source system improvements
   - Negotiate better data before extraction

3. **Specific Error Descriptions**
   - Catalog expected errors in ETL processes
   - Document how each error is handled
   - Maintain error handling specifications

4. **Complete Error Capture Framework**
   - Record ALL quality errors via error event schema
   - Measure data quality metrics over time
   - Enable root cause analysis

5. **Quality Confidence Metrics**
   - Attach quality scores to final data via audit dimension
   - Enable filtering/grouping by quality
   - Track quality trends and improvements

## Data Profiling Outputs

### Minimum Profiling Documentation

For each source table, document:

**Numeric Columns**:
- Minimum value
- Maximum value
- Average value
- Count of distinct values
- NULL count and percentage

**Text Columns**:
- Count of distinct values
- Maximum length
- NULL count and percentage
- Top 10-20 most frequent values

**Date Columns**:
- Earliest date
- Latest date
- Count of distinct dates
- NULL count
- Future-dated records count

**Hierarchies**:
- Validation that many-to-one relationships hold
- Documentation of any violations
- Proposed remediation

**Referential Integrity**:
- Foreign key violation counts
- Orphaned record identification
- Missing reference data

### Data Quality Scorecard

Create scorecard for each data source:

| Quality Dimension | Score (0-100) | Issues | Remediation |
|------------------|---------------|--------|-------------|
| Completeness | 85 | 15% NULL customer names | Request source fix |
| Validity | 92 | 8% invalid ZIP codes | Cleansing screen |
| Consistency | 78 | Product hierarchy violations | Manual cleanup |
| Timeliness | 95 | Delayed by 4 hours | Acceptable |
| Accuracy | Unknown | Requires business validation | Schedule review |

**Overall Data Quality Score**: Weighted average across dimensions

## Integration with ETL Subsystems

This skill directly implements these ETL subsystems from Chapter 19:

- **Subsystem 1**: Data Profiling
- **Subsystem 4**: Data Cleansing System (quality screens portion)
- **Subsystem 5**: Error Event Schema
- **Subsystem 6**: Audit Dimension Assembler

Quality screens feed into:
- **Subsystem 4**: Data Cleansing System (transformation portion)
- **Subsystem 7**: Deduplication System
- **Subsystem 8**: Conforming System

## Common Mistakes to Avoid

1. **Skipping Strategic Profiling**
   - Late discovery of unsuitable data derails projects
   - Always profile BEFORE committing resources

2. **Insufficient Tactical Profiling**
   - Underestimating data quality problems
   - Missing project milestones due to unexpected dirty data
   - Build quality machinery UPFRONT based on profiling results

3. **Over-Reliance on Technical Solutions**
   - Technical fixes without process reengineering
   - Example: Forcing SSN format causes clerks to enter fake numbers
   - Address root business process causes

4. **Using Suspense Files**
   - Records stuck in limbo
   - Database integrity questionable
   - Prefer tagging and continuing

5. **Halting on Minor Errors**
   - Too much manual intervention
   - Prefer tagging for non-critical errors

6. **No Quality Measurement Framework**
   - Flying blind without error event schema
   - Can't demonstrate quality improvements
   - Unable to identify systematic issues

7. **Ignoring Data Entry Staff**
   - Quality depends on people creating data
   - Must promote awareness and appreciation
   - Celebrate excellence publicly

## Quick Reference: Data Profiling Workflow

```
1. STRATEGIC PROFILING (Light Assessment)
   ↓
2. GO/NO-GO DECISION
   ↓ (if GO)
3. TACTICAL PROFILING (Detailed Analysis)
   ↓
4. DESIGN QUALITY SCREENS
   - Column screens
   - Structure screens
   - Business rule screens
   ↓
5. IMPLEMENT ERROR EVENT SCHEMA
   - Error event fact table
   - Error event detail fact table
   - Supporting dimensions
   ↓
6. BUILD AUDIT DIMENSION
   - For each fact table
   - Attach quality metadata to data
   ↓
7. ESTABLISH QUALITY CULTURE
   - Executive commitment
   - Process reengineering
   - Continuous measurement
```

## Deliverables

### Pre-ETL Phase
- [ ] Data profiling reports (strategic and tactical)
- [ ] Data quality scorecards per source
- [ ] Go/no-go decision documentation
- [ ] Source system improvement requests

### ETL Design Phase
- [ ] Quality screen specifications
  - [ ] Column screens catalog
  - [ ] Structure screens catalog
  - [ ] Business rule screens catalog
- [ ] Error event schema DDL
- [ ] Audit dimension design per fact table
- [ ] Error handling strategy documentation

### Operational Phase
- [ ] Quality monitoring dashboards
- [ ] Error event trend reports
- [ ] Data quality improvement initiatives
- [ ] Quality culture assessment

## Related Skills

- **SKILL 8: ETL System Architecture Design** - Quality screens are core components
- **SKILL 9: Historic Data Loading** - Quality validation during initial load
- **SKILL 10: Incremental ETL Processing** - Ongoing quality monitoring
- **SKILL 1: Business Requirements Gathering** - Understanding quality requirements
- **SKILL 6: Conformed Dimension Management** - Quality of conformed data critical

---

**Remember**: Data profiling is not optional. Do it upfront, do it thoroughly, and build quality monitoring into every ETL process. Use profiling results to set realistic expectations and protect project timelines.
