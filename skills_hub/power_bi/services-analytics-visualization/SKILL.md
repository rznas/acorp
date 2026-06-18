---
name: services-analytics-visualization
description: Recommends Power BI visualizations and DAX measures for services-industry analytical questions, mapping call detail records, patient encounters, student enrollment, flight/passenger activity, and timesheet billing to specific Power BI visuals. Use when building Power BI reports or dashboards for healthcare (hospital census, claims, diagnoses), education (enrollment, course registration), telecommunications (call centers, CDR, churn), transportation/airlines (on-time, load factor, routes), or professional services (consulting, legal, accounting utilization). Triggers include call center, telecom, churn, bed occupancy, admissions, load factor, on-time performance, billable utilization, occupancy rate, and service KPIs.
---

# Services Analytics Visualization

Recommend Power BI visualizations for services-industry business questions. Services facts share traits that drive visual choice: high-volume transaction grains (calls, encounters, flights), semi-additive snapshots (bed census, headcount), accumulating-snapshot pipelines (admissions, claims), and factless fact tables (course registration, facility utilization).

This is a **Layer 2 domain skill**. It builds on the Layer 1 foundation skills:
- `visualizing-dimensional-models` - fact-type → visual affinity, additivity, cardinality
- `mapping-questions-to-charts` - question-intent taxonomy (comparison, trend, part-to-whole, etc.)
- `understanding-fact-additivity` - SUM vs AVERAGE vs ratio measures

Read those first when the question is generic; this skill adds services-specific patterns and KPIs.

## Workflow

1. **Identify the sub-industry**: call-center/telecom, healthcare, education, transportation/airlines, or professional services.
2. **Classify the fact grain** (drives the visual):
   - **Transaction grain** (one row per call/encounter/flight/time-entry) → trends use **line chart**; rankings use **bar chart** / **column chart**.
   - **Periodic snapshot** with semi-additive measures (bed census, enrolled headcount) → **card**, **gauge chart**, **KPI**; average across time, never SUM.
   - **Accumulating snapshot** pipeline (admissions, claims, enrollment milestones) → **funnel chart** for stage progression; **card** for milestone lag.
   - **Factless fact** (course registration, facility utilization) → counts via an artificial `= 1` measure; **table** with conditional formatting, **funnel chart** for eligibility→completion.
3. **Map the question** to a visual and DAX measure using `reference/services-question-map.md`.
4. **Compose the dashboard** for the audience using `reference/services-dashboards.md`.
5. **Author measures** using `reference/services-dax-library.md`. Validate additivity: occupancy, utilization, load factor, churn, on-time %, and fill rate are all **non-additive ratios** computed as `DIVIDE(numerator, denominator)`, never averaged from a precomputed column.

## Reference files

- **Question → visualization map** (25+ questions per sub-industry, each with the exact Power BI visual and a DAX measure): `reference/services-question-map.md`
- **Dashboard templates** (call-center manager, hospital administrator, enrollment officer, flight ops, professional-services partner): `reference/services-dashboards.md`
- **DAX measure library** (utilization, occupancy, load factor, churn rate, on-time %, average handle time, lag, fill rate): `reference/services-dax-library.md`

## Allowed Power BI visuals

Use ONLY visuals confirmed in the Power BI visualization-types documentation. For services work the relevant set is:

| Visual | Primary services use |
|--------|----------------------|
| Line chart | Trends over time (call volume, enrollment, load factor, occupancy %) |
| Area chart (basic / stacked) | Cumulative trend, enrollment by program over time |
| Column chart | Volume by category/time bin (calls by hour, admissions by department) |
| Bar chart | Ranked comparison (consultants by utilization, routes by on-time %) |
| Combo chart | Two measures, one a ratio (count column + ratio line: e.g., flights + load factor) |
| Card (single number) | Single semi-additive/ratio KPI (current occupancy, churn rate, AHT) |
| Multirow card | A few KPIs together |
| KPI | Progress vs goal with trend (utilization vs target) |
| Gauge chart | Current value vs capacity/target (bed occupancy vs beds, backlog vs capacity) |
| Funnel chart | Pipeline stages (inquiry→enrolled, reported→closed claim, eligible→completed) |
| Waterfall chart | Running total / variance (route profitability, project P&L, claim balance) |
| Treemap | Hierarchical part-to-whole (diagnosis chapter→code, program→course) |
| Pie / Donut chart | Part-to-whole when fewer than ~8 categories (call dispositions, coverage type) |
| Matrix | Multi-dimension grid with drill-down + conditional formatting (dept × month, segment × tier) |
| Table | Detail rows with conditional formatting (low-enrollment courses, delayed claims) |
| Scatter / bubble | Two-measure relationship (handle time vs satisfaction, fare vs load factor) |
| Filled map (choropleth) | Geographic intensity (on-time % by destination, calls by region) |
| Shape map / ArcGIS map / Azure map | Regional comparison and richer spatial analysis |
| Key influencers | "What factors drive [outcome]?" (churn, no-show, readmission) |
| Decomposition tree | AI root-cause drill (why is occupancy/profit down? drill by dimension) |
| Smart narrative | Auto-generated text summary of a report page |
| Q&A visual | Ad-hoc natural-language questions over a well-modeled semantic model |
| Ribbon chart | Rank change over time (top routes/programs shifting by period) |
| Slicers | Date range, department, segment, region filtering |

Do not invent visuals. There is no built-in "histogram" or "Sankey"; for distributions use a **column chart** over binned categories, and for stage flow use a **funnel chart**.

## Critical correctness rules

- **Semi-additive snapshots** (occupied beds, enrolled headcount, accounts in service): use `AVERAGE` or a period-end pattern across time, never `SUM` across dates. Display in **card**, **gauge chart**, or **line chart** of the average.
- **Ratio KPIs** (occupancy %, utilization %, load factor %, churn %, on-time %, fill rate): compute as `DIVIDE(SUM(num), SUM(denom))`. Show in **card**, **KPI**, **gauge chart**, or as the line in a **combo chart**. Never `AVERAGE` a per-row percentage.
- **Accumulating-snapshot lags** (days admission→discharge, claim→payment, order→delivery): compute milestone-date differences as DAX measures; show as **card** or trend them in a **line chart**.
- **Factless fact tables**: aggregate the artificial `= 1` count measure (e.g., `SUM(Registration[RegistrationCount])`), not row counts of keys.

---

*Provenance: implements SKILL 7 (Services Analytics Visualization, Layer 2) of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`. Visualization names verified against `powerbi-docs/visuals/power-bi-visualization-types-for-reports-and-q-and-a.md`; domain patterns from `data_warehouse_kimball/SKILL/industry-services-patterns/SKILL.md`. Depends on Layer 1 skills visualizing-dimensional-models, mapping-questions-to-charts, understanding-fact-additivity.*
