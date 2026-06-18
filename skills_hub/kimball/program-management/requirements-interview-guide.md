# Requirements Interview Guide - Detailed Reference

## Purpose

Business requirements gathering is absolutely essential to DW/BI success. This guide provides comprehensive interview techniques, questionnaires, facilitation approaches, and documentation templates for collecting, consolidating, and prioritizing business requirements.

---

## Interview vs Facilitation Decision

<forum_selection>
**Interview Approach:**
- **Format**: One-on-one or small group (2-3 people)
- **Advantages**:
  - Encourages individual participation
  - Easier to schedule
  - More depth per individual
  - Less intimidating for junior staff
- **Disadvantages**:
  - More elapsed time (multiple sessions)
  - May hear conflicting perspectives
  - Requires subsequent consolidation

**Facilitated Session Approach:**
- **Format**: Larger group workshop (5-15 people)
- **Advantages**:
  - Reduces elapsed time to gather requirements
  - Immediate consensus building
  - Group dynamics surface conflicts early
  - Shared understanding across participants
- **Disadvantages**:
  - Requires more time commitment from each participant
  - Dominant personalities may overshadow others
  - Harder to schedule (getting everyone together)
  - Requires skilled facilitator

**Hybrid Approach (Recommended):**
- Conduct individual/small group interviews first (detail gathering)
- Follow with facilitated session (consensus building and prioritization)
- Balances depth of interviews with efficiency of group facilitation
- Interview → consolidate → facilitate → finalize

**Survey Approach (NOT RECOMMENDED):**
- Flat and two-dimensional
- Self-selected respondents
- Answer only questions you thought to ask in advance
- No option to probe more deeply
- Doesn't forge bond between users and DW/BI initiative
- **Exception**: Pre-interview survey to gather basic info (job role, key reports, pain points) before detailed interview
</forum_selection>

---

## Interview Team Roles and Preparation

### Interview Team Composition

<interview_team>
**Lead Interviewer:**
- **Primary responsibility**: Ask great open-ended questions
- **Skills needed**: Active listening, curiosity, ability to probe deeper
- **Focus**: Engage interviewee, understand business context and needs
- **Not responsible for**: Note-taking (that's scribe's job)

**Interview Scribe:**
- **Primary responsibility**: Take copious notes
- **Skills needed**: Fast, accurate note-taking; ability to capture essence while typing
- **Focus**: Document what's said, key reports shared, decisions made
- **Not recommended**: Tape recorder (changes meeting dynamics, makes people cautious)

**Observers (1-2 Additional Team Members):**
- **Primary responsibility**: Listen and learn about business
- **Skills needed**: Observation, pattern recognition
- **Focus**: Hear users' input directly (not filtered through documentation)
- **Value**: Another brain, set of eyes and ears; begin learning source systems and business processes
</interview_team>

### Pre-Interview Preparation

<preparation_checklist>
**Research and Homework:**
- Review organization's annual report
- Examine public website (products, services, markets)
- Study internal organization chart (reporting relationships, business functions)
- Research industry trends and competitive landscape
- Review any existing analytic reports or dashboards (if available)

**Draft Interview Questionnaire:**
- **Purpose**: Organize your thoughts, not script to read verbatim
- **Use as**: Fallback device if mind goes blank during session
- **Update throughout**: Refine questions as team becomes better versed in business

**Mindset:**
- Don't presume you already know it all
- You WILL learn more about business during sessions
- Approach with genuine curiosity and respect for interviewee's expertise
- Business users are the experts on their business—you're there to learn

**Logistics:**
- Confirm meeting room (preferably interviewee's location for comfort and access to materials)
- Bring copies of agenda and any pre-read materials
- Laptop for scribe's note-taking
- Business cards for team introductions
</preparation_checklist>

---

## Interview Questionnaire Templates

### Executive Interview Questionnaire

**Audience**: C-level executives, VPs, senior directors

<executive_questions>
**Opening (5 minutes):**
- Introduction of DW/BI initiative and project team
- Interviewee's role and background (brief)

**Strategic Vision (15 minutes):**
1. What is your vision for how the organization could better leverage information?
2. What decisions do you make regularly that would benefit from better data access?
3. How do you currently track the health of your business area?
4. What questions do you wish you could answer today but can't?

**Organizational Impact (10 minutes):**
5. What would improved access to information enable your organization to do differently?
6. How would you measure the success of a DW/BI initiative?
7. What is the quantifiable impact of better decision making in your area? (Revenue increase? Cost reduction? Risk mitigation?)

**Expectations and Concerns (10 minutes):**
8. What are your expectations for the DW/BI deliverable? (Ad hoc exploration vs standardized reports vs dashboards?)
9. What concerns do you have about this initiative?
10. What has been your experience with prior data/reporting initiatives?

**Deliverables and Timing (5 minutes):**
11. What would a successful Phase 1 deliverable look like to you?
12. What's your timeline expectation?
13. Who else should we talk to in your organization?

**Wrap-Up (5 minutes):**
14. What are your success criteria for this project? (Measurable specifics)
15. How would you like to be involved going forward?
</executive_questions>

**Interview Duration**: 1 hour (executives are time-constrained)

**Key Focus**: Strategic direction, expectations, quantifiable business impact. Avoid diving into tactical details.

---

### Middle Management Interview Questionnaire

**Audience**: Directors, senior managers overseeing operational teams

<management_questions>
**Opening (5 minutes):**
- Introduction and context setting
- Interviewee's role, team size, and responsibilities

**Business Process Understanding (20 minutes):**
1. What are the key business processes your team manages? (Sales, procurement, inventory, customer service, etc.)
2. Walk me through a typical [process] from start to finish. What are the key steps?
3. What systems support these processes?
4. What are your key performance indicators (KPIs) for [process]?
5. How frequently do you track these KPIs? (Daily, weekly, monthly?)
6. What triggers action or concern when you review these metrics?

**Dimensionality and Context (15 minutes):**
7. How do you naturally group or categorize [products/customers/stores/regions]?
8. What are the important hierarchies for analyzing [dimension]? (e.g., Product: SKU → Brand → Category → Department)
9. How do you distinguish between different [products/customers/facilities/agents]? (What attributes matter?)
10. What date/time perspectives are important for your analysis? (Calendar vs fiscal? Day vs week vs month?)

**Current Analysis and Pain Points (15 minutes):**
11. Walk me through your most important reports and analyses. (Interviewee shares reports brought to meeting)
12. How do you use these analyses today? Who else uses them?
13. What are the biggest challenges with current reporting? (Data quality? Timeliness? Accessibility? Flexibility?)
14. What questions can't you answer today that you wish you could?
15. How much time does your team spend gathering data vs analyzing it?

**Improvement Opportunities (10 minutes):**
16. If you could improve one thing about data access, what would it be?
17. What would better information enable you to do differently?
18. What's the business impact? (Be specific with examples)

**Data Sources and Quality (10 minutes):**
19. Where does your data come from? (Source systems, manual entry, spreadsheets?)
20. What data quality issues do you encounter?
21. How do you handle data discrepancies between systems?

**Wrap-Up (5 minutes):**
22. Who else on your team should we talk to?
23. What does success look like for this initiative?
24. How would you like to be involved going forward?
</management_questions>

**Interview Duration**: 1.5 hours for individual; 1.5-2 hours for group of 2-3

**Key Focus**: Business processes, KPIs, dimensionality, current analyses, improvement opportunities

---

### Power Analyst / Data Steward Interview Questionnaire

**Audience**: Business analysts, report developers, data stewards, power users

<analyst_questions>
**Opening (5 minutes):**
- Introduction and context
- Interviewee's role and how they support business with data/reports

**Current Analytic Landscape (20 minutes):**
1. What reports and analyses do you create for the business?
2. Walk me through your top 5 most important analyses. (Examine reports brought to meeting)
3. How frequently are these generated? (Ad hoc vs scheduled?)
4. Who are the consumers of these analyses?
5. What tools do you use today? (Excel, SQL, reporting tools, BI tools?)

**Data Sources and Integration (20 minutes):**
6. What source systems do you pull data from?
7. Walk me through how you create [specific report]. (Understand data extraction, transformation, analysis flow)
8. How do you integrate data from multiple sources?
9. What data transformations or calculations do you perform?
10. How do you handle slowly changing data? (e.g., customer address changes, product reclassifications)

**Data Quality and Challenges (15 minutes):**
11. What data quality issues do you encounter most frequently?
12. Where are the data quality problems most severe?
13. How do you resolve data discrepancies?
14. What's your biggest frustration with current data access?
15. How much time do you spend on data gathering/cleansing vs actual analysis?

**Business Rules and Logic (15 minutes):**
16. What business rules do you apply when creating reports? (Definitions, calculations, filters)
17. How do you define [key metric]? (e.g., "active customer", "revenue", "on-time delivery")
18. Are there variations in how different groups define these metrics?
19. What are the most complex calculations or derivations you perform?

**User Community and Requirements (10 minutes):**
20. How do users request new reports or analyses?
21. What are the most common ad hoc requests you receive?
22. What analysis capabilities are users asking for that you can't currently provide?

**Improvement Opportunities (10 minutes):**
23. If you could design the ideal data/reporting environment, what would it include?
24. What would make your job significantly easier?
25. What capabilities would enable business users to be more self-sufficient?

**Wrap-Up (5 minutes):**
26. Can you share copies of key reports and data extracts? (Valuable for understanding dimensionality)
27. Would you be willing to participate in dimensional modeling sessions?
28. What does success look like for this initiative from your perspective?
</analyst_questions>

**Interview Duration**: 1.5 hours

**Key Focus**: Detailed data understanding, integration challenges, business rules, data quality, user community needs

**Value**: These individuals often have deepest understanding of source system realities and existing workarounds

---

### Source System Expert / IT Interview Questionnaire

**Audience**: DBAs, application developers, IT managers responsible for source systems

<it_questions>
**Opening (5 minutes):**
- Introduction and context
- Interviewee's role and systems supported

**Source System Landscape (15 minutes):**
1. What operational systems do you support?
2. What is the purpose and scope of each system? (Business processes supported)
3. What is the data volume in each system? (Row counts for major tables)
4. How frequently is data updated? (Real-time, batch, periodic?)
5. What is the data retention policy? (How much history is available?)

**Data Structures and Availability (20 minutes):**
6. What are the major data entities in [system]? (Customers, products, orders, transactions, etc.)
7. Is there data model documentation available?
8. Can you identify the key tables for [business process]?
9. What is the granularity of transaction data? (Line-item level? Header level?)
10. Is historical data available? How far back?

**Data Quality and Challenges (15 minutes):**
11. What are known data quality issues in [system]?
12. Which data elements are most problematic? (Nulls, inconsistencies, errors)
13. Are there data validation rules enforced by the application?
14. How do users correct data errors?
15. Are there audit trails or data lineage capabilities?

**Change Data Capture and Extraction (20 minutes):**
16. How would we identify changed/new records since last extraction?
   - Timestamp columns (created_date, modified_date)?
   - Database triggers or logs?
   - Application-level change tracking?
17. What is the feasibility of adding timestamp columns if they don't exist?
18. Are there database logs we could leverage for CDC?
19. What are the constraints on querying production databases? (Performance impact, access permissions)
20. What is the preferred extraction method? (Direct database query, API, file export, message queue?)

**Integration and Conformance (15 minutes):**
21. Are there overlapping entities across multiple source systems? (e.g., customer in CRM and ERP)
22. How are entities matched across systems today? (Common identifiers, matching logic)
23. What are known data inconsistencies between systems?
24. Which system is considered the "master" for each entity?

**Operational Considerations (10 minutes):**
25. What are the batch processing windows? (When can we extract without impacting operations?)
26. What are the security and access requirements for extracting data?
27. Who needs to approve DW/BI data extraction processes?
28. What support or accommodations are you willing to provide for DW/BI initiative?

**Wrap-Up (5 minutes):**
29. Can you provide data profiling results for key tables? (Domain values, null percentages, counts)
30. Can you provide sample data extracts for analysis?
31. Who else should we talk to about [system]?
</it_questions>

**Interview Duration**: 1.5 hours

**Key Focus**: Data availability, quality, extraction feasibility, change data capture, integration points

**Value**: Essential for understanding data realities and feasibility of supporting business requirements

---

## Interview Execution Best Practices

### Setting the Tone (First 5 Minutes)

<interview_launch>
**Pre-Scripted Introduction (Designated Kickoff Person):**

"Thank you for taking time to meet with us today. As [business sponsor] communicated, we're launching a data warehouse and business intelligence initiative to give the organization better access to information for decision making.

Our goal today is to understand how you and your team use data, what questions you're trying to answer, and where we can provide better support. We're here to learn from you about the business—you're the expert.

This is not about evaluating current systems or finding fault. We're focused on understanding your needs and opportunities for improvement.

We have about [1 hour/1.5 hours] scheduled. [Lead interviewer name] will be asking questions, [scribe name] will be taking notes, and [observer names] are here to learn about the business alongside us.

Everything we discuss is confidential within the project team. We'll be consolidating feedback from all interviews and will share back the overall findings for validation.

Do you have any questions before we begin?"

**What to Avoid:**
- ❌ Technical jargon (hardware, software, databases, ETL)
- ❌ Rambling about project history and challenges
- ❌ Committing to specific capabilities or timelines
- ❌ Defensive tone about existing systems

**What to Emphasize:**
- ✅ Business-centric message
- ✅ Learning from interviewee (they're the expert)
- ✅ Confidentiality and consolidation
- ✅ Opportunity for validation
</interview_launch>

---

### Active Listening and Probing Techniques

<listening_techniques>
**Open-Ended Questions:**
- ✅ "Tell me about..." / "Walk me through..." / "Describe..."
- ✅ "What are..." / "How do you..." / "Why do you..."
- ❌ "Do you..." / "Is it..." / "Can you..." (yes/no answers)

**The Five Whys (Probing Deeper):**
1. "We need better sales reporting."
   → "What's limiting sales reporting today?"
2. "Data comes from multiple systems and doesn't match."
   → "Why doesn't the data match?"
3. "Product codes are different in each system."
   → "Why are product codes different?"
4. "Systems were implemented at different times without coordination."
   → "Why is this a problem for sales reporting?"
5. "Sales team makes decisions based on incomplete picture, missing cross-sell opportunities."
   → **Root need identified**: Integrated product view enabling cross-sell analysis

**Active Listening Signals:**
- Maintain eye contact
- Nod and use verbal affirmations ("I see", "That makes sense", "Tell me more")
- Take notes but don't be buried in laptop (scribe handles detailed notes)
- Paraphrase back what you heard ("So what I'm hearing is...")
- Ask clarifying questions immediately

**Silence is Golden:**
- Don't fill every silence with next question
- Allow interviewee time to think
- Pauses often lead to interviewee volunteering additional valuable insights

**Follow the Energy:**
- When interviewee becomes animated about a topic, lean into it
- "You seem passionate about this—tell me more"
- Deviations from questionnaire often reveal important insights
</listening_techniques>

---

### Handling Difficult Interview Situations

<difficult_situations>
**Situation 1: Dominant Personality in Group Interview**
- **Challenge**: One person monopolizes discussion, others don't contribute
- **Technique**: "That's great input, thank you. [Other person's name], I'd like to hear your perspective on this as well."
- **Prevention**: Round-robin for specific questions to ensure everyone contributes

**Situation 2: Technical Deep Dive (Losing Business Focus)**
- **Challenge**: Interviewee diving into technical details of how systems work
- **Technique**: "That's helpful context. Let me bring us back to the business side—what decisions does this data help you make?"
- **Redirect**: Questionnaire's business-focused questions

**Situation 3: Vague or Generic Answers**
- **Challenge**: "We need better data" / "Reports should be easier" / "Everything should be faster"
- **Technique**: Ask for specific examples
  - "Can you show me an example of a report that's difficult today?"
  - "What does 'better data' mean specifically? More accurate? More timely? Different content?"
  - "How fast is acceptable? What's the current time and what's the target?"

**Situation 4: Hostile or Skeptical Interviewee**
- **Challenge**: "We've tried this before and it failed" / "Another IT boondoggle"
- **Technique**: Acknowledge and empathize
  - "I understand you've had disappointing experiences. What went wrong in the past?"
  - "What would need to be different this time for you to feel confident?"
  - "Your skepticism is valuable—it helps us avoid past mistakes. Tell me more."
- **Goal**: Turn skeptic into constructive critic

**Situation 5: "Crown Jewel" Reports Reluctantly Shared**
- **Challenge**: Interviewee initially reluctant to share competitive advantage reports
- **Pattern**: Almost always they enthusiastically race back to office at end of interview to retrieve reports
- **Technique**:
  - Don't pressure early in interview
  - Build trust and rapport throughout session
  - At end: "These reports you mentioned sound really valuable for understanding requirements. Would you be willing to share anonymized examples?"
  - Assure confidentiality within project team

**Situation 6: "Everything is Fine" (No Requirements)**
- **Challenge**: Interviewee says current reporting meets all needs
- **Technique**:
  - "That's great to hear. What reports do you use most?"
  - "Walk me through your most recent analysis—what steps were involved?"
  - "If you could wave a magic wand, what would you improve?"
  - Often process of walking through workflow reveals pain points they've normalized
</difficult_situations>

---

## Post-Interview Activities

### Immediate Debrief (Within 30 Minutes of Interview)

<debrief_process>
**Debrief Attendees**: Entire interview team (interviewer, scribe, observers)

**Debrief Agenda (15-20 minutes):**
1. **Alignment Check**: Are we on same page about what was learned?
2. **Key Takeaways**: What are the 3-5 most important insights?
3. **Business Processes Identified**: What processes did we hear about?
4. **Dimensions and Facts**: What dimensions and facts mentioned?
5. **Data Quality Concerns**: What data issues surfaced?
6. **Feasibility Concerns**: Any red flags about data availability?
7. **Follow-Up Actions**: Who do we need to talk to next? What clarifications needed?
8. **Questionnaire Refinement**: What questions worked well? What should we add/remove/revise?

**Capture Decisions**: Scribe documents debrief insights immediately
</debrief_process>

---

### Notes Cleanup and Filing

<notes_management>
**Timing**: Within 24-48 hours of interview (while still fresh)

**Activities:**
1. **Review and Fill Gaps**: Scribe reviews notes, fills in abbreviations and partial sentences
   - "Cust segmt by RFM" → "Customer segmentation by Recency, Frequency, Monetary value"
   - Abbreviations become incomprehensible after a few days

2. **Organize Notes**: Structure notes by topic (business process, KPIs, dimensions, pain points, etc.)

3. **Examine Collected Reports**:
   - Study reports to understand dimensionality
   - Identify metrics (facts)
   - Identify groupings and hierarchies (dimensions)
   - Note calculated fields and business rules

4. **File and Share**:
   - Store notes in shared team folder (organized by interviewee name/role)
   - Share reports folder with modeling team
   - Update interview tracking spreadsheet with completion status
</notes_management>

---

## Consolidated Findings Document

### Document Structure

<findings_document>
**1. Executive Summary (1-2 pages)**
- DW/BI initiative overview and objectives
- Scope of requirements gathering (who, when, how many interviews)
- Key findings at high level
- Recommended priorities (preview of detailed prioritization)

**2. Project Overview (1 page)**
- Requirements process description (interviews, facilitation, etc.)
- Participants list (interviewed stakeholders)
- Timeline of activities

**3. Business Processes (Bulk of Document)**

**For each business process identified:**

<process_section>
**Process Name**: [e.g., Retail Sales]

**Why Analyze This Process:**
- Business drivers and motivations
- Strategic importance
- Decisions supported by analysis of this process

**What Capabilities Are Wanted:**
- Key performance indicators (KPIs) to track
- Dimensions for analysis (customer, product, time, geography, etc.)
- Required hierarchies and drill-down paths
- Desired frequency (daily, weekly, monthly reporting)
- Ad hoc vs standardized reporting needs
- Integration with other processes (drill-across requirements)

**Current Limitations:**
- Data quality issues
- Timeliness problems (data latency)
- Accessibility challenges (who has access, how difficult to obtain)
- Flexibility constraints (can't answer certain questions)
- Integration gaps (data silos, inconsistent definitions)

**Potential Benefits and Impact:**
- Quantified business value (revenue increase, cost reduction, risk mitigation)
- Specific examples of improved decision making
- Stakeholder testimonials and quotes
- Estimated time savings from improved data access

**Data Feasibility:**
- Source systems available
- Data quality assessment (from data-centric interviews)
- Change data capture feasibility
- Integration challenges across sources
- Historical data availability
- Known data quality issues and remediations
- Preliminary complexity assessment (straightforward, moderate, complex)
</process_section>

**4. Opportunity/Stakeholder Matrix (Visual)**

Matrix showing:
- **Rows**: Business processes
- **Columns**: Organizational groups/functions
- **Cells**: Impact level (high, medium, low) or checkmarks

**Example:**

| Business Process | Sales | Marketing | Finance | Operations | Supply Chain |
|------------------|-------|-----------|---------|------------|--------------|
| Retail Sales | ✓✓✓ | ✓✓ | ✓✓✓ | ✓ | ✓ |
| Inventory Management | ✓ | | ✓✓ | ✓✓✓ | ✓✓✓ |
| Customer Acquisition | ✓✓ | ✓✓✓ | ✓ | | |
| Procurement | | | ✓✓ | ✓✓ | ✓✓✓ |
| Order Fulfillment | ✓✓✓ | | ✓ | ✓✓✓ | ✓✓ |

**Insight**: Shows which processes have broad organizational impact (dense rows = high priority candidates)

**5. Simplified Bus Matrix (Preview)**

Early-stage bus matrix showing:
- **Rows**: Business processes (same as opportunity matrix)
- **Columns**: Preliminary conformed dimensions identified

**Note**: This is preliminary—detailed bus matrix developed during dimensional modeling

**6. Next Steps**
- Prioritization approach (see next section)
- Proposed timeline for Phase 1 project
- Stakeholder involvement going forward
</findings_document>

**Deliverable**: Consolidated findings document (typically 20-40 pages)

---

## Requirements Validation and Prioritization

### Findings Review and Prioritization Meeting

<prioritization_meeting>
**Attendees:**
- Senior business representatives (ideally those interviewed)
- DW/BI manager and senior IT management
- Business sponsor (critical for decision authority)
- Project manager (facilitator)

**Meeting Duration**: 2-3 hours

**Meeting Agenda:**

**1. Welcome and Context (10 minutes)**
- Project overview and objectives
- Requirements gathering process recap
- Purpose of today's session: Review findings and establish priorities

**2. Findings Presentation (60-90 minutes)**
- Overview of EACH identified business process (5-10 minutes per process)
- Use process sections from consolidated findings document
- Ensure common understanding across all attendees

**3. Opportunity/Stakeholder Matrix Review (10 minutes)**
- Visual representation of impact across organization
- Discussion of broad vs narrow organizational interest

**4. Simplified Bus Matrix Review (10 minutes)**
- Enterprise data architecture preview
- Conformed dimensions identified across processes

**5. BREAK (10 minutes)**

**6. Prioritization Exercise (45-60 minutes)**
- Introduce prioritization grid (impact vs feasibility)
- Place each business process on grid based on group consensus
- Discuss and debate placement
- Identify Phase 1 candidates (upper-right quadrant)
</prioritization_meeting>

---

### Prioritization Grid (Impact vs Feasibility)

<prioritization_grid>
**Two-Dimensional Assessment:**

**Vertical Axis: Business Impact/Value**
- High Impact: Large revenue opportunity, significant cost reduction, critical strategic importance, broad organizational benefit
- Medium Impact: Moderate business value, departmental benefit
- Low Impact: Limited business value, nice-to-have

**Horizontal Axis: Feasibility**
- High Feasibility: Data available, good quality, straightforward integration, single source system, team expertise exists
- Medium Feasibility: Some data quality issues, moderate integration complexity, multiple sources but manageable
- Low Feasibility: Significant data quality problems, complex integration, data not currently collected, requires major source system changes

**Four Quadrants:**

```
High Impact
    ^
    |
  2 | 1
----|-----> High Feasibility
  3 | 4
    |
```

**Quadrant 1 (Upper-Right): START HERE**
- High impact AND highly feasible
- **Characteristics**: Meaningful business value, data available and reasonable quality, manageable complexity
- **Action**: Phase 1 immediate attention
- **Example**: Retail sales analysis (high business value, POS data available and good quality)

**Quadrant 2 (Upper-Left): MEANINGFUL BUT NOT FEASIBLE YET**
- High impact BUT low feasibility
- **Characteristics**: Large business payback but currently infeasible (data quality, integration complexity, political challenges)
- **Action**: Other IT teams address feasibility limitations (data quality remediation, MDM, source system improvements) while DW/BI focuses on Quadrant 1
- **Example**: 360-degree customer view (high value but requires MDM across 12 systems)

**Quadrant 3 (Lower-Left): AVOID**
- Low impact AND low feasibility
- **Characteristics**: Missions impossible—little business value and very difficult
- **Action**: Avoid like the plague, never prioritize
- **Example**: Analyzing rarely-used legacy product line with poor data quality

**Quadrant 4 (Lower-Right): DON'T PRIORITIZE SHORT-TERM**
- Low impact BUT high feasibility
- **Characteristics**: Doable but not very crucial
- **Action**: Project teams sometimes gravitate here because "easy wins", but doesn't move business needle. Defer.
- **Example**: Departmental report automation with limited organizational value
</prioritization_grid>

**Prioritization Grid Facilitation:**

1. **Pre-populate grid** with each business process on a sticky note
2. **Project team places** sticky notes on projected grid based on findings
3. **Group discussion and adjustment**: Attendees debate and move sticky notes until consensus
4. **Take photo** of final grid for documentation
5. **Extract Phase 1 candidates** from Quadrant 1 (upper-right)

**Typical Outcome**:
- Quadrant 1: 2-4 processes → Phase 1 candidates (select 1 for first iteration)
- Quadrant 2: 3-6 processes → Future phases (after feasibility addressed)
- Quadrant 3: 1-3 processes → Never prioritize
- Quadrant 4: 2-4 processes → Defer

---

## Requirements Documentation Best Practices

<documentation_best_practices>
**Clarity and Conciseness:**
- Write for audience who wasn't in interviews
- Use business terminology (not technical jargon)
- Include concrete examples and quotes from interviewees
- Avoid abbreviations unless spelled out first

**Visuals:**
- Include diagrams (opportunity matrix, bus matrix, prioritization grid)
- Screenshots of example reports discussed
- Tables for comparing processes or stakeholder perspectives

**Traceability:**
- Link business requirements to architectural implications
- Cite specific interviewees for controversial or critical points
- Preserve dissenting opinions (don't sanitize disagreements)

**Actionability:**
- Clear next steps and recommendations
- Quantify when possible (number of users, data volumes, response time needs)
- Identify owners for follow-up actions

**Validation:**
- Circulate draft findings to key interviewees for review
- Incorporate feedback before final presentation
- Builds credibility and buy-in
</documentation_best_practices>

---

## Common Requirements Gathering Mistakes

<requirements_mistakes>
**1. Technology-Driven Requirements**
- **Mistake**: Asking "What reports do you want from data warehouse?"
- **Fix**: Ask "What decisions do you make?" and "What questions do you need answered?"

**2. Skipping Requirements (Assuming You Know)**
- **Mistake**: "We already know what users need, let's just build it"
- **Fix**: Always validate assumptions with business—you'll learn new things

**3. Only Talking to Power Analysts (Ignoring Executives)**
- **Mistake**: Gravitate toward tactical analysts, miss strategic direction
- **Fix**: Cover organization vertically (executives, management, analysts)

**4. Only Talking to Single Department (Missing Conformed Dimensions)**
- **Mistake**: Focus narrowly on initial department, miss enterprise perspective
- **Fix**: Cover organization horizontally to identify common data and vocabulary

**5. Asking Leading Questions**
- **Mistake**: "Wouldn't it be great if you could analyze by customer segment?"
- **Fix**: Open-ended questions letting user describe their needs

**6. Designing in Interview Room**
- **Mistake**: "So you need a Customer dimension with SCD Type 2 for address changes"
- **Fix**: Listen and learn in interviews, design in modeling sessions later

**7. Over-Promising in Interviews**
- **Mistake**: "We'll definitely include that in Phase 1"
- **Fix**: "We'll capture that requirement and prioritize with the team"

**8. Under-Documenting**
- **Mistake**: Brief summary notes, missing nuances and examples
- **Fix**: Copious notes, consolidate into structured findings document

**9. Ignoring Data Feasibility**
- **Mistake**: Collecting business "wish list" without validating data exists
- **Fix**: Intersperse data-centric interviews with source system experts

**10. No Validation Loop**
- **Mistake**: Document requirements, never show back to interviewees
- **Fix**: Review consolidated findings with business for validation and corrections
</requirements_mistakes>

---

## Requirements Gathering Checklist

### Pre-Planning
- [ ] Requirements forum selected (interviews, facilitation, or hybrid)
- [ ] Interview team identified (lead, scribe, observers)
- [ ] Interview questionnaires drafted for each audience (executives, management, analysts, IT)
- [ ] Interview team prepared (research, homework, mindset)
- [ ] Business representatives identified (horizontal and vertical coverage)
- [ ] Executive sponsor communication sent to interviewees
- [ ] Interviews scheduled with buffer time between sessions
- [ ] Logistics confirmed (rooms, laptops, materials)

### Execution
- [ ] Introduction scripted and practiced
- [ ] Interviews conducted with active listening and probing
- [ ] Notes taken by scribe (not lead interviewer)
- [ ] Reports and examples collected from interviewees
- [ ] Immediate debrief after each interview
- [ ] Notes cleaned up within 24-48 hours
- [ ] Collected reports examined and filed
- [ ] Data-centric interviews interspersed to validate feasibility

### Documentation
- [ ] Consolidated findings document created (executive summary, process sections, matrices)
- [ ] Opportunity/stakeholder matrix developed
- [ ] Simplified bus matrix drafted
- [ ] Findings document reviewed by interview team
- [ ] Draft circulated to key interviewees for validation
- [ ] Feedback incorporated into final document

### Prioritization
- [ ] Findings review and prioritization meeting scheduled
- [ ] Senior business representatives and sponsors invited
- [ ] Findings presentation prepared (process overview, matrices)
- [ ] Prioritization grid facilitated (impact vs feasibility)
- [ ] Phase 1 candidates identified (Quadrant 1: high impact, high feasibility)
- [ ] Prioritized business processes documented
- [ ] Next steps and timeline communicated

---

## Summary

Business requirements gathering is collaborative, iterative process:
- **Forum**: Hybrid approach (interviews for depth + facilitation for consensus)
- **Team**: Lead interviewer + scribe + observers
- **Preparation**: Research, questionnaires, proper mindset (you're there to learn)
- **Execution**: Open-ended questions, active listening, probing deeper
- **Documentation**: Consolidated findings organized by business process
- **Validation**: Review with interviewees, incorporate feedback
- **Prioritization**: Impact vs feasibility grid, identify Phase 1 candidates

**Critical Success Factors:**
- Business user involvement (early and often)
- Horizontal and vertical organizational coverage
- Focus on business needs (not technology)
- Data feasibility validation (intersperse IT interviews)
- Clear documentation with traceability
- Business-driven prioritization (not IT-driven)

**Deliverables:**
- Interview notes and collected reports
- Consolidated findings document (20-40 pages)
- Opportunity/stakeholder matrix
- Simplified bus matrix
- Prioritized business processes with Phase 1 recommendation

**Time Investment**: 4-6 weeks for comprehensive requirements gathering with 15-25 interviews.
