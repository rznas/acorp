---
name: optimizing-model-performance
description: Diagnoses and fixes slow or oversized Power BI solutions by reducing semantic model size, triaging with Performance Analyzer, adding aggregation tables, tuning expensive DAX and visuals, preserving query folding for faster refresh, and separating reports from models. Use when reports load or filter slowly, a model is too large for its capacity, data refresh is slow, DAX queries or visuals are the bottleneck, or you need to tune Import, DirectQuery, or Composite model performance.
---

# Optimizing Power BI Model Performance

## Overview

Power BI performance problems live in distinct architectural layers, and the fix depends on which layer is slow. This skill follows the Microsoft optimization guidance: optimize at the **data source**, the **data model (semantic model)**, the **visualizations**, and the **environment** (capacity, gateway, network). The discipline is: **measure first, then fix the layer that is actually slow.** Most authors guess; Performance Analyzer tells you whether a visual is slow because of its DAX query or its rendering.

The biggest, cheapest win is almost always **reducing the Import model size** — every column dropped and every row not loaded makes refresh faster, queries faster, and memory pressure lower. A well-shaped star schema is what makes the VertiPaq engine compress and scan efficiently.

## When to Use

Use this skill when:

- A report is slow to open, or slow to update when users move slicers/filters.
- An Import model is too large for its capacity (shared capacity caps models at 1 GB; Premium SKUs allow larger).
- Data refresh is slow or times out.
- You suspect a specific DAX measure or visual is the bottleneck.
- You are tuning a DirectQuery or Composite model whose queries hit the source too hard.
- You want one model reused by many reports (report/model separation).

**Dependencies (from the full-lifecycle plan):** build on `modeling-semantic-data` (relationships, star schema, cardinality, marked date table) and `authoring-dax-measures` (filter/row context, CALCULATE). For correct aggregation of balances and ratios while tuning measures, follow `understanding-fact-additivity` — never naive-SUM a semi-additive balance across time. For source-side tuning of a dimensional warehouse (indexing, partitioning, source aggregates), cross-reference the Kimball skill `physical-database-design-optimization`.

## The four optimization layers

| Layer | What you tune | Primary techniques |
|-------|---------------|--------------------|
| **Data source** | The relational DB / warehouse behind DirectQuery or feeding refresh | Indexes, materialized/computed columns, indexed views, distributed tables, a materialized date table, query folding |
| **Data model** | The semantic model (Import/DirectQuery/Composite) | Reduce size (drop columns, fewer rows, lower cardinality, disable auto date/time), star schema, aggregation tables, simple measures |
| **Visualizations** | Report pages, dashboards, paginated reports | Restrictive filters, fewer visuals per page, vet custom visuals, dashboard tile cache |
| **Environment** | Capacity, gateway, network | Manage/scale capacity, size the gateway, keep source+gateway+capacity close (low latency) |

## Optimization workflow

Copy this checklist and work top-down. Do not optimize blindly — step 1 tells you where the problem is.

```
Performance optimization progress:
- [ ] Step 1: Triage with Performance Analyzer (find the slow visual + whether DAX or rendering)
- [ ] Step 2: If DAX query is slow -> fix the model (size, star schema, aggregations) or the DAX
- [ ] Step 3: If visual display is slow -> reduce data shown / count of visuals / vet custom visuals
- [ ] Step 4: If refresh is slow -> reduce model size + preserve query folding
- [ ] Step 5: If DirectQuery is slow -> optimize source + model + report per DirectQuery guidance
- [ ] Step 6: Re-measure with Performance Analyzer; confirm the bottleneck moved or shrank
- [ ] Step 7: If still slow and on Premium/Fabric capacity -> check the Fabric Capacity Metrics app
```

**Step 1 — Triage.** In Power BI Desktop open the **Optimize** ribbon and select **Performance Analyzer** (in the service: **Edit** > **View** menu > **Performance Analyzer**). Select **Start recording**, interact with the report (or use **Refresh visuals**), then **Stop**. Each visual breaks down into **DAX query**, **Direct query**, **Visual display**, **Other**, and **Evaluated parameters** durations. This tells you whether the query or the rendering is the problem. Full triage procedure: see [performance-analyzer-triage.md](performance-analyzer-triage.md).

**Step 2 — Slow DAX query.** The model is doing too much work. In priority order: reduce model size and shape a star schema (see step 4 + the data-reduction table), add **aggregation tables** for large DirectQuery facts, then simplify the measure. Copy the visual's DAX with **Copy query** and run it in **DAX query view** to iterate.

**Step 3 — Slow visual display.** Apply the most restrictive filters, limit the number of visuals on the page, and **evaluate custom visual performance** (a poorly built custom visual can slow the whole report).

**Step 4 — Slow refresh / large model.** Apply data-reduction techniques and preserve **query folding** so transformations run at the source, not in the mashup engine.

**Step 5 — Slow DirectQuery.** Push work to the source and reduce per-page query volume. See the DirectQuery section below.

**Step 6/7 — Re-measure.** Performance work is a feedback loop. Re-run Performance Analyzer after each change; on Premium/Fabric, use the **Fabric Capacity Metrics app** to see capacity pressure that no report-side fix can solve.

## Reducing model size (the highest-leverage layer)

Import models are compressed by the **VertiPaq** engine (often ~10x; another ~20% on disk). Smaller models still win: they fit capacity limits, reduce memory contention/eviction, refresh faster, and evaluate calculations faster. Apply these techniques (full table with how/when/tradeoff in [data-reduction-techniques.md](data-reduction-techniques.md)):

- **Remove unnecessary columns** (vertical filtering) — keep only columns used for reporting or model structure. High-cardinality unused columns hurt most.
- **Remove unnecessary rows** (horizontal filtering) — filter by time (don't load all history; use a Power Query date parameter, even relative) and/or by entity.
- **Group by and summarize** — raise the fact grain (e.g., to month) for large size reductions; tradeoff is loss of detail.
- **Optimize column data types** — numeric columns use value encoding (best); convert text like `SO123456` to a whole number where possible; set such columns to `Do Not Summarize`.
- **Prefer Power Query (M) custom columns over DAX calculated columns** — DAX calculated columns compress less well and extend refresh.
- **Disable query load** for staging queries not needed in the model.
- **Disable Auto date/time** — it creates a hidden calculated date table per date column. Use one explicit, marked date table instead (see `applying-time-intelligence`).
- **Use DirectQuery storage mode for very large fact tables** — combine with summarized Import tables to form a Composite model.

> Star-schema shape matters for VertiPaq efficiency and for relationship-based aggregation hits. For source-side physical tuning (indexing, partitioning, pre-aggregated views), cross-reference Kimball `physical-database-design-optimization`.

## Aggregation tables

Aggregations cache data at an aggregated grain in memory to speed queries over large **DirectQuery** semantic models. Define them in **Model** view: right-click the aggregation table > **Manage aggregations**, then set **Summarization**, **Detail Table**, and **Detail Column** per column.

Key rules grounded in the docs:

- The **Detail Table must be DirectQuery** (not Import); the aggregation table itself is fastest as an **Import** table.
- **Relationship-based aggregations** (typical for star schemas): GroupBy entries are optional (except for DISTINCTCOUNT); aggregations get hit via regular relationships. Setting the agg table to Import lets related dimension tables become **Dual**.
- **GroupBy-column aggregations** (denormalized big-data models): GroupBy entries are **mandatory** or the aggregation won't be hit.
- Aggregation hits require **regular** relationships; many-to-many relationships are always limited.
- For **RLS**, the role's filter must filter **both** the aggregation table and the detail table, or the agg won't be used (and filtering only the agg table is disallowed).
- Aggregation tables are **hidden** from Report view; consumers query the detail table.
- Verify hits with SQL Server Profiler's `Query Processing\Aggregate Table Rewrite Query` event.

`COUNTROWS`, `AVERAGE` (folds to SUM/COUNT), DAX time-intelligence (e.g. `DATESYTD`), and even `DISTINCTCOUNT` (with a GroupBy entry, up to ~2–5M distinct values) can hit aggregations.

## Efficient DAX and visuals

- Keep measures **simple first** — `SUM`, `COUNT`, `MIN`, `MAX`, `AVERAGE`. Add complexity (CALCULATE filter manipulation) only after confirming it stays responsive; complex CALCULATE can generate expensive source queries under DirectQuery.
- Use **DIVIDE** for safe ratios and respect additivity — see `understanding-fact-additivity`.
- **Apply the most restrictive filters**: don't leave a table unfiltered over 100M+ rows; use a **Top N** filter to cap rows displayed (e.g. 10,000) — same UX, far less memory.
- **Limit visuals per report page**; use **drillthrough pages** and **report page tooltips** to add detail without more visuals on the page.
- **Evaluate custom visual performance** before shipping.
- **Dashboards** serve tiles from cache (except live-report and streaming tiles), so a dashboard makes a good fast landing page. Note dynamic **RLS** caches tiles per user, multiplying cache work.

Layer-by-layer checklist (model / DAX / visual / refresh): see [performance-tuning-checklist.md](performance-tuning-checklist.md).

## Query folding for faster refresh

Query folding pushes Power Query transformations into a single source query.

- **DirectQuery and Dual** tables **must** fold.
- For **Import**, folding gives best refresh performance when a single `SELECT` can be built; otherwise minimize the work the mashup engine does.
- **Delegate processing to the source**: find the step that breaks folding and move earlier steps before it; for relational sources use a native SQL query via `Value.NativeQuery` (note: a DirectQuery native query must be a `SELECT` with no CTEs/stored procedures, and incremental refresh can't use a native query), or prepare/materialize the data in a source view or warehouse.

This is the optimization face of `transforming-data-power-query`.

## DirectQuery and Composite tuning

When the model is DirectQuery, optimize all three sub-layers (full guidance in the DirectQuery doc):

- **Source**: ensure dimension keys are complete (add an "unknown" -1 member to repair integrity), add indexes / columnstore, materialize computed columns and a date table; avoid relationships on calculated columns and on GUID columns; set **Assume Referential Integrity** to get INNER joins; hide the one-side relationship column.
- **Model**: keep measures simple; avoid bidirectional filtering unless needed for a many-to-many bridge; tune **Maximum Connections per Data Source**.
- **Report**: enable **Query Reduction** options (Apply button on slicers/filters; disable cross-highlight/filter by default), apply filters before adding fields, limit visuals per page, switch off unnecessary visual interactions, switch off table/matrix totals when costly. Watch out for **measure filters**, **Top N filters**, **Median**, and multi-select slicers — each can fire extra source queries and risks the 1-million-row limit.
- **Convert to Composite** to combine Import speed with DirectQuery freshness and add aggregation tables.

## Separating report from model

Separate the report from the model (live connection to a published semantic model) when modelers and report authors differ, or when one model will feed many reports — this enables reuse and is the ALM-friendly pattern (feeds `managing-deployment-lifecycle`). When you separate, **preserve the model interface**: avoid renaming tables/columns/measures, changing data types, or moving measures to a different home table, since these break dependent visuals. Use **Data lineage** view to find affected reports before a breaking change.

## Common pitfalls

- **Optimizing before measuring.** Always triage with Performance Analyzer first; fix the layer that is actually slow.
- **Leaving Auto date/time on.** Hidden per-column date tables bloat the model — disable it and use one marked date table.
- **Loading all history and all columns "just in case."** This is the most common cause of bloated, slow models.
- **DAX calculated columns where a Power Query column would do.** Worse compression and slower refresh.
- **Bidirectional cross-filtering by default** (especially under DirectQuery) — generates poorly performing queries; reserve for many-to-many bridges.
- **Naive SUM of a semi-additive balance across time** — inflates totals; see `understanding-fact-additivity`.
- **Aggregation table with mismatched RLS** — if the role doesn't filter both agg and detail tables, the aggregation is never hit.
- **Treating a capacity problem as a report problem.** If Performance Analyzer is fine but reports are slow at peak, check the Fabric Capacity Metrics app and manage/scale capacity.

## Success criteria

- Performance Analyzer identifies the slow visual and whether DAX query, Direct query, or Visual display dominates.
- Model size is minimized: only needed columns/rows, low cardinality, Auto date/time off, no avoidable DAX calculated columns.
- Large DirectQuery facts are served by Import aggregation tables that verifiably get hit (Profiler) and respect RLS.
- Import refresh preserves query folding wherever a single source query is feasible.
- Re-measurement shows the bottleneck shrank or moved; capacity pressure (if any) is addressed at the capacity layer.

---
*Provenance: built per SKILL 19 "Optimizing Model Performance" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `guidance/power-bi-optimization.md`, `guidance/import-modeling-data-reduction.md`, `guidance/monitor-report-performance.md`, `guidance/report-performance-troubleshoot.md`, `create-reports/performance-analyzer.md`, `transform-model/aggregations-advanced.md`, `guidance/power-query-folding.md`, `guidance/directquery-model-guidance.md`, and `guidance/report-separate-from-model.md`; cross-references Kimball `physical-database-design-optimization`.*
