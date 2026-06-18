# Enterprise Bus Matrix Examples by Industry

This reference provides complete, real-world enterprise data warehouse bus matrix examples across multiple industries. Each example illustrates how business processes (rows) and conformed dimensions (columns) are identified and mapped to create an integrated DW/BI architecture.

## How to Use These Examples

1. **Find similar industry** - Locate an example closest to your organization
2. **Adapt processes** - Modify rows to match your business processes
3. **Customize dimensions** - Adjust columns for your specific dimensions
4. **Mark intersections** - Shade cells where dimensions apply to processes
5. **Validate with business** - Review with stakeholders to ensure completeness

## Retail Industry Bus Matrix

### Context
Large grocery chain with 100+ stores, centralized distribution centers, online ordering, and frequent shopper program.

### Bus Matrix

```
Business Process          | Date | Product | Store | Customer | Promotion | Vendor | Employee | Payment Method | Shipper
--------------------------|------|---------|-------|----------|-----------|--------|----------|----------------|--------
POS Retail Sales          |  X   |    X    |   X   |    X     |     X     |        |          |       X        |
Store Inventory Snapshot  | Month|    X    |   X   |          |           |   X    |          |                |
Purchase Orders           |  X   |    X    |       |          |           |   X    |    X     |                |
Shipments Received        |  X   |    X    |       |          |           |   X    |          |                |   X
Product Returns           |  X   |    X    |   X   |    X     |           |        |          |       X        |
Promotion Coverage        |  X   |    X    |   X   |          |     X     |        |          |                |
Frequent Shopper Behavior |  X   |    X    |   X   |    X     |     X     |        |          |                |
E-commerce Orders         |  X   |    X    |       |    X     |     X     |        |          |       X        |
E-commerce Clickstream    |  X   |    X    |       |    X     |           |        |          |                |
Warehouse Inventory       | Daily|    X    | Whse  |          |           |   X    |          |                |
```

### Key Observations

**Process-Centric Rows**: Notice "POS Retail Sales" not "Sales Department" - focuses on operational activity, not org chart.

**Atomic Granularity**: Product dimension at SKU level (most granular), not product category.

**Shrunken Dimensions**:
- Store Inventory uses "Month" grain for Date (monthly snapshots)
- Warehouse Inventory references warehouse locations (subset of Store dimension)

**Multiple Date Grains**: Daily for transactions, Monthly for inventory snapshots - annotated in cells.

**Common Dimensions**: Date, Product, Customer, Promotion used across multiple processes - prime conformance candidates.

---

## Financial Services (Banking) Bus Matrix

### Context
Regional bank with retail branches, commercial lending, online banking, ATM network, and credit card services.

### Bus Matrix

```
Business Process              | Date | Account | Customer | Household | Product | Branch | Employee | Channel | Transaction Type
------------------------------|------|---------|----------|-----------|---------|--------|----------|---------|------------------
Account Monthly Balance       | Month|    X    |    X     |     X     |    X    |   X    |          |         |
Account Transactions          |  X   |    X    |    X     |     X     |    X    |   X    |          |    X    |       X
Loan Originations             |  X   |    X    |    X     |     X     |    X    |   X    |    X     |    X    |
Loan Payment Transactions     |  X   |    X    |    X     |     X     |    X    |        |          |    X    |       X
Credit Card Transactions      |  X   |    X    |    X     |     X     |    X    |        |          |    X    |       X
ATM Transactions              |  X   |    X    |    X     |           |         |        |          |  ATM    |       X
Customer Profitability        | Month|    X    |    X     |     X     |    X    |   X    |          |         |
Branch Daily Activity         |  X   |         |          |           |         |   X    |    X     |         |
Online Banking Sessions       |  X   |    X    |    X     |           |         |        |          | Online  |
Customer Service Calls        |  X   |    X    |    X     |           |         |        |    X     |  Phone  |
```

### Key Observations

**Household Dimension**: Enables joint account analysis and economic unit tracking - unique to banking.

**Product Dimension**: Account types (checking, savings, mortgage, credit card) treated as products.

**Channel Dimension**: Multiple banking channels (branch, ATM, online, phone, mobile) - critical for multi-channel analysis.

**Periodic Snapshots**: Monthly balance snapshots semi-additive across time - track account balances over time.

**Multiple Account Dimensions**: May need separate dimensions for different account types (supertype/subtype pattern).

---

## Healthcare Bus Matrix

### Context
Multi-hospital health system with outpatient clinics, emergency departments, and physician practices.

### Bus Matrix

```
Business Process           | Date | Patient | Provider | Diagnosis | Procedure | Facility | Payer | Pharmacy Item | Episode
---------------------------|------|---------|----------|-----------|-----------|----------|-------|---------------|--------
Patient Encounters         |  X   |    X    |    X     |     X     |           |    X     |   X   |               |
Procedures Performed       |  X   |    X    |    X     |           |     X     |    X     |   X   |               |
Lab Tests Ordered          |  X   |    X    |    X     |           | Lab Test  |    X     |       |               |
Pharmacy Prescriptions     |  X   |    X    |    X     |     X     |           |    X     |   X   |       X       |
Claim Transactions         |  X   |    X    |    X     |     X     |     X     |    X     |   X   |               |
Claim Accumulating Pipeline|Multi |    X    |    X     |     X     |     X     |    X     |   X   |               |   X
Patient Admissions         |  X   |    X    |    X     |     X     |           |    X     |   X   |               |
Patient Census (Daily)     | Daily|    X    |          |           |           |    X     |       |               |
Emergency Dept Visits      |  X   |    X    |    X     |     X     |           |    X     |   X   |               |
Care Episode Tracking      |Multi |    X    |    X     |     X     |     X     |    X     |   X   |               |   X
```

### Key Observations

**HIPAA Compliance**: Patient dimension must support privacy requirements and de-identification.

**Diagnosis Dimension**: ICD-10 codes with hierarchies - complex medical coding system.

**Procedure Dimension**: CPT/HCPCS codes for billable procedures.

**Provider Dimension**: Physicians, nurses, specialists - hierarchies by department and specialty.

**Episode Dimension**: Accumulating snapshot tracks patient journey through care continuum.

**Multiple Date Roles**: Admit date, discharge date, procedure date, claim submission date, payment date.

---

## Manufacturing Bus Matrix

### Context
Discrete manufacturing company producing consumer electronics with global supply chain.

### Bus Matrix

```
Business Process               | Date | Product | Component | Vendor | Customer | Factory | Machine | Employee | Work Order | Defect Type
-------------------------------|------|---------|-----------|--------|----------|---------|---------|----------|------------|------------
Purchase Orders                |  X   |         |     X     |   X    |          |         |         |          |            |
Material Receipts              |  X   |         |     X     |   X    |          |   X     |         |          |            |
Production Work Orders         |  X   |    X    |           |        |          |   X     |         |    X     |     X      |
Production Runs                |  X   |    X    |     X     |        |          |   X     |    X    |    X     |     X      |
Quality Inspections            |  X   |    X    |     X     |        |          |   X     |    X    |    X     |            |    X
Finished Goods Inventory       | Daily|    X    |           |        |          |   X     |         |          |            |
Component Inventory            | Daily|         |     X     |   X    |          |   X     |         |          |            |
Equipment Maintenance          |  X   |         |           |        |          |   X     |    X    |    X     |            |
Machine Downtime Events        |  X   |         |           |        |          |   X     |    X    |          |            |
Product Shipments              |  X   |    X    |           |        |    X     |   X     |         |          |            |
Customer Orders                |  X   |    X    |           |        |    X     |         |         |          |            |
Product Defects/Returns        |  X   |    X    |     X     |        |    X     |         |         |          |            |    X
```

### Key Observations

**Bill of Materials (BOM)**: Component dimension with parent-child relationships to Product dimension.

**Work Order Degenerate Dimension**: Work order number in fact table without separate dimension.

**Machine Dimension**: Manufacturing equipment with maintenance schedules and utilization tracking.

**Two Inventory Types**: Finished goods vs. raw materials/components - different grains and dimensionality.

**Quality Dimensions**: Defect types, inspection results - critical for Six Sigma analysis.

---

## Insurance Bus Matrix

### Context
Property & casualty insurance carrier offering auto, home, and commercial policies.

### Bus Matrix

```
Business Process                | Date | Policy | Policyholder | Agent | Coverage | Covered Item | Claimant | Claim | Vendor | Transaction Type
--------------------------------|------|--------|--------------|-------|----------|--------------|----------|-------|--------|------------------
Policy Transactions             |  X   |   X    |      X       |   X   |    X     |      X       |          |       |        |       X
Premium Periodic Snapshot       | Month|   X    |      X       |   X   |    X     |      X       |          |       |        |
Claim Transactions              |  X   |   X    |      X       |   X   |    X     |      X       |    X     |   X   |   X    |       X
Claim Accumulating Snapshot     | Multi|   X    |      X       |   X   |    X     |      X       |    X     |   X   |        |
Policy/Claim Consolidated       | Month|   X    |      X       |   X   |    X     |      X       |          |   X   |        |
Accident Involvements (Factless)|  X   |        |      X       |       |          |Vehicle/Party |          |       |        |
Underwriting Submissions        |  X   |   X    |      X       |   X   |    X     |      X       |          |       |        |       X
Commission Payments             |  X   |   X    |      X       |   X   |          |              |          |       |        |
Policy Renewal Analysis         | Month|   X    |      X       |   X   |    X     |              |          |       |        |
```

### Key Observations

**Pay-in-Advance Revenue**: Premium snapshot tracks written premium vs. earned premium (unearned premium liability).

**Accumulating Snapshot**: Claim pipeline with multiple milestone dates (opened, reserved, investigated, paid, closed).

**Heterogeneous Products**: Auto, home, commercial policies have different attributes - supertype/subtype pattern.

**Covered Item Dimension**: Insured assets (vehicles, properties) with characteristics and valuations.

**Factless Fact Table**: Accident involvements track many-to-many party/vehicle relationships without metrics.

---

## Telecommunications Bus Matrix

### Context
Wireless telecommunications provider with mobile voice, data, and messaging services.

### Bus Matrix

```
Business Process              | Date | Customer | Account | Device | Cell Tower | Plan | Employee | Store | Call Type | Location
------------------------------|------|----------|---------|--------|------------|------|----------|-------|-----------|----------
Call Detail Records (CDR)     |  X   |    X     |    X    |   X    |     X      |  X   |          |       |     X     |    X
Data Session Records          |  X   |    X     |    X    |   X    |     X      |  X   |          |       |           |    X
SMS/MMS Messaging             |  X   |    X     |    X    |   X    |     X      |  X   |          |       |           |
Customer Monthly Billing      | Month|    X     |    X    |   X    |            |  X   |          |       |           |
Service Activations           |  X   |    X     |    X    |   X    |            |  X   |    X     |   X   |           |
Customer Service Calls        |  X   |    X     |    X    |        |            |      |    X     |       |           |
Network Performance Snapshot  |  X   |          |         |        |     X      |      |          |       |           |    X
Device Inventory              | Daily|          |         |   X    |            |      |          |   X   |           |
Churn Prediction Snapshot     | Month|    X     |    X    |        |            |  X   |          |       |           |
```

### Key Observations

**Massive Volume**: CDR fact table extremely large - billions of rows annually.

**Cell Tower Dimension**: Network infrastructure dimension for coverage and capacity analysis.

**Device Dimension**: Phones, tablets, IoT devices with make, model, capabilities.

**Multiple Call Types**: Voice, data session, SMS - may require separate fact tables due to different dimensionality.

**Location Dimension**: Geographic location at cell tower granularity for coverage maps.

---

## E-Commerce Bus Matrix

### Context
Online retailer with website, mobile app, marketplace sellers, and fulfillment centers.

### Bus Matrix

```
Business Process              | Date | Product | Customer | Session | Page | Event | Referral | Payment | Shipper | Seller | Warehouse
------------------------------|------|---------|----------|---------|------|-------|----------|---------|---------|--------|----------
Website Page Views            |  X   |    X    |    X     |    X    |  X   |   X   |    X     |         |         |        |
Website Events (Cart/Checkout)|  X   |    X    |    X     |    X    |  X   |   X   |          |         |         |        |
Website Session Summary       |  X   |    X    |    X     |    X    |      |       |    X     |         |         |        |
Order Transactions            |  X   |    X    |    X     |         |      |       |          |    X    |    X    |   X    |
Order Fulfillment Pipeline    | Multi|    X    |    X     |         |      |       |          |         |    X    |   X    |   X
Product Returns               |  X   |    X    |    X     |         |      |       |          |         |         |   X    |   X
Product Inventory             | Daily|    X    |          |         |      |       |          |         |         |   X    |   X
Customer Reviews              |  X   |    X    |    X     |         |      |       |          |         |         |        |
Marketplace Seller Transactions|  X  |    X    |    X     |         |      |       |          |    X    |         |   X    |
Promotional Campaign Results  |  X   |    X    |    X     |         |      |       |          |         |         |        |
```

### Key Observations

**Clickstream Dimensions**: Page, Event, Session, Referral - unique to web analytics.

**Session Identification**: Cookies and visitor tracking - complex identity resolution.

**Two Fact Table Grains**: Page-level events (atomic) vs. session-level summary (aggregated).

**GMT + Local Time**: Dual timestamps for global synchronization and visitor timezone.

**Marketplace Seller**: Third-party sellers dimension for multi-vendor platforms.

---

## Higher Education Bus Matrix

### Context
University with undergraduate, graduate, and continuing education programs.

### Bus Matrix

```
Business Process              | Date | Student | Course | Instructor | Department | Building | Room | Term | Degree Program | Financial Aid
------------------------------|------|---------|--------|------------|------------|----------|------|------|----------------|---------------
Course Registrations          |  X   |    X    |   X    |     X      |     X      |    X     |  X   |  X   |       X        |
Daily Class Attendance        |  X   |    X    |   X    |     X      |     X      |    X     |  X   |  X   |                |
Grade Assignments             |  X   |    X    |   X    |     X      |     X      |          |      |  X   |       X        |
Student Enrollment Snapshot   | Term |    X    |        |            |     X      |          |      |  X   |       X        |      X
Tuition Payments              |  X   |    X    |        |            |            |          |      |  X   |                |      X
Financial Aid Disbursements   |  X   |    X    |        |            |            |          |      |  X   |                |      X
Residence Hall Assignments    |  X   |    X    |        |            |            | Residence| Room |  X   |                |
Student Applicant Pipeline    | Multi|    X    |        |            |     X      |          |      |      |       X        |
Alumni Giving                 |  X   |  Alumni |        |            |            |          |      |      |       X        |
Faculty Workload              | Term |         |   X    |     X      |     X      |          |      |  X   |                |
```

### Key Observations

**Term Dimension**: Academic calendar with semesters, quarters, summer sessions.

**Student Dimension**: Slowly changing as students progress - classification (freshman, sophomore), GPA, major.

**Course Offering**: Course catalog vs. course sections (specific time/instructor/room).

**Factless Fact Tables**: Registrations and attendance may have no measured facts.

**Accumulating Snapshot**: Applicant pipeline tracks admission journey (applied → admitted → enrolled).

---

## Opportunity/Stakeholder Matrix Example

This variation replaces dimension columns with organizational functions to show cross-functional interest in processes.

### Retail Example

```
Business Process          | Merchandising | Marketing | Finance | Store Ops | Supply Chain | IT
--------------------------|---------------|-----------|---------|-----------|--------------|----
POS Retail Sales          |       X       |     X     |    X    |     X     |              |
Store Inventory Snapshot  |       X       |           |    X    |     X     |      X       |
Purchase Orders           |       X       |           |    X    |           |      X       |
Shipments Received        |       X       |           |         |     X     |      X       |
Product Returns           |       X       |     X     |    X    |     X     |              |
Promotion Coverage        |       X       |     X     |         |           |              |
Frequent Shopper Behavior |       X       |     X     |    X    |           |              |
E-commerce Orders         |       X       |     X     |    X    |           |      X       |  X
```

**Usage**: Identifies stakeholders for requirements sessions, design reviews, and user acceptance testing.

---

## Detailed Implementation Bus Matrix Example

This expands business process rows to show specific fact tables with grain statements.

### Retail Sales Process - Detailed

```
Fact Table                    | Grain Statement                           | Date | Product | Store | Customer | Promotion | Facts
------------------------------|-------------------------------------------|------|---------|-------|----------|-----------|-------
Sales Transaction Fact        | One row per product per POS transaction   |  X   |    X    |   X   |    X     |     X     | Qty, Amount, Cost, Margin
Daily Store Sales Aggregate   | One row per product per store per day     |  X   |    X    |   X   |          |           | Qty, Amount, Transactions
Monthly Product Sales Agg     | One row per product per month             |  X   |    X    |       |          |           | Qty, Amount, Customers
```

**Purpose**: Documents precise grain and facts for each dimensional model during implementation planning.

---

## Common Patterns Across Industries

### Universal Dimensions

These dimensions appear in virtually every industry:
1. **Date** - Present in every fact table, often with role-playing
2. **Time of Day** - Separate dimension when needed for shift analysis
3. **Geography** - Locations, regions, territories, countries

### Industry-Specific Dimension Clusters

**Retail**: Product, Store, Customer, Promotion, Vendor
**Financial Services**: Account, Customer, Product (financial), Branch, Channel
**Healthcare**: Patient, Provider, Diagnosis, Procedure, Payer, Facility
**Manufacturing**: Product, Component, Factory, Machine, Work Order
**Insurance**: Policy, Policyholder, Coverage, Claim, Agent
**Telecommunications**: Customer, Account, Device, Plan, Cell Tower

### Fact Table Type Patterns by Process

**Transaction Grain**: Sales, orders, payments, calls, clicks, shipments
**Periodic Snapshot**: Account balances, inventory levels, enrollment counts
**Accumulating Snapshot**: Order fulfillment, claim processing, student applications, loan originations

---

## Matrix Evolution Strategy

### Phase 1: First Project (Weeks 1-16)
- Implement **one business process row** (high impact, high feasibility)
- Build **core conformed dimensions** (Date, primary master data)
- Deliver BI applications
- Prove DW/BI value

### Phase 2: Second Project (Weeks 17-28)
- Implement **second business process row**
- **Reuse** existing conformed dimensions
- Add **new conformed dimensions** as needed
- Demonstrate drill-across integration

### Phase 3: Subsequent Projects
- Continue **one process at a time**
- **Dimension library grows** with each project
- **Integration increases** - more drill-across possibilities
- **Velocity accelerates** - less dimension development per project

---

## Validation Checklist

Use this checklist to validate your bus matrix:

**Process Rows (Vertical Scan)**:
- [ ] Rows represent operational business processes, not departments
- [ ] Each row recognizable by source system that captures the data
- [ ] Rows are neither too broad ("All Finance") nor too narrow (specific report)
- [ ] Each row will translate into one or more dimensional models

**Dimension Columns (Horizontal Scan)**:
- [ ] Columns represent atomic-level dimensions, not hierarchy levels
- [ ] No overly generic dimensions lumping unrelated populations
- [ ] Dimension names unique across all columns
- [ ] Each column represents data requiring enterprise governance

**Cell Markings**:
- [ ] Shaded cells indicate dimension relevance to process
- [ ] Grain variations noted (Daily, Monthly, Brand-level, etc.)
- [ ] Empty cells represent dimensions not applicable to process

**Overall Architecture**:
- [ ] Matrix reviewed with business management
- [ ] Conformed dimensions identified and prioritized
- [ ] Data stewards assigned to dimensions
- [ ] Implementation sequence prioritized (impact/feasibility)

---

## Reference

For complete guidance on creating and using enterprise bus matrices, see the main skill document: **enterprise-bus-matrix-design/SKILL.md**
