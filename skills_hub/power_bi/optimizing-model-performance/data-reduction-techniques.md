# Data reduction techniques for Import models

Techniques to reduce the data loaded into an Import semantic model, ranked roughly by leverage. Grounded in `guidance/import-modeling-data-reduction.md`. Smaller models: fit capacity size limits, reduce memory contention/eviction, refresh faster, and evaluate calculations faster.

> VertiPaq context: Import data is compressed (often ~10x in memory; ~+20% reduction persisted to disk). Numeric columns use **value encoding** (most efficient); text/non-numeric use **hash encoding** (a numeric id per distinct value). High cardinality is the enemy of compression.

## Contents
- Techniques table
- Detail notes per technique
- Cross-references

---

## Techniques table

| # | Technique | What it does | How (Power BI) | Tradeoff / caution |
|---|-----------|--------------|----------------|--------------------|
| 1 | **Remove unnecessary columns** (vertical filtering) | Drops columns not used for reporting or model structure | Remove in Power Query / model; keep only columns used by visuals, relationships, calcs, RLS, or color formatting | Easier to add columns later than remove; removing can break reports/model structure |
| 2 | **Remove unnecessary rows** (horizontal filtering) | Loads fewer rows: by time or by entity | Power Query filter; use a date **parameter** (can be relative to refresh date, e.g. last 5 years); or load one entity (region) per model | Time filter is safe to revisit (just changes history available). Entity filtering means many models, specific permissions, and duplicate reports — but can remove the need for RLS |
| 3 | **Group by and summarize** | Raises fact grain to cut row count dramatically | Group/aggregate in the source, in Power Query, or via a pre-aggregated view | Loss of detail (e.g., month grain loses day/order-line reporting). Can mitigate with a DirectQuery detail table for drillthrough |
| 4 | **Optimize column data types** | Pushes columns toward efficient value encoding | Convert text keys (e.g. `SO123456` -> `123456` whole number); set such columns to **Do Not Summarize** | Only where a clean numeric conversion exists; biggest win on high-cardinality columns |
| 5 | **Prefer Power Query (M) custom columns over DAX calculated columns** | Better compression and faster refresh | Create computed columns in Power Query (or in source SQL / materialized in source) instead of DAX | Use a DAX calculated column only when the formula needs measures or DAX-only functionality (e.g. parent-child hierarchy functions) |
| 6 | **Disable query load** | Keeps staging/integration queries out of the model | In Power Query, disable **Enable load** for helper queries | None — these queries still support other queries; they just aren't materialized |
| 7 | **Disable Auto date/time** | Removes hidden per-column calculated date tables | Options/Settings: turn off Auto date/time; use one explicit, marked date table | Without a proper date table, time intelligence relies on the removed hidden tables — set up a marked date table (see applying-time-intelligence) |
| 8 | **Use DirectQuery storage mode for large fact tables** | Stores only metadata for the biggest table; data stays at source | Set the fact table storage mode to DirectQuery (forms a Composite model with Import tables) | DirectQuery has security/performance implications; pairs well with #3 (summarized Import + DirectQuery drillthrough) |

---

## Detail notes per technique

**1. Remove unnecessary columns.** A model column earns its place only if it serves *reporting* (filter/group/summarize) or *model structure* (relationships, calculations, security roles, data color formatting). Design for the right number of columns from known requirements; adding later is cheaper than removing.

**2. Remove unnecessary rows.**
- *Filter by time:* don't default to all history. Implement time-based Power Query filters with parameters; relative periods (relative to refresh date) keep the window rolling. Changing the window later doesn't break reports — it just changes available history.
- *Filter by entity:* load a subset (e.g., one sales region) to produce many smaller models. This can eliminate RLS, but requires per-model permissions and duplicate reports; Power Query parameters + Power BI Template (.pbit) files simplify management/publication.

**3. Group by and summarize.** The single most effective size reducer. Grouping order-line sales to date/customer/product is large; grouping date to *month level* can approach ~99% reduction. The cost is permanent loss of finer detail unless a DirectQuery detail table backs a drillthrough page.

**4. Optimize column data types.** VertiPaq is most efficient on numeric (value-encoded) data. Stripping a constant text prefix and storing the numeric portion as a whole number can yield significant reduction on large, high-cardinality columns. Set the column's default summarization to **Do Not Summarize** so an id isn't accidentally summed.

**5. Power Query columns over DAX calculated columns.** VertiPaq stores DAX calculated columns with typically *less efficient* compression, and builds them after all Power Query tables load, extending refresh. Prefer M computed columns; or compute in the source SQL / materialize in the source. Reserve DAX calculated columns for cases that require measures or DAX-only functions.

**6. Disable query load.** Queries that only support integration with other queries shouldn't be loaded to the model — uncheck Enable load.

**7. Disable Auto date/time.** When on, Power BI creates a hidden calculated date table for *each* date column, inflating model size. Replace with a single explicit date table (mark it as a date table) — see sibling skill `applying-time-intelligence`.

**8. DirectQuery for large fact tables.** DirectQuery tables hold only metadata; native queries fetch data on demand. Setting a large fact table to DirectQuery (with Import dimensions/aggregates = Composite model) trades query latency for a far smaller model. Combine with summarized Import data for fast summary reporting and a DirectQuery drillthrough for in-context detail.

---

## Cross-references

- **Aggregation tables** (Import cache over a DirectQuery detail table) — see SKILL.md "Aggregation tables" and `transform-model/aggregations-advanced.md`. This is the performance complement to technique #3/#8.
- **Source-side reduction** (pre-aggregated/indexed views, partitioning, columnstore in a dimensional warehouse) — cross-reference Kimball `physical-database-design-optimization`.
- **Shaping foldable, star-schema tables** in Power Query — see sibling skill `transforming-data-power-query` and the query-folding section in SKILL.md.
