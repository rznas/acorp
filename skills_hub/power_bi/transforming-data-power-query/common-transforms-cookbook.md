# Common Transforms Cookbook (Power Query Editor)

A task-oriented reference for the transforms used most when cleaning and shaping data in the Power BI Desktop Power Query Editor. Each entry gives the UI path, what it does, the M function it generates (visible in the formula bar / Advanced Editor), and gotchas.

> All transforms are recorded as ordered **applied steps**. The source data is never changed — steps replay on each refresh. Undo any step with its **X** in **Applied Steps**.

## Contents
- Column cleanup: remove, rename, reorder, choose
- Change data type
- Filter and remove rows
- Replace values
- Split column
- Group By (summarize)
- Pivot Column
- Unpivot Columns
- Custom Column
- Merge queries (join)
- Append queries (union)
- Combine Files (folder of same-schema files)
- Viewing and editing M (formula bar / Advanced Editor)

---

## Column cleanup: remove, rename, reorder, choose

**Remove columns** — Select the column(s), then **Home > Remove Columns** (or right-click the header > **Remove Columns**). Removing unneeded columns early makes the model smaller and refresh faster, and is often foldable.

**Rename** — Double-click the header, or right-click > **Rename**. Use business-friendly names (for example, `ProductID` -> `Product ID`).

These appear in **Applied Steps** as `Removed Columns` and `Renamed Columns`. Typical generated M:

```m
= Table.RemoveColumns(Source, {"InternalNotes", "ETL_LoadTime"})
= Table.RenameColumns(#"Removed Columns", {{"ProductID", "Product ID"}})
```

Gotcha: many right-click menu items duplicate ribbon buttons — they produce the same step either way.

---

## Change data type

Select a column, then **Transform > Data Type**, or click the type icon at the left of the column header and pick the type. Power BI auto-detects types on load as a **Changed Type** step; fix wrong detections by editing that step rather than adding a second type change.

```m
= Table.TransformColumnTypes(Source, {{"OrderDate", type date}, {"Amount", type number}, {"Qty", Int64.Type}})
```

Gotcha: a wrong type (for example, a key read as a number with a leading-zero loss, or a date read as text) corrupts sorts, relationships, and aggregations downstream. Verify types before reshaping.

---

## Filter and remove rows

**Filter** — Click the column's dropdown arrow and clear (uncheck) the values to exclude, or use the filter conditions; **OK** applies. Filtering early reduces data volume and usually folds.

```m
= Table.SelectRows(Source, each ([Region] <> "Test") and ([Amount] > 0))
```

**Remove top/bottom rows** — You can remove any number of rows from the top or bottom (useful for stripping report headers/footers from spreadsheet sources).

---

## Replace values

Right-click a column > **Replace Values** (or **Home/Transform > Replace Values**). Replace a found value with a new value across the column.

```m
= Table.ReplaceValue(Source, "N/A", null, Replacer.ReplaceValue, {"Status"})
```

Use to normalize codes, clear placeholder text, or standardize categories before building a dimension.

---

## Split column

**Transform > Split Column** — split by delimiter, by number of characters, by position, or by transitions. Common for breaking `City, State` or compound keys into separate columns.

```m
= Table.SplitColumn(Source, "Location", Splitter.SplitTextByDelimiter(", "), {"City", "State"})
```

---

## Group By (summarize)

Select the column to group on, then **Transform > Group By** (also on **Home**). In the dialog:
1. The unlabeled dropdown sets the column to group by (defaults to the selected column; changeable).
2. **New column name** — name for the aggregated result.
3. **Operation** — **Sum**, **Median**, **Count Distinct Rows**, **Count Rows** (default), etc.
4. **Advanced** unlocks **Add grouping** and **Add aggregation** so you can group by several columns and produce several aggregations in one step. Remove a row via its `...` ellipsis > **Delete**.

```m
= Table.Group(Source, {"State Abbr"}, {{"Agency Count", each Table.RowCount(_), Int64.Type}})
```

Use Group By to roll a transactional source up to a coarser grain (for example, daily -> monthly), or to count distinct entities.

---

## Pivot Column

Select the column whose unique values should become new column headers, then **Transform > Pivot Column**. The dialog confirms which column supplies the **values**; expand **Advanced options** to choose the aggregate function applied to those values.

```m
= Table.Pivot(Source, List.Distinct(Source[CategoryName]), "CategoryName", "ProductCount", List.Sum)
```

Use to turn a tall category column into a cross-tab (for example, a count of products per category as columns).

---

## Unpivot Columns

Select the columns to collapse, then **Transform > Unpivot Columns** (variants: Unpivot Other Columns, Unpivot Only Selected Columns). Turns a wide "one column per period/metric" layout into tidy **attribute** + **value** rows — essential for building a clean, long fact table.

```m
= Table.UnpivotOtherColumns(Source, {"Product"}, "Month", "Sales")
```

Example: columns `Jan, Feb, Mar` become rows of `Month / Sales`, leaving `Product` as the identifier. This is the inverse of Pivot.

---

## Custom Column

**Add Column > Custom Column** — write an M expression that operates over other columns and produces a new column. Example: a percentage from two columns.

```m
= Table.AddColumn(Source, "Percent ELL", each [ELL] / [TotalStudents], type number)
```

Gotcha: custom columns are a common **query-folding breaker** when the logic can't be expressed in the source language — add them after the foldable steps (see query-folding-checklist.md). As with any step, delete the **Added Custom** step in **Applied Steps** if it isn't what you want.

---

## Merge queries (join)

**Home > Merge queries** (merge into the current query) or **Merge queries as new** (create a new query). In the dialog:
- Pick the first table and the second table.
- Select the matching key column(s) in each (click the same column header in both).
- Choose a join kind, for example **Left Outer (all from first, matching from second)** — the default.
- **OK** creates a merged column; click its **expand** icon and select which columns to bring in.

```m
= Table.NestedJoin(Sales, {"Product ID"}, Returns, {"Product ID"}, "Returns", JoinKind.LeftOuter)
= Table.ExpandTableColumn(#"Merged Queries", "Returns", {"Return Reason"}, {"Return Reason"})
```

Use Merge to attach a foreign key or a few descriptive columns to a fact, or to look up values. Expand **only** the columns you need to keep the table lean.

---

## Append queries (union)

**Home > Append queries** (append to current) or **Append queries as new**. Stacks two or more queries that share the **same schema** (same columns) into one table — the SQL `UNION` equivalent.

```m
= Table.Combine({Sales2023, Sales2024})
```

Use Append to union same-grain sources, such as one query per year or per region, into a single fact table.

---

## Combine Files (folder of same-schema files)

When you have **multiple files with the same schema**, combine them into one logical table:
1. **Get data > File > Folder > Connect**; enter the folder path; **OK**; **Transform data** to see the file list in the editor.
2. Select the **Content** column, then **Home > Combine Files** (or the **Combine Files** icon next to **Content**).

The Combine Files transform:
- Analyzes each input file to determine its format (text, Excel workbook, JSON).
- Lets you pick a specific object from the first file (for example, a worksheet, or the root to combine the whole file).
- Auto-creates an **example (exemplar) query** that performs the extraction on a single file, a linked **function query** that parameterizes the file input, and applies that function to every row of the folder query — expanding the result as top-level columns.

To add more extraction/transform logic, edit the **exemplar query**; changes flow automatically into the linked function query.

Gotcha: all files must share the **same file type and structure** (same columns). Your selection scope inside an Excel workbook (a specific worksheet vs. the root) changes the combine behavior.

---

## Viewing and editing M (formula bar / Advanced Editor)

- **Formula bar** — turn on with **View > Formula Bar**. Shows the M for the currently selected step; edit a single step inline.
- **Advanced Editor** — **View > Advanced Editor**. Shows the full M (`let ... in`) for the whole query; edit any step or write code directly in the **Power Query M formula language**. Close with **Done** or **Cancel**.

Power Query keeps every applied step as editable M text. For the complete function set, consult the Power Query M function reference (linked from the docs); only use functions confirmed there rather than inventing function names.
