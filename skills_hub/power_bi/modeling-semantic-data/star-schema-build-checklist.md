# Star-Schema Build Checklist

Step-by-step procedure for building a star-schema semantic model in Power BI Desktop, grounded in `guidance/star-schema.md` and the relationship docs. For the dimensional theory behind grain, additivity, and conformed dimensions, cross-reference Kimball `four-step-dimensional-modeling` and `understanding-fact-additivity`.

## Contents
- Why star schema in Power BI
- Build checklist
- Classifying tables (dimension vs fact)
- Surrogate keys for dimensions
- Snowflake vs flatten decision
- Degenerate dimensions
- Junk dimensions
- Factless / bridging tables
- Slowly changing dimensions (what Power BI can and can't do)
- Hierarchies and Data-pane hygiene
- Verification

## Why star schema in Power BI

Each report visual sends a query that filters, groups, and summarizes. A star schema serves this directly: **dimension tables enable filtering and grouping; fact tables enable summarization.** Table roles are determined by relationships, not a property — in a 1:* relationship the "one" side is the dimension and the "many" side is the fact. Strive for the right number of tables with the right relationships, and load each fact table at a single consistent grain.

## Build checklist

```
Star-schema build:
- [ ] 1. List business processes; one fact table per process at atomic grain (Kimball four-step)
- [ ] 2. Classify every table: dimension (describes) or fact (measures). Don't mix.
- [ ] 3. Give each dimension a single unique key column (add a surrogate index key if missing)
- [ ] 4. Create 1:* relationships: dimension key (one) → fact key (many)
- [ ] 5. Default cross-filter direction = Single
- [ ] 6. Decide snowflake vs flatten for each multi-table dimension (prefer flatten)
- [ ] 7. Add degenerate dimensions (IDs kept in the fact) where filtering/grouping is needed
- [ ] 8. Consolidate small low-cardinality attribute tables into a junk dimension if useful
- [ ] 9. Use a bridging (factless fact) table for many-to-many dimensions
- [ ] 10. Confirm fact-table grain is consistent; facts are true to the grain
- [ ] 11. Build hierarchies within single tables; set sort-by and formats
- [ ] 12. Hide surrogate keys, bridge tables, and non-reporting columns
- [ ] 13. Verify totals, counts, and slicers behave correctly
```

## Classifying tables (dimension vs fact)

- **Dimension table** — describes business entities (products, people, places, time). Has a unique key column plus descriptive columns used to filter and group. Usually few rows.
- **Fact table** — stores observations/events (orders, balances, exchange rates). Holds dimension key columns (defining its dimensionality) and numeric measure columns. Grows large over time; the key *values* define its granularity.

Avoid mixing the two in one table. The documented exception is a **degenerate dimension** (an attribute kept in the fact for filtering, e.g., order number).

## Surrogate keys for dimensions

A relationship needs a single unique column on the "one" side. If a dimension has no single unique column, add a **Power Query index column** as a surrogate key, then **merge** that query into the "many"-side (fact) query so the key exists on both sides. Load both, then create the 1:* relationship. Source data often arrives **denormalized** (e.g., a flat export) — use Power Query to split it into normalized fact and dimension tables before modeling.

## Snowflake vs flatten decision

A snowflake dimension is a normalized set of tables for one entity (e.g., Category → Subcategory → Product). In Power BI you can mimic the snowflake or flatten into one denormalized dimension table.

**Prefer flattening into a single dimension table.** Reasons the docs give:
- Snowflaking loads more tables → larger model, slower (extra relationship columns).
- Longer filter-propagation chains are less efficient.
- More tables clutter the Data pane (especially tiny 1–2 column tables).
- You **cannot build a hierarchy across multiple tables** — a single flattened table lets you define a Category→Subcategory→Product hierarchy.

Flattening can increase storage (redundant denormalized values), especially for very large dimensions — weigh data volume vs usability. This is the one case where denormalizing a snowflake into a single table is acceptable.

## Degenerate dimensions

An attribute of the fact required for filtering (e.g., sales order number). Don't create a one-column table for it — keep the column in the fact table to filter/group by it. If you have several related fact attributes (order number + order line number) both needed for filtering, splitting them into a separate degenerate-dimension table via a one-to-one relationship is good design (see relationship-design-matrix one-to-one section).

## Junk dimensions

Consolidate many small, low-value attributes (order status, flags, demographic buckets like gender/age group) into one **junk dimension** to cut model size and Data-pane clutter. Typically the Cartesian product of attribute members plus a surrogate (index) key; build it in the warehouse or with Power Query full outer joins. Merge its key into the fact query to create a 1:* relationship.

## Factless / bridging tables

A **factless fact table** has only dimension keys, no measures. Two uses:
- Record events defined purely by dimension keys (e.g., a customer signed in at a date/time) — count rows with a measure.
- **Bridging table** for many-to-many dimension relationships (the recommended approach): store associations (e.g., salesperson key + region key) and relate the two dimensions through it (two 1:* relationships + one bidirectional — see relationship-design-matrix).

## Slowly changing dimensions (what Power BI can and can't do)

Power BI uses Power Query, which **cannot detect and version changes itself**:
- **Type 1 (overwrite latest):** achieved by a normal (non-incremental) refresh that reloads the dimension with current values.
- **Type 2 (versioned history):** Power BI **cannot produce** versions; it can only **load** a pre-built Type 2 dimension (from a data warehouse) that already has a surrogate key, validity dates (`StartDate`/`EndDate`), and optionally an `IsCurrent` flag. Provide two columns: one to filter the member across all versions, one with a non-ambiguous version label (e.g., `David Campbell (06/27/2019-Current)`), and a hierarchy to drill to the version. If you need SCD/large volumes, build a warehouse + ETL first and connect the model to it.

## Hierarchies and Data-pane hygiene

- Build hierarchies only within a single table (another reason to flatten snowflakes).
- Hide surrogate keys, bridge tables, and any column not meant for reporting.
- Use display folders to organize fields without splitting into extra tables.
- Add table descriptions so report authors see filter behavior in Data-pane tooltips (important for generic names like `Date`).

## Verification

- Put a dimension attribute and a fact measure in a table visual — totals should differ per row, not repeat one value (a repeated total means a missing or wrong relationship).
- Cross-check a count that requires reverse propagation (e.g., employees per project) — if it needs Both, confirm the matrix justifies it.
- Check slicers don't show stray BLANK members (referential-integrity violations or inactive-relationship expansion).
- Confirm each fact table loads a single consistent grain.
