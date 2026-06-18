---
name: bi-application-design-patterns
description: Design effective BI applications, reports, and dashboards that leverage dimensional models' strengths through drilling down/across, conformed dimensions, and user-centric templates. Use when building reports, dashboards, ad-hoc query tools, or any analytic applications that query the presentation area.
---

# BI Application Design Patterns

## When to Use This Skill

Use this skill during the **BI Application Development** phase of the Kimball DW/BI Lifecycle when:

- Designing reports, dashboards, or analytic applications that query dimensional models
- Building parameter-driven templates for business users
- Creating navigation structures for accessing warehouse data
- Developing drill-down and drill-across capabilities
- Designing ad-hoc query interfaces for power users
- Building OLAP cube navigation and analysis tools
un- Creating information portals or executive dashboards
- Specifying BI tool requirements based on dimensional model capabilities

**Lifecycle Context**: This skill applies after dimensional models and ETL systems are designed, typically concurrent with or following physical database deployment.

## Core BI Application Design Principles

### 1. Leverage the Dimensional Model's Strengths

The dimensional model is purposely designed to support intuitive, high-performance analytics:

<design_principle>
**Simplicity and Symmetry**: Dimensional models' star schema structure with fact tables surrounded by dimension tables makes them immediately recognizable to business users. The BI application should preserve this simplicity, not obscure it.

**Predictable Framework**: All dimensions are symmetrically equal entry points. The model has no built-in bias regarding expected query patterns, supporting unexpected user questions gracefully.

**Performance Optimization**: Database optimizers efficiently process dimensional queries by constraining heavily indexed dimension tables first, then attacking the fact table with the Cartesian product of satisfied dimension keys.
</design_principle>

### 2. Translate Dimensional Components to Report Elements

**Dimension Attributes Supply**:
- Report filters (WHERE clause constraints)
- Report labels (column headers, row labels)
- Grouping/break values (GROUP BY attributes)

**Facts Supply**:
- Numeric report values
- Aggregated metrics (SUM, AVG, COUNT)
- Calculated derived metrics

<example>
Simple report structure from dimensional model:

```
SELECT
    store.district_name,
    product.brand,
    SUM(sales_facts.sales_dollars) AS "Sales Dollars"
FROM store, product, date, sales_facts
WHERE
    date.month_name = "January" AND
    date.year = 2013 AND
    store.store_key = sales_facts.store_key AND
    product.product_key = sales_facts.product_key AND
    date.date_key = sales_facts.date_key
GROUP BY
    store.district_name,
    product.brand
```

**Report Structure**:
- **Filters**: January 2013 (from date dimension)
- **Row Headers**: District Name, Brand (from dimension attributes)
- **Metrics**: Sales Dollars (from fact table)
</example>

## Drilling Down and Drilling Up

**Drilling Down** = Adding more dimension attributes to row headers for additional detail

**Drilling Up** = Removing dimension attributes from row headers for summarization

### How Drilling Works

<drill_pattern>
1. **Start with Summary**: Sales Dollars by Department
2. **Drill Down on Hierarchy**: Add Brand → See Sales by Department and Brand
3. **Drill Down on Non-Hierarchy Attribute**: Add Fat Content → See Sales by Department, Brand, and Fat Content
4. **Drill Across Hierarchies**: Combine attributes from multiple hierarchies freely

**Key Insight**: You can drill down or up on:
- Attributes from explicit hierarchies (Product → Brand → Category → Department)
- Attributes NOT part of any hierarchy (Package Type, Fat Content)
- Multiple hierarchies simultaneously
</drill_pattern>

<sql_example>
Drilling from Department to Brand:

**Summary Query**:
```sql
SELECT product.department, SUM(sales_dollars)
FROM product, sales_facts
WHERE product.product_key = sales_facts.product_key
GROUP BY product.department
```

**Drill-Down Query** (add product.brand):
```sql
SELECT product.department, product.brand, SUM(sales_dollars)
FROM product, sales_facts
WHERE product.product_key = sales_facts.product_key
GROUP BY product.department, product.brand
```
</sql_example>

### Best Practices for Drill-Down Design

1. **Make all dimension attributes available** for drilling, not just hierarchy levels
2. **Provide clear attribute labels** so users understand drilling options
3. **Support drill-down across multiple dimensions** in the same query
4. **Enable drill-through to atomic detail** when users need transaction-level data
5. **Use verbose dimension attribute values** (not cryptic codes) for intuitive drill paths

## Drilling Across Fact Tables

**Drilling Across** = Combining metrics from different business process fact tables using conformed dimensions

### How Drill-Across Works

<drill_across_pattern>
**Scenario**: Compare Product Sales and Product Inventory using shared Product dimension

**Step 1 - Query Sales Fact Table**:
```sql
SELECT product.product_name, SUM(sales_facts.sales_dollars) AS sales
FROM product, sales_facts
WHERE product.product_key = sales_facts.product_key
  AND <date/store constraints>
GROUP BY product.product_name
```

**Step 2 - Query Inventory Fact Table**:
```sql
SELECT product.product_name, AVG(inventory_facts.quantity_on_hand) AS avg_inventory
FROM product, inventory_facts
WHERE product.product_key = inventory_facts.product_key
  AND <date/store constraints>
GROUP BY product.product_name
```

**Step 3 - Outer Join Results** on product_name (conformed attribute):
```
Product Name    | Sales      | Avg Inventory
----------------|------------|---------------
Cheerios 18oz   | $45,231    | 1,250
Tide 64oz       | $89,445    | 890
...
```

**Critical**: Use FULL OUTER JOIN to include all products from both fact tables, even if they only appear in one.
</drill_across_pattern>

### Requirements for Successful Drill-Across

<requirements>
1. **Conformed Dimensions**: Shared dimensions must have:
   - Identical attribute names across fact tables
   - Identical attribute values (e.g., "Brand A" not "BRAND A")
   - Consistent business definitions

2. **BI Tool Support**: Tool must support:
   - Multi-pass SQL queries
   - Result set joining (in temp tables, app server, or report layer)
   - May be called: multipass, multi-select, multi-fact, or stitch queries

3. **Cross-Fact Calculations**: Metrics combining facts from different tables (e.g., Sales per Unit of Inventory) must be calculated in the BI application after drill-across results return
</requirements>

**Vendor Terminology**: Different BI tools use different names for drill-across:
- Multipass queries
- Multi-select queries
- Multi-fact queries
- Stitch queries
- Federated queries

## BI Application Categories

### 1. Ad-Hoc Query Tools

**User Profile**: Power analysts (small percentage of user population)

**Characteristics**:
- Direct access to dimensional models
- Ability to construct custom SQL or drag-and-drop queries
- Maximum flexibility for unexpected analysis
- Requires strong understanding of data model
- Higher training requirements

**Design Considerations**:
- Provide comprehensive metadata for self-service
- Ensure dimension attribute labels are business-friendly
- Include detailed data lineage and definitions
- Support query result export to Excel/CSV
- Implement query governors to prevent runaway queries

### 2. Parameter-Driven Template Applications

**User Profile**: Majority of business users

**Characteristics**:
- Pre-built reports with user-selectable parameters
- Consistent presentation style and calculations
- Lower training requirements
- Constrained but still flexible analysis
- Ensures calculation consistency across organization

<template_example>
**Sales Analysis Template**:

**Parameters**:
- Date Range: [Start Date] to [End Date]
- Geographic Level: [ ] Store [ ] District [X] Region
- Product Level: [ ] SKU [X] Brand [ ] Category
- Metrics: [X] Sales Dollars [X] Units [ ] Gross Profit

**Generated Report**:
Shows selected metrics at chosen dimensional granularity with standard formatting
</template_example>

**Design Best Practices**:
- Limit parameters to 5-7 to avoid overwhelming users
- Provide sensible defaults (e.g., "Last Month" for date range)
- Use hierarchical parameter dependencies (select Region → filter to Districts in Region)
- Include metric definitions accessible from template
- Enable export and scheduling capabilities

### 3. Dashboards and Scorecards

**User Profile**: Executives and managers monitoring KPIs

**Characteristics**:
- Highly visual with charts, gauges, and traffic lights
- Focus on key metrics and exception highlighting
- Drill-down to supporting detail
- Often combines metrics from multiple business processes
- Automated refresh and delivery

**Design Patterns**:
- **Traffic Light Indicators**: Red/yellow/green based on thresholds
- **Trend Charts**: Time series showing metric direction
- **Exception Reports**: Highlight outliers requiring attention
- **Comparative Analysis**: Actual vs. target, this year vs. last year
- **Drill-Anywhere**: Click any metric to see supporting detail

### 4. Analytic Applications

**User Profile**: Specialized analysts performing advanced analytics

**Characteristics**:
- Sophisticated algorithms (forecasting, data mining, modeling)
- May write results back to operational systems or presentation area
- Often integrated with specialized statistical tools
- Leverage atomic grain data for maximum analytical power

## Standard BI Application Specification Template

<specification_template>
For each BI application, document:

### Business Purpose
- **Objective**: What business question does this answer?
- **Users**: Who will use this application?
- **Frequency**: How often will it be run?

### Data Sources
- **Fact Tables**: Which dimensional models are queried?
- **Conformed Dimensions**: Which shared dimensions enable drill-across?
- **Grain**: What level of detail is displayed?

### Report Layout
- **Filters/Parameters**: User-selectable constraints
- **Row Headers**: Dimension attributes in rows
- **Column Headers**: Dimension attributes or metrics in columns
- **Metrics**: Facts and calculations displayed
- **Sort Order**: Default sorting logic
- **Breaks/Subtotals**: Grouping and aggregation levels

### Calculations
- **Derived Facts**: Formulas for calculated metrics (define numerator/denominator for ratios)
- **Aggregation Rules**: How facts aggregate (SUM, AVG, COUNT)
- **Business Rules**: Special handling (e.g., semi-additive balance facts)

### Interactivity
- **Drill-Down Paths**: Which attributes can users drill to?
- **Drill-Across**: Can users combine with other fact tables?
- **Filters**: Which dimensions can users filter on interactively?
- **Export**: Can results be exported? To what formats?

### Performance
- **Expected Row Count**: Estimated result set size
- **Aggregates Leveraged**: Pre-built summary tables used?
- **Response Time Target**: Expected query duration
</specification_template>

## Metadata and Navigation

### Metadata for BI Applications

<metadata_requirements>
**Column-Level Metadata**:
- Business-friendly names (not database column names)
- Definitions and calculation formulas
- Data lineage (source system → ETL → warehouse table)
- Valid values and ranges
- Update frequency and latency

**Table-Level Metadata**:
- Fact table grain statements
- Dimension descriptions
- Relationships and join paths
- Refresh schedule
- Data quality metrics

**Application-Level Metadata**:
- Report purpose and audience
- Parameter definitions and valid values
- Calculation business rules
- Related reports and navigation paths
</metadata_requirements>

### Navigation Design

**Approaches for Organizing BI Applications**:

1. **By Business Process**: Group by value chain activity (Sales, Inventory, Procurement)
2. **By Organizational Function**: Group by department (Marketing, Finance, Operations)
3. **By User Role**: Group by job function (Executive, Analyst, Store Manager)
4. **By Analysis Type**: Group by analytical purpose (Performance Monitoring, Exception Analysis, Trend Analysis)

**Best Practice**: Use information portals or dashboards as starting points with multiple navigation paths to accommodate different user mental models.

## OLAP Cube Navigation

OLAP cubes provide enhanced navigation beyond relational BI tools:

<olap_advantages>
1. **Pre-calculated Aggregations**: Instant drill-down/up without re-querying
2. **Advanced Analytics**: Sophisticated calculations beyond SQL capabilities
3. **User-Friendly Manipulation**: Drag-and-drop pivoting, slicing, dicing
4. **Hierarchical Navigation**: Ragged hierarchies handled elegantly
5. **What-If Analysis**: Writeback capabilities for planning scenarios
6. **Non-Additive Fact Handling**: Semi-additive measures like balances properly averaged over time
</olap_advantages>

**OLAP Limitations**:
- Load performance penalty for large data volumes
- May require partial/full cube reprocessing for dimension attribute updates
- Limited support for accumulating snapshot fact tables (due to update constraints)
- Vendor-specific data structures less portable than relational

## Common Mistakes to Avoid

<anti_patterns>
1. **Building Applications Before Data is Ready**: Don't design BI apps in a vacuum without understanding the dimensional model

2. **Focusing Only on Top 10 Reports**: Requirements change; design for the business process, not just current report requests

3. **Inconsistent Calculations**: Different apps calculating same metric differently; enforce calculation standards

4. **Exposing Technical Complexity**: Showing database codes, surrogate keys, or technical jargon to business users

5. **Ignoring Performance**: Failing to leverage aggregates, indexes, or partitioning for acceptable query response

6. **Poor Naming**: Using database column names (PROD_SK, SLS_DLR_AMT) instead of business terms (Product, Sales Dollars)

7. **Bypassing Conformed Dimensions**: Building app-specific dimension subsets that break drill-across integration

8. **No Drill-Through to Detail**: Providing only summary data without paths to atomic transactions

9. **Overloading with Parameters**: Too many user selections create confusion and decision paralysis

10. **Static Design**: Failing to iteratively enhance applications based on actual user behavior and feedback
</anti_patterns>

## Implementation Checklist

<checklist>
**BI Application Specification Phase**:
- [ ] Identify 10-15 highest-priority reports/applications from business requirements
- [ ] Establish BI application standards (layout, navigation, naming, calculations)
- [ ] Specify each template using standard specification format
- [ ] Design navigation structure and information portal
- [ ] Get business user sign-off on specifications

**BI Application Development Phase**:
- [ ] Install BI tools and configure metadata
- [ ] Load subset of historical data for development/testing
- [ ] Establish coding standards and calculation libraries
- [ ] Develop template applications per specifications
- [ ] Test for data quality issues and query performance
- [ ] Conduct user acceptance testing with business representatives
- [ ] Create user documentation and training materials
- [ ] Establish support model (tiered support structure)

**BI Application Deployment Phase**:
- [ ] End-to-end system testing (data quality, performance, usability)
- [ ] User training (introductory, power user, refresher courses)
- [ ] Implement support infrastructure (help desk, online resources)
- [ ] Deploy applications with phased rollout approach
- [ ] Monitor usage patterns and performance metrics
- [ ] Collect user feedback for iterative enhancements
</checklist>

## Integration with DW/BI Lifecycle

<lifecycle_integration>
**BI Applications Track** (from Chapter 17):

1. **Business Requirements Definition** → Identify analytic needs, sample reports
2. **Dimensional Modeling** → Understand fact/dimension structure for app design
3. **Physical Design** → Aggregates and indexes driven by anticipated BI usage
4. **ETL Development** → Data quality issues discovered during BI app development
5. **BI Application Specification** → Prioritize and specify initial template set
6. **BI Application Development** → Build, test, refine applications
7. **Deployment** → Deploy with training, support, and ongoing enhancements

**Key Dependencies**:
- BI app development CAN BEGIN when database design is complete and subset of data is loaded
- BI app developers often find data quality issues that ETL team must fix
- BI app testing validates performance tuning strategies (aggregates, indexes)
- User feedback during BI app development shapes final ETL and physical design
</lifecycle_integration>

## Summary

BI applications are the ultimate deliverable of the DW/BI system—the interface through which business users access data to make better decisions. Effective BI application design:

- **Leverages dimensional model strengths**: Simplicity, symmetry, performance optimization
- **Supports intuitive analysis**: Drill-down, drill-up, drill-across using conformed dimensions
- **Addresses varied user needs**: Ad-hoc tools for power analysts, templates for casual users, dashboards for executives
- **Ensures consistency**: Standard calculations and business rules across the organization
- **Focuses on usability**: Business-friendly labels, clear metadata, intuitive navigation
- **Delivers performance**: Leverages aggregates, indexes, and optimization strategies

Remember: The technology and data architecture are merely means to an end. The true measure of DW/BI success is business user adoption and improved decision making enabled by well-designed BI applications.

**Next Steps**:
- Review existing reports from business requirements for design patterns
- Specify BI applications using standard template format
- Establish calculation libraries for consistent metric definitions
- Design navigation structure and information portal
- Develop and test applications iteratively with user feedback

For detailed BI application specification and development processes, see supporting reference files and Chapter 17 of *The Data Warehouse Toolkit*.
