---
name: dashboard-composition
description: Assembles multiple Power BI visualizations into cohesive, audience-specific dashboards (report pages) that balance information density, visual hierarchy, and interactivity. Covers executive, operational, and analytical dashboard archetypes, layout and grid rules, color-palette and theme guidance for accessibility and corporate branding, interactivity patterns (slicer placement, cross-filtering, edit interactions, bookmarks, drillthrough), and performance heuristics (keep visuals to only what's necessary, aiming for fewer than ~20 per page). Use when designing a dashboard or report page, choosing a layout, picking a color theme, placing slicers, configuring cross-filtering or bookmarks, deciding what goes on one screen for executives vs operators vs analysts, or optimizing report page performance.
---

# Dashboard Composition

Design Power BI report pages and service dashboards that present multiple visuals as one coherent, audience-tuned screen. This skill is **Layer 3 (Advanced Analytical Patterns)** in the visualization master plan and assumes the individual visuals have already been chosen.

## Prerequisites (sibling skills)

This skill composes the output of earlier skills. Run these first when relevant:

- **mapping-questions-to-charts** — selects the correct visual per question.
- **visualizing-dimensional-models** and **understanding-fact-additivity** — guarantee each visual aggregates correctly (semi-additive balances use AVERAGE/period-end, ratios use weighted DIVIDE).
- Domain skills (**retail-analytics-visualization**, **financial-services-visualization**, **supply-chain-visualization**, **services-analytics-visualization**) supply the KPI set per role.
- **drill-pattern-design** — owns drillthrough/drill-down mechanics referenced here.
- **ai-powered-analysis-selection** — owns Key influencers, Decomposition tree, Smart narrative, Q&A placement on analytical pages.

Composition is the **last** step: pick visuals -> validate aggregation -> then lay them out, theme, wire interactivity, and tune performance with this skill.

## Terminology note

In Power BI, a **dashboard** is a Power BI service artifact built from pinned tiles; a **report page** is a canvas of visuals in Power BI Desktop or the service. The composition principles are shared. This skill says "dashboard" for the audience-facing screen and notes where service-dashboard-only behavior applies.

## Composition workflow

Copy this checklist and track progress:

```
Dashboard Composition Progress:
- [ ] Step 1: Identify audience archetype (executive / operational / analytical)
- [ ] Step 2: Select a template from reference/dashboard-templates.md
- [ ] Step 3: Lay out visuals per reference/layout-composition-rules.md (Z-pattern, grid, keep visuals to only what's needed)
- [ ] Step 4: Apply a theme and palette per reference/color-and-theme-guidance.md
- [ ] Step 5: Wire interactivity per reference/interactivity-patterns.md (slicers, cross-filter, bookmarks)
- [ ] Step 6: Run the accessibility + performance validation loop below
```

### Step 1: Identify the audience archetype

| Archetype | Audience | Purpose | Density | Default page count |
|-----------|----------|---------|---------|--------------------|
| **Executive** | C-suite, VPs | Monitor health at a glance, one screen, no scroll | Low (4-8 visuals) | 1 |
| **Operational** | Managers, frontline | Track current status, alerts, act today | Medium, status-led | 1-2 |
| **Analytical** | Analysts, data teams | Explore, find causes, slice freely | High, drill-heavy | 2-4 |

Pick the archetype from how the audience *uses* the screen (monitor vs act vs explore), not their job title alone. Full templates with concrete visual lists and DAX are in **[reference/dashboard-templates.md](reference/dashboard-templates.md)** (10+ examples across all three archetypes).

### Step 2-3: Layout

Apply the core layout rules (full detail in **[reference/layout-composition-rules.md](reference/layout-composition-rules.md)**):

- **One screen, no scroll** for executive/operational dashboards. Tell one story; remove everything non-essential.
- **Z / top-left priority**: highest-level data top-left; detail flows left-to-right, top-to-bottom (Western reading order). Place KPI **Card** / **KPI** / **Multi-row card** visuals across the top row.
- **Grid and alignment**: use gridlines + snap-to-grid; keep consistent gutters; align edges. Keep slicers in the same on-page position across all pages.
- **Right visual for the data**: prefer **Bar**/**Column** for comparison over **Pie**/**Donut**/**Gauge**; reserve **Pie**/**Donut** for part-to-whole with fewer than 8 categories; **Gauge** for current value vs a goal. Avoid 3D and decorative variety.
- **Number formatting**: scale to "3.4 million", not "3,400,000"; keep 1-2 digits left of the decimal; don't mix precision/time frames across visuals on one page.

### Step 4: Color and theme

Apply a **report theme** (View > Themes) so every visual inherits one palette; for corporate branding import a custom theme JSON. Use the built-in **Color-blind safe** or **Accessible** themes, or verify a 4.5:1 text/background contrast ratio. Reserve a single accent color to highlight one data point; encode status with conditional formatting (Rules / Gradient / Field value) on **Table**/**Matrix** plus an icon or text so color is never the *only* signal. Full guidance, the unsafe color-pair list, and the semantic status palette are in **[reference/color-and-theme-guidance.md](reference/color-and-theme-guidance.md)**.

### Step 5: Interactivity

Decide deliberately how visuals interact (full patterns in **[reference/interactivity-patterns.md](reference/interactivity-patterns.md)**):

- **Slicers**: place a vertical-list **Slicer** column on the left, or a horizontal **Tile** style slicer (with Responsive layout) along the top. Use **Sync slicers** to share selections across pages and keep slicers in the same position on every page.
- **Cross-filtering / cross-highlighting**: by default a click filters/highlights peer visuals. Use **Edit interactions** to set each target visual to Filter, Highlight, or None so a context visual stays stable.
- **Bookmarks**: capture views/states; group them and drive them from a **Button** bookmark navigator for view switching (e.g., Revenue vs Units toggle). Turn off the bookmark's **Data** property when switching views so slicers aren't reset.
- **Drillthrough & tooltips**: keep detail off the main canvas; push it to drillthrough pages and report-page tooltips (this also helps performance). Coordinate with **drill-pattern-design**.

### Step 6: Validation loop (accessibility + performance)

Run this loop; fix and repeat until all pass:

1. **Visual count**: keep visuals to only what the page needs (Microsoft guidance names no number; aim for fewer than ~20 as a heuristic ceiling). If crowded, move detail to drillthrough pages or tooltips.
2. **Most-restrictive filters**: every table/visual has the minimum data it needs; apply a **Top N** visual filter to large **Table** visuals instead of leaving them unfiltered.
3. **Contrast**: title, axis-label, and data-label text vs background is at least 4.5:1.
4. **Color independence**: no information conveyed by color alone (add markers/icons/text).
5. **Alt text** set on every non-decorative visual; decorative shapes hidden from **Tab order** via the Selection pane.
6. **Tab order** set to match reading order.
7. **Custom visuals** profiled — poorly optimized custom visuals slow the whole page.

The performance checklist and the service-dashboard tile-cache behavior are detailed in **[reference/layout-composition-rules.md](reference/layout-composition-rules.md)**.

## Quick example: executive sales dashboard

Question set: "Are we hitting revenue target? How is the trend? Where (region) and what (top products) drive it?"

- Top row: four **Card** visuals — Revenue, Gross Margin %, Units, Transactions.
- Middle left: **Line** chart, revenue by month (additive measure, safe to SUM).
- Middle right: **Filled map**, revenue by region.
- Bottom left: **Clustered bar** (horizontal), top 10 products.
- Bottom right: **Donut**, revenue by category (only because categories < 8).
- Left rail: synced **Slicer** visuals for Date, Region, Category.

DAX for the top-row cards (additive — SUM is safe; margin is a weighted ratio, not an average of ratios):

```dax
Total Revenue = SUM ( Sales[Revenue] )

Gross Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )

Revenue vs Target % =
DIVIDE ( [Total Revenue], SUM ( Target[Revenue_Target] ), 0 )
```

More worked dashboards (operational store/inventory/claims, analytical churn/profit-driver) with their DAX are in **[reference/dashboard-templates.md](reference/dashboard-templates.md)**.

## Confirmed Power BI visuals used by this skill

Only these (all confirmed in the visualization-types/customization source) appear in templates: Area, Bar, Card, Multi-row card, Column, Combo, Donut, Filled map, Funnel, Gauge, Key influencers, KPI, Line, Map, Azure Map, Matrix, Pie, Q&A, Scatter, Shape, Slicer, Table, Textbox (for Smart narrative), Treemap, Waterfall, and Decomposition tree. Do not introduce other chart types.

---
*Provenance: implements SKILL 10 (Dashboard Composition), Layer 3, of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
