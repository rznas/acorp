# ETL Subsystems Quick Reference

Complete index of all 34 ETL subsystems with purpose, when to use, and key techniques.

---

# EXTRACTING (Subsystems 1-3)

## Subsystem 1: Data Profiling
**Purpose**: Understand source data content, consistency, and structure
**When to Use**: Strategic (go/no-go) during requirements; Tactical during ETL design
**Key Techniques**: Column screens, structure screens, business rule screens
**Output**: Data profiling reports, quality scorecards, screen specifications

## Subsystem 2: Change Data Capture (CDC)
**Purpose**: Isolate changed source data since last update
**When to Use**: All incremental loads after initial historic load
**Key Techniques**: Audit columns, full diff compare, log scraping, message queues
**Avoid**: Timed extracts (unreliable)

## Subsystem 3: Extract System
**Purpose**: Physical extraction from source systems
**When to Use**: All ETL processes
**Key Techniques**: File-based extract, stream-based extract, compression, encryption
**Decision**: File vs. stream based on source type and restart requirements

---

# CLEANING & CONFORMING (Subsystems 4-8)

## Subsystem 4: Data Cleansing System
**Purpose**: Improve data quality through systematic screening
**When to Use**: All ETL processes
**Quality Screen Types**:
- **Column**: Single-field tests (nulls, ranges, formats)
- **Structure**: Multi-field relationships (hierarchies, FK/PK)
- **Business Rule**: Complex validations (thresholds, time-dependent)

**Response Options**:
1. Halt process (least preferred)
2. Suspense file (questionable integrity)
3. Tag and continue (PREFERRED)

## Subsystem 5: Error Event Schema
**Purpose**: Centralized recording of all quality screen errors
**When to Use**: When comprehensive quality tracking required
**Structure**:
- Error event fact table (grain = one error)
- Error event detail fact table (grain = one field per error)
**Dimensions**: Calendar date, batch job, screen

## Subsystem 6: Audit Dimension Assembler
**Purpose**: Attach metadata context to fact table rows
**When to Use**: All fact table loads
**Typical Attributes**: ETL version, quality scores, load batch ID, pass/fail flags
**Pattern**: One audit dimension row per batch, attached to all fact rows in batch

## Subsystem 7: Deduplication System
**Purpose**: Merge dimension data from multiple sources
**When to Use**: Customer master, product master, any multi-source dimension
**Key Techniques**: Fuzzy matching, survivorship rules, back references
**Tools**: Data integration, data standardization packages

## Subsystem 8: Conforming System
**Purpose**: Align dimension content across enterprise
**When to Use**: Building conformed dimensions for drill-across
**Process**: Combine → Deduplicate → Filter → Standardize
**Integration**: Works with Subsystems 17 (Dimension Manager) and 18 (Fact Provider)

---

# DELIVERING (Subsystems 9-21)

## Subsystem 9: Slowly Changing Dimension (SCD) Manager
**Purpose**: Handle dimension attribute changes systematically
**When to Use**: All dimension table loads with any changing attributes
**SCD Types**:
- Type 0: Retain original
- Type 1: Overwrite (no history)
- Type 2: Add row (track history) — PRIMARY
- Type 3: Add column (previous vs. current)
- Type 4: Mini-dimension
- Type 5: Mini-dimension + Type 1 outrigger
- Type 6: Type 1 + Type 2 combined
- Type 7: Dual Type 1/Type 2

**Type 2 Housekeeping**: Row effective date/time, row end date/time, current flag, change date, reason

## Subsystem 10: Surrogate Key Generator
**Purpose**: Generate meaningless integer keys for dimensions
**When to Use**: All dimension table loads
**Best Practice**: Database sequence generator or ETL tool
**Avoid**: Triggers (performance), concatenated natural key + timestamp (doesn't scale)

## Subsystem 11: Hierarchy Manager
**Purpose**: Populate and maintain dimensional hierarchies
**When to Use**: Dimensions with multiple rollup paths
**Hierarchy Types**:
- **Fixed depth**: Separate attributes per level
- **Ragged/variable depth**: Bridge table required

## Subsystem 12: Special Dimensions Manager
**Purpose**: Handle unique dimension types
**Dimension Types**:
- **Date/Time**: Pre-built, no conventional source
- **Junk**: Miscellaneous flags (on-the-fly or pre-populate)
- **Mini**: Rapidly changing attributes in large dimension
- **Shrunken subset**: Built from base dimension
- **Small static**: ETL-created lookup tables
- **User maintained**: Business-owned with ETL interface

## Subsystem 13: Fact Table Builders
**Purpose**: Load three primary fact table types
**Fact Table Types**:

**Transaction**:
- Grain: One row per event
- Load: Bulk insert
- Updates: Rare (error correction only)

**Periodic Snapshot**:
- Grain: One row per period
- Load: Insert at period end or update hot rolling
- Facts: Often semi-additive (balances)

**Accumulating Snapshot**:
- Grain: One row per process lifecycle
- Load: INSERT new + UPDATE as process progresses
- Multiple date dimensions for milestones

## Subsystem 14: Surrogate Key Pipeline
**Purpose**: Replace natural keys with surrogate keys before fact loading
**When to Use**: All fact table loads
**Process**: Lookup current surrogate key for each natural key
**Type 2 Logic**: Use BETWEEN on effective/end dates
**RI Guarantee**: All dimension updates complete first

## Subsystem 15: Multivalued Dimension Bridge Table Builder
**Purpose**: Link dimensions with multiple values at fact grain
**When to Use**: Patient diagnoses, sales commissions, variable depth hierarchies
**Challenges**: Build unique groups vs. reuse, time variance, weighting factors

## Subsystem 16: Late Arriving Data Handler
**Purpose**: Handle facts/dimensions arriving after expected timeline
**When to Use**: Distributed sources, real-time systems
**Late Arriving Dimensions**: Create dummy row → Type 1 overwrite when complete
**Late Arriving Facts**: Find dimension key effective at transaction time

## Subsystem 17: Dimension Manager System
**Purpose**: Centralized authority for publishing conformed dimensions
**When to Use**: Enterprise integration with conformed dimensions
**Responsibilities**: Implement common labels, add Type 2 rows, modify Type 1/3 in place, replicate to fact providers
**Governance**: Single source per conformed dimension

## Subsystem 18: Fact Provider System
**Purpose**: Receive conformed dimensions and manage fact tables
**When to Use**: Enterprise integration, receiving from dimension managers
**Responsibilities**: Receive dimensions, add/modify fact rows, manage aggregates, QA, inform users

## Subsystem 19: Aggregate Builder
**Purpose**: Build and maintain aggregate fact tables
**When to Use**: Performance requirements demand pre-aggregation
**Strategies**: Incremental (fastest), drop/rebuild (Type 1 dimension changes)
**Critical Rule**: MUST match atomic base data

## Subsystem 20: OLAP Cube Builder
**Purpose**: Load OLAP cubes from dimensional schemas
**When to Use**: OLAP tools in BI architecture
**Source**: Relational dimensional schema (foundation)
**Timing**: After conventional ETL (leverage RI and cleansing)
**Warning**: Type 1 SCDs can corrupt/drop cubes

## Subsystem 21: Data Propagation Manager
**Purpose**: Extract from presentation layer for external consumption
**When to Use**: Partners, government reporting, analytic apps, data mining
**Approach**: ETL where target requirements non-negotiable

---

# MANAGING (Subsystems 22-34)

## Subsystem 22: Job Scheduler
**Purpose**: Metadata-driven job control
**When to Use**: All production ETL
**Capabilities**: Job definition, time/event scheduling, dependencies, metadata capture, logging, notification

## Subsystem 23: Backup System
**Purpose**: Enable recovery after failures
**When to Use**: All production systems
**Requirements**: High performance, simple administration, lights-out automation
**Archive**: Lower-cost media, long-term retention, regular refresh

## Subsystem 24: Recovery and Restart System
**Purpose**: Resume or back out failed jobs
**When to Use**: Production ETL
**Techniques**: Checkpoint functionality, commit small sets, fact table surrogate key for range-based operations

## Subsystem 25: Version Control System
**Purpose**: Archive and recover ETL logic and metadata
**When to Use**: All ETL development
**Capabilities**: Check-out/in, source comparison, librarian function, master version numbers

## Subsystem 26: Version Migration System
**Purpose**: Move ETL between Dev → Test → Production
**When to Use**: All releases
**Requirements**: Version control interface, connection settings, rollback capability

## Subsystem 27: Workflow Monitor
**Purpose**: Dashboard and reporting for ETL operations
**When to Use**: All production ETL
**Monitors**: Job status, performance metrics, infrastructure, trending, alerts

## Subsystem 28: Sorting System
**Purpose**: High-performance sorting
**When to Use**: Aggregation, joining flat files, quality diagnosis
**Options**: ETL tool, DBMS, dedicated sort package

## Subsystem 29: Lineage and Dependency Analyzer
**Purpose**: Track data lineage and impact analysis
**When to Use**: Compliance, change management
**Lineage** (backward): Data element → sources → transformations
**Dependency** (forward): Source element → downstream tables/reports

## Subsystem 30: Problem Escalation System
**Purpose**: Automated notification and resolution tracking
**When to Use**: All production ETL
**Levels**: Help desk → Sys admin/DBA → ETL manager → ETL developer

## Subsystem 31: Parallelizing/Pipelining System
**Purpose**: Load within time window using multiple processors
**When to Use**: Large volumes, tight time windows
**Opportunities**: Extraction, transformation, loading

## Subsystem 32: Security System
**Purpose**: Protect ETL data, metadata, processes
**When to Use**: All ETL systems
**Requirements**: Role-based security, access records, encryption, controlled production access

## Subsystem 33: Compliance Manager
**Purpose**: Maintain chain of custody
**When to Use**: Highly compliant environments
**Requirements**: No destructive changes, all changes as inserts, begin/end timestamps, change/source references

## Subsystem 34: Metadata Repository Manager
**Purpose**: Capture and manage ETL metadata
**When to Use**: All ETL systems
**Types**: Process, technical, business metadata
**Strategy**: Balance between nothing and too much

---

# Subsystem Selection Matrix

| Requirement | Required Subsystems |
|-------------|-------------------|
| **Minimum Viable** | 1, 3, 4, 6, 9, 10, 13, 14, 22, 23, 24, 30 |
| **Incremental Processing** | Add 2 |
| **Multiple Sources** | Add 7, 8 |
| **Enterprise Integration** | Add 17, 18 |
| **Compliance** | Add 5, 29, 33 |
| **Performance** | Add 19, 20, 31 |
| **Large Dimensions** | Add 12 (mini, junk) |
| **Complex Relationships** | Add 11, 15 |
| **Real-Time** | Add 2 (advanced), modify pipeline |
| **External Consumers** | Add 21 |

---

# Performance Bottleneck Priority Order

1. Poorly indexed queries
2. SQL syntax forcing wrong optimizer choice
3. Insufficient RAM causing thrashing
4. Sorting in RDBMS
5. Slow transformation steps
6. Excessive I/O
7. Unnecessary write-then-read cycles
8. Drop/rebuild aggregates vs. incremental
9. Late-applied filtering (CDC)
10. Untapped parallelization
11. Unnecessary transaction logging
12. Network/file transfer overhead

---

# Critical Decision Points

## ETL Tool Selection (Subsystem Decision)
**Factors**: Source systems, transformation complexity, developer skills, budget
**Benefits**: Self-documentation, metadata, version control, advanced transforms, performance optimization

## CDC Strategy (Subsystem 2)
**Options**: Audit columns (reliable if trustworthy), full diff (thorough but expensive), log scraping (messy but comprehensive), message queues (low overhead but no replay)
**Avoid**: Timed extracts

## SCD Strategy (Subsystem 9)
**Type 1**: No history needed or corrections only
**Type 2**: Track history (most common)
**Advanced**: Types 4-7 for special cases

## Real-Time vs. Batch (Multiple Subsystems)
**Daily**: Full ETL pipeline, reconciled data
**Intra-day**: Micro-batch, reduced quality screens
**Instantaneous**: EII solution, no caching

Use this reference to quickly identify which subsystems are needed for your specific ETL architecture requirements.
