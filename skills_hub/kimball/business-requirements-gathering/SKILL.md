---
name: business-requirements-gathering
description: Guide the process of gathering business requirements for data warehouse/business intelligence projects using Kimball methodology. Use when starting any DW/BI initiative, before dimensional modeling begins, to understand business needs, identify key stakeholders, define KPIs, and prioritize business processes.
---

# Business Requirements Gathering

## When to Use This Skill

Apply this skill at the very beginning of any DW/BI project, before any design work begins. This is the first step in the Kimball Lifecycle that ensures the DW/BI system focuses on business needs rather than technology.

## Core Objectives

The business requirements gathering process has two parallel objectives:

1. **Understand business needs**: Uncover objectives based on key performance indicators, compelling business issues, decision-making processes, and supporting analytic needs
2. **Assess data realities**: Meet with source system experts and perform high-level data profiling to assess data feasibilities

## Critical Success Factors

Before beginning requirements gathering, assess organizational readiness:

### Strong Executive Business Sponsor
- Most critical factor for DW/BI success
- Should have clear vision for DW/BI impact
- Must be politically astute and able to convince peers
- Preferably from business side, not just IT/CIO

### Compelling Business Motivation
- Must solve critical business problems
- Creates sense of urgency
- Can be external (competitive pressures) or internal (cross-org analysis needs)

### Data Feasibility
- Verify you're collecting real data in operational systems
- Assess data quality and granularity
- Most crucial because there's no short-term fix for missing data

## Requirements Preplanning

### Choose the Forum

**Interview Approach**:
- Easier to schedule
- Encourages individual participation
- One-on-one allows deeper probing
- 1 hour for individuals, 1.5 hours for small groups (2-3 people)

**Facilitated Sessions**:
- Reduces elapsed time
- Builds consensus
- Requires greater time commitment

**Recommended: Hybrid Approach**
- Conduct interviews to gather details
- Use facilitation to bring group to consensus
- Avoid surveys - they're too flat and miss opportunities to probe deeper

### Prepare the Requirements Team

**Interview Roles**:
- **Lead interviewer**: Asks open-ended questions
- **Scribe**: Takes copious notes (prefer human over tape recorder)
- **Observers**: 1-2 additional team members to hear directly from users

**Interview Preparation**:
- Research organization (annual report, website, org chart)
- Draft questionnaire as thinking tool (not rigid script)
- Ensure proper mindset - you will learn, don't presume you know it all
- Update questionnaire throughout interview process as you learn

### Select Business Representatives

**Horizontal Coverage**:
- Talk to people across organizational spectrum
- Critical for formulating enterprise data warehouse bus matrix
- Understand common data and vocabulary across core business functions

**Vertical Coverage**:
- Don't just focus on power analysts
- Include senior executives and middle management
- Avoid being overly tactical; understand strategic direction

**Scheduling Guidelines**:
- Allow 30 minutes between meetings for debriefing
- Maximum 3-4 sessions per day (interviewing is taxing)
- Have executive sponsor communicate importance to participants
- Ask interviewees to bring key reports and analyses

## Conducting Requirements Sessions

### Launch Phase

Script the introduction covering:
- Business-centric message about goals
- Avoid hardware/software/technical jargon
- Set professional, collaborative tone

### Interview Flow

**Start with Easy Questions**:
- Job responsibilities and organizational fit
- Key performance metrics (translates to facts)
- How they track progress and success

**Understand Business Dimensionality** (for hands-on data users):
- "How do you distinguish between products/agents/providers/facilities?"
- "How do you naturally categorize [dimension entity]?"
- Identifies key dimension attributes and hierarchies

**Explore Analytics** (for analytic users):
- Types of analysis currently generated
- Ad hoc vs. standardized reports
- Review brought reports: current use and improvement opportunities

**Strategic Direction** (for executives):
- Vision for better leveraging information
- Balance between ad hoc vs. standardized delivery
- Ensure deliverable matches business demand

**Quantify Benefits**:
- Ask about impact of improved access to information
- Capture quantifiable benefits for justification

### Wrap-Up

- Ask about success criteria (must be measurable)
- Make broad disclaimer: discussed ≠ guaranteed for first phase
- Thank participants
- Communicate next steps and their future involvement

## Parallel Data-Centric Interviews

Intersperse business interviews with source system expert sessions:

**Goals**:
- Ascertain whether necessary core data exists
- Assess data quality before momentum builds
- Request initial data profiling (domain values, counts)
- Manage expectations appropriately

## Documentation

### Immediate Post-Interview Activities

**Debrief**:
- Ensure team alignment on learnings
- Review notes while fresh (fill in gaps)
- Examine collected reports for dimensionality insights

### Documentation Deliverables

**Individual Interview Write-Ups** (optional but helpful):
- Not stream-of-consciousness transcript
- Organized, coherent narrative

**Consolidated Findings Document** (critical):
- **Executive summary**
- **Project overview**: Process used, participants involved
- **Business processes**: For each process, document:
  - Why users want to analyze performance metrics
  - What capabilities they want
  - Current limitations
  - Potential benefits/impact
  - Feasibility assessment
- **Opportunity/Stakeholder Matrix**: Shows which organizational groups want access to which business processes
- **Simplified Bus Matrix**: Rows are business processes, columns are conformed dimensions

## Prioritization

### Prioritization Session

**Participants**:
- Senior business representatives (who participated in interviews)
- DW/BI manager and senior IT management

**Process**:
1. Present overview of each identified business process
2. Review opportunity/stakeholder matrix
3. Review simplified bus matrix
4. Place each business process on prioritization grid

### Prioritization Grid

Two axes for evaluation:

**Vertical Axis: Business Impact/Value**
- High impact = upper half
- Low impact = lower half

**Horizontal Axis: Feasibility**
- High feasibility = right half
- Low feasibility = left half

**Four Quadrants**:
- **Upper-right (shaded)**: High impact + high feasibility = START HERE
- **Upper-left**: High impact + low feasibility = Address feasibility barriers while others work on upper-right
- **Lower-right**: Low impact + high feasibility = Don't prioritize (too easy but not important)
- **Lower-left**: Low impact + low feasibility = AVOID (mission impossible)

## Publishing Metaphor: Understanding Your Role

Think of yourself as a magazine editor-in-chief for data:

### Understand the Readers (Business Users)
- Their demographic characteristics
- What they want from the DW/BI system
- Identify "best" users who make high-impact decisions
- Find potential new users

### Deliver Quality Content
- Choose robust, actionable data
- Make interfaces simple and template-driven
- Ensure accuracy and trustworthiness
- Apply consistent labeling across enterprise
- Continuously monitor data accuracy

### Sustain the Environment
- Take credit for business decisions made using DW/BI
- Update system regularly
- Maintain business users' trust
- Keep stakeholders and management happy

## Key Requirements Principles

### Focus on Business Processes, Not Departments
- Business processes cross organizational boundaries
- Examples: taking an order, processing a claim, registering students
- Multiple departments often want to analyze same performance metrics
- Avoid creating departmental data silos

### Listen for Dimensional Indicators

When business users describe their needs, listen for:
- **Facts**: "We need to track sales, quantity, profit..."
- **Dimensions**: "We want to analyze by product, market, time, customer..."
- **Grain**: "We need to see every transaction / daily snapshots / order lifecycle..."
- **Hierarchies**: "Roll up from store to district to region"

### Avoid Common Pitfalls

**Don't**:
- Focus on technology before understanding business needs
- Design for top 10 reports in isolation
- Presume you know what users need
- Make prioritization decisions in IT vacuum
- Rely solely on surveys
- Schedule too many interviews per day

**Do**:
- Collaborate with business throughout
- Focus on business process measurement events
- Ask open-ended questions
- Probe deeper when you hear vague requirements
- Validate data feasibility in parallel
- Document immediately after sessions

## Deliverables Checklist

At the end of requirements gathering, you should have:

- [ ] Consolidated findings document
- [ ] Opportunity/stakeholder matrix
- [ ] Simplified enterprise data warehouse bus matrix
- [ ] Prioritized list of business processes (via prioritization grid)
- [ ] Initial KPI definitions
- [ ] Stakeholder matrix
- [ ] Data feasibility assessment
- [ ] Quantified business benefits
- [ ] Collection of sample reports and analyses

## Next Steps After Requirements

Once requirements are gathered, documented, and prioritized:

1. **Present findings** to senior management and participants
2. **Gain consensus** on priorities
3. **Scope first project** focusing on single business process (avoid the Law of Too: too many sources, users, locations, requirements)
4. **Begin dimensional modeling** for highest-priority business process
5. **Maintain business partnership** throughout design and development

## Remember

Business requirements drive everything in the Kimball Lifecycle. Without solid requirements:
- You can't design meaningful dimensional models
- You can't prioritize what to build
- You can't measure success
- Business users won't adopt the system

The requirements process is not just about gathering information - it's about building a partnership with the business that will sustain the DW/BI initiative through all subsequent phases.
