# Design Session Facilitation Guide

## Purpose

This guide provides practical facilitation techniques for running collaborative dimensional modeling workshops. Dimensional models should never be designed in isolation; they require active participation from business representatives, source system experts, and data governance stewards.

## When to Use This Guide

Use this guide when:
- Planning and organizing dimensional modeling design sessions
- Facilitating collaborative workshops with cross-functional teams
- Managing group dynamics and driving to consensus
- Keeping modeling sessions productive and on track

## Why Collaborative Modeling Matters

**Benefits of Collaboration:**
- **Richer designs**: Cross-functional input creates more complete models
- **Fewer misunderstandings**: Direct business involvement reduces rework
- **Better buy-in**: Participants become advocates for the dimensional model
- **Shared understanding**: Team learns together, improving future designs
- **Business validation**: Real-time feedback ensures requirements are met
- **Source data realities**: Source experts identify data quality issues early

**Risks of Solo Modeling:**
- Data-driven designs that don't meet business needs
- Missed requirements discovered late in ETL development
- Business resistance due to lack of ownership
- Incorrect assumptions about source data availability or quality
- Naming and definitions that don't resonate with users

---

## Core Team Composition

### Required Participants

**1. Data Modeler (Facilitator/Owner)**
- **Role**: Facilitates sessions, owns deliverables, drives design decisions
- **Skills**: Dimensional modeling expertise, facilitation, diplomacy
- **Responsibilities**:
  - Prepare agenda and materials for each session
  - Lead four-step design process
  - Capture design decisions in worksheets
  - Update documentation between sessions
  - Escalate issues requiring governance decisions
- **Time Commitment**: 80-100% dedicated during modeling effort (3-4 weeks)

**2. Business Representatives (Subject Matter Experts)**
- **Role**: Provide business requirements, validate design, approve naming/definitions
- **Ideal Profile**: Analysts who use source systems and understand business processes
- **Responsibilities**:
  - Explain business requirements and use cases
  - Validate grain, dimensions, and facts against real business needs
  - Approve business-friendly names and definitions
  - Identify critical vs. nice-to-have attributes
  - Serve as liaisons to broader business community
- **Time Commitment**: 2-3 hours per session, 3-4 days/week (mornings or afternoons)
- **Quantity**: 2-4 representatives (more can slow progress)

**3. Source System Experts (Data Realists)**
- **Role**: Explain source data structure, content, relationships, and quality
- **Responsibilities**:
  - Describe available source data and transformations needed
  - Identify data quality issues and remediation approaches
  - Validate technical feasibility of proposed attributes/facts
  - Explain operational system idiosyncrasies
  - Clarify business rules embedded in source systems
- **Time Commitment**: 2-3 hours per session, as needed (not every session)
- **Quantity**: 1-2 experts per major source system

**4. Data Governance Representatives (Stewards)**
- **Role**: Ensure conformed dimensions, drive organizational consensus
- **Responsibilities**:
  - Represent enterprise data governance policies
  - Drive agreement on conformed dimension names and definitions
  - Escalate conflicts to governance steering committee
  - Document data governance decisions
  - Ensure compliance with naming conventions
- **Time Commitment**: Attend sessions for core conformed dimensions (customer, product, date, location)
- **Quantity**: 1-2 stewards

### Optional but Valuable Participants

**5. Physical DBA**
- Provides input on partitioning, indexing, performance considerations
- Attends selected sessions to understand design rationale
- Avoids temptation to normalize during implementation

**6. ETL Team Members**
- Learns design rationale and transformation requirements
- Provides early feedback on ETL feasibility
- Begins thinking about ETL specifications

**7. BI Application Developers**
- Understands how model will support BI tools
- Validates model usability from BI perspective
- Begins planning report/dashboard development

---

## Pre-Session Preparation

### Week Before First Session

**1. Assemble the Team** (1 week prior)
- [ ] Identify and confirm all required participants
- [ ] Get calendar commitments for 3-4 weeks of sessions
- [ ] Send calendar invitations with session schedule (2-3 hours, 3-4 days/week)
- [ ] Reserve dedicated conference room with whiteboards
- [ ] Arrange projector for laptop presentations

**2. Prepare Prerequisite Materials** (3-5 days prior)
- [ ] Review business requirements documentation
- [ ] Extract relevant bus matrix row(s) for target business process
- [ ] Compile list of source systems identified in requirements
- [ ] Gather sample source data extracts or access to development database
- [ ] Create initial bubble chart draft (skeleton diagram)

**3. Distribute Pre-Reading** (2 days prior)
- [ ] Send business requirements summary (5-10 pages)
- [ ] Send relevant section of bus matrix
- [ ] Send draft bubble chart (if available)
- [ ] Provide dimensional modeling primer (if team is new to concepts)
- [ ] Clarify goals and expected outcomes of session

**4. Logistics Preparation** (1 day prior)
- [ ] Verify room reservation and equipment (projector, whiteboards, markers)
- [ ] Prepare whiteboard with four-step process outline
- [ ] Print worksheets and templates
- [ ] Bring self-stick flip chart paper
- [ ] Test data profiling tool access (if using during session)
- [ ] Prepare laptop with modeling spreadsheet templates

---

## Facilitation Best Practices

### Facilitator Mindset

**Be the Guide, Not the Expert:**
- Ask questions rather than declaring answers
- Draw out expertise from business representatives
- Synthesize diverse viewpoints into coherent design
- Stay neutral during disagreements; facilitate consensus

**Balance Speed and Thoroughness:**
- Keep sessions moving; don't get lost in minutiae
- Park non-critical issues for later resolution
- Focus on answering: grain, dimensions, facts
- Save detailed attribute work for specific dimension deep-dives

**Manage Energy and Engagement:**
- Take breaks every 60-90 minutes
- Use whiteboards and visual aids, not just verbal discussion
- Encourage participation from quieter members
- Redirect dominating participants politely

**Stay Business-Focused:**
- Use business terminology, not technical jargon
- Frame discussions around business requirements
- Avoid premature physical design discussions (partitioning, indexes)
- Defer ETL complexity to later phases

---

## Session Structure

### Recommended Schedule

**Week 1: High-Level Design and Simple Dimensions**

**Session 1 (2-3 hours): Kickoff and High-Level Bubble Chart**
- Introductions and role clarifications (15 min)
- Dimensional modeling education (if needed) (30 min)
- Review business requirements (30 min)
- Four-step process walkthrough (30 min)
  - Step 1: Confirm business process
  - Step 2: Declare grain (critical discussion)
  - Step 3: Identify major dimensions (preliminary)
  - Step 4: Identify major facts (preliminary)
- Create high-level bubble chart together (30 min)
- Next steps and homework (15 min)

**Session 2 (2-3 hours): Date Dimension Detailed Design**
- Review Session 1 bubble chart (10 min)
- Design date dimension table (90 min)
  - Standard calendar attributes (day, week, month, quarter, year)
  - Fiscal calendar attributes (if different from calendar)
  - Holiday indicators, weekday/weekend flags
  - Special attributes (last day of month, shopping season, etc.)
- Document in dimension worksheet (30 min)
- Data profiling activity (if time permits) (20 min)
- Next steps (10 min)

**Session 3 (2-3 hours): Additional Simple Dimensions**
- Review date dimension worksheet (10 min)
- Design 1-2 additional straightforward dimensions (100 min)
  - Store/location dimension (geographic and organizational hierarchies)
  - Payment method dimension
  - Cashier/employee dimension (subset of attributes)
- Document worksheets (20 min)
- Next steps (10 min)

**Week 2: Complex Core Dimensions**

**Session 4 (2-3 hours): Product Dimension (Part 1)**
- Review progress and worksheets (15 min)
- Product dimension high-level structure (30 min)
  - Merchandise hierarchies (SKU, brand, category, department)
  - Non-hierarchical attributes (package type, size, organic, etc.)
  - Multiple hierarchies discussion
- Begin detailed attribute definition (60 min)
- Data profiling of product source tables (30 min)
- Next steps (15 min)

**Session 5 (2-3 hours): Product Dimension (Part 2)**
- Review draft product dimension (15 min)
- Continue detailed attribute definition (75 min)
  - Embedded codes (decode and expand)
  - Numeric values as attributes vs. facts decision
  - SCD strategy per attribute
- Complete product dimension worksheet (30 min)
- Data governance discussion (if needed) (15 min)
- Next steps (15 min)

**Session 6 (2-3 hours): Promotion/Causal Dimension**
- Review product dimension worksheet (10 min)
- Design promotion dimension (90 min)
  - Price reductions, ads, displays, coupons
  - Combined vs. separated dimensions discussion
  - "No Promotion" row handling
- Complete promotion dimension worksheet (30 min)
- Next steps (10 min)

**Week 3: Remaining Dimensions and Fact Tables**

**Session 7 (2-3 hours): Customer/Remaining Dimensions**
- Review all dimension worksheets completed so far (20 min)
- Design any remaining dimensions (70 min)
  - Customer/frequent shopper dimension (if applicable)
  - Any other dimensions from bubble chart
- Complete worksheets (30 min)
- Next steps (10 min)

**Session 8 (2-3 hours): Fact Table Design**
- Review complete set of dimension worksheets (15 min)
- Revisit grain statement (confirm still correct) (10 min)
- Identify all facts consistent with grain (60 min)
  - List all measurements from source system
  - Classify: additive, semi-additive, non-additive
  - Decide which derived facts to store
  - Identify numerator/denominator for ratios
- Complete fact table worksheet (45 min)
- Next steps (10 min)

**Session 9 (2-3 hours): SCD Strategies and Issues Resolution**
- Review fact table worksheet (15 min)
- Define SCD strategy for each dimension attribute (60 min)
  - Walk through each dimension
  - Business decision for each attribute (Type 0, 1, 2, etc.)
  - Document rationale
- Review open issues log (40 min)
  - Resolve what can be resolved
  - Assign remaining issues
  - Escalate governance issues
- Next steps (15 min)

**Week 4: Review, Validation, and Finalization**

**Session 10 (2-3 hours): Internal IT Review Preparation**
- Review complete dimensional model (30 min)
- Prepare IT review presentation (60 min)
- Walk through presentation (dry run) (30 min)
- Finalize all worksheets (20 min)

**Session 11 (2-3 hours): IT Review Meeting**
- Present bus matrix for context (15 min)
- Present bubble charts (15 min)
- Review dimension worksheets table-by-table (60 min)
- Review fact table worksheet (30 min)
- Review open issues (20 min)
- Capture feedback and action items (10 min)

**Session 12 (2-3 hours): Business User Review Meeting**
- Present bus matrix as enterprise roadmap (10 min)
- Present bubble charts and hierarchy diagrams (20 min)
- Demonstrate sample queries using model (40 min)
- Review critical dimensions (product, customer) (40 min)
- Capture feedback (20 min)
- Next steps and celebration (10 min)

**Session 13 (2-3 hours): Finalization**
- Update model based on all review feedback (60 min)
- Resolve final issues (30 min)
- Finalize all documentation (30 min)
- Hand off to ETL and physical design teams (15 min)

---

## Facilitation Techniques

### Opening Each Session

**1. State the Objective** (2 min)
- "Today we're designing the product dimension"
- "Our goal is to identify all facts for the sales fact table"
- Clear, focused objective keeps session on track

**2. Review Previous Session** (5-10 min)
- Walk through what was accomplished last time
- Highlight any changes made since last session
- Address questions or concerns from previous session

**3. Preview Today's Agenda** (2 min)
- List major topics to be covered
- Identify decision points requiring group input
- Set time expectations

### During the Session

**1. The Four-Step Process as Anchor**

Always return to the four-step process when discussions wander:

- "Let's refocus on the grain. What does one row represent?"
- "Is this proposed dimension single-valued at our grain?"
- "Is this fact true to the grain we declared?"

**2. Visual Facilitation**

Use whiteboards to:
- Draw dimension hierarchies
- Sketch fact table structure with foreign keys
- Illustrate one-to-many relationships
- List candidate attributes/facts for group discussion
- Track decisions and parking lot items

**3. Question-Driven Discovery**

Ask open-ended questions rather than providing answers:
- "How do business users describe products when analyzing sales?"
- "What attributes would help you filter and group this data?"
- "Are there different ways products roll up? Multiple hierarchies?"
- "What measurements result from a sales transaction?"
- "Is gross margin additive or non-additive? Why?"

**4. Manage the Parking Lot**

- Use self-stick flip chart paper for "Parking Lot" items
- Write down tangential topics, deferrable decisions, future enhancements
- Promise to return to parking lot items (and actually do)
- Prevents derailing while capturing good ideas

**5. Manage Disagreements**

**When business representatives disagree:**
- Listen to both perspectives without judgment
- Probe for underlying requirements ("Why is that important to you?")
- Look for common ground
- Table decision for data governance committee if needed
- Document disagreement and rationale in issues log

**When business and IT disagree:**
- Business defines "what"; IT defines "how"
- Validate if business requirement is truly needed
- Explore alternative solutions that meet requirement differently
- Avoid premature optimization ("We can't do that, it's too hard")

**6. Time Management**

- Allocate specific time to each topic on agenda
- Provide time warnings ("We have 10 minutes left on the product dimension")
- Be willing to extend critical discussions into next session
- Don't rush to bad decisions just to stay on schedule

**7. Encourage Quiet Participants**

- Directly ask quieter members for input
- "Sarah, you're familiar with the source product master. What do you think?"
- "John, from your analyst perspective, would this attribute be useful?"
- Use round-robin techniques to ensure everyone contributes

**8. Redirect Dominating Participants**

- Politely interrupt and redirect
- "Thanks, Mike. Let's hear from others on this."
- "That's a great point. Let's capture it and hear other perspectives."
- Privately ask dominant participants to allow others to contribute

### Closing Each Session

**1. Recap Decisions and Progress** (5 min)
- Summarize what was accomplished
- Review key decisions made
- Celebrate progress

**2. Review Action Items** (5 min)
- Assign open issues to specific people
- Set deadlines for action items
- Clarify expectations for intersession work

**3. Preview Next Session** (3 min)
- State objective of next session
- Identify any pre-work or preparation needed
- Confirm next session date/time

**4. Update Documentation Immediately After**
- Update worksheets while decisions are fresh
- Document rationale for key decisions
- Distribute updated materials to team

---

## Managing Common Challenges

### Challenge 1: Participants Unfamiliar with Dimensional Modeling

**Symptoms:**
- Confusion about grain, facts, dimensions
- Desire to normalize dimension tables
- Resistance to "redundant" dimension attributes
- Concern about surrogate keys

**Solutions:**
- Provide 30-60 minute dimensional modeling education at kickoff
- Use retail sales example (universally understood)
- Show side-by-side 3NF vs. dimensional model comparison
- Explain "different goals: OLTP efficiency vs. query simplicity"
- Reference "Resisting Normalization Urges" from Chapter 3

### Challenge 2: Business Representatives Too Busy

**Symptoms:**
- Cancellations or no-shows
- Sending delegates unfamiliar with business process
- Distracted during sessions (laptops open, responding to email)
- Delegating decisions without adequate input

**Solutions:**
- Escalate to executive sponsor to reinforce importance
- Schedule shorter, more frequent sessions (90 min instead of 3 hours)
- Emphasize that their input now prevents expensive rework later
- Make sessions engaging and productive so they see value
- Consider dedicating business representatives full-time for duration

### Challenge 3: Scope Creep During Design

**Symptoms:**
- "While we're at it, let's also model inventory..."
- Adding dimensions or facts beyond original requirements
- Designing for hypothetical future requirements
- Analysis paralysis over nice-to-have attributes

**Solutions:**
- Refer back to bus matrix and original scope
- Use parking lot for future enhancements
- Distinguish "critical for first release" vs. "nice to have later"
- Remind team of graceful extensibility (can add attributes/facts later)
- Get sponsor approval for scope changes

### Challenge 4: Data Quality Concerns Derail Design

**Symptoms:**
- "We can't use that data; it's terrible quality"
- Lengthy discussions about source data cleanup
- Desire to fix all data quality issues before designing model
- Paralysis over imperfect source data

**Solutions:**
- Acknowledge data quality issues but keep designing
- Document data quality concerns in issues log
- Design model for ideal state; ETL can handle imperfections
- Use "Unknown" or "Not Applicable" for missing/bad data
- Separate data quality improvement from initial model design

### Challenge 5: Unable to Reach Consensus on Conformed Dimensions

**Symptoms:**
- Different business units want different attribute names
- Disagreement on attribute definitions or values
- Political conflicts over dimension ownership
- Inability to agree on single version of truth

**Solutions:**
- Escalate to data governance steering committee
- Document all perspectives in issues log
- Have executive sponsor broker agreement
- Consider implementing "partial conformance" initially
- Focus on grain and core attributes first; defer contentious attributes
- Remind team that lack of conformance means no integrated analytics

### Challenge 6: Analysis Paralysis Over SCD Strategies

**Symptoms:**
- Inability to decide SCD type for attributes
- Fear of choosing wrong strategy
- Desire to track history for everything (all Type 2)
- Excessive time spent on SCD decisions

**Solutions:**
- Start with "When do you need historical analysis?" question
- Default to Type 1 unless history explicitly required
- Use Type 2 for attributes business frequently analyzes over time
- Remind team that changing SCD type later is possible (not ideal, but possible)
- Focus on most critical dimensions first (product, customer)
- Defer SCD for less important dimensions

### Challenge 7: Source System Expert Becomes Bottleneck

**Symptoms:**
- Frequent "I need to check on that" responses
- Long delays waiting for source data analysis
- Source expert overwhelmed with data profiling requests
- Progress stalled due to missing source information

**Solutions:**
- Prioritize data profiling requests (critical path first)
- Assign source expert specific homework between sessions
- Continue design where possible without detailed source knowledge
- Document assumptions; validate later when source data available
- Engage additional source system experts if possible
- Leverage data profiling tools for self-service investigation

---

## Facilitation Checklist

### Before Each Session
- [ ] Review previous session notes and updated worksheets
- [ ] Prepare agenda with time allocations
- [ ] Identify key decisions needed in this session
- [ ] Prepare whiteboard templates or diagrams
- [ ] Bring printed worksheets, bus matrix, and requirements
- [ ] Verify room and equipment

### During Each Session
- [ ] Start on time (don't wait for latecomers)
- [ ] State objective and agenda
- [ ] Review previous session decisions
- [ ] Facilitate using four-step process as anchor
- [ ] Use whiteboard for visual facilitation
- [ ] Manage time and keep discussions focused
- [ ] Capture decisions in worksheets
- [ ] Maintain parking lot for tangential topics
- [ ] Assign action items with owners and deadlines
- [ ] Recap decisions and preview next session
- [ ] End on time

### After Each Session
- [ ] Update all worksheets immediately
- [ ] Document design rationale for key decisions
- [ ] Update issues log with new items
- [ ] Distribute updated materials to team
- [ ] Follow up on action items
- [ ] Prepare for next session

---

## Measuring Facilitation Success

**Good indicators your sessions are effective:**

- **High attendance and engagement**: Participants attend consistently and contribute actively
- **Decisions are made**: Sessions produce concrete design progress, not just discussion
- **Business validation**: Business representatives confirm model meets their needs
- **Minimal rework**: Few "we need to redo this" moments as design unfolds
- **Team learning**: Team gains dimensional modeling understanding and can explain decisions
- **On schedule**: Completing design within 3-4 week timeframe for single business process
- **Quality worksheets**: Thorough documentation that ETL team can use
- **Positive feedback**: Participants feel sessions are valuable use of their time

**Warning signs of facilitation problems:**

- **Declining attendance**: People stop showing up or send delegates
- **Circular discussions**: Same topics resurface repeatedly without resolution
- **Silence**: Participants not contributing; facilitator does all the talking
- **Scope drift**: Design expanding beyond original business process
- **Technical focus**: Discussions dominated by physical design, not logical model
- **Incomplete documentation**: Worksheets lack sufficient detail for ETL development

---

## Summary

Effective facilitation of collaborative dimensional modeling sessions requires:

1. **Right People**: Business representatives, source system experts, data stewards
2. **Clear Process**: Four-step dimensional design process as anchor
3. **Structured Sessions**: 2-3 hour sessions, 3-4 days/week, over 3-4 weeks
4. **Active Facilitation**: Question-driven discovery, visual aids, time management
5. **Thorough Documentation**: Worksheets updated immediately after each session
6. **Issue Management**: Parking lot and issues log to track and resolve concerns

Remember: The dimensional model you create together is far better than one created in isolation. The time invested in collaborative design is more than repaid through reduced rework, better business alignment, and stronger organizational buy-in.

The facilitator's job is not to design the perfect model alone, but to guide the team to discover the best model together.
