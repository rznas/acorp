# Performance Analyzer triage steps

How to locate the bottleneck in a slow Power BI report before changing anything. Grounded in `create-reports/performance-analyzer.md`, `guidance/monitor-report-performance.md`, and `guidance/report-performance-troubleshoot.md`.

> Performance Analyzer runs in **Report view** of Power BI Desktop, or when editing a report in the Power BI service. It **cannot** monitor Premium Per User (PPU) activities or capacity — use the Fabric Capacity Metrics app for capacity.

## Contents
- Open Performance Analyzer
- Record a session
- Read the duration breakdown
- Decide what to fix (decision table)
- Inspect and rewrite the DAX query
- When the report isn't the problem (capacity/network)
- Other monitoring tools

---

## Open Performance Analyzer

- **Power BI Desktop:** select the **Optimize** ribbon, then **Performance Analyzer**. The pane opens to the right of the canvas.
- **Power BI service:** open the report, select **Edit**, then the **View** menu, then **Performance Analyzer**.

## Record a session

1. Select **Start recording**. (Start recording grays out; **Stop** becomes active.)
2. Reproduce the slow experience: open the page, move the slow slicer/filter, or interact as a user would. Each action gets a labeled section in the pane (e.g., "Changed a slicer").
3. To force timings for the whole page, select **Refresh visuals**. To time one visual, with recording on select **analyze this visual** in that visual's top-right corner.
4. Select **Stop**. Results stay in the pane for analysis. **Clear** erases them (not saved). **Export** writes a `.json` log of the session.

## Read the duration breakdown

Each visual logs **Duration (ms)** split into categories:

| Category | Meaning |
|----------|---------|
| **DAX query** | Time from the visual sending its query to the model returning results. High = model/measure work. |
| **Direct query** | (DirectQuery tables only) time for the external source query to return. High = source/network. |
| **Visual display** | Time to draw the visual, including fetching web images / geocoding. High = rendering or data volume. |
| **Other** | Preparing queries, waiting for other visuals, background processing. |
| **Evaluated parameters** (preview) | Time spent evaluating field parameters in the visual. |

Durations are the difference between start/end timestamps; most canvas/visual operations run sequentially on one UI thread, so reported time includes queue wait. DAX query time grows with calculation complexity, number of tables, and the relationships between them.

## Decide what to fix (decision table)

| Dominant duration | Likely cause | Fix layer (see performance-tuning-checklist.md) |
|-------------------|-------------|-------------------------------------------------|
| **DAX query** | Model too large / star schema not clean / expensive measure | Model size layer + DAX layer; consider aggregation tables for large DirectQuery facts |
| **Direct query** | Slow source query / network / source not optimized | DirectQuery & Composite layer (source + model + report) |
| **Visual display** | Too much data shown / too many visuals / slow custom visual | Visual / report layer (restrictive filters, Top N, fewer visuals, vet custom visuals) |
| **Other** (waiting) | Too many visuals contending on one page | Visual / report layer (limit visuals per page) |
| All low but report still slow | Not a report problem | Environment layer (capacity / network) |

## Inspect and rewrite the DAX query

When **DAX query** dominates:

1. In Performance Analyzer select **Copy query** for the slow visual. For DirectQuery tables this includes the DAX plus the translated SQL/KQL.
2. Select **Run in DAX query view** (Desktop) to run it in **DAX query view** and inspect the result and logic. In the service, open the data model (web modeling) > DAX query view, then paste/run the copied query.
3. Iterate on a simpler/cheaper measure; re-record to confirm the DAX query duration dropped. Visual-generated DAX is intentionally verbose (VARs, TOPN) to support visual-type switching, so the goal is a correct, faster equivalent — not byte-for-byte identity.

## When the report isn't the problem (capacity/network)

Follow the troubleshooting flowchart logic:

- **On Premium/Fabric capacity:** use the **Fabric Capacity Metrics app**. If the capacity frequently exceeds its resources, **manage or scale** it; if resources are adequate, investigate activity during typical usage.
- **On shared capacity:** you can't monitor capacity health. Then:
  - If slowness is at specific **times** (many users at once): migrate the model to **Azure Analysis Services** or a **Premium/Fabric capacity**, or use Performance Analyzer to optimize the report/model.
  - If slowness is isolated to a **geography/region**: the source is likely remote — change architecture (Azure Analysis Services) or optimize the **on-premises data gateway**.
  - If **no time pattern and all regions** are slow: use Performance Analyzer to optimize the report/model. If it's isolated to specific **devices/clients/browsers**, raise a support ticket.

## Other monitoring tools

- **Query Diagnostics** (Power Query Editor, including the **Diagnose Step** function) — see what Power Query does while previewing/applying, to diagnose slow refresh / non-folding steps.
- **SQL Server Profiler** (in SQL Server Management Studio) — for SQL Server / SSAS / Azure AS sources. Connect Analysis Services to `localhost:[port]` (find the **msmdsrv.exe** port via `netstat -b -n`). Also detects whether queries hit the cache or push to DirectQuery, and (via the `Query Processing\Aggregate Table Rewrite Query` event) whether aggregations are hit.
  - Caution: connecting tools to the diagnostics port is supported, but making unsupported changes to the Desktop model through it can cause corruption/data loss.
- **Fabric Capacity Metrics app** — monitor production performance on Premium/Fabric capacity.
