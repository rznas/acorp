---
name: modeling-semantic-data
description: Builds the Power BI Desktop semantic model as a star schema — creating relationships, choosing cardinality (1:*, *:1, 1:1, *:*) and cross-filter direction (single vs both), handling role-playing dimensions with USERELATIONSHIP, marking a date table, and setting data categories. Use when designing the data model, creating or editing relationships, fixing wrong totals caused by filter propagation, deciding whether bidirectional filtering is safe, modeling many-to-many or one-to-one, splitting role-playing dimensions, or preparing the model for DAX, RLS, and time intelligence.
---

# Modeling Semantic Data in Power BI

## Overview

A Power BI semantic model is a set of tables connected by relationships. A **relationship propagates filters** from one table to another along a deterministic path; the cardinality and cross-filter direction of each relationship decide where filters flow and therefore whether report totals are correct. This skill builds that model as a **star schema** (dimension tables for filtering/grouping, fact tables for summarizing) and prescribes the correct relationship settings, date table, and data categories.

Source of truth: the Power BI `guidance/` and `transform-model/` docs. Do not invent UI, properties, or DAX beyond what those docs describe.

## When to Use

Use when you are:
- Designing the model layout (which tables are dimensions vs facts, how they connect).
- Creating or editing relationships in **Manage relationships** / the **Properties** pane / Model view.
- Diagnosing wrong results (e.g., every customer shows the same total, a count is identical across rows, a slicer shows no data) that trace back to filter propagation.
- Deciding whether to set **Cross filter direction** to **Both**.
- Modeling role-playing dimensions (e.g., one `Date` filtering order date, ship date, delivery date).
- Modeling many-to-many or one-to-one relationships.
- Marking a date table for time intelligence, or categorizing geo/URL/image columns.

**Prerequisites:** data is already loaded/shaped. Pair with `transforming-data-power-query` for shaping fact/dimension tables and surrogate keys. This skill feeds `authoring-dax-measures`, `applying-time-intelligence`, `securing-row-level-access`, and `optimizing-model-performance`. For the conceptual dimensional design (grain, additivity, conformed dimensions) cross-reference Kimball `four-step-dimensional-modeling` and `understanding-fact-additivity`.

## Core principle: relationships ARE the model

There is no table property that marks a table as "dimension" or "fact" — the relationship **cardinality** determines the role. In a one-to-many relationship the **"one" side is always a dimension table** and the **"many" side is always a fact table**. A well-designed model gives dimension tables for filtering/grouping and fact tables for summarizing. Build dimensions and facts; never mix the two types in one table (degenerate dimensions are the documented exception — see star-schema build checklist).

> Model relationships do **not** enforce data integrity. Referential-integrity violations surface as blank "unknown member" rows, not errors. Fix them in the source/Power Query, not by hoping a relationship will hide them.

## Star-schema build workflow

Copy this checklist and work top-down:

```
Semantic model build:
- [ ] Step 1 — Classify each table as dimension or fact (by intended role)
- [ ] Step 2 — Ensure each dimension has ONE unique key column (add a surrogate key if needed)
- [ ] Step 3 — Create one-to-many relationships: "one" side = dimension key, "many" side = fact key
- [ ] Step 4 — Keep cross-filter SINGLE by default; flip to BOTH only for the three justified cases
- [ ] Step 5 — Handle role-playing dimensions (duplicate the table, or inactive + USERELATIONSHIP)
- [ ] Step 6 — Model many-to-many (bridge table) and one-to-one (consolidate) cases deliberately
- [ ] Step 7 — Add/mark a Date table; set data types and (for Classic time intelligence) Mark as date table
- [ ] Step 8 — Set data categories (geo/URL/image), hide keys/bridge tables, build hierarchies
- [ ] Step 9 — Verify totals and slicers; confirm no unintended blank members
```

**Step 1–3 — Build the star.** At least one table in each relationship must have unique key values, or you get the error *"One of the columns must have unique values"*. If a dimension lacks a single unique column, add a Power Query **index column** as a surrogate key and merge it into the fact query. The full step-by-step build, snowflake-vs-flatten and degenerate-dimension rules live in [star-schema-build-checklist.md](star-schema-build-checklist.md).

**Step 4 — Cardinality and cross-filter direction.** This is where most modeling mistakes happen. Use the decision matrix in [relationship-design-matrix.md](relationship-design-matrix.md): it maps every cardinality × cross-filter combination to use / caution / avoid, with the documented reason. Quick rules:
- One-to-many, **Single** direction = the default and the safe choice for a classic star.
- **Both** (bidirectional) negatively impacts performance and can create ambiguous filter paths — use only as needed (three justified cases below).
- One-to-one and many-to-many are uncommon and usually signal a suboptimal design; see the matrix.

**Step 5 — Role-playing dimensions.** A role-playing dimension filters facts differently depending on which fact column it joins (e.g., `Date` as order/ship/delivery date). There can be only **one active** relationship between two tables; the rest must be **inactive**. Two patterns, fully worked in [relationship-design-matrix.md](relationship-design-matrix.md):
- **Preferred — one table per role:** duplicate the dimension (calculated table for Import, or duplicate the Power Query query for DirectQuery), give each an active relationship, and prefix columns by role (e.g., `Ship Year`). Allows simultaneous filtering by different roles and works with RLS.
- **Inactive + USERELATIONSHIP:** keep one table, make extra relationships inactive, and activate the needed one inside a measure. Use only when there is no need to filter by two roles at once.

```dax
-- Activate the inactive Date↔ShipDate relationship for this measure only
Orders Shipped =
CALCULATE(
    COUNTROWS(Sales),
    USERELATIONSHIP('Date'[Date], Sales[ShipDate])
)
```

> RLS filters propagate **only** through active relationships — never through inactive ones, even when `USERELATIONSHIP` is used. If RLS roles are defined, prefer the one-table-per-role pattern.

**Step 6 — Many-to-many and one-to-one.** See the matrix. Summary: relate two dimensions through a **bridging table** (a factless fact table) using two one-to-many relationships and exactly **one** bidirectional relationship; do not relate dimensions with raw *:* cardinality. Relate two facts via shared dimensions, not directly. For one-to-one where data spans tables, **consolidate into one table** (merge queries) rather than keeping a 1:1 relationship.

**Step 7 — Date table.** Time intelligence needs a proper date table. Requirements, the four ways to create one (Auto date/time, Power Query connect/generate, DAX `CALENDAR`/`CALENDARAUTO`, clone with DAX), Mark-as-date-table validation, and when marking is required are in [date-table-setup.md](date-table-setup.md).

**Step 8 — Data categorization.** Set a column's **Data category** (Column tools > Properties > Data category) so visuals treat it correctly — e.g., disambiguate a geo code as Country/Region vs US State for maps, or flag Web URL / Image URL columns. Geographic categories are disabled for date/time columns.

## Common pitfalls and corrections

| Pitfall | Symptom | Correction |
|---------|---------|------------|
| No relationship between tables | Same total repeated for every category (e.g., 256 for all) | Create the one-to-many relationship on the shared key |
| Wrong active relationship auto-picked | Visual shows only a subset (e.g., only project sponsors) | Set the intended relationship active in Manage relationships |
| Bidirectional everywhere | Slow queries; "ambiguous path" error; relationship won't commit | Default to Single; use Both only for the three justified cases |
| Bridge dimension filters one way | Each customer shows the grand total (non-additive collapsed) | Set the Account↔bridge relationship to Both so filters reach the fact |
| Relating two dimensions with raw *:* | Duplicate key values, confusing results | Use a bridging table + two 1:* relationships + one bidirectional |
| Relating two fact tables directly (*:*) | Can only group by one shared column; rows dropped on integrity issues | Add shared dimension tables; relate facts via 1:* to them |
| 1:1 with data spanning tables | BLANK groupings when rows don't match; Data pane clutter | Merge queries into one table (left outer join), disable second query's load |
| Date as DateTime with time portion | Date relationship "doesn't match"; time intelligence misbehaves | Remove the time portion in Power Query; set Date/Time or Date type |
| Relying on Auto date/time for a shared calendar | Can't propagate one date table to many fact tables | Build/connect a single date table and Mark as date table |
| Geo code charts on wrong map | AL maps to Albania instead of Alabama | Set Data category (Country/Region vs State or Province) |

## Success criteria

- The model is a star: 1:* relationships from each dimension's unique key to fact keys; no dimension/fact mixing except documented degenerate dimensions.
- Cross-filter direction is **Single** unless a justified case requires **Both** (and that case is documented in the matrix).
- Exactly one active relationship per table pair; role-playing handled by separate tables or inactive + USERELATIONSHIP.
- Many-to-many uses a bridging table; one-to-one cases are consolidated where possible.
- A date table exists, has the required column properties, and is marked as a date table when Classic time intelligence is used.
- Totals and slicers are verified correct; no unintended blank/unknown members from referential-integrity violations.

---
*Provenance: built per SKILL 13 "Modeling Semantic Data" (lifecycle stage: Data pipeline) in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `guidance/star-schema.md`, `transform-model/desktop-create-and-manage-relationships.md`, `desktop-relationships-understand.md`, `desktop-bidirectional-filtering.md`, `desktop-many-to-many-relationships.md`, `desktop-date-tables.md`, `desktop-data-categorization.md`, and `guidance/relationships-active-inactive.md`, `relationships-many-to-many.md`, `relationships-one-to-one.md`, `relationships-bidirectional-filtering.md`, `model-date-tables.md`; cross-references Kimball `four-step-dimensional-modeling`.*
