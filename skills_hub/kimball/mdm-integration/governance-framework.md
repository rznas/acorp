# Data Governance Framework for Master Data Management

This reference provides comprehensive guidance for establishing the business governance structure required for successful MDM.

---

## Why Governance is Critical for MDM Success

**Technical MDM without governance fails** because:
- Arbitrary technical rules don't reflect business reality
- No organizational accountability for data quality
- Conflicting definitions persist across departments
- Business users don't trust or adopt the data
- Political resistance prevents consensus on standards
- Source systems continue creating bad data

**Governance-driven MDM succeeds** because:
- Business stakeholders own the definitions
- Clear accountability for data quality at the source
- Consensus on conformed attributes and values
- Business validation ensures accuracy
- Executive sponsorship drives adoption
- Source system improvements complement MDM

**Fundamental Principle**: MDM is 20% technology, 80% organizational change management.

---

## Governance Structure

### Roles and Responsibilities

#### 1. Executive Sponsor

**Level**: VP or SVP with cross-functional authority

**Responsibilities**:
- Provide executive-level support and funding
- Break organizational stalemates on contentious decisions
- Communicate strategic importance to the organization
- Remove political and organizational barriers
- Hold business units accountable for participation
- Review and approve governance charter and policies

**Time Commitment**: 2-4 hours/month for governance council meetings

**Example**: Chief Data Officer, VP of Enterprise Architecture, SVP of Customer Experience

**Critical Success Factor**: Must have authority to compel cooperation across business units.

---

#### 2. Data Governance Council

**Members**:
- Executive Sponsor (Chair)
- Business Unit Leaders (VP/Director level)
- IT Leadership (CIO or delegate)
- Data Stewards (see below)
- Legal/Compliance representative
- Data Quality Manager

**Responsibilities**:
- Approve data governance policies and standards
- Resolve cross-functional conflicts on data definitions
- Prioritize MDM scope and roadmap
- Review data quality scorecards and metrics
- Authorize changes to conformed dimensions
- Approve exceptions to governance policies

**Meeting Frequency**: Monthly

**Decision-Making Authority**:
- Majority vote for routine decisions
- Executive Sponsor breaks ties
- Escalate major conflicts to C-level if needed

**Example Agenda**:
```
Data Governance Council Meeting - March 2024

1. Review February Data Quality Scorecard
   - Customer duplicate rate: 2.3% (target <3%) ✓
   - Product missing category: 8.1% (target <5%) ✗

2. Approve Conformed Attribute Addition
   - Proposal: Add "Customer Lifetime Value Segment" to customer dimension
   - Business case: Enable segmented marketing campaigns
   - Impact: 3 business processes, 2-week implementation
   - Decision: Approved unanimously

3. Resolve Definition Conflict
   - Issue: Sales defines "Active Customer" as purchase in last 12 months
   - Issue: Marketing defines "Active Customer" as email engagement in last 6 months
   - Discussion: Sales definition more aligned with revenue recognition
   - Decision: Adopt Sales definition, Marketing to use "Engaged Customer" for their metric

4. Review MDM Roadmap
   - Customer MDM: Production (complete)
   - Product MDM: Development (on track, 60% complete)
   - Supplier MDM: Planning (start Q2)
```

---

#### 3. Data Stewards

**One steward per master data domain** (Customer, Product, Supplier, Location, Employee).

**Profile**:
- Senior business analyst or manager with deep domain knowledge
- Respected by peers and stakeholders
- Broad understanding of cross-functional needs
- Strong communication and negotiation skills
- Analytical mindset with attention to detail
- Political savvy to navigate organizational dynamics

**Time Commitment**: 50-100% depending on domain complexity (often dedicated role)

**Customer Data Steward Responsibilities**:

**Define**:
- Define conformed customer attributes and business rules
- Establish customer type taxonomy (Individual, Commercial, Government, etc.)
- Define customer segmentation schemes
- Document customer lifecycle states (Prospect, Active, Inactive, Closed)
- Create customer dimension business glossary

**Enforce**:
- Approve survivorship rules for customer attributes
- Designate authoritative source systems per attribute
- Review and resolve data quality issues
- Approve or reject customer merge/split requests
- Validate match accuracy (sample review)

**Communicate**:
- Train business users on customer dimension usage
- Publish customer dimension documentation
- Communicate changes to customer definitions
- Solicit feedback from business stakeholders
- Escalate unresolved issues to governance council

**Monitor**:
- Track customer data quality metrics
- Monitor customer duplicate rates
- Review unmatched customer records
- Analyze customer dimension usage in BI reports
- Identify improvement opportunities

**Example - Weekly Steward Activities**:
```
Monday:
- Review weekend ETL run: 1,250 new customers, 380 updates, 15 duplicates merged
- Check data quality scorecard: 3 new issues flagged
- Review match confidence: 12 medium-confidence matches for manual review

Tuesday:
- Manual review session: Approve 9 matches, reject 3 as non-matches
- Investigate quality issue: 45 customers missing email (source system bug identified)
- Create ticket for source system team to fix email validation

Wednesday:
- Governance council meeting: Present customer data quality status
- Approve new conformed attribute: Customer Preferred Contact Method

Thursday:
- Meet with Marketing team: Explain customer segmentation attribute usage
- Update customer dimension documentation with new attribute
- Plan training session for next week

Friday:
- Review monthly metrics for governance council report
- Analyze top 10 customer merge requests (identify patterns)
- Research industry best practices for B2B customer hierarchies
```

---

#### 4. Domain Subject Matter Experts (SMEs)

**Role**: Provide deep expertise for specific source systems or business processes.

**Responsibilities**:
- Participate in data profiling and attribute mapping
- Validate business rules and transformations
- Assist steward with complex issues
- Represent departmental needs in governance decisions

**Example**:
- CRM administrator (customer source system SME)
- Sales operations manager (sales customer definitions)
- Customer service manager (service customer attributes)

**Time Commitment**: 5-10 hours/month, as-needed for issues

---

#### 5. IT Data Management Team

**Role**: Technical implementation of governance decisions.

**Responsibilities**:
- Build and maintain MDM ETL processes
- Implement matching algorithms and survivorship rules
- Develop data quality monitoring and alerting
- Provide technical feasibility input to governance decisions
- Create steward tools and interfaces
- Maintain MDM documentation

**Key Positions**:
- MDM Architect: Overall technical design
- ETL Developers: Build data integration processes
- Data Quality Analysts: Profile data, design quality screens
- Database Administrators: Manage MDM data stores

**Partnership with Stewards**: IT implements what governance defines.

---

### Governance Artifacts

#### 1. Data Governance Charter

**Purpose**: Foundational document establishing governance authority and framework.

**Contents**:

```
ENTERPRISE DATA GOVERNANCE CHARTER

Effective Date: January 1, 2024
Version: 1.0

1. PURPOSE AND SCOPE
   This charter establishes the data governance framework for master data
   management across the enterprise, covering Customer, Product, Supplier,
   Location, and Employee master data domains.

2. GOVERNANCE AUTHORITY
   The Data Governance Council has authority to:
   - Approve enterprise data standards and policies
   - Resolve cross-functional data definition conflicts
   - Enforce data quality standards
   - Authorize changes to conformed dimensions
   - Allocate funding for data governance initiatives

3. GOVERNANCE STRUCTURE
   Executive Sponsor: Chief Data Officer
   Governance Council: [List members and roles]
   Data Stewards: [List by domain]
   Meeting Frequency: Monthly council, weekly steward syncs

4. DECISION-MAKING PROCESS
   - Routine decisions: Steward authority
   - Cross-functional impacts: Governance council approval required
   - Major policy changes: Executive sponsor approval required
   - Escalation path: Steward → Council → Sponsor → C-level

5. DATA OWNERSHIP MODEL
   - Business units own the data they create
   - Data stewards own the conformed definitions
   - IT owns the technical implementation
   - All share accountability for data quality

6. ENFORCEMENT
   - Data quality metrics published monthly
   - Non-compliance escalated to business unit leaders
   - Persistent non-compliance escalated to executive sponsor
   - Extreme non-compliance: System access restrictions

7. CHARTER REVIEW
   This charter will be reviewed annually and updated as needed.

Approved:
_________________ (Chief Data Officer, Executive Sponsor)
Date: January 1, 2024
```

---

#### 2. Conformed Dimension Specification

**Purpose**: Detailed definition of each conformed dimension's structure, content, and business rules.

**Template**:

```
CONFORMED DIMENSION SPECIFICATION: CUSTOMER

Version: 2.1
Last Updated: March 15, 2024
Data Steward: Jane Doe

1. BUSINESS PURPOSE
   The Customer dimension provides a single, integrated view of all individuals
   and organizations that interact with the company as prospects, buyers,
   service recipients, or partners. This dimension enables analysis of customer
   behavior, segmentation, and profitability across all customer-facing
   business processes.

2. GRAIN
   One row per unique customer (individual or organization), with Type 2 slowly
   changing dimension tracking for selected attributes.

3. SCOPE
   - Individual consumers (B2C)
   - Commercial customers (B2B)
   - Government/non-profit customers
   - Excludes: Internal employees (see Employee dimension)
   - Excludes: Suppliers (see Supplier dimension)

4. SOURCE SYSTEMS
   Primary Sources:
   - CRM (Salesforce): Prospect and customer master
   - Order Management (Oracle): Transaction customer data
   - Customer Service (Zendesk): Service interaction customers
   - Billing (SAP): Billing account customers

   External Sources:
   - Dun & Bradstreet: Commercial firmographic data
   - Experian: Consumer demographic data

5. AUTHORITATIVE SOURCES BY ATTRIBUTE
   Customer Name:          CRM (customer-verified)
   Customer Type:          CRM (official classification)
   Billing Address:        Billing System (financial system of record)
   Shipping Address:       Order Management (most current)
   Phone:                  Customer Service (customer-verified)
   Email:                  Marketing (validated via campaigns)
   Credit Score:           Finance System (credit bureau feed)
   Account Status:         CRM (lifecycle management)
   Customer Segment:       Marketing (segmentation model output)
   Geographic Attributes:  External (census geocoding)

6. ATTRIBUTE DEFINITIONS

   | Attribute Name | Data Type | Length | Nullable | Definition |
   |----------------|-----------|--------|----------|------------|
   | Customer_Key | INTEGER | - | No | Surrogate primary key (auto-generated) |
   | Customer_ID | VARCHAR | 20 | No | Durable natural key (format: CUST-######) |
   | Customer_Name | VARCHAR | 100 | No | Full name (parsed from source) |
   | Customer_Type | VARCHAR | 20 | No | Individual, Commercial, Government (domain values) |
   | Customer_Status | VARCHAR | 20 | No | Prospect, Active, Inactive, Closed (domain values) |
   | Customer_Segment | VARCHAR | 30 | Yes | Premium, Standard, Basic, Unassigned (domain values) |
   | ... | ... | ... | ... | ... |

7. BUSINESS RULES
   - Customer_Type: Must be one of {Individual, Commercial, Government}
   - Customer_Status: Required for all customers
   - Customer_Name: No special characters except hyphen, apostrophe, period
   - Email: Must pass RFC 5322 validation if not NULL
   - Phone: US/Canada format: ###-###-#### (10 digits)
   - Billing_Address: Must validate against USPS postal database

8. SLOWLY CHANGING DIMENSION STRATEGY
   Type 1 (Overwrite, no history):
   - Phone
   - Email
   - Billing_Address
   - Shipping_Address

   Type 2 (Track history, new row):
   - Customer_Type
   - Customer_Status
   - Customer_Segment
   - Geographic attributes (City, State, Country)

   Type 0 (Never change):
   - Customer_ID (durable key)
   - Customer_Since_Date

9. SURVIVORSHIP RULES
   See detailed survivorship matrix in Appendix A.

   Summary:
   - Name: CRM priority, then most complete value
   - Contact info: Most recent non-NULL
   - Segmentation: Marketing system (authoritative)
   - Demographics: External data provider (most comprehensive)

10. DATA QUALITY STANDARDS
    Completeness Targets:
    - Customer_Name: 100%
    - Customer_Type: 100%
    - Email: 80% (not all customers provide)
    - Phone: 70% (not all customers provide)

    Validity Targets:
    - Email format validation: 100% of non-NULL
    - Phone format validation: 100% of non-NULL
    - Address postal validation: 95% (some international addresses unvalidatable)

    Uniqueness Targets:
    - Customer duplicate rate: < 3%
    - Cross-system match rate: > 85%

11. CONFORMANCE SCHEDULE
    Phase 1 (Complete): Customer_Type, Customer_Status, Geographic attributes
    Phase 2 (Complete): Customer_Segment, Demographic attributes
    Phase 3 (In Progress): Firmographic attributes (commercial customers)
    Phase 4 (Planned Q3): Customer hierarchy (B2B parent/child relationships)

12. USAGE GUIDANCE
    Dimensional Models Using This Dimension:
    - Sales Transactions
    - Customer Service Interactions
    - Marketing Campaigns
    - Product Returns
    - Website Visits
    - Customer Profitability

    Common Query Patterns:
    - Sales by Customer Segment
    - Customer acquisition by Source Channel
    - Customer retention by Status changes (Type 2 tracking)
    - Geographic analysis by State/Metro Area

13. CHANGE MANAGEMENT
    Changes to this specification require:
    - Minor changes (add optional attribute): Data Steward approval
    - Moderate changes (modify business rule): Governance Council approval
    - Major changes (change grain, remove attribute): Executive Sponsor approval

    Change notification:
    - Email to all BI developers and business analysts
    - Update BI metadata and documentation
    - 2-week notice before production deployment

14. CONTACT
    Data Steward: Jane Doe (jane.doe@company.com)
    Backup Steward: John Smith (john.smith@company.com)
    IT Lead: Mary Johnson (mary.johnson@company.com)

APPENDICES
A. Detailed Survivorship Matrix
B. Attribute Data Lineage Map
C. Source System Mapping Details
D. Data Quality Metrics Dashboard
```

---

#### 3. Survivorship Matrix

**Purpose**: Document exactly which source system value survives for each attribute under various conditions.

**Example Matrix**:

```
CUSTOMER DIMENSION SURVIVORSHIP MATRIX

Attribute: Customer_Name

Priority Rules:
1. If CRM value is non-NULL → Use CRM
2. Else if Sales value is non-NULL → Use Sales
3. Else if Service value is non-NULL → Use Service
4. Else use Billing value
5. If all NULL → Use "Unknown Customer" + flag for steward review

Special Cases:
- If CRM = NULL but Sales has "Cash Customer" or "Guest" → Use Billing if available
- If name contains numbers or special characters → Flag for steward review, use most complete alpha-only value temporarily

Examples:
| CRM | Sales | Service | Billing | Survived Value | Reason |
|-----|-------|---------|---------|----------------|--------|
| John Smith | J. Smith | NULL | Smith, John | John Smith | CRM priority |
| NULL | Jane Doe | Jane D. | J. Doe | Jane Doe | CRM NULL, use Sales (most complete) |
| NULL | Cash Customer | NULL | Mary Johnson | Mary Johnson | Sales="Cash Customer" → use Billing |
| John123 | John Smith | NULL | NULL | John Smith | CRM has invalid chars, use Sales |

---

Attribute: Billing_Address

Priority Rules:
1. Use Billing System (authoritative for financial records)
2. If Billing NULL or invalid → Use Order Management
3. Validate against postal database, flag if validation fails

Special Cases:
- If customer has multiple billing addresses in Billing System → Use most recent
- International addresses: Validation rules vary by country

---

Attribute: Email

Priority Rules:
1. If Marketing value is validated (email_validated_flag=Y) → Use Marketing
2. Else if CRM value passes format validation → Use CRM
3. Else if Sales value passes format validation → Use Sales
4. Else NULL (acceptable for customers without email)

Special Cases:
- Multiple emails for same customer → Choose validated email over non-validated
- Bounced emails: NULL out email if bounce rate > 3 consecutive sends

---

Attribute: Customer_Segment

Priority Rules:
1. Always use Marketing system (runs segmentation model)
2. If Marketing NULL → Use "Unassigned" temporarily, trigger Marketing segmentation

Special Cases:
- Marketing updates segment monthly, not daily
- Segment changes trigger Type 2 SCD (history tracking)

---

[Continue for all 50+ customer dimension attributes...]
```

---

#### 4. Data Quality Scorecard

**Purpose**: Monitor ongoing data quality and hold stakeholders accountable.

**Example Scorecard**:

```
CUSTOMER MASTER DATA QUALITY SCORECARD
Report Date: March 31, 2024
Data Steward: Jane Doe

1. COMPLETENESS METRICS

| Attribute | Target | Actual | Status | Trend | Owner |
|-----------|--------|--------|--------|-------|-------|
| Customer_Name | 100% | 99.8% | ✓ Green | ↑ | CRM Team |
| Customer_Type | 100% | 100% | ✓ Green | → | CRM Team |
| Email | 80% | 76.2% | ✗ Red | ↓ | Marketing |
| Phone | 70% | 68.5% | ⚠ Yellow | ↓ | Service |
| Billing_Address | 95% | 97.1% | ✓ Green | ↑ | Finance |

Actions:
- Email: Marketing to launch email collection campaign (target: 80% by end of Q2)
- Phone: Service to add phone collection to support interaction script

---

2. VALIDITY METRICS

| Validation Rule | Target | Actual | Status | Violations | Owner |
|-----------------|--------|--------|--------|------------|-------|
| Email Format (RFC 5322) | 100% | 98.5% | ⚠ Yellow | 1,245 | Marketing |
| Phone Format (###-###-####) | 100% | 99.2% | ✓ Green | 680 | Service |
| Address Postal Validation | 95% | 94.1% | ⚠ Yellow | 4,890 | Finance |

Actions:
- Email: ETL cleansing improved from 96% last month → on track to 100%
- Address: Mostly international addresses (no USPS validation) → acceptable

---

3. UNIQUENESS METRICS

| Metric | Target | Actual | Status | Count | Trend |
|--------|--------|--------|--------|-------|-------|
| Customer Duplicate Rate | <3% | 2.1% | ✓ Green | 1,680 dupes | ↓ |
| Cross-System Match Rate | >85% | 87.3% | ✓ Green | - | ↑ |

Duplicate Analysis:
- 1,680 duplicate customers identified (of 80,000 total)
- 1,200 auto-merged based on high confidence match (>90%)
- 480 flagged for steward review (70-90% match confidence)

Actions:
- Continue monthly duplicate review sessions
- Tune matching thresholds (reduced false positives by 15% this month)

---

4. TIMELINESS METRICS

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| ETL SLA (completion by 6 AM) | 100% | 96.7% | ⚠ Yellow |
| Data Latency (source to DW) | <24 hrs | 18 hrs avg | ✓ Green |

Late ETL Runs: 1 day this month (March 15 - source system outage)

---

5. SOURCE SYSTEM CONTRIBUTIONS

| Source System | Records | % of Total | Quality Score | Owner |
|---------------|---------|------------|---------------|-------|
| CRM | 65,000 | 81% | 94/100 | Sales Ops |
| Order Mgmt | 72,000 | 90% | 89/100 | Supply Chain |
| Service | 38,000 | 47% | 91/100 | Service Ops |
| Billing | 80,000 | 100% | 96/100 | Finance |

Note: % of Total > 100% due to customer overlap across systems (expected)

Quality Score Formula:
(Completeness × 0.4) + (Validity × 0.4) + (Timeliness × 0.2)

---

6. ISSUES AND ESCALATIONS

| Issue ID | Description | Severity | Opened | Owner | Status |
|----------|-------------|----------|--------|-------|--------|
| CUST-451 | CRM not capturing customer type for 200 new customers | High | 3/20 | CRM Team | In Progress |
| CUST-449 | Order system using generic "Guest Customer" instead of capturing name | Medium | 3/10 | Order Mgmt | Resolved |
| CUST-438 | External demographic provider changed schema | Low | 2/28 | IT MDM | Resolved |

---

7. GOVERNANCE ACTIONS

This Month:
- Approved: Add "Customer_Preferred_Contact_Method" attribute (email, phone, mail, none)
- Approved: Change survivorship rule for customer segment (Marketing now authoritative)
- Escalated: Sales refusing to adopt conformed Customer_Type values → Council to address

---

Next Month Focus:
- Achieve 80% email completeness (Marketing campaign)
- Complete medium-confidence duplicate review backlog (480 records)
- Implement new customer hierarchy attributes for B2B customers

---

Scorecard Reviewed: Data Governance Council Meeting, April 5, 2024
```

---

### Governance Processes

#### 1. Attribute Definition and Approval Process

**When**: Adding new attribute to conformed dimension

**Process**:

```
Step 1: Business Request
- Source: Business stakeholder or data steward
- Document: Business case (why needed, expected usage, impact)

Step 2: Steward Analysis
- Assess: Attribute definition, domain values, data type, source system
- Determine: Authoritative source, survivorship rule
- Estimate: Implementation effort, affected systems

Step 3: SME Review
- Validate: Definition accuracy, completeness
- Confirm: Source system availability and quality

Step 4: Approval
- Minor (optional attribute, low impact): Steward approves
- Moderate (affects multiple systems): Governance Council approves
- Major (changes grain or core attributes): Executive Sponsor approves

Step 5: Implementation
- IT: Update conformed dimension specification
- IT: Implement ETL changes
- Steward: Update documentation and training
- Steward: Communicate to stakeholders

Step 6: Validation
- IT: Verify data populated correctly
- Steward: Sample validation of data quality
- Business: Confirm availability in BI tools

Typical Timeline:
- Minor: 2 weeks
- Moderate: 4-6 weeks
- Major: 8-12 weeks
```

---

#### 2. Data Quality Issue Resolution Process

**When**: Data quality metric falls below threshold or issue reported

**Process**:

```
Step 1: Issue Detection
- Source: Automated quality monitoring, steward review, or user report
- Log: Create issue ticket with details (metric, threshold, actual value, impact)

Step 2: Triage
- Steward: Assess severity (Critical, High, Medium, Low)
  - Critical: Halts business operations or reporting
  - High: Impacts key metrics or large user population
  - Medium: Affects subset of users or non-critical metrics
  - Low: Cosmetic or minimal impact
- Steward: Assign owner based on source system or root cause

Step 3: Root Cause Analysis
- Owner: Investigate underlying cause
  - Source system bug or process change?
  - ETL transformation error?
  - Business rule change needed?
  - External data provider issue?

Step 4: Resolution Plan
- Owner: Propose fix with timeline
- Steward: Review and approve plan
- Escalate to Governance Council if:
  - Requires business rule change
  - Cross-functional coordination needed
  - Budget/resources required

Step 5: Implementation
- Owner: Execute fix
- IT: Deploy ETL changes if needed
- Steward: Validate fix effectiveness

Step 6: Closure and Prevention
- Steward: Verify issue resolved and metric back to target
- Owner: Document root cause and fix
- Steward: Update quality screens or monitoring to prevent recurrence

SLA Targets:
- Critical: 4 hours response, 24 hours resolution
- High: 1 business day response, 1 week resolution
- Medium: 3 business days response, 2 weeks resolution
- Low: 1 week response, 1 month resolution
```

---

#### 3. Customer Merge/Split Review Process

**When**: Manual review needed for medium-confidence matches or split requests

**Steward Review Session** (Weekly):

```
Attendees:
- Data Steward (Customer)
- Customer SMEs (CRM, Sales, Service)
- Optional: IT Data Quality Analyst

Agenda:
1. Review Medium-Confidence Matches
   - System presents: 2 customer records, match score (70-89%), matching attributes
   - Steward decision: Merge, Don't Merge, or Need More Info
   - If Merge: Approve survivorship, document reason
   - If Don't Merge: Flag as non-match (inform ML model), document reason
   - If Need More Info: Assign SME research, defer to next session

2. Review Merge Requests from Business Users
   - Business user submitted: "These 2 customers are the same person"
   - Steward validates: Match confidence, attribute comparison
   - Approve or reject with justification

3. Review Split Requests from Business Users
   - Business user submitted: "This customer record is actually 2 different people"
   - Steward validates: Evidence of distinct entities
   - If approved: Define split logic (how to allocate attributes and history)
   - Execute split, notify requestor

Typical Session:
- Duration: 1-2 hours
- Volume: 50-100 medium-confidence matches reviewed
- Outcome: 70% merged, 25% don't merge, 5% need more info

Tools:
- Steward UI: Side-by-side customer attribute comparison
- History: Transaction and interaction history for each customer
- Match Explanation: Which attributes contributed to match score
- Actions: Approve Merge, Reject Match, Request More Info, Split Customer
```

---

## Governance Maturity Model

Organizations progress through governance maturity stages:

### Level 1: Ad Hoc (No Governance)

**Characteristics**:
- No formal governance structure
- IT makes technical decisions without business input
- Inconsistent data definitions across departments
- No accountability for data quality
- Reactive issue resolution

**Typical Problems**:
- Multiple versions of truth
- Low business user trust in data
- Significant manual reconciliation
- Failed BI initiatives

**How to Progress**: Appoint executive sponsor and data steward for one domain (Customer), establish basic governance charter.

---

### Level 2: Defined (Initial Governance)

**Characteristics**:
- Governance charter and council established
- Data stewards appointed for key domains
- Basic policies and standards documented
- Reactive quality monitoring

**Typical Progress**:
- One or two domains under governance (Customer, Product)
- Monthly governance council meetings
- Basic data quality scorecards
- Conformed dimensions for critical attributes

**How to Progress**: Expand to more domains, implement proactive quality monitoring, establish routine steward processes.

---

### Level 3: Managed (Operational Governance)

**Characteristics**:
- Comprehensive governance across all master data domains
- Routine steward operations (weekly reviews, issue resolution)
- Proactive data quality monitoring with automated alerts
- Business rules enforced at source systems
- Regular governance council meetings with consistent attendance

**Typical Capabilities**:
- 5+ master data domains under governance
- Data quality scorecards with trending analysis
- Steward tools and workflows operational
- Business user training and documentation
- Governance integrated into project delivery

**How to Progress**: Optimize processes, implement predictive quality analytics, expand to transactional data governance.

---

### Level 4: Optimized (Strategic Governance)

**Characteristics**:
- Data governance embedded in organizational culture
- Continuous improvement based on metrics
- Predictive data quality (identify issues before impact)
- Strong business ownership and accountability
- Governance enables competitive advantage

**Typical Capabilities**:
- Master and transactional data governed
- AI/ML for quality prediction and anomaly detection
- Self-service data certification
- Data-driven culture with executive accountability
- Governance ROI measured and communicated

---

## Common Governance Challenges and Solutions

### Challenge 1: Business Resistance to Standardization

**Symptom**: "We've always done it this way, why change?"

**Root Cause**:
- Loss of departmental control
- Fear of unfamiliar definitions
- Political power dynamics

**Solution**:
- Lead with business value: "What questions can't you answer today?"
- Start with least-common-denominator: Conform only critical attributes
- Iterative approach: Phase 1 minimal, expand over time
- Executive sponsorship: Senior leader communicates strategic importance
- Include resistors: Give skeptics a seat at the governance table

---

### Challenge 2: IT-Driven Governance Without Business Buy-In

**Symptom**: Governance policies ignored, business users work around systems.

**Root Cause**:
- IT designed governance without business input
- Policies seen as technical constraints, not business enablers

**Solution**:
- Restart with business leadership: Business chair governance council
- Business stewards, not IT: Steward role must be business, not technical
- Translate to business value: Show how governance solves business problems
- Engage stakeholders early: Include business in design, not just review

---

### Challenge 3: Steward Lack of Authority

**Symptom**: Steward recommendations ignored by business units.

**Root Cause**:
- Steward lacks organizational gravitas or executive backing
- No consequences for non-compliance

**Solution**:
- Select senior, respected stewards: Not junior analysts
- Executive sponsor backing: Sponsor publicly supports steward decisions
- Escalation path: Council resolves disputes steward cannot
- Tie to performance: Data quality goals in business unit scorecards

---

### Challenge 4: Governance Bureaucracy Slows Delivery

**Symptom**: "We can't deliver because we're waiting for governance approval."

**Root Cause**:
- Overly complex approval processes
- Infrequent governance meetings
- Unclear decision authority

**Solution**:
- Tiered authority: Steward approves routine, council approves major
- Frequent decision cycles: Weekly steward, monthly council (not quarterly)
- Defined SLAs: Approval within 5 business days for routine requests
- Exception process: Fast-track for critical business needs

---

### Challenge 5: Lack of Executive Support

**Symptom**: Governance council canceled due to executive no-shows.

**Root Cause**:
- Executives don't see governance value
- Governance seen as IT initiative, not business priority

**Solution**:
- Demonstrate business impact: Show metrics (duplicate customer cost, bad data rework)
- Strategic framing: Position governance as enabler of strategic initiatives (customer 360, personalization)
- Executive peer pressure: CIO + CFO + CMO jointly sponsor (not just IT)
- Board-level visibility: Report governance metrics to board of directors

---

## Summary

Effective data governance is the foundation of successful MDM. It requires business-led stewardship with executive sponsorship, clear roles and responsibilities, documented policies and standards, and routine processes for defining conformed attributes, resolving quality issues, and ensuring accountability. Organizations must mature through governance levels, starting with one domain and expanding over time. The key success factor is recognizing that governance is an organizational change initiative, not merely a technical implementation.
