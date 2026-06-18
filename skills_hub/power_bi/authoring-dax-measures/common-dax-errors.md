# Common DAX Errors and Corrections

The recurring DAX mistakes and how to fix them. Each row: the error, the symptom you'd notice, and the correction.

## Contents
- Wrong-object errors
- Aggregation / additivity errors
- Ratio errors
- Context errors
- Syntax and reference errors
- Calculation-group gotchas

---

## Wrong-object errors

| Error | Symptom | Correction |
|-------|---------|------------|
| Calculation written as a **calculated column** that should be a measure (e.g. a stored margin %) | Value is static; doesn't change when sliced; subtotal/total is wrong | Make it a **measure** so it re-aggregates in filter context: `Gross Margin % = DIVIDE([Total Profit], [Total Sales])` |
| **Measure** used where you need to group/slice (on an axis, legend, slicer, or relationship key) | Field can't be placed on the axis / can't form a relationship | Use a **calculated column** — only stored per-row values can sit on axes or key relationships |
| High-cardinality **calculated column** used only to feed a SUM | Model bloats, refresh slows | Replace with a **measure + iterator**: `SUMX(Sales, Sales[Qty] * Sales[Price])` |
| **Calculated table** expected to refresh under DirectQuery | Stale data | Calculated tables reflect changes only after a semantic-model refresh; keep them in DirectQuery if the source is DirectQuery |

## Aggregation / additivity errors

| Error | Symptom | Correction |
|-------|---------|------------|
| `SUM` of a semi-additive **balance** over a date axis | Inventory/cash total grows every day; year total ≈ N× reality | Average over distinct dates or take a period-end value: `AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(Inventory[QtyOnHand])))` or `LASTNONBLANK` pattern |
| `AVERAGE(QtyOnHand)` for "average daily inventory" | Skewed by row count (products × days), not day count | `AVERAGEX(VALUES('Date'[Date]), CALCULATE(SUM(...)))` divides by distinct days |
| `SUM` of unit prices | Meaningless total price | Store/sum the **extended amount** (qty × price); price-per-unit is a `DIVIDE` ratio |
| Implicit measure relied on after adding a calculation group | Column blocked from visuals; calc item doesn't apply | Create an **explicit measure**; calculation items apply only to explicit measures |

This category is the most consequential. When a fact comes from a Kimball periodic-snapshot table (DW `fact-table-design-patterns`), its balance facts are semi-additive — confirm classification with `understanding-fact-additivity` before writing the aggregation.

## Ratio errors

| Error | Symptom | Correction |
|-------|---------|------------|
| Using `/` for a ratio | Division-by-zero throws an error or shows Infinity | `DIVIDE(num, den)` returns blank on zero; `DIVIDE(num, den, 0)` for an explicit fallback |
| **Average of per-row ratios** (e.g. AVG of stored margin %) | Small and large items weighted equally; wrong total | **Ratio of sums**: `DIVIDE(SUM(Profit), SUM(Revenue))` |
| Stored 0 instead of blank for inapplicable facts | Averages deflated, counts inflated | Leave inapplicable facts blank; `SUM`/`AVERAGE`/`COUNT` ignore blanks |

## Context errors

| Error | Symptom | Correction |
|-------|---------|------------|
| Expecting row-by-row behavior in a bare measure | Result aggregates the whole column instead of per-row math | Use an **iterator** (`SUMX`/`AVERAGEX`) to create row context |
| Forgetting `CALCULATE`/iterator when referencing a measure per row | Same value repeats for every row; no context transition | Wrap in `CALCULATE` (or use an iterator) so row context transitions to filter context |
| `ALL` removing more filters than intended | A "% of total" denominator ignores slicers users expect to apply | Use `ALLSELECTED(col)` or `ALL(specific column)` instead of `ALL(whole table)` |
| Calculation ignores an inactive (role-playing) relationship | Sales show against the wrong date (e.g. order date not ship date) | `CALCULATE([Total Sales], USERELATIONSHIP(Sales[ShipDateKey], 'Date'[DateKey]))` |
| Surprising total when filtering **two columns of the same table** (auto-exist) | An "across all" measure drops non-existing combinations and returns less than expected | Understand value-filter behavior; the model **Value filter behavior** setting (Automatic/Independent/Coalesced) controls it. **Preview feature** — verify current behavior in docs |

## Syntax and reference errors

| Error | Symptom | Correction |
|-------|---------|------------|
| Missing/misplaced closing parenthesis | Syntax error in the formula bar | Most syntax errors are an unbalanced `)`; check parentheses first |
| Table name with spaces not quoted | Invalid reference | Wrap in single quotes: `'Reseller Sales'[SalesAmount]` |
| Column referenced bare like a measure | Wrong reference resolved | Fully qualify columns as `Table[Column]`; reference measures bare as `[Measure]` |
| Locale uses comma as decimal separator with quick measures | Quick measure DAX doesn't work | Quick-measure DAX uses commas only as argument separators; this conflicts with comma-decimal locales |
| Time-intelligence quick measure attempted in DirectQuery | Not offered / fails | Time-intelligence quick measures aren't available in DirectQuery mode |

## Calculation-group gotchas

| Error | Symptom | Correction |
|-------|---------|------------|
| Math calc item applied to a non-numeric measure | "Cannot convert value … of type Text to type Numeric" | Guard with `ISNUMERIC(SELECTEDMEASURE())` before applying math |
| Dynamic format string breaks after adding a calc group | Format error; measures became **variant** type | Wrap with `FORMAT([…], "")` to coerce back to string, or use a DAX user-defined function |
| Multiple calculation groups apply in the wrong order | Unexpected combined result | Set the **precedence** property on each calculation group |
