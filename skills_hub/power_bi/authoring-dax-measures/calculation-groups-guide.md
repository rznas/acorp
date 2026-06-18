# Calculation Groups Guide

Calculation groups let you define DAX once as **calculation items** and apply it to any measure — eliminating dozens of near-duplicate measures (the classic case: one set of time-intelligence variants instead of YTD/PY/YoY% per measure). Grounded in `calculation-groups.md`.

## Contents
- What a calculation group is
- Creating one (Model view, and TMDL)
- SELECTEDMEASURE placeholder
- Using a calculation group in reports and measures
- Dynamic format strings on calculation items
- Precedence (multiple groups)
- Selection expressions
- Considerations and pitfalls

---

## What a calculation group is

- A **calculation group** contains **calculation items**, each a DAX expression that transforms whatever measure it is applied to.
- It surfaces a **calculation group column** (used on a visual axis/columns or in a slicer) whose values are the item names.
- Applying an item to a measure produces the transformed result — e.g. a "YTD" item turns `[Sales]` into year-to-date sales, `[Orders]` into year-to-date orders, with no per-measure code.

## Creating one

In **Model view**, select the **Calculation group** button in the ribbon. If **Discourage implicit measures** is off, you're prompted to turn it on (required — see pitfalls). Then define each item's DAX in the formula bar.

You can also author it in **TMDL view** (source-control-friendly):

```TMDL
createOrReplace

    table 'Calculation group'

        calculationGroup
            precedence: 1

            calculationItem 'Calculation item' = SELECTEDMEASURE()

        column 'Calculation group column'
            dataType: string
            summarizeBy: none
            sourceColumn: Name
            sortByColumn: Ordinal

            annotation SummarizationSetBy = Automatic

        column Ordinal
            dataType: int64
            formatString: 0
            summarizeBy: sum
            sourceColumn: Ordinal

            annotation SummarizationSetBy = Automatic
```

Rename the group, its column, and items by double-clicking in the **Data** pane or via the **Properties** pane. Reorder items via the Properties pane or right-click context menu. Add items via right-click **New calculation item**.

## SELECTEDMEASURE placeholder

`SELECTEDMEASURE()` is the placeholder for the measure the item is applied to. Example time-intelligence items:

```dax
calculationItem 'Current'  = SELECTEDMEASURE()
calculationItem 'YTD'      = TOTALYTD(SELECTEDMEASURE(), 'Date'[Date])
calculationItem 'PY'       = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))
calculationItem 'YOY%'     =
    VAR Current = SELECTEDMEASURE()
    VAR Prior   = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))
    RETURN DIVIDE(Current - Prior, Prior)
```

> If items reference a date table, use the actual table name in your model (the docs note errors when an example used `DimDate` but the model's table was `Date`). For the time-intelligence items themselves, defer to `applying-time-intelligence`.

## Using a calculation group in reports and measures

**In a report:** put the **calculation group column** on a matrix's columns (or in a slicer), and a measure (e.g. `Orders`) on values — each item shows that measure transformed.

**In a measure:** apply a specific item with `CALCULATE` filtering the calculation-group column:

```dax
Orders YOY% =
    CALCULATE(
        [Orders],
        'Time Intelligence'[Time Calculation] = "YOY%"
    )
```

## Dynamic format strings on calculation items

Items use the underlying measure's formatting by default. To override (e.g. show "YOY%" as a percentage), select the item, turn on **Dynamic format string** in the Properties pane, and supply a format expression — e.g. `#,##0.00%`.

## Precedence (multiple groups)

If a model has more than one calculation group, set the **precedence** property on each to control the order in which they apply to a measure.

## Selection expressions

Set **selection expressions** on a calculation group for fine-grained control of what it returns when users make multiple, invalid, or no selections.

## Considerations and pitfalls

- **Discourage implicit measures is forced on.** Once a calculation group exists, Power BI stops showing the summation symbol and blocks dragging raw columns onto visuals as values. Calculation items apply only to **explicit measures**, not implicit ones. Existing implicit measures already in visuals keep working.
- **Measures become the variant data type** while any calculation group exists (they revert if all groups are removed). This can break dynamic format strings that reuse a measure — fix with `FORMAT([Dynamic format string], "")`, or use a DAX user-defined function.
- **Math on a non-numeric measure errors** ("Cannot convert value … of type Text to type Numeric"). Guard with `ISNUMERIC`:
  ```dax
  Calculation item safe =
      IF(
          ISNUMERIC(SELECTEDMEASURE()),
          SELECTEDMEASURE() * 2,
          SELECTEDMEASURE()
      )
  ```
- Items can be written to **ignore a measure by name or data type** for measures you don't want transformed.
