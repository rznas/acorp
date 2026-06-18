# Query Folding Do / Don't Checklist

**Query folding** is the ability of a Power Query query to generate a **single source query** (for example, one SQL `SELECT`) that retrieves and transforms the data at the source, instead of pulling raw data and transforming it locally in the Power Query mashup engine. Folding pushes work to the source, which is faster and lighter.

## Contents
- Why folding matters (by storage mode)
- Do: practices that preserve folding
- Don't: practices that break folding
- Native SQL query rules
- Detecting and fixing a broken fold
- Quick checklist

---

## Why folding matters (by storage mode)

| Storage mode | Folding requirement |
|---|---|
| **DirectQuery** | **MUST fold.** Every step has to translate to a source query, or the table won't work. |
| **Dual** | **MUST fold** (same requirement as DirectQuery). |
| **Import** | **Should fold** for best data-refresh performance — especially for large semantic models. When the source is relational and a single `SELECT` can be built, ensure folding occurs; otherwise the mashup engine processes the transformations locally. |

For Import, if the mashup engine must still process some transformations, **minimize the work it does**, particularly for large tables.

(Storage-mode selection itself — Import vs DirectQuery vs Dual/Composite — belongs to the `connecting-data-sources` skill.)

---

## Do: practices that preserve folding

- **Delegate as much processing to the data source as possible.** The goal is for the source to do the filtering/joining/aggregating.
- **Filter rows and remove columns early.** Reducing data at the start is foldable and shrinks everything downstream.
- **Find the step that prevents folding**, then, where correct, **move foldable steps earlier** so they get factored into the source query. (The mashup engine may also reorder steps itself when generating the source query.)
- **Prepare/transform data in the source** when steps simply won't fold: write a **database view** that logically transforms the data, or physically **materialize** prepared data in advance — a relational **data warehouse** is the ideal example of pre-integrated, pre-prepared data.
- **Use a native SQL query** for relational transforms that can't be expressed as foldable steps (see rules below).

---

## Don't: practices that break folding

- **Don't add non-foldable steps early in the query.** Once a step can't fold, **no later step folds either** — everything after it runs locally.
- **Don't pile custom/complex logic before foldable steps.** Custom columns and operations the source language can't express are common fold-breakers; keep them after the foldable steps.
- **Don't assume local transforms are "free" on large Import models.** They cost refresh time; prefer source-side work.

> Note: the docs state the general principle ("discover the step that prevents query folding") rather than an exhaustive list of which functions fold. Verify folding behavior per source using the detection method below rather than asserting that a specific transform always or never folds.

---

## Native SQL query rules

When retrieving from a relational source, some sources let you supply a **native SQL query** (any valid statement, including a stored-procedure execution). Use the **`Value.NativeQuery`** M function — it is designed to safely and conveniently pass parameter values.

Critical constraints:
- Once you use a native SQL query, **the mashup engine can't fold later steps** — so put **all** (or as much as possible) transformation logic **inside** the native query statement.
- **DirectQuery**: the native query **must be a `SELECT`** statement; it **can't use Common Table Expressions (CTEs)** or a stored procedure.
- **Incremental refresh can't use a native SQL query** — doing so forces the engine to retrieve all source rows and then filter locally to find incremental changes (defeating the purpose).
- If the statement produces multiple result sets, **only the first is returned**.

> Security: a native SQL query can do more than read — any valid statement may execute (possibly multiple times), including ones that modify or delete data. Apply **least privilege**: the account used should have **read-only** permission on the required data.

---

## Detecting and fixing a broken fold

1. Inspect **Applied Steps** top to bottom and identify the **first** step where folding likely stops (typically the first non-source-expressible transform).
2. If that step could instead be expressed in a single `SELECT` (or in stored-procedure logic), consider a **native SQL query** or a **source view** in its place.
3. Where the step's result is order-independent, **move foldable steps before it** so they fold.
4. If it still can't fold, **push the transform into the source** (view or materialized warehouse table).
5. Re-check, especially for any **DirectQuery/Dual** table — these are not optional: they must fold end to end.

---

## Quick checklist

```
Query folding check:
- [ ] Storage mode known? DirectQuery/Dual => folding is mandatory.
- [ ] Columns removed and rows filtered EARLY in the query.
- [ ] Identified the first step that breaks folding (if any).
- [ ] Foldable steps moved before fold-breaking steps where correct.
- [ ] Custom/complex columns placed AFTER foldable steps.
- [ ] Non-foldable relational logic pushed to a source view / warehouse, OR
      expressed via a native SQL query using Value.NativeQuery.
- [ ] Native query for DirectQuery is a single SELECT (no CTE, no stored proc).
- [ ] Native query NOT combined with incremental refresh.
- [ ] Source account has read-only (least-privilege) access for native queries.
- [ ] DirectQuery/Dual tables fold end to end (verified, not assumed).
```
