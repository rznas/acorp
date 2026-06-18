---
name: transforming-data-power-query
description: Cleans and shapes data in the Power BI Desktop Power Query Editor (M) before modeling, covering common transforms (filter, remove/rename columns, change type, split, group by, pivot/unpivot, merge, append, custom columns, combine files), query folding to preserve refresh and DirectQuery performance, referenced vs duplicated queries and dataflows, and shaping source tables into star-schema fact and dimension tables. Use when preparing, cleaning, combining, or transforming data in Power Query Editor before building the model — when someone mentions Transform data, Power Query, M, applied steps, query folding, merge/append queries, unpivot, Group By, combine files from a folder, or a slow refresh caused by transformations.
---

# Transforming Data with Power Query

## Overview

Power Query Editor is where you clean and shape source data **before** it loads into the Power BI model. Every action you take is recorded as an ordered **applied step** (M code), and the underlying source data is never modified — Power Query only adjusts its view of the data and replays the steps on each refresh. Get the shape right here and modeling, DAX, and performance all become easier.

This skill covers: the Power Query Editor workflow, the common transforms cookbook, how to keep **query folding** alive (critical for refresh and DirectQuery performance), how referenced queries actually execute, and how to shape raw source tables into clean **fact and dimension tables** for a star schema.

**Prerequisites (from the full-lifecycle plan):** depends on `connecting-data-sources` (you have a connection and chose a storage mode — Import / DirectQuery / Dual). Feeds `modeling-semantic-data`. Cross-reference the Kimball DW skill `four-step-dimensional-modeling` whenever you decide what a fact vs. dimension table should contain.

## When to Use

Use this skill when the task involves any of:
- Cleaning source data: removing/renaming columns, changing types, filtering rows, replacing values.
- Reshaping: pivot, unpivot, split column, group by, custom columns.
- Combining: **Merge** (join) or **Append** (stack) queries, or **Combine Files** from a folder.
- Preserving **query folding** so refresh is fast and DirectQuery works.
- Deciding between referenced queries, duplicated queries, or a dataflow.
- Splitting one wide source table into star-schema-shaped fact and dimension tables.

If the task is choosing a connector or Import-vs-DirectQuery storage mode, that belongs to `connecting-data-sources`. If it is relationships, cardinality, or filter direction, that belongs to `modeling-semantic-data`.

## The Power Query Editor workflow

Open it with **Home > Transform data** in Power BI Desktop. The window has four areas:

| Area | Purpose |
|---|---|
| **Ribbon** (Home, Transform, Add Column, View, Tools, Help) | Transform commands. Many are also on the right-click column menu. |
| **Queries pane** (left) | All queries; select one to shape it. |
| **Data pane** (center) | Preview of the selected query's data. |
| **Query Settings pane** (right) | Query **Properties** (name) and the ordered **Applied Steps** list. |

Core loop:
1. Connect (**New Source** / **Get data**) and select tables in the **Navigator**.
2. Apply transforms via the ribbon or right-click menu. Each becomes an **applied step**.
3. Inspect / reorder / rename / delete steps in **Applied Steps** (right-click a step). Steps run top-to-bottom in order.
4. View or edit the generated M in the **formula bar** (turn on via **View > Formula Bar**) or the full **Advanced Editor** (**View > Advanced Editor**).
5. **Home > Close & Apply** (also **File > Close & Apply**) to load the shaped data into the model.

To undo, select the **X** next to a step in **Applied Steps** — the source data is untouched, so experiment freely.

## Common transforms (quick reference)

These are the transforms you will use most. Full cookbook with M snippets, dialog options, and gotchas: see [common-transforms-cookbook.md](common-transforms-cookbook.md).

| Task | How (UI) | Use it to |
|---|---|---|
| Remove columns | Select column(s) > **Home > Remove Columns**, or right-click > Remove | Drop unneeded columns early (smaller model, faster refresh). |
| Rename column | Double-click header, or right-click > **Rename** | Give business-friendly names. |
| Change type | Select column > **Transform > Data Type**, or the type icon in the header | Set correct types (date, decimal, whole number, text). |
| Filter rows | Column dropdown arrow > clear/keep values | Exclude rows you don't need. |
| Split column | **Transform > Split Column** | Break "City, State" or delimited keys into parts. |
| Group By | **Transform > Group By** (or Home) | Summarize rows (Sum, Count Rows, Count Distinct Rows, Median, etc.); supports multiple groupings/aggregations under **Advanced**. |
| Pivot Column | **Transform > Pivot Column** | Turn unique values of a column into columns of aggregated values. |
| Unpivot Columns | **Transform > Unpivot Columns** | Turn wide columns into attribute/value rows (essential for tidy fact tables). |
| Custom Column | **Add Column > Custom Column** | Build a new column from an M expression over other columns. |
| Merge queries | **Home > Merge queries** (or **Merge queries as new**) | Join two queries on key column(s); pick a join kind, then expand. |
| Append queries | **Home > Append queries** | Stack queries with the same schema into one table. |
| Combine Files | Connect to **File > Folder**, then **Combine Files** | Combine many same-schema files (CSV/Excel/JSON) into one table. |

Tip: the type-change is recorded as a step (**Changed Type**), so a single wrong auto-detected type can be fixed by editing that one step.

## Query folding (preserve refresh performance)

**Query folding** is Power Query's ability to translate your steps into a single source query (for example, one SQL `SELECT`) executed by the data source instead of locally. Folding matters because it pushes work to the source, which is faster and lighter.

Hard rules:
- **DirectQuery and Dual storage-mode tables MUST fold** — every step has to translate to a source query, or the table won't work in DirectQuery.
- **Import tables should fold for best refresh performance**, especially large ones. When the source is relational and a single `SELECT` can be built, ensure folding occurs; otherwise the local mashup engine does the work.

Keep folding alive:
- **Delegate as much as possible to the source.** When a step breaks folding, find that step and, where correct, move foldable steps earlier so they fold. (The engine may also reorder steps itself.)
- **Use a native SQL query** (`Value.NativeQuery`, which safely passes parameters) when a relational transform can't be expressed as foldable steps — but then no later step can fold, so put all logic in the native query. For DirectQuery the native query must be a `SELECT` (no CTEs, no stored procedures), and **incremental refresh can't use a native SQL query**.
- **Prepare/transform in the source** (a database view or a materialized data-warehouse table) when steps simply won't fold.

Full do/don't checklist (which transforms typically break folding, how to detect it, ordering rules): see [query-folding-checklist.md](query-folding-checklist.md).

## Referenced queries, duplicated queries, and dataflows

A referenced query is **not** computed once and reused. If **Query1** (load disabled) is referenced by **Query2**, **Query3**, and **Query4**, Power Query executes Query1's steps **inside each** referencing query — Query1 effectively runs three times. This can slow refresh and overload the source. `Table.Buffer` does not fix this: a buffered table only lives within a single query execution.

Guidance:
- Referencing queries is still recommended to **avoid duplicating transformation logic** — just know the source is re-read per referencing query.
- When the shared, expensive source/transform must be computed **once**, create a **dataflow** (in the Power BI service) and point the downstream queries at it. A dataflow is a persisted store, so its data retrieval is fast and the source logic is evaluated once. Connect from Desktop via **Get data > Power Platform > Dataflows**.
- Disable load (right-click query > uncheck **Enable load**) on staging queries you only use as inputs, so they don't bloat the model.

For faster refresh in files with many queries, enable **File > Options and settings > Options > Data Load > Allow data preview to download in the background** (per-file). Refresh stale previews with **Home > Refresh Preview** in the editor.

## Shaping data for a star schema

Transform raw, often wide/operational tables into clean **fact** and **dimension** tables before modeling. Apply the Kimball `four-step-dimensional-modeling` thinking inside Power Query: pick the business process, declare the grain (one row =), identify dimensions, identify facts — then split and shape accordingly.

Core moves:
- **Separate facts from dimensions.** Keep numeric, additive measurements + foreign keys in fact queries; keep descriptive attributes in dimension queries.
- **Build dimension tables** by selecting descriptive columns, **Remove Duplicates** on the key, and giving business-friendly names/types.
- **Build the fact table** by removing descriptive attributes (they live in dimensions), keeping keys + measures, and setting correct numeric types.
- **Merge** to bring a foreign key onto a fact (join on the natural key, expand only the key/needed columns).
- **Unpivot** wide "one column per period/metric" layouts into a tidy long fact.
- **Append / Combine Files** to union same-grain sources (for example, monthly files) into one fact.

Full recipe with worked steps and the four-step cross-reference: see [star-schema-shaping-recipe.md](star-schema-shaping-recipe.md).

## Recommended order of steps

Step order affects both correctness and folding. A sensible default:
1. Source / navigation.
2. Remove unneeded columns and filter unneeded rows **early** (smaller data, more likely to fold).
3. Change data types.
4. Split / merge / group / pivot / unpivot reshaping.
5. Add custom columns last (custom logic is a common fold-breaker — keep it after foldable steps).
6. Rename for business friendliness.

## Common pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Folding broken early in the query | Slow refresh; DirectQuery table errors | Find the fold-breaking step; move it later or push it to the source. See [query-folding-checklist.md](query-folding-checklist.md). |
| Assuming a referenced query runs once | Refresh slow; source hit multiple times | Use a dataflow for the shared logic; don't rely on `Table.Buffer`. |
| Loading staging queries into the model | Bloated model, clutter | Disable load on intermediate queries. |
| Keeping descriptive text columns on the fact table | Wide fact, larger model, weak star schema | Move attributes to dimension tables; keep keys + measures on the fact. |
| Native SQL query for DirectQuery using CTE/stored proc | DirectQuery fails | Use a single `SELECT` only; no CTEs/stored procedures for DirectQuery. |
| Native SQL query with incremental refresh | Full reload every refresh | Don't combine native SQL with incremental refresh. |
| Wrong auto-detected type | Wrong sorts/aggregations downstream | Fix the **Changed Type** step (or set type explicitly). |
| Combine Files on mixed schemas | Errors / missing columns | Combine only files with the same file type and structure. |

## Success criteria

- The query loads clean, correctly typed data with business-friendly names.
- Unneeded columns/rows are removed early; the model is no larger than needed.
- For DirectQuery/Dual, **every** step folds; for large Import tables, folding is preserved as far as possible.
- Shared/expensive logic that multiple queries need is in a dataflow, not a re-read referenced query.
- Output queries are shaped as clean fact and dimension tables ready for `modeling-semantic-data`.
- Staging/input-only queries have load disabled.

---
*Provenance: built per SKILL 12 "Transforming Data with Power Query" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `transform-model/desktop-query-overview.md`, `desktop-common-query-tasks.md`, `desktop-combine-binaries.md`, `desktop-connect-dataflows.md`, `connect-data/desktop-shape-and-combine-data.md`, and `guidance/power-query-folding.md`, `power-query-referenced-queries.md`, `power-query-background-refresh.md`; cross-references the Kimball DW skill `four-step-dimensional-modeling`.*
