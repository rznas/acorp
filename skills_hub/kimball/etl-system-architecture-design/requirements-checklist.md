# ETL Requirements Roundup Checklist

Before designing the ETL architecture, complete this comprehensive requirements assessment across all 10 critical areas.

---

## 1. Business Needs

**Definition**: Information content that business users need to make informed decisions

**Checklist**:
- [ ] List all key performance indicators (KPIs) the project intends to support
- [ ] Document drill-down targets for investigating "why?" KPIs changed
- [ ] Document drill-across requirements for integrated analysis
- [ ] Identify analytic requirements that drive dimensional model design
- [ ] Map KPIs to source systems and data elements

**Deliverable**: KPI inventory with drill-down/drill-across specifications

---

## 2. Compliance

**Definition**: Legal and reporting requirements for data accuracy, completeness, and tamper-proof evidence

**Checklist**:
- [ ] Consult legal department/chief compliance officer
- [ ] List all data and final reports subject to compliance restrictions
- [ ] Identify data inputs and transformation steps requiring chain of custody
- [ ] List data requiring proof of security (offline and online copies)
- [ ] Document data that must be archived with retention policies
- [ ] Determine usable lifetime of archives (5, 10, 50 years?)
- [ ] Identify compliance requirements overlapping with security

**Deliverable**: Compliance requirements matrix with chain of custody specifications

---

## 3. Data Quality

**Definition**: Measures and processes to ensure data accuracy and reliability

**Checklist**:
- [ ] List data elements with known unacceptable quality
- [ ] Document agreements with source systems to correct data before extraction
- [ ] List data elements discovered during profiling requiring continuous monitoring
- [ ] Identify data elements to be continuously flagged as part of ETL process
- [ ] Define quality metrics and thresholds
- [ ] Establish error tolerance levels per data element

**Deliverable**: Data quality assessment with monitoring specifications

**Note**: Three converging forces drive data quality to top priority:
1. Knowledge workers instinctively believe data is crucial to function
2. Distributed data sources require effective integration
3. Compliance demands mean careless data handling won't be excused

---

## 4. Security

**Definition**: Protection of data from unauthorized access, both online and in backups

**Checklist**:
- [ ] Expand compliance checklist to encompass security and privacy requirements
- [ ] Identify data elements with extra security sensitivity
- [ ] Define access restrictions (need-to-know basis vs. wide publishing)
- [ ] Establish physical backup security requirements
- [ ] Document security requirements for media removed from backup vault
- [ ] Invite experienced security manager to join design team (if needed)

**Deliverable**: Security and privacy requirements document

**Note**: Data warehouse goal (publish widely) conflicts with security mentality (restrict access)—seek clear guidance from senior management

---

## 5. Data Integration

**Definition**: Making separated databases work together through conformed dimensions and facts

**Checklist**:
- [ ] Use bus matrix to generate priority list for conformed dimensions (columns)
- [ ] Annotate each row (business process) with executive demand for integration
- [ ] Confirm ETL team agreement on conforming each business process
- [ ] Identify conformed dimension ownership and governance
- [ ] Document agreements on common business metrics (KPIs)
- [ ] Plan for conforming dimensions and conforming facts

**Deliverable**: Annotated bus matrix with conformance priorities and ownership

**Key Concepts**:
- **Conformed Dimensions**: Common dimensional attributes across separated databases enabling drill-across
- **Conformed Facts**: Agreements on common metrics (KPIs) enabling mathematical comparison

---

## 6. Data Latency

**Definition**: How quickly source system data must be delivered to business users

**Checklist**:
- [ ] List all business demands for daily data delivery
- [ ] List demands for intra-day delivery (many times per day)
- [ ] List demands for near-real-time delivery (within seconds)
- [ ] List demands for instantaneous delivery
- [ ] Annotate each with business understanding of data quality trade-offs
- [ ] Identify which demands have been well-vetted vs. aspirational
- [ ] Determine if micro-batch or streaming architecture required

**Deliverable**: Data latency requirements matrix with quality trade-off acknowledgments

**Architecture Impact**:
- **Daily**: Batch-oriented architecture (standard)
- **Intra-day**: Micro-batch processing, message queues, reduced quality screens
- **Instantaneous**: Major paradigm shift—EII solution, no caching, limited complexity

---

## 7. Archiving and Lineage

**Definition**: Long-term data retention and tracking of data transformations

**Checklist**:
- [ ] List data sources to be archived with retention policies
- [ ] List intermediate data steps to be archived
- [ ] Document compliance, security, and privacy constraints on archives
- [ ] Plan staging after each major ETL pipeline activity (extract, clean/conform, deliver)
- [ ] Decide which staged data becomes permanent archive
- [ ] Document lineage metadata (origins and processing steps) for each archived dataset
- [ ] Establish archive media strategy (disk, tape, cloud)
- [ ] Plan for archive refresh and migration over time

**Deliverable**: Archive and lineage strategy document

**Conservative Recommendation**: Archive all staged data unless conscious decision made that specific datasets will never be recovered

**Lineage**: Each archived dataset must have accompanying metadata describing origins and processing steps

---

## 8. BI Delivery Interfaces

**Definition**: Data handoff structure and format for BI applications

**Checklist**:
- [ ] List all fact and dimension tables directly exposed to BI tools (from dimensional model)
- [ ] List all OLAP cubes required
- [ ] List special database structures required by BI tools
- [ ] List all known indexes agreed upon for BI performance
- [ ] List all aggregations for BI performance support
- [ ] Work with BI developers to determine exact handoff requirements
- [ ] Identify BI tool sensitivities to avoid
- [ ] Identify BI tool features to exploit based on physical data format

**Deliverable**: BI interface specifications with performance support plan

**Critical Principle**: ETL/modeling team must take responsibility for content and structure making BI applications simple and fast. Never hand off normalized physical model and walk away.

---

## 9. Available Skills

**Definition**: In-house technical capabilities to build and manage ETL system

**Checklist**:
- [ ] Inventory operating system skills
- [ ] Inventory ETL tool skills
- [ ] Inventory scripting language skills (Perl, Python, etc.)
- [ ] Inventory programming language skills (C++, Java, C#, etc.)
- [ ] Inventory SQL skills
- [ ] Inventory DBMS skills (Oracle, SQL Server, PostgreSQL, etc.)
- [ ] Inventory OLAP skills
- [ ] Document skill exposure/shortage risks
- [ ] List skills required for current systems
- [ ] List skills required for likely future systems
- [ ] Plan for skill acquisition or training

**Deliverable**: Skills inventory with gap analysis

**Critical Decision**: Don't build ETL system dependent on skills not in-house or reasonably acquirable (e.g., critical C++ modules without C++ expertise)

---

## 10. Legacy Licenses

**Definition**: Existing software licenses that may mandate or constrain ETL design

**Checklist**:
- [ ] List legacy operating system licenses
- [ ] List legacy ETL tool licenses
- [ ] List legacy scripting language licenses
- [ ] List legacy programming language licenses
- [ ] List legacy SQL licenses
- [ ] List legacy DBMS licenses
- [ ] List legacy OLAP licenses
- [ ] Document whether exclusive use is mandated or merely recommended for each
- [ ] Identify problematic legacy license constraints
- [ ] Prepare business case if challenging legacy license use

**Deliverable**: Legacy license inventory with mandate/recommendation status

**Warning**: Sometimes senior management mandates legacy license use that may be a mistake. Be prepared to make the case if you feel strongly, but be willing to accept final decision.

---

## Requirements Roundup Summary Template

| Area | Status | Owner | Critical Issues | Resolution Date |
|------|--------|-------|----------------|----------------|
| Business Needs | | | | |
| Compliance | | | | |
| Data Quality | | | | |
| Security | | | | |
| Data Integration | | | | |
| Data Latency | | | | |
| Archiving/Lineage | | | | |
| BI Interfaces | | | | |
| Available Skills | | | | |
| Legacy Licenses | | | | |

**Instructions**: Provide short response for each area. Any area can be a showstopper—visit all topics before launching ETL design and development.

---

## Next Steps After Requirements Roundup

1. **Review with stakeholders**: Present requirements summary to business sponsors and senior management
2. **Resolve showstoppers**: Address any critical issues that could derail project
3. **Prioritize**: Identify which requirements are must-have vs. nice-to-have
4. **Set expectations**: Use requirements to establish realistic development schedules
5. **Begin architecture design**: Use requirements to inform 34 subsystem selection and design
6. **Maintain living document**: Update requirements as project evolves and new information discovered

This comprehensive requirements roundup prevents surprises later and ensures ETL architecture addresses all known constraints and needs.
