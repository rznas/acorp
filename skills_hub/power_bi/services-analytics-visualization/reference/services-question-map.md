# Services Question → Visualization Map

Each entry: business question → underlying fact/grain → recommended Power BI visual → DAX measure. Visuals are limited to those confirmed in the Power BI visualization-types doc. Measures are illustrative; adapt table/column names to the model. See `services-dax-library.md` for full measure definitions.

## Contents
- [Call Center / Telecom](#call-center--telecom) (27 questions)
- [Healthcare](#healthcare) (27 questions)
- [Education](#education) (26 questions)
- [Transportation / Airlines](#transportation--airlines) (26 questions)
- [Professional Services](#professional-services) (25 questions)
- [Cross-sub-industry notes](#cross-sub-industry-notes)

---

## Call Center / Telecom

Underlying facts: **call detail record (CDR)** transaction grain (one row per call/message/data session); **monthly service-line billing** periodic snapshot; **service provisioning** accumulating snapshot.

| # | Question | Fact / grain | Power BI visual | DAX measure |
|---|----------|--------------|-----------------|-------------|
| 1 | What is call volume by hour-of-day? | CDR transaction | Column chart (hour bins) | `Call Count = COUNTROWS(CDR)` |
| 2 | How does call volume vary by hour across days of the week? | CDR | Line chart, small multiples by weekday | `Call Count` |
| 3 | What is total call volume trend over the last 12 months? | CDR | Line chart | `Call Count` |
| 4 | What is average call duration by customer segment? | CDR | Bar chart | `Avg Call Duration = AVERAGE(CDR[DurationSeconds])` |
| 5 | What is average handle time (AHT) trend? | CDR | Line chart | `AHT = DIVIDE(SUM(CDR[HandleSeconds]), [Call Count])` |
| 6 | What is the call abandonment rate? | CDR | Card | `Abandon Rate % = DIVIDE([Abandoned Calls], [Call Count])` |
| 7 | How does abandonment rate trend vs offered calls? | CDR | Combo chart (column: offered, line: abandon %) | `Offered Calls`, `Abandon Rate %` |
| 8 | What is first-call resolution (FCR) rate by team? | CDR | Bar chart | `FCR % = DIVIDE([Resolved First Call], [Call Count])` |
| 9 | What is current service level vs target? | CDR | Gauge chart | `Service Level % = DIVIDE([Calls Answered <=SLA], [Call Count])` |
| 10 | What is the distribution of call dispositions? | CDR | Donut chart (<8 categories) or column chart | `Call Count` |
| 11 | How many calls handled per agent? | CDR | Bar chart | `Call Count` grouped by Agent |
| 12 | What is agent occupancy vs target? | CDR | KPI | `Agent Occupancy % = DIVIDE([Handle + Hold Sec], [Logged-In Sec])` |
| 13 | What factors influence customer churn? | Billing snapshot + churn flag | Key influencers (target: Churned) | `Churned Customers` |
| 14 | What is the monthly churn rate trend? | Billing snapshot | Line chart | `Churn Rate % = DIVIDE([Churned Customers], [Customers at Start])` |
| 15 | Which customer segments churn most? | Billing snapshot | Bar chart | `Churn Rate %` by segment |
| 16 | What is ARPU (avg revenue per user) by plan? | Billing snapshot | Bar chart | `ARPU = DIVIDE([Total Billed], [Active Service Lines])` |
| 17 | What is total billed revenue trend? | Billing snapshot | Line chart | `Total Billed = SUM(Billing[TotalBilled])` |
| 18 | How is revenue split across rate plans? | Billing snapshot | Treemap or donut chart | `Total Billed` by rate plan |
| 19 | What is data usage (GB) trend by plan? | CDR (data sessions) | Stacked area chart | `Data Volume GB = SUM(CDR[DataMB]) / 1024` |
| 20 | What is the dropped-call rate by cell tower? | CDR | Filled map (choropleth, tower geography) | `Dropped Call % = DIVIDE([Dropped Calls], [Call Count])` |
| 21 | Which regions have the highest call volume? | CDR | Filled map or shape map | `Call Count` by region |
| 22 | What is roaming charge contribution? | Billing snapshot | Waterfall chart | `Roaming Charges = SUM(Billing[RoamingCharges])` |
| 23 | How many active service lines vs target? | Billing snapshot (semi-additive) | Card | `Active Service Lines` (period-end count) |
| 24 | What is net subscriber adds trend? | Billing snapshot | Combo chart (column: gross adds, line: net) | `Net Adds = [Gross Adds] - [Churned Customers]` |
| 25 | How long does provisioning take (order→activation)? | Provisioning accumulating snapshot | Card | `Avg Activation Days = AVERAGE lag measure` |
| 26 | How many activations are stuck at each stage? | Provisioning accumulating snapshot | Funnel chart | `Lines at Stage` count per milestone |
| 27 | Why did handle time spike last week? | CDR | Decomposition tree (drill AHT by team→queue→reason) | `AHT` |

## Healthcare

Underlying facts: **patient encounter** transaction grain (one row per visit); **hospital bed/census** periodic snapshot (semi-additive occupied beds); **claims billing** accumulating snapshot (one row per claim line, milestone dates); **diagnosis** via bridge table (multivalued).

| # | Question | Fact / grain | Power BI visual | DAX measure |
|---|----------|--------------|-----------------|-------------|
| 1 | What is the patient admission trend by department? | Encounter | Line chart (dept slicer/legend) | `Admissions = COUNTROWS(Encounter)` |
| 2 | How do admissions compare across departments? | Encounter | Bar chart | `Admissions` |
| 3 | What is the bed occupancy rate over time? | Census snapshot | Line chart (occupancy %) | `Occupancy % = DIVIDE([Avg Occupied Beds], [Total Beds])` |
| 4 | What is current bed occupancy vs capacity? | Census snapshot (semi-additive) | Gauge chart | `Avg Occupied Beds` vs `Total Beds` |
| 5 | What is average daily census by unit? | Census snapshot | Card or matrix | `Avg Occupied Beds = AVERAGE(Census[OccupiedBeds])` |
| 6 | Which diagnoses are most common? | Diagnosis bridge | Treemap (ICD chapter→code) | `Diagnosis Count = COUNTROWS(DiagnosisBridge)` |
| 7 | What is the readmission rate? | Encounter | Card | `Readmission % = DIVIDE([Readmissions], [Discharges])` |
| 8 | What factors drive 30-day readmission? | Encounter | Key influencers (target: Readmitted) | `Readmissions` |
| 9 | What is average length of stay (ALOS) by department? | Encounter / claims | Bar chart | `ALOS = AVERAGE(Encounter[LengthOfStayDays])` |
| 10 | How does ALOS trend over time? | Encounter | Line chart | `ALOS` |
| 11 | What is the ED no-show rate? | Encounter (appointments) | Card | `No-Show % = DIVIDE([No-Shows], [Scheduled])` |
| 12 | What drives appointment no-shows? | Encounter | Key influencers (target: NoShow) | `No-Shows` |
| 13 | How many claims are at each pipeline stage? | Claims accumulating snapshot | Funnel chart (treatment→billed→paid→closed) | `Claims at Stage` |
| 14 | What is average days from billing to payment? | Claims accumulating snapshot | Card | `Avg Days To Payment = AVERAGE lag` |
| 15 | How does claim settlement time trend? | Claims accumulating snapshot | Line chart | `Avg Days To Payment` |
| 16 | What is total billed vs collected? | Claims | Combo chart (column billed, line collection %) | `Total Billed`, `Collection % = DIVIDE([Total Paid],[Total Billed])` |
| 17 | What is the denial rate by payer? | Claims | Bar chart | `Denial % = DIVIDE([Denied Claims], [Submitted Claims])` |
| 18 | What is revenue by payer (primary/secondary)? | Claims | Treemap or donut chart | `Total Paid` by payer |
| 19 | What is the claim balance waterfall (billed→insurance paid→patient paid→write-off→remaining)? | Claims | Waterfall chart | balance component measures |
| 20 | Which claims are aging beyond target? | Claims | Table with conditional formatting | `Days Outstanding` lag |
| 21 | What is patient volume by region? | Encounter | Filled map (choropleth) | `Admissions` by region |
| 22 | What is OR (operating room) utilization vs available time? | Facility utilization factless | KPI | `OR Utilization % = DIVIDE([Utilized Minutes],[Available Minutes])` |
| 23 | Which time blocks are underutilized? | Facility utilization factless | Matrix (room × time block, conditional formatting) | `Utilization Flag SUM` |
| 24 | What is physician productivity (encounters per physician)? | Encounter | Bar chart | `Admissions` by physician |
| 25 | What is the case mix (encounters by diagnosis chapter)? | Diagnosis bridge | Treemap | `Diagnosis Count` |
| 26 | Why is occupancy down this month? | Census snapshot | Decomposition tree (drill by unit→service line) | `Occupancy %` |
| 27 | Summarize this department's KPIs for the report page | Any | Smart narrative | n/a |

## Education

Underlying facts: **applicant pipeline** accumulating snapshot (one row per applicant, 6+ milestone dates); **course registration** factless fact (one row per student per course per term, artificial count = 1); **enrollment** periodic snapshot; **student attendance** factless fact.

| # | Question | Fact / grain | Power BI visual | DAX measure |
|---|----------|--------------|-----------------|-------------|
| 1 | What is the student enrollment trend by program? | Enrollment snapshot | Line chart (program legend) | `[Enrolled]` (semi-additive — see services-dax-library.md; use LASTNONBLANK, never SUM across dates) |
| 2 | How is enrollment distributed across programs over time? | Enrollment snapshot | Stacked area chart | `Enrolled` |
| 3 | What is total enrollment vs target? | Enrollment snapshot (semi-additive) | Gauge chart | `Enrolled` (period-end) vs target |
| 4 | What is the admissions funnel (inquiry→visit→applied→admitted→enrolled)? | Applicant accumulating snapshot | Funnel chart | `Applicants at Stage` |
| 5 | What is the yield rate (enrolled / admitted)? | Applicant snapshot | Card | `Yield % = DIVIDE([Enrolled], [Admitted])` |
| 6 | What is the admit rate (admitted / applied)? | Applicant snapshot | Card | `Admit % = DIVIDE([Admitted], [Applied])` |
| 7 | What factors influence whether an admit enrolls? | Applicant snapshot | Key influencers (target: Enrolled) | `Enrolled Applicants` |
| 8 | How many days from application to decision? | Applicant snapshot | Card | `Avg Days To Decision = AVERAGE lag` |
| 9 | What is the application volume trend? | Applicant snapshot | Line chart | `Applied` |
| 10 | What is the course registration rate (eligible→registered→completed)? | Course registration factless | Funnel chart | `Registration Count = SUM(Reg[RegistrationCount])` |
| 11 | Which courses have low enrollment? | Course registration factless | Table with conditional formatting (red < min) | `Registration Count` per course |
| 12 | Which courses are most popular? | Course registration factless | Bar chart | `Registration Count` |
| 13 | What is enrollment by department/college? | Course registration factless | Treemap (college→department→course) | `Registration Count` |
| 14 | What is the student-to-faculty ratio by department? | Registration + faculty | Bar chart | `Student-Faculty Ratio = DIVIDE([Students],[Faculty Count])` |
| 15 | What is the retention rate term-over-term? | Enrollment snapshot | Line chart | `Retention % = DIVIDE([Retained Students],[Students Prior Term])` |
| 16 | What factors drive student attrition? | Enrollment snapshot | Key influencers (target: Withdrew) | `Withdrawals` |
| 17 | What is the graduation rate by cohort? | Enrollment snapshot | Bar chart | `Grad Rate % = DIVIDE([Graduates],[Cohort Size])` |
| 18 | What is class attendance rate by course? | Attendance factless | Bar chart | `Attendance % = DIVIDE([Attended],[Enrolled])` |
| 19 | How does attendance trend over the term? | Attendance factless | Line chart | `Attendance %` |
| 20 | What is facility/classroom utilization? | Facility utilization factless | KPI | `Room Utilization % = DIVIDE([Utilized Blocks],[Available Blocks])` |
| 21 | Which classrooms are underused by time block? | Facility utilization factless | Matrix (room × time block, conditional formatting) | `Utilization Flag SUM` |
| 22 | How does enrollment compare by demographic segment? | Enrollment snapshot | Matrix (segment × program) | `Enrolled` |
| 23 | Where do applicants come from geographically? | Applicant snapshot | Filled map (choropleth) | `Applied` by region |
| 24 | What is financial aid offered vs accepted? | Applicant snapshot | Combo chart (column offered, line accept %) | `Aid Offered`, `Aid Accept %` |
| 25 | Which programs are gaining/losing share over terms? | Enrollment snapshot | Ribbon chart | `Enrolled` |
| 26 | Why is retention down this term? | Enrollment snapshot | Decomposition tree (drill by program→cohort) | `Retention %` |

## Transportation / Airlines

Underlying facts: **flight segment** transaction grain (one row per boarding pass / segment); **leg** operational grain; **cargo shipment** leg grain; role-playing airport/date/time dimensions; city-pair route dimension.

| # | Question | Fact / grain | Power BI visual | DAX measure |
|---|----------|--------------|-----------------|-------------|
| 1 | What is on-time performance by route? | Flight segment/leg | Bar chart | `On-Time % = DIVIDE([On-Time Flights],[Total Flights])` |
| 2 | Which destinations have the worst on-time %? | Flight leg | Filled map (choropleth by destination) | `On-Time %` |
| 3 | What is the on-time % trend? | Flight leg | Line chart | `On-Time %` |
| 4 | What is the load factor trend (% seats filled)? | Flight segment | Line chart | `Load Factor % = DIVIDE([Passengers],[Seats Available])` |
| 5 | What is load factor by route? | Flight segment | Bar chart | `Load Factor %` |
| 6 | How do flights and load factor relate over time? | Flight segment | Combo chart (column flights, line load factor) | `Total Flights`, `Load Factor %` |
| 7 | What is the average departure delay by airport? | Flight leg | Bar chart | `Avg Dep Delay = AVERAGE(Leg[DepDelayMin])` |
| 8 | What is the distribution of delay minutes? | Flight leg | Column chart (delay bins) | `Flight Count` |
| 9 | What is total passenger revenue trend? | Flight segment | Line chart | `Pax Revenue = SUM(Segment[BaseFareRevenue])` |
| 10 | Which routes are most profitable? | Flight segment + cost | Waterfall chart (revenue→costs→profit) or treemap | `Route Profit` |
| 11 | What is revenue per available seat mile (RASM)? | Flight segment | Card | `RASM = DIVIDE([Pax Revenue],[Available Seat Miles])` |
| 12 | What is yield (revenue per passenger mile)? | Flight segment | Card | `Yield = DIVIDE([Pax Revenue],[Revenue Pax Miles])` |
| 13 | How is revenue split by class of service? | Flight segment | Donut chart | `Pax Revenue` by class |
| 14 | What is passenger volume by trip origin/destination? | Flight segment | Matrix (origin × destination) | `Passengers` |
| 15 | Where are passengers actually going (trip-level)? | Flight segment (trip O/D) | Filled map | `Passengers` by trip destination |
| 16 | What is the cancellation rate by route? | Flight leg | Bar chart | `Cancel % = DIVIDE([Cancelled],[Scheduled])` |
| 17 | How many flights at each operational stage (scheduled→boarded→departed→arrived)? | Flight leg | Funnel chart | `Flights at Stage` |
| 18 | What is the relationship between fare and load factor? | Flight segment | Scatter / bubble (x: avg fare, y: load factor, size: pax) | `Avg Fare`, `Load Factor %` |
| 19 | What is frequent-flyer mix by tier? | Segment + passenger mini-dim | Bar chart or donut chart | `Passengers` by tier |
| 20 | What is ancillary revenue contribution? | Flight segment | Waterfall chart | `Ancillary Revenue` |
| 21 | What is the connection/misconnect rate? | Flight segment (trip) | Card | `Misconnect % = DIVIDE([Misconnects],[Connections])` |
| 22 | What is cargo volume/weight by route? | Cargo leg | Bar chart | `Cargo Weight = SUM(Cargo[Weight])` |
| 23 | What is mishandled-baggage rate? | Flight segment | Card | `Mishandled Bag Rate = DIVIDE([Mishandled Bags],[Total Bags])` |
| 24 | Which routes are gaining/losing rank by revenue? | Flight segment | Ribbon chart | `Pax Revenue` |
| 25 | What factors drive flight delays? | Flight leg | Key influencers (target: Delayed) | `Delayed Flights` |
| 26 | Why did load factor drop on this route? | Flight segment | Decomposition tree (drill by date→fare basis→channel) | `Load Factor %` |

## Professional Services

Underlying facts: **timesheet** transaction grain (one row per time entry); **project** snapshot; **invoice** transaction grain. Key concept: billable vs non-billable hours, utilization, realization.

| # | Question | Fact / grain | Power BI visual | DAX measure |
|---|----------|--------------|-----------------|-------------|
| 1 | What is billable utilization by consultant? | Timesheet | Bar chart with target reference line | `Utilization % = DIVIDE([Billable Hours],[Available Hours])` |
| 2 | How does utilization trend over time? | Timesheet | Line chart | `Utilization %` |
| 3 | What is firm-wide utilization vs target? | Timesheet | KPI or gauge chart | `Utilization %` |
| 4 | What is utilization by practice/department? | Timesheet | Matrix (practice × month, conditional formatting) | `Utilization %` |
| 5 | What is the realization rate (billed / standard value)? | Timesheet + invoice | Card | `Realization % = DIVIDE([Billed Amount],[Standard Value])` |
| 6 | What is the project profitability (revenue − cost)? | Project / invoice | Waterfall chart | `Project Profit` |
| 7 | How do projects compare on margin? | Project | Bar chart | `Margin % = DIVIDE([Project Profit],[Project Revenue])` |
| 8 | What is revenue trend by service line? | Invoice | Stacked area chart | `Revenue = SUM(Invoice[Amount])` |
| 9 | Which clients generate the most revenue? | Invoice | Bar chart or treemap | `Revenue` by client |
| 10 | What is the billable vs non-billable hours split? | Timesheet | Donut chart or stacked column chart | `Billable Hours`, `Non-Billable Hours` |
| 11 | What is average bill rate by role? | Timesheet | Bar chart | `Avg Bill Rate = DIVIDE([Billed Amount],[Billable Hours])` |
| 12 | What is WIP (work in progress) by project? | Project snapshot | Table with conditional formatting | `WIP = [Unbilled Value]` |
| 13 | What is the project pipeline by stage (lead→proposal→won→delivery)? | Opportunity accumulating snapshot | Funnel chart | `Opportunities at Stage` |
| 14 | What is the win rate? | Opportunity snapshot | Card | `Win % = DIVIDE([Won],[Decided]) ` |
| 15 | How long is the average sales cycle? | Opportunity snapshot | Card | `Avg Cycle Days = AVERAGE lag` |
| 16 | What is days sales outstanding (DSO)? | Invoice | Card | `DSO = DIVIDE([AR Balance] * 365,[Revenue])` |
| 17 | Which invoices are overdue? | Invoice | Table with conditional formatting (aging) | `Days Overdue` |
| 18 | What is headcount trend? | HR snapshot (semi-additive) | Line chart of average | `Avg Headcount = AVERAGE(HR[Headcount])` |
| 19 | What is revenue per employee? | Invoice + HR | Card | `Revenue Per FTE = DIVIDE([Revenue],[Avg Headcount])` |
| 20 | What factors drive low project margin? | Project | Key influencers (target: Low Margin flag) | `Low Margin Projects` |
| 21 | Which consultants are over/under target utilization? | Timesheet | Bar chart with conditional formatting | `Utilization %` vs target |
| 22 | What is the relationship between utilization and realization? | Timesheet + invoice | Scatter / bubble | `Utilization %`, `Realization %` |
| 23 | How is revenue split across clients (concentration)? | Invoice | Treemap | `Revenue` |
| 24 | Which service lines are gaining/losing rank? | Invoice | Ribbon chart | `Revenue` |
| 25 | Why did utilization fall last quarter? | Timesheet | Decomposition tree (drill by practice→role→consultant) | `Utilization %` |

## Cross-sub-industry notes

- **Semi-additive measures** (occupied beds, enrolled headcount, active service lines, headcount): present current value in a **card** or **gauge chart**, or trend the **average** in a **line chart**. Never SUM across dates.
- **Pipeline stage questions** across all sub-industries (provisioning, claims, admissions, project pipeline, flight operations) use a **funnel chart**; the lag between milestone dates is a **card** or trended in a **line chart**.
- **"What factors / why" questions** → **Key influencers** (single outcome variable) or **decomposition tree** (drill a metric by dimensions). Use these instead of forcing a static chart.
- **Geographic questions** → **filled map (choropleth)**, **shape map**, **ArcGIS map**, or **Azure map**.
- **Part-to-whole** with many categories → **treemap** (hierarchical); with few (<8) → **donut** or **pie chart**.
- **Two measures where one is a ratio** → **combo chart** (count as column, ratio as line).
