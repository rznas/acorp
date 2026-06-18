---
name: drill-pattern-design
description: Designs drill-down, drill-through, and drill-across navigation in Power BI reports so interactivity follows dimensional-model hierarchies and conformed dimensions. Use when building interactive reports that need hierarchical navigation (Year to Quarter to Month), summary-to-detail exploration (click a product to see its transactions), or multi-fact comparison (sales next to inventory). Triggers include drill down, drill up, drill through, drill across, decomposition tree, matrix drill, detail page, breadcrumbs, back button, report tooltips, conformed dimensions, and drill-through performance.
---

# Drill Pattern Design for Power BI

Design three distinct interaction patterns so report navigation mirrors the dimensional model: **drill-down** (move along one dimension's hierarchy), **drill-through** (jump from a summary value to a filtered detail page), and **drill-across** (compare metrics from different fact tables joined on conformed dimensions). Picking the wrong pattern produces reports that either dead-end at summaries or overcount semi-additive facts.

> Prerequisite skills: `visualizing-dimensional-models` (grain, fact/dimension structure, hierarchies) and a working grasp of conformed dimensions. Drill-across in particular is meaningless without conformed dimensions.

## Choose the pattern first

| User need | Pattern | Power BI mechanism |
|-----------|---------|--------------------|
| "Show sales by year, then expand into quarters/months" | Drill-down | Date hierarchy on a chart axis, matrix rows, or decomposition tree |
| "Click a product to see its individual transactions" | Drill-through | Detail page filtered by the clicked value |
| "Show inventory level AND sales for the same product" | Drill-across | Separate visuals using measures from different fact tables over a shared dimension |

Decision rule:
- One dimension, more granularity on the SAME fact -> **drill-down**.
- Same context, but you need ATOMIC rows of the same fact -> **drill-through**.
- Same dimension context, but metrics live in DIFFERENT fact tables -> **drill-across**.

## Pattern 1: Drill-down (one fact, deeper grain)

Drill-down adds dimension attributes to the row/axis to reveal detail; drill-up removes them. Build the dimension hierarchy in the model (for example a Date hierarchy Year > Quarter > Month > Day, or Product Department > Category > Brand > SKU), then place it on a visual that supports drilling.

Visuals that support built-in drilling (all confirmed Power BI visuals):
- **Line chart** and **column charts** / **bar charts** — put the hierarchy on the X-axis; four drill icons appear: Drill up, Drill down, Go to next level, Expand all.
- **Matrix** — stepped row/column hierarchies with expand/collapse icons on each header; best for tabular drill across many levels.
- **Decomposition tree** — AI visual; user drills into any dimension in any order by selecting the `+` node. Analyze takes a measure; Explain by takes the dimensions.
- **Treemap** — hierarchical part-to-whole; supports the same drill icons.

Example — "How have sales trended, and let users break the trend down by month then day?"
- Visual: **Line chart**, X-axis = Date hierarchy, Y-axis = `Total Sales`.
- Measure:
  ```dax
  Total Sales = SUM ( Sales[SalesAmount] )
  ```
- Enable drill mode; the user clicks a year to drill into its quarters/months/days.

Example — "Compare regions, then drill into stores, in a table layout."
- Visual: **Matrix**, Rows = Region then Store (hierarchy), Values = `Total Sales`, `Margin %`.
- Use expand/collapse icons on row headers; freeze row headers so labels stay visible when scrolling.

CRITICAL — semi-additive facts when drilling on Date. Balances/inventory/headcount must not SUM across time. Drill the hierarchy with an averaging or last-period measure:
```dax
Avg Inventory On Hand = AVERAGE ( Inventory[QuantityOnHand] )

Ending Balance =
CALCULATE (
    SUM ( Inventory[QuantityOnHand] ),
    LASTNONBLANK ( 'Date'[Date], 1 )
)
```
Decomposition tree caps: max 50 levels, 5,000 visible data points, top 10 children per level (lower levels are truncated to top N). Plan drill paths to land on meaningful values within these limits.

## Pattern 2: Drill-through (summary to filtered detail page)

Drill-through lets a user right-click a data point (or press a button) and jump to a **detail page** automatically filtered to the selected value. Use it to expose atomic transaction rows that you deliberately keep off the summary page.

Setup:
1. Create a detail report page.
2. Drag the entity field (for example `Product[ProductName]`) into the page's **Drillthrough filters** well. Add more fields for compound filtering.
3. Add detail visuals to that page (a **Table** of transactions, supporting **Cards**, a **KPI**).
4. The summary visual must contain the same field; the field must NOT be aggregated in the source visual, or the drill-through entry won't appear.
5. Users right-click a data point -> select the detail page, or use a **drillthrough button** (set button Action type to **Drill through**, target = the detail page).

Example — "Click a product in the top-products chart to see its individual transactions."
- Summary: **bar chart**, Axis = `Product[ProductName]`, Value = `Total Sales`.
- Detail page: **Drillthrough filters** = `Product[ProductName]`; page shows a **Table** at transaction grain plus Cards:
  ```dax
  Total Sales = SUM ( Sales[SalesAmount] )
  Transaction Count = COUNTROWS ( Sales )
  ```

Cross-report drill-through (Power BI service only): publish source and target to the same workspace, enable Cross-report drillthrough in the target's settings, and use **identical field names and data types** across reports — this is the dimensional "conformance" rule applied to report design.

Performance: drill-through detail pages are the main place atomic-grain queries hit the model. See [drill-pattern-reference.md](drill-pattern-reference.md) for the optimization checklist (filter before navigating, narrow visuals, cap returned rows, leverage aggregations).

## Pattern 3: Drill-across (multiple facts, conformed dimensions)

Drill-across combines metrics from two or more fact tables that share conformed dimensions. In dimensional terms the BI tool issues one query per fact and FULL OUTER JOINs the results on the conformed attribute; cross-fact ratios are computed AFTER the join. In a Power BI star schema this maps directly: shared dimension tables relate to each fact table, and you place measures from different facts onto the same visual (or coordinated visuals), letting the shared dimension align them.

Requirements (carried straight from conformed-dimension rules):
- The shared dimension must be ONE table related to every fact (identical attribute names/values).
- Never relate facts directly; always go through the conformed dimension.
- Cross-fact ratios are separate measures, never pre-aggregated inside one fact.

Example — "Show sales and average inventory per product, plus sales-per-unit-of-inventory."
- Model: `Product` (conformed) relates to both `Sales` and `Inventory`.
- Visual: **Matrix**, Rows = `Product[ProductName]`, Values = three measures:
  ```dax
  Total Sales = SUM ( Sales[SalesAmount] )

  Avg Inventory = AVERAGE ( Inventory[QuantityOnHand] )   -- semi-additive: AVERAGE, not SUM

  Sales per Unit Inventory = DIVIDE ( [Total Sales], [Avg Inventory] )
  ```
- The conformed `Product` dimension lines up both facts row by row, exactly like a FULL OUTER JOIN on product name.

Example — actual vs. budget at different grains (Sales daily, Budget monthly): create a conformed month-level relationship (a shared Date table at month grain) and compare `[Total Sales]` against `[Budget Amount]` in a **combo chart** (columns = actual, line = budget). Roll the finer-grained fact UP to the coarser grain; do not split the coarse fact down.

For the value-chain pattern (PO -> receipts -> deliveries -> sales across four facts on a conformed Product), coordinated **small multiples** or a single **Matrix** of measures is the cleanest layout. Full taxonomy, the conformance checklist, and DAX appear in [drill-pattern-reference.md](drill-pattern-reference.md).

## Navigation best practices (apply to every drill report)

- **Back button**: on drill-through/detail pages, add a button with Action type **Back** so users return to the summary in one click.
- **Breadcrumbs / view switching**: build a breadcrumb or tab strip with buttons whose Action type is **Page navigation** (no bookmarks to manage) or **Bookmark navigation** (when you also need to restore filter/slicer state). Bookmarks capture slicer values, cross-highlight state, and sort order.
- **Report-page tooltips**: design a small tooltip page and assign it as a visual's tooltip to preview detail on hover before a full drill-through — cheaper than navigating.
- **Conditional button visibility**: show/hide the drill-through button based on whether a valid selection exists (conditional formatting on the button).
- **Verbose labels**: drill on business-friendly attribute values, never surrogate keys or codes.
- **Lock decomposition-tree levels** you don't want consumers to change; consumers can still explore paths within locked levels.

## Workflow checklist

```
Drill Design Progress:
- [ ] Classify the need: down (same fact) / through (atomic detail) / across (multi-fact)
- [ ] Confirm the dimension hierarchy exists in the model (drill-down)
- [ ] Verify conformed dimension relates to all facts (drill-across)
- [ ] Check fact additivity: use AVERAGE/LASTNONBLANK for semi-additive across Date
- [ ] Build the pattern (hierarchy on axis / detail page / shared-dimension measures)
- [ ] Add Back button + breadcrumb/page-navigation buttons
- [ ] Apply the drill-through performance checklist (reference file)
- [ ] Test: drill, then drill back; confirm filters carry context correctly
```

For the full pattern taxonomy, implementation details, conformed-dimension conformance checklist, and the drill-through performance optimization checklist, see [drill-pattern-reference.md](drill-pattern-reference.md).

---
*Provenance: implements Skill 9 (Drill Pattern Design), Layer 3, of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
