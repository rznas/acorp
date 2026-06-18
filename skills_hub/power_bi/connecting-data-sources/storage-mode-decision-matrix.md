# Storage-Mode Decision Matrix

All modes, behaviors, and limits below are confirmed in the Power BI docs `transform-model/desktop-storage-mode.md`, `connect-data/desktop-directquery-about.md`, `transform-model/desktop-composite-models.md`, and `guidance/directquery-model-guidance.md` / `composite-model-guidance.md`. Do not assert behavior beyond these.

## Contents
- Quick decision guide (what to consider first)
- Mode-by-mode tradeoff matrix
- DirectQuery in depth: limitations and limits
- Dual mode and propagation
- Composite models and source groups
- Conversion rules (what you can and can't switch)
- Decision checklist

---

## Quick decision guide

Start here; this mirrors the docs' quick table. **Import is the default** — move beyond it only on a stated constraint.

| If you need | Consider first | Why |
|-------------|----------------|-----|
| Maximum interactivity + full modeling features | **Import** | In-memory columnar engine, richest feature set |
| Near real-time on recent facts + historical context | **Hybrid table** (Import + DirectQuery partition) | Queries source for hot data, caches history |
| Lakehouse/warehouse scale, low-latency reads (Fabric) | **Direct Lake** | Bypasses scheduled refresh, keeps Import behaviors |
| Federate multiple external sources without full ingestion | **DirectQuery** (composite) | Leaves data in place, blends sources |
| Reuse a central governed enterprise model | **Live connection** to semantic model / Analysis Services | Reuses curated model, avoids duplication |
| Push user filters to source at runtime | **DirectQuery + dynamic M parameters** | Reduces scanned data |
| High concurrency / remote latency | **Import**, or **Aggregations over DirectQuery** | Aggregations accelerate common queries |

## Mode-by-mode tradeoff matrix

| Dimension | Import | DirectQuery | Dual | Direct Lake | Live connection | Composite |
|-----------|--------|-------------|------|-------------|-----------------|-----------|
| Data location | Cached in model (VertiPaq) | Stays in source, queried per visual | Cache **or** query | Scanned from OneLake delta tables | Remote model | Per table (mixed) |
| Data freshness | As of last refresh | Real-time (per query) | Either | Latest by default (reframing) | Per remote model | Per table |
| Performance | Fastest | Depends on source latency/concurrency | Fast when cache hit | Fast (in-memory-like) | Per remote model | Mixed; cross-source slower |
| Refresh required | **Yes** (Import data refresh) | No data refresh (tiles/visuals only) | Refresh the cached side | No (reframes; can disable auto-sync) | No | Only Import/Dual tables |
| Feature set | Full | Reduced (see limits) | Same constraints as DirectQuery | Import-like | Limited modeling (external model) | Per table; some model-wide limits |
| Gateway | If on-prem source | If on-prem source (or SSO) | If on-prem source | Fabric | Per source | Per source (mixed-mode needs gateway mapping) |
| Best for | Most models | Huge/volatile data, source security, sovereignty | Dimensions shared with DQ facts (same source) | Fabric lakehouse/warehouse | Reusing enterprise model | Blending sources, extending a model |

## DirectQuery in depth: limitations and limits

Use DirectQuery only when Import is genuinely infeasible. Confirmed limits:

- **1,000,000-row intermediate result limit** — any query/intermediate operation returning > 1M rows fails (Premium can raise it). Measure filters, TopN, Median, and DistinctCount totals can hit this.
- **4-minute per-query timeout** in the service; visuals exceeding it fail. Target visual refresh **< 5 seconds**; > 30 seconds is unusable.
- **Refresh to see latest data** — visual/tile/result caches can show prior results until refreshed; visuals aren't always time-consistent.
- **Schema changes require a Desktop refresh** — the service won't auto-detect dropped/renamed columns.
- **Case-insensitivity risk** — the Power BI engine is case-insensitive; a case-sensitive source can yield undefined results. Normalize casing in the source or Power Query.
- **Transformations must fold** — non-foldable steps, stored procedures, and CTEs aren't supported in a foldable way. OLAP sources (SAP BW) disallow in-query transforms.
- **Modeling limits:** no automatic date hierarchy (build an explicit Date table); time precision to seconds; calculated columns limited to foldable row-level expressions; no parent-child PATH functions; no clustering.
- **Reporting limits:** long text > 32,764 chars unsupported; measure/TopN filters, Median, multi-select slicers, and totals/subtotals (especially DistinctCount) add queries.

**DirectQuery modeling guidance (to make it perform):**
- Keep Power Query steps simple and foldable; preview **View Native Query** often.
- Materialize transformations, computed columns, and a date table in the source; add indexes; favor integer join keys.
- Hide the one-side (key) column of relationships.
- Set **Assume referential integrity** to get `INNER JOIN` (faster) where data integrity holds.
- Limit bidirectional filtering; limit visuals per page; use query-reduction options (Apply button, disable cross-highlighting).
- Adjust **Maximum connections per data source** (default 10; Pro/Report Server cap 10, Premium per SKU).

**Don't:** use Power Query relative-date filters on DirectQuery — they freeze into a fixed literal date. Put relative-date columns in the date table instead.

## Dual mode and propagation

- A **Dual** table can act as Import or DirectQuery depending on the query — it returns from cache when possible, else reverts to DirectQuery.
- Setting shared **dimension** tables to Dual reduces **limited relationships** with DirectQuery facts from the **same source** and improves performance.
- When you convert a DirectQuery table to Import, Power BI offers to set the related dimension tables to Dual. Propagation logic traverses only the **one** side of one-to-many relationships and computes the minimum set of dimensions to mark Dual.
- **Keep caches in sync:** Dual sometimes hits the cache, sometimes doesn't, so an out-of-date cache can disagree with the source. Dual is a performance optimization — your responsibility to manage data flows.

## Composite models and source groups

- A **composite model** = tables in more than one storage mode, **or** DirectQuery tables from different sources. Import-only (or Direct-Lake-only) is not composite.
- A **source group** = items from one DirectQuery source, or all import sources combined (all imported tables are one source group regardless of underlying origin; Direct Lake + Import count as the same group).
- **Intra-source-group** relationships: regular (unless many-to-many, then limited). **Cross-source-group** relationships: **always limited** (INNER JOIN — unmatched rows drop; can cause missing rows like a future-year target with no matching sales).
- Cross-source relationships default to **many-to-many** cardinality; you can't use DAX to pull one-side values from the many side, and there's a performance cost.
- **Cross-source join columns:** keep low cardinality (≈ < 50,000 unique values). For string keys, ensure `A * C < 250,000` (avg length × cardinality). Raise relationship granularity (e.g., join on MonthKey not DateKey) when possible.
- **Storage-mode recommendations in a composite model:** DirectQuery for large/volatile facts; Import for tables not used to filter/group DQ facts; Dual for dimensions queried with same-source DQ facts; Hybrid for latest-data-plus-history.
- **User-defined aggregations** can be added to DirectQuery tables (they cache like Import, making the model composite); aim for the aggregation to be ≥ 10× smaller than the base table. Hybrid tables don't support aggregations.
- **Security:** composite queries can move data values between sources (Desktop shows a warning). Composite-on-published-models needs **Build** permission on every model in the **chain (max length 3)**; RLS applies only on the source group where defined.
- **Unsupported in composite:** SAP HANA and SAP Business Warehouse (multidimensional/live tabular); SSAS Tabular before 2022; SSAS Multidimensional; usage metrics; streaming semantic models. Incremental refresh in composite is supported only for SQL, Oracle, Teradata.

## Conversion rules (what you can and can't switch)

- You **cannot** globally convert an Import-only model to DirectQuery.
- A **DirectQuery** table can convert to **Import** or **Dual** — but **not back to DirectQuery** afterward (exception: Power BI web modeling / live editing in Desktop have version control to reverse it).
- A table added via a DirectQuery-capable connection can move between DirectQuery / Import / Dual **only if all applied transformations still fold**.
- For most tables, you can set storage mode **only when the table is added** (you can later change a table created as DirectQuery or Direct Lake on OneLake).
- To migrate toward DirectQuery: add a new DirectQuery connection and remap visuals, build a composite model (Import dimensions + DirectQuery facts, optionally Dual), or use Hybrid tables.

## Decision checklist

```
Storage-mode decision:
- [ ] Is Import feasible (size fits memory + refresh window, freshness acceptable)?  YES -> Import.
- [ ] Need real-time on recent data only?  -> Hybrid (incremental refresh + real-time DirectQuery, Premium/PPU/Embedded).
- [ ] Fabric lakehouse/warehouse at scale?  -> Direct Lake.
- [ ] Source-enforced (per-user) security must stay authoritative?  -> DirectQuery with SSO.
- [ ] Data sovereignty forbids persisted copies?  -> DirectQuery.
- [ ] Reuse an existing governed model unchanged?  -> Live connection.
- [ ] Blend multiple sources or extend a published model?  -> Composite (Dual dims, DirectQuery facts).
- [ ] Chose DirectQuery? confirm folding (View Native Query), watch 1M-row + 4-min limits.
```
