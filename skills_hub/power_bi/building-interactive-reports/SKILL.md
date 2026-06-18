---
name: building-interactive-reports
description: Assembles interactive Power BI report pages using filters and slicers, visual interactions, bookmarks, buttons and navigators, drillthrough, report tooltips, conditional table formatting, report themes, and mobile-optimized layouts. Use when building or refining report pages in Power BI Desktop or the service Editing view — when choosing filter scope (visual/page/report/drillthrough), setting how visuals cross-filter or cross-highlight each other, capturing report state in bookmarks, adding navigation buttons or page/bookmark navigators, creating drillthrough or tooltip pages, applying conditional formatting to tables and matrices, applying a theme, or creating a portrait mobile layout.
---

# Building Interactive Reports in Power BI

## Overview

A report page becomes *interactive* through layered features: filters narrow the data, visual interactions control how selections propagate, and navigation features (bookmarks, buttons, navigators, drillthrough) let users move between focused views. This skill maps each interactivity feature to the use case it solves, gives navigation recipes, and checklists a mobile layout. Authoring requires **Edit permission** to the report (Power BI Desktop, or the Power BI service **Editing view**); readers interact in **Reading view** but can't add features.

**Prerequisites (from the full-lifecycle plan):** depends on the visualization skills 1-10 (especially `drill-pattern-design` for drill behavior) and `authoring-dax-measures` (measures feed conditional formatting and tooltips). Complements `dashboard-composition`. After the report is built, hand off to `publishing-and-sharing-content`.

## When to Use

Use this skill when the request involves any of: filters or slicers, "how visuals filter each other", edit interactions, bookmarks, buttons, page/bookmark navigators, drillthrough pages, report-page tooltips, conditional formatting of a table/matrix, report themes, or a mobile/phone layout. For *which chart to pick*, use `mapping-questions-to-charts`; for *drill up/down within one visual*, use `drill-pattern-design`.

## Build workflow

Copy this checklist and work top-down:

```
Interactive report build:
- [ ] Step 1 — Pick filter scope per field (visual / page / report / drillthrough)
- [ ] Step 2 — Set visual interactions (filter / highlight / none) per pair
- [ ] Step 3 — Add navigation (buttons, page/bookmark navigators, drillthrough)
- [ ] Step 4 — Capture states with bookmarks (views, spotlight, slide show)
- [ ] Step 5 — Add detail layers (report tooltips, conditional formatting)
- [ ] Step 6 — Apply a report theme for consistency
- [ ] Step 7 — Build the mobile (portrait) layout
- [ ] Step 8 — Test in Reading view + mobile app
```

For the full feature → use-case decision table (which to reach for and what it can/can't do), see [interactivity-feature-map.md](interactivity-feature-map.md).

## Step 1 — Filters and slicers (scope)

Filters live in the **Filters pane** (right side of the canvas; expand with the `>>` arrows). Drag a field into one of three buckets to set its **scope**:

- **Filters on this visual** — one visual only. Fields already in a visual become automatic filters; you can also add a field that isn't shown, including a *numeric* column with a condition (e.g. `TotalUnits is greater than 10,000`).
- **Filters on this page** — every visual on the page.
- **Filters on all pages** — every visual in the report (report-level).
- **Drillthrough** — a fourth, navigation-driven scope; see Step 3.

Readers can switch **Basic** (pick from the value list, with search) and **Advanced** (contains / starts with, AND/OR conditions) filtering unless you turn switching off. Advanced filters support up to 100 distinct conditions (temporary 24-condition limit in Desktop); for more, build a dedicated table in the model instead.

Slicers are on-canvas filters readers manipulate directly — use them instead of (or with) the Filters pane when the control should be visible. Filter selections **persist** across sessions in the service; readers use **Reset to default** (top menu, Reading view) to return to the author's saved default state.

Filter *types* differ in what an author can edit/clear/delete/hide/lock/sort/rename: **Automatic** and **Manual** are the common, fully-editable ones; **Include/Exclude**, **Drill-down**, **Cross-drill**, **Drillthrough**, **URL**, and **Pass-through (Q&A)** are created by other features and are more restricted. See [interactivity-feature-map.md](interactivity-feature-map.md) for the full capability matrix.

## Step 2 — Visual interactions (filter / highlight / none)

By default, selecting a data point in one visual **cross-filters** and **cross-highlights** the others. The two behaviors differ:

- **Cross-highlighting** dims unrelated data but keeps it visible (column/bar charts). Removed by re-selecting the point or clicking empty space in that visual.
- **Cross-filtering** removes unrelated data, like a real filter (line charts, scatter charts, maps — these can only be cross-filtered, **not** cross-highlighted).

To change behavior, enable the controls, then set each target per source visual:

- **Desktop:** select a source visual, then **Format** ribbon > **Edit interactions**.
- **Service:** Editing view > report menu bar > **Visual interactions** > turn on **Edit interactions**.

Power BI adds **Filter**, **Highlight**, and **None** icons to every *other* visual; the bold icon is active. Pick:
- **Filter** — source cross-filters that target,
- **Highlight** — source cross-highlights that target,
- **None** — source has no effect on that target.

Changes save with the report (authors and readers get the same experience). By default **drilling** one visual does *not* affect others; to change that, turn on **Apply drill down filters to > Entire page** (Desktop Format menu) or **Drilling filters other visuals > On** (service Visual interactions dropdown).

## Step 3 — Navigation: buttons, navigators, drillthrough

**Buttons** (Insert ribbon > **Buttons**, or **Buttons** in the service menu bar) make a report act like an app. Set **Action** to **On** in the Format button pane, then choose an action **Type**:

- **Back** — return to the previous page (auto-added on drillthrough pages).
- **Bookmark** — go to a saved bookmark state.
- **Drill through** — go to a drillthrough page filtered to the selection (no bookmark).
- **Page navigation** — go to another page (no bookmark); also works on shapes/images.
- **Bookmark navigation** — go to a different state via a bookmark.
- **Q&A**, **Web URL**, **Apply all slicers**, **Clear all slicers**.

Buttons have states (**Default**, **On hover**, **On press**, **Disabled**, **Loading**) you can format independently. Test a button in edit mode with **Ctrl+select**.

**Navigators** (Insert > **Buttons** > **Navigator** > **Page navigator** or **Bookmark navigator**) auto-build and auto-sync a whole navigation bar — labels, order, and current selection track your pages or bookmarks automatically. Use these instead of hand-built buttons for a full nav experience.

**Drillthrough** sends a user from a summary point to a detail page filtered to their selection. Create a target page, drag the entity field (e.g. `Customer Name`) into the **Drillthrough filters** well, then add visuals that respond. Users trigger it by **right-clicking** a data point or via a drillthrough button. A **Back** button is added automatically.

For step-by-step navigation recipes (table-of-contents page, toggle/deselect, conditional destination, drillthrough page), see [navigation-pattern-recipes.md](navigation-pattern-recipes.md).

## Step 4 — Bookmarks (capture state)

A **report bookmark** saves the current state of a page: current page, filters, slicers (type + state), visual selection/cross-highlight, sort order, drill location, object visibility (from the **Selection** pane), and spotlight/focus. Enable the pane via **View > Bookmarks**, configure the page, then **Add**.

Key controls (More options ... next to a bookmark): **Update**, **Rename**, **Delete**, and toggles for **Data** (filters/slicers), **Display** (visibility/spotlight), **Current page**, and **All visuals** vs **Selected visuals**.

> When a bookmark only switches *views* (e.g. show/hide visuals), turn **off** its **Data** property so it doesn't reset users' filters.

Group bookmarks (**Ctrl**-select > **Group**) to scope a bookmark navigator. Use **View** for a slide-show walkthrough. Use the **Selection** pane to toggle object visibility — slicers keep filtering even when hidden, which is what makes layered bookmark views work. Bookmarks save *state*, not *position* (moving a visual later won't be restored).

## Step 5 — Detail layers: tooltips and conditional formatting

**Report tooltips (tooltip pages)** show a rich custom hover instead of the default value tooltip. Create a page, set **Canvas Settings > Type = Tooltip** for the small ready-made size, set **Page Information > Tooltip = On**, then drag the trigger field(s) into the **Tooltip fields** bucket. Any visual containing that field then shows your page on hover. Override per visual via the visual's **Tooltips** card > **Page** dropdown (set to **Auto** to revert to default). Always **hide** tooltip pages; keep them small. Page tooltips are **not interactive** — if users must click, use a drillthrough page instead. For a quick (non-page) tooltip, drag fields into the visual's **Tooltips** bucket.

**Conditional formatting** applies to *values* in **Table** and **Matrix** visuals (not headers; for matrices, it targets the lowest visible level). Right-click / down-arrow a field in the Visualizations pane > **Conditional formatting**, then: **Background color** / **Font color** (by **Gradient** color scale, by **Rules**, or by **Field value**), **Data bars**, **Icons**, or **Web URL**. For percentage fields, enter rule values as decimals and choose **Number** format, not **Percent**. Use `DIVIDE` to avoid NaN, which breaks auto min/max and percentage rules. To color *text* states, build a color measure and format by field value:

```dax
StatusColor = SWITCH('Table'[Status], "Accepted", "blue", "Declined", "red", "None", "grey")
```

## Step 6 — Report themes

Themes set report-wide defaults (colors, fonts, visual styling). **View** ribbon > **Themes** dropdown > pick a built-in theme (including **Accessible** themes and **Color-blind safe**), **Customize current theme** (Name and colors, Text, Visuals, Page, Filter pane), or **Browse for themes** to import a JSON file. Save with **Save current theme**.

> Themes apply in **Power BI Desktop only** — you can't apply a theme to a report already in the service.

A minimal theme JSON requires only `name`; everything else is optional and unspecified settings keep Power BI defaults:

```json
{
    "name": "Corporate",
    "dataColors": ["#118DFF", "#12239E", "#E66C37", "#6B007B"],
    "background": "#FFFFFF",
    "foreground": "#252423",
    "tableAccent": "#118DFF"
}
```

Theme colors travel with the report when published. Explicitly set per-data-point colors are **not** overridden by a new theme (use **Reset to default** / **Revert to default** to re-enable theming).

## Step 7 — Mobile (portrait) layout

Build a separate portrait layout in **mobile layout view**: **View** ribbon > **Mobile layout** (Desktop), or Editing view > **Mobile layout** / layout switcher (service). When a mobile layout exists, the mobile app shows it automatically in portrait; tipping the phone landscape shows the regular page. **Auto-create mobile layout** generates a starting layout (reads left-to-right from the top; handles visible and hidden visuals). Drag visuals from the **Page visuals** pane onto the gridded canvas; format via the Visualizations pane; reorder layers via the Selection pane. Publishing the main report (or saving in the service) publishes the mobile layout too. Note: mobile-optimized views appear in the **iOS/Android apps only**, not in a web browser.

For the full mobile checklist, see [mobile-layout-checklist.md](mobile-layout-checklist.md).

## Step 8 — Test

- **Ctrl+select** every button/navigator in edit mode; verify destinations and back behavior.
- Switch to **Reading view** and confirm filters, slicers, bookmarks, drillthrough (right-click), and tooltips behave for a reader.
- Drillthrough source fields must **not** be aggregated and must match the target's drillthrough filter field; use **Performance analyzer** to debug missing menus or filters.
- Open the report in the **mobile app** (portrait) to verify the mobile layout.

## Common pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Wrong filter scope | A "page" filter set as visual-level only narrows one chart | Drag the field into **Filters on this page** / **Filters on all pages** |
| View bookmark resets filters | Switching views via a bookmark clears slicers | Turn **off** the bookmark's **Data** property |
| Slicer added after bookmarking | New slicer is cleared / bookmarks behave oddly | Add slicers first; **Update** affected bookmarks |
| Visibility revert with Selection pane | Bookmark loses a show/hide change | After changing visibility, right-click the bookmark > **Update** |
| Tooltip page used for interaction | Clicks do nothing in the tooltip | Page tooltips aren't interactive — use a **drillthrough page** |
| Tooltip page not hidden | Users navigate onto the raw tooltip page | **Hide** tooltip pages; keep them small |
| Drillthrough item missing from menu | Right-click shows no drillthrough page | Ensure the field is in the **Drillthrough filters** well, matches the source field, and isn't aggregated |
| Percentage conditional rules wrong | Colors off for `%` fields | Enter decimals and select **Number** (not **Percent**); use `DIVIDE` to avoid NaN |
| Trying to theme a published report | Theme dropdown unavailable in the service | Apply themes in **Power BI Desktop**, then republish |
| Cross-filter expected on a line/scatter/map | "Highlight" produced no dimming | Those visuals cross-**filter** only; set the interaction to **Filter** |
| Mobile layout invisible in browser | Phone view never appears | Mobile-optimized views show in the **mobile apps** only |

## Success criteria

- Each field filters at the intended scope (visual/page/report/drillthrough).
- Visual interactions match intent (filter vs highlight vs none) for each pair.
- Navigation (buttons/navigators/drillthrough) works via Ctrl+select and in Reading view; back behavior is correct.
- Bookmarks restore the intended state without unintentionally resetting filters.
- Tooltip pages are hidden, small, and non-interactive; conditional formatting reads correctly (percentages as Number, no NaN).
- A theme is applied in Desktop and a portrait mobile layout exists and verifies in the mobile app.

---
*Provenance: implements Skill 16 "Building Interactive Reports" (Phase 6, Report & Distribute) of `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI `create-reports` docs (filters, visual-interactions, bookmarks, buttons, button-navigators, drillthrough, tooltips, custom-tooltips, conditional-table-formatting, report-themes, mobile-optimized layout) and the `guidance` docs report-drillthrough and report-page-tooltips.*
