# Services DAX Measure Library

Reusable, syntactically valid DAX for services KPIs. Adapt table/column names to the actual model. Replace `'TableName'[Column]` placeholders as needed.

Additivity guardrails applied throughout:
- **Ratios** (occupancy, utilization, load factor, churn, on-time, fill rate, collection) use `DIVIDE(SUM(num), SUM(denom))` — never `AVERAGE` of a precomputed percentage.
- **Semi-additive snapshots** (occupied beds, headcount, enrollment, active lines) use `AVERAGE`/period-end — never `SUM` across dates.
- **Lags** are milestone-date differences computed per row, then aggregated with `AVERAGE`.

## Contents
- [Generic counts and the artificial factless count](#generic-counts-and-the-artificial-factless-count)
- [Call Center / Telecom](#call-center--telecom)
- [Healthcare](#healthcare)
- [Education](#education)
- [Transportation / Airlines](#transportation--airlines)
- [Professional Services](#professional-services)
- [Reusable patterns: lag, semi-additive, ratio](#reusable-patterns-lag-semi-additive-ratio)

---

## Generic counts and the artificial factless count

```dax
-- Transaction-grain count (calls, encounters, flights, time entries)
Row Count = COUNTROWS ( 'Fact' )

-- Factless fact: aggregate the artificial RegistrationCount = 1 column,
-- NOT COUNT of a key (per Kimball factless-fact guidance)
Registration Count = SUM ( 'CourseRegistration'[RegistrationCount] )
```

## Call Center / Telecom

```dax
Call Count = COUNTROWS ( 'CDR' )

Abandoned Calls =
CALCULATE ( [Call Count], 'CDR'[Disposition] = "Abandoned" )

Abandon Rate % = DIVIDE ( [Abandoned Calls], [Call Count] )

-- Average handle time (seconds) per answered call
AHT Seconds =
DIVIDE ( SUM ( 'CDR'[HandleSeconds] ), [Calls Answered] )

Calls Answered =
CALCULATE ( [Call Count], 'CDR'[Answered] = TRUE () )

-- Service level: % of calls answered within the SLA threshold
Calls Within SLA =
CALCULATE ( [Call Count], 'CDR'[AnswerSeconds] <= 'SLA'[ThresholdSeconds] )

Service Level % = DIVIDE ( [Calls Within SLA], [Call Count] )

-- First-call resolution
First Call Resolved =
CALCULATE ( [Call Count], 'CDR'[ResolvedFirstCall] = TRUE () )

FCR % = DIVIDE ( [First Call Resolved], [Call Count] )

-- Agent occupancy: productive time over logged-in time
Agent Occupancy % =
DIVIDE (
    SUM ( 'AgentState'[HandleSeconds] ) + SUM ( 'AgentState'[HoldSeconds] ),
    SUM ( 'AgentState'[LoggedInSeconds] )
)

-- Churn rate over the period (uses customers active at period start)
Churned Customers =
CALCULATE ( DISTINCTCOUNT ( 'Billing'[CustomerKey] ), 'Billing'[Churned] = TRUE () )

Customers At Start =
CALCULATE (
    DISTINCTCOUNT ( 'Billing'[CustomerKey] ),
    DATEADD ( 'Date'[Date], -1, MONTH )
)

Churn Rate % = DIVIDE ( [Churned Customers], [Customers At Start] )

Total Billed = SUM ( 'Billing'[TotalBilled] )

-- Active service lines: period-end (semi-additive) count
Active Service Lines =
CALCULATE (
    DISTINCTCOUNT ( 'Billing'[ServiceLineKey] ),
    LASTNONBLANK ( 'Date'[Date], 1 )
)

ARPU = DIVIDE ( [Total Billed], [Active Service Lines] )

Data Volume GB = DIVIDE ( SUM ( 'CDR'[DataMB] ), 1024 )

Dropped Calls = CALCULATE ( [Call Count], 'CDR'[Dropped] = TRUE () )

Dropped Call % = DIVIDE ( [Dropped Calls], [Call Count] )

Net Adds = [Gross Adds] - [Churned Customers]

Gross Adds =
CALCULATE ( DISTINCTCOUNT ( 'Billing'[ServiceLineKey] ), 'Billing'[IsNewLine] = TRUE () )
```

## Healthcare

```dax
Admissions = COUNTROWS ( 'Encounter' )

Discharges =
CALCULATE ( [Admissions], NOT ISBLANK ( 'Encounter'[DischargeDateKey] ) )

-- Average daily census: occupied beds are SEMI-ADDITIVE (avg across dates)
Avg Occupied Beds = AVERAGE ( 'Census'[OccupiedBeds] )

Total Beds = SELECTEDVALUE ( 'Unit'[TotalBeds] )

Occupancy % = DIVIDE ( [Avg Occupied Beds], [Total Beds] )

-- Average length of stay
ALOS Days = AVERAGE ( 'Encounter'[LengthOfStayDays] )

-- Readmission within 30 days
Readmissions =
CALCULATE ( [Admissions], 'Encounter'[Readmit30Day] = TRUE () )

Readmission % = DIVIDE ( [Readmissions], [Discharges] )

-- Diagnosis frequency via bridge (multivalued, no weighting per Kimball)
Diagnosis Count = COUNTROWS ( 'DiagnosisBridge' )

No-Shows = CALCULATE ( [Admissions], 'Encounter'[Status] = "No-Show" )

Scheduled Appts = CALCULATE ( [Admissions], 'Encounter'[Scheduled] = TRUE () )

No-Show % = DIVIDE ( [No-Shows], [Scheduled Appts] )

-- Claims: billed, paid, collection, denial
Total Billed Claims = SUM ( 'Claim'[BilledAmount] )

Total Paid Claims = SUM ( 'Claim'[TotalPaidAmount] )

Collection % = DIVIDE ( [Total Paid Claims], [Total Billed Claims] )

Submitted Claims = COUNTROWS ( 'Claim' )

Denied Claims = CALCULATE ( [Submitted Claims], 'Claim'[Status] = "Denied" )

Denial % = DIVIDE ( [Denied Claims], [Submitted Claims] )

-- Accumulating-snapshot lag: billing date to primary payment date
Avg Days To Payment =
AVERAGEX (
    FILTER ( 'Claim', NOT ISBLANK ( 'Claim'[PrimaryPaymentDate] ) ),
    DATEDIFF ( 'Claim'[BillingDate], 'Claim'[PrimaryPaymentDate], DAY )
)

-- Operating room / facility utilization (coverage factless: flag = 1 utilized)
Utilized Minutes =
CALCULATE ( SUM ( 'FacilityUtilization'[DurationMinutes] ), 'FacilityUtilization'[UtilizationFlag] = 1 )

Available Minutes = SUM ( 'FacilityUtilization'[DurationMinutes] )

OR Utilization % = DIVIDE ( [Utilized Minutes], [Available Minutes] )
```

## Education

```dax
-- Enrollment is SEMI-ADDITIVE across time: present period-end headcount
Enrolled =
CALCULATE ( SUM ( 'Enrollment'[EnrolledCount] ), LASTNONBLANK ( 'Date'[Date], 1 ) )

-- Applicant pipeline counts (accumulating snapshot)
Applied = CALCULATE ( COUNTROWS ( 'Applicant' ), NOT ISBLANK ( 'Applicant'[AppSubmittedDate] ) )

Admitted = CALCULATE ( COUNTROWS ( 'Applicant' ), 'Applicant'[DecisionStatus] = "Admitted" )

Enrolled Applicants = CALCULATE ( COUNTROWS ( 'Applicant' ), NOT ISBLANK ( 'Applicant'[EnrolledDate] ) )

Admit % = DIVIDE ( [Admitted], [Applied] )

Yield % = DIVIDE ( [Enrolled Applicants], [Admitted] )

Avg Days To Decision =
AVERAGEX (
    FILTER ( 'Applicant', NOT ISBLANK ( 'Applicant'[DecisionDate] ) ),
    DATEDIFF ( 'Applicant'[AppSubmittedDate], 'Applicant'[DecisionDate], DAY )
)

-- Course registration (factless, artificial count)
Registration Count = SUM ( 'CourseRegistration'[RegistrationCount] )

Students = DISTINCTCOUNT ( 'CourseRegistration'[StudentKey] )

Faculty Count = DISTINCTCOUNT ( 'CourseRegistration'[FacultyKey] )

Student-Faculty Ratio = DIVIDE ( [Students], [Faculty Count] )

-- Retention term over term
Retained Students =
CALCULATE ( [Students], 'Enrollment'[RetainedNextTerm] = TRUE () )

Students Prior Term = CALCULATE ( [Students], PREVIOUSMONTH ( 'Date'[Date] ) )

Retention % = DIVIDE ( [Retained Students], [Students Prior Term] )

-- Graduation rate by cohort
Graduates = CALCULATE ( [Students], 'Enrollment'[Graduated] = TRUE () )

Grad Rate % = DIVIDE ( [Graduates], [Cohort Size] )

Cohort Size = CALCULATE ( [Students], ALLEXCEPT ( 'Enrollment', 'Enrollment'[CohortKey] ) )

-- Attendance (factless flag)
Attended = CALCULATE ( SUM ( 'Attendance'[AttendanceCount] ), 'Attendance'[AttendanceFlag] = 1 )

Attendance % = DIVIDE ( [Attended], [Registration Count] )

-- Classroom utilization (coverage factless)
Utilized Blocks = CALCULATE ( SUM ( 'RoomUtilization'[UtilizationFlag] ) )

Available Blocks = COUNTROWS ( 'RoomUtilization' )

Room Utilization % = DIVIDE ( [Utilized Blocks], [Available Blocks] )
```

## Transportation / Airlines

```dax
Total Flights = COUNTROWS ( 'FlightLeg' )

On-Time Flights =
CALCULATE ( [Total Flights], 'FlightLeg'[ArrDelayMin] <= 15 )

On-Time % = DIVIDE ( [On-Time Flights], [Total Flights] )

Cancelled = CALCULATE ( COUNTROWS ( 'FlightLeg' ), 'FlightLeg'[Status] = "Cancelled" )

Scheduled Flights = COUNTROWS ( 'FlightLeg' )

Cancel % = DIVIDE ( [Cancelled], [Scheduled Flights] )

Avg Dep Delay = AVERAGE ( 'FlightLeg'[DepDelayMin] )

-- Load factor: passengers over seats available (revenue grain = segment)
Passengers = SUM ( 'FlightSegment'[PassengerCount] )

Seats Available = SUM ( 'FlightSegment'[SeatsAvailable] )

Load Factor % = DIVIDE ( [Passengers], [Seats Available] )

Pax Revenue = SUM ( 'FlightSegment'[BaseFareRevenue] )

Ancillary Revenue = SUM ( 'FlightSegment'[AncillaryFees] )

Available Seat Miles = SUMX ( 'FlightSegment', 'FlightSegment'[SeatsAvailable] * 'FlightSegment'[SegmentMiles] )

Revenue Pax Miles = SUMX ( 'FlightSegment', 'FlightSegment'[PassengerCount] * 'FlightSegment'[SegmentMiles] )

-- Revenue per available seat mile / yield
RASM = DIVIDE ( [Pax Revenue], [Available Seat Miles] )

Yield = DIVIDE ( [Pax Revenue], [Revenue Pax Miles] )

Avg Fare = DIVIDE ( [Pax Revenue], [Passengers] )

-- Route profit (combine with cost fact via conformed route dimension)
Route Profit = [Pax Revenue] + [Ancillary Revenue] - SUM ( 'FlightCost'[OperatingCost] )

Mishandled Bags = CALCULATE ( SUM ( 'Baggage'[BagCount] ), 'Baggage'[Mishandled] = TRUE () )

Total Bags = SUM ( 'Baggage'[BagCount] )

Mishandled Bag Rate = DIVIDE ( [Mishandled Bags], [Total Bags] )

Cargo Weight = SUM ( 'CargoLeg'[Weight] )
```

## Professional Services

```dax
Billable Hours = CALCULATE ( SUM ( 'Timesheet'[Hours] ), 'Timesheet'[Billable] = TRUE () )

Non-Billable Hours = CALCULATE ( SUM ( 'Timesheet'[Hours] ), 'Timesheet'[Billable] = FALSE () )

Available Hours = SUM ( 'Timesheet'[AvailableHours] )

Utilization % = DIVIDE ( [Billable Hours], [Available Hours] )

Billed Amount = SUM ( 'Invoice'[Amount] )

Standard Value = SUMX ( 'Timesheet', 'Timesheet'[Hours] * RELATED ( 'Rate'[StandardRate] ) )

Realization % = DIVIDE ( [Billed Amount], [Standard Value] )

Avg Bill Rate = DIVIDE ( [Billed Amount], [Billable Hours] )

Project Revenue = SUM ( 'Invoice'[Amount] )

Project Cost = SUM ( 'Project'[LaborCost] ) + SUM ( 'Project'[Expenses] )

Project Profit = [Project Revenue] - [Project Cost]

Margin % = DIVIDE ( [Project Profit], [Project Revenue] )

Won = CALCULATE ( COUNTROWS ( 'Opportunity' ), 'Opportunity'[Stage] = "Won" )

Decided = CALCULATE ( COUNTROWS ( 'Opportunity' ), 'Opportunity'[Stage] IN { "Won", "Lost" } )

Win % = DIVIDE ( [Won], [Decided] )

Avg Cycle Days =
AVERAGEX (
    FILTER ( 'Opportunity', NOT ISBLANK ( 'Opportunity'[CloseDate] ) ),
    DATEDIFF ( 'Opportunity'[CreatedDate], 'Opportunity'[CloseDate], DAY )
)

AR Balance = SUM ( 'Invoice'[OpenBalance] )

DSO = DIVIDE ( [AR Balance] * 365, [Project Revenue] )

-- Headcount is SEMI-ADDITIVE: average across the period
Avg Headcount = AVERAGE ( 'HRSnapshot'[Headcount] )

Revenue Per FTE = DIVIDE ( [Project Revenue], [Avg Headcount] )
```

## Reusable patterns: lag, semi-additive, ratio

**Accumulating-snapshot lag** (any milestone pair). Filter out unreached milestones (TBD) before averaging:

```dax
Avg Lag Days =
AVERAGEX (
    FILTER ( 'Fact', NOT ISBLANK ( 'Fact'[EndDate] ) ),
    DATEDIFF ( 'Fact'[StartDate], 'Fact'[EndDate], DAY )
)
```

**Semi-additive period-end** (balance/headcount/census at last date in context):

```dax
Period-End Balance =
CALCULATE (
    SUM ( 'Snapshot'[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( SUM ( 'Snapshot'[Balance] ) ) )
)
```

**Semi-additive average across time** (preferred for "average daily" KPIs):

```dax
Avg Across Time = AVERAGE ( 'Snapshot'[Measure] )
```

**Generic ratio KPI** (occupancy, utilization, load factor, on-time, churn, fill rate):

```dax
Ratio KPI = DIVIDE ( SUM ( 'Fact'[Numerator] ), SUM ( 'Fact'[Denominator] ) )
```

Never compute a ratio as `AVERAGE('Fact'[PrecomputedPct])`; that averages the wrong grain. Always aggregate numerator and denominator separately, then divide.
