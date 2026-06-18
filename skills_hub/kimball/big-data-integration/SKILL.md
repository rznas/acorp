---
name: big-data-integration
description: Apply dimensional modeling principles to big data environments (Hadoop/MapReduce, extended RDBMS). Use when integrating structured/unstructured data, building data highways with multiple latency caches, implementing streaming analytics, or combining conventional DW with big data platforms for comprehensive enterprise analytics.
---

# Big Data Integration with Dimensional Modeling

## When to Use This Skill

Use this skill when:
- **Integrating big data platforms** with conventional data warehouses
- **Working with Hadoop/MapReduce** or extended RDBMS architectures for big data
- **Processing unstructured or semistructured data** (text, images, social media, clickstreams, IoT sensor data)
- **Building multi-latency data pipelines** from real-time to historical analytics
- **Implementing streaming analytics** with continuous query processing
- **Combining structured and unstructured data** in unified analytic ecosystems
- **Scaling to petabyte-level data volumes** with distributed processing
- **Extracting facts from unstructured sources** (sentiment analysis, document mining, image recognition)

## Core Concept: Big Data as a Paradigm Shift

Big data is not just about volume—it's a paradigm shift in:
- **Data variety**: Structured, semistructured, unstructured, raw formats
- **Data velocity**: Real-time streaming, high-speed ingestion (GB/sec)
- **Analysis complexity**: User-defined functions (UDFs), machine learning, complex event processing
- **Storage approach**: Schema-on-read vs schema-on-write
- **Processing model**: Distributed computing across thousands of nodes

**Critical Principle**: Dimensional modeling principles (dimensions, facts, conformed dimensions, SCDs, surrogate keys) remain valid and essential for big data integration with enterprise DW.

## Two Big Data Architecture Approaches

### 1. Extended RDBMS Architecture

**What It Is**: Traditional relational databases extended to handle new big data types.

**Characteristics**:
- Extends relational data types to include vectors, matrices, hyperstructured data, images, video, name-value pairs
- Processes new data types in-database via user-defined functions (UDFs)
- Uses RDBMS as fact extractor: first pass extracts facts from raw data, second pass loads extracted facts as conventional rows/columns
- Leverages existing SQL skills and RDBMS infrastructure
- Provides mature transaction support, indexing, query optimization

**When to Choose**:
- Data can be structured into relational format
- SQL query performance is critical
- Strong ACID transaction requirements
- Team has deep RDBMS expertise
- Existing RDBMS infrastructure investment

### 2. MapReduce/Hadoop Architecture

**What It Is**: Open-source distributed processing framework (Apache Hadoop) for massive-scale data processing.

**Characteristics**:
- Runs on Hadoop Distributed File System (HDFS) or cloud storage (Amazon S3)
- Processes data via MapReduce jobs (UDF execution framework)
- Schema-on-read: load data before declaring structure
- Highly distributed across commodity hardware
- Can process data in original formats without transformation
- Flexible ETL environment with Hive, Pig, HBase, MapReduce code

**When to Choose**:
- Data is unstructured or highly variable
- Schema is unknown or rapidly changing
- Petabyte-scale storage required
- Cost-effective commodity hardware preferred
- Flexible prototyping and experimentation needed

**Coexistence Pattern**: Both architectures have complementary strengths and likely will coexist. Use Hadoop for ETL/staging/exploration, then load structured results into RDBMS for production analytics.

See `architecture-comparison.md` for detailed decision framework.

## Data Highway Architecture: Multiple Latency Caches

Design a logical data highway with up to five caches of increasing latency and data quality:

### Cache Architecture

```
[Raw Source] → [Real-Time] → [Business Activity] → [Top Line] → [Data Warehouse]
   <1 sec        <10 sec        <1 hour              <24 hour      Historical
   Dirtiest                                                         Cleanest
```

### Five Data Caches

<cache_definitions>

**1. Raw Source Applications (Immediate, <1 second)**
- **Use Cases**: Credit card fraud detection, network attack detection, complex event processing (CEP)
- **Data Quality**: Dirtiest—no time for comprehensive cleansing
- **Processing**: Field-level validation only, immediate threshold checks
- **Storage**: Often in-memory or streaming buffers

**2. Real-Time Applications (Near-immediate, <10 seconds)**
- **Use Cases**: Web ad selection, personalized pricing, online game monitoring
- **Data Quality**: Basic cleansing, structural validation possible
- **Processing**: Simple business rules, single-source validation
- **Storage**: High-speed cache (Redis, Memcached) or HDFS

**3. Business Activity Applications (Low latency, <1 hour)**
- **Use Cases**: KPI dashboards, trouble ticket tracking, process monitoring, customer service portals
- **Data Quality**: Cross-source validation, relationship checks
- **Processing**: Multi-source integration, complex business rules
- **Storage**: Operational Data Store (ODS) style

**4. Top Line Applications (Daily, <24 hours)**
- **Use Cases**: Tactical reporting, promotion tracking, midcourse corrections, social media buzz analysis
- **Data Quality**: Complete transaction sets, repudiated transactions removed
- **Processing**: Full cleansing, conforming, matching
- **Storage**: Daily snapshot fact tables

**5. Data Warehouse and Long Time Series (Historical, days to years)**
- **Use Cases**: Historical analysis, master data management, temporal dynamics, Markov chain analysis
- **Data Quality**: Highest—comprehensive cleansing, governance, conformance
- **Processing**: Full dimensional modeling, conformed dimensions, SCD tracking
- **Storage**: Enterprise data warehouse with full history

</cache_definitions>

### Data Highway Flow Patterns

**Forward Flow (Source → Warehouse)**:
- ETL processes move data down the highway
- Each cache adds value: filtering, cleansing, conforming, diagnosing
- Data quality improves at each stage
- Processing time increases, latency increases

**Backflow (Warehouse → Earlier Caches)**:
- Push conformed dimensions (customer, product, date) with durable surrogate keys back to earlier caches
- Transfer derived metrics (historical summaries, data mining scores) to real-time caches
- Enable upstream applications to join with rich DW content using surrogate keys
- **Job One**: Replace proprietary source keys with durable surrogate keys at each extraction step

See `data-highway-patterns.md` for implementation details and cache selection guidance.

## Dimensional Modeling Best Practices for Big Data

### 1. Think Dimensionally: Divide the World into Dimensions and Facts

**Apply to All Data Types**: Even unstructured data has associated dimensions.

**Example—Twitter Tweet Dimensionalization**:
- Raw tweet: "Wow! That is awesome!"
- **Extracted dimensions**: Customer, location, product/service, marketplace condition, provider, weather, cohort group, session, triggering event, final outcome
- **Process**: Automated dimensionalizing required for high-velocity streams
- **Timing**: Dimensionalize at earliest extraction step, as close to real-time as possible

**Business Value**: Dimensions make data queryable, filterable, and understandable to business users regardless of underlying format.

### 2. Integrate with Conformed Dimensions

**Most Powerful Best Practice**: Conformed dimensions are the glue holding together separate big data sources.

**How It Works**:
1. Identify variations of same dimension across data sources (e.g., customer dimension in CRM, sales, support, social media)
2. Define one or more **enterprise attributes** present in all variations (e.g., customer demographic category)
3. Embed enterprise attributes in all dimension variations, even at different grains
4. Enable drill-across queries via sort-merge on enterprise attributes after separate queries

**Implementation**:
- Incremental, agile, non-disruptive rollout
- Existing applications continue unchanged
- Big data sources (Hadoop, NoSQL) can participate via enterprise attributes

**See also**: `conformed-dimension-management` skill for detailed conformance process.

### 3. Anchor Dimensions with Durable Surrogate Keys

**Critical for Big Data**: Natural keys from sources are incompatible, poorly administered, controlled by others.

**Durable Surrogate Key Characteristics**:
- **Durable**: No business rule can change the key (belongs to DW/BI system, not source)
- **Surrogate**: Simple integers assigned sequentially or via robust hashing algorithm
- **No content**: Just an identifier with no application semantics

**Big Data Implementation**:
- **Job One on every extraction**: Embed durable surrogate keys for customer, product, date, location
- **Enables backflow**: Push data backward down data highway using surrogate keys for joins
- **Supports schema-on-read**: Attach surrogate keys to name-value pairs even before structure is known

**Example**: Medical study combines conventional DW data + images (X-rays, EKGs) + physician notes + social media sentiments + cohort linkages—all joined on patient surrogate key.

### 4. Use Slowly Changing Dimensions (SCDs)

Track time variance in big data dimensions using SCD techniques (Types 0-7).

**Why It Matters in Big Data**:
- IoT devices change location, configuration, ownership over time
- Social media user profiles evolve (demographics, interests, influence)
- Product hierarchies change (categories, brands, suppliers)
- Customer 360 views require tracking relationship changes across time

**Implementation**: Apply same SCD patterns from conventional DW (see `fact-table-design-patterns` and Chapter 5 for SCD details).

### 5. Declare Data Structure at Analysis Time (Schema-on-Read)

**Big Data Advantage**: Load data into Hadoop/data grid without declaring structure upfront.

**Benefits**:
- Data structures may be unknown at load time
- Variable content doesn't fit single structure
- Avoid resource-intensive schema design before understanding data
- Different analysts can view same data in different ways

**Trade-offs**:
- **Penalty**: Data without declared structure may be difficult to index for rapid access (unlike RDBMS)
- **Mitigation**: Most big data algorithms process entire data sets without filtering subsets
- **Hybrid approach**: Use Hadoop for exploration, then transfer to RDBMS with declared structure for production

**ETL Value**: Transfer from Hadoop name-value pairs to RDBMS named columns is a valuable ETL step adding structure and governance.

### 6. Load Data as Name-Value Pairs

**Handle Unexpected Content**: Big data sources contain surprises—undocumented or variable content arriving at GB/sec.

**Name-Value Pair Pattern**:
- Load data gracefully even when encountering unexpected attributes
- Example: Financial asset disclosure includes "rare postage stamp = $10,000"—loaded successfully even if never seen before
- Defer structure declaration until analysis time
- MapReduce frameworks often require name-value pair format

**Workflow**:
1. Load raw data as name-value pairs into Hadoop/data grid
2. Dimensionalize: parse, match, apply identity resolution
3. Transform to structured format for RDBMS or leave in Hadoop for specialized analytics

### 7. Rapidly Prototype Using Data Virtualization

**Data Virtualization**: Present different logical data structures on underlying physical data (e.g., SQL views).

**Benefits**:
- Rapid prototyping of data structures
- Quick schema alterations or alternatives
- Test multiple dimensional models on same physical data

**Trade-offs**:
- **Runtime compute cost** vs **ETL cost** to build physical tables
- **Best practice**: Prototype with virtual schemas, then materialize when vetted and performance is needed

## Big Data as Fact Extractor

Use big data analytics to extract structured facts from unstructured sources, then feed to next cache or DW.

### Fact Extraction Examples

**Unstructured Text (Twitter Sentiment Analysis)**:
- **Input**: Raw tweets (unstructured text)
- **Extracted facts**: Share of voice, audience engagement, conversation reach, active advocates, advocate influence, advocacy impact, resolution rate, resolution time, satisfaction score, topic trends, sentiment ratio, idea impact
- **Output**: Trendable numerical measures suitable for fact tables

**Image Analysis**:
- **Input**: Product photos, medical images, satellite imagery
- **Extracted facts**: Object counts, quality scores, anomaly flags, similarity scores
- **Output**: Numeric facts joinable to product, patient, location dimensions

**Clickstream Analysis**:
- **Input**: Web server logs, application event streams
- **Extracted facts**: Page views, session duration, conversion events, abandonment points
- **Output**: Session-level and event-level fact tables (see `industry-retail-patterns` skill)

**IoT Sensor Data**:
- **Input**: Continuous sensor readings (temperature, vibration, pressure)
- **Extracted facts**: Threshold violations, anomaly scores, predictive maintenance indicators
- **Output**: Periodic snapshot or transaction fact tables

## Streaming Data Analytics

Implement streaming analytics to begin analysis as data flows in, before transfer process terminates.

### Streaming Use Cases

- **Financial fraud**: Detect suspicious patterns in real-time transaction streams
- **Network security**: Identify cyber attacks as packets arrive
- **Complex Event Processing (CEP)**: Correlate multiple event streams for pattern detection
- **Ad targeting**: Select personalized ads based on streaming clickstream

### Continuous Query Language (CQL) Concepts

- **SQL-like queries** on streaming data
- **Dynamic time windows**: Moving windows on streaming data (last 5 minutes, last 1000 events)
- **Threshold halting**: Stop analysis when results exceed threshold, without processing full stream
- **Streaming + loading**: Ideal implementation analyzes data while loading at GB/sec

### Implementation Architecture

```
[Streaming Source] → [Stream Processor] → [Cache 1: Real-time]
                          ↓
                    [Threshold Check]
                          ↓
                   [Alert / Action]
```

**Modern Tools**: Apache Kafka, Apache Flink, Apache Storm, AWS Kinesis for streaming data processing.

## Management Best Practices

### Structure Environments Around Analytics

Every step from source to analyst's screen must support complex analytic routines:
- **Loaders**: Accept UDFs for on-the-fly transformations
- **Cleansers**: Embed custom quality rules as UDFs
- **Integrators**: Support complex matching algorithms
- **BI tools**: Execute analytic extensions (R, Python, SAS, MatLab)

### Delay Building Legacy Environments

**Why**: Big data environment changing too rapidly for long-lasting legacy foundation.

**Instead**:
- Plan for disruptive changes: new data types, programming approaches, hardware, cloud services
- Maintain balance among Hadoop, grid computing, RDBMS pushdown, on-premise, cloud, mainframe
- **Assume**: You will reprogram and rehost all big data applications within two years
- **Choose**: Approaches that can be reprogrammed/rehosted easily
- **Consider**: Metadata-driven codeless development environments for insulation from technology changes

**Platform as a Service (PaaS)**: Can help assemble compatible tool sets.

### Build From Sandbox Results

**Pattern**: Allow data scientists to prototype in preferred languages/environments, then IT systematically reprojects for production.

**Workflow**:
1. **Data scientist**: Builds proof of concept in R, Python, Scala, custom code accessing Hadoop
2. **IT turnover team**: Re-implements in production technologies (MatLab in PostgreSQL, SAS in Teradata, ETL tool)
3. **Production deployment**: Scalable, highly available, secure, supportable over long haul

**Key insight**: IT must be tolerant of data scientists' technology choices and prepared to re-implement in standardized tools.

### Try Simple Applications First (Toe in the Water)

**Low-risk starting point**: Use Hadoop for backup and archiving.

**Benefits**:
- Low-cost, flexible backup technology
- Stores data in all formats (unstructured to highly structured)
- Addresses sunsetting challenge (original applications unavailable due to licensing)
- Build big data skills while providing business value

## Architecture Best Practices

### Plan for Data Quality Along the Highway

**Trade-off**: Latency vs quality—very low latency data is unavoidably dirty.

**Quality Improvement by Cache**:

| Cache Level | Latency | Quality Checks Possible |
|------------|---------|------------------------|
| Raw Source | <1 sec | Field-level only |
| Real-time | <10 sec | Structural relationships |
| Business Activity | <1 hour | Complex business rules, threshold checks |
| Top Line | <24 hour | Complete transaction sets, repudiation removal |
| Data Warehouse | Days+ | Comprehensive cleansing, governance, conformance |

**Analyst Expectation**: Immediate data is dirty; wait for later caches if quality is critical.

### Add Value as Early as Possible

Apply filtering, cleansing, pruning, conforming, matching, joining, diagnosing at earliest touch points:

- **Filtering/cleansing/pruning**: Reduce data transferred to next cache
- **Conforming**: Add enterprise attributes (conformed dimensions) for integration
- **Matching/joining**: Enable cross-source analysis with surrogate keys
- **Diagnosing**: Add confidence tags, behavior cluster identifiers, quality scores

**Alternative approach**: Some argue for cleansing only at analysis runtime to preserve "interesting outliers"—balance with governance requirements.

### Avoid Boundary Crashes

**Problem**: Running out of disk space or memory causes application failure.

**Big Data Risk**: Returns with petabyte-scale data.

**Hadoop Advantage**: Indefinitely add commodity hardware for scale-out (but must provision, plug in, connect).

**Best Practice**: Plan far ahead for scaling to huge volumes and throughputs.

### Move Prototypes from Public to Private Cloud

**Public cloud advantages**:
- Instant provisioning and scaling
- Quick in-and-out prototyping (if data sensitivity allows)

**Caution**: Don't leave huge data sets online over weekends (cost risk).

**Limitation**: May not provide data storage control needed for rack-aware MapReduce processes.

**Production pattern**: Prototype on public cloud, move to private cloud for production.

### Strive for 10x to 100x Performance Improvements

**Paradigm shift**: Analysis at high speeds enables new analytics.

**Opportunity**: Hundreds of tightly coded spot solutions for specific analyses (100x faster than standard techniques).

**Examples**:
- Big join problem: Billion-row dimension joined to trillion-row fact table
- Petabyte data visualization: "Flying around" massive data sets
- Semantic zooming without pre-aggregations: Descend from aggregated to atomic detail in unstructured data (analogous to map zooming)

**Lesson**: Be prepared to add revolutionary 10x-100x performance tools to your suite.

### Monitor Compute Resources

Separate big data analytic workloads from conventional DW to preserve service level agreements.

**If big data in Hadoop**: Probably doesn't compete with RDBMS resources.

**If big data on DW machine**: Caution—big data requirements change rapidly in direction of more compute resources.

### Exploit In-Database Analytics

Major RDBMS vendors invest in in-database analytics:
- **PostgreSQL**: Extensible syntax for powerful UDFs in inner loop (open source)
- **Oracle, SQL Server, Teradata**: Analytic extensions combining SQL with advanced analytics
- **Value**: After paying load cost, SQL + analytic extensions are extremely powerful

## Governance Best Practices for Big Data

### There is No Such Thing as "Big Data Governance"

**Point**: Data governance must be comprehensive across entire ecosystem, not isolated to big data.

**Governance scope**: Privacy, security, compliance, data quality, metadata management, master data management, business glossary.

### Dimensionalize Data Before Applying Governance

**Challenge**: Apply governance when you don't know content of incoming data (GB/minute, name-value pairs).

**Best Practice**: Dimensionalize at earliest stage—parse, match, apply identity resolution on-the-fly.

**Why**: Dimensionalizing enables classification for governance (PII detection, sensitivity tagging, compliance rules).

### Privacy is the Most Important Governance Perspective

**Risk**: Compromising individual/organizational privacy damages reputation, diminishes trust, exposes to lawsuits, violates laws.

**Minimum requirements**:
- Mask personal details
- Aggregate data enough to prevent individual identification

**Hadoop-specific concern** (at time of book writing):
- Hadoop doesn't manage updates well after data is written
- Use **persistent data masking** (mask/encrypt on write) OR **dynamic data masking** (mask on read)

**Modern note**: Newer Hadoop components and cloud data lakes have improved update capabilities, but privacy remains critical.

### Don't Choose Big Data Over Governance

Even for exploratory prototypes, maintain governance checklist:
- Privacy requirements
- Security classification
- Compliance regulations
- Data retention policies
- Audit logging

**Goal**: Agile governance, not ineffective bureaucracy, but not skipping governance entirely.

## Integration Pattern: Structured + Unstructured Data

Combine conventional DW data with big data for comprehensive ecosystems.

### Medical Study Example

**Data sources combined** (all joined on patient surrogate key):
- **Conventional DW**: Demographics, health status, claims, medications
- **Image data**: Photographs, X-rays, EKGs, MRIs
- **Text data**: Physician notes, care plans
- **Social media**: Patient opinions of treatment, support group discussions
- **Cohort linkages**: Patients with similar situations, doctors with similar patients

**Enablers**: Conformed dimensions, durable surrogate keys, dimensional thinking across all data types.

### Financial Institution Example

**Data sources**:
- **Structured**: Millions of accounts in RDBMS
- **Documents**: Tens of millions of paper documents scanned
- **Social network**: Secure communication among professionals, partners, customers
- **Storage**: Capture all in Hadoop, dimensionalize, use in business, backup and archive

**Value**: Queryable, integrated ecosystem spanning structured and unstructured data.

## Common Mistakes to Avoid

1. **Ignoring conventional DW best practices**: Dimensional modeling, conformed dimensions, SCDs, surrogate keys remain critical
2. **Building big data in isolation**: Must integrate with enterprise DW via conformed dimensions
3. **Skipping dimensionalization**: Leaving data in raw form without dimensional context limits queryability
4. **Not using surrogate keys**: Relying on source natural keys prevents integration and backflow
5. **Declaring structure too early**: Forcing variable big data into rigid schema before understanding content
6. **Ignoring governance for prototypes**: Privacy, security, compliance violations have real consequences
7. **Choosing single architecture**: Both extended RDBMS and Hadoop have strengths—use both where appropriate
8. **Expecting perfect data quality immediately**: Accept latency-quality trade-off; immediate data is dirty
9. **Over-investing in legacy foundation**: Technology changing too rapidly—plan for re-platforming
10. **Separating big data governance**: Governance must be comprehensive across entire data ecosystem

## Implementation Checklist

### Architecture Design
- [ ] Choose primary architecture (extended RDBMS, Hadoop, or hybrid)
- [ ] Design data highway with appropriate caches for latency requirements
- [ ] Plan backflow mechanisms from DW to earlier caches
- [ ] Define streaming analytics requirements and implement CQL-capable tools
- [ ] Size infrastructure for scale-out (plan for petabytes and boundary crashes)

### Dimensional Modeling
- [ ] Identify dimensions in all data sources (structured and unstructured)
- [ ] Define enterprise attributes for conformed dimensions across big data and conventional DW
- [ ] Assign durable surrogate keys to all major entities (customer, product, time, location)
- [ ] Design fact extraction processes for unstructured sources
- [ ] Choose SCD strategies for time-variant big data dimensions
- [ ] Define schema-on-read vs schema-on-write strategy by data source

### ETL and Data Pipeline
- [ ] Implement Job One: Embed surrogate keys at earliest extraction step
- [ ] Build dimensionalization processes for name-value pair data
- [ ] Design filtering, cleansing, conforming steps at each cache
- [ ] Implement backflow of conformed dimensions to upstream caches
- [ ] Set up streaming data processing for real-time caches
- [ ] Build fact extractors for unstructured sources (text, images, sensors)

### Governance
- [ ] Extend existing data governance framework to cover big data sources
- [ ] Implement privacy controls (masking, encryption, aggregation)
- [ ] Classify data sensitivity and apply compliance rules
- [ ] Define metadata management for name-value pair data
- [ ] Establish data quality expectations by cache level (latency-quality trade-off)
- [ ] Create governance checklist for prototype projects

### Management and Operations
- [ ] Structure environment around analytics (UDFs in loaders, cleansers, BI tools)
- [ ] Design for re-platforming (metadata-driven, avoid legacy lock-in)
- [ ] Establish sandbox-to-production workflow for data scientist prototypes
- [ ] Separate big data workloads from conventional DW for SLA preservation
- [ ] Monitor performance and add 10x-100x improvement tools as available
- [ ] Plan simple first application (e.g., backup/archiving) to build skills

## Summary

Big data expands the DW/BI mission but does not replace fundamental dimensional modeling principles. Conformed dimensions, durable surrogate keys, slowly changing dimensions, and dimensional thinking remain critical for integrating big data with enterprise analytics. The data highway architecture balances latency and quality across multiple caches. Schema-on-read, name-value pairs, and streaming analytics are new techniques complementing conventional DW practices. Governance extends to big data with special emphasis on privacy. Both extended RDBMS and Hadoop architectures have long-term value and will coexist.

## References

- **Master Plan**: Skill #19 in `data_warehouse_kimball/planning/00-SKILLS-MASTER-PLAN.md`
- **Source Chapter**: Chapter 21 (Big Data Analytics)
- **Related Skills**:
  - `conformed-dimension-management`: Techniques for enterprise integration
  - `fact-table-design-patterns`: Fact table types applicable to big data
  - `industry-retail-patterns`: Clickstream integration example
  - `etl-system-architecture-design`: ETL subsystems applicable to big data pipelines
  - `data-profiling-quality-assessment`: Quality assessment techniques for big data sources

## Supporting Reference Files

- `architecture-comparison.md`: Detailed comparison of extended RDBMS vs MapReduce/Hadoop with decision framework and use case mapping
- `data-highway-patterns.md`: Complete data highway implementation guide with cache selection criteria, backflow patterns, ETL processes, and real-world examples
