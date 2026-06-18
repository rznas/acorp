---
name: mdm-integration
description: Integrate Master Data Management (MDM) systems with dimensional data warehouses to create conformed customer, product, and other entity dimensions through upstream MDM or downstream ETL deduplication, matching, and survivorship techniques.
---

# Master Data Management Integration

## When to Use This Skill

Use this skill when you need to:
- Integrate customer, product, supplier, or other master data from multiple source systems into conformed dimensions
- Decide between upstream (operational) MDM vs downstream (DW/BI) MDM architectures
- Implement deduplication, matching, and survivorship logic for creating golden records
- Establish data governance for master reference data across the enterprise
- Build conformed dimensions from disparate, inconsistent source systems
- Create a 360-degree view of customers, products, or other critical business entities

This skill applies during the **Architecture & ETL** phases of the Kimball DW/BI Lifecycle, particularly when designing conformed dimension strategies (bus matrix architecture) and ETL conforming processes.

---

## The Master Data Management Challenge

Master data represents the core business entities that multiple operational systems reference: customers, products, suppliers, employees, locations, and accounts. In most organizations, master data faces three critical problems:

**1. Multiple Versions of Truth**
- Different systems maintain separate customer masters with different IDs
- Same customer appears multiple times with variations (J. Smith, John Smith, Jon Smyth)
- Product hierarchies differ across divisions or geographic regions
- No single authoritative source for entity definitions

**2. Data Quality Issues**
- Inconsistent formats, spellings, and abbreviations
- Missing or incomplete information
- Conflicting attribute values across systems
- Lack of validation at data capture

**3. Integration Barriers**
- No universal key across systems
- Differing business rules and definitions by department
- Political resistance to standardization
- Technical complexity of merging disparate sources

**The DW/BI Impact**: Without consistent master data, you cannot build conformed dimensions. Without conformed dimensions, you cannot integrate across business processes or enable drill-across reporting. The entire enterprise data warehouse bus architecture depends on solving the master data problem.

---

## Two MDM Architecture Patterns

There are two fundamental approaches to master data management in relation to the data warehouse:

### Pattern 1: Upstream (Operational) MDM

In this pattern, an operational MDM system sits between source systems and the data warehouse.

<mdm_upstream_architecture>
**Architecture**:
Source Systems → Operational MDM Hub → Data Warehouse → BI Applications

**Characteristics**:
- MDM system is authoritative source for master data
- Deduplication and survivorship happen at operational level
- Source systems may read from MDM hub (optional)
- Data warehouse extracts clean, conformed master data from MDM
- Single version of truth established before DW extraction

**Advantages**:
- Master data quality improves at the source
- Operational systems benefit from clean data
- Data warehouse inherits pre-conformed dimensions
- Reduces ETL complexity in data warehouse
- Enables real-time operational use of golden records

**Challenges**:
- Requires major operational system changes
- High organizational and political barriers
- Expensive MDM platform investment
- Long implementation timeframes
- Requires strong executive sponsorship
- Source systems may resist using MDM data
</mdm_upstream_architecture>

### Pattern 2: Downstream (DW/BI) MDM

In this pattern, the data warehouse ETL system performs master data management.

<mdm_downstream_architecture>
**Architecture**:
Source Systems → Data Warehouse ETL (MDM Logic) → Conformed Dimensions → BI Applications

**Characteristics**:
- Multiple source system masters extracted to staging area
- ETL system performs deduplication, matching, survivorship
- Conformed dimensions created in DW presentation layer
- Operational systems unchanged
- DW team owns master data integration

**Advantages**:
- No changes required to operational systems
- Faster to implement (no operational dependencies)
- DW team has full control
- Lower political barriers
- Can integrate external data sources easily
- Iterative, agile implementation possible

**Challenges**:
- Operational systems don't benefit from clean master data
- ETL processing complexity increases
- Data quality problems persist at source
- Requires sophisticated ETL tools and skills
- Master data only clean in DW, not operational systems
</mdm_downstream_architecture>

### Which Pattern to Choose?

<decision_framework>
**Choose Upstream MDM if**:
- Executive sponsorship and funding secured
- Organization committed to enterprise data governance
- Operational systems need clean master data (e.g., customer service, sales)
- Multiple operational systems need to share master data
- Long-term strategic investment approved
- Willing to accept 12-24 month implementation
- Real-time operational access to golden records required

**Choose Downstream MDM if**:
- Need analytic integration quickly without operational changes
- Limited budget or executive sponsorship
- Operational system changes blocked politically or technically
- DW/BI is primary consumer of integrated master data
- Agile, iterative delivery required
- Organization not ready for operational MDM governance
- Want to prove value before broader MDM investment

**Hybrid Approach**:
Many organizations start with downstream MDM to prove value, then migrate to upstream MDM as the business case strengthens and organizational readiness improves.
</decision_framework>

---

## Core MDM Processes

Whether implementing upstream or downstream MDM, four core processes are required:

### 1. Deduplication (ETL Subsystem 7)

Deduplication identifies and removes duplicate records representing the same real-world entity.

<deduplication_process>
**Exact Match Deduplication**:
Use when records share a reliable common key across systems.

**Steps**:
1. Identify matching key column(s): customer number, product SKU, email address
2. Sort incoming records by matching key
3. Group records with identical key values
4. Select one record as master (survivorship rules)
5. Flag or remove duplicates

**Example - Customer Deduplication**:
```
Source 1: CustomerID=12345, Name="John Smith", Email="jsmith@email.com"
Source 2: CustomerID=98765, Name="John Smith", Email="jsmith@email.com"
Match on: Email address (exact match)
Result: Single customer record created
```

**Fuzzy Match Deduplication**:
Use when no reliable key exists; match based on similarity of multiple attributes.

**Matching Techniques**:
- **Name matching**: Soundex, metaphone, edit distance algorithms
- **Address matching**: Parse and standardize, compare components
- **Composite scoring**: Weight multiple attributes (name 40%, address 30%, phone 20%, DOB 10%)
- **Threshold-based**: Match score > 85% = duplicate, 70-85% = manual review, < 70% = unique

**Example - Fuzzy Customer Match**:
```
Record 1: J. Smith, 123 Main St, Boston MA
Record 2: Jon Smith, 123 Main Street, Boston MA
Record 3: John Smyth, 123 Main St, Boston MA

Match Score (1 vs 2): 92% → Duplicate
Match Score (1 vs 3): 88% → Duplicate
Match Score (2 vs 3): 85% → Duplicate
Result: All three = same customer
```

**Commercial Tools**:
Specialized data quality and MDM platforms provide sophisticated matching:
- Pattern recognition and machine learning algorithms
- Pre-built matching rules for names, addresses, companies
- Reference data libraries (postal codes, company registries)
- Confidence scoring and manual review workflows
- Examples: Informatica MDM, IBM InfoSphere, Talend, Microsoft MDS
</deduplication_process>

### 2. Matching

Matching links records across systems that represent the same entity, even when using different identifiers.

<matching_strategies>
**Deterministic Matching**:
Rule-based matching using defined criteria.

**Example Rules**:
- If SSN matches exactly → Same person (confidence 99%)
- If LastName + FirstName + DOB match → Same person (confidence 95%)
- If Email matches exactly → Same person (confidence 90%)
- If Phone + ZipCode match → Same person (confidence 75%)

**Probabilistic Matching**:
Statistical algorithms calculate match probability.

**Approach**:
1. Compare each attribute between record pairs
2. Assign match weight based on attribute discriminating power
3. Sum weights to calculate total match score
4. Apply threshold: High score = match, medium = review, low = non-match

**Example - Customer Matching Across Systems**:
```
CRM System:     CustomerID=C-5678
Sales System:   AccountID=A-9012
Service System: ClientID=CL-3456

Matching Process:
- Name: "John M. Smith" vs "J. Smith" vs "Smith, John" → Partial match (weight 30)
- Address: "123 Main St" vs "123 Main Street" vs "123 Main St" → Match (weight 40)
- Phone: "617-555-1234" vs "617-555-1234" vs NULL → Partial match (weight 20)
- DOB: "1975-03-15" vs NULL vs "03/15/1975" → Partial match (weight 10)

Total Score: 85/100 → Match confirmed
Result: Link C-5678 = A-9012 = CL-3456 (same customer)
```

**Cross-Reference Table**:
Maintain mappings between source system keys and master golden record key.

```
Golden_Customer_Key | Source_System | Source_Customer_ID
--------------------|---------------|-------------------
GC-00001           | CRM           | C-5678
GC-00001           | Sales         | A-9012
GC-00001           | Service       | CL-3456
```
</matching_strategies>

### 3. Survivorship

Survivorship determines which attribute values survive when merging duplicate or matched records into a golden record.

<survivorship_rules>
**Common Survivorship Strategies**:

**1. Source System Priority**:
Establish a hierarchy of authoritative sources by attribute.

```
Customer Name:       CRM system (most reliable)
Shipping Address:    Order system (most current)
Credit Score:        Finance system (official)
Email:               Marketing system (validated)
Phone:               Service system (customer-verified)
```

**2. Most Recent Value**:
Use the value with the latest timestamp.

```
Address changes over time:
2023-01-15: 123 Main St, Boston MA     (from Sales)
2023-06-20: 456 Oak Ave, Boston MA     (from CRM)
2023-09-10: 789 Elm St, Cambridge MA   (from Service)

Survived Value: 789 Elm St, Cambridge MA (most recent)
```

**3. Most Complete Value**:
Prefer non-null, non-blank values.

```
Source 1: FirstName="John", MiddleInit=NULL, LastName="Smith"
Source 2: FirstName="John", MiddleInit="M", LastName="Smith"
Source 3: FirstName="J", MiddleInit=NULL, LastName="Smith"

Survived Value: FirstName="John", MiddleInit="M", LastName="Smith"
(Combines most complete values)
```

**4. Most Frequent Value**:
Use the value appearing most often across sources.

```
Customer Status across 5 systems:
System 1: "Gold"
System 2: "Gold"
System 3: "Gold"
System 4: "Silver"
System 5: NULL

Survived Value: "Gold" (appears 3 of 4 times)
```

**5. Longest Value**:
Prefer longer strings (often more descriptive).

```
Company Name variants:
"IBM"
"IBM Corp"
"International Business Machines Corporation"

Survived Value: "International Business Machines Corporation"
```

**6. Business Rule-Based**:
Apply domain-specific logic.

```
Annual Income (choose highest as customers tend to inflate):
Source 1: $75,000
Source 2: $82,000
Source 3: $78,000

Survived Value: $82,000 (max value rule)
```

**Multi-Attribute Survivorship Example**:

```
Golden Customer Record Creation:

Attribute           | CRM      | Sales    | Service  | Survivorship Rule      | Survived Value
--------------------|----------|----------|----------|------------------------|---------------
Customer Name       | J. Smith | John S.  | John Smith| CRM Priority          | J. Smith
Shipping Address    | NULL     | 123 Main | 123 Main | Most Complete         | 123 Main St
Email               | js@x.com | NULL     | jsmith@y | Most Recent (CRM newer)| js@x.com
Phone               | 617-5551 | 617-5551 | 617-5552 | Most Frequent         | 617-5551
Status              | Gold     | Silver   | Gold     | Source Priority (CRM) | Gold
Last Purchase Date  | 2023-11  | 2023-12  | NULL     | Most Recent           | 2023-12-15
```

**Maintain Data Lineage**:
Track which source provided each survived attribute value.

```
Customer Dimension Row:
Customer_Key:     GC-00001
Customer_Name:    "John Smith"  (source: CRM, date: 2023-09-01)
Email:            "jsmith@x.com" (source: Marketing, date: 2023-11-15)
Phone:            "617-555-1234" (source: Service, date: 2023-08-20)
Address:          "123 Main St"  (source: Sales, date: 2023-12-01)
```
</survivorship_rules>

### 4. Conforming (ETL Subsystem 8)

Conforming standardizes attribute content and structure across all instances of a dimension.

<conforming_process>
**Structural Conforming**:
Align table structures so identical dimensions have identical schemas.

**Steps**:
1. Define standard dimension table structure (columns, data types, names)
2. Map source system attributes to standard structure
3. Transform data during ETL to match standard schema
4. Ensure all instances share identical column names and definitions

**Example - Customer Dimension Conforming**:

```
Standard Structure (Target):
Customer_Key (INT)
Customer_ID (VARCHAR)
Customer_Name (VARCHAR)
Customer_Type (VARCHAR) - domain: Individual|Commercial
City (VARCHAR)
State (CHAR 2)
Country (VARCHAR)
Customer_Since_Date (DATE)

Source System 1 Mapping:
CUST_NO        → Customer_ID
CUST_NM        → Customer_Name
CUST_TYPE      → Customer_Type (code translation required)
CITY_NAME      → City
STATE_CD       → State
COUNTRY_NAME   → Country
ACCT_OPEN_DT   → Customer_Since_Date

Source System 2 Mapping:
AccountID      → Customer_ID
FullName       → Customer_Name
AccountType    → Customer_Type (translation required)
ShipCity       → City
ShipState      → State
ShipCountry    → Country
CreateDate     → Customer_Since_Date
```

**Content Conforming**:
Standardize attribute values and domain contents.

**Techniques**:

1. **Code Translation**: Map source codes to standard codes
```
Customer Type Conforming:
Source 1: "P" → "Individual"
Source 1: "B" → "Commercial"
Source 2: "PERSON" → "Individual"
Source 2: "COMPANY" → "Commercial"
Source 3: "Retail" → "Individual"
Source 3: "Wholesale" → "Commercial"
```

2. **Value Standardization**: Apply consistent formatting
```
State Conforming:
"Massachusetts" → "MA"
"MASS" → "MA"
"Mass." → "MA"
"ma" → "MA"

Phone Conforming:
"(617) 555-1234" → "617-555-1234"
"617.555.1234" → "617-555-1234"
"6175551234" → "617-555-1234"
```

3. **Reference Data Alignment**: Use standard lookup tables
```
Country Conforming (ISO 3166-1):
"USA" → "United States" (country code: US)
"U.S.A." → "United States" (country code: US)
"United States of America" → "United States" (country code: US)
```

4. **Hierarchical Conforming**: Ensure consistent hierarchy relationships
```
Product Hierarchy Conforming:

Source 1:                    Source 2:                   Conformed:
SKU → Brand → Category      Item → Line → Group         SKU → Brand → Category → Department

Map "Line" to "Brand"
Map "Group" to "Category"
Add "Department" level to Source 1
```

5. **Attribute Enrichment**: Add standard attributes from authoritative sources
```
Customer dimension conforming with external data:

Internal Data:
Customer Name: "John Smith"
Address: "123 Main St, Boston MA 02108"

External Enrichment (Census, demographic provider):
ZIP Code: "02108"
→ Add: County: "Suffolk"
→ Add: Metro Area: "Boston-Cambridge-Newton MSA"
→ Add: Census Tract: "25025000100"
→ Add: Median Household Income: "$75,000"
→ Add: Population Density: "High"
```

**Incremental Conformity**:
You don't need to conform all attributes immediately. Use an agile approach:

**Phase 1**: Conform minimal integration attributes
- Customer Category (Individual/Commercial)
- Geographic attributes (City, State, Country)

**Phase 2**: Add more conformed attributes
- Customer Status (Active/Inactive/Closed)
- Customer Segment (Premium/Standard/Basic)

**Phase 3**: Expand to comprehensive conformity
- All demographic attributes
- All behavioral scores
- All relationship attributes

As each attribute is conformed and added to dimension tables across business processes, the scope of drill-across integration expands.
</conforming_process>

---

## Implementing Downstream MDM in the Data Warehouse

When implementing MDM logic in the data warehouse ETL system, follow this systematic process:

### Step 1: Establish Data Governance

<governance_foundation>
**Critical Success Factor**: MDM requires business-led data governance, not just IT implementation.

**Governance Activities**:

1. **Identify Business Stakeholders**
   - Executive sponsor (VP/SVP level)
   - Data stewards per domain (Customer, Product, Supplier)
   - Subject matter experts from each source system
   - Data quality analysts
   - Compliance/legal representatives (for PII, GDPR)

2. **Define Governance Scope**
   - Which entities require MDM? (Customer, Product, Supplier, Location, Employee)
   - Which source systems are in scope?
   - Which attributes must conform?
   - What is the authoritative source per attribute?

3. **Establish Governance Policies**
   - Data ownership and stewardship roles
   - Survivorship rules by attribute
   - Data quality standards and acceptance criteria
   - Change management procedures
   - Issue escalation paths

4. **Create Governance Artifacts**
   - Conformed dimension specifications
   - Attribute business glossary
   - Survivorship rule documentation
   - Source system data mapping
   - Data quality scorecards

**Example Governance Document - Customer Master**:

```
CUSTOMER DIMENSION GOVERNANCE CHARTER

Data Steward: Jane Doe, VP Customer Experience
Executive Sponsor: John Smith, Chief Data Officer

Scope: All customer-facing systems (CRM, Sales, Service, Billing)

Authoritative Sources by Attribute:
- Customer Name: CRM (highest data quality, customer verified)
- Billing Address: Billing System (financial system of record)
- Shipping Address: Order Management (most current)
- Email: Marketing (validated via email campaigns)
- Phone: Customer Service (customer-verified during interactions)
- Credit Score: Finance System (official credit bureau data)

Survivorship Rules:
- Name: CRM priority, unless NULL then Sales system
- Address: Most recent non-NULL value
- Contact info: Most complete non-NULL value
- Demographics: External data provider priority

Data Quality Standards:
- Name: Must not contain special characters or numbers
- Address: Must validate against postal authority
- Email: Must pass RFC 5322 validation
- Phone: Must be 10 digits (US) with valid area code

Update Frequency: Daily batch (11 PM ET)

Issue Escalation: Data quality issues > 5% trigger steward review
```
</governance_foundation>

### Step 2: Profile Source Systems

<data_profiling>
Before building MDM logic, thoroughly understand source data realities.

**Profiling Activities**:

1. **Identify all customer (or other entity) sources**
   ```
   Customer Sources Inventory:
   - CRM database (Salesforce): 250,000 customer records
   - Order Management (Oracle): 180,000 customer records
   - Customer Service (Custom): 95,000 customer records
   - Billing System (SAP): 220,000 customer records
   - External: D&B company data (50,000 commercial customers)
   ```

2. **Assess data quality per source**
   - Completeness: % of NULL values per attribute
   - Validity: % of values passing business rules
   - Consistency: % of values matching standard formats
   - Uniqueness: Duplicate record rate within source

3. **Analyze matching potential**
   - Identify common keys across systems
   - Assess name/address quality for fuzzy matching
   - Calculate expected match rates
   - Identify unmatchable orphan records

4. **Estimate overlap and gaps**
   ```
   Expected Customer Counts After Deduplication:

   Total source records: 795,000
   Est. duplicates within sources: -45,000 (5.7%)
   Est. duplicates across sources: -150,000 (20%)
   Est. unique customers: ~600,000

   Overlap Analysis:
   - In CRM only: ~75,000 (prospects, no orders)
   - In Order only: ~25,000 (one-time buyers, not in CRM)
   - In Service only: ~5,000 (warranty claims, no purchase)
   - In multiple systems: ~495,000 (active customers)
   ```

**Profiling Deliverables**:
- Data quality scorecard per source
- Matching key analysis
- Survivorship source recommendations
- Expected golden record counts
- Data integration complexity assessment
</data_profiling>

### Step 3: Design Integration Architecture

<integration_architecture>
**Architecture Components**:

1. **Staging Area**: Raw extracts from all source systems
2. **Integration Area**: MDM processing (match, merge, conform)
3. **Master Tables**: Golden records for each entity type
4. **Cross-Reference Tables**: Source key to master key mappings
5. **Presentation Area**: Conformed dimensions for BI

**Example Architecture - Customer MDM**:

```
[Source Systems]
     ↓
[Staging Layer]
  STG_CRM_Customer
  STG_Sales_Customer
  STG_Service_Customer
  STG_Billing_Customer
     ↓
[Integration Layer - MDM Processing]
  INT_Customer_Matched        (Deduplication & Matching)
  INT_Customer_Survived       (Survivorship applied)
  INT_Customer_Conformed      (Conforming transformations)
     ↓
[Master Data Layer]
  MDM_Customer_Master         (Golden records)
  MDM_Customer_XRef           (Source key mappings)
     ↓
[Presentation Layer]
  DIM_Customer                (Conformed customer dimension)
  DIM_Customer_SCD_History    (Type 2 change tracking)
```

**Key Tables**:

**MDM_Customer_Master** (Golden Records):
```
Customer_Key           INT PK (surrogate key)
Customer_Name          VARCHAR(100)
Customer_Type          VARCHAR(20)
Billing_Address        VARCHAR(200)
Shipping_Address       VARCHAR(200)
Email                  VARCHAR(100)
Phone                  VARCHAR(20)
...
Golden_Record_Created_Date   DATE
Golden_Record_Updated_Date   DATE
Data_Quality_Score           DECIMAL(5,2)
```

**MDM_Customer_XRef** (Cross-Reference):
```
Customer_Key           INT FK (to MDM_Customer_Master)
Source_System          VARCHAR(50)
Source_Customer_ID     VARCHAR(50)
Source_Record_Created  DATE
Match_Confidence       DECIMAL(5,2)
Relationship_Type      VARCHAR(20) - (Primary|Duplicate|Linked)
```

**Process Flow**:
```
1. Extract → Load raw data to staging tables (no transformation)

2. Standardize → Cleanse and standardize within each source
   - Parse names and addresses
   - Standardize phone/email formats
   - Validate against business rules
   - Flag quality issues

3. Match → Identify duplicates and cross-system matches
   - Intra-source deduplication (within CRM, within Sales, etc.)
   - Cross-source matching (CRM customer = Sales customer?)
   - Generate match clusters (groups of matched records)

4. Survive → Apply survivorship rules to create golden records
   - One golden record per match cluster
   - Best attribute value per survivorship rules
   - Maintain source lineage per attribute

5. Conform → Apply enterprise standardization
   - Map to standard codes and values
   - Enrich with external reference data
   - Build conformed dimension structure

6. Load → Populate presentation layer
   - DIM_Customer with current golden record state
   - Track Type 2 SCD changes over time
   - Update cross-reference table
```
</integration_architecture>

### Step 4: Implement ETL Processing

<etl_implementation>
**ETL Subsystems for MDM** (from Chapter 19):

**Subsystem 1: Data Profiling**
- Profile each source during development
- Ongoing quality monitoring in production
- Alert on quality threshold violations

**Subsystem 4: Data Cleansing**
- Name/address parsing and standardization
- Format standardization (phone, email, dates)
- Code translation and validation
- NULL handling and default values

**Subsystem 5: Error Event Schema**
- Track all data quality exceptions
- Tag records with quality scores
- Flag records for steward review

**Subsystem 6: Audit Dimension**
- Track source system provenance per attribute
- Record survivorship decisions
- Capture ETL lineage metadata

**Subsystem 7: Deduplication System**
- Exact and fuzzy matching logic
- Duplicate identification and flagging
- Match confidence scoring

**Subsystem 8: Conforming System**
- Structural conforming (schema alignment)
- Content conforming (value standardization)
- Hierarchy conforming (rollup consistency)
- Attribute enrichment from external sources

**Subsystem 9: Slowly Changing Dimension Manager**
- Detect changes to golden records
- Apply Type 1 (overwrite) for corrections
- Apply Type 2 (new row) for history tracking
- Maintain current flag and effective dates

**Subsystem 10: Surrogate Key Generator**
- Assign surrogate keys to golden records
- Independent of source system keys
- Persist cross-reference mappings

**Key ETL Logic - Customer Master Example**:

```sql
-- Step 1: Standardize CRM customer data
INSERT INTO INT_Customer_Standardized
SELECT
    'CRM' AS Source_System,
    Customer_ID AS Source_Customer_ID,
    UPPER(TRIM(Customer_Name)) AS Customer_Name_Std,
    CASE Customer_Type
        WHEN 'P' THEN 'Individual'
        WHEN 'B' THEN 'Commercial'
    END AS Customer_Type_Std,
    -- Address parsing and standardization
    Parse_Street(Address) AS Street_Std,
    Parse_City(Address) AS City_Std,
    Standardize_State(Parse_State(Address)) AS State_Std,
    -- etc...
FROM STG_CRM_Customer
WHERE Load_Date = CURRENT_DATE;

-- Step 2: Fuzzy match to identify duplicates
INSERT INTO INT_Customer_Matched
SELECT
    a.Source_System AS Source_System_A,
    a.Source_Customer_ID AS Source_ID_A,
    b.Source_System AS Source_System_B,
    b.Source_Customer_ID AS Source_ID_B,
    Calculate_Match_Score(
        a.Customer_Name_Std, b.Customer_Name_Std,
        a.Street_Std, b.Street_Std,
        a.City_Std, b.City_Std,
        a.State_Std, b.State_Std,
        a.Phone_Std, b.Phone_Std
    ) AS Match_Score
FROM INT_Customer_Standardized a
JOIN INT_Customer_Standardized b
    ON a.Source_System < b.Source_System  -- Avoid duplicate pairs
WHERE Calculate_Match_Score(...) >= 0.75;  -- Threshold

-- Step 3: Create match clusters (groups of matched records)
-- Use graph traversal to identify all records that match directly or transitively
-- Each cluster gets a unique Cluster_ID

-- Step 4: Apply survivorship rules to create golden record per cluster
INSERT INTO MDM_Customer_Master
SELECT
    NEXT_VAL('Customer_Key_Seq') AS Customer_Key,
    -- Survivorship: CRM has priority for name
    COALESCE(
        MAX(CASE WHEN Source_System='CRM' THEN Customer_Name_Std END),
        MAX(Customer_Name_Std)
    ) AS Customer_Name,
    -- Survivorship: Most recent for address
    MAX(Address_Std) KEEP (DENSE_RANK LAST ORDER BY Source_Update_Date) AS Address,
    -- etc. for each attribute per survivorship rules
    CURRENT_DATE AS Golden_Record_Created_Date,
    AVG(Source_Quality_Score) AS Data_Quality_Score
FROM INT_Customer_Matched_Clusters
GROUP BY Cluster_ID;

-- Step 5: Populate cross-reference table
INSERT INTO MDM_Customer_XRef
SELECT
    m.Customer_Key,
    s.Source_System,
    s.Source_Customer_ID,
    s.Source_Record_Created,
    c.Match_Score AS Match_Confidence,
    CASE
        WHEN s.Source_Customer_ID = m.Primary_Source_ID THEN 'Primary'
        ELSE 'Duplicate'
    END AS Relationship_Type
FROM MDM_Customer_Master m
JOIN INT_Customer_Matched_Clusters c ON m.Customer_Key = c.Customer_Key
JOIN INT_Customer_Standardized s ON c.Source_Customer_ID = s.Source_Customer_ID;
```
</etl_implementation>

### Step 5: Handle Special MDM Scenarios

<special_scenarios>
**Late Arriving Source Records**

New source system added after MDM established.

**Challenge**: Existing golden records may match new source records.

**Solution**:
1. Run matching between new source and existing master
2. For matches: Add to existing golden record cluster, reapply survivorship
3. For non-matches: Create new golden records
4. Update cross-reference table with new source

**Source System Decommissioning**

Source system retired and replaced.

**Challenge**: Golden records may lose their authoritative source.

**Solution**:
1. Before decommission: Snapshot final state from old system
2. Map replacement system attributes to same governance rules
3. Update survivorship rules if authoritative source changed
4. Add new source to cross-reference, mark old source as inactive

**Mergers and Acquisitions**

Organization acquires another company with separate customer base.

**Challenge**: Combine customer bases, identify overlapping customers.

**Solution**:
1. Integrate acquired company source systems into ETL
2. Run full matching between existing and acquired customers
3. Apply survivorship, giving priority based on business rules
4. Create consolidated golden records
5. Track acquisition via metadata attribute

**External Data Integration**

Enriching master data with third-party sources (D&B, credit bureaus, demographic providers).

**Challenge**: External data has different keys and update cycles.

**Solution**:
1. Match external records to internal golden records (fuzzy matching on name/address)
2. Add external attributes to golden records via survivorship
3. Track external source and as-of-date in audit dimension
4. Refresh external data on appropriate cycle (weekly/monthly)
5. Handle unmatched external records (create placeholder or ignore)

**Customer Merge/Split Events**

Business process merges two customers or splits one customer into multiple.

**Challenge**: Maintain referential integrity with historical facts.

**Solution for Merge**:
1. Create new golden record combining both customers
2. Add both old customer keys to cross-reference table pointing to new key
3. Issue Type 2 dimension change for both old customers (end-date rows)
4. Create new Type 2 dimension row for merged customer
5. Historical facts retain original customer keys (accurate history)
6. New facts use merged customer key

**Solution for Split**:
1. Create two new golden records
2. Determine business rules for splitting attributes and fact history
3. Update cross-reference: original key maps to both new keys
4. Historical facts: Allocate to one or both customers based on business rules
5. New facts use appropriate new customer key
</special_scenarios>

---

## Integration with Conformed Dimension Strategy

MDM is not separate from dimensional modeling—it is the implementation mechanism for conformed dimensions.

<mdm_conformed_dimensions>
**The Relationship**:

**Enterprise Data Warehouse Bus Matrix** defines which dimensions must be conformed across business processes.

**Master Data Management** is the ETL technique for creating and maintaining those conformed dimensions.

**Integration Pattern**:

```
[Bus Matrix] → Identifies conformed dimensions required (Customer, Product, Date, Location)
      ↓
[Data Governance] → Establishes business ownership and standards
      ↓
[MDM Processing] → Deduplicates, matches, survives, conforms source data
      ↓
[Conformed Dimensions] → Customer, Product, Location dimensions used by all fact tables
      ↓
[Drill-Across Queries] → Enabled by consistent dimension attributes
```

**Dimension Manager System (ETL Subsystem 17)**:

The dimension manager owns the conformed dimension and publishes it to fact providers.

**Responsibilities**:
1. Extract from MDM master tables (golden records)
2. Apply SCD logic (Type 1, Type 2, etc.)
3. Manage surrogate key assignments
4. Publish conformed dimension to dimensional model
5. Notify fact providers of dimension changes
6. Coordinate dimension updates across fact tables

**Example - Customer Dimension Manager**:

```
MDM_Customer_Master (Golden Records)
        ↓
DIM_Customer (Conformed Dimension with Type 2 SCD)
        ↓
[Published to all customer-facing fact tables]:
  - FACT_Sales
  - FACT_Customer_Service
  - FACT_Marketing_Campaign
  - FACT_Website_Visits
  - FACT_Product_Returns

All fact tables share same DIM_Customer with identical:
- Customer_Key (surrogate key)
- Customer attributes (name, type, segment, status, geography)
- SCD Type 2 history (effective dates, current flag)

Result: Drill-across queries work seamlessly
```

**Fact Provider System (ETL Subsystem 18)**:

Fact providers consume conformed dimensions published by dimension managers.

**Process**:
1. Receive notification of dimension updates from dimension manager
2. Look up current dimension surrogate key for fact's natural key
3. Load fact with dimension's surrogate key (referential integrity)
4. Handle late-arriving dimensions with provisional dimension rows

**Partial Conformity for Agile Implementation**:

You don't need full MDM for all attributes before starting. Use incremental conformity:

**Iteration 1**: Conform Customer Category
- Add "Customer_Category" attribute to customer dimensions across business processes
- Enables limited drill-across (e.g., sales + service by customer category)

**Iteration 2**: Add geographic conformity
- Conform City, State, Country attributes
- Enables geographic drill-across analysis

**Iteration 3**: Add behavioral conformity
- Conform Customer Status, Customer Segment
- Enables segmentation and status-based drill-across

**Iteration 4**: Expand to comprehensive conformity
- All name/address attributes
- All demographic/firmographic attributes
- External data enrichment

Each iteration expands integration scope without breaking existing applications.
</mdm_conformed_dimensions>

---

## Common Mistakes and How to Avoid Them

<common_mistakes>
**1. IT-Driven MDM Without Business Governance**

**Mistake**: IT team implements MDM technical solution without business stewardship and governance.

**Consequence**:
- Arbitrary survivorship rules not aligned with business needs
- Lack of organizational buy-in and adoption
- Data quality issues persist due to no source accountability
- Conformed dimensions don't reflect business definitions

**Solution**:
- Establish business-led data governance before technical implementation
- Identify executive sponsor and data stewards
- Document business-driven survivorship rules
- Gain consensus on conformed attribute definitions

**2. Overly Ambitious Scope**

**Mistake**: Attempt to conform all entities (customer, product, supplier, location, employee) and all attributes simultaneously.

**Consequence**:
- Project takes years with no interim value delivery
- Organizational fatigue and loss of momentum
- Requirements change before delivery
- Agile development paralyzed

**Solution**:
- Start with one high-value entity (typically Customer)
- Conform critical attributes first (category, geography)
- Deliver iteratively, expanding scope over time
- Prove value before expanding to additional entities

**3. Perfect Matching Expectation**

**Mistake**: Expect 100% accurate automated matching without manual review.

**Consequence**:
- False positives (incorrectly merged distinct customers)
- False negatives (duplicates not detected)
- Loss of business confidence in data quality
- Rework to untangle incorrect merges

**Solution**:
- Accept that fuzzy matching is probabilistic, not perfect
- Implement match confidence scoring (high/medium/low)
- Route medium-confidence matches to steward review
- Track false positive/negative rates and tune thresholds
- Provide steward UI for manual merge/unmerge operations

**4. Ignoring Source System Data Quality**

**Mistake**: Rely solely on ETL cleansing to fix poor source data quality.

**Consequence**:
- Ongoing ETL complexity and maintenance burden
- Data quality problems replicate faster than ETL can fix
- Golden records only as good as worst source
- Operational systems continue with bad data

**Solution**:
- Work with source system teams to improve data capture quality
- Implement validation rules at point of data entry
- Publish data quality scorecards to source system owners
- Tie data quality goals to operational KPIs
- Consider upstream MDM if operational quality critical

**5. No Cross-Reference Table**

**Mistake**: Golden records lack mappings back to source system keys.

**Consequence**:
- Cannot trace golden record attributes to source
- Cannot answer "which CRM customer is this?"
- Difficult to debug matching errors
- Cannot integrate new sources to existing masters

**Solution**:
- Always maintain cross-reference table (MDM_Entity_XRef)
- Include source system, source key, match confidence, relationship type
- Enable bidirectional lookup (source → golden, golden → sources)

**6. Monolithic Golden Record Updates**

**Mistake**: Recreate all golden records from scratch daily.

**Consequence**:
- Unnecessary processing time and resource usage
- Surrogate keys change unnecessarily
- Type 2 SCD history lost or incorrect
- Downstream fact tables broken by key changes

**Solution**:
- Implement change data capture at source systems
- Process only changed records incrementally
- Preserve existing golden records and surrogate keys
- Add new golden records only for new match clusters
- Update existing golden records only when attributes change

**7. Survivorship Rules Not Documented**

**Mistake**: Survivorship logic buried in ETL code without documentation.

**Consequence**:
- Business users don't understand data provenance
- Cannot explain why specific attribute value was chosen
- Difficult to modify rules as business needs evolve
- Knowledge loss when ETL developers leave

**Solution**:
- Document survivorship rules in data governance charter
- Include source system priority per attribute
- Maintain data lineage in audit dimension
- Publish survivorship documentation to business users
- Version control governance documents

**8. No Data Quality Monitoring**

**Mistake**: No ongoing measurement of MDM data quality and match accuracy.

**Consequence**:
- Quality degradation goes unnoticed
- Duplicate customers multiply over time
- Source system changes break matching logic
- Business loses confidence in master data

**Solution**:
- Implement data quality scorecards with ongoing metrics
- Monitor: duplicate rate, match rate, null rate, invalid value rate
- Set quality thresholds and alert on violations
- Regular steward review of match results
- Continuous tuning of matching algorithms
</common_mistakes>

---

## MDM Implementation Checklist

Use this checklist to ensure complete MDM implementation:

### Governance and Planning
- [ ] Executive sponsor identified and committed
- [ ] Data stewards appointed for each entity type
- [ ] Governance charter documented with policies and procedures
- [ ] Business glossary created for conformed attributes
- [ ] Survivorship rules documented per attribute
- [ ] Source system authoritative ownership assigned
- [ ] Data quality standards and acceptance criteria defined
- [ ] Escalation procedures established

### Source System Analysis
- [ ] All source systems identified and inventoried
- [ ] Data profiling completed per source
- [ ] Data quality baseline measured
- [ ] Matching keys identified (exact and fuzzy)
- [ ] Overlap analysis completed (expected duplicate rates)
- [ ] Source system update frequency documented
- [ ] Source system stakeholders engaged

### Architecture and Design
- [ ] MDM architecture pattern selected (upstream vs downstream)
- [ ] Staging area designed for raw source extracts
- [ ] Integration area designed for MDM processing
- [ ] Master table structure designed (golden records)
- [ ] Cross-reference table designed (source key mappings)
- [ ] Presentation layer designed (conformed dimensions)
- [ ] Data lineage and audit requirements specified

### ETL Development
- [ ] Subsystem 1 (Data Profiling) implemented
- [ ] Subsystem 4 (Data Cleansing) implemented
- [ ] Subsystem 5 (Error Event Schema) implemented
- [ ] Subsystem 6 (Audit Dimension) implemented
- [ ] Subsystem 7 (Deduplication) implemented
- [ ] Subsystem 8 (Conforming) implemented
- [ ] Subsystem 9 (SCD Manager) implemented
- [ ] Subsystem 10 (Surrogate Key Generator) implemented
- [ ] Change data capture configured for source systems
- [ ] Matching algorithms configured and tuned
- [ ] Survivorship rules implemented in ETL logic
- [ ] Cross-reference table population automated

### Testing and Validation
- [ ] Unit testing completed for all ETL components
- [ ] Match accuracy tested (sample of known matches/non-matches)
- [ ] Survivorship rules validated with business stewards
- [ ] Data quality thresholds validated
- [ ] Type 2 SCD logic tested with change scenarios
- [ ] End-to-end integration testing completed
- [ ] Performance testing completed (volume, timing)
- [ ] Reconciliation between source counts and golden record counts

### Deployment and Operations
- [ ] Production deployment completed
- [ ] Data quality scorecards published
- [ ] Steward review workflows established
- [ ] Monitoring and alerting configured
- [ ] Issue tracking and resolution process established
- [ ] Documentation delivered (technical and business)
- [ ] Training completed for stewards and users
- [ ] Ongoing governance meetings scheduled

### Integration with Dimensional Models
- [ ] Conformed dimensions identified on bus matrix
- [ ] Dimension manager system implemented (Subsystem 17)
- [ ] Fact provider systems updated (Subsystem 18)
- [ ] All fact tables reference conformed dimensions via surrogate keys
- [ ] Drill-across queries tested and validated
- [ ] BI applications updated to use conformed dimensions

---

## Summary

Master Data Management integration with dimensional data warehouses is essential for creating conformed dimensions that enable enterprise integration. Whether implementing upstream MDM in operational systems or downstream MDM in the data warehouse ETL, the core processes remain the same: deduplication to identify duplicates, matching to link records across systems, survivorship to select best attribute values, and conforming to standardize content and structure.

Success requires business-led data governance to establish policies, ownership, and standards. Technical implementation leverages ETL Subsystems 7 (Deduplication) and 8 (Conforming) to create golden records in master tables, with cross-reference tables mapping source system keys. These golden records feed the dimension manager (Subsystem 17) which publishes conformed dimensions to fact providers (Subsystem 18).

The result is an integrated enterprise data warehouse where business users can perform drill-across queries to analyze data from multiple business processes with consistent, conformed dimensions—the foundation of the enterprise data warehouse bus architecture.
