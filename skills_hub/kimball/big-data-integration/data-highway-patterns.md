# Data Highway Implementation Patterns

## Overview

The data highway is a logical architecture with multiple physical caches of increasing data latency and quality. Each cache serves distinct analytic use cases with different trade-offs between speed and data quality.

## The Latency-Quality Trade-Off

**Fundamental Principle**: Very low latency (immediate) data is unavoidably dirty because there are limits to how much cleansing and diagnosing can be done in very short time intervals.

```
Latency:  <1 sec    <10 sec    <1 hour    <24 hour    Days+
Quality:  Dirtiest  ──────────────────────────────────→ Cleanest
Cache:    Raw       Real-time  Business   Top Line    Data Warehouse
                               Activity
```

## Five Data Caches: Detailed Design

### Cache 1: Raw Source Applications (Immediate, <1 second)

#### Characteristics

- **Latency**: <1 second (often milliseconds)
- **Data Quality**: Dirtiest—only field-level validation possible
- **Processing**: Immediate threshold checks, streaming analytics
- **Storage**: In-memory buffers, streaming platforms, message queues
- **Use Cases**: Mission-critical decisions requiring instant response

#### Quality Checks Possible

| Check Type | Feasible | Example |
|-----------|----------|---------|
| Field-level validation | ✅ Yes | Data type, range, format, null checks |
| Structural relationships | ❌ No | Not enough time to check cross-field consistency |
| Cross-source validation | ❌ No | Other sources not yet integrated |
| Complex business rules | Partial | Only simple threshold checks (e.g., transaction > $10,000) |
| Historical context | ❌ No | No time to look up customer history |

#### Technology Stack

- **Message brokers**: Apache Kafka, AWS Kinesis, Azure Event Hubs, Google Pub/Sub
- **Stream processing**: Apache Flink, Apache Storm, Kafka Streams, Spark Streaming
- **In-memory databases**: Redis, Memcached
- **Complex Event Processing**: Esper, Apache Samza, Drools Fusion

#### Example Use Cases

**Credit Card Fraud Detection**:
```
Transaction Stream → Threshold Check → Alert
                         ↓
                  Amount > $5,000 AND
                  Merchant category = High-Risk AND
                  Time since last transaction < 5 minutes
                         ↓
                  Block transaction, notify customer
```

**Network Security (Cyber Attack Detection)**:
- Monitor packet streams for known attack signatures
- Flag suspicious patterns (port scanning, SQL injection attempts)
- Immediate firewall rule updates

**Complex Event Processing (CEP)**:
- Correlate multiple event streams in real time
- Detect patterns: "3 failed login attempts within 60 seconds"
- Trigger alerts or automated responses

**In-Flight Aircraft Status**:
- Sensor data from aircraft systems
- Immediate anomaly detection (engine temperature, pressure)
- Alert flight crew for corrective action

#### ETL Pattern

```
[Source System]
    ↓ (continuous stream)
[Kafka Topic]
    ↓
[Flink Stream Processor]
    ├─ Deserialize
    ├─ Field validation
    ├─ Threshold checks
    └─ Pattern detection (CEP)
    ↓
[Action]
    ├─ Alert
    ├─ Block transaction
    └─ Store in Cache 2 (Real-time)
```

### Cache 2: Real-Time Applications (<10 seconds)

#### Characteristics

- **Latency**: Seconds to 10 seconds
- **Data Quality**: Basic cleansing, structural validation possible
- **Processing**: Simple business rules, single-source validation
- **Storage**: High-speed cache (Redis, Memcached), fast NoSQL (Cassandra, DynamoDB)
- **Use Cases**: Personalization, immediate response required

#### Quality Checks Possible

| Check Type | Feasible | Example |
|-----------|----------|---------|
| Field-level validation | ✅ Yes | Full field-level cleansing |
| Structural relationships | ✅ Yes | Dates in order, FK exists in lookup table |
| Cross-source validation | Partial | Limited to pre-loaded reference data |
| Complex business rules | ✅ Yes | Rules evaluable on single record |
| Historical context | ✅ Yes | Lookup recent customer history from cache |

#### Technology Stack

- **Key-value stores**: Redis, Memcached
- **NoSQL databases**: Cassandra, DynamoDB, Couchbase, MongoDB
- **In-memory data grids**: Hazelcast, Apache Ignite
- **Real-time OLAP**: Apache Druid, ClickHouse

#### Example Use Cases

**Web Page Ad Selection**:
```
Page View Event
    ↓
Lookup: Customer profile (demographics, interests)
    ↓
Lookup: Recent browsing history (last 24 hours)
    ↓
Apply: Targeting rules (product affinity, price sensitivity)
    ↓
Select: Top 3 ads by relevance score
    ↓
Render: Display ads (<10 sec total latency)
```

**Personalized Price Promotions**:
- Customer views product page
- Lookup customer segment, purchase history, cart value
- Calculate personalized price or promotion
- Display within seconds

**Online Game Gesture Tracking**:
- Player actions streaming from game client
- Validate moves, update game state
- Broadcast to other players in real time

**Customer Service Portal**:
- Agent opens customer record
- Real-time assembly of customer 360 view
- Recent interactions, open cases, account status
- Displayed within seconds

#### ETL Pattern

```
[Cache 1 or Source]
    ↓
[Kafka → Redis/Cassandra]
    ├─ Enrich: Lookup customer dimension (by surrogate key)
    ├─ Enrich: Lookup product dimension (by surrogate key)
    ├─ Validate: Structural checks (dates, relationships)
    ├─ Apply: Simple business rules
    └─ Cache: Write to Redis with TTL (time-to-live)
    ↓
[Application Layer]
    ├─ Read: Sub-second lookup
    └─ Act: Personalize, recommend, alert
    ↓
[Async Write to Cache 3]
```

**Key Pattern**: Surrogate keys enable fast lookups of dimension data cached in Redis.

### Cache 3: Business Activity Applications (<1 hour)

#### Characteristics

- **Latency**: Minutes to 1 hour
- **Data Quality**: Cross-source validation, relationship checks
- **Processing**: Multi-source integration, complex business rules
- **Storage**: Operational Data Store (ODS), fast RDBMS, NoSQL
- **Use Cases**: Operational dashboards, process tracking, customer service

#### Quality Checks Possible

| Check Type | Feasible | Example |
|-----------|----------|---------|
| Field-level validation | ✅ Yes | Complete field cleansing |
| Structural relationships | ✅ Yes | Full referential integrity checks |
| Cross-source validation | ✅ Yes | Integrate multiple sources, detect conflicts |
| Complex business rules | ✅ Yes | Multi-step validation (waiting for threshold, complete transaction sets) |
| Historical context | ✅ Yes | Join with recent history (last 90 days) |

#### Technology Stack

- **ODS databases**: PostgreSQL, SQL Server, MySQL, Oracle
- **Columnar databases**: Vertica, Greenplum (for faster aggregations)
- **Hybrid OLTP/OLAP**: SAP HANA, MemSQL, VoltDB

#### Example Use Cases

**Low-Latency KPI Dashboards**:
```
Hourly ETL
    ↓
Aggregate: Sales by region, product, hour
    ↓
Calculate: KPIs (revenue, margin, units, conversion rate)
    ↓
Load: To ODS fact tables
    ↓
Push: Dashboard updates to users (web, mobile)
```

**Trouble Ticket Tracking**:
- Integrate: Ticket creation, assignments, status changes
- Calculate: SLA compliance, time to resolution, escalation flags
- Dashboard: Real-time ticket queue for support managers

**Process Completion Tracking**:
- Monitor: Order fulfillment pipeline (order → pick → pack → ship)
- Alert: Orders at risk of missing SLA
- Dashboard: Current status of all in-flight orders

**Customer Service Portal**:
- Assemble: Complete customer view (transactions, support cases, shipments, payments)
- Integrate: Data from CRM, ERP, billing, logistics systems
- Display: In portal for agents within seconds

**Fused CEP Reporting**:
- Combine: Multiple complex event streams
- Correlate: Events across systems (web, mobile, call center, store)
- Report: Integrated view of customer interactions

#### ETL Pattern

```
[Multiple Source Systems]
    ↓
[Incremental ETL (every 15-60 minutes)]
    ├─ Extract: Changes since last run (CDC)
    ├─ Cleanse: Full field-level cleansing
    ├─ Conform: Apply enterprise attributes (conformed dimensions)
    ├─ Match: Link customer records across sources
    ├─ Integrate: Join transactions with dimensions
    ├─ Validate: Referential integrity, business rules
    └─ Diagnose: Add quality scores, confidence tags
    ↓
[ODS Fact and Dimension Tables]
    ├─ Fact: Transaction details with surrogate keys
    └─ Dimension: Current state (SCD Type 1 overwrite)
    ↓
[BI Layer]
    ├─ Dashboards: Pre-built KPI visualizations
    └─ Ad hoc: User queries for operational analysis
```

**Key Pattern**: ODS uses Type 1 SCD (overwrite) for most dimensions—current state matters more than history for operational use cases.

### Cache 4: Top Line Applications (<24 hours)

#### Characteristics

- **Latency**: Daily (often overnight batch)
- **Data Quality**: Complete transaction sets, repudiated transactions removed
- **Processing**: Full ETL cleansing, comprehensive business rules
- **Storage**: Daily snapshot tables, summary fact tables
- **Use Cases**: Daily reporting, tactical decisions, midcourse corrections

#### Quality Checks Possible

| Check Type | Feasible | Example |
|-----------|----------|---------|
| Field-level validation | ✅ Yes | Complete cleansing |
| Structural relationships | ✅ Yes | Full referential integrity |
| Cross-source validation | ✅ Yes | All sources integrated |
| Complex business rules | ✅ Yes | All rules including time-delayed checks |
| Historical context | ✅ Yes | Join with full history |
| Transaction completeness | ✅ Yes | Wait for batch completion, remove incomplete sets |

#### Technology Stack

- **RDBMS**: SQL Server, Oracle, PostgreSQL, MySQL
- **Columnar DW**: Vertica, Greenplum, Snowflake, Redshift, BigQuery
- **MPP databases**: Teradata, Netezza

#### Example Use Cases

**Tactical Reporting**:
- Yesterday's sales by store, region, product category
- Top-performing SKUs, underperforming regions
- Daily P&L flash report

**Promotion Tracking**:
- Promotion effectiveness: lift, cannibalization, profitability
- Daily rollup of promotional sales vs baseline

**Midcourse Corrections Based on Social Media Buzz**:
- Daily sentiment analysis aggregated by product, brand
- Trend detection: sudden sentiment shifts
- Alert marketing team for response

**Senior Management "Top Line" Review**:
- What happened in enterprise over past 24 hours
- Key metrics: revenue, orders, customer acquisition, inventory levels
- Exception reporting: significant variances from plan

#### ETL Pattern

```
[Source Systems End-of-Day Extracts]
    ↓
[Nightly ETL (runs after source day closes)]
    ├─ Extract: Full day's transactions
    ├─ Wait: For batch completion (all sources finished)
    ├─ Cleanse: Full cleansing pipeline
    ├─ Validate: Complete transaction sets (remove incomplete)
    ├─ Remove: Repudiated transactions, cancellations
    ├─ Conform: Enterprise-wide conformance
    ├─ Integrate: Join all sources on conformed dimensions
    └─ Aggregate: Daily summaries by key dimensions
    ↓
[Daily Snapshot Fact Tables]
    ├─ Grain: One row per day per product per store (or similar)
    ├─ Dimensions: Product, store, date (calendar date, not timestamp)
    └─ Facts: Daily sales, units, transactions, margin
    ↓
[Morning Reports]
    ├─ Email: Top line summary to executives
    └─ Dashboard: Detailed daily performance
```

**Advantage over Cache 3**: Fundamentally more complete data—sources have finished daily processing, incomplete transactions removed, corrections applied.

### Cache 5: Data Warehouse and Long Time Series (Days to years)

#### Characteristics

- **Latency**: Days or longer (historical data)
- **Data Quality**: Highest—comprehensive cleansing, governance, conformance
- **Processing**: Full dimensional modeling, Type 2 SCD history tracking, master data management
- **Storage**: Enterprise data warehouse (star schemas), OLAP cubes
- **Use Cases**: Strategic analysis, trend analysis, forecasting, regulatory reporting

#### Quality Checks Possible

All checks from Cache 4, plus:

| Check Type | Example |
|-----------|---------|
| Comprehensive governance | Data stewardship review, approval workflows |
| Master data management | Golden customer, product, location records |
| Historical integrity | Type 2 SCD change tracking, audit trails |
| Regulatory compliance | SOX, GDPR, HIPAA compliance checks |
| Data lineage | Full source-to-target tracking |

#### Technology Stack

- **Traditional DW**: Oracle Exadata, Teradata, SQL Server Enterprise, IBM Db2
- **Cloud DW**: Snowflake, Amazon Redshift, Google BigQuery, Azure Synapse
- **Columnar MPP**: Vertica, Greenplum
- **OLAP**: Microsoft Analysis Services, Oracle OLAP, IBM Cognos TM1

#### Example Use Cases

**Historical Analysis**:
- Multi-year trend analysis: sales growth, customer retention, market share
- Seasonality analysis: compare this year vs last 3 years
- Product lifecycle analysis: introduction → growth → maturity → decline

**Master Data Management**:
- Golden customer records: deduplicated, conformed, trusted
- Golden product records: complete hierarchies, all attributes
- Dimensional conformance: enterprise-wide integration

**Large-Scale Temporal Dynamics**:
- Customer lifetime value analysis over years
- Market basket analysis across full transaction history
- Supply chain optimization using multi-year patterns

**Markov Chain Analysis**:
- Customer state transitions: prospect → customer → repeat buyer → churn
- Product affinity sequences: purchase path analysis
- Channel migration patterns: store → web → mobile app

**Regulatory Reporting**:
- Financial reports with full audit trails
- GDPR data lineage and retention enforcement
- Healthcare outcomes reporting with patient history

#### ETL Pattern

```
[Source Systems + Cache 4]
    ↓
[Full ETL with History Tracking]
    ├─ Extract: Incremental changes + late-arriving data
    ├─ Dimension Processing:
    │   ├─ SCD Type 2: Track attribute changes with effective dates
    │   ├─ SCD Type 1: Overwrite non-historical attributes
    │   ├─ Surrogate keys: Assign durable keys
    │   └─ Conformance: Apply enterprise-wide conformed attributes
    ├─ Fact Processing:
    │   ├─ Lookup: Surrogate keys from dimensions (Type 2 aware)
    │   ├─ Validate: Referential integrity against dimension keys
    │   ├─ Late-arriving facts: Handle with historical key lookup
    │   └─ Audit: Attach audit dimension for lineage
    ├─ Quality:
    │   ├─ Profile: Comprehensive data profiling
    │   ├─ Screen: All quality screens (column, structure, business rule)
    │   └─ Tag: Error events captured, tagged, continued
    └─ Governance:
        ├─ Stewardship: Data quality review by business stewards
        ├─ Approval: Conformed dimension changes approved
        └─ Lineage: Full source-to-target metadata
    ↓
[Star Schema (Dimensional Model)]
    ├─ Dimension Tables: SCD Type 2 with history
    └─ Fact Tables: Atomic grain with surrogate keys
    ↓
[Aggregates and OLAP Cubes]
    ├─ Pre-built aggregates for performance
    └─ OLAP cubes for multidimensional analysis
    ↓
[Enterprise BI]
    ├─ Standard reports and dashboards
    ├─ Ad hoc analysis by business users
    ├─ Advanced analytics (R, Python, SAS in-database)
    └─ Data mining and predictive modeling
```

**Distinguishing Characteristic**: Type 2 SCD history tracking provides time travel capability—query data warehouse as it existed at any point in time.

## Data Highway Flow Patterns

### Forward Flow: Source → Warehouse

```
[Raw Source]
    ↓ (Extract, assign surrogate keys)
[Real-time Cache]
    ↓ (Cleanse, validate structure)
[Business Activity Cache]
    ↓ (Integrate sources, apply business rules)
[Top Line Cache]
    ↓ (Complete transactions, comprehensive cleansing)
[Data Warehouse]
    ↓ (SCD history, governance, conformance)
```

**Value Added at Each Step**:

| Step | Added Value |
|------|-------------|
| Raw → Real-time | Field validation, threshold checks, surrogate key assignment |
| Real-time → Business Activity | Structural validation, single-source integration, simple rules |
| Business Activity → Top Line | Multi-source integration, complex rules, transaction completeness |
| Top Line → DW | Historical tracking, conformance, governance, MDM |

**Job One at Every Step**: Replace source proprietary keys with durable surrogate keys.

### Backflow: Warehouse → Earlier Caches

**Why Backflow**:
- Highly administered dimensions from DW (customer, product, date) should be available in earlier caches
- Derived data from DW (historical summaries, data mining scores) enriches upstream analytics
- Real-time applications can join with rich DW content via surrogate keys

**Backflow Example 1: Conformed Customer Dimension**

```
[Data Warehouse]
    ├─ Customer Dimension: Complete, conformed, trusted
    │   ├─ Customer Key (surrogate): 123456
    │   ├─ Customer demographic cluster: "High-Value Urban Professional"
    │   ├─ Customer lifetime value segment: "Platinum"
    │   └─ Customer credit score band: "Excellent"
    ↓
[Push to Business Activity Cache]
    ├─ Store: Customer dimension with surrogate keys
    └─ Enable: ODS applications to join on customer key
    ↓
[Push to Real-time Cache]
    ├─ Store: Customer key, segment, cluster in Redis
    └─ Enable: Web application to personalize using customer key
```

**Result**: Real-time web application can join clickstream event (with customer surrogate key) to customer dimension for personalization—using same key as in DW.

**Backflow Example 2: Data Mining Scores**

```
[Data Warehouse]
    ├─ Advanced Analytics: Customer churn prediction model
    │   ├─ Input: 3 years of transaction history
    │   ├─ Model: Gradient boosting classifier
    │   └─ Output: Churn probability score (0.0 to 1.0)
    ↓
[Add Score to Customer Dimension]
    ├─ Customer Key: 123456
    └─ Churn probability: 0.78 (high risk)
    ↓
[Push to Business Activity Cache]
    └─ Enable: Call center application to see churn risk
    ↓
[Push to Real-time Cache]
    └─ Enable: Web app to show retention offer to high-risk customers
```

**Result**: Upstream applications leverage sophisticated DW analytics without re-running models.

**Backflow Example 3: Historical Summaries**

```
[Data Warehouse]
    ├─ Customer 12-month purchase summary:
    │   ├─ Total purchases: $5,432
    │   ├─ Average order value: $127
    │   ├─ Product categories purchased: Electronics, Books, Toys
    │   └─ Recency: 12 days since last purchase
    ↓
[Push to Real-time Cache]
    └─ Enable: Product recommendation engine to use purchase history
```

**Implementation Requirement**: All caches must use same durable surrogate keys for joins.

### Multi-Path Flow

Some data takes multiple paths simultaneously:

```
[Raw Source: Transaction]
    ├─ Path 1: → Real-time Cache (for fraud detection)
    ├─ Path 2: → Business Activity Cache (for ODS reporting)
    └─ Path 3: → Direct to Top Line Cache (daily batch doesn't need real-time path)
```

**Use Case Example**: Credit card transaction
- **Path 1**: Real-time fraud scoring (<1 sec) to approve or decline
- **Path 2**: Business activity dashboard (within 1 hour) for call center agents
- **Path 3**: Daily batch (24 hours) for settlement and accounting

## Cache Selection Criteria

### Decision Matrix

| Requirement | Recommended Cache(s) |
|-------------|---------------------|
| **Latency Requirements** |
| Must respond in <1 second | Cache 1: Raw Source |
| Must respond in <10 seconds | Cache 2: Real-time |
| Can wait up to 1 hour | Cache 3: Business Activity |
| Daily updates acceptable | Cache 4: Top Line |
| Historical analysis (days/years) | Cache 5: Data Warehouse |
| **Data Quality Requirements** |
| Dirty data acceptable (speed critical) | Cache 1-2 |
| Basic cleansing required | Cache 2-3 |
| Comprehensive cleansing required | Cache 4-5 |
| Governance and compliance required | Cache 5: Data Warehouse |
| **Analysis Type** |
| Alerting and threshold detection | Cache 1 |
| Personalization and recommendations | Cache 2 |
| Operational dashboards and tracking | Cache 3 |
| Daily reporting and tactical decisions | Cache 4 |
| Strategic analysis and forecasting | Cache 5 |
| **Data Integration** |
| Single-source analysis | Cache 1-2 |
| Multi-source operational integration | Cache 3 |
| Enterprise-wide integration | Cache 4-5 |
| **Historical Context** |
| No history needed (current state only) | Cache 1-3 |
| Recent history (last 90 days) | Cache 3-4 |
| Full history with time travel | Cache 5: Data Warehouse (Type 2 SCD) |

### Minimum Viable Highway

Not every implementation needs all five caches. Start with minimum viable architecture:

**Minimal (2 caches)**:
- **Cache 5**: Data Warehouse (daily batch)
- **Cache 3 or 4**: Business Activity or Top Line (operational)

**Common (3 caches)**:
- **Cache 5**: Data Warehouse (strategic)
- **Cache 4**: Top Line (daily reporting)
- **Cache 3**: Business Activity (operational dashboards)

**Advanced (4-5 caches)**:
- Add **Cache 2**: Real-time for personalization, recommendations
- Add **Cache 1**: Raw source for fraud detection, security monitoring

**Evaluate Need for Each Cache**:
1. Do users require this latency level? (If not, skip cache)
2. Is there a distinct use case not served by adjacent caches? (If not, consolidate)
3. Can technology support required latency? (If not, defer until capability available)

## ETL Processes Between Caches

### General Pattern

```
[Cache N]
    ↓
[ETL Process N+1]
    ├─ Extract: Pull changes from Cache N
    ├─ Add Value:
    │   ├─ Cleansing (more time available)
    │   ├─ Validation (more comprehensive checks)
    │   ├─ Integration (additional sources)
    │   ├─ Conforming (enterprise attributes)
    │   └─ Diagnosing (quality scores, confidence tags)
    ├─ Transform:
    │   ├─ Dimensionalize (identify/extract dimensions)
    │   ├─ Surrogate keys (assign or lookup)
    │   └─ Aggregate (if moving to summary grain)
    └─ Load: To Cache N+1
```

### Cache 1 → Cache 2 ETL

**Frequency**: Continuous (streaming) or micro-batch (every few seconds)

**Process**:
1. **Extract**: Consume from message queue (Kafka)
2. **Validate**: Field-level checks (data type, range, format)
3. **Enrich**: Lookup reference data (cached in Redis)
4. **Assign**: Surrogate keys for customer, product (if not already assigned)
5. **Apply**: Simple business rules (single-record checks)
6. **Load**: Write to Cache 2 storage (Redis, Cassandra, DynamoDB)

**Technology**: Apache Flink, Kafka Streams, Spark Streaming

### Cache 2 → Cache 3 ETL

**Frequency**: Every 5-30 minutes (mini-batch)

**Process**:
1. **Extract**: Read from Cache 2 (last 30 minutes of data)
2. **Integrate**: Join with other sources
3. **Validate**: Structural checks (date order, FK existence)
4. **Conform**: Add enterprise attributes to dimensions
5. **Diagnose**: Add quality scores
6. **Aggregate**: Optionally roll up to hourly grain
7. **Load**: Upsert to ODS tables (merge)

**Technology**: ETL tool (Informatica, Talend), custom scripts (Python, Spark)

### Cache 3 → Cache 4 ETL

**Frequency**: Daily (overnight batch)

**Process**:
1. **Extract**: Read from Cache 3 (last 24 hours)
2. **Wait**: For source completeness (all systems finished daily close)
3. **Validate**: Complete transaction sets, remove incomplete
4. **Remove**: Repudiated transactions, cancellations
5. **Integrate**: Join all sources on conformed dimensions
6. **Aggregate**: Daily summaries by key dimensions
7. **Load**: Insert into daily snapshot tables

**Technology**: ETL tool, database stored procedures, Spark batch jobs

### Cache 4 → Cache 5 ETL

**Frequency**: Daily or hourly (depending on DW latency requirement)

**Process**:
1. **Extract**: Incremental changes from Cache 4 or sources
2. **Dimension Processing**:
   - **Change detection**: Compare to existing dimension rows
   - **SCD Type 2**: Insert new rows for changes with effective dates
   - **SCD Type 1**: Update existing rows for non-historical attributes
   - **Surrogate keys**: Assign to new dimension members
3. **Fact Processing**:
   - **Lookup**: Surrogate keys from dimensions (historical lookup for Type 2)
   - **Validate**: Referential integrity
   - **Late arrivals**: Handle with historical key lookup
   - **Audit**: Attach audit dimension
4. **Quality**:
   - **Profile**: Comprehensive data profiling
   - **Screen**: Column, structure, business rule screens
   - **Tag**: Error events (tag and continue, don't halt)
5. **Governance**:
   - **Stewardship**: Data quality review
   - **Lineage**: Record source-to-target metadata
6. **Load**: Insert into fact tables, dimension tables

**Technology**: Enterprise ETL tool (Informatica PowerCenter, Talend, SSIS, DataStage), custom frameworks

## Real-World Examples

### Example 1: Retail E-Commerce

**Cache 1: Raw Source (Message Streaming)**
- **Data**: Clickstream events (page views, clicks, add-to-cart, purchases)
- **Latency**: <100 milliseconds
- **Storage**: Kafka topics
- **Use Case**: Fraud detection (block suspicious orders immediately)

**Cache 2: Real-time (Operational)**
- **Data**: Session-level clickstream, customer profile, real-time inventory
- **Latency**: <5 seconds
- **Storage**: Redis cache
- **Use Case**: Product recommendations, personalized pricing, inventory availability

**Cache 3: Business Activity (ODS)**
- **Data**: Orders, shipments, customer service interactions
- **Latency**: 15-minute refresh
- **Storage**: PostgreSQL ODS
- **Use Case**: Call center agent dashboard (customer 360 view), order tracking

**Cache 4: Top Line (Daily Snapshot)**
- **Data**: Daily sales by product, store, promotion
- **Latency**: Updated 6 AM daily
- **Storage**: SQL Server summary tables
- **Use Case**: Daily sales flash report for executives, promotion performance tracking

**Cache 5: Data Warehouse (Enterprise DW)**
- **Data**: 5 years of transaction history, Type 2 SCD dimensions
- **Latency**: Daily load (previous day available by 8 AM)
- **Storage**: Snowflake cloud DW
- **Use Case**: Customer lifetime value analysis, market basket analysis, seasonal trend analysis

**Backflow**: Customer segments from DW → pushed to Redis → enables real-time personalization

### Example 2: Financial Services (Banking)

**Cache 1: Raw Source (Transaction Stream)**
- **Data**: ATM transactions, card swipes, wire transfers
- **Latency**: <500 milliseconds
- **Storage**: Kafka + Flink
- **Use Case**: Fraud detection (decline suspicious transactions)

**Cache 2: Real-time (Account Balances)**
- **Data**: Current account balances after each transaction
- **Latency**: <3 seconds
- **Storage**: Redis + DynamoDB
- **Use Case**: ATM balance inquiry, mobile app balance display

**Cache 3: Business Activity (ODS)**
- **Data**: Account transactions, customer interactions, loan applications
- **Latency**: 30-minute refresh
- **Storage**: Oracle ODS
- **Use Case**: Branch teller customer 360 view, loan officer dashboard

**Cache 4: Top Line (Daily Balances)**
- **Data**: End-of-day account balances, daily transaction summaries
- **Latency**: Daily (after end-of-day processing)
- **Storage**: Oracle summary tables
- **Use Case**: Daily regulatory reporting, management dashboards

**Cache 5: Data Warehouse (Enterprise DW)**
- **Data**: Multi-year account history, Type 2 SCD for customer demographics
- **Latency**: Daily (overnight batch)
- **Storage**: Teradata DW
- **Use Case**: Customer profitability analysis, churn prediction, regulatory compliance (CCAR, stress testing)

**Backflow**: Customer credit score bands from DW → pushed to ODS → enables real-time credit decisioning

### Example 3: Healthcare (Hospital System)

**Cache 1: Raw Source (IoT Sensors)**
- **Data**: Patient vital signs (heart rate, blood pressure, oxygen saturation)
- **Latency**: <1 second
- **Storage**: Apache Kafka + InfluxDB (time-series)
- **Use Case**: ICU monitoring, automated alerts for critical vital signs

**Cache 2: Real-time (Patient Status)**
- **Data**: Current patient location, status, assigned staff
- **Latency**: <10 seconds
- **Storage**: MongoDB
- **Use Case**: Nurse station dashboard, bed management

**Cache 3: Business Activity (Clinical ODS)**
- **Data**: Lab results, medication orders, care plans, nurse notes
- **Latency**: 15-minute refresh
- **Storage**: SQL Server ODS
- **Use Case**: Electronic Health Record (EHR) system, clinical decision support

**Cache 4: Top Line (Daily Census)**
- **Data**: Daily patient census, admissions, discharges, bed utilization
- **Latency**: Daily (updated 5 AM)
- **Storage**: SQL Server summary tables
- **Use Case**: Hospital management dashboard, capacity planning

**Cache 5: Data Warehouse (Enterprise DW)**
- **Data**: Multi-year patient outcomes, Type 2 SCD for patient demographics and diagnoses
- **Latency**: Daily (overnight batch)
- **Storage**: Snowflake cloud DW
- **Use Case**: Outcomes research, population health management, quality reporting (HEDIS, CMS measures)

**Backflow**: Patient risk scores from DW → pushed to clinical ODS → enables risk-stratified care management

## Common Mistakes to Avoid

1. **Building all five caches when not needed**: Start with minimum viable highway, add caches only when clear use case exists
2. **Not assigning surrogate keys at extraction**: Job One must be surrogate key assignment, or backflow won't work
3. **Expecting high quality from low-latency caches**: Accept latency-quality trade-off; don't over-invest in cleansing at Cache 1-2
4. **Skipping backflow**: Upstream caches miss opportunity to leverage DW dimensions and analytics
5. **Using different keys across caches**: All caches must use same durable surrogate keys for customer, product, etc.
6. **Not planning for scale**: Cache 1-2 may have 10-100x volume of Cache 5; size infrastructure accordingly
7. **Ignoring governance in early caches**: Privacy, security apply to all caches, not just DW
8. **Overloading DW with operational queries**: Separate operational workloads (Cache 3-4) from analytical DW (Cache 5)
9. **Building caches in isolation**: Each cache must integrate with adjacent caches via ETL processes
10. **Not documenting the highway**: Users need to understand which cache to use for which purpose

## Summary

The data highway architecture balances latency and quality across five potential caches. Each cache serves distinct use cases with different trade-offs. Forward flow adds value at each step (cleansing, integration, conformance). Backflow pushes DW dimensions and analytics to upstream caches. Durable surrogate keys enable integration across all caches. Not every implementation needs all five caches—start with minimum viable highway and expand based on business requirements. The latency-quality trade-off is fundamental: immediate data is dirty, comprehensive quality takes time.
