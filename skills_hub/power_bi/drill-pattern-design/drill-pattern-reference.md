# Drill Pattern Reference: Taxonomy, Implementations, and Performance

## Contents
- Drill-pattern taxonomy (definitions and when each applies)
- Drill-down implementation details (visuals, hierarchies, drill icons, semi-additive caution)
- Drill-through implementation details (detail pages, buttons, cross-report)
- Drill-across implementation details (conformed dimensions in a Power BI star schema)
- Conformed-dimension conformance checklist
- DAX measure library for drill scenarios
- Navigation building blocks (button actions, bookmarks, tooltips)
- Drill-through performance optimization checklist
- Anti-patterns

---

## Drill-pattern taxonomy

| Pattern | Dimensional meaning | Stays on one fact? | Power BI realization |
|---------|---------------------|--------------------|----------------------|
| **Drill-down / drill-up** | Add/remove dimension attributes on row headers or axis to change grain shown | Yes | Hierarchy on chart axis, matrix rows/columns, decomposition tree, treemap |
| **Drill-through** | Navigate from an aggregated value to the atomic transaction rows that compose it | Yes (summary and detail are the same fact, different grain) | A detail report page filtered to the selected value |
| **Drill-across** | Combine metrics from DIFFERENT business-process fact tables that share conformed dimensions | No (two or more facts) | Shared dimension tables relating each fact; measures from different facts on the same/coordinated visuals |

Selection logic:
1. Need finer grain of the **same** measure? -> drill-down.
2. Need the **individual rows** behind a summary number? -> drill-through.
3. Need to put **two metrics from different processes** side by side over a common dimension? -> drill-across.

Drilling down (in Kimball terms) means adding dimension attributes — and crucially this works for attributes that are NOT part of a formal hierarchy (e.g., Package Type, Fat Content), not just declared hierarchy levels. Expose all useful dimension attributes for drilling, not only hierarchy rungs.

---

## Drill-down implementation details

### Build the hierarchy
Create the hierarchy in the model (e.g., Date: Year > Quarter > Month > Day; Geography: Country > State > City; Product: Department > Category > Brand > SKU). Place the hierarchy field on the visual's axis (charts) or Rows/Columns (matrix).

### Drill controls
On a chart with a hierarchy on the axis, four drill icons appear:
- **Drill up** — move to a higher level.
- **Drill down** — enable drill mode, then select a data point/axis value to descend into it.
- **Go to next level** — expand the whole next level for every category.
- **Expand all** — show all hierarchy levels at once.

The **Matrix** uses per-header expand/collapse icons instead of the four-icon bar, supporting independent row and column hierarchies simultaneously. Freeze row headers (default) so labels persist during horizontal scroll.

### Visuals confirmed to support drilling
Line chart, column charts, bar charts, treemap (axis/hierarchy drill icons); matrix (expand/collapse); decomposition tree (free-order node drilling). Pie charts and donut charts represent part-to-whole at a single level and are not the primary drill surface — prefer treemap or matrix when hierarchical drilling is required.

### Decomposition tree specifics
- **Analyze** well: one measure/aggregate. **Explain by** well: one or more dimensions.
- The user selects `+` on a node and chooses any dimension, in any order.
- **AI Splits** (light-bulb option) auto-suggest the dimension that yields the High Value or Low Value of the measure; switch Absolute vs. Relative to find standout values. AI levels can be chained, but a manual (non-AI) level cannot follow an AI level.
- **Locking**: a creator can lock leading levels so consumers can explore but not remove/change them; you cannot have unlocked levels preceding locked ones.
- **Limits**: max 50 levels; max 5,000 data points shown at once; top N per level currently 10 (lower levels truncated). AI splits are not supported on Azure Analysis Services, Power BI Report Server, Publish to Web, or complex/extension-schema measures.

### Semi-additive caution when drilling on Date
Balances, inventory on hand, headcount, and account balances are semi-additive: summable across product/location but NOT across time. A naive `SUM` overcounts as you drill into finer date grains. Use `AVERAGE` for period-average or `LASTNONBLANK`-style logic for period-end. See the DAX library below.

---

## Drill-through implementation details

### Components
| Component | Role |
|-----------|------|
| Drillthrough page | A page designed to detail a single item; visuals respond to the drillthrough filter |
| Drillthrough filters well | Holds the field(s) that filter the page based on the user's selection; multiple fields = compound filter |
| Drillthrough button | A button with Action type **Drill through** targeting the detail page; supports tooltips and conditional show/hide |
| Cross-report drillthrough | Navigate to a detail page in another report in the same workspace (service only) |

### Steps
1. Add a detail page; drag the entity field (e.g., `Product[ProductName]`) into **Drillthrough filters**.
2. Add detail visuals — typically a **Table** at transaction grain, plus **Cards** / **KPI** for context.
3. Ensure the source visual contains the same field and that the field is **not aggregated** there (otherwise the drill-through option will not show).
4. User right-clicks a data point -> picks the detail page; or uses a drillthrough button.
5. Add a **Back** button on the detail page.

### Cross-report drillthrough rules (service only)
- Source and target published to the **same workspace**.
- Enable **Cross-report drillthrough** in the target report settings.
- Field **names and data types must match exactly** between source and target (the report-level analog of conformed dimensions).
- Refresh both reports after publishing.

### Troubleshooting quick map
- Detail page missing from right-click menu -> filter field absent from Drillthrough filters well, or field is aggregated in the source visual.
- Filters not applied -> field duplicated/transformed in Power Query, or relationships block filter propagation; use Performance analyzer.

---

## Drill-across implementation details

Drill-across is the payoff of conformed dimensions. The classic dimensional procedure: query each fact separately with identical dimension constraints, group by the same conformed attribute, FULL OUTER JOIN the results on that attribute, and compute cross-fact ratios afterward.

In a Power BI star schema this becomes structural rather than procedural:
1. Model each business process as its own fact table (Sales, Inventory, Budget, ...).
2. Relate every fact to the SAME conformed dimension table(s) (one `Product`, one `Date`, one `Store`).
3. Never relate fact tables to each other; the shared dimension does the alignment.
4. Author measures per fact (`Total Sales` from Sales; `Avg Inventory` from Inventory).
5. Drop measures from different facts onto the same **Matrix** / **combo chart** / coordinated **small multiples**; the conformed dimension aligns rows like the outer join would.
6. Author cross-fact ratios as their own measures referencing the per-fact measures (e.g., `Sales per Unit Inventory = DIVIDE([Total Sales],[Avg Inventory])`). Never pre-aggregate a ratio inside one fact — sum of ratios is not the ratio of sums.

### Different grains
If facts are at different grains (Sales daily vs. Budget monthly), roll the finer fact UP to the coarser grain via a shared coarser dimension (a month-grain relationship to `Date`). Prefer rolling up over splitting the coarse fact down (splitting introduces allocation error).

### Semi-additive facts in drill-across
When one drill-across metric is a balance (inventory), aggregate it with `AVERAGE` across time while the additive metric uses `SUM`. Cross-fact metric (e.g., inventory turns = sales qty / avg inventory) is computed after both are resolved.

---

## Conformed-dimension conformance checklist

Before enabling any drill-across (or cross-report drill-through):
```
- [ ] Shared dimension is a single table related to every participating fact
- [ ] Attribute names identical across facts (product_name, not product_desc)
- [ ] Attribute values identical ("Brand A", not "BRAND A" / "Brand_A")
- [ ] Data types identical
- [ ] Identical dimension constraints/filters applied to every fact
- [ ] Cross-fact ratios authored as separate measures (post-join), not pre-aggregated
- [ ] Facts NOT related directly to each other (only via conformed dimensions)
- [ ] Tested an entity that exists in one fact but not the other (no silent row loss)
```

---

## DAX measure library for drill scenarios

```dax
-- Additive base measures (safe to SUM across all dimensions; safe at any drill level)
Total Sales      = SUM ( Sales[SalesAmount] )
Total Quantity   = SUM ( Sales[Quantity] )
Transaction Count = COUNTROWS ( Sales )

-- Semi-additive (inventory / balances): AVERAGE across time
Avg Inventory On Hand = AVERAGE ( Inventory[QuantityOnHand] )

-- Semi-additive period-end balance
Ending Balance =
CALCULATE (
    SUM ( Inventory[QuantityOnHand] ),
    LASTNONBLANK ( 'Date'[Date], 1 )
)

-- Non-additive ratio: compute from SUMs, never average a ratio
Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )

-- Cross-fact (drill-across) ratio: built from per-fact measures
Sales per Unit Inventory = DIVIDE ( [Total Sales], [Avg Inventory On Hand] )

-- Budget vs actual (drill-across over conformed Date at month grain)
Budget Amount = SUM ( Budget[BudgetAmount] )
Variance      = [Total Sales] - [Budget Amount]
Variance %    = DIVIDE ( [Variance], [Budget Amount] )
```

---

## Navigation building blocks

### Button action types (confirmed)
- **Back** — return to the previous page; ideal on drill-through/detail pages.
- **Bookmark** — go to a saved report state (filters, slicers, cross-highlight, sort).
- **Drill through** — navigate to a drillthrough page filtered to selection (no bookmark needed).
- **Page navigation** — jump to another page (no bookmark); use for breadcrumbs/tabs.
- **Bookmark navigation** — switch to a different report state, same or different page.
- **Web URL** — open an external page.

### Bookmarks
Capture filters, slicers (including slicer type/state), cross-highlight selection, and sort order. Use to build view switching and to restore exploration state. Note: bookmarks do not restore visual positions, only state.

### Report-page tooltips
Build a dedicated tooltip page and assign it as a visual's tooltip to preview a record's detail on hover — a low-cost alternative to a full drill-through navigation for quick context.

### Breadcrumbs
Implement with a row of **Page navigation** buttons (lightweight) or a bookmark navigator (when restoring state matters). Always pair drill-through detail pages with a **Back** button.

---

## Drill-through performance optimization checklist

Detail/drill-through pages are where atomic-grain queries are most expensive. Optimize:
```
- [ ] Filter BEFORE navigating: drillthrough filters constrain the detail query to one entity,
      so the detail page never scans the whole fact table
- [ ] Cap returned rows: detail Table visual should target a single entity's transactions,
      not an unbounded set; add Top N or date constraints where natural
- [ ] Keep the detail page lean: fewer visuals = fewer concurrent queries (favor one Table + a few Cards)
- [ ] Leverage aggregations: summary visuals read aggregated/summary measures; only the
      drill-through page touches atomic grain
- [ ] Match grain on roll-up for drill-across (roll fine fact up; avoid down-allocation)
- [ ] Avoid duplicated/transformed drill fields in Power Query (breaks filter propagation and adds cost)
- [ ] Use Performance analyzer to inspect the detail page's query and filter application
- [ ] For cross-report drillthrough, confirm matching field names/types so the filter binds efficiently
```

Rationale: the drillthrough filter applies the selected value automatically, so a well-designed detail page issues a tightly filtered query (one product, one customer, one region) rather than an open scan — this single design choice is the largest driver of drill-through responsiveness.

---

## Anti-patterns

- **Dead-end summaries**: providing only aggregated visuals with no drill-through to atomic detail.
- **Summing semi-additive facts across time** while drilling on a Date hierarchy (inventory/balances overcount). Use AVERAGE or period-end logic.
- **Drilling/joining on surrogate keys** instead of business attributes for drill-across; surrogate keys differ across facts and never match.
- **Relating fact tables directly** instead of through a conformed dimension.
- **Pre-aggregating ratios** inside one fact before a drill-across join (sum of ratios is not the ratio of sums).
- **Exposing codes/surrogate keys** as drill labels instead of verbose business attribute values.
- **Unfiltered detail pages** that scan the whole fact table on every drill-through (slow, no benefit).
- **Mismatched cross-report fields**: differing names/types break cross-report drillthrough silently.
