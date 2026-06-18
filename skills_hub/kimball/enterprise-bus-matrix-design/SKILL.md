---
name: enterprise-bus-matrix-design
description: Design the enterprise data warehouse bus architecture and matrix that ensures data integration through conformed dimensions across business processes. Use after requirements gathering and before detailed dimensional modeling begins to create the overall DW/BI architectural framework.
---

# Enterprise Data Warehouse Bus Matrix Design

## When to Use This Skill

Use this skill during the **Project Planning & Architecture** phase of the DW/BI lifecycle, specifically:
- **After** business requirements gathering is complete
- **Before** detailed dimensional modeling of individual business processes begins
- When planning the overall enterprise DW/BI architecture
- When you need to ensure integration across multiple business process projects
- When establishing data governance for conformed dimensions

## Overview

The enterprise data warehouse bus architecture provides a rational approach to decomposing the enterprise DW/BI planning task into manageable, bite-sized chunks while maintaining architectural consistency. The bus matrix is the primary artifact that communicates this architecture.

**Critical Success Principle**: Build the enterprise incrementally, one business process at a time, but within a coherent architectural framework that ensures all pieces fit together through conformed dimensions.

## Understanding the Bus Architecture

### What Is the Bus Architecture?

The word "bus" comes from the electrical power industry (not "business"). A bus is a common structure to which everything connects and from which everything derives power. Like a computer's bus standard that enables peripheral devices to plug in and work together:

- **The bus architecture defines a standard interface for the DW/BI environment**
- Separate dimensional models can be implemented by different teams at different times
- Business process subject areas "plug together" and usefully coexist if they adhere to the standard
- The standard is based on **conformed dimensions** and **conformed facts**

### Key Architectural Principles

1. **Process-Centric Design**: Focus on business processes, not departments or reports
2. **Atomic Grain**: Start with the most detailed level of data capture
3. **Conformed Dimensions**: Share standardized dimensions across processes
4. **Incremental Build**: Implement one business process at a time
5. **Integration Through Conformance**: Enable drill-across analysis via shared dimensions

## Creating the Enterprise Bus Matrix

### Step 1: Identify Business Processes (Matrix Rows)

Business processes are the operational activities performed by your organization. Each process:
- Represents a primary activity or event that generates measurable results
- Is recognizable by its operational source system
- Becomes a row in the bus matrix
- Ultimately translates into one or more dimensional models

**Examples of Business Processes** (NOT Departments):
- Retail: POS Retail Sales, Store Inventory, Product Shipments, Promotions
- Order Management: Order Transactions, Invoice Transactions, Order Fulfillment Pipeline
- Finance: General Ledger Postings, Budget Allocations, Account Balances
- Telecom: Call Detail Records, Customer Billing, Network Performance

**Common Mistakes to Avoid**:
- ❌ **Departmental rows**: Don't list organizational boxes (Sales Dept, Marketing Dept)
- ❌ **Overly encompassing rows**: Don't create rows like "All Finance Data"
- ❌ **Report-centric rows**: Don't list specific reports (Monthly Sales Report)
- ❌ **Too narrowly defined**: Don't create rows for every minor variation

**How to Identify Processes**:
```xml
<process_identification>
1. Review the organizational value chain
2. Identify operational source systems
3. Ask: "What measurable events does this system capture?"
4. Look for natural operational activities with clear metrics
5. Validate with business: "Is this a distinct operational activity?"
</process_identification>
```

### Step 2: Identify Common Dimensions (Matrix Columns)

Dimensions are the "who, what, where, when, why, and how" descriptors shared across business processes. Each dimension:
- Must be defined at its most granular level
- Should have significance across multiple processes
- Becomes a column in the bus matrix
- Will be built once and reused everywhere

**Examples of Common Dimensions**:
- Date/Time
- Product
- Customer
- Store/Location
- Employee
- Promotion
- Vendor/Supplier
- Account

**Common Mistakes to Avoid**:
- ❌ **Overly generalized columns**: Don't create generic "Person" or "Location" for unrelated populations
- ❌ **Separate columns per hierarchy level**: Use one "Date" column, not "Day", "Month", "Quarter", "Year"
- ❌ **One attribute per column**: Don't treat every dimension attribute as a separate column

**Dimension Granularity Rules**:
- Define dimensions at their most atomic level (e.g., individual product, not product category)
- Use a single column for the dimension family
- Document rollup levels within matrix cells when needed

### Step 3: Populate the Matrix

Mark the intersections where dimensions relate to business processes:

1. For each business process row, identify which dimensions are relevant
2. Mark the cell with an "X" or shading
3. When using shrunken dimensions, annotate the grain (e.g., "Month" for monthly snapshots)

**Example Bus Matrix Structure**:
```
                        Date  Product  Customer  Store  Promotion  Vendor
POS Retail Sales         X       X        X        X       X
Store Inventory       Month     X                 X
Purchase Orders          X       X                         X        X
Shipments Received       X       X                         X        X
```

### Step 4: Document Detailed Matrix

For complex environments, create a **detailed implementation bus matrix** that includes:
- Process name and description
- Grain statement for each process
- Primary facts/metrics for each process
- Dimension granularity when it differs from atomic

### Step 5: Create Opportunity/Stakeholder Matrix

This variation helps with project planning and governance:

**Structure**:
- **Rows**: Same business processes
- **Columns**: Organizational functions (Merchandising, Marketing, Finance, Operations, etc.)
- **Cells**: Mark which functions are interested in which processes

**Uses**:
- Identifies cross-functional requirements
- Helps prioritize projects based on breadth of impact
- Identifies stakeholders to involve in requirements, design, and review sessions
- Demonstrates enterprise value beyond single departments

## Conformed Dimensions: The Integration Cornerstone

### What Makes a Dimension Conformed?

Conformed dimensions are shared, standardized dimensions used across multiple fact tables. They conform when they have:

1. **Identical Conformed Dimensions** (most common):
   - Same dimension keys
   - Same attribute column names
   - Same attribute definitions
   - Same attribute values and labels
   - Example: The Date dimension used for Sales is identical to Date for Inventory

2. **Shrunken Rollup Dimensions** (attribute subset):
   - Contains subset of attributes from more granular dimension
   - Used when facts exist at different granularity levels
   - Example: Month dimension is a subset of Day dimension (fewer rows, fewer attributes)

3. **Shrunken Subset Dimensions** (row subset):
   - Same level of detail but limited to subset of rows
   - Example: Corporate Product dimension vs. Line-of-Business Product dimension

### Representing Shrunken Dimensions on the Matrix

Two approaches:

**Approach 1**: Annotate within cells
```
                     Date
Store Inventory     Month    (annotate the grain)
Daily Sales         Day      (or just mark with X for atomic)
```

**Approach 2**: Subdivide columns
```
                   Date          Product
                 Day | Month   Atomic | Brand
Store Inventory      X                   X
Daily Sales       X             X
```

### The Power of Conformed Dimensions

**Drilling Across Fact Tables**:
Conformed dimensions enable you to combine metrics from different business processes in a single report:

```xml
<drill_across_example>
Query 1: SELECT Product_Name, SUM(Sales_Amount)
         FROM Sales_Facts
         JOIN Product_Dimension...

Query 2: SELECT Product_Name, AVG(Inventory_Quantity)
         FROM Inventory_Facts
         JOIN Product_Dimension...

Result: Full outer join on Product_Name to show sales and inventory together
</drill_across_example>
```

This only works if Product_Name values are **identical** across both Product dimensions.

## Data Governance for Conformed Dimensions

### Why Governance Is Critical

- Reaching consensus on dimension attributes is a **business challenge**, not a technical one
- Requires cross-organizational agreement on names, definitions, and values
- Without governance, you'll build incompatible data silos
- IT cannot succeed alone—business leadership must drive conformance

### Data Governance Requirements

**Business-Driven Leadership**:
Identify subject matter experts from the business with:
- Organizational respect and authority
- Broad enterprise knowledge
- Ability to balance departmental vs. enterprise needs
- Political savvy and negotiation skills
- Strong communication abilities

**Governance Objectives**:
1. Agree on common data definitions and labels
2. Establish standard domain values
3. Define data quality standards
4. Assign dimension ownership/stewardship
5. Define policies for handling changes (SCD strategies)
6. Establish security and access controls

**Governance Roles**:
- **Data Stewards**: Own specific conformed dimensions
- **Dimension Managers**: Responsible for dimension ETL processes
- **Fact Providers**: Use conformed dimensions to load fact tables

## Bus Matrix as Communication Tool

### Multiple Uses of the Matrix

1. **Architecture Planning**: Defines overall data architecture framework
2. **Database Design**: Guides dimensional model creation
3. **Data Governance**: Identifies which dimensions need stewardship
4. **Project Estimation**: Helps gauge project scope by counting dimensions
5. **Team Coordination**: Enables parallel development by multiple teams
6. **Organizational Communication**: Succinct visual for IT and business management

### Communicating the Matrix

**Target Audiences**:
- **Development Teams**: Understand which dimensions to build/reuse
- **Project Managers**: Gauge effort based on dimensionality
- **IT Management**: Coordinate across projects, resist departmental silos
- **Business Management**: Understand rollout staging and governance needs
- **Data Governance Teams**: Know which dimensions require cross-functional agreement

**Key Messages**:
1. This is the architectural blueprint for the enterprise DW/BI
2. Each row is a potential project
3. Each column is a dimension that must be built once and shared
4. Shaded cells show integration points
5. Projects can proceed in parallel if they follow the architecture

## Implementation Guidance

### Prioritizing Business Processes

After creating the bus matrix, prioritize which processes to implement first:

**Prioritization Grid** (Impact vs. Feasibility):
```
High Impact  │  [Avoid for now]  │  [START HERE!]
             │                   │
             │────────────────────────────────
Low Impact   │  [Avoid]          │  [Easy but low value]
             └───────────────────────────────
                Low Feasibility    High Feasibility
```

**Factors for "Impact"**:
- Business value and urgency
- Number of stakeholders interested (from opportunity matrix)
- Strategic importance
- Potential for quick wins

**Factors for "Feasibility"**:
- Data availability and quality
- Source system accessibility
- Technical complexity
- Political challenges
- Team skills and experience

**Start with**: High-impact, high-feasibility projects in upper-right quadrant

### Sequencing Projects

1. **First Project**:
   - Choose a single business process row
   - Implement core conformed dimensions
   - Build fact table for that process
   - Deliver BI applications

2. **Second Project**:
   - Choose another business process
   - Reuse existing conformed dimensions
   - Add new conformed dimensions as needed
   - Build new fact table
   - Leverage shared dimensions for drill-across

3. **Subsequent Projects**:
   - Continue building one process at a time
   - Portfolio of conformed dimensions grows
   - Integration increases with each project
   - Development velocity accelerates

### Dealing with Existing Non-Conformed Models

If you have standalone dimensional models built without the bus architecture:

**Assessment Steps**:
1. Conduct honest appraisal of current state
2. Map existing dimensions to proposed conformed standards
3. Identify gap between current and target architecture
4. Estimate effort to retrofit vs. rebuild

**Conversion Options**:
- **Map and Rebuild**: Cross-reference existing dimension to conformed version, rebuild fact table with new keys
- **Shut Down and Rebuild**: If too flawed, retire and rebuild in proper architecture
- **Accept as Non-Conforming**: Only if truly isolated with no integration needs

**Prerequisites for Success**:
- Senior management must understand current data chaos
- Business and IT leadership must support conversion plan
- Adequate resources and funding must be committed
- DW/BI manager needs political air cover from executives

## Conformed Dimensions and Agile Development

### Enabling Agile Through Conformance

**Conformed dimensions enable agile DW/BI**:
- Build dimensions once, reuse across sprints
- New projects leverage existing dimension "shelf"
- Faster time-to-market for new business processes
- Focus new development on fact tables and ETL

**Agile Approach to Conformance**:
- Start with minimal set of conformed attributes
- Even a single conformed attribute (e.g., Product Category) enables some integration
- Iteratively expand dimensions sprint by sprint
- Architectural sprints can focus on dimension design
- Bundle multiple sprint deliverables into releases

**Warning**:
Without conformed dimensions, "agile" development creates inconsistent data silos that cause users to waste time reconciling data rather than making agile decisions.

## Conformed Facts

In addition to conformed dimensions, key facts must also conform across business processes:

**Requirements for Conformed Facts**:
- Same name = Same definition
- Same dimensional context
- Same units of measure
- Same calculation formulas

**Examples**:
- Revenue across Sales, Returns, Forecasts
- Costs across Manufacturing, Procurement, Inventory
- Quantities across Sales, Shipments, Inventory

**If facts cannot conform**:
- Give them different names
- Document the differences clearly
- Prevent business users from incorrectly combining incompatible metrics

**Multiple Units of Measure**:
- Store facts in multiple units to support drill-across
- Example: Store product movement in both "shipping cases" and "retail units"
- Enables seamless reporting across value chain

## Common Pitfalls and How to Avoid Them

| Pitfall | Impact | Solution |
|---------|--------|----------|
| Building isolated dimensional models | Data silos, no integration | Mandate bus architecture adherence |
| IT-driven dimension design | Business rejection | Business leads data governance |
| Skipping bus matrix | Chaotic, unplanned architecture | Create matrix as first deliverable |
| Department-focused rows | Siloed by org chart | Focus on business processes |
| Too many dimensions for first project | Implementation risk | Start with single process row |
| Inconsistent dimension naming | Integration failures | Establish naming standards early |
| Assuming conformance is "too hard" | Permanent data chaos | Engage senior leadership |

## Deliverables Checklist

- [ ] Enterprise Data Warehouse Bus Matrix
- [ ] Opportunity/Stakeholder Matrix
- [ ] Conformed Dimension List with Definitions
- [ ] Data Governance Assignments (dimension stewards)
- [ ] Detailed Implementation Bus Matrix (with grains and facts)
- [ ] Phased Implementation Roadmap
- [ ] Prioritization Grid with Project Sequence

## Next Steps

After completing the bus matrix design:

1. **Establish Data Governance**: Assign stewards for each conformed dimension
2. **Select First Project**: Choose high-impact, high-feasibility business process
3. **Design Dimensional Model**: Apply four-step process to first business process
4. **Build Conformed Dimensions**: Implement shared dimensions first
5. **Develop ETL and BI Apps**: Complete first project delivery
6. **Iterate**: Move to next business process, reusing conformed dimensions

## References

See also:
- **SKILL 1**: Business Requirements Gathering (prerequisite)
- **SKILL 3**: Four-Step Dimensional Modeling Process (next step)
- **SKILL 6**: Conformed Dimension Management (detailed implementation)
- **Chapter 4**: Complete bus architecture discussion
- **Chapter 17**: Lifecycle planning and prioritization
- **Chapter 2**: Conformed dimension techniques
