# Services Dashboard Templates

Audience-specific layouts. Every visual named below is confirmed in the Power BI visualization-types doc. Layout convention: most important KPIs top-left (Western reading order); slicers in a left rail or top bar; cross-filtering enabled so selecting a category filters the page. Keep visuals per page modest (target under ~20) for performance.

## Contents
- [Call-Center Manager Dashboard](#call-center-manager-dashboard)
- [Hospital Administrator Dashboard](#hospital-administrator-dashboard)
- [Enrollment Officer Dashboard](#enrollment-officer-dashboard)
- [Flight Operations Dashboard](#flight-operations-dashboard)
- [Professional-Services Partner Dashboard](#professional-services-partner-dashboard)
- [Composition rules](#composition-rules)

---

## Call-Center Manager Dashboard

Operational, refreshed intraday. Source: CDR transaction fact + billing snapshot.

- **Top KPI row** (`multirow card` or four `card` visuals): Calls handled today, Average handle time (AHT), Abandonment rate %, Service level %.
- **Service level vs target** (`gauge chart`): `Service Level %` vs SLA target.
- **Call volume by hour** (`column chart`): hour-of-day bins; lets manager match staffing to demand.
- **Volume + abandonment** (`combo chart`): column `Offered Calls`, line `Abandon Rate %`.
- **Agent performance** (`bar chart`): `Call Count` per agent, with a target reference line and conditional formatting for occupancy.
- **Call disposition mix** (`donut chart`): when fewer than ~8 dispositions; otherwise `column chart`.
- **Churn drivers** (`key influencers`): target = Churned, explained by contract type, tenure, support-call count.
- **Slicers**: date range, queue/skill, region.

Drill: from the agent `bar chart`, drill through to a `table` of that agent's individual calls.

## Hospital Administrator Dashboard

Mixed operational/strategic. Source: census periodic snapshot (semi-additive), encounter transaction fact, claims accumulating snapshot.

- **Top KPI row** (`card` x4): Average daily census, Bed occupancy %, Average length of stay (ALOS), 30-day readmission %.
- **Occupancy vs capacity** (`gauge chart`): `Avg Occupied Beds` vs `Total Beds`. Critical: occupancy uses `AVERAGE`/period-end, never SUM across dates.
- **Occupancy trend** (`line chart`): `Occupancy %` over time, with anomaly detection.
- **Admissions by department** (`bar chart`) and **admissions trend** (`line chart`).
- **Claims pipeline** (`funnel chart`): treatment → billed → insurance paid → patient paid → closed.
- **Billed vs collected** (`combo chart`): column `Total Billed`, line `Collection %`.
- **Top diagnoses** (`treemap`): ICD chapter → diagnosis code (from diagnosis bridge).
- **Aging claims** (`table` with conditional formatting): days outstanding, red beyond target.
- **Patient volume by region** (`filled map`).
- **Slicers**: date range, facility/unit, payer, service line.

Drill: `decomposition tree` on `Occupancy %` (drill by unit → service line) for root-cause when occupancy dips.

## Enrollment Officer Dashboard

Strategic + pipeline. Source: applicant accumulating snapshot, course-registration factless fact, enrollment periodic snapshot.

- **Top KPI row** (`card` x4): Total enrolled, Yield % (enrolled/admitted), Admit %, Application volume.
- **Enrollment vs target** (`gauge chart`): `Enrolled` (period-end, semi-additive) vs target.
- **Admissions funnel** (`funnel chart`): inquiry → campus visit → applied → admitted → enrolled.
- **Enrollment trend by program** (`stacked area chart` or `line chart`).
- **Program share over terms** (`ribbon chart`): which programs gain/lose rank.
- **Yield drivers** (`key influencers`): target = Enrolled (among admits), explained by program, aid offered, distance, test score.
- **Low-enrollment courses** (`table` with conditional formatting): red below minimum.
- **Applicant geography** (`filled map`).
- **Aid offered vs accepted** (`combo chart`): column `Aid Offered`, line `Aid Accept %`.
- **Slicers**: term, program/college, applicant source, region.

Drill: from program `bar chart`/area, drill through to a `table` of applicants at each milestone.

## Flight Operations Dashboard

Operational. Source: flight leg (operational) + flight segment (revenue) transaction facts; role-playing airport/date/time dimensions.

- **Top KPI row** (`card` x4): On-time % today, Load factor %, Cancellation rate %, Average departure delay.
- **On-time vs target** (`gauge chart`): `On-Time %` vs target.
- **On-time by destination** (`filled map`): choropleth colored by `On-Time %`.
- **Load factor trend** (`line chart`) and **flights + load factor** (`combo chart`: column `Total Flights`, line `Load Factor %`).
- **Delay distribution** (`column chart`): delay-minute bins.
- **On-time by route** (`bar chart`) with conditional formatting (green >85%, yellow, red).
- **Operational stage flow** (`funnel chart`): scheduled → boarded → departed → arrived.
- **Delay drivers** (`key influencers`): target = Delayed, explained by origin airport, aircraft, time of day, weather code.
- **Route revenue rank shifts** (`ribbon chart`).
- **Slicers**: flight date, origin airport, destination airport, aircraft type.

Drill: `decomposition tree` on `Avg Dep Delay` (drill by airport → aircraft → delay reason).

## Professional-Services Partner Dashboard

Strategic. Source: timesheet transaction fact, invoice transaction fact, project/opportunity snapshots.

- **Top KPI row** (`card` x4): Firm utilization %, Realization %, Revenue, Days sales outstanding (DSO).
- **Utilization vs target** (`KPI` or `gauge chart`): `Utilization %` vs target with trend.
- **Utilization by practice** (`matrix`): practice × month, conditional formatting (red below target).
- **Utilization by consultant** (`bar chart`): with target reference line.
- **Revenue trend by service line** (`stacked area chart`).
- **Project profitability** (`waterfall chart`): revenue → labor cost → expenses → profit; or `bar chart` of `Margin %` by project.
- **Pipeline** (`funnel chart`): lead → proposal → won → delivery.
- **Client concentration** (`treemap`): revenue by client.
- **Utilization vs realization** (`scatter`/`bubble`): identifies consultants busy but under-realizing.
- **Overdue invoices** (`table` with conditional formatting on aging).
- **Slicers**: period, practice, client, role.

Drill: `decomposition tree` on `Utilization %` (drill by practice → role → consultant).

## Composition rules

- **Semi-additive KPIs** (census, enrollment, headcount, active lines) belong in `card`/`gauge chart`/`KPI`, or as the AVERAGE in a `line chart` — never SUM across time on a dashboard tile.
- **Ratio KPIs** (occupancy, utilization, load factor, on-time, churn, realization) are computed measures (`DIVIDE`); show them in `card`/`KPI`/`gauge chart` or as the line of a `combo chart`.
- **Pipeline** = `funnel chart`; **variance / running total** = `waterfall chart`; **rank change over time** = `ribbon chart`.
- **AI tiles**: add a `key influencers` and/or `decomposition tree` to every dashboard so users can answer "why" without a new report; optionally a `smart narrative` to auto-summarize the page and a `Q&A visual` for ad-hoc questions.
- **Slicers** drive the whole page; place a date-range slicer on every dashboard since all services facts are time-anchored.
