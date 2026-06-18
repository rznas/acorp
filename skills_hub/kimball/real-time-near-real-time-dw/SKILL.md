---
name: real-time-near-real-time-dw
description: Design real-time and near-real-time data warehouse architectures with appropriate latency triage (instantaneous/intra-day/daily), real-time partitions, micro-batch processing, and data quality trade-offs. Use when requirements demand low-latency data delivery beyond traditional daily batch processing.
---

# Real-Time and Near-Real-Time Data Warehouse Design

## Overview

Real-time data warehousing extends the traditional batch-oriented DW/BI architecture to deliver data with reduced latency—from many times per day (intra-day) to within seconds (instantaneous). This skill guides you through understanding true business requirements for low-latency data, selecting appropriate architectural approaches, and managing the inherent trade-offs between speed and data quality.

**Critical principle**: As data latency decreases, data quality inevitably suffers. Business users must be informed participants in this trade-off decision.

## When to Use This Skill

- Business users demand data fresher than daily batch updates
- Operational decisions require near-current data visibility
- Source systems can support frequent or continuous data extraction
- Requirements exist for intra-day reporting or instantaneous views
- Use cases include inventory status tracking, fraud detection, real-time personalization, operational dashboards

**Prerequisites**: Understanding of traditional ETL architecture (SKILL 8), incremental ETL processing (SKILL 10), and change data capture methods.

---

## Step 1: Perform Real-Time Triage

Asking business users if they want "real-time" data without constraints yields worthless responses ("That sounds good; go for it!"). Instead, use a structured triage process.

### Three Latency Categories

Divide requirements into three distinct categories with different architectural implications:

<real_time_categories>

**1. Daily (Batch)**
- **Definition**: Data updated once per day, typically after end-of-day reconciliation
- **Latency**: 24 hours
- **Data quality**: Highest—complete transaction sets, full quality screens, resolved foreign keys
- **Architecture**: Traditional batch ETL with file extracts
- **When appropriate**: Most analytic use cases, compliance reporting, financial consolidation

**2. Intra-Day (Micro-Batch)**
- **Definition**: Data updated multiple times per day (hourly, every 15 minutes, etc.)
- **Latency**: Minutes to hours
- **Data quality**: Moderate—may have incomplete transactions, limited quality screens
- **Architecture**: Micro-batch processing with message queues, transaction logs, or frequent extracts
- **When appropriate**: Operational dashboards, same-day sales visibility, inventory monitoring

**3. Instantaneous (Real-Time Streaming)**
- **Definition**: Screen reflects true current state of source system at every instant
- **Latency**: Seconds or less
- **Data quality**: Lowest—transaction fragments, minimal quality checks, unresolved keys
- **Architecture**: Enterprise Information Integration (EII) with direct source system queries or event streams
- **When appropriate**: Live inventory commitment, fraud detection, real-time personalization

</real_time_categories>

### Triage Process

**For each data requirement:**

1. **Classify the true latency need**: Daily, intra-day, or instantaneous?
2. **Document the business justification**: What decision requires this latency?
3. **Assess data quality awareness**: Do users understand the trade-offs?
4. **Identify source system capability**: Can the source support the required latency?
5. **Estimate cost and complexity**: Resources required vs business value

**Reality check questions:**
- Will users actually make different decisions with faster data?
- Can the operational process respond to decisions at this speed?
- Is the source data stable enough at this latency (or will it be corrected later)?
- Do users understand they may see provisional, incomplete, or incorrect data?

---

## Step 2: Understand Data Quality Trade-Offs

As latency decreases, data quality compromises accumulate. Business users must consciously accept these trade-offs.

<data_quality_tradeoffs>

### Daily Batch (Highest Quality)
- **Complete transaction sets**: Only fully committed, reconciled transactions
- **Full quality screening**: Column, structure, and business rule screens
- **Resolved foreign keys**: All dimensional context available
- **Conformed dimensions**: Full conforming process applied
- **End-of-day corrections**: Source system reconciliation applied
- **Full audit trail**: Complete lineage and archiving

### Intra-Day Micro-Batch (Moderate Quality)
- **Incomplete transactions**: Order placed but credit check pending
- **Limited quality screening**: Column screens only; structure/business rule screens skipped
- **Unresolved foreign keys**: New customers/products may not be in dimension yet
- **Partial conforming**: May use stale or provisional dimension values
- **No reconciliation**: Source corrections applied in nightly batch
- **Partial audit trail**: Limited lineage tracking

### Instantaneous Streaming (Lowest Quality)
- **Transaction fragments**: Individual events without context
- **Minimal quality checks**: Only basic type validation
- **Missing foreign keys**: Dimensions arrive after facts
- **No conforming**: Data directly from source without standardization
- **No corrections**: Raw unprocessed data
- **No audit trail**: No persistent storage in ETL pipeline

</data_quality_tradeoffs>

### Recommended Hybrid Approach

**Dual pipeline pattern**:
- **Real-time/intra-day pipeline**: Delivers provisional data with low latency
- **Nightly batch pipeline**: Overwrites real-time data with fully quality-checked, reconciled data

This approach provides speed when needed while ensuring overnight data quality convergence.

**Implementation**: Use real-time partition (see Step 4) that is rebuilt/replaced by nightly batch process.

---

## Step 3: Select Appropriate Architecture

Based on triage results, select the architectural pattern that matches latency requirements.

<architecture_patterns>

### Architecture 1: Daily Batch (Traditional)

**Data flow**:
1. Source system end-of-day batch file or reconciliation flag
2. Extract to staging area
3. Full quality screening (column, structure, business rule)
4. Conforming and integration
5. Surrogate key assignment
6. Load to presentation dimensional model

**Advantages**:
- Maximum data quality
- Complete audit trail and archiving
- Proven, stable architecture
- Lower complexity and cost

**Technology**: ETL tool or scripts, scheduled batch jobs, file-based staging

### Architecture 2: Intra-Day Micro-Batch

**Data flow**:
1. Change data capture from message queue, transaction log, or frequent polling
2. Extract to staging (file or stream)
3. Limited quality screening (column screens only)
4. Conforming (with provisional values if needed)
5. Surrogate key lookup (with "dummy" members for missing keys)
6. Load to real-time partition
7. Nightly batch process overwrites with fully quality-checked data

**Advantages**:
- Balances latency and quality
- Retains most ETL subsystems
- Supports dimensional modeling
- Nightly correction available

**Trade-offs**:
- Requires change data capture infrastructure (message queues, log scraping)
- Eliminates structure and business rule quality screens
- May use provisional dimension values
- Requires dual pipeline (real-time + batch)

**Technology**: Message queue monitoring (e.g., Kafka, JMS), CDC tools, micro-batch ETL processing, real-time partitions

### Architecture 3: Instantaneous EII (Enterprise Information Integration)

**Data flow**:
1. Direct query to source system or subscribe to event stream
2. Minimal transformation (data type conversion only)
3. Stream directly to user screen (no persistent storage in DW)

**Advantages**:
- True instantaneous view
- No data staleness

**Trade-offs**:
- Query complexity limited by source system performance
- No data quality processing
- No caching or staging (challenges for backup, recovery, archiving, compliance)
- Source system bears query load
- No conforming or integration
- Not a true "data warehouse"

**Technology**: EII tools, direct database connections, event streaming platforms, operational data stores (ODS)

</architecture_patterns>

### Decision Framework

| Requirement | Architecture | Key Considerations |
|-------------|--------------|-------------------|
| Latency: 24 hours | Daily batch | Use unless specific intra-day justification exists |
| Latency: 1-4 hours | Intra-day micro-batch | Requires message queue or log scraping CDC |
| Latency: < 1 minute | Instantaneous EII | Verify source system can handle query load |
| Data quality critical | Daily batch | Compliance, financial reporting |
| Operational decisions | Intra-day or instantaneous | Inventory commitment, fraud detection |

---

## Step 4: Design Real-Time Partitions

For intra-day architectures, implement a **real-time partition** as a physical extension of the static data warehouse fact table.

### Real-Time Partition Requirements

<partition_requirements>

A real-time partition should:

1. **Contain only recent activity**: All data since last static DW update (e.g., since midnight)
2. **Match grain and structure**: Same dimensional design as static fact table
3. **Be lightly indexed or unindexed**: Enable continuous data loading ("trickle-feed")
4. **Be pinned in memory**: Provide fast queries despite lack of indexes
5. **Be seamlessly queryable**: BI tools query across static + real-time partitions transparently
6. **Be physically partitioned**: Use database partitioning by activity date if possible

</partition_requirements>

### Transaction Fact Table Real-Time Partition

**Use case**: Retail sales, web clickstream, call center transactions

**Design**:
- Grain: One row per transaction (identical to static fact table)
- Content: Today's transactions only
- Indexing: Completely unindexed (or minimally indexed on foreign keys)
- Size: Manageable for in-memory (e.g., 10M transactions/day × 40 bytes = 400 MB)
- Query strategy: Pin in memory, rely on in-memory scans

**Daily process**:
1. At midnight, append real-time partition to static fact table as new date partition
2. Apply full indexing to yesterday's partition
3. Empty real-time partition for today

**Example** (retail environment):
```
Static fact table: 150 GB (1+ years), heavily indexed, aggregates available
Real-time partition: 400 MB (today only), unindexed, pinned in memory
Combined view: UNION ALL query or partitioned table spanning both
```

### Periodic Snapshot Fact Table Real-Time Partition

**Use case**: Account balances, inventory levels, rolling month metrics

**Design**:
- Grain: One row per entity per period (e.g., account by month)
- Content: Current developing period (e.g., month-to-date balances)
- Updates: Continuous updates to same rows (not inserts)
- Size: One period's worth of entities (e.g., 15M accounts × 32 bytes = 480 MB)

**End-of-period process**:
1. On last day of month, merge real-time partition to static fact table
2. Apply final calculations (e.g., month-end balances)
3. Build indexes on new static partition
4. Empty real-time partition for new period

**Example** (banking):
```
Static fact table: 540M rows (15M accounts × 36 months), indexed
Real-time partition: 15M rows (current month only), updated continuously
Combined view: 36 months of history + current developing month
```

### Accumulating Snapshot: Not Recommended for Real-Time Partition

Accumulating snapshots (order pipeline, claim lifecycle) involve **updating** existing rows as milestones occur. This pattern does not benefit from real-time partitioning because:
- No clear date-based partition boundary
- Rows span multiple dates
- Updates distributed across many partitions

**Alternative**: Use transaction fact table for real-time events, maintain accumulating snapshot in batch process.

---

## Step 5: Implement Architectural Trade-Offs

When implementing intra-day or instantaneous architectures, consciously make the following trade-offs:

<architectural_tradeoffs>

### Trade-Off 1: Replace Batch Files

**Daily batch approach**:
- Extract from end-of-day batch file prepared by source system
- File represents clean, consistent, reconciled view
- Foreign keys resolved, business rules applied

**Intra-day/instantaneous approach**:
- Extract from message queue, transaction log, or database triggers
- Raw instantaneous data without reconciliation
- Foreign keys may be unresolved
- Business rules not yet applied by source system

**Mitigation**:
- Implement provisional dimension rows for unresolved keys
- Run parallel nightly batch to correct real-time data
- Inform users of provisional data status

### Trade-Off 2: Limit Data Quality Screens

**Daily batch approach**:
- Column screens (single field tests, simple decodes)
- Structure screens (multi-field, multi-record validations)
- Business rule screens (complex logic, external service calls)

**Intra-day approach**:
- Column screens only (retain these even in aggressive real-time)
- Eliminate structure screens (no time for multi-table lookups)
- Eliminate business rule screens (no time for address validation, credit checks, etc.)

**Instantaneous approach**:
- Minimal type validation only

**Mitigation**:
- Tag real-time data with quality confidence indicators
- Provide users separate "provisional" vs "certified" data views
- Nightly batch applies full quality screening and overwrites

### Trade-Off 3: Post Facts with Old Dimensions

**Issue**: Facts arrive before dimensions (transaction before customer update)

**Daily batch approach**:
- Wait for dimension to be available before posting fact

**Intra-day/instantaneous approach**:
- Post fact with old dimension value (if customer exists) or
- Post fact with generic "dummy" dimension member (if new customer)
- Update dimension when it arrives (immediately or in batch)

**Mitigation**:
- Implement late-arriving dimension handler (ETL Subsystem 14)
- Use provisional dimension rows with flags
- Overwrite in nightly batch with correct dimensional context

### Trade-Off 4: Eliminate Data Staging

**Daily batch approach**:
- Stage data after extraction
- Stage data after cleaning/conforming
- Stage data after delivery
- Archive all staged data for recovery and lineage

**Intra-day approach**:
- May skip some staging to reduce latency
- Minimal staging for recovery

**Instantaneous EII approach**:
- No staging (stream directly to screen)
- No persistent storage in ETL pipeline

**Mitigation**:
- For intra-day, retain staging after delivery for minimum recovery capability
- For EII, ensure source system handles backup, recovery, archiving, compliance
- Document gaps in audit trail and compliance coverage

</architectural_tradeoffs>

---

## Step 6: Implement Change Data Capture for Intra-Day

Intra-day architectures require CDC mechanisms beyond traditional batch extracts.

### CDC Methods for Intra-Day/Real-Time

<cdc_methods>

**1. Message Queue Monitoring** (Recommended for real-time)
- Source applications publish transactions to message queue (Kafka, JMS, RabbitMQ, AWS Kinesis)
- ETL subscribes to queue and processes events
- **Advantages**: Low overhead, event-driven, designed for streaming
- **Challenges**: No replay if connection lost; ensure guaranteed delivery

**2. Database Log Scraping**
- Snapshot or poll database redo/transaction log for table changes
- **Advantages**: Captures all changes regardless of application path
- **Challenges**: Messy, log may be purged during crises, vendor-specific
- **Use when**: All other methods exhausted

**3. Audit Column Polling** (Frequent polling)
- Poll source tables every N minutes for rows where last_modified_date > last_poll_time
- **Advantages**: Simple, no special infrastructure
- **Challenges**: Frequent polling loads source system, may miss deletes
- **Use when**: Source has reliable audit columns and can tolerate polling frequency

**4. Database Triggers**
- Database triggers fire on INSERT/UPDATE/DELETE and write to staging table
- **Advantages**: Guaranteed capture, low latency
- **Challenges**: Performance impact on source OLTP, trigger management overhead
- **Use when**: Source DBAs approve and source system can tolerate overhead

</cdc_methods>

**Recommendation**: Message queue monitoring is the most scalable and performant approach for high-volume intra-day/real-time requirements.

---

## Step 7: Manage the Dual Pipeline

For intra-day architectures, implement and coordinate real-time and batch pipelines.

<dual_pipeline>

### Real-Time Pipeline (Intra-Day)

**Frequency**: Continuous or every N minutes
**Scope**: Minimal transformations for speed
**Target**: Real-time partition

**Process**:
1. Capture changes from message queue/log
2. Extract to staging (optional, may stream)
3. Column screens only
4. Surrogate key lookup (use provisional/dummy for missing)
5. Load to real-time partition (INSERT or UPDATE depending on fact type)

### Batch Pipeline (Nightly)

**Frequency**: Once per day (e.g., midnight)
**Scope**: Full quality processing
**Target**: Static fact table + dimension updates

**Process**:
1. Extract from end-of-day batch file or reconciliation point
2. Full quality screening (column, structure, business rule)
3. Full conforming process
4. Surrogate key assignment/lookup
5. Load to static fact table
6. **Overwrite or replace real-time partition** with corrected data
7. Apply indexing, aggregates

### Coordination Points

1. **Partition swap**: At midnight, append real-time partition to static table, clear for new day
2. **Dimension synchronization**: Batch process resolves provisional dimension members
3. **Fact correction**: Batch process corrects facts posted with wrong dimensional context
4. **Quality tagging**: Real-time data tagged with quality confidence flag; batch removes flag

</dual_pipeline>

### User Communication

**Provide distinct data views**:
- **"Live" view**: Includes real-time partition (provisional data)
- **"Certified" view**: Static data only (fully quality-checked)
- **"Complete" view**: Combined with quality confidence indicators

**Dashboard labeling**:
- "As of [timestamp]" with data freshness indicator
- "Provisional data—subject to overnight correction" warning on real-time views

---

## Common Mistakes to Avoid

<common_mistakes>

1. **Saying "yes" to real-time without triage**
   - *Mistake*: Accepting all "real-time" requests at face value
   - *Solution*: Use three-category triage; most needs satisfied by intra-day or even daily

2. **Underestimating data quality impact**
   - *Mistake*: Promising same data quality with lower latency
   - *Solution*: Explicitly document quality trade-offs; get business sign-off

3. **Implementing real-time without nightly correction**
   - *Mistake*: Loading provisional data and never correcting it
   - *Solution*: Always implement dual pipeline with batch reconciliation

4. **Eliminating all staging/archiving**
   - *Mistake*: Streaming data without persistent storage for compliance/recovery
   - *Solution*: Stage at minimum after delivery; archive as needed for compliance

5. **Over-indexing real-time partition**
   - *Mistake*: Building indexes on real-time partition prevents fast loading
   - *Solution*: Minimize or eliminate indexes; pin partition in memory instead

6. **Not informing users about provisional data**
   - *Mistake*: Users don't understand data may be incorrect or incomplete
   - *Solution*: Label real-time data clearly; provide separate certified views

7. **Using timed extracts without safeguards**
   - *Mistake*: SELECT WHERE date = SYSDATE-1 causes duplicate/missing data on failures
   - *Solution*: Use message queues or audit columns with watermark tracking instead

8. **Ignoring source system load**
   - *Mistake*: Frequent polling or EII queries overwhelm source OLTP system
   - *Solution*: Use change data capture methods designed for low overhead

</common_mistakes>

---

## Implementation Checklist

### Triage Phase
- [ ] Classify each requirement as daily/intra-day/instantaneous
- [ ] Document business justification for low latency
- [ ] Verify users understand data quality trade-offs
- [ ] Assess source system capability and capacity
- [ ] Estimate cost vs value for each latency tier

### Architecture Design Phase
- [ ] Select architecture (batch, micro-batch, or EII) per requirement
- [ ] Design real-time partition structure (transaction or periodic snapshot)
- [ ] Define partition size, indexing strategy, memory pinning approach
- [ ] Design dual pipeline coordination (real-time + batch)
- [ ] Plan data quality confidence tagging and user views

### CDC Implementation Phase
- [ ] Select CDC method (message queue, log scraping, audit columns, triggers)
- [ ] Implement CDC infrastructure (message queue setup, subscribers)
- [ ] Test CDC reliability (failure scenarios, replay capability)
- [ ] Establish monitoring for CDC lag and failures

### ETL Development Phase
- [ ] Build real-time ETL with limited quality screens (column only)
- [ ] Implement provisional dimension handling (dummy members, late-arriving)
- [ ] Build nightly batch ETL with full quality screening
- [ ] Implement partition swap/merge process
- [ ] Test dual pipeline coordination and data correction

### Presentation Layer Phase
- [ ] Create real-time partition and static fact table union views
- [ ] Implement query performance optimization (memory pinning, partitioning)
- [ ] Build "Live", "Certified", and "Complete" user views
- [ ] Add data freshness timestamps and provisional data warnings

### Operations Phase
- [ ] Document quality trade-offs and communicate to users
- [ ] Establish monitoring for real-time partition size and load performance
- [ ] Monitor batch reconciliation and correction rates
- [ ] Track and publish data quality metrics over time
- [ ] Review and adjust latency tiers based on actual usage

---

## Integration with Other Skills

- **SKILL 8 (ETL Architecture Design)**: Real-time extends the 34 subsystems with streaming variants
- **SKILL 10 (Incremental ETL)**: Real-time is extreme incremental processing with minimal transformations
- **SKILL 7 (Data Profiling/Quality)**: Quality screens reduced; quality monitoring becomes more important
- **SKILL 11 (Physical Database Design)**: Real-time partitions require specialized indexing and memory strategies
- **Industry skills**: Real-time patterns vary by industry (retail clickstream, financial fraud, healthcare alerts)

---

## Summary

Real-time and near-real-time data warehousing demands careful triage, explicit trade-offs, and appropriate architecture selection. The three-tier classification (daily/intra-day/instantaneous) provides a framework for aligning business needs with technical capabilities and data quality realities. Real-time partitions enable intra-day delivery while maintaining dimensional design. The dual pipeline pattern (real-time + nightly batch) balances speed with quality. Success requires informed business users who understand and accept the inherent compromises of low-latency data.

**Key principle**: Fast data and quality data exist on a spectrum. Choose consciously based on business value and risk tolerance.
