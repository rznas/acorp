# Business Process Prioritization Workshop Guide

This guide provides step-by-step facilitation instructions for conducting the prioritization workshop where business processes identified during requirements gathering are ranked by impact and feasibility to determine DW/BI implementation sequence.

## Workshop Purpose

Following requirements gathering and bus matrix creation, this workshop brings together senior business and IT leadership to:
1. Build common understanding of identified business processes
2. Assess each process by business impact and technical feasibility
3. Prioritize processes on the impact/feasibility grid
4. Agree on implementation sequence for DW/BI projects

**Critical Success Factor**: This is a **collaborative business decision**, not an IT decision made in isolation.

---

## Pre-Workshop Preparation (2-3 Weeks Before)

### Deliverables Required

You must complete these artifacts before scheduling the prioritization workshop:

1. **Consolidated Requirements Findings Document** (20-40 pages)
   - Executive summary of key themes
   - Business process descriptions with requirements
   - Data feasibility assessment for each process
   - Potential business benefits quantified

2. **Enterprise Data Warehouse Bus Matrix** (preliminary)
   - Business processes as rows
   - Common dimensions as columns
   - Cells marked showing dimension usage

3. **Opportunity/Stakeholder Matrix**
   - Business processes as rows
   - Organizational functions as columns
   - Shows cross-functional interest

### Scheduling and Logistics

**Participant Selection**:
- **Required**: Business sponsor/executive committee (decision authority)
- **Required**: Senior business representatives who participated in requirements interviews
- **Required**: DW/BI project manager
- **Required**: Senior IT management (CIO or delegate)
- **Optional**: Data governance representatives
- **Avoid**: Too many participants (>12 becomes unwieldy)

**Workshop Duration**: 2-3 hours (half-day maximum)

**Room Setup**:
- U-shaped or conference table seating
- Projector/large screen for presentations
- Whiteboard or flip charts for capturing discussion
- Printouts of matrices for each participant

**Materials to Prepare**:
- Slide deck (20-30 slides maximum)
- Printed findings document (distribute 1 week before)
- Printed bus matrix (one per participant)
- Printed opportunity matrix (one per participant)
- Large poster-sized prioritization grid
- Sticky notes (different colors for each process)
- Markers for whiteboard/flip charts

**Pre-Read Distribution** (1 week before):
Send participants:
- Executive summary from findings document
- Simplified bus matrix
- Workshop agenda with objectives
- Request: Review materials before attending

---

## Workshop Agenda Template

**Total Time**: 2.5 hours

```
09:00-09:15  Welcome and Context Setting
09:15-09:45  Requirements Findings Overview
09:45-10:15  Business Process Deep Dive
10:15-10:30  Break
10:30-10:45  Opportunity/Stakeholder Matrix Review
10:45-11:15  Prioritization Exercise
11:15-11:30  Next Steps and Commitment
```

---

## Part 1: Welcome and Context Setting (15 minutes)

### Facilitator: Business Sponsor

**Objectives**:
- Set tone: This is business-driven, business decision
- Establish meeting purpose and desired outcomes
- Reinforce importance of cross-functional collaboration

### Script Template

> "Thank you all for joining today. Over the past [X weeks], our team has conducted extensive interviews across [Y departments] to understand our DW/BI needs and opportunities.
>
> Today's purpose is to **collectively decide** which business processes we'll tackle first in our data warehouse implementation. This is not a technical decision—it's a **strategic business decision** about where we'll focus resources to deliver maximum value.
>
> We'll review what we learned, discuss feasibility realities, and together prioritize our roadmap. I'm counting on your candid input and collaborative spirit. By the end of this session, we need agreement on our starting point.
>
> I'll turn it over to [Project Manager] to walk us through the findings."

### Facilitator Tips

- **Keep it brief** - Don't let sponsor ramble for 30 minutes
- **Set collaborative tone** - This is discussion, not presentation to executives
- **Emphasize outcomes** - Must leave with prioritized list

---

## Part 2: Requirements Findings Overview (30 minutes)

### Facilitator: DW/BI Project Manager or Business Analyst

**Objectives**:
- Present consolidated findings at executive level (not overwhelming detail)
- Ensure common understanding of business processes identified
- Surface any missing processes or misunderstandings early

### Presentation Structure

#### Slide 1-2: Requirements Process Recap
- **Who we talked to**: List interview participants by function (not names)
  - "15 executives across Finance, Sales, Marketing, Operations, Supply Chain"
- **What we asked**: High-level question themes
  - "Key performance metrics, decision-making challenges, analytic needs"
- **Data sources reviewed**: Systems evaluated for feasibility
  - "ERP, CRM, POS, supply chain, HR systems"

#### Slide 3-4: Key Themes Discovered
Present 3-5 major analytic themes that cut across interviews:
```
Theme 1: Inability to understand true customer profitability
  • Sales sees revenue, Finance sees costs, no integrated view
  • Different customer categorizations prevent comparison
  • Opportunity: $2-5M annual margin improvement via better pricing

Theme 2: Inventory optimization challenges
  • Out-of-stocks cost $500K monthly in lost sales
  • Excess inventory ties up $8M in working capital
  • No visibility to inventory across distribution network

[Continue with remaining themes...]
```

**Facilitation Tips**:
- Keep themes **business-focused**, not data-focused
- **Quantify impact** where possible (revenue opportunity, cost savings, time savings)
- Show **cross-functional nature** - multiple groups interested

#### Slide 5-12: Business Process Summaries (5-7 minutes each)

For each identified business process, present:

**Process Name**: [E.g., "Retail Store Sales"]

**Description**: What operational activity this represents (1-2 sentences)

**Business Questions This Enables** (3-5 examples):
- "Which products are selling in which stores?"
- "How effective are promotions in driving incremental sales?"
- "Which customer segments are most profitable?"

**Who Wants This** (from opportunity matrix):
- Merchandising, Marketing, Store Operations, Finance (show breadth)

**Sample Metrics Available**:
- Sales quantity, sales amount, gross margin, units per transaction

**Data Feasibility Assessment**:
- ✅ **Good**: Daily POS transaction data available, 3 years history
- ⚠️ **Challenge**: Customer identification only for loyalty program members (40% of transactions)
- ✅ **Good**: Product, store, promotion data well-structured

**Potential Business Value**:
- $3-5M annual opportunity via markdown optimization
- Reduce out-of-stocks by 20% ($500K monthly revenue recovery)

**Estimated Complexity**: [High / Medium / Low]

---

**Repeat this structure for each business process** (Recommend limiting to 6-8 processes for first workshop)

---

#### Facilitation Techniques

**Encourage Questions**:
> "Before we move to the next process, any questions or clarifications on [Process Name]?"

**Validate Understanding**:
> "Does this match your understanding of the [Sales/Operations/Finance] challenges?"

**Surface Disagreements Early**:
> "I see some heads shaking—what are we missing?"

**Capture Additions**:
> "Great point. Let me add that to the process description. Any other business questions we should be answering?"

**Watch the Clock**: Allocate time proportionally (8 processes = ~5 minutes each)

---

## Part 3: Break (15 minutes)

**Purpose**: Let participants process information, have side conversations, network

**Facilitator Actions During Break**:
- Note any concerns surfaced in hallway conversations
- Adjust presentation if timing running long
- Set up prioritization grid materials

---

## Part 4: Opportunity/Stakeholder Matrix Review (15 minutes)

### Facilitator: Project Manager

**Objective**: Visually demonstrate cross-functional nature of identified processes

### Presentation

**Display Opportunity/Stakeholder Matrix** (large format on screen):

```
Business Process        | Merch | Marketing | Finance | Ops | Supply Chain
------------------------|-------|-----------|---------|-----|-------------
Retail Store Sales      |   X   |     X     |    X    |  X  |
Store Inventory         |   X   |           |    X    |  X  |      X
Purchase Orders         |   X   |           |    X    |     |      X
Product Returns         |   X   |     X     |    X    |  X  |
Promotion Analysis      |   X   |     X     |         |     |
Customer Behavior       |   X   |     X     |    X    |     |
```

**Key Talking Points**:

> "This matrix shows which business functions are interested in which processes. Notice that **Retail Store Sales** has interest from five different departments. This indicates high cross-functional value.
>
> Conversely, some processes have narrower stakeholder bases. That doesn't make them unimportant, but it factors into our prioritization discussion.
>
> The breadth of interest often correlates with potential business impact—more stakeholders typically means broader organizational benefit."

**Discussion Prompts**:
- "Do these stakeholder mappings look accurate?"
- "Any surprises about who's interested in which processes?"
- "Where do you see potential conflicts between departmental needs?"

---

## Part 5: Prioritization Exercise (30 minutes)

### Facilitator: Project Manager with Business Sponsor Support

This is the **critical section** where decisions are made.

### Step 1: Introduce the Prioritization Grid (5 minutes)

**Display Large Prioritization Grid**:

```
                         HIGH IMPACT

        Avoid for Now              START HERE!
        (High Impact              (High Impact
        Low Feasibility)          High Feasibility)
              ↑                         ↑
              |                         |
   -----------|-------------------------|------------
              |                         |
              ↓                         ↓
        Avoid                    Easy Wins
        (Low Impact              (Low Impact
        Low Feasibility)         High Feasibility)

                         LOW IMPACT

        ← LOW FEASIBILITY    HIGH FEASIBILITY →
```

**Explain the Dimensions**:

> "We'll evaluate each business process on two dimensions:
>
> **BUSINESS IMPACT** (Vertical Axis):
> - Potential financial benefit (revenue, cost savings, margin improvement)
> - Strategic importance to organization
> - Number of stakeholders/users affected
> - Urgency of business need
> - Potential for competitive advantage
>
> **TECHNICAL FEASIBILITY** (Horizontal Axis):
> - Data availability and quality
> - Source system accessibility
> - Technical complexity
> - Political/organizational challenges
> - Team skills and experience
> - Resource availability"

**Explain the Quadrants**:

> "**Upper Right (High Impact, High Feasibility)**: These are our **top priorities**. High business value and achievable—start here.
>
> **Upper Left (High Impact, Low Feasibility)**: Important but challenging. We can't tackle these first, but other IT teams should address feasibility barriers.
>
> **Lower Right (Low Impact, High Feasibility)**: Easy wins but low value. Might make sense as quick wins but don't justify major investment.
>
> **Lower Left (Low Impact, Low Feasibility)**: Avoid. Neither important nor feasible—not worth pursuing."

### Step 2: Establish Evaluation Criteria (5 minutes)

**Impact Criteria Quick Reference**:

Write on whiteboard/flip chart:

```
HIGH IMPACT:
• >$1M annual financial benefit
• Addresses top 3 strategic initiatives
• Supports >50 power users + hundreds of casual users
• Urgent competitive/regulatory driver

MEDIUM IMPACT:
• $250K-$1M annual benefit
• Addresses important but not top-tier priorities
• Supports 20-50 power users
• Valuable but not time-critical

LOW IMPACT:
• <$250K annual benefit
• Addresses nice-to-have needs
• Supports <20 users
• No compelling urgency
```

**Feasibility Criteria Quick Reference**:

```
HIGH FEASIBILITY:
• Clean, accessible data in operational systems
• Single, well-structured source system
• Low technical complexity
• Strong data governance support
• Team has skills/experience
• No major political obstacles

MEDIUM FEASIBILITY:
• Data exists but requires cleansing
• Multiple source systems requiring integration
• Moderate technical complexity
• Some governance challenges
• Team can learn required skills
• Minor political/organizational challenges

LOW FEASIBILITY:
• Poor data quality or unavailable data
• Complex integration across many systems
• High technical complexity (real-time, big data)
• Significant governance/political challenges
• Major skill gaps
• Organizational resistance
```

**Facilitator Note**: Adapt dollar amounts to your organization's scale.

### Step 3: Collaborative Placement Exercise (15 minutes)

**Approach 1: Physical Grid with Sticky Notes (Recommended)**

Materials:
- Large poster-sized prioritization grid (4-6 feet wide)
- Sticky notes pre-labeled with each business process name
- Different colored sticky notes for extra visual clarity

**Instructions to Group**:

> "We're going to place each business process on this grid together. For each process, I'll facilitate a quick discussion on impact and feasibility, then we'll place it on the grid.
>
> This is **consensus-based**, not voting. We need to talk through disagreements, not majority-rule them. The sponsor has final say if we can't reach consensus.
>
> Let's start with [Process Name]..."

**For Each Process** (2-3 minutes):

1. **Read process name and brief description**

2. **Discuss Impact**:
   - "What's the potential financial benefit?"
   - "How many users/stakeholders?"
   - "How strategic is this?"
   - Tentative impact level: High / Medium / Low

3. **Discuss Feasibility**:
   - "What's the data situation?"
   - "What's the technical complexity?"
   - "Any political/organizational challenges?"
   - Tentative feasibility level: High / Medium / Low

4. **Place on Grid**:
   - Position sticky note in appropriate quadrant
   - Exact placement within quadrant less important (High/Medium/Low sufficient)

5. **Check for Consensus**:
   - "Any strong disagreements with this placement?"
   - Resolve disagreements through discussion
   - Sponsor arbitrates if needed

**Example Facilitation**:

> **Facilitator**: "Okay, next is **Retail Store Sales**. From our findings, this has potential for $3-5M annual benefit from markdown optimization plus $500K monthly from reducing out-of-stocks. Five departments are interested. Data is readily available from POS systems with 3 years history. Main challenge is incomplete customer identification—only 40% of transactions.
>
> Thoughts on **impact**—high, medium, or low?"
>
> **CFO**: "This is definitely high impact. Those are real dollars."
>
> **VP Marketing**: "Agreed. We've been flying blind on promotion effectiveness."
>
> **Facilitator**: "Consensus on high impact. Now **feasibility**—high, medium, or low?"
>
> **IT Director**: "Data is good. POS system is clean. Customer gap is manageable—we can start with the 40% we can identify. I'd say high feasibility."
>
> **VP Merchandising**: "But won't we need the product dimension built first? Isn't that a dependency?"
>
> **Project Manager**: "Yes, but product data is in great shape. That dimension is straightforward."
>
> **Facilitator**: "So high feasibility overall? [Wait for nods] Great. That puts **Retail Store Sales** in the **upper-right quadrant**—high impact, high feasibility. This is a top candidate." [Places sticky note]

**Facilitation Tips**:

- **Keep momentum** - Don't let one process consume 15 minutes
- **Defer deep dives** - "Let's capture that for detailed planning later"
- **Acknowledge uncertainty** - "We're making best estimates, not perfect assessments"
- **Document assumptions** - Note key assumptions made for each process
- **Watch for clustering** - Multiple processes in same quadrant is common

**Approach 2: Slide-Based Presentation (If Physical Not Possible)**

Show pre-positioned processes on digital prioritization grid. Invite discussion and adjustments.

Less engaging but viable for remote workshops.

### Step 4: Review and Adjust Placements (5 minutes)

**Step Back and Review**:

> "Let's look at the overall landscape. We have [X] processes in the high-impact, high-feasibility quadrant—our top priorities. We have [Y] in the upper-left that are important but currently challenging. And we have [Z] that are easy but lower value.
>
> Before we finalize, any processes that feel misplaced?"

**Common Adjustments**:
- Swap two processes in same quadrant (fine-tuning)
- Move process from Medium to High (or vice versa) based on new information
- Combine two similar processes into one (consolidation)

**Check for Consensus**:
> "Can everyone live with these placements as the basis for our implementation sequence?"

---

## Part 6: Implementation Sequence Discussion (15 minutes)

### Facilitator: Project Manager with Sponsor

**Objective**: Convert the prioritization grid into a sequenced roadmap.

### Step 1: Identify "First Project" Candidates (5 minutes)

**Focus on Upper-Right Quadrant Only**:

> "These processes in the upper-right are all strong candidates. Typically, we recommend starting with a **single business process** to minimize risk and deliver value quickly.
>
> From this set, which should we tackle first?"

**Evaluation Criteria for First Project**:
- **Meaningful**: Significant business value (justify investment)
- **Manageable**: Team can deliver in realistic timeframe (3-6 months)
- **Foundation**: Builds conformed dimensions reusable for future projects
- **Sponsored**: Has committed executive sponsor

**Discussion Prompts**:
- "Which process, if delivered, would generate the most excitement?"
- "Which process has the strongest executive champion?"
- "Are there dependencies between these processes?"
- "Which builds foundational dimensions we'll reuse?"

### Step 2: Sequence Remaining Projects (5 minutes)

**Identify Second and Third Projects**:

> "After we deliver the first project successfully, which process should be second? Third?"

**Sequencing Considerations**:
- **Dimension reuse**: Leverage conformed dimensions from earlier projects
- **Business momentum**: Balance different stakeholder groups
- **Complexity progression**: Build team capability incrementally
- **Dependency management**: Sequence based on data/dimension dependencies

**Example Sequencing Logic**:

> "If we do **Retail Store Sales** first, we'll build Product, Store, Date, Customer, and Promotion dimensions. Those dimensions can be reused for **Store Inventory** and **Product Returns**, making those faster second and third projects. That creates quick momentum."

### Step 3: Address Upper-Left Quadrant (5 minutes)

**High-Impact, Low-Feasibility Processes**:

> "These processes are important but currently infeasible. What do we need to do to move them to the right?
>
> - Is it a data quality issue? → Data governance initiative
> - Is it a source system issue? → System upgrade/replacement
> - Is it a technical skills issue? → Training or hiring
> - Is it a political issue? → Executive alignment"

**Assign Accountable Parties**:
- IT addresses technical barriers
- Business addresses organizational barriers
- Revisit feasibility quarterly

**Document Blockers**:
Capture specific barriers for each infeasible-but-important process.

---

## Part 7: Next Steps and Commitment (15 minutes)

### Facilitator: Business Sponsor

**Objective**: Secure commitment, outline next steps, set expectations.

### Commitment From Sponsor

**Script Template**:

> "Thank you all for this collaborative session. We have consensus that our first DW/BI project will focus on **[Process Name]**.
>
> I want to be clear about what this means:
>
> **Resources**: We're committing [X] IT resources plus [Y] business resources for the next [Z] months.
>
> **Scope**: We're building a dimensional model for [Process], supporting approximately [N] users, with data refreshed [daily/weekly].
>
> **Timeline**: We're targeting [Date] for initial delivery. This is aggressive but achievable.
>
> **Involvement**: I need continued engagement from [Department leads]. This succeeds only with your partnership.
>
> **Next Steps**: [Project Manager] will schedule kickoff sessions for dimensional modeling workshops. Plan for [X] hours over the next [Y] weeks.
>
> Any questions or concerns before we commit?"

### Outline Next Steps

**Project Manager Presents**:

```
Next 2 Weeks:
• Finalize project charter and resource assignments
• Schedule dimensional modeling workshops
• Set up project infrastructure (tools, environments)

Weeks 3-6:
• Conduct collaborative dimensional modeling workshops
• Design dimension and fact tables
• Begin physical database design
• Architecture and product selection (parallel track)

Weeks 7-12:
• ETL development
• BI application development
• Testing and quality assurance

Weeks 13-16:
• Deployment preparation
• User training
• Production deployment
• Hypercare support

Week 17+:
• Return to prioritization grid for Project 2
```

### Set Expectations

**Project Manager Communicates**:

> "Key points about what's ahead:
>
> **Business Involvement Required**: Dimensional modeling workshops need your active participation—not delegation to junior staff.
>
> **Iterative Development**: You'll see progress incrementally. We'll show working prototypes, get your feedback, adjust.
>
> **Change is Normal**: Requirements will evolve as you see the data. We'll manage scope changes collaboratively.
>
> **Deployment Readiness**: We won't deploy until YOU tell us the data and applications are right.
>
> **Long-term Program**: This first project is the beginning. We'll iterate through the prioritization grid systematically, building an integrated enterprise DW/BI environment."

### Closing

**Sponsor Closes**:

> "I'm excited about the roadmap we've defined together. This represents significant investment in our decision-making capabilities.
>
> Thank you for your time and thoughtful input today. Let's make this first project a success that generates momentum for the entire program."

---

## Post-Workshop Actions

### Immediate (Within 1 Week)

**Document Outcomes**:
- [ ] Update bus matrix based on workshop discussions
- [ ] Document final prioritization grid (take photo of physical grid + digital version)
- [ ] Capture implementation sequence with rationale
- [ ] List barriers to infeasible-but-important processes with owners
- [ ] Create project charter for first project

**Communicate Results**:
- [ ] Send prioritization summary to all workshop participants
- [ ] Communicate decision to broader organization (intranet, email, town hall)
- [ ] Schedule kickoff meeting for first project
- [ ] Notify participants who'll be involved in dimensional modeling workshops

### Follow-Up (Weeks 2-4)

- [ ] Schedule dimensional modeling workshops
- [ ] Assign business representatives to modeling sessions
- [ ] Finalize project resource commitments
- [ ] Begin architecture and product selection activities (parallel track)

---

## Troubleshooting Common Challenges

### Challenge 1: Executive Doesn't Show Up

**Impact**: Workshop loses authority and decision-making power.

**Prevention**:
- Get sponsor commitment 2+ weeks in advance
- Send multiple calendar holds and reminders
- Emphasize this is THE prioritization decision point

**If It Happens**:
- Proceed but frame as "recommendations to sponsor" not final decisions
- Schedule follow-up with sponsor to validate/adjust
- Risk: Delays project by 1-2 weeks

### Challenge 2: Too Many Processes to Discuss

**Symptom**: 12+ business processes identified, can't cover all in 2.5 hours.

**Solution**:
- **Pre-filter**: Eliminate obvious low-impact, low-feasibility processes before workshop
- **Batch processes**: Group similar processes (e.g., "All procurement processes")
- **Two workshops**: Split into separate sessions by business area

### Challenge 3: Analysis Paralysis

**Symptom**: Group debates exact placement endlessly, can't reach consensus.

**Solution**:
- **Set time limits**: "We have 3 minutes to place this process"
- **Good enough**: "We're making estimates, not perfect predictions"
- **Sponsor decides**: "Let's hear both views, then [Sponsor] breaks the tie"
- **Defer precision**: "Whether it's high-medium or medium-high doesn't matter—it's not our first project either way"

### Challenge 4: Political Conflicts

**Symptom**: Departments lobbying for their processes, undermining feasibility of others.

**Solution**:
- **Refocus on enterprise value**: "What's best for the organization overall?"
- **Data-driven**: "Let's look at the quantified benefits we documented"
- **Sequencing reassurance**: "Every high-impact process will get done, just not all at once"
- **Sponsor intervention**: Sponsor must arbitrate political conflicts

### Challenge 5: IT Dominates Discussion

**Symptom**: IT focuses on technical challenges, drowning out business impact discussion.

**Solution**:
- **Separate impact and feasibility**: "Let's assess business impact first WITHOUT discussing technical challenges. Then we'll address feasibility."
- **Business speaks first**: "Let's hear from the business stakeholders before IT weighs in"
- **Facilitator redirects**: "That's a great technical point—let's hold it for the feasibility discussion"

### Challenge 6: Scope Creep During Workshop

**Symptom**: "While we're at it, we should also include..."

**Solution**:
- **Parking lot**: "Great idea. Let's capture it as a future enhancement"
- **Scope boundary**: "Today we're prioritizing these [N] processes we identified. Additional processes can be evaluated in future planning"
- **Impact of delay**: "Adding scope now delays our first delivery by [X] months. Is that trade-off worthwhile?"

---

## Workshop Success Metrics

### Immediate Indicators (End of Workshop)

✅ **Consensus achieved** on prioritization grid placement
✅ **First project selected** with specific business process identified
✅ **Implementation sequence** agreed for top 3-5 processes
✅ **Sponsor commitment** secured for first project resources
✅ **Business representatives** committed to participate in modeling workshops

### Leading Indicators (2-4 Weeks After)

✅ **Dimensional modeling workshops** scheduled with participants confirmed
✅ **Project resources** assigned and available
✅ **Project charter** approved and funded
✅ **Momentum** maintained - no stalling or re-opening prioritization decisions

### Lagging Indicators (3-6 Months)

✅ **First project delivered** on agreed timeline
✅ **Business value** realized as projected
✅ **Second project initiated** leveraging first project's conformed dimensions
✅ **Organization requests** to accelerate subsequent projects (demand signal)

---

## Template Materials

### Slide Deck Template (Outline)

```
Slide 1: Title - "[Organization] DW/BI Prioritization Workshop"
Slide 2: Agenda
Slide 3: Requirements Process Recap
Slide 4-5: Key Themes Discovered
Slide 6-13: Business Process Summaries (one per process, use template)
Slide 14: Opportunity/Stakeholder Matrix
Slide 15: Break
Slide 16: Prioritization Framework Introduction
Slide 17: Impact Criteria
Slide 18: Feasibility Criteria
Slide 19: Prioritization Grid (Blank)
Slide 20: Prioritization Grid (Filled - after exercise)
Slide 21: Implementation Sequence
Slide 22: Next Steps
Slide 23: Questions
```

### Business Process Summary Template (Slide)

```
[PROCESS NAME]

Description: [1-2 sentence description of operational activity]

Business Questions:
• [Question 1]
• [Question 2]
• [Question 3]

Stakeholders: [Dept 1] | [Dept 2] | [Dept 3]

Sample Metrics: [Metric 1], [Metric 2], [Metric 3]

Data Feasibility: ✅ [What's good] | ⚠️ [What's challenging]

Business Value: $[X-Y]M opportunity | [Other quantified benefits]

Complexity: [High / Medium / Low]
```

---

## Related Skills

- **Business Requirements Gathering** (Skill 1): This workshop follows requirements collection
- **Enterprise Bus Matrix Design** (Skill 2): Prioritization determines bus matrix implementation sequence
- **Four-Step Dimensional Modeling** (Skill 3): Next step after prioritization—design selected process
- **Program Management** (Skill 20): Prioritization is key program management activity

---

## Reference

This guide supports the **enterprise-bus-matrix-design** skill. For complete context, see **SKILL.md** in this directory.
