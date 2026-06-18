# Performance tuning checklist (by layer)

A layer-by-layer checklist for tuning a Power BI solution. Work the layer that Performance Analyzer points to (see performance-analyzer-triage.md). Every item below is grounded in the Microsoft optimization, data-reduction, DirectQuery, query-folding, and Performance Analyzer docs.

## Contents
- Model size layer
- DAX layer
- Visual / report layer
- Refresh / Power Query layer
- DirectQuery & Composite layer (when applicable)
- Environment layer (capacity / gateway / network)

---

## Model size layer

Smaller Import models fit capacity limits, reduce memory eviction, refresh faster, and evaluate calculations faster. (VertiPaq compresses ~10x in memory, ~+20% on disk — but minimize anyway.)

```
- [ ] Remove unnecessary columns (vertical filtering): keep only columns used for reporting or model structure (relationships, calcs, RLS, color formatting). Drop high-cardinality unused columns first.
- [ ] Remove unnecessary rows (horizontal filtering): filter by time (don't load all history; use a Power Query date parameter, optionally relative to refresh date) and/or by entity (e.g., one region).
- [ ] Group by and summarize: raise fact grain (e.g., day -> month) where detail isn't required. Tradeoff: detail-level reporting is lost.
- [ ] Optimize column data types: numeric uses value encoding (best). Convert text keys like "SO123456" to whole numbers where possible; set such a column to "Do Not Summarize".
- [ ] Prefer Power Query (M) custom columns over DAX calculated columns: DAX calculated columns compress less efficiently and extend refresh.
- [ ] Disable query load for staging/integration queries not needed in the model.
- [ ] Disable Auto date/time (Options/Settings): it creates a hidden calculated date table per date column. Use one explicit marked date table instead.
- [ ] Hide columns/measures not meant for reporting (focus + discourages misuse).
- [ ] Shape a star schema (fact + denormalized dimensions) for VertiPaq efficiency and relationship-based aggregation hits.
- [ ] For very large fact tables, consider DirectQuery storage mode + summarized Import tables (Composite model).
```

Detailed technique-by-technique guidance with tradeoffs: see data-reduction-techniques.md.

---

## DAX layer

Slow DAX query in Performance Analyzer means the model/measure is doing too much work.

```
- [ ] Keep measures simple first: SUM, COUNT, MIN, MAX, AVERAGE. Add CALCULATE filter manipulation only after confirming responsiveness (complex CALCULATE can generate expensive native queries under DirectQuery).
- [ ] Use DIVIDE for safe ratios (avoids divide-by-zero) and compute ratio-of-sums, not sum/average-of-ratios.
- [ ] Respect additivity: never naive-SUM a semi-additive balance across time (use AVERAGEX over dates, LASTNONBLANK, or CLOSINGBALANCE patterns). See sibling skill understanding-fact-additivity.
- [ ] Reduce model size first (above): smaller row counts make calculations evaluate faster.
- [ ] Consider aggregation tables for large DirectQuery facts so high-level queries hit a small in-memory cache.
- [ ] Iterate: in Performance Analyzer use Copy query, then run it in DAX query view to test rewrites.
```

---

## Visual / report layer

Slow Visual display in Performance Analyzer means rendering or data volume, not the model.

```
- [ ] Apply the most restrictive filters (report/page/visual). Don't leave a table showing all 100M+ rows.
- [ ] Use a Top N filter to cap rows in expensive visuals (e.g., 10,000) — same UX, far less memory. Tables are especially expensive.
- [ ] Limit the number of visuals on a report page; prefer more, simpler pages.
- [ ] Move detail into drillthrough pages and report page tooltips instead of adding visuals to the page.
- [ ] Replace several card visuals with a single multi-row card to cut query count.
- [ ] Evaluate custom visual performance — a poorly optimized custom visual can slow the whole report.
- [ ] Use dashboards as a fast first landing page: tiles serve from cache (except live-report and streaming tiles). Note: dynamic RLS caches tiles per user, multiplying cache work.
```

---

## Refresh / Power Query layer

Slow Import refresh is usually too much data plus broken query folding.

```
- [ ] Reduce model size first (fewer columns/rows lowers refresh cost).
- [ ] Preserve query folding: for relational sources aim for a single foldable SELECT; minimize work left to the mashup engine.
- [ ] Find the step that breaks folding (View Native Query shows the generated SQL when folding); move foldable steps before it.
- [ ] Delegate to the source: use a native SQL query via Value.NativeQuery, a database view, or pre-materialized/warehoused data.
- [ ] DirectQuery/Dual tables MUST fold — verify with View Native Query.
- [ ] Native-query caveats: a DirectQuery native query must be a SELECT (no CTEs / stored procedures); incremental refresh can't use a native query.
- [ ] Use Query Diagnostics (Diagnose Step) in Power Query Editor to see what each step does during preview/apply.
```

---

## DirectQuery & Composite layer (when applicable)

Apply only when tables use DirectQuery. Optimize source + model + report.

```
Source:
- [ ] Complete data integrity: dimension keys unique; add an "unknown" member with an out-of-range key (e.g., -1) and substitute missing fact keys.
- [ ] Add indexes (and columnstore for volatile/analytic sources).
- [ ] Materialize computed columns and indexed views (pre-aggregated grain) in the source.
- [ ] Materialize a date table with useful columns (year/quarter/month/week) and relative-time offset columns.

Model:
- [ ] Avoid complex Power Query queries (one query per source table/view); check View Native Query.
- [ ] Avoid relationships on calculated columns and on GUID/unique-identifier columns; use COMBINEVALUES for multi-column relationships.
- [ ] Set "Assume Referential Integrity" to generate INNER (not OUTER) joins where data supports it.
- [ ] Hide the one-side (key) column of relationships to avoid unnecessary joins.
- [ ] Avoid bidirectional cross-filtering unless implementing a many-to-many bridge.
- [ ] Tune "Maximum Connections per Data Source" (more parallelism vs. more source load).

Report:
- [ ] Enable Query Reduction (Options): Apply button on slicers/filters; disable cross-highlight/cross-filter by default.
- [ ] Apply filters before mapping fields to a visual.
- [ ] Limit visuals per page; switch off unnecessary visual interactions.
- [ ] Switch off table/matrix totals when they fire extra queries (Count Distinct, Median, SAP HANA/BW always).
- [ ] Watch measure filters, Top N filters, Median, and multi-select slicers — each can trigger extra source queries and risk the 1M-row limit.
- [ ] Consider converting to a Composite model and adding aggregation tables.
```

---

## Environment layer (capacity / gateway / network)

When report-side fixes don't help, the bottleneck is the environment.

```
- [ ] Premium/Fabric capacity: use the Fabric Capacity Metrics app to check for resource pressure; manage or scale capacity (or investigate activity during typical usage).
- [ ] Shared capacity: you can't monitor capacity health — migrate the model to Azure Analysis Services or a Premium/Fabric capacity if peak-time slowness affects many users.
- [ ] Gateway: size the on-premises data gateway for its workload.
- [ ] Network latency: keep data sources, gateways, and the Power BI capacity close (ideally same region); host gateways/sources in cloud VMs near the capacity if needed.
```
