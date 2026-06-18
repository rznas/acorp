# Relationship Design Decision Matrix

Decision reference for choosing **cardinality** and **cross-filter direction** in Power BI Desktop relationships, plus worked patterns for role-playing dimensions, many-to-many, and one-to-one. All guidance is grounded in the Power BI relationship docs.

## Contents
- Cardinality reference (the four types)
- Cross-filter direction reference (Single vs Both)
- Decision matrix: cardinality × cross-filter → use / caution / avoid
- When bidirectional (Both) is justified — the three cases
- Bidirectional risks
- Role-playing dimensions (two patterns + USERELATIONSHIP)
- Many-to-many patterns (dimensions, facts, higher-grain)
- One-to-one patterns
- Regular vs limited relationships (why it matters)

## Cardinality reference

Set in the **Create/Edit relationship** dialog or **Properties** pane. The "one" side must contain unique values; the "many" side may contain duplicates. A refresh that loads duplicate values into a "one" side column fails.

| Cardinality | Notation | Meaning | Typical use |
|-------------|----------|---------|-------------|
| Many-to-one | *:1 | many side related to one side | Most common; fact → dimension |
| One-to-many | 1:* | one side related to many side | Most common; dimension → fact (same thing, opposite order) |
| One-to-one | 1:1 | both columns unique | Uncommon; usually suboptimal (redundant data) |
| Many-to-many | *:* | both columns may duplicate | Infrequent; relate *:* facts or higher-grain facts |

Power BI auto-detects cardinality at relationship creation (Import: internal storage stats; DirectQuery: profiling queries). It can guess wrong before data is loaded or when sample data is coincidentally unique — override it if you know future data will introduce duplicates.

## Cross-filter direction reference

| Cardinality | Allowed cross-filter options |
|-------------|------------------------------|
| One-to-many / Many-to-one | Single, Both |
| One-to-one | Both (always — cannot be configured otherwise) |
| Many-to-many | Single (Table1→Table2), Single (Table2→Table1), Both |

- **Single** = filters flow one direction only, toward the table where values are aggregated. The default and the safe choice. When a cardinality includes a "one" side, filters always propagate from that "one" side.
- **Both** (bi-directional) = both tables filter each other. Required for one-to-one; needed for specific bridge and slicer scenarios. When set to Both, an extra option *Apply security filter in both directions* appears for RLS.

You can also override direction at calculation time without changing the relationship, using the `CROSSFILTER` DAX function (set to one direction, both, or none).

## Decision matrix: cardinality × cross-filter → use / caution / avoid

| Cardinality | Cross-filter | Verdict | When / why |
|-------------|-------------|---------|-----------|
| 1:* (or *:1) | **Single** | **USE** (default) | Classic star schema: dimension → fact. Fastest filter propagation; deterministic; works with RLS. |
| 1:* (or *:1) | **Both** | **CAUTION** | Only for the three justified cases below. Hurts query performance; can create ambiguous paths Power BI will refuse to commit. Avoid when two+ fact tables share lookup tables. |
| 1:1 | **Both** (forced) | **CAUTION → usually AVOID** | Signals data redundancy. If data spans tables, consolidate into one table instead. Use only for degenerate-dimension separation or unavoidable cross-source splits. |
| *:* (dimension↔dimension, direct) | any | **AVOID** | Implies duplicate keys on a dimension. Use a bridging table with two 1:* relationships + one bidirectional instead. |
| *:* (fact↔fact, direct) | Single | **CAUTION** | Quick exploration only. You can only group by the one shared column, and integrity issues silently drop rows (limited relationship). Prefer shared dimensions. |
| *:* (higher-grain fact, non-date) | Single (dim→fact) | **USE (with measure logic)** | When a fact is stored at a higher grain than the dimension (e.g., targets by category). Control summarization with measures that BLANK out lower levels. |
| any | Both, in a model with loops | **AVOID** | Ambiguous propagation. Break the loop (duplicate a table to make a star) or mark a relationship inactive, then Both may be allowed. |

## When bidirectional (Both) is justified — the three cases

Per the bidirectional guidance, minimize Both. It is justified only for:

1. **Special model relationships.** One-to-one relationships are always bidirectional (not configurable). Many-to-many between two dimension tables needs a **bridging table** with one bidirectional relationship so filters cross the bridge.
2. **Slicer options "with data."** Both can limit slicer options to values that have related data. *Preferred alternative:* instead of a bidirectional relationship, add a visual-level filter on the slicer using a measure with the "is not blank" condition:
   ```dax
   Total Quantity = SUM(Sales[Quantity])
   ```
   Then filter the slicer by `Total Quantity` is not blank.
3. **Dimension-to-dimension analysis** (treat a fact as a bridge to count one dimension within another's context). *Preferred alternative:* don't make the relationship bidirectional in the model — activate it per-measure with `CROSSFILTER`:
   ```dax
   Different Countries Sold =
   CALCULATE(
       DISTINCTCOUNT(Customer[Country-Region]),
       CROSSFILTER(Customer[CustomerCode], Sales[CustomerCode], BOTH)
   )
   ```

## Bidirectional risks

- **Performance:** bidirectional relationships require more processing; impact grows as the number of Both relationships increases.
- **Ambiguity:** can create multiple filter paths. Power BI may refuse to commit the change (error), or may allow an ambiguous model that resolves paths by priority then weight — surprising results.
- **Confusing UX:** slicer options appearing/disappearing as users interact confuses report consumers.
- **RLS:** Both exposes the *Apply security filter in both directions* option; bidirectional security filtering increases the security surface and processing. Combine with RLS only deliberately (see `securing-row-level-access`).

## Role-playing dimensions (two patterns)

A role-playing dimension filters facts by different roles — e.g., a `Date` table used as order date, ship date, delivery date; or an `Airport` table used as departure and arrival. Only **one active** relationship can exist between two tables; others must be inactive.

### Pattern A — One table per role (PREFERRED)
Duplicate the dimension so each role has its own table with an **active** relationship.
- Import: create a **calculated table** that clones the source, e.g.:
  ```dax
  Departure Airport = 'Arrival Airport'
  ```
  For a date role: `Ship Date = 'Date'`
- DirectQuery: duplicate the Power Query query.
- Rename columns by role so visuals are self-describing (`Ship Year`, `Arrival Airport Name`).
- Add table descriptions so report authors see how each table filters (Data pane tooltip).

Benefits: simultaneous filtering by multiple roles; report authors can use implicit summarization (no special measures); works with RLS. Cost: minor model-size increase (dimensions are small relative to facts), which overrides data-reduction guidance here.

> Cloning with DAX copies only columns — not formats, descriptions, or hierarchies. Reapply those on the clone.

### Pattern B — Inactive relationship + USERELATIONSHIP
Keep one dimension table; make extra relationships **inactive** (dashed line in Model view); activate the needed one inside a measure.
```dax
Orders = COUNTROWS(Sales)                 -- uses the active OrderDate relationship

Orders Shipped =
CALCULATE(
    COUNTROWS(Sales),
    USERELATIONSHIP('Date'[Date], Sales[ShipDate])   -- activates inactive ShipDate relationship
)
```
Use only when: there is no need to filter by two roles at the same time, AND you will write the measures. Limitations: report authors can't summarize via inactive relationships without a report-level measure; you can't plot two roles simultaneously; **RLS never propagates through inactive relationships**.

> Inactive relationships are still expanded at query time (table expansion), so a slicer on the role-playing dimension can surface a BLANK option when the many-side key is blank (e.g., unshipped orders).

## Many-to-many patterns

### Relate two dimensions (e.g., customers ↔ accounts)
- Add both dimensions (each with a unique ID) and a **bridging table** (factless fact table) storing one row per association.
- Create two **one-to-many** relationships: Account 1:* AccountCustomer, Customer 1:* AccountCustomer.
- Set **one** relationship to **Both** so filters propagate through the bridge all the way to the fact table (without it, every customer shows the grand total).
- Hide the bridging table and surrogate ID columns; if an ID stays visible, keep it on the "one" side.
- Expect **non-additive** customer balances (a balance can belong to multiple accounts) — communicate this to users.

### Relate two facts (e.g., Order ↔ Fulfillment)
Direct *:* between facts is **not recommended**: you can only group by the one shared column and integrity issues drop rows. Instead implement a **star schema** — add shared dimension tables (`OrderLine`, `OrderDate`, `Product`, `FulfillmentDate`) with 1:* relationships to both facts. Then you can filter/group by any dimension column and summarize either fact, and all relationships are regular.

### Relate higher-grain facts (e.g., Target by category/year)
When a fact is stored at a higher grain than the dimension:
- **Higher-grain dates:** store the first date of the period (e.g., Jan 1 for a year) and use a normal 1:* date relationship. Control summarization so lower levels return BLANK:
  ```dax
  Target Quantity =
  IF(
      NOT ISFILTERED('Date'[Date]) && NOT ISFILTERED('Date'[Month]),
      SUM(Target[TargetQuantity])
  )
  ```
- **Higher-grain non-date:** use a *:* relationship filtering single direction (dimension → fact), and BLANK out levels below the join grain:
  ```dax
  Target Quantity =
  IF(
      NOT ISFILTERED('Product'[ProductID])
          && NOT ISFILTERED('Product'[Product])
          && NOT ISFILTERED('Product'[Color]),
      SUM(Target[TargetQuantity])
  )
  ```
- For both: hide summarizable fact columns so only measures summarize them.

## One-to-one patterns

One-to-one is uncommon and usually suboptimal. Two legitimate scenarios:
- **Degenerate dimension:** split filtering/grouping columns (e.g., order number + line) out of the fact into a separate table keyed by a surrogate (`OrderLineNumberID`). Reduces storage, simplifies calculations, cleaner Data pane.
- **Row data spans tables:** the same entity loaded as two tables. **Recommendation: consolidate into one table** by merging the Power Query queries (left outer join keeping the complete table), disable the second query's load, replace nulls with a token (e.g., `[Undefined]`), and build hierarchies / display folders. Avoid the 1:1 relationship — it clutters the Data pane, blocks cross-table hierarchies, and produces BLANKs when rows don't match.
  - *Intra source group* 1:1 is a regular relationship → integrity gaps show as BLANKs.
  - *Cross source group* 1:1 is a **limited** relationship → unmatched rows are eliminated (rows silently disappear). Ensure matching rows on both sides or pre-consolidate at the source.

## Regular vs limited relationships (why it matters)

Not a setting — inferred from cardinality and source.
- **Regular:** the engine knows the "one" side (all 1:* intra-source-group relationships). Enables table expansion with LEFT OUTER JOIN semantics; integrity violations appear as blank unknown members. `RELATED` works. Fastest.
- **Limited:** no guaranteed "one" side — either *:* cardinality OR cross-source-group (composite models). Uses INNER JOIN semantics, so unmatched rows are dropped (no blank members). `RELATED` can't fetch the "one" side; RLS has topology restrictions.

Filter-propagation performance, fastest to slowest: 1) 1:* intra-source-group; 2) *:* via intermediary table with at least one bidirectional relationship; 3) *:* cardinality relationships; 4) cross-source-group relationships. Prefer 1:* single-direction for everything you can.
