# Interview Questionnaires for Business Requirements Gathering

This reference provides detailed questionnaires organized by audience type. Use these as thinking tools and starting points—not rigid scripts—to guide your requirements sessions.

## General Interview Principles

### Before Using These Questionnaires

- **Customize for your organization**: Replace generic terms with company-specific language
- **Update as you learn**: Refine questions throughout the interview process
- **Stay flexible**: Follow interesting threads even if they're not on your list
- **Listen actively**: The best insights often come from follow-up probing

### Interview Team Roles

- **Lead interviewer**: Asks questions, probes deeper, maintains flow
- **Scribe**: Takes detailed notes, captures quotes and examples
- **Observers**: 1-2 additional team members listening and learning

---

## Executive Interview Questionnaire

**Target Audience**: C-level executives, senior vice presidents, business unit leaders
**Duration**: 1 hour
**Goals**: Understand strategic vision, business priorities, success criteria

### Opening (5 minutes)

1. Thank you for your time. To start, can you briefly describe your role and responsibilities?
2. What are your top 3-5 business priorities for the next 12-18 months?

### Strategic Vision (15-20 minutes)

3. What is your vision for how better information access could impact the organization?
4. What business decisions do you currently struggle to make due to information gaps?
5. If you could wave a magic wand and have perfect information, what would that enable you to do?
6. How do you currently measure success in your organization? (KPIs, metrics)
7. What keeps you up at night regarding business performance?

### Current State Pain Points (10-15 minutes)

8. When you need to make a critical decision, where do you get your information today?
9. How much time do your teams spend preparing information for you vs. analyzing it?
10. Can you describe a recent situation where lack of timely/accurate information cost you?
11. Have you encountered situations where different groups report conflicting numbers?
12. What decisions are you making based on gut feel rather than data?

### Competitive & External Factors (10 minutes)

13. What competitive pressures are driving the need for better information?
14. Are there regulatory or compliance requirements affecting your information needs?
15. What external trends (market, customer behavior, etc.) do you need to monitor?

### Success Criteria & Expectations (10-15 minutes)

16. How will you measure the success of this DW/BI initiative?
17. What would constitute a home run in the first 6-12 months?
18. What level of investment do you expect this to require?
19. What's your tolerance for a phased, iterative approach vs. big-bang deployment?
20. Do you prefer standardized reports/dashboards or ad hoc query capabilities for your team?

### Organizational Dynamics (5-10 minutes)

21. Who else in the organization should we be talking to?
22. Are there any political or organizational challenges we should be aware of?
23. What's your expectation for ongoing business involvement in this initiative?

### Closing

24. What else should we know that we haven't asked about?
25. May we follow up if additional questions arise?

---

## Middle Management Interview Questionnaire

**Target Audience**: Directors, managers, department heads
**Duration**: 1-1.5 hours
**Goals**: Understand tactical needs, day-to-day decision making, operational metrics

### Opening (5 minutes)

1. Can you describe your role and your team's responsibilities?
2. How does your work fit into the larger organization?
3. Who are your primary internal and external customers?

### Business Process Understanding (15-20 minutes)

4. What are the key business processes you manage? (e.g., order processing, claims handling)
5. Walk me through a typical [business process]. What are the key steps?
6. Where do things typically go wrong or get delayed in this process?
7. How do you currently measure process performance?
8. What process improvements would you make if you had better information?

### Performance Metrics & KPIs (15-20 minutes)

9. What are your top 5-10 performance metrics?
10. For each metric:
    - How is it currently calculated?
    - How often do you review it?
    - What actions do you take based on this metric?
    - What's your target or goal for this metric?
11. Are there metrics you wish you had but don't currently track?
12. How do your metrics roll up to executive-level KPIs?

### Current Reporting & Analysis (15-20 minutes)

13. Can you show me your most important reports? (review brought materials)
14. For each key report:
    - How often do you use this?
    - What decisions does it support?
    - What's missing or could be improved?
    - How long does it take to produce?
15. How much of your team's time is spent creating reports vs. analyzing them?
16. What analysis do you do in Excel after extracting data from operational systems?

### Dimensionality Questions (Hands-On Data Users) (10-15 minutes)

17. How do you distinguish between [products/customers/facilities/etc.]?
18. How do you naturally categorize or group [dimension entity]?
19. What hierarchies or roll-ups do you use? (e.g., product to brand to category)
20. What attributes are most important for filtering or grouping?
21. Are there seasonal or time-based patterns you need to analyze?

### Data Quality & Challenges (10 minutes)

22. What data quality issues do you currently face?
23. Which data sources do you trust? Which don't you trust?
24. How do you currently reconcile data from multiple systems?
25. Have you found errors in your current reports? How often?

### Future Needs & Opportunities (10 minutes)

26. If you could answer three questions you can't answer today, what would they be?
27. What new capabilities would help you do your job better?
28. Are there analyses you'd like to do but can't with current systems?
29. How would improved information impact your team's effectiveness?

### Closing

30. Who else in your organization should we talk to?
31. Can we follow up with you as we develop the solution?

---

## Analyst/Power User Interview Questionnaire

**Target Audience**: Business analysts, data analysts, power users
**Duration**: 1.5 hours
**Goals**: Deep understanding of data needs, analysis patterns, technical requirements

### Opening (5 minutes)

1. Describe your role and primary responsibilities.
2. What types of analysis do you perform regularly?
3. Who consumes your analyses?

### Current Analysis Approach (20-25 minutes)

4. Walk me through your typical analysis workflow from start to finish.
5. What tools do you currently use? (Excel, SQL, reporting tools, etc.)
6. Can you show me examples of recent analyses? (review brought materials)
7. For each analysis:
    - What business question were you trying to answer?
    - What data sources did you use?
    - How long did it take to complete?
    - What challenges did you encounter?
    - How could this analysis be improved?

### Data Sources & Integration (15-20 minutes)

8. What operational systems do you extract data from?
9. How do you combine data from multiple sources?
10. What data integration challenges do you face?
11. How do you handle different definitions of the same thing across systems?
    (e.g., "customer" means different things in different systems)
12. What manual data manipulation do you do in Excel?

### Detailed Dimensionality (20-25 minutes)

13. Let's talk about [primary dimension - e.g., Product]:
    - What product attributes do you use for filtering?
    - What product hierarchies exist? (e.g., SKU → Brand → Category → Department)
    - How do products change over time?
    - What product groupings matter for analysis?

14. For [another key dimension - e.g., Customer]:
    - How do you segment or categorize customers?
    - What customer attributes drive analysis?
    - How do customer relationships work? (households, accounts, etc.)

15. Regarding Time:
    - What time periods do you analyze? (daily, weekly, monthly, quarterly)
    - Do you need fiscal vs. calendar analysis?
    - What historical depth do you need?
    - Are there seasonal patterns or comparisons important to you?

### Metrics & Calculations (15-20 minutes)

16. What metrics do you calculate most frequently?
17. For key metrics, what's the exact calculation formula?
18. Are there derived or calculated metrics that are complex?
19. Do you perform trending, forecasting, or predictive analysis?
20. What statistical or analytical functions do you use?

### Ad Hoc vs. Standardized (10 minutes)

21. What percentage of your work is ad hoc vs. standardized reporting?
22. For standardized reports, how often do they change?
23. What makes an analysis "ad hoc" for you?
24. Do you create analyses for others to reuse?

### Technical Requirements (10-15 minutes)

25. What query performance is acceptable? (Seconds? Minutes?)
26. How much data do you typically analyze? (Row counts, time periods)
27. Do you need to export data to other tools? Which ones?
28. Do you need real-time or near-real-time data? For what?
29. What data refresh frequency do you need?

### Pain Points & Wishlist (10 minutes)

30. What frustrates you most about current data access?
31. What analysis capabilities do you wish you had?
32. If you could change one thing about how you work with data, what would it be?
33. What would make you 50% more productive?

### Closing

34. Can you share sample datasets or reports for us to review?
35. Would you be willing to participate in design reviews?

---

## IT/Source System Expert Interview Questionnaire

**Target Audience**: DBAs, system administrators, application owners
**Duration**: 1-1.5 hours
**Goals**: Understand data feasibility, technical constraints, source system realities

### Opening (5 minutes)

1. What systems and databases are you responsible for?
2. What is the primary business purpose of each system?
3. How long have these systems been in production?

### Data Availability (15-20 minutes)

4. For [key business process mentioned by business users]:
    - Is this data captured in your systems?
    - At what level of detail/granularity?
    - How long is history retained online? In archives?
    - Is there a unique transaction identifier?

5. What is the smallest grain of data available?
6. Are there aggregations or summarizations in the source?
7. What data is purged and how frequently?

### Data Quality Assessment (15-20 minutes)

8. What data quality issues exist in the source systems?
9. Which fields have high percentages of nulls or unknown values?
10. Are there mandatory vs. optional fields?
11. What validation rules exist at data entry?
12. How are errors or exceptions handled?
13. Can you provide data profiling statistics? (domain values, counts, ranges)
14. What does "soft delete" vs. "hard delete" look like in your data?

### Data Structures (15-20 minutes)

15. Can you walk me through the key tables and relationships?
16. Are there master data tables for [key dimensions - product, customer, etc.]?
17. What are the primary and foreign key relationships?
18. Where are code/decode lookup tables?
19. How are hierarchies represented in the data?
20. What date/time stamps are available?

### Change Data Capture (10-15 minutes)

21. How can we identify new and changed records?
22. Are there audit columns (created date, modified date, modified by)?
23. Are database logs available for change data capture?
24. How are deletes handled? (soft delete flags vs. hard deletes)
25. What is the frequency of batch updates vs. real-time transactions?

### Integration & Access (10-15 minutes)

26. What other systems integrate with yours? How?
27. What are the current extraction methods? (ETL tools, queries, APIs, files)
28. What is the acceptable extraction window for DW/BI?
29. Are there technical limitations on query volume or duration?
30. What security/access controls exist?

### Technical Constraints (10 minutes)

31. What are the peak transaction volumes and times?
32. What technical environment (database platform, version, hardware)?
33. Are there planned system upgrades or migrations?
34. Are there compliance or regulatory requirements affecting the data?

### Lessons Learned (5-10 minutes)

35. What lessons have you learned from previous reporting/integration efforts?
36. What hasn't worked well in the past?
37. What recommendations do you have for us?

### Closing

38. Can you provide sample data extracts for profiling?
39. Who else should we talk to about the source systems?
40. Can we schedule follow-up sessions as questions arise?

---

## Question Types Reference

### Open-Ended Questions (Use Frequently)

Use to encourage detailed responses:
- "Can you describe...?"
- "Walk me through...?"
- "Tell me about...?"
- "How do you...?"
- "What happens when...?"

### Probing Questions (Use to Go Deeper)

Use when you get vague or incomplete answers:
- "Can you give me an example?"
- "What do you mean by [term]?"
- "How often does that happen?"
- "What would that look like?"
- "Why is that important?"
- "What happens next?"

### Clarifying Questions

Use to ensure understanding:
- "So you're saying that...?"
- "Help me understand...?"
- "Do you mean X or Y?"
- "Is this similar to...?"

### Quantifying Questions

Use to get specific numbers:
- "How many...?"
- "How often...?"
- "What percentage...?"
- "How long does it take...?"
- "What's the volume...?"

### Questions to Avoid

Don't use leading questions:
- ❌ "Don't you think it would be better if...?"
- ❌ "Wouldn't you agree that...?"

Don't use yes/no questions without follow-up:
- ❌ "Do you need this data?" (Too simple)
- ✓ "Tell me about how you would use this data." (Better)

Don't use compound questions:
- ❌ "How do you track performance and what tools do you use and who gets the reports?"
- ✓ Ask one question at a time

---

## Interview Tips

### Active Listening Techniques

1. **Maintain eye contact** (when culturally appropriate)
2. **Take notes but don't hide behind them**—look up frequently
3. **Use verbal acknowledgments**: "I see", "That makes sense", "Interesting"
4. **Paraphrase back**: "So what I'm hearing is..."
5. **Don't interrupt**—wait for natural pauses
6. **Allow silence**—give people time to think

### Handling Difficult Situations

**If someone is too high-level and vague:**
- "That's helpful context. Can you give me a specific example?"
- "Let's drill down into that. What does that look like day-to-day?"

**If someone goes off on tangents:**
- "That's interesting. Let me make a note of that. Going back to..."
- "I want to make sure we cover everything. Can we return to...?"

**If someone doesn't know the answer:**
- "No problem. Who would know about that?"
- "Is this something we can follow up on later?"

**If someone dominates a group interview:**
- "That's great, thank you. Let's hear from others. [Name], what's your perspective?"

**If someone provides too much technical detail:**
- "Help me understand the business impact of that..."
- "From a user perspective, what does that mean?"

### After the Interview

**Immediate Debrief (30 minutes)**:
1. Ensure team alignment on what was heard
2. Capture key insights while fresh
3. Review notes for gaps
4. Identify follow-up questions
5. Update questionnaire for next interview

**Note Taking Best Practices**:
- Capture direct quotes for important statements
- Note body language and enthusiasm levels
- Flag items needing follow-up with "FOLLOW-UP:"
- Mark conflicting information between interviews with "CONFLICT:"
- Highlight potential quick wins with "QUICK WIN:"

---

## Customization Checklist

Before your first interview, customize these questionnaires by:

- [ ] Replacing generic industry terms with company-specific language
- [ ] Adding company-specific business process names
- [ ] Including relevant system names
- [ ] Incorporating terminology from annual reports/websites
- [ ] Removing questions not applicable to your organization
- [ ] Adding organization-specific context questions
- [ ] Tailoring to the specific project scope
- [ ] Getting feedback from business sponsor on appropriateness

Remember: These questionnaires are starting points. The best interviews follow the natural flow of conversation while ensuring you cover critical topics.
