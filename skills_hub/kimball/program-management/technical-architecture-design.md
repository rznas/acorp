# Technical Architecture Design - Detailed Reference

## Purpose

The technical architecture serves as the blueprint for the DW/BI environment's technical services and infrastructure, much like housing blueprints enable contractors to build a home. It supports integration of technologies and applications while enabling communication across stakeholders.

---

## Architecture Benefits

<architecture_value>
**Integration Framework:**
- Organizes and integrates multiple technologies and applications
- Coordinates parallel development efforts
- Enables modular component reuse

**Problem Prevention:**
- Catches problems on paper before mid-project surprises
- Identifies immediately required vs future components
- Minimizes rework through upfront planning

**Communication Tool:**
- Technical team internal communication
- Upward communication to management
- Outward communication to vendors
- Consistent technical requirements across all parties
</architecture_value>

**Architecture Analogy**: Like home construction blueprints enabling architect, general contractor, subcontractors, and homeowner to communicate from common document.

---

## The Architecture Spectrum Problem

Teams typically approach architecture from opposite extremes—both unhealthy:

<architecture_extremes>
**Too Little Architecture (Bypass Approach):**
- **Mindset**: Architecture feels nebulous, impediment to progress
- **Action**: Skip architecture design, focus only on delivery
- **Result**: Piece together first iteration with "chewing gum and bailing wire"
- **Consequence**: Integration/interfaces taxed as more data, users, functionality added
- **Outcome**: Eventually rebuild because non-architected structure couldn't withstand stress

**Too Much Architecture (Analysis Paralysis):**
- **Mindset**: Architecture must address every plausible technical challenge
- **Action**: Invest two years designing perfect architecture
- **Result**: Forget primary purpose is solving business problems
- **Consequence**: Missed business opportunities while designing
- **Outcome**: Architecture becomes academic exercise disconnected from business needs
</architecture_extremes>

**Healthy Middle Ground**: Every DW/BI system has technical architecture—question is whether it's planned and explicit or merely implicit. Eight-step process navigates middle path.

---

## Eight-Step Architecture Design Process

### Step 1: Establish Architecture Task Force

**Task Force Composition:**
- **Size**: 2-3 people (small and focused)
- **Roles**:
  - Technical architect (lead)
  - ETL architect/designer (back room representation)
  - BI application architect/designer (front room representation)

**Rationale**: Small team ensures both ETL and BI perspectives represented while maintaining agility.

---

### Step 2: Collect Architecture-Related Requirements

**Primary Input: Business Requirements**

Architecture supports business needs, not technology for technology's sake. Filter business requirements for architectural implications:

<architectural_filter>
**Timing Requirements:**
- "We need sales data updated hourly" → Real-time ETL architecture, partition strategy
- "Reports must be available by 7 AM" → ETL processing window constraints
- "Month-end close requires 24-hour turnaround" → Capacity planning, parallel processing

**Availability Requirements:**
- "Global users across all time zones" → 24/7 availability, data mirroring for loads
- "Must support disaster recovery" → Backup/restore architecture, replication

**Performance Requirements:**
- "Sub-second dashboard response" → In-memory caching, OLAP cubes, aggregates
- "Support 5,000 concurrent users" → Load balancing, query optimization, indexing strategy
- "Process 10 TB nightly" → ETL scalability, parallel processing, hardware capacity

**Security Requirements:**
- "Row-level security by sales territory" → Security architecture, access control
- "HIPAA compliance required" → Audit logging, encryption, data masking
</architectural_filter>

**Secondary Input: IT Organization Interviews**

Conduct technology-focused sessions to understand:
- Current IT standards (platforms, databases, ETL tools, BI tools)
- Planned technical directions (cloud migration, platform consolidation)
- Non-negotiable boundaries (security policies, vendor restrictions)
- Lessons learned from prior information delivery projects
- Willingness to accommodate operational change for DW/BI (e.g., CDC in source systems)

**Deliverable**: Architecture requirements interview notes

---

### Step 3: Document Architecture Requirements

**Format**: Simple tabular documentation

| Business Requirement | Architectural Implications |
|---------------------|---------------------------|
| Deliver global sales performance data on nightly basis | • 24/7 worldwide availability<br>• Data mirroring for loads<br>• Robust metadata for global access<br>• Adequate network bandwidth<br>• Sufficient ETL horsepower for complex integration |
| Support 2,000 concurrent users with sub-5-second response | • Load balancing architecture<br>• Query result caching<br>• Aggregate table strategy<br>• OLAP cubes for frequent queries<br>• Connection pooling |
| Row-level security by customer account | • Security dimension in fact tables<br>• Security views or database roles<br>• BI tool security integration<br>• Audit logging for compliance |

**Purpose**: Creates clear traceability from business needs to technical solutions.

**Deliverable**: Architecture requirements matrix document

---

### Step 4: Create the Architecture Model

**Process:**
1. Architecture team sequesters in conference room for several days of intensive design
2. Group architecture requirements into major components
3. Draft and refine high-level architectural model
4. Create detailed component diagrams

**Major Architecture Components:**

<architecture_components>
**ETL Services:**
- Data extraction (CDC, full extracts, APIs)
- Data staging and transformation
- Data quality and cleansing
- Dimension management (SCD processing)
- Fact table loading
- Aggregate building
- ETL orchestration and scheduling
- Error handling and recovery

**Data Storage Services:**
- Staging area (transient)
- Integration area (optional, for persistent staging)
- Presentation area (dimensional models)
- Aggregate storage (OLAP cubes or relational)
- Metadata repository

**BI Services:**
- Ad-hoc query tools
- Report development and delivery
- Dashboard and scorecard platforms
- OLAP analysis tools
- Data mining and advanced analytics
- Mobile BI access

**Metadata Services:**
- Technical metadata (lineage, dependencies)
- Business metadata (definitions, ownership)
- Process metadata (ETL execution logs)
- Metadata repository and catalog

**Infrastructure Services:**
- Database servers (development, test, production)
- ETL processing servers
- BI application servers
- Web servers for report delivery
- Network and storage infrastructure
- Backup and disaster recovery
- Monitoring and alerting
</architecture_components>

**High-Level Architecture Diagram:**

The diagram illustrates architecture from "street view"—what it looks like at high level. BUT it's dangerously simplistic because significant details embedded in component diagrams that follow.

**Warning**: Like home blueprints' front elevation page—shows overall appearance but critical details in subsequent pages.

**Deliverable**: High-level architecture diagram and component diagrams

---

### Step 5: Determine Architecture Implementation Phases

**Prioritization Approach:**

Like homeowner's dream house, can't implement everything at once. Establish priorities:

<implementation_phases>
**Phase 0: Mandatory Foundation (Initial Project)**
- Capabilities absolutely required for first business process delivery
- Non-negotiable for project success
- Example components:
  - Core ETL extraction, transformation, loading
  - Dimensional database (staging + presentation)
  - Basic BI query and reporting tool
  - Initial metadata capture
  - Development/test/production environments

**Phase 1: High-Value Quick Wins (First 6 Months)**
- Significant business value with reasonable effort
- Build on foundation without major rework
- Example components:
  - Advanced BI dashboards
  - Email-based report distribution
  - Basic data quality monitoring
  - Incremental ETL automation

**Phase 2: Advanced Capabilities (6-12 Months)**
- Enhanced functionality as user adoption grows
- More sophisticated technical capabilities
- Example components:
  - OLAP cubes for performance
  - Advanced data quality framework
  - Real-time data feeds
  - Mobile BI access
  - Comprehensive metadata catalog

**Phase 3: Nice-to-Haves (Future)**
- Defer until clear business demand emerges
- May become mandatory as business evolves
- Example components:
  - Advanced analytics (predictive, ML)
  - Embedded BI in operational apps
  - Data virtualization layer
  - Master data management integration
</implementation_phases>

**Critical**: Refer back to business requirements to establish priorities. Must minimally provide architectural elements needed to deliver initial project.

**Deliverable**: Phased architecture implementation roadmap

---

### Step 6: Design and Specify Subsystems

**Standard vs Custom Components:**

- **Standard functionality** (majority): Vendor standard offerings (ETL tools, databases, BI tools)
- **Custom subsystems**: Specialized needs not met by off-the-shelf products

**Custom Subsystem Design Requirements:**

For each custom subsystem, document in sufficient detail for build-vs-buy evaluation:

<subsystem_specification>
**Subsystem Name**: [e.g., Real-Time Event Processing]

**Purpose**: [What business problem does it solve?]

**Functional Requirements:**
- Input sources and data formats
- Processing logic and transformations
- Output destinations and formats
- Performance requirements (throughput, latency)

**Technical Requirements:**
- Platform and technology constraints
- Integration points with existing systems
- Scalability and availability needs
- Security and compliance requirements

**Acceptance Criteria:**
- Measurable success metrics
- Test scenarios and expected results

**Build vs Buy Decision Factors:**
- Estimated custom development effort
- Available commercial products
- Total cost of ownership comparison
</subsystem_specification>

**Common Custom Subsystems:**
- Specialized data extraction from legacy systems
- Industry-specific transformation logic
- Custom data quality rules engine
- Real-time event stream processing
- Complex hierarchical dimension builder
- Custom security and access control
- Audit and lineage tracking system

**Deliverable**: Custom subsystem specifications

---

### Step 7: Create the Architecture Plan Document

**Document Structure:**

<architecture_plan_document>
**1. Executive Summary**
- DW/BI initiative overview
- Architecture purpose and scope
- Key architectural decisions
- Implementation approach

**2. Business Requirements Summary**
- Key business drivers
- Architectural implications
- Requirements traceability matrix

**3. Architecture Principles**
- Guiding design principles
- Technology standards
- Integration philosophy
- Scalability approach
- Security model

**4. High-Level Architecture Model**
- Overall architecture diagram
- Major component overview
- Data flow illustration
- Integration points

**5. Component Specifications**
For each major component (ETL, Data Storage, BI, Metadata, Infrastructure):
- Component purpose and capabilities
- Subcomponent details
- Interfaces and integration
- Technology recommendations (generic, not specific products)

**6. Custom Subsystems**
- Detailed specifications for custom-built components
- Build vs buy justification
- Integration with standard components

**7. Implementation Phases**
- Phased rollout plan
- Phase 0 (mandatory foundation)
- Subsequent phases with dependencies
- Timeline estimates

**8. Operational Considerations**
- Backup and recovery strategy
- Monitoring and alerting approach
- Performance management
- Security and access control
- Disaster recovery

**9. Open Issues and Risks**
- Unresolved architectural questions
- Technology risks and mitigation
- Dependency risks
</architecture_plan_document>

**Document Characteristics:**
- Adequate detail for skilled professionals to proceed with construction
- Much like carpenters frame house based on blueprint
- Typically does NOT reference specific products (exception: already in-house)
- Product-agnostic until product selection phase

**Deliverable**: Complete technical architecture plan document

---

### Step 8: Review and Finalize Technical Architecture

**Review Audiences:**

<review_strategy>
**Audience 1: Project Team**
- **Purpose**: Detailed understanding for implementation
- **Content**: Full architecture plan with deep dive into each component
- **Format**: Working sessions (2-3 hours)
- **Outcome**: Team alignment, identified gaps, implementation assignments

**Audience 2: IT Colleagues (Broader IT Organization)**
- **Purpose**: IT coordination and standards alignment
- **Content**: High-level model + components impacting IT (infrastructure, security, operations)
- **Format**: 1-hour presentation with Q&A
- **Outcome**: IT buy-in, identified dependencies, operational commitments

**Audience 3: Business Leads (Sponsors and Drivers)**
- **Purpose**: Business alignment and investment justification
- **Content**: Business requirements → architecture approach → phased implementation
- **Format**: 30-minute executive briefing
- **Outcome**: Business approval, confirmed priorities, funding commitment
</review_strategy>

**Review Process:**
1. Schedule reviews with appropriate audiences
2. Tailor content and depth to audience needs
3. Solicit feedback and capture issues
4. Update architecture plan based on feedback
5. Obtain final approvals from key stakeholders

**Post-Review Activities:**
- Update and finalize architecture plan document
- Incorporate review feedback and decisions
- Distribute final architecture plan to project team
- **Immediate next use**: Product selection process (architecture plan as shopping list)

**Deliverable**: Finalized and approved technical architecture plan

---

## Architecture Components - Detailed Breakdown

### ETL Services Architecture

<etl_architecture>
**Data Extraction Layer:**
- Change data capture mechanisms (log scraping, timestamps, message queues)
- Full extract processes for periodic loads
- API-based extraction for cloud sources
- Flat file and database extraction
- Data profiling and source analysis

**Staging and Transformation Layer:**
- Transient staging area (landing zone)
- Persistent staging area (optional, for audit and reprocess)
- Data cleansing and standardization
- Business rule application
- Data type conversion and formatting
- Deduplication and matching

**Integration and Conforming Layer:**
- Master data management integration
- Conformed dimension creation
- Cross-source data integration
- Survivorship and golden record processing
- Data quality error handling

**Delivery Layer:**
- Dimension table processing (SCD Types 0-7)
- Fact table loading (transaction, snapshot, accumulating)
- Aggregate table building
- OLAP cube loading
- Late-arriving dimension and fact handling

**Management and Operations Layer:**
- Job scheduling and orchestration
- Error handling and recovery
- Workflow monitoring and alerting
- Backup and restart capabilities
- Performance monitoring
- Metadata capture (lineage, statistics)
</etl_architecture>

**ETL Architecture Decision Points:**
- Batch vs real-time vs micro-batch processing
- ETL tool vs hand-coded vs hybrid approach
- Persistent staging vs transient staging
- Push vs pull extraction patterns
- Parallel processing strategy

---

### Data Storage Architecture

<storage_architecture>
**Staging Area:**
- Purpose: Transient landing zone for source data extracts
- Characteristics: Simple relational tables mirroring source structure
- Retention: Short-term (days to weeks)
- Technology: Relational database (same as presentation or separate)

**Integration Area (Optional):**
- Purpose: Persistent staging for audit trail and reprocessing
- Characteristics: Normalized or source-aligned structure
- Retention: Long-term (months to years)
- Use cases: Regulatory compliance, ETL reprocessing, data science exploration

**Presentation Area:**
- Purpose: Dimensional models for BI query and analysis
- Characteristics: Star schemas with fact and dimension tables
- Retention: Historical data per business requirements (often years)
- Technology: Relational database or columnar database

**Aggregate Storage:**
- Purpose: Pre-calculated summaries for performance
- Characteristics: Relational aggregate tables or OLAP cubes
- Technology: Same RDBMS, OLAP database (SSAS, Oracle OLAP), or in-memory (SAP HANA)

**Metadata Repository:**
- Purpose: Technical, business, and process metadata storage
- Characteristics: Relational tables or specialized metadata database
- Technology: Custom database, ETL tool repository, or metadata management platform
</storage_architecture>

**Storage Architecture Decision Points:**
- Relational vs columnar database for presentation
- Star schema vs OLAP cubes (or hybrid)
- Integration area inclusion (compliance, reprocessing needs)
- Aggregate strategy (relational, OLAP, or both)
- Partitioning approach (date-based, geographic, organizational)

---

### BI Services Architecture

<bi_architecture>
**Ad-Hoc Query and Analysis:**
- Desktop query tools (SQL clients, OLAP clients)
- Web-based query builders
- Self-service data preparation tools

**Report Development and Delivery:**
- Enterprise reporting platform (SSRS, Crystal, Cognos)
- Pixel-perfect formatted reports
- Parameterized report templates
- Scheduled report distribution (email, portal)

**Dashboard and Scorecard Platforms:**
- Executive dashboards
- Operational scorecards
- KPI monitoring
- Drill-down and drill-across navigation

**OLAP Analysis Tools:**
- Multidimensional analysis clients
- Pivot table and slice-dice interfaces
- What-if scenario modeling

**Advanced Analytics:**
- Data mining and pattern discovery
- Predictive analytics and forecasting
- Statistical analysis tools
- Machine learning model deployment

**Mobile BI:**
- Mobile-optimized dashboards
- Responsive web design
- Native mobile apps
- Offline data access

**Embedded BI:**
- BI content embedded in operational applications
- APIs for programmatic access
- White-labeled BI for external users
</bi_architecture>

**BI Architecture Decision Points:**
- Suite approach (single vendor) vs best-of-breed (multiple vendors)
- Proprietary tools vs open-source
- Server-based vs desktop-based
- Licensing model (named user, concurrent, consumption)

---

### Infrastructure Architecture

<infrastructure_architecture>
**Server Architecture:**
- Development environment (isolated for development and unit testing)
- Test/QA environment (mirrors production for validation)
- Production environment (operational DW/BI system)
- Disaster recovery environment (optional, for business continuity)

**Network Architecture:**
- Internal network connectivity (users to DW/BI servers)
- Source system connectivity (operational systems to ETL servers)
- External connectivity (cloud sources, external data providers)
- Bandwidth planning for data movement and user query traffic

**Storage Architecture:**
- Database storage (SAN, NAS, or local direct-attached)
- Backup storage (disk, tape, or cloud)
- Archive storage (long-term historical data)
- Performance tiers (SSD for hot data, spinning disk for warm, tape/cloud for cold)

**Backup and Disaster Recovery:**
- Database backup strategy (full, differential, transaction log)
- ETL process backup (code, metadata, control files)
- Disaster recovery plan (RPO, RTO objectives)
- Geographic redundancy (if required)

**Monitoring and Alerting:**
- Infrastructure monitoring (server health, disk space, CPU, memory)
- Database monitoring (query performance, blocking, deadlocks)
- ETL monitoring (job success/failure, data volume, processing time)
- BI usage monitoring (user activity, report execution, query patterns)
- Alerting and escalation (email, SMS, dashboards)

**Security Architecture:**
- Authentication (Active Directory, LDAP, SSO)
- Authorization (database roles, BI tool security, row-level security)
- Network security (firewalls, VPNs, DMZs)
- Data encryption (at rest, in transit)
- Audit logging and compliance
</infrastructure_architecture>

---

## Architecture Anti-Patterns to Avoid

<anti_patterns>
**1. Technology-Driven Architecture**
- **Symptom**: Architecture designed around available products rather than business needs
- **Problem**: Solution in search of a problem
- **Fix**: Start with business requirements; technology serves business, not vice versa

**2. Analysis Paralysis**
- **Symptom**: Months spent designing perfect architecture addressing every plausible scenario
- **Problem**: Forget primary purpose is solving business problems
- **Fix**: Focus on Phase 0 mandatory foundation; defer nice-to-haves until business demand emerges

**3. Bypass Architecture**
- **Symptom**: Jump directly to product selection and implementation without architecture design
- **Problem**: Chewing gum and bailing wire integration that can't scale
- **Fix**: Invest 3-4 weeks in architecture design to avoid costly rebuild later

**4. Ivory Tower Architecture**
- **Symptom**: Architecture team sequesters for months, emerges with 200-page document no one reads
- **Problem**: Disconnected from project reality and business timeline
- **Fix**: Iterative architecture with frequent project team and business reviews

**5. Product-Specific Architecture**
- **Symptom**: Architecture plan specifies vendor products before evaluation
- **Problem**: Limits options, creates vendor lock-in, may not fit needs
- **Fix**: Product-agnostic architecture plan; product selection is separate subsequent step

**6. Ignoring Operational Realities**
- **Symptom**: Architecture with no consideration for backup, monitoring, security, disaster recovery
- **Problem**: Operational nightmare post-deployment
- **Fix**: Include operational architecture components (Step 6 custom subsystems)

**7. One-Size-Fits-All**
- **Symptom**: Apply same architecture to every project regardless of requirements
- **Problem**: Over-architected small projects, under-architected large projects
- **Fix**: Architecture tailored to business requirements and organizational readiness

**8. Ignoring Legacy and Constraints**
- **Symptom**: Greenfield architecture ignoring existing systems, standards, and commitments
- **Problem**: Can't implement due to organizational/technical constraints
- **Fix**: Collect IT requirements in Step 2 (standards, directions, boundaries)
</anti_patterns>

---

## Architecture Maturity Model

Organizations evolve through architecture maturity stages:

<maturity_model>
**Level 1: Ad Hoc**
- No explicit architecture
- Point solutions without integration plan
- Technology decisions made project-by-project
- **Risk**: Technical debt, costly rebuilds, data silos

**Level 2: Repeatable**
- Basic architecture documentation
- Some technology standards emerging
- Architecture created reactively per project
- **Risk**: Inconsistency across projects, limited reuse

**Level 3: Defined**
- Enterprise architecture group established
- Documented architecture standards and patterns
- Architecture design standard part of project lifecycle
- **Risk**: Architecture may become bureaucratic overhead if not pragmatic

**Level 4: Managed**
- Architecture actively managed as program asset
- Regular architecture reviews and updates
- Metrics tracked (reuse, adherence, value delivered)
- **Risk**: Over-focus on process vs business value

**Level 5: Optimizing**
- Architecture continuously improved based on lessons learned
- Proactive architecture evolution anticipating business needs
- Balance between standards and innovation
- **Success**: Architecture enables business agility and DW/BI value
</maturity_model>

**Target State**: Most organizations benefit from Level 3-4 (defined and managed). Level 5 optional for large enterprises with mature DW/BI programs.

---

## Summary

The technical architecture design process:
1. Starts with business requirements (not technology)
2. Balances structure with pragmatism (middle ground between bypass and analysis paralysis)
3. Provides integration framework for phased implementation
4. Serves as communication tool across stakeholders
5. Enables informed product selection (architecture as shopping list)

**Critical Success Factors:**
- Small, cross-functional architecture task force (2-3 people)
- Business requirements drive architecture, not vice versa
- Phased implementation (Phase 0 mandatory, defer nice-to-haves)
- Document at appropriate detail level (enables construction, not encyclopedic)
- Review with project team, IT, and business leads before finalizing
- Immediate use in product selection process

**Time Investment**: 3-4 weeks for architecture design process. Time well spent to avoid costly rebuilds later.
