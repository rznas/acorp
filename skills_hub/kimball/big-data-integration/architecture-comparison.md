# Big Data Architecture Comparison: Extended RDBMS vs MapReduce/Hadoop

## Overview

Two architectural approaches have emerged for big data analytics, each with distinct strengths and long-term advantages. They are likely to coexist far into the future, serving complementary use cases.

## The 19 Capabilities Required for Big Data

A candidate system must support:

1. **Petabyte-scale storage**: Easily support thousands of terabytes
2. **Massive distribution**: Thousands of processors, potentially geographically dispersed and heterogeneous
3. **Store in original formats**: Query and analyze without converting or moving data
4. **Subsecond SQL response**: For highly constrained standard queries
5. **Complex UDFs**: Embed arbitrarily complex user-defined functions within processing requests
6. **Multi-language UDFs**: Implement UDFs in wide variety of industry-standard procedural languages
7. **Extensive UDF libraries**: Reusable functions crossing most/all use cases
8. **Petabyte relation scans**: Execute UDFs as relation scans over petabyte data in minutes
9. **Diverse data types**: Images, waveforms, hierarchical structures, name-value pairs
10. **High-speed loading**: At least gigabytes per second
11. **High-speed integration**: Integrate multiple sources during load at GB/sec
12. **Schema-on-read**: Load data before declaring or discovering structure
13. **Streaming analytics**: Execute certain queries in real time on incoming load data
14. **In-place updates**: Update data at full load speeds
15. **Massive joins**: Billion-row dimension to trillion-row fact without preclustering
16. **Complex workflows**: Schedule and execute multi-hundred node workflows
17. **No single point of failure**: Configure for high availability
18. **Failover and continuation**: Process continuation when nodes fail
19. **Extreme mixed workloads**: Thousands of geographically dispersed users executing variety of requests (ad hoc queries, strategic analysis) while loading batch and streaming data

## Extended RDBMS Architecture

### Core Concept

Existing RDBMS vendors extend classic relational data types to include new types required by big data.

### Architecture Diagram Components

```
[Traditional RDBMS Core]
    ↓
[Data Type Extensions]
    ├─ Complex Structures (vectors, matrices, hyperstructured data)
    ├─ Unstructured/Semistructured (text, images, video)
    └─ Data Bags (name-value pair collections)
    ↓
[UDF Processing in Inner Loop]
    ↓
[Fact Extractor Pattern]
    ├─ Pass 1: Extract facts from original data
    └─ Pass 2: Feed extracted facts back as relational rows/columns
```

### Key Characteristics

**Data Type Support**:
- Traditional scalar types (numbers, text, dates)
- Complex structures: vectors, matrices, custom hyperstructured data
- Unstructured: text, images, video
- Semistructured: name-value pair collections (data bags)

**Processing Model**:
- UDFs written by business analyst users
- UDFs execute within DBMS inner loop (not external applications)
- UDFs process new data types in-place

**Fact Extractor Pattern**:
- **First pass**: RDBMS processes original data (e.g., text, images) with UDFs to extract facts
- **Second pass**: Extracted results automatically fed back to RDBMS input as conventional relational rows/columns
- **Example**: Process tweets → extract sentiment scores → load as fact table rows

**SQL Integration**:
- Standard SQL queries work on traditional columns
- Extended SQL or UDFs access new data types
- Mature query optimization for relational portions

### Strengths

| Capability | Extended RDBMS Advantage |
|-----------|-------------------------|
| SQL query performance | Mature query optimization, indexing, materialized views |
| ACID transactions | Strong consistency, rollback, recovery |
| Existing skills | Leverage SQL expertise and existing applications |
| Data quality | Mature constraint enforcement, referential integrity |
| BI tool integration | Native connectivity to Tableau, Power BI, Cognos, etc. |
| Governance | Built-in security, row-level security, audit logging |
| Structured analytics | Fast aggregations, joins, GROUP BY queries |
| In-database analytics | UDFs in PostgreSQL, R in SQL Server, SAS in Teradata |

### Weaknesses

| Limitation | Impact |
|-----------|--------|
| Schema rigidity | Must declare structure before loading (slower iteration) |
| Scaling cost | Expensive to scale to petabytes (high-end hardware) |
| Unstructured data handling | Better than before, but still not native strength |
| Variable content | Difficult to handle highly variable name-value pair data |
| Geographic distribution | More complex than Hadoop's native distribution model |

### Best Use Cases

1. **Structured data with some unstructured**: Mostly relational with occasional text/images
2. **Strong consistency requirements**: Financial transactions, regulated industries
3. **Complex SQL analytics**: Heavy use of joins, aggregations, window functions
4. **BI tool requirements**: Dashboards and reports from standard tools
5. **Existing RDBMS investment**: Leverage current infrastructure and skills
6. **Data governance critical**: Strong security, compliance, audit requirements

### Platform Examples

- **PostgreSQL**: Open source, extensible UDF syntax, MatLab integration
- **Oracle Exadata**: Extreme scale, in-database analytics, Smart Scan
- **Teradata**: Massively parallel processing, SAS integration
- **SQL Server**: R integration, columnstore indexes, PolyBase for Hadoop
- **Vertica**: Columnar storage, analytics optimized
- **Greenplum**: Open source MPP, MADlib for machine learning

## MapReduce/Hadoop Architecture

### Core Concept

Open-source distributed processing framework for massive-scale data processing with schema-on-read approach.

### Architecture Components

```
[Hadoop Ecosystem]
    ├─ Storage Layer: HDFS (Hadoop Distributed File System), Amazon S3, others
    ├─ Processing Framework: MapReduce (UDF execution framework)
    ├─ Query Interfaces:
    │   ├─ Hive (SQL-like)
    │   ├─ Pig (dataflow scripting)
    │   ├─ HBase (NoSQL database)
    │   └─ Custom MapReduce code (Java, Python, Scala, etc.)
    ├─ Resource Management: YARN (Yet Another Resource Negotiator)
    └─ Data Ingestion: Sqoop, Flume, Kafka
```

### Key Characteristics

**Storage Model**:
- **HDFS**: Hadoop Distributed File System—files split into blocks, replicated across commodity hardware
- **Cloud storage**: Amazon S3, Azure Data Lake, Google Cloud Storage
- **Schema-on-read**: Store data in original format, declare structure when querying

**Processing Model**:
- **MapReduce**: Framework for distributed processing
  - **Map phase**: Process data in parallel across nodes
  - **Shuffle phase**: Redistribute data by key
  - **Reduce phase**: Aggregate results
- **UDF execution framework**: "F" can be extraordinarily complex
- **Multiple languages**: Java (native), Python, Scala, R, others

**Data Formats**:
- Original captured formats preserved
- Text files, JSON, XML, Avro, Parquet, ORC
- Images, video, binary files
- Name-value pairs, nested structures

**Distribution**:
- Thousands of commodity nodes
- Data locality: Process data where it's stored (move compute to data, not data to compute)
- Automatic failover and replication

### Strengths

| Capability | Hadoop Advantage |
|-----------|------------------|
| Petabyte scaling | Linear scale-out on commodity hardware (cost-effective) |
| Schema flexibility | Load before understanding structure, multiple views of same data |
| Unstructured data | Native handling of text, images, logs, sensor data |
| Variable content | Name-value pairs handle unexpected attributes gracefully |
| ETL flexibility | Same data accessed with Hive, Pig, HBase, MapReduce simultaneously |
| Geographic distribution | Native distributed architecture across data centers |
| Open source | Large community, hundreds of components, no vendor lock-in |
| Programming freedom | Implement arbitrarily complex UDFs in many languages |

### Weaknesses

| Limitation | Impact |
|-----------|--------|
| Query latency | Minutes to hours for MapReduce jobs (not subsecond) |
| SQL limitations | Hive SQL not as full-featured as RDBMS SQL |
| Lack of transactions | No ACID guarantees (eventual consistency) |
| Update complexity | Updates and deletes difficult (write-once model historically) |
| Skills gap | Requires new skills (Java, distributed computing concepts) |
| BI tool integration | Less mature than RDBMS connectivity |
| Data quality | No referential integrity constraints, must implement in code |

### Best Use Cases

1. **Massive unstructured data**: Petabytes of logs, clickstreams, sensor data, social media
2. **ETL and staging**: Flexible environment for cleansing, transforming, integrating before loading to RDBMS
3. **Schema unknown**: Data structure evolving or not understood upfront
4. **Exploration and discovery**: Data scientists prototyping with variable approaches
5. **Archival and backup**: Low-cost storage for all data formats
6. **Batch analytics**: Jobs that process entire data sets (not interactive queries)
7. **Complex transformations**: Custom algorithms not expressible in SQL

### Platform Examples

- **Apache Hadoop**: Original open-source distribution
- **Cloudera**: Commercial Hadoop distribution with management tools
- **Hortonworks**: Open-source focused Hadoop distribution (now part of Cloudera)
- **MapR**: High-performance Hadoop with real-time capabilities
- **Amazon EMR**: Elastic MapReduce—managed Hadoop on AWS
- **Azure HDInsight**: Managed Hadoop on Microsoft Azure
- **Google Cloud Dataproc**: Managed Hadoop/Spark on Google Cloud

## Side-by-Side Comparison

| Characteristic | Extended RDBMS | MapReduce/Hadoop |
|---------------|---------------|------------------|
| **Primary Strength** | Fast SQL queries on structured data | Massive scale, unstructured data |
| **Scaling Model** | Scale-up (bigger machines) | Scale-out (more commodity machines) |
| **Data Loading** | Schema-on-write (declare before load) | Schema-on-read (declare at query time) |
| **Query Latency** | Subsecond to seconds | Minutes to hours (MapReduce) |
| **SQL Support** | Full SQL, mature optimization | SQL-like (Hive), less full-featured |
| **Data Types** | Extended relational types | Any format, native unstructured |
| **Consistency** | ACID transactions | Eventual consistency |
| **Updates** | Fast in-place updates | Write-once (or slow updates) |
| **Skills Required** | SQL, database administration | Java, Python, distributed computing |
| **Cost Model** | Expensive high-end hardware | Commodity hardware (cost-effective) |
| **BI Tool Integration** | Mature, native connectivity | Improving, less mature |
| **Data Governance** | Built-in security, audit | Implement in application layer |
| **Use Case Focus** | Reporting, dashboards, analytics | ETL, exploration, archival, batch processing |

## Hybrid Architecture: Best of Both Worlds

### Recommended Pattern

Use **both** architectures in complementary roles:

```
[Raw Big Data Sources]
    ↓
[Hadoop: ETL, Cleansing, Exploration]
    ├─ Load raw data in original formats
    ├─ Explore, profile, dimensionalize
    ├─ Extract structured facts from unstructured sources
    ├─ Cleanse, conform, integrate
    └─ Transform to structured dimensional model
    ↓
[Extended RDBMS: Production DW]
    ├─ Load dimensionally modeled data
    ├─ Serve BI tools with fast SQL queries
    ├─ Support ad hoc analysis and dashboards
    └─ Enforce governance, security, quality
    ↓
[Business Users and BI Applications]
```

### Hybrid Workflow

**Phase 1: Ingestion and Exploration (Hadoop)**
1. Ingest raw data at GB/sec into HDFS (text, JSON, images, logs, clickstreams)
2. Data scientists explore using Hive, Pig, custom MapReduce code
3. Profile data to understand structure, quality, patterns
4. Build prototypes of analytic models

**Phase 2: Transformation and Dimensionalization (Hadoop)**
1. Dimensionalize: Identify and extract dimensions (customer, product, time, location)
2. Extract facts from unstructured sources (sentiment from text, features from images)
3. Assign durable surrogate keys to dimensions
4. Apply conformed dimension attributes for enterprise integration
5. Cleanse, conform, validate using MapReduce jobs or Hive queries

**Phase 3: Load to Data Warehouse (Extended RDBMS)**
1. Export structured dimensional model from Hadoop
2. Load into RDBMS star schema (fact and dimension tables)
3. Apply indexes, partitions, aggregates for query performance
4. Implement security, governance, audit controls

**Phase 4: Production Analytics (Extended RDBMS)**
1. BI tools query dimensional model via SQL
2. Dashboards, reports, ad hoc queries with subsecond response
3. In-database analytics (R, SAS, MatLab) for advanced users

**Phase 5: Backflow (RDBMS → Hadoop)**
1. Push conformed dimensions with surrogate keys back to Hadoop
2. Enable upstream Hadoop analytics to join with DW content
3. Transfer data mining scores, historical summaries to Hadoop for enrichment

### Hybrid Architecture Benefits

- **Hadoop flexibility** for variable, unstructured, exploratory workloads
- **RDBMS performance** for structured, governed, production analytics
- **Conformed dimensions** integrate both environments for drill-across queries
- **Right tool for right job**: Don't force RDBMS to handle petabyte unstructured data; don't force Hadoop to serve interactive dashboards
- **Incremental transition**: Start with Hadoop for ETL, gradually move mature use cases to RDBMS for production

## Decision Framework

### Choose Extended RDBMS When:

- [ ] Data is mostly structured or can be structured
- [ ] Interactive query performance is critical (subsecond to seconds)
- [ ] BI tools need native SQL connectivity
- [ ] Strong ACID transaction requirements
- [ ] Team has deep SQL and RDBMS expertise
- [ ] Data governance and security are paramount
- [ ] Existing RDBMS infrastructure and licenses
- [ ] Data volume is <100 TB (or budget allows high-end hardware for more)

### Choose Hadoop When:

- [ ] Data is unstructured or semistructured (text, images, logs, clickstreams, sensor data)
- [ ] Schema is unknown or rapidly changing
- [ ] Data volume is petabytes or growing toward petabytes
- [ ] Batch processing acceptable (not interactive queries)
- [ ] Cost-effective commodity hardware preferred
- [ ] ETL and data staging are primary use cases
- [ ] Data scientists need flexible exploration environment
- [ ] Archival and backup are requirements

### Choose Hybrid When:

- [x] **Most environments**: Combine Hadoop's flexibility with RDBMS performance
- [x] Data variety spans structured and unstructured
- [x] Need both exploration (Hadoop) and production (RDBMS)
- [x] Budget allows both platforms
- [x] Team can develop skills in both technologies

## Use Case Mapping

### Extended RDBMS Use Cases

| Use Case | Why RDBMS |
|---------|-----------|
| Customer 360 dashboard | Fast SQL joins across dimensions, BI tool connectivity |
| Sales analysis by product/region/time | Star schema optimization, mature aggregation |
| Financial reporting and compliance | ACID transactions, audit logging, governed access |
| Inventory management dashboards | Real-time updates, referential integrity |
| Ad hoc business user queries | SQL familiarity, interactive performance |

### Hadoop Use Cases

| Use Case | Why Hadoop |
|---------|------------|
| Clickstream analysis (petabytes of logs) | Massive scale, schema-on-read for variable log formats |
| Social media sentiment analysis | Unstructured text processing, NLP via UDFs |
| IoT sensor data processing | High-volume ingestion (GB/sec), time-series analytics |
| Image/video analysis | Native handling of binary formats, custom ML algorithms |
| Data lake / archival | Cost-effective storage for all formats |
| ETL staging for DW | Flexible transformations, integration across sources |
| Fraud detection model training | Full data set scans, complex ML algorithms |

### Hybrid Use Cases

| Use Case | Hadoop Role | RDBMS Role |
|---------|-------------|------------|
| Customer churn prediction | Train ML models on full history, score customers | Serve scores to CRM dashboards |
| Product recommendation engine | Build recommendation model on clickstream + purchases | Deliver recommendations via real-time query |
| Supply chain optimization | Integrate sensor data + transactional data | Production reporting and KPIs |
| Healthcare analytics | Process EMR text notes, medical images | Patient dashboards, clinical reporting |

## Modern Evolution (Since Book Publication)

The big data landscape has evolved since the Kimball book publication:

### Apache Spark

- **In-memory processing**: 10-100x faster than MapReduce for iterative algorithms
- **Unified analytics**: Batch, streaming, SQL, ML in single framework
- **SQL support**: Spark SQL with better performance than Hive
- **Replaces MapReduce**: Most new Hadoop deployments use Spark instead
- **Impact**: Blurs line between batch and interactive (Spark SQL can be interactive)

### Cloud Data Warehouses

- **Snowflake, BigQuery, Redshift Spectrum**: Combine RDBMS SQL with big data scale
- **Separation of storage and compute**: Independently scale, pay for what you use
- **Schema-on-read support**: Query data lakes without loading to warehouse
- **Impact**: Extended RDBMS can now scale like Hadoop for many use cases

### Data Lakehouses

- **Delta Lake, Apache Iceberg, Apache Hudi**: ACID transactions on data lakes
- **Schema evolution**: Update schemas in place on HDFS/S3
- **Time travel**: Query historical versions of data
- **Impact**: Hadoop gets transactional capabilities, closing gap with RDBMS

### Recommendation Update

Both architectures continue to converge on capabilities. The hybrid approach remains best practice, but technology choice is less binary than in 2013:

- **Modern data platform**: Data lake (Hadoop/S3) + lakehouse layer (Delta/Iceberg) + cloud DW (Snowflake/BigQuery) + streaming (Kafka/Flink)
- **Dimensional modeling still critical**: Regardless of technology, conformed dimensions and star schemas remain fundamental for integration
- **Surrogate keys still essential**: Durable keys enable backflow and cross-platform integration
- **Governance spans all**: Unified governance across lake, lakehouse, warehouse

## Summary

Extended RDBMS and MapReduce/Hadoop address different aspects of the 19 big data capabilities. Extended RDBMS excels at fast SQL queries on structured data with strong governance. Hadoop excels at massive scale, unstructured data, and flexible ETL. Both architectures have long-term value and will coexist. A hybrid approach using Hadoop for ingestion/exploration/staging and RDBMS for production analytics leverages the strengths of both. Regardless of platform choice, dimensional modeling principles (dimensions, facts, conformed dimensions, surrogate keys, SCDs) remain critical for enterprise integration.
