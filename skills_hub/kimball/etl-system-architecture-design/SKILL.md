---
name: etl-system-architecture-design
description: Design comprehensive ETL system architecture using the 34 subsystems framework for extracting, cleaning, conforming, delivering, and managing data warehouse operations. Use when planning the complete back-room ETL architecture after dimensional models are designed and before ETL development begins.
---

# ETL System Architecture Design

## When to Use This Skill

Use this skill **after dimensional models are designed** and **before ETL development** begins to:
- Plan the complete ETL architecture for a dimensional DW/BI system
- Identify which of the 34 subsystems are required for your specific needs
- Develop high-level ETL system plan and specifications
- Make critical ETL tool and technology decisions
- Define default strategies for common ETL operations

**Prerequisites**: Completed dimensional model design, source-to-target mappings drafted, and high-level architecture plan outlined.

## Supporting Reference Files

- **[requirements-checklist.md](requirements-checklist.md)**: Detailed 10-area requirements roundup with checklists
- **[subsystem-quick-reference.md](subsystem-quick-reference.md)**: Complete index of all 34 subsystems with selection matrix

## Core Principle

The ETL system is the most resource-intensive component of DW/BI development. A structured approach using the 34 ETL subsystems prevents the "spaghetti-mess" anti-pattern and ensures reliability, availability, and manageability.

---

# The Four Major ETL Components

Every ETL architecture consists of these four major components:

## 1. EXTRACTING (Subsystems 1-3)
Gathering raw data from source systems and writing to ETL environment

## 2. CLEANING & CONFORMING (Subsystems 4-8)
Adding value through quality improvement and enterprise data integration

## 3. DELIVERING (Subsystems 9-21)
Physically structuring and loading data into dimensional models

## 4. MANAGING (Subsystems 22-34)
Orchestrating and controlling the ETL environment

---

# PHASE 1: Round Up the Requirements

Before designing ETL architecture, document requirements across these 10 critical areas:

## 1. Business Needs
- List key performance indicators (KPIs) to be supported
- Document drill-down and drill-across requirements
- Identify analytic requirements driving the design

## 2. Compliance
- List data and reports subject to compliance restrictions
- Document chain of custody requirements
- Define data security and privacy requirements for data under ETL control
- Specify archive retention policies and usable lifetime

## 3. Data Quality
- List data elements with known quality issues
- Document agreements with source systems for data correction
- Identify elements requiring continuous monitoring and flagging

## 4. Security
- Expand compliance checklist for security and privacy
- Define sensitive data elements
- Establish physical backup security requirements

## 5. Data Integration
- Use bus matrix to generate priority list for conformed dimensions
- Annotate each business process with executive demand for integration
- Confirm ETL team agreements on conformance

## 6. Data Latency
- Categorize by frequency: daily, intra-day (micro-batch), or instantaneous
- Document data quality trade-offs for each latency choice
- Annotate with business understanding of these trade-offs

## 7. Archiving and Lineage
- List data sources and intermediate steps to be archived
- Define retention policies
- Document compliance, security, and privacy constraints

## 8. BI Delivery Interfaces
- List all fact and dimension tables exposed to BI tools
- Document OLAP cubes and special database structures required
- List indexes and aggregations for BI performance

## 9. Available Skills
- Inventory staff skills: OS, ETL tools, scripting, programming, SQL, DBMS, OLAP
- Identify skill gaps for current and future systems
- Plan for skill acquisition or constraints

## 10. Legacy Licenses
- List legacy licenses (OS, ETL, scripting, programming, SQL, DBMS, OLAP)
- Document whether use is mandated or recommended
- Identify any problematic legacy constraints

---

# PHASE 2: The 34 ETL Subsystems

## EXTRACTING Subsystems (1-3)

### Subsystem 1: Data Profiling
**Purpose**: Technical analysis of data content, consistency, and structure

**Two Roles**:
- **Strategic**: Early go/no-go assessment of candidate data sources
- **Tactical**: Detailed profiling during modeling/ETL design to identify quality issues

**Key Activities**:
- Column profiling (nulls, ranges, formats, cardinality)
- Structure profiling (hierarchies, relationships)
- Business rule validation
- Establish quality screen specifications

### Subsystem 2: Change Data Capture (CDC)
**Purpose**: Identify changed source data since last update

**CDC Techniques** (evaluate for each source):
- **Audit Columns**: Date/time stamps via triggers or application
- **Timed Extracts**: SYSDATE-based selection (unreliable—avoid)
- **Full Diff Compare**: Snapshot comparison (thorough but resource-intensive)
- **Database Log Scraping**: Extract from redo logs (messy but comprehensive)
- **Message Queue Monitoring**: Capture from transaction streams

**Goals**:
- Capture all changes (deletions, edits, insertions)
- Tag with reason codes (error correction vs. true update)
- Support compliance tracking
- Perform CDC as early as possible

### Subsystem 3: Extract System
**Purpose**: Physical extraction from diverse source systems

**Extract Methods**:
- **File-based**: Extract to file → transfer → transform → load (restartable, verifiable)
- **Stream-based**: Direct source-to-staging flow (faster, less flexible)

**Considerations**:
- COBOL copybooks, EBCDIC/ASCII conversion for mainframe sources
- Native database drivers preferred over ODBC
- Data compression for large/remote transfers
- Data encryption for public networks (compress first, then encrypt)
- XML payload ratio consideration (often <10%)

---

## CLEANING & CONFORMING Subsystems (4-8)

### Subsystem 4: Data Cleansing System
**Purpose**: Marshal technology to support data quality

**Quality Screens** (Three Types):
1. **Column Screens**: Single-field tests (nulls, ranges, formats)
2. **Structure Screens**: Multi-field relationships (hierarchies, FK/PK, addresses)
3. **Business Rule Screens**: Complex validations (thresholds, time-dependent rules)

**Response Options** (when screen fails):
1. **Halt process** (requires manual intervention—least preferred)
2. **Suspense file** (questionable integrity—avoid for minor issues)
3. **Tag and continue** (PREFERRED—maintain data flow integrity)

**Data Quality Culture** (Nine-Step Template):
1. Declare high-level commitment to data quality culture
2. Drive process reengineering at executive level
3. Improve data entry environment
4. Improve application integration
5. Change how processes work
6. Promote end-to-end team awareness
7. Promote interdepartmental cooperation
8. Publicly celebrate data quality excellence
9. Continuously measure and improve

### Subsystem 5: Error Event Schema
**Purpose**: Centralized dimensional schema recording every quality screen error

**Design**:
- **Error Event Fact Table**: Grain = one row per error thrown
- **Dimensions**: Calendar date, batch job, screen that threw error
- **Facts**: Error date/time stamp, time between events
- **Error Event Detail Fact**: Lower grain for multi-field errors

### Subsystem 6: Audit Dimension Assembler
**Purpose**: Attach metadata context to fact table rows at load time

**Typical Attributes**:
- Master ETL version number
- Transformation logic version
- Data quality scores/flags
- Load batch identifier
- Quality categories (passed/problems encountered)

**Usage**: Generate one audit dimension row per batch; attach to all rows loaded in that batch

### Subsystem 7: Deduplication System
**Purpose**: Merge dimension data from multiple sources

**Processes**:
- Match records across sources (identical keys, fuzzy matching)
- **Survivorship**: Combine best-quality columns from matched records
- Maintain back references to all source systems
- Establish business rules for priority sequence of column values

### Subsystem 8: Conforming System
**Purpose**: Align dimension content across the enterprise

**Responsibilities**:
- Combine data from multiple sources into structurally identical format
- Deduplicate and filter invalid data
- Standardize content for conformed image
- Support incremental conformance (start with one attribute, grow over time)
- Interface with Subsystems 17 (Dimension Manager) and 18 (Fact Provider)

---

## DELIVERING Subsystems (9-21)

### Subsystem 9: Slowly Changing Dimension (SCD) Manager
**Purpose**: Handle dimension attribute changes systematically

**SCD Types**:
- **Type 0**: Retain original (never change)
- **Type 1**: Overwrite (no history)
- **Type 2**: Add new row (track history) — PRIMARY TECHNIQUE
- **Type 3**: Add new attribute (previous vs. current)
- **Type 4**: Add mini-dimension (rapidly changing attributes)
- **Type 5**: Mini-dimension + Type 1 outrigger
- **Type 6**: Type 1 + Type 2 attributes combined
- **Type 7**: Dual Type 1 and Type 2 dimensions

**Type 2 Housekeeping Columns**:
1. Row Effective Date/Time
2. Row End Date/Time (default 12/31/9999 for current)
3. Current Flag (current/expired)
4. Change Date (FK to date dimension)
5. Reason for Change (optional)

### Subsystem 10: Surrogate Key Generator
**Purpose**: Generate meaningless integer keys for dimension primary keys

**Best Practices**:
- Independent key sequence per dimension
- Use database sequence generators or ETL tool
- Avoid triggers (performance bottleneck)
- Never concatenate operational key + timestamp (doesn't scale)

### Subsystem 11: Hierarchy Manager
**Purpose**: Populate and maintain dimensional hierarchies

**Hierarchy Types**:
- **Fixed Depth**: Consistent levels (model as separate attributes)
- **Ragged/Variable Depth**: Unbalanced (use bridge table)

**Note**: Snowflakes acceptable in staging area but not presentation layer

### Subsystem 12: Special Dimensions Manager
**Purpose**: Handle unique dimension types

**Special Dimension Types**:
- **Date/Time**: Pre-built from spreadsheet, no conventional source
- **Junk**: Assembled from miscellaneous flags (build on-the-fly or pre-populate)
- **Mini-Dimensions**: Track changes in large dimensions (Type 4 pattern)
- **Shrunken Subset**: Built from base dimension (independent surrogate keys required)
- **Small Static**: Created entirely by ETL (lookup tables)
- **User Maintained**: Business-owned with ETL-provided interface

### Subsystem 13: Fact Table Builders
**Purpose**: Load the three primary fact table types

**Transaction Fact Table**:
- Grain: One row per transaction event
- Load: Bulk insert new rows
- Partitioning: By time for performance
- Late arriving data: Subsystem 16

**Periodic Snapshot Fact Table**:
- Grain: One row per period (day/week/month)
- Load: Insert at period end or update "hot rolling" current period
- Semi-additive facts common (balances)

**Accumulating Snapshot Fact Table**:
- Grain: One row per process lifecycle
- Load: INSERT new + UPDATE existing as process progresses
- Multiple date dimensions for milestones
- Destructive updates common

### Subsystem 14: Surrogate Key Pipeline
**Purpose**: Replace operational natural keys with dimension surrogate keys before fact loading

**Process**:
1. Complete all dimension table updates first
2. Lookup current surrogate key for each natural key
3. Use BETWEEN logic for Type 2 dimensions (effective/end dates)
4. Handle key collisions (data quality screen)
5. Load fact row with referential integrity guaranteed

**Error Handling**:
- Match failure triggers referential integrity error resolution
- Feed back to responsible ETL process
- Use "current row indicator" or effective dates for lookups

### Subsystem 15: Multivalued Dimension Bridge Table Builder
**Purpose**: Link dimensions with multiple values at fact grain

**Challenges**:
- Build unique groups or reuse identical sets
- Handle time variance in bridge table
- Assign weighting factors for proper allocation

### Subsystem 16: Late Arriving Data Handler
**Purpose**: Handle facts/dimensions arriving after expected timeline

**Late Arriving Dimensions**:
- Create dummy dimension row with natural key only
- Type 1 overwrite with complete data when available
- OR assign new surrogate key and update all subsequent fact FK references

**Late Arriving Facts**:
- Search dimension history for correct surrogate key at transaction time
- Adjust semi-additive balances in subsequent fact rows
- Interface with compliance subsystem (changing history)

### Subsystem 17: Dimension Manager System
**Purpose**: Centralized authority for publishing conformed dimensions

**Responsibilities**:
- Implement common descriptive labels agreed by stakeholders
- Add new rows (new source data + Type 2 changes) with new surrogate keys
- Modify in place (Type 1 and Type 3) without changing keys
- Update dimension version number for Type 1/Type 3 changes
- Replicate simultaneously to all fact providers

### Subsystem 18: Fact Provider System
**Purpose**: Receive conformed dimensions and manage fact tables

**Responsibilities**:
- Receive/download replicated dimensions from managers
- Add new fact rows after surrogate key replacement
- Modify fact rows (error correction, accumulating snapshots, late arriving dimensions)
- Remove invalidated aggregates
- Recalculate affected aggregates
- Quality assure base and aggregate tables
- Inform users of database updates

### Subsystem 19: Aggregate Builder
**Purpose**: Build and maintain aggregate fact tables for performance

**Strategies**:
- Incremental update (fastest)
- Drop and rebuild (required for Type 1 dimension changes)
- External sort utility (sometimes faster than DBMS aggregation)

**Critical Rule**: Aggregates MUST be consistent with atomic base data

### Subsystem 20: OLAP Cube Builder
**Purpose**: Load OLAP cubes from dimensional schemas

**Best Practices**:
- Relational dimensional schema is foundation for OLAP
- Load cubes AFTER conventional ETL (leverage RI and cleansing)
- Enforce hierarchy integrity strongly
- Type 2 SCDs fit well (new key = new member)
- Type 1 SCDs problematic (can corrupt/drop cubes)

### Subsystem 21: Data Propagation Manager
**Purpose**: Extract from presentation layer for external consumption

**Use Cases**:
- Share data with business partners/customers/vendors
- Government reporting (Medicare, regulatory)
- Feed package analytic applications
- Load data mining tools

**Approach**: Treat as ETL where target requirements are non-negotiable

---

## MANAGING Subsystems (22-34)

### Subsystem 22: Job Scheduler
**Purpose**: Metadata-driven job control environment

**Capabilities Required**:
- Job definition with execution flow
- Time and event-based scheduling
- Dependency management between jobs
- Metadata capture (what/when/duration)
- Logging to database (not just text files)
- Notification interface to problem escalation

### Subsystem 23: Backup System
**Purpose**: Enable recovery after failures

**Backup Requirements**:
- High performance (fit in time window)
- Simple administration
- Lights-out automation
- Physical backup of database image

**Archive Requirements**:
- Archive to lower-cost media
- Regular refresh and migration
- Long-term compliance (5, 10, 50 years)
- Sunset old systems to vanilla format

### Subsystem 24: Recovery and Restart System
**Purpose**: Resume or back out failed jobs

**Techniques**:
- Checkpoint functionality in ETL tool
- Commit small record sets
- Fact table surrogate key for range-based backup/resume
- Stage data after major pipeline activities
- Modular ETL design for resilience

### Subsystem 25: Version Control System
**Purpose**: Archive and recover all ETL logic and metadata

**Capabilities**:
- Check-out/check-in for all ETL modules
- Source comparison between versions
- Librarian function for complete ETL context
- Master version numbers for overall ETL system

### Subsystem 26: Version Migration System
**Purpose**: Move ETL system between environments

**Environments**: Development → Test → Production

**Requirements**:
- Interface to version control
- Single interface for connection settings
- Rollback capability
- Identical test and production configuration

### Subsystem 27: Workflow Monitor
**Purpose**: Dashboard and reporting for ETL operations

**Monitoring Areas**:
- Job status (pending/running/completed/suspended)
- Performance metrics (records processed, error summaries, actions taken)
- Infrastructure (CPU, memory, disk, database, server utilization)
- Trending over time
- Alerts for anomalies

**Performance Bottleneck Checklist** (ordered by importance):
1. Poorly indexed queries
2. SQL syntax forcing wrong optimizer choice
3. Insufficient RAM causing thrashing
4. Sorting in RDBMS
5. Slow transformation steps
6. Excessive I/O
7. Unnecessary write-then-read cycles
8. Dropping/rebuilding aggregates vs. incremental
9. Late-applied filtering (CDC)
10. Untapped parallelization opportunities
11. Unnecessary transaction logging
12. Network/file transfer overhead

### Subsystem 28: Sorting System
**Purpose**: Provide high-performance sorting capability

**Options**:
- ETL tool sort function
- DBMS SQL SORT clause
- Dedicated sort package (awesomely fast for flat files)

**Uses**: Aggregation, joining flat files, data quality diagnosis

### Subsystem 29: Lineage and Dependency Analyzer
**Purpose**: Track data lineage and impact analysis

**Lineage** (backward): For any data element, identify:
- Ultimate source
- All upstream intermediate tables
- All transformations applied

**Dependency** (forward): For any source element, identify:
- All downstream intermediate tables
- All final BI reports
- All transformations applied

**Use Cases**: Compliance, impact analysis, change management

### Subsystem 30: Problem Escalation System
**Purpose**: Automated notification and resolution tracking

**Escalation Levels**:
1. Help desk (first contact)
2. Systems administrator/DBA (infrastructure)
3. ETL manager (production process)
4. ETL developer (detailed analysis)

**Capabilities**:
- Automatic notifications (email, SMS, operator messages)
- Problem database (type, status, resolution)
- Severity-based routing

### Subsystem 31: Parallelizing/Pipelining System
**Purpose**: Load data within processing window using multiple processors

**Parallelization Opportunities**:
- Extraction (partition on attribute ranges)
- Transformation (automatic in some ETL tools)
- Loading (multiple partitions simultaneously)

**Critical**: Should be automatic unless specific conditions preclude it

### Subsystem 32: Security System
**Purpose**: Protect ETL data, metadata, and processes

**Requirements**:
- Role-based security on all ETL data/metadata
- Historical access records by individual and role
- Data encryption for network transfers
- Controlled production access (not everyone, not nobody)
- Backup media security

### Subsystem 33: Compliance Manager
**Purpose**: Maintain chain of custody for compliance-sensitive data

**Requirements**:
- No destructive changes (Type 1/Type 3 prohibited)
- All changes become inserts with begin/end timestamps
- Fact table surrogate key to track versions
- Change reference (reason) for all modifications
- Source reference for revised data

**Fact Table Compliance Columns**:
1. Fact table surrogate key
2. Begin version date/time
3. End version date/time (default 12/31/9999)
4. Change reference
5. Source reference

### Subsystem 34: Metadata Repository Manager
**Purpose**: Capture and manage ETL metadata

**Metadata Types**:
- **Process**: Job execution, performance, errors
- **Technical**: Data structures, transformations, lineage
- **Business**: Definitions, ownership, quality metrics

**Strategy**: Balance between doing nothing and doing too much; assign metadata manager role

---

# PHASE 3: Develop the ETL Plan

## Step 1: Draw High-Level Plan
Create schematic showing:
- Source systems (boxes representing logical groupings)
- Target dimensional models
- Major transformation stages
- Unresolved issues and questions

**Maintain Two Versions**:
- Simple for external communication
- Detailed for internal DW/BI team

## Step 2: Choose an ETL Tool

**Benefits of ETL Tools**:
- Self-documentation via graphical interface
- Metadata foundation
- Version control
- Advanced transformations (fuzzy matching, data mining)
- Performance optimization at lower expertise level
- Automatic parallelization
- One-step virtualized-to-physical conversion

**Note**: ROI may not appear in first phase (learning curve), but pays off in future phases and maintenance

## Step 3: Develop Default Strategies

Document default approaches for:

**Extract**: Default method per source system (push to file, stream, log read)

**Archive**: Retain extracted/staged data (minimum 1 month, consider permanent)

**Data Quality**: Monitor during ETL (don't wait for user discovery)

**SCD Management**: Default strategy per attribute type

**Availability**: Document when sources available, block out job sequencing

**Data Auditing**: Tag each row with load metadata

**Staging**: Organize staging area, write to disk after major activities

## Step 4: Drill Down by Target Table

For each dimension and fact table:

**Hierarchies**: Investigate cleanliness (best if normalized in source with FK constraints)

**Detailed Schematics**: Document:
- Source systems
- Transformation logic
- Dependencies
- Load sequence

---

# ETL Specification Document

## Required Sections

### Section 1: General Architecture
- Requirements summary (10 areas)
- High-level plan schematic
- ETL tool selection justification
- Default strategies

### Section 2: Per-Table Specifications (2-10 pages each)

**Design**:
- Column names, data types, keys, constraints

**Volumes**:
- Historic load parameters and row counts
- Incremental volumes (new/updated rows per cycle)

**Load Strategy**:
- Load frequency
- SCD handling per attribute
- Partitioning strategy
- Late arriving data approach

**Sources**:
- Overview and unusual characteristics
- Source-to-target mapping
- Data profiling results (min/max, distinct count, NULLs)
- Extract strategy

**Processing**:
- Dependencies (load order)
- Transformation logic (pseudo code/diagrams preferred)
- Preconditions to avoid errors
- Cleanup steps

**Effort**: Estimate (easy/medium/difficult)

---

# Subsystem Selection Guide

## Minimum Viable ETL Architecture

**Every ETL system requires**:
- Subsystems 1, 3 (Profiling, Extract)
- Subsystems 4, 6 (Cleansing, Audit Dimension)
- Subsystems 9, 10, 13, 14 (SCD, Surrogate Keys, Fact Builders, Key Pipeline)
- Subsystems 22, 23, 24, 30 (Scheduler, Backup, Recovery, Problem Escalation)

## Add Based on Requirements

**Incremental Processing**: Subsystem 2 (CDC)

**Multiple Sources**: Subsystems 7, 8 (Deduplication, Conforming)

**Enterprise Integration**: Subsystems 17, 18 (Dimension Manager, Fact Provider)

**Compliance**: Subsystems 5, 29, 33 (Error Event Schema, Lineage, Compliance Manager)

**Performance**: Subsystems 19, 20, 31 (Aggregates, OLAP, Parallelization)

**Large Dimensions**: Subsystem 12 (Special Dimensions - mini, junk)

**Complex Relationships**: Subsystems 11, 15 (Hierarchy Manager, Bridge Tables)

**Real-Time**: Subsystem 2 (advanced CDC), modified pipeline

**External Consumers**: Subsystem 21 (Data Propagation)

---

# Critical Success Factors

1. **Structured Approach**: Use the 34 subsystems framework—avoid spaghetti
2. **Requirements First**: Complete the 10-area requirements roundup before designing
3. **Data Profiling**: Invest heavily—strategic AND tactical
4. **Quality Culture**: Technical solutions alone won't fix broken business processes
5. **ETL Tool**: Use commercial tools for maintainability and scalability
6. **Incremental Development**: Build one business process at a time
7. **Reliability**: ETL must consistently run to completion
8. **Availability**: Meet service level agreements
9. **Manageability**: Gracefully evolve as business changes
10. **Documentation**: Maintain current ETL specifications

---

# Common Mistakes to Avoid

**Don't**:
- Skip data profiling (causes late project failures)
- Use timed extracts for CDC (unreliable)
- Halt process for minor quality issues (tag and continue instead)
- Use suspense files (questionable integrity)
- Generate surrogate keys via triggers (performance killer)
- Snowflake in presentation layer
- Load Type 1 changes into OLAP cubes without rebuilding
- Use UPDATE else INSERT (performance killer)
- Skip version control
- Ignore workflow monitoring
- Build custom solutions for commodity capabilities
- Confuse "real-time requirements" with actual business needs

---

# Real-Time Considerations

If data latency requirements demand micro-batch or streaming:

**Triage Requirements**:
- **Daily**: Batch file, full ETL pipeline, reconciled data (RECOMMENDED)
- **Intra-day**: Micro-batch, message queues, reduced quality screens
- **Instantaneous**: EII solution, no caching, limited complexity

**Architecture Trade-Offs**:
- Replace batch files → raw message queues (dirty data)
- Limit quality screens → column only (no structure/business rules)
- Post facts with old dimensions → provisional context
- Eliminate staging → compliance/recovery risks

**Data Quality Impact**: As latency decreases, quality decreases. Business users must understand trade-offs.

---

# Next Steps After Architecture Design

1. Develop ETL specification document
2. Create development sandbox source systems
3. Begin historic load development (one-time)
4. Develop incremental load processing (ongoing)
5. Implement aggregate and OLAP loads
6. Deploy to production with full automation

This architecture provides the foundation for all subsequent ETL development work.
