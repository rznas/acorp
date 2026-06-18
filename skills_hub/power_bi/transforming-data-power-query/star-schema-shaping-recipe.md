# Star-Schema Shaping Recipe (Power Query)

How to transform raw, often wide or fully-denormalized operational source tables into clean **fact** and **dimension** tables — inside Power Query Editor, before they load into the model. This sets up a proper star schema so relationships, DAX, and performance work well in the downstream `modeling-semantic-data` skill.

The design thinking is the Kimball **four-step dimensional modeling** process — applied here as a Power Query shaping plan. Cross-reference the DW skill `four-step-dimensional-modeling` for the full method; this recipe maps each step to concrete Power Query actions.

## Contents
- The four-step plan applied in Power Query
- Recipe A: split one wide table into fact + dimensions
- Recipe B: build a dimension table (dedupe to one row per entity)
- Recipe C: build a fact table (keys + measures only)
- Recipe D: union same-grain sources (Append / Combine Files)
- Recipe E: unpivot a wide layout into a tidy fact
- Staging queries and load settings
- Checklist

---

## The four-step plan applied in Power Query

| Kimball step | Question | Power Query action |
|---|---|---|
| 1. Select business process | Which operational activity? | Identify the source table(s) for that process (for example, sales transactions). |
| 2. Declare the grain | What does ONE fact row represent? | Decide the fact query's grain (for example, "one row per product per order line"). Group/filter so the query matches that grain. |
| 3. Identify dimensions | Who / what / where / when / why / how? | Split descriptive attributes into separate **dimension** queries, one row per entity. |
| 4. Identify facts | What is being measured? | Keep numeric measurements + foreign keys in the **fact** query; set correct numeric types. |

Keep additivity in mind when choosing facts: additive measures (quantities, amounts) are ideal; for ratios, keep the numerator and denominator as separate additive columns and compute the ratio later in DAX (cross-reference the `understanding-fact-additivity` skill). Do not pre-divide a ratio in Power Query if you will need to re-aggregate it.

---

## Recipe A: split one wide table into fact + dimensions

Start: one wide source table mixing measures and descriptive attributes (for example, a flat sales export with product name, category, customer city, and amounts all in one table).

1. **Reference** the source query once per output table (right-click query > **Reference**), or duplicate. Reference avoids re-typing logic, but remember each referencing query re-reads the source — use a **dataflow** if that source/transform is expensive and shared (see SKILL.md "Referenced queries").
2. In the **fact** reference: **Remove Columns** for all descriptive attributes, keeping the key(s) and the numeric measures. Set numeric types.
3. In each **dimension** reference: keep only that dimension's key + its descriptive attributes; **Remove Duplicates** on the key (Recipe B).
4. Disable load on the original wide query if it is only a staging input.

Result: a narrow fact query plus one query per dimension, all ready to relate in the model.

---

## Recipe B: build a dimension table (dedupe to one row per entity)

1. Start from the source (or a reference of it).
2. **Choose / Remove Columns** so only the dimension key and its descriptive attributes remain.
3. Right-click the key column > **Remove Duplicates** (or **Home > Remove Rows > Remove Duplicates**) so there is exactly **one row per entity**.

   ```m
   = Table.Distinct(#"Removed Other Columns", {"Product ID"})
   ```
4. **Rename** columns to business-friendly names and **set types** (text for descriptive attributes).
5. (Optional) **Replace Values** to standardize categories, and **Split Column** to break compound attributes (for example, `City, State`).

Keep dimensions **flat/denormalized** (the descriptive attributes live together on the one table) rather than snowflaked into many lookup tables — this matches star-schema design.

---

## Recipe C: build a fact table (keys + measures only)

1. Start from the source (or a reference).
2. **Remove** descriptive attribute columns — they belong on dimensions.
3. Keep the **foreign keys** (one per dimension) and the **numeric measures**.
4. **Change Type** so every measure is the correct numeric type and every key matches its dimension key's type (mismatched key types break relationships downstream).
5. **Filter** out rows that don't belong at the grain (test rows, zero/placeholder rows as appropriate).
6. If a needed foreign key isn't present, **Merge** it on from another query (Recipe in the cookbook): join on the natural key and expand **only** the key column.

```m
= Table.NestedJoin(Fact, {"ProductCode"}, DimProduct, {"ProductCode"}, "DimProduct", JoinKind.LeftOuter)
= Table.ExpandTableColumn(#"Merged", "DimProduct", {"Product ID"}, {"Product ID"})
```

---

## Recipe D: union same-grain sources (Append / Combine Files)

When the fact data arrives split across same-schema sources at the **same grain** (for example, one file or query per month/region):

- **Append queries** (**Home > Append queries** / **Append queries as new**) to stack queries with identical columns into one fact.
- **Combine Files** when the sources are many same-schema files in a **folder** (CSV/Excel/JSON): **Get data > File > Folder**, then **Combine Files** (see cookbook). All files must share the same file type and structure.

Shape and type the combined result once (edit the **exemplar query** for Combine Files), then proceed with Recipe C cleanup.

---

## Recipe E: unpivot a wide layout into a tidy fact

Operational exports often store one column per period or metric (for example, `Jan, Feb, Mar` sales columns). A fact table needs one measurement per row:

1. Select the identifier column(s) to keep (for example, `Product`).
2. **Transform > Unpivot Other Columns** to collapse the period columns into `Attribute` / `Value` rows.
3. **Rename** the attribute/value columns meaningfully (for example, `Month` / `Sales`) and set types.

```m
= Table.UnpivotOtherColumns(Source, {"Product"}, "Month", "Sales")
```

Now each row is one product-month measurement — a clean grain you can relate to a Date and Product dimension.

---

## Staging queries and load settings

- Right-click intermediate/input-only queries > uncheck **Enable load** so they don't load into the model (less clutter, smaller file).
- Referencing a staging query does **not** compute it once and reuse it — its steps run inside every referencing query. For a shared, expensive source/transform, build a **dataflow** so it is evaluated once.
- For files with many queries, enable **File > Options and settings > Options > Data Load > Allow data preview to download in the background** to speed refresh, and **Home > Refresh Preview** to update stale previews.

---

## Checklist

```
Star-schema shaping check:
- [ ] Business process and fact grain declared ("one row = ...").
- [ ] Facts = numeric measures + foreign keys only; correct numeric types.
- [ ] Descriptive attributes moved to dimension queries (not on the fact).
- [ ] Each dimension deduped to one row per entity (Remove Duplicates on key).
- [ ] Dimensions kept flat/denormalized (not snowflaked).
- [ ] Foreign key types match their dimension key types.
- [ ] Same-grain sources unioned (Append / Combine Files); not mixed grains.
- [ ] Wide period/metric columns unpivoted into tidy rows where needed.
- [ ] Ratios kept as numerator + denominator (computed later in DAX).
- [ ] Staging/input queries have load disabled; shared expensive logic in a dataflow.
- [ ] Output queries ready for modeling-semantic-data (relationships next).
```
