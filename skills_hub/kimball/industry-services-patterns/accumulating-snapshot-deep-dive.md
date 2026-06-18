# Accumulating Snapshot Fact Tables - Deep Dive

This reference provides comprehensive guidance on accumulating snapshot fact tables, with detailed examples from education applicant pipelines, healthcare claims billing, and other service industry workflows.

---

## The Three Fact Table Types Recap

### Transaction Fact Tables

**Grain:** One row per transaction event

**Characteristics:**
- Row inserted once, never updated
- Sparse (not all dimension combinations have transactions)
- Most common fact table type
- Date dimension represents when transaction occurred

**Example:** Retail POS sales, phone call detail records, patient procedures

### Periodic Snapshot Fact Tables

**Grain:** One row per period (day, week, month) per entity

**Characteristics:**
- Row inserted at regular intervals (daily, monthly)
- Dense (predictable pattern of rows)
- Semi-additive facts (balances, inventory levels)
- Date dimension represents snapshot period end date

**Example:** Account balances, inventory on hand, course enrollment counts

### Accumulating Snapshot Fact Tables

**Grain:** One row per instance of workflow/pipeline lifecycle

**Characteristics:**
- **Row updated** as workflow progresses through milestones
- Multiple date dimensions (one per milestone)
- Facts represent both milestone metrics AND elapsed durations
- Appropriate for **short-lived processes** with defined beginning, end, standard milestones

**Example:** Applicant pipeline, order fulfillment, claims billing

---

## Accumulating Snapshot Characteristics

### Distinguishing Features

<accumulating_snapshot_features>
**1. Single row = complete workflow history**
- One row tracks entire lifecycle of pipeline instance
- Example: One row per applicant (from inquiry → enrolled/withdrawn)

**2. Multiple date dimensions**
- Each standard milestone has its own date dimension role
- 6-8+ date foreign keys common
- Role-playing date dimension with views for each milestone

**3. Rows are updated**
- **Destructively updated** as instance progresses through pipeline
- Both foreign keys AND facts may change during updates
- Contrast with transaction fact tables (insert once, never update)

**4. Multiple facts**
- **Milestone-specific facts:** Metrics captured at each milestone
- **Status counts:** How many instances in each stage
- **Elapsed durations (lags):** Days/hours between milestones
- **Cumulative totals:** Running sums as workflow progresses

**5. Short-lived processes**
- Workflow has defined beginning and end
- Standard milestones (not infinite variations)
- Completes within predictable timeframe (days to months, not years)

**6. TBD date handling**
- Future milestone dates unknown when row created
- Foreign keys point to "To Be Determined" date dimension row
- Updated to actual dates when milestones occur
</accumulating_snapshot_features>

### When to Use Accumulating Snapshots

**Appropriate for:**

✅ **Pipelines with standard milestones**
- Education: Inquiry → Visit → Apply → Accept → Enroll
- Healthcare: Treatment → Bill primary → Bill secondary → Payment → Zero balance
- Order fulfillment: Order → Pick → Pack → Ship → Deliver

✅ **Short-lived processes**
- Complete within days to months
- Not multi-year processes (use periodic snapshots instead)

✅ **Status tracking requirements**
- Business users need current state of each instance
- Want to see bottlenecks, time in each stage

✅ **Standard workflow analysis**
- Conversion rates between milestones
- Average duration in each stage
- Success/failure patterns

**NOT appropriate for:**

❌ **Long-running processes** (multi-year)
- Customer lifetime from acquisition to churn (use periodic snapshots)
- Employee career progression (too long, too variable)

❌ **Highly variable workflows**
- No standard milestones (every instance follows different path)
- Too many milestone variations to model cleanly

❌ **Simple status tracking**
- If only need current status (not history), use Type 1 SCD in dimension

---

## Education Applicant Pipeline Example

### Business Process

**Track prospective students through admissions milestones:**

Inquiry → Campus Visit → Application → File Complete → Decision → Enrolled/Withdrawn

### Grain Declaration

**One row per prospective student applicant**

This is the lowest level of detail captured when prospect enters pipeline.

### Standard Milestones (Dates)

1. **Initial inquiry date** (first contact with university)
2. **Campus visit date** (attended campus tour or open house)
3. **Application submitted date**
4. **Application file completed date** (all materials received)
5. **Admissions decision notification date** (accept/reject/waitlist)
6. **Enrolled date** (student confirms enrollment) OR **Withdrawn date** (student withdraws)

Each date is a role-playing dimension.

### Dimensional Model

```
Applicant Pipeline Accumulating Snapshot Fact Table

Dimensions:
- Initial_inquiry_date_key (FK to date dimension view "inquiry_date")
- Campus_visit_date_key (FK to date dimension view "visit_date")
- Application_submitted_date_key (FK to date dimension view "submit_date")
- File_completed_date_key (FK to date dimension view "file_complete_date")
- Decision_notification_date_key (FK to date dimension view "decision_date")
- Enrolled_date_key (FK to date dimension view "enroll_date")
- Withdrawn_date_key (FK to date dimension view "withdraw_date")
- Applicant_key (FK to applicant dimension)
- Intended_program_key (FK to academic program dimension)
- Admissions_recruiter_key (FK to employee dimension)
- Decision_status_key (FK to decision status dimension: accepted, rejected, waitlist, pending)
- Geographic_region_key (FK to geography dimension)

Facts:
- Inquiry_to_visit_days (lag)
- Visit_to_application_days (lag)
- Application_to_complete_days (lag)
- Complete_to_decision_days (lag)
- Decision_to_enrollment_days (lag)
- Total_pipeline_days (inquiry to final disposition)
- Estimated_enrollment_probability (0.0 to 1.0)
- Financial_aid_offered_amount
- Application_fee_paid_amount
- [Status counts - see below]
```

### Status Counts Pattern

**Alternative approach:** Include status indicator facts

```
Facts:
- Inquiry_status_count (1 if reached inquiry milestone, else 0)
- Visit_status_count (1 if attended visit, else 0)
- Application_status_count (1 if submitted application, else 0)
- File_complete_status_count (1 if file completed, else 0)
- Decision_status_count (1 if decision made, else 0)
- Enrolled_status_count (1 if enrolled, else 0)
- Withdrawn_status_count (1 if withdrawn, else 0)
```

**Use:** Easily count how many applicants reached each milestone with simple SUM().

### Sample Rows Over Time

**Row created when applicant first inquires (Jan 10):**

| Applicant | Inquiry Date | Visit Date | Submit Date | Complete Date | Decision Date | Enroll Date | Withdraw Date | Inquiry-Visit Days |
|-----------|--------------|------------|-------------|---------------|---------------|-------------|---------------|--------------------|
| A001 | 2024-01-10 | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | NULL |

**Row updated after campus visit (Jan 25):**

| Applicant | Inquiry Date | Visit Date | Submit Date | Complete Date | Decision Date | Enroll Date | Withdraw Date | Inquiry-Visit Days |
|-----------|--------------|------------|-------------|---------------|---------------|-------------|---------------|--------------------|
| A001 | 2024-01-10 | **2024-01-25** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **15** |

**Row updated after application submitted (Feb 05):**

| Applicant | Inquiry Date | Visit Date | Submit Date | Complete Date | Decision Date | Enroll Date | Withdraw Date | Visit-Submit Days |
|-----------|--------------|------------|-------------|---------------|---------------|-------------|---------------|-------------------|
| A001 | 2024-01-10 | 2024-01-25 | **2024-02-05** | **TBD** | **TBD** | **TBD** | **TBD** | **11** |

**Final row after enrolled (May 01):**

| Applicant | Inquiry Date | Visit Date | Submit Date | Complete Date | Decision Date | Enroll Date | Withdraw Date | Total Days |
|-----------|--------------|------------|-------------|---------------|---------------|-------------|---------------|------------|
| A001 | 2024-01-10 | 2024-01-25 | 2024-02-05 | 2024-02-15 | 2024-03-15 | **2024-05-01** | **N/A** | **111** |

**Alternative final row if applicant withdrew:**

| Applicant | Inquiry Date | Visit Date | Submit Date | Complete Date | Decision Date | Enroll Date | Withdraw Date | Total Days |
|-----------|--------------|------------|-------------|---------------|---------------|-------------|---------------|------------|
| A002 | 2024-01-12 | TBD | TBD | **N/A** | **N/A** | **N/A** | **2024-01-20** | **8** |

### TBD Date Dimension Row

**Special row in date dimension:**

```
Date Dimension:
date_key: -1 (or 0, or 99991231)
full_date: NULL
calendar_year: NULL
calendar_month: NULL
date_description: "To Be Determined"
```

**Usage:**
- All future milestone dates initially point to TBD row
- Prevents NULL foreign keys in fact table (violates referential integrity)
- Enables LEFT OUTER JOIN patterns in BI queries

**Alternative TBD rows:**
- **TBD (Not Reached):** Milestone not yet reached
- **N/A (Not Applicable):** Milestone will never occur (e.g., enroll date for withdrawn applicant)

### BI Queries

**Count applicants at each pipeline stage:**

```sql
SELECT
    COUNT(CASE WHEN inquiry_date_key != -1 THEN 1 END) AS inquiry_count,
    COUNT(CASE WHEN visit_date_key != -1 THEN 1 END) AS visit_count,
    COUNT(CASE WHEN application_date_key != -1 THEN 1 END) AS application_count,
    COUNT(CASE WHEN decision_date_key != -1 THEN 1 END) AS decision_count,
    COUNT(CASE WHEN enrolled_date_key != -1 THEN 1 END) AS enrolled_count
FROM
    applicant_pipeline_fact;
```

**Average days between milestones:**

```sql
SELECT
    AVG(inquiry_to_visit_days) AS avg_inquiry_to_visit,
    AVG(visit_to_application_days) AS avg_visit_to_application,
    AVG(application_to_complete_days) AS avg_application_to_complete,
    AVG(complete_to_decision_days) AS avg_complete_to_decision,
    AVG(decision_to_enrollment_days) AS avg_decision_to_enrollment
FROM
    applicant_pipeline_fact
WHERE
    enrolled_date_key != -1;  -- Only enrolled students
```

**Conversion rates (funnel analysis):**

```sql
SELECT
    COUNT(DISTINCT CASE WHEN inquiry_date_key != -1 THEN applicant_key END) AS inquiry_count,
    COUNT(DISTINCT CASE WHEN visit_date_key != -1 THEN applicant_key END) AS visit_count,
    COUNT(DISTINCT CASE WHEN application_date_key != -1 THEN applicant_key END) AS application_count,
    COUNT(DISTINCT CASE WHEN enrolled_date_key != -1 THEN applicant_key END) AS enrolled_count,

    -- Conversion rates
    COUNT(DISTINCT CASE WHEN visit_date_key != -1 THEN applicant_key END) * 100.0 /
        NULLIF(COUNT(DISTINCT CASE WHEN inquiry_date_key != -1 THEN applicant_key END), 0) AS inquiry_to_visit_rate,

    COUNT(DISTINCT CASE WHEN enrolled_date_key != -1 THEN applicant_key END) * 100.0 /
        NULLIF(COUNT(DISTINCT CASE WHEN application_date_key != -1 THEN applicant_key END), 0) AS application_to_enroll_rate
FROM
    applicant_pipeline_fact;
```

**Bottleneck identification:**

```sql
-- Find applicants stuck in pipeline for > 60 days without progress

SELECT
    ap.applicant_name,
    ap.email,
    d1.full_date AS last_milestone_date,
    DATEDIFF(CURRENT_DATE, d1.full_date) AS days_since_last_milestone
FROM
    applicant_pipeline_fact f
    JOIN applicant_dim ap ON f.applicant_key = ap.applicant_key
    JOIN date_dim d1 ON f.file_completed_date_key = d1.date_key
WHERE
    f.decision_date_key = -1  -- Decision not yet made
    AND DATEDIFF(CURRENT_DATE, d1.full_date) > 60
ORDER BY
    days_since_last_milestone DESC;
```

---

## Healthcare Claims Billing Example

### Business Process

**Track medical claims from charge through payment:**

Treatment → Bill Primary Insurance → Bill Secondary Insurance → Bill Responsible Party → Payments → Zero Balance

### Grain Declaration

**One row per line item on medical claim**

Line item represents single procedure/service on claim (claim may have multiple line items).

### Standard Milestones (Dates)

1. **Treatment date** (service provided to patient)
2. **Primary insurance billing date**
3. **Secondary insurance billing date**
4. **Responsible party billing date**
5. **Last primary insurance payment date**
6. **Last secondary insurance payment date**
7. **Last responsible party payment date**
8. **Zero balance date** (claim fully paid or written off)

### Dimensional Model

```
Claims Billing Pipeline Accumulating Snapshot Fact Table

Dimensions:
- Treatment_date_key
- Primary_billing_date_key
- Secondary_billing_date_key
- Responsible_party_billing_date_key
- Primary_payment_date_key
- Secondary_payment_date_key
- Responsible_payment_date_key
- Zero_balance_date_key
- Patient_key
- Responsible_party_key (guarantor)
- Physician_key
- Physician_organization_key
- Procedure_key (HCPCS/CPT code)
- Facility_key
- Diagnosis_group_key (FK to diagnosis bridge table)
- Primary_payer_key (insurance organization)
- Secondary_payer_key (role-playing payer dimension)

Degenerate Dimensions:
- Master_patient_bill_ID

Facts:
- Billed_amount
- Primary_insurance_paid_amount
- Secondary_insurance_paid_amount
- Responsible_party_paid_amount
- Total_paid_amount (calculated)
- Amount_sent_to_collections
- Amount_written_off
- Amount_remaining (calculated)
- Length_of_stay (inpatient)
- Days_to_primary_payment (lag)
- Days_to_secondary_payment (lag)
- Days_to_responsible_payment (lag)
- Days_to_zero_balance (lag)
```

### Lifecycle Example

**Row created when charge received from physician (Jan 15):**

| Treatment Date | Primary Bill Date | Primary Pay Date | Zero Balance Date | Billed | Primary Paid | Remaining |
|----------------|-------------------|------------------|-------------------|--------|--------------|-----------|
| 2024-01-15 | **TBD** | **TBD** | **TBD** | $500 | $0 | $500 |

**Updated after primary insurance billed (Jan 20):**

| Treatment Date | Primary Bill Date | Primary Pay Date | Zero Balance Date | Billed | Primary Paid | Remaining |
|----------------|-------------------|------------------|-------------------|--------|--------------|-----------|
| 2024-01-15 | **2024-01-20** | **TBD** | **TBD** | $500 | $0 | $500 |

**Updated after primary payment received (Feb 10):**

| Treatment Date | Primary Bill Date | Primary Pay Date | Zero Balance Date | Billed | Primary Paid | Days to Primary Pay | Remaining |
|----------------|-------------------|------------------|-------------------|--------|--------------|---------------------|-----------|
| 2024-01-15 | 2024-01-20 | **2024-02-10** | **TBD** | $500 | **$400** | **21** | **$100** |

**Final state after fully paid (Mar 05):**

| Treatment Date | Primary Bill | Primary Pay | Resp Pay | Zero Balance | Billed | Primary | Resp Party | Total Paid | Remaining | Days to Zero |
|----------------|--------------|-------------|----------|--------------|--------|---------|------------|------------|-----------|--------------|
| 2024-01-15 | 2024-01-20 | 2024-02-10 | 2024-02-25 | **2024-03-05** | $500 | $400 | **$100** | **$500** | **$0** | **49** |

### Destructive Updates Challenge

**Database Administrator Concerns:**

1. **Frequent updates** to fact table rows (violates traditional data warehouse best practice)
2. **Disk storage bloat** from row modifications
3. **Performance degradation** over time

**Solutions:**

**1. Physical Partitioning by Treatment Date**

```sql
CREATE TABLE claims_billing_fact (
    treatment_date_key INT,
    ...
)
PARTITION BY RANGE (treatment_date_key) (
    PARTITION p202401 VALUES LESS THAN (20240201),  -- Jan 2024
    PARTITION p202402 VALUES LESS THAN (20240301),  -- Feb 2024
    ...
);
```

**Benefits:**
- Treatment date never changes (good for clustering)
- Most updates occur within recent partitions
- Older partitions stabilize and can be reorganized

**2. Physical Reorganization Schedule**

```sql
-- Monthly: Reorganize partitions older than 90 days
ALTER TABLE claims_billing_fact REORGANIZE PARTITION p202401;
```

**When to reorganize:**
- After claims in partition have stabilized (stopped changing)
- Typical: 90-180 days after treatment date
- Recovers disk storage, improves performance

**3. Separate Active vs Historical Tables**

```sql
-- Active claims (still being updated)
CREATE TABLE claims_billing_active (
    ...
);

-- Historical claims (finalized, no more updates)
CREATE TABLE claims_billing_historical (
    ...
);

-- Union view for BI queries
CREATE VIEW claims_billing_fact AS
SELECT * FROM claims_billing_active
UNION ALL
SELECT * FROM claims_billing_historical;
```

**ETL Process:**
- New/updated rows go to active table
- When claim reaches zero balance AND no changes for 60 days → move to historical
- Historical table never updated (traditional data warehouse pattern)

---

## Accumulating Snapshot vs Alternatives

### Complementary Transaction Tables

**Accumulating snapshot limitations:**

❌ Cannot fully describe **messy scenarios**:
- Healthcare: Multiple payments for single line item
- Healthcare: Single payment applies to multiple claims
- Order fulfillment: Order canceled then reinstated
- Applicants: Withdrew then reapplied

**Solution:** Build **transaction fact tables** alongside accumulating snapshot.

<complementary_transactions>
**Healthcare Example:**

**Charge Transaction Fact Table:**
- Grain: One row per charge event
- Dimensions: Charge date, patient, physician, procedure, facility
- Facts: Charge amount

**Payment Transaction Fact Table:**
- Grain: One row per payment received
- Dimensions: Payment date, payer, patient, payment method
- Facts: Payment amount

**Claims Billing Accumulating Snapshot:**
- Grain: One row per claim line item
- Provides unified framework for standard workflow analysis
- Simplified current status and lag calculations

**Together:**
- Accumulating snapshot: Standard workflow analysis, current status, conversion rates
- Transactions: Complete audit trail, reconciliation, messy scenarios
</complementary_transactions>

### Periodic Snapshots as Alternative

**When accumulating snapshot doesn't preserve critical point-in-time data:**

**Example:** University admission early decision notification date

Business requirement: **Preserve applicant counts and statuses at early decision deadline** for year-over-year comparison.

**Problem with accumulating snapshot alone:**
- Rows updated as applicants progress
- Can't see "How many had applied as of early decision deadline Nov 1?"

**Solution: Add periodic snapshot table**

```
Applicant Status Periodic Snapshot
Grain: One row per applicant per snapshot date

Dimensions:
- Snapshot_date_key
- Applicant_key
- Current_status_key (inquiry, visited, applied, accepted, enrolled, withdrawn)
- [Other dimensions]

Facts:
- Status_count = 1
- Cumulative_applicant_count
```

**Snapshot Dates:**
- Early decision deadline
- Regular decision deadline
- Enrollment deposit deadline
- Start of term

**Enables:**
- Year-over-year comparison at critical milestones
- "As of" queries: "How many applicants had visited campus as of Nov 1, 2024?"

---

## ETL Processing for Accumulating Snapshots

### Initial Row Creation

**When:** Pipeline instance first appears in source system

**Example:** Applicant submits initial inquiry

```python
def create_applicant_pipeline_row(applicant_id, inquiry_date):
    # Lookup/create dimension keys
    applicant_key = lookup_or_create_applicant(applicant_id)
    inquiry_date_key = lookup_date_dimension(inquiry_date)
    TBD_date_key = -1  # Predefined TBD date dimension row

    # Insert fact row
    insert_fact_row(
        applicant_key=applicant_key,
        inquiry_date_key=inquiry_date_key,
        visit_date_key=TBD_date_key,
        application_date_key=TBD_date_key,
        decision_date_key=TBD_date_key,
        enrolled_date_key=TBD_date_key,
        withdrawn_date_key=TBD_date_key,
        # All lag facts NULL or 0
    )
```

### Incremental Updates

**When:** Applicant reaches next milestone

**Example:** Applicant attends campus visit

```python
def update_applicant_milestone(applicant_id, milestone, milestone_date):
    # Lookup fact row
    fact_row = lookup_fact_row_by_applicant(applicant_id)

    # Lookup date key
    milestone_date_key = lookup_date_dimension(milestone_date)

    # Update appropriate date foreign key
    if milestone == 'CAMPUS_VISIT':
        fact_row.visit_date_key = milestone_date_key
        fact_row.inquiry_to_visit_days = calculate_days(
            fact_row.inquiry_date_key,
            milestone_date_key
        )

    elif milestone == 'APPLICATION_SUBMITTED':
        fact_row.application_date_key = milestone_date_key
        fact_row.visit_to_application_days = calculate_days(
            fact_row.visit_date_key,
            milestone_date_key
        )

    # ... handle other milestones

    # Destructively update fact row
    update_fact_row(fact_row)
```

### Handling Out-of-Sequence Updates

**Challenge:** Milestones sometimes occur out of expected order or data arrives late.

**Example:** Application submitted before campus visit recorded

```python
def validate_milestone_sequence(fact_row, new_milestone, new_date):
    # Define expected sequence
    milestone_order = ['INQUIRY', 'VISIT', 'APPLICATION', 'DECISION', 'ENROLLED']

    current_milestones = get_reached_milestones(fact_row)

    if new_milestone not in expected_next_milestones(current_milestones):
        log_warning(f"Out of sequence: {new_milestone} for applicant {fact_row.applicant_key}")

        # Options:
        # 1. Allow update anyway (may result in negative lag calculations)
        # 2. Reject update (log for manual review)
        # 3. Backfill missing milestones with interpolated dates
```

### Change Data Capture for Updates

**Challenge:** Identify which fact rows need updating during incremental ETL.

**Solution:** Operational system provides change feed:

```sql
-- Source system change table
CREATE TABLE applicant_milestone_changes (
    applicant_id INT,
    milestone_type VARCHAR(50),
    milestone_date DATE,
    change_timestamp TIMESTAMP
);
```

**ETL Process:**

```python
def incremental_update_applicant_pipeline():
    # Get milestones changed since last ETL run
    changes = get_milestone_changes_since(last_etl_timestamp)

    for change in changes:
        # Lookup fact row
        fact_row = lookup_fact_row_by_applicant(change.applicant_id)

        if fact_row:
            # Update existing row
            update_applicant_milestone(
                change.applicant_id,
                change.milestone_type,
                change.milestone_date
            )
        else:
            # Row doesn't exist yet - create it
            create_applicant_pipeline_row(
                change.applicant_id,
                change.milestone_date
            )

    update_etl_timestamp()
```

---

## Common Mistakes and Solutions

### Mistake 1: Too Many Milestones

**Problem:** Modeling every possible status change as separate date dimension (15+ milestone dates).

**Solution:** Model only **standard, major milestones** that apply to most instances. Use status dimension or transaction table for variable intermediate steps.

### Mistake 2: Long-Running Processes

**Problem:** Using accumulating snapshot for multi-year customer lifecycle.

**Solution:** Use periodic snapshot instead. Accumulating snapshots appropriate for days-to-months workflows, not years.

### Mistake 3: NULL Date Foreign Keys

**Problem:** Allowing NULL foreign keys for future milestones.

**Solution:** Use TBD date dimension row. Prevents NULL FKs, enables clean BI queries.

### Mistake 4: Not Partitioning on Stable Date

**Problem:** Partitioning on milestone dates that change during updates.

**Solution:** Partition on **earliest date** that never changes (treatment date, inquiry date, order date).

### Mistake 5: No Reorganization Strategy

**Problem:** Fact table performance degrades over time from frequent updates.

**Solution:** Scheduled reorganization of stabilized partitions. Or separate active/historical tables.

### Mistake 6: Ignoring Messy Scenarios

**Problem:** Trying to model all edge cases in accumulating snapshot (multiple payments, cancellations, etc.).

**Solution:** Use accumulating snapshot for **standard workflow** only. Add transaction tables for complete audit trail.

### Mistake 7: Overwriting Historical Milestone Data

**Problem:** ETL replaces previously recorded milestone dates with updated information.

**Solution:** Preserve original milestone dates unless explicit business rule to correct retroactively. Use audit dimension to track changes.

---

## Performance Optimization

### Indexing Strategy

```sql
-- Applicant pipeline fact table indexes

-- Dimension foreign keys
CREATE INDEX idx_applicant ON applicant_pipeline_fact(applicant_key);
CREATE INDEX idx_inquiry_date ON applicant_pipeline_fact(inquiry_date_key);

-- Partitioning key
CREATE INDEX idx_partition ON applicant_pipeline_fact(inquiry_date_key);

-- Common query patterns
CREATE INDEX idx_decision_status ON applicant_pipeline_fact(decision_status_key, enrolled_date_key);
```

### Aggregate Tables

**Pre-compute common milestone summaries:**

```sql
CREATE TABLE applicant_pipeline_summary AS
SELECT
    inquiry_date_key,
    intended_program_key,
    geographic_region_key,
    COUNT(*) AS inquiry_count,
    COUNT(CASE WHEN visit_date_key != -1 THEN 1 END) AS visit_count,
    COUNT(CASE WHEN application_date_key != -1 THEN 1 END) AS application_count,
    COUNT(CASE WHEN enrolled_date_key != -1 THEN 1 END) AS enrolled_count,
    AVG(inquiry_to_visit_days) AS avg_inquiry_to_visit_days,
    AVG(total_pipeline_days) AS avg_total_pipeline_days
FROM
    applicant_pipeline_fact
GROUP BY
    inquiry_date_key,
    intended_program_key,
    geographic_region_key;
```

**Refresh:** Nightly or after ETL updates complete.

---

## Summary

**Accumulating snapshots excel at:**
- ✅ Short-lived pipeline/workflow tracking
- ✅ Current status visibility across entire instance population
- ✅ Conversion rate and funnel analysis
- ✅ Bottleneck identification
- ✅ Standard milestone metrics

**Complement with:**
- Transaction fact tables for complete audit trail and messy scenarios
- Periodic snapshots for point-in-time preservation at critical dates

**Remember:**
- One row per pipeline instance
- Multiple date dimensions (TBD for future milestones)
- Rows updated as workflow progresses
- Appropriate for short-lived processes only
- Partition on stable earliest date
- Schedule reorganization for stabilized partitions
