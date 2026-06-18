---
name: program-management
description: Manage DW/BI projects and programs throughout the Kimball Lifecycle—from readiness assessment, scoping, requirements gathering, and prioritization through team staffing, project planning, deployment, and ongoing maintenance. Use when planning new DW/BI initiatives, managing active projects, or maintaining production systems.
---

# DW/BI Program Management

## When to Use This Skill

Use this skill throughout the entire data warehouse and business intelligence lifecycle:

**Planning Phase:**
- Assessing organizational readiness for DW/BI initiatives
- Scoping and justifying new DW/BI projects
- Staffing cross-functional teams (business and IT)
- Developing project plans and communication strategies

**Active Project Management:**
- Managing parallel technology, data, and BI application tracks
- Coordinating business requirements gathering and prioritization
- Facilitating stakeholder communication and managing scope
- Planning deployment with education and support

**Production Operations:**
- Maintaining deployed DW/BI systems with ongoing support
- Planning for growth and expansion
- Managing the program portfolio across multiple iterations

This skill complements all other Kimball dimensional modeling and ETL skills by providing the project and program management framework that enables successful delivery.

---

## Core Concepts

### The Kimball DW/BI Lifecycle

The Kimball Lifecycle is an iterative, business-focused approach with three core principles:

1. **Focus on business needs** - Technology serves business requirements, not vice versa
2. **Present dimensionally structured data** - Simple, queryable dimensional models
3. **Tackle manageable iterations** - Deliver one business process at a time

The lifecycle includes three parallel tracks following requirements:
- **Technology Track**: Architecture design → Product selection → Installation
- **Data Track**: Dimensional modeling → Physical design → ETL development
- **BI Applications Track**: BI app specification → Development

All tracks converge at **Deployment**, followed by **Maintenance and Growth**.

### Alignment with Agile Methodologies

The Kimball Lifecycle shares common ground with agile:
- Focus on business value
- Collaborate with business users
- Develop incrementally

**Key difference**: DW/BI requires solid data architecture (bus matrix) and governance foundation. Bundle multiple agile deliverables into full-function releases before broad deployment.

---

## Step-by-Step Process

### Phase 1: Program/Project Planning and Launch

#### Step 1: Assess Organizational Readiness

Evaluate three critical readiness factors (in rank order):

<readiness_assessment>
**Factor 1: Strong Executive Business Sponsor (MOST CRITICAL)**
- Clear vision for DW/BI impact on the organization
- Track record of success with internal initiatives
- Politically astute leader who can convince peers
- Business sponsor preferred over CIO sponsor
- **Red flag**: IT-driven sponsorship without business champion

**Factor 2: Compelling Business Motivation**
- Solves critical business problems
- Creates sense of urgency
- Driven by external factors (competitive pressure) or internal needs (cross-org analysis post-merger)
- **Red flag**: Vague justification like "single version of truth" without quantifiable impact

**Factor 3: Data Feasibility**
- Real data exists in operational source systems
- Reasonably clean source data at right granularity
- **Red flag**: Required data not currently collected (no short-term fix)
</readiness_assessment>

**Deliverable**: Go/no-go decision with readiness assessment documentation

#### Step 2: Scope and Justify the Project

**Scoping Guidelines:**
- Joint input from IT and business management required
- Meaningful to business + manageable for IT
- Focus on single business process initially
- Save cross-process projects for later phases
- **Avoid the "Law of Too"**: Too brief timeline, too many sources, too many users, too many locations, too diverse requirements

**Justification Approach:**
- IT estimates costs (include room for short-term growth)
- Business determines financial benefits (increased revenue/profit, not just expense reduction)
- Benefits should grossly outweigh costs
- Peel back the layers: "single version of truth" → specific quantifiable impact of improved decisions
- **Symptom of wrong focus**: Struggling with justification means wrong sponsor or wrong problem

**Deliverable**: Project scope statement and business case with ROI

#### Step 3: Staff the Cross-Functional Team

<team_roles>
**Business Roles:**
- **Business Sponsor**: Ultimate client and strongest advocate (may be steering committee)
- **Business Driver**: Middle manager delegate if sponsor too removed
- **Business Lead**: Highly involved, daily communication with project manager
- **Business Users**: Enthusiastic fans involved early and often

**Business/IT Straddler Roles:**
- **Business Analyst**: Determines needs, translates to architectural/data/BI requirements
- **Data Steward**: Subject matter expert on data meaning, usage, inconsistencies
- **BI Application Designer/Developer**: Designs analytic templates and provides support

**IT Roles:**
- **Project Manager**: Comfortable with executives and technical resources; stellar communication skills
- **Technical Architect**: Overall technical architecture and product evaluation
- **Data Architect/Modeler**: Embraces dimensional modeling; empathetic to business needs
- **Database Administrator**: Willing to set aside OLTP truisms (e.g., single index per table)
- **Metadata Coordinator**: Establishes metadata repository strategy
- **ETL Architect/Designer**: Designs ETL environment and processes
- **ETL Developer**: Builds and automates ETL processes (likely using ETL tool)
</team_roles>

**Note**: These are roles, not people. Small teams fill multiple roles per person.

**Deliverable**: Team roster with roles, responsibilities, and RACI matrix

#### Step 4: Develop and Maintain the Project Plan

**Plan Development:**
- Identify all necessary Lifecycle tasks (detailed task list at kimballgroup.com)
- Key team members estimate effort for their tasks (don't dictate time)
- Include acceptance checkpoints with business after every major milestone
- Establish communication strategy for sponsors, business community, and IT colleagues

**Scope Management:**
When confronted with changes, three options:
1. **Increase scope**: Add time, resources, or budget
2. **Zero-sum game**: Retain original scope by giving up something
3. **Say "no" diplomatically**: Handle as enhancement request for next iteration

**Critical**: Scope decisions should NOT be made in IT vacuum. Leverage business partnership.

**Deliverable**: Detailed project plan with tasks, estimates, dependencies, checkpoints, and communication plan

---

### Phase 2: Business Requirements Definition

#### Step 5: Pre-Plan Requirements Sessions

**Choose the Forum:**
- **Interviews**: Individual participation, easier to schedule
- **Facilitated sessions**: Reduce elapsed time, require more commitment
- **Hybrid approach (recommended)**: Interviews for details + facilitation for consensus
- **Avoid surveys**: Flat, two-dimensional, no probing, don't build user bond

**Identify and Prepare Requirements Team:**
- **Lead interviewer**: Asks great open-ended questions
- **Interview scribe**: Takes copious notes (no tape recorder—changes dynamics)
- **Additional observers** (1-2): Hear user input directly

**Requirements Team Mindset:**
- Don't presume you know it all
- Do homework: annual report, website, org chart
- Draft questionnaires (not scripts—tools to organize thoughts)

**Select, Schedule, and Prepare Business Representatives:**
- **Horizontal spectrum**: Understand common data/vocabulary across functions (for bus matrix)
- **Vertical coverage**: Executives, middle management, power analysts (don't ignore strategic direction)
- **Scheduling**: Executives individually; 2-3 homogeneous groups; 1 hour individual, 1.5 hours groups; 0.5 hour between for debrief
- **Preparation**: Business sponsor communicates commitment; interviewees bring key reports/analyses

**Deliverable**: Interview questionnaires, scheduled sessions, prepared participants

#### Step 6: Collect Business Requirements

**Launch:**
- Script introduction talking points
- Convey crisp, business-centric message (avoid technical jargon)

**Interview Flow:**

<interview_questions>
**Opening (nonthreatening lob-ball):**
- Job responsibilities and organizational fit

**Core Business Questions:**
- "What are your key performance metrics?" → Identifies business processes and facts
- "How do you track progress and success?" → Translates to dimensional model

**For Hands-On Data Users:**
- "How do you distinguish between products/agents/providers/facilities?" → Key dimension attributes
- "How do you naturally categorize products?" → Hierarchies

**For Analytic Users:**
- Types of analysis currently generated (ad hoc vs standardized) → BI tool requirements
- How they use analyses today and improvement opportunities
- **Resist**: Narrowing design to "top five reports" (questions will change)

**For Executives:**
- Vision for better leveraging information (avoid tactical details)
- Expectations: totally ad hoc vs standardized analyses
- Impact of improved access to information → Quantifiable benefits

**Wrap-Up:**
- Success criteria (measurable specifics, not "easy to use" or "fast")
- Disclaimer: Discussion doesn't guarantee inclusion in first phase
- What's happening next and their involvement
</interview_questions>

**Deliverable**: Interview notes and collected sample reports

#### Step 7: Conduct Data-Centric Interviews (Interspersed)

**Purpose**: Evaluate feasibility of supporting business needs before momentum builds

**Activities:**
- Sessions with source system data gurus/subject matter experts
- Initial data profiling results (domain values, counts of critical fields)
- Ascertain necessary core data exists (avoid standing on quicksand)
- Manage organization's expectations appropriately

**Deliverable**: Data feasibility assessment for each business process

#### Step 8: Document Requirements

**Immediate Post-Interview:**
- Debrief with interview team (ensure same page)
- Review notes shortly after to fill gaps (abbreviations become incomprehensible)
- Examine gathered reports for dimensionality insight

**Documentation Levels:**

1. **Individual Interview Write-Ups (optional, time-consuming)**
   - Not stream-of-consciousness transcripts
   - Must make sense to non-attendees

2. **Consolidated Findings Document (CRITICAL)**
   - Organized around key business processes (future implementation efforts)
   - Executive summary
   - Project overview (process used, participants)
   - **For each process**: Why analyze, what capabilities wanted, current limitations, potential benefits, feasibility
   - Opportunity/stakeholder matrix (processes × organizational groups)

**Deliverable**: Consolidated findings document with opportunity/stakeholder matrix

#### Step 9: Prioritize Requirements

**Preparation:**
- Consolidated findings as basis for presentation
- Participants: Senior business reps (from interviews), DW/BI manager, senior IT management

**Presentation Flow:**
1. Overview of each identified business process
2. Review opportunity/stakeholder matrix
3. Review simplified bus matrix

**Prioritization Grid:**

```
         High Feasibility
              ^
              |
    Low   |       | Start Here!
    Impact|-------|-----------> High Impact
          | Avoid | Meaningful
          |       | (but currently infeasible)
```

- **Vertical axis**: Potential impact/value to business
- **Horizontal axis**: Feasibility
- **Upper-right (shaded)**: Immediate attention—high impact AND highly feasible
- **Lower-left**: Avoid—low impact AND infeasible (missions impossible)
- **Lower-right**: Don't prioritize short-term—doable but not crucial
- **Upper-left**: Meaningful opportunities requiring other IT teams to address feasibility first

**Deliverable**: Prioritized list of business processes with impact/feasibility ratings

---

### Phase 3: Parallel Track Execution

The Lifecycle proceeds with three concurrent tracks (details in separate skills):

**Technology Track:**
1. Technical architecture design (8-step process—see supporting reference)
2. Product selection and installation (6-step process—see supporting reference)

**Data Track:**
1. Dimensional modeling (Use **four-step-dimensional-modeling** skill)
2. Physical design (Use **physical-database-design-optimization** skill)
3. ETL design and development (Use **etl-system-architecture-design**, **historic-data-loading-strategy**, **incremental-etl-processing** skills)

**BI Applications Track:**
1. BI application specification (Use **bi-application-design-patterns** skill)
2. BI application development

**Project Manager Responsibilities During Parallel Tracks:**
- Coordinate dependencies between tracks
- Manage resources across multiple work streams
- Maintain communication with business and IT stakeholders
- Track progress against plan and manage scope
- Identify and resolve cross-track issues

---

### Phase 4: Deployment

#### Step 10: Pre-Deployment Testing and Readiness Assessment

**Testing Categories:**
- **End-to-end system testing**: Complete data flow from source to BI application
- **Data quality assurance**: Validate data accuracy and completeness
- **Operations processing**: ETL job scheduling, monitoring, error handling
- **Performance testing**: Query response times under realistic load
- **Usability testing**: Business user acceptance

**Readiness Assessment:**
- **Data is the "main entrée"**: Most unpredictable task
- **Honesty required**: Don't deploy undercooked data on promised date
- **Better to delay**: Than deliver poor quality and lose user trust

**Deployment Analogy**: Like serving holiday meal—can't serve until main entrée (data) is fully cooked. Slow down side dishes (BI apps, infrastructure) if needed rather than serving undercooked data.

**Deliverable**: Test results, quality scorecards, and go/no-go deployment decision

#### Step 11: Package Education and Support

**Education:**
- User training materials (role-based)
- Power user advanced training
- BI tool-specific courses
- Self-service resources and documentation

**Support Strategy (Tiered Structure):**

<support_tiers>
**Tier 1: Self-Service Support**
- Website with FAQs, documentation, training videos
- Searchable knowledge base
- Sample reports and templates

**Tier 2: Business Area Power Users**
- Business-embedded resources who understand domain
- First line of defense for business-specific questions
- Escalate to Tier 3 when needed

**Tier 3: Centralized DW/BI Team**
- Final line of defense
- Technical issues, data quality problems, ETL failures
- Enhancement requests
</support_tiers>

**Deliverable**: Education curriculum and support model documentation

#### Step 12: Deploy to Production

**Deployment Activities:**
- Migrate from development/test to production environment
- Execute final ETL historic load
- Validate production data quality
- Enable user access (phased rollout recommended)
- Activate monitoring and support processes

**Phased Rollout Approach (Recommended):**
1. **Pilot**: Limited user group (power users, friendly executives)
2. **Controlled expansion**: Department by department or region by region
3. **General availability**: Broad business community

**Deliverable**: Production DW/BI system with enabled user community

---

### Phase 5: Maintenance and Growth

#### Step 13: Provide Ongoing Support

**User Support (Immediately Crucial Post-Deployment):**
- Relocate to business community temporarily (easy access)
- Don't assume no news is good news (likely no one using system)
- Be honest about problems to build credibility
- Take immediate corrective action
- **Quality Impact**: Poor quality = overwhelming support demands for reconciliation and rework

**Continuing Education Program:**
- Formal refresher and advanced courses
- Repeat introductory courses for new users
- Informal education for developers and power users
- Encourage interchange of ideas

**Technical Support (Production Environment):**
- Service level agreements
- Proactive performance monitoring
- System capacity trend analysis
- **Don't rely on business to report degradation**

**Program Support (Beyond Single Phase):**
- Monitor and market success
- Communicate with varied constituencies
- Ongoing checkpoint reviews
- Assess and identify improvement opportunities

**Deliverable**: Support metrics, user satisfaction surveys, performance reports

#### Step 14: Plan for Growth

**Signs of Demand for Growth (Success Indicators):**
- New users
- New data / additional business processes
- New BI applications
- Major enhancements to existing deliverables

**Growth Prioritization:**
- **Don't decide in IT vacuum**: Business involvement required
- Leverage prioritization grid (impact vs feasibility)
- Establish executive business sponsorship committee for portfolio prioritization
- Align priorities with organizational objectives

**Return to Lifecycle Beginning:**
After identifying new priorities:
1. Go back to Program/Project Planning (Phase 1)
2. Leverage existing technical, data, and BI foundations
3. Focus on new requirements
4. **Iterative approach**: Each subsequent project returns to beginning of Lifecycle

**Deliverable**: Growth roadmap with prioritized initiatives

---

## Technical Architecture Design Process (8 Steps)

The technical architecture serves as the integration framework, much like housing blueprints. It supports coordination of parallel efforts, modular component reuse, and phased implementation.

<architecture_process>
**Step 1: Establish Architecture Task Force**
- 2-3 people: Technical architect + ETL architect/designer + BI application architect/designer
- Ensures back room and front room representation

**Step 2: Collect Architecture-Related Requirements**
- Leverage business requirements with "architectural implications" filter
- Listen for timing, availability, and performance requirements
- Conduct IT organization interviews: current standards, planned directions, non-negotiable boundaries, lessons learned, willingness to accommodate operational change

**Step 3: Document Architecture Requirements**
- Simplistic tabular format: Business requirement → Architectural implications
- Example: "Global sales data nightly" → 24/7 availability, data mirroring, robust metadata, network bandwidth, ETL horsepower

**Step 4: Create Architecture Model**
- Sequester architecture team for several days
- Group requirements into major components (ETL, BI, metadata, infrastructure)
- Draft high-level architectural model (front elevation view)
- Significant details embedded in following pages

**Step 5: Determine Architecture Implementation Phases**
- Mandatory capabilities vs nice-to-haves
- Refer to business requirements for priorities
- Minimally provide elements needed for initial project delivery

**Step 6: Design and Specify Subsystems**
- Major functionality from vendor standard offerings
- Define custom subsystems in detail for build-vs-buy evaluation

**Step 7: Create Architecture Plan Document**
- Adequate detail for skilled professionals to construct framework
- Typically doesn't reference specific products (except existing in-house)

**Step 8: Review and Finalize Technical Architecture**
- Communicate at varying levels to project team, IT colleagues, business leads
- Update documentation
- Use immediately in product selection process
</architecture_process>

**Critical**: Neither extreme is healthy—don't bypass architecture (ends in rebuild) or spend two years designing perfect architecture (forgets business problems). Middle ground is appropriate.

**Deliverable**: Technical architecture plan document

**Reference**: See `technical-architecture-design.md` for detailed guidance

---

## Product Selection Process (6 Steps)

The architecture plan serves as shopping list for products fitting the framework.

<product_selection>
**Step 1: Understand Corporate Purchasing Process**
- Internal hardware and software purchase processes
- Procurement requirements and timelines

**Step 2: Develop Product Evaluation Matrix**
- Start from architecture plan
- Spreadsheet with evaluation criteria and weighting factors
- More specific criteria = better (avoid vague criteria where every vendor says "yes")

**Step 3: Conduct Market Research**
- Understand players and offerings
- **Avoid RFPs if possible**: Tremendously time-consuming, beauty contest responses, value may not warrant effort
- Exception: Some organizations have no choice

**Step 4: Evaluate Short List of Options**
- Focus on vendors meeting functionality AND technical requirements
- Compare preliminary scores from evaluation matrix
- **Involve business representatives** if evaluating BI tools
- **You drive the process**, not vendors
- Share relevant architecture plan information
- Talk with references (formal and informal network)

**Step 5: Conduct Prototype (If Necessary)**
- Clear winner? Bypass prototype (save time and money)
- Sole candidate via experience/relationships/commitments? Bypass prototype
- No apparent winner? Prototype with maximum 2 products
- Develop limited yet realistic business case study
- **You take charge** of prototype process

**Step 6: Select Product, Install on Trial, and Negotiate**
- Make private commitment to single vendor (preserve negotiating power)
- Trial period: Put product to real use in your environment
- **Don't use trial as tire-kicking**: Walk this path only with vendor you intend to buy from
- Trial requires significant energy (install, training, using)
- Negotiate purchase at end of trial (beneficial to all parties)
</product_selection>

**Deliverable**: Product selection documentation and negotiated contracts

**Reference**: See `product-selection-guide.md` for detailed criteria and evaluation templates

---

## Common Pitfalls to Avoid (Top 10)

Lethal errors—one alone may bring down the initiative:

<pitfalls>
**Pitfall 10**: Technology and data enamor over business requirements and goals
**Pitfall 9**: Fail to embrace/recruit influential, accessible, reasonable senior business sponsor
**Pitfall 8**: Tackle galactic multiyear project instead of manageable iterations
**Pitfall 7**: Build normalized data structure, run out of budget before dimensional presentation
**Pitfall 6**: Prioritize back room operational performance over front room query performance and ease of use
**Pitfall 5**: Make queryable presentation area overly complex (support users for a year to appreciate simpler solutions)
**Pitfall 4**: Populate dimensional models standalone without conformed dimensions architecture
**Pitfall 3**: Load only summarized data into dimensional structures (lose atomic detail)
**Pitfall 2**: Presume business, requirements, analytics, data, and technology are static
**Pitfall 1**: Neglect business acceptance as success criterion (without user acceptance, efforts are futile)
</pitfalls>

---

## Implementation Checklist

### Program/Project Planning
- [ ] Readiness assessment completed (sponsor, motivation, feasibility)
- [ ] Project scope defined (meaningful + manageable, single business process)
- [ ] Business case with ROI documented
- [ ] Cross-functional team staffed with defined roles
- [ ] Project plan developed with task estimates and checkpoints
- [ ] Communication strategy established

### Business Requirements
- [ ] Requirements team identified and prepared
- [ ] Interview questionnaires drafted
- [ ] Business representatives scheduled and prepared
- [ ] Requirements interviews conducted (business and data-centric)
- [ ] Consolidated findings document created
- [ ] Opportunity/stakeholder matrix developed
- [ ] Requirements prioritized using impact/feasibility grid
- [ ] Bus matrix updated with prioritized business processes

### Architecture and Products
- [ ] Architecture task force established
- [ ] Architecture requirements collected and documented
- [ ] Architecture model created with implementation phases
- [ ] Architecture plan document finalized and reviewed
- [ ] Product evaluation matrix developed
- [ ] Short list of products evaluated
- [ ] Products selected and contracts negotiated

### Deployment
- [ ] End-to-end system testing completed
- [ ] Data quality validated (readiness honestly assessed)
- [ ] Performance testing results acceptable
- [ ] Usability testing with business users successful
- [ ] Education materials and curriculum developed
- [ ] Tiered support model established
- [ ] Deployment executed (phased rollout recommended)

### Maintenance and Growth
- [ ] User support provided (relocate to business temporarily)
- [ ] Continuing education program established
- [ ] Technical support with SLAs and monitoring active
- [ ] Program support with checkpoint reviews ongoing
- [ ] Success metrics monitored and communicated
- [ ] Growth priorities identified and approved
- [ ] Next iteration planned (return to Lifecycle beginning)

---

## Integration with Other Skills

This program management skill orchestrates all other Kimball skills:

**Foundation Skills (Phase 1):**
- **business-requirements-gathering**: Detailed interview techniques and requirements documentation
- **enterprise-bus-matrix-design**: Enterprise architecture framework from requirements

**Design Skills (Phase 3 - Data Track):**
- **four-step-dimensional-modeling**: Core modeling process
- **dimension-table-design-patterns**, **fact-table-design-patterns**: Detailed design techniques
- **physical-database-design-optimization**: Physical implementation

**ETL Skills (Phase 3 - Data Track):**
- **etl-system-architecture-design**: Complete ETL framework (34 subsystems)
- **data-profiling-quality-assessment**: Pre-ETL and ongoing quality
- **historic-data-loading-strategy**, **incremental-etl-processing**: ETL development

**BI Skills (Phase 3 - BI Track):**
- **bi-application-design-patterns**: BI application development

**Integration Skills (Ongoing):**
- **conformed-dimension-management**: Enterprise integration across projects

**Program management provides**:
- Overall project governance and coordination
- Business partnership and stakeholder management
- Scope, schedule, and resource management
- Quality gates and acceptance criteria
- Deployment and production support framework

---

## Key Success Factors

1. **Strong business sponsorship**: Visible, committed, politically astute executive
2. **Compelling business motivation**: Solve critical problems with quantifiable impact
3. **Business-IT partnership**: Joint ownership and decision-making
4. **Appropriate scope**: Meaningful to business + manageable for IT (avoid "Law of Too")
5. **Honest readiness assessment**: Don't deploy undercooked data on promised date
6. **Continuous business involvement**: Requirements → Design → Review → Acceptance
7. **Solid data architecture**: Bus matrix and conformed dimensions for integration
8. **Iterative delivery**: One business process at a time, build on existing foundations
9. **Quality focus**: Testing, validation, and honest assessment before deployment
10. **Ongoing support and communication**: Monitor success, market value, plan growth

---

## Quick Reference: Lifecycle Phases

| Phase | Key Activities | Primary Skills | Duration |
|-------|---------------|----------------|----------|
| **Program/Project Planning** | Readiness, scope, staff, plan | Program management | 2-4 weeks |
| **Requirements** | Interview, document, prioritize | Business requirements gathering | 4-6 weeks |
| **Architecture** | Design, product selection | Technical architecture | 3-4 weeks |
| **Dimensional Modeling** | High-level + detailed design | Four-step modeling | 3-4 weeks per process |
| **Physical Design** | Database, indexing, aggregates | Physical design | 2-3 weeks |
| **ETL Development** | Design, historic load, incremental | ETL skills | 10-16 weeks |
| **BI Development** | Specification, development | BI application design | 6-8 weeks |
| **Deployment** | Testing, education, support, rollout | Program management | 2-3 weeks |
| **Maintenance** | Support, monitoring, enhancements | Program management | Ongoing |
| **Growth** | Prioritize, plan next iteration | Program management | Return to start |

**Note**: Phases overlap. Technology, data, and BI tracks run in parallel after requirements.

---

## References

**Primary Chapters:**
- Chapter 17: Complete Kimball DW/BI Lifecycle overview
- Chapter 18: Dimensional modeling process and tasks (collaborative workshop approach)

**Supporting Files:**
- `technical-architecture-design.md`: Complete 8-step architecture design process
- `product-selection-guide.md`: Detailed product evaluation criteria and templates
- `requirements-interview-guide.md`: Comprehensive questionnaires and facilitation techniques

**External Resources:**
- Detailed Lifecycle task list: kimballgroup.com (Tools & Utilities → The Data Warehouse Lifecycle Toolkit, 2nd Edition)
- Dimensional design worksheet template: kimballgroup.com (same location)

---

**Master Plan Reference**: This is Skill #20 in the Data Warehouse Skills Master Plan. Use throughout all lifecycle phases for DW/BI project and program management.
