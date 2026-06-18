# Interactivity Feature → Use-Case Map

The decision reference for `building-interactive-reports`: which Power BI interactivity feature to reach for, its scope/behavior, and what authors can do with it. All features require **Edit permission** (Power BI Desktop or service Editing view).

## Contents
- Feature → use-case quick map
- Filter scope (visual / page / report / drillthrough)
- Basic vs Advanced filtering
- Filter types capability matrix
- Visual interactions: filter vs highlight vs none
- Bookmarks: what is captured and which property to toggle
- Buttons: action types
- Navigators: page vs bookmark
- Tooltips: quick vs report-page
- Conditional formatting options (table/matrix)

## Feature → use-case quick map

| You want to… | Use | Where |
|---|---|---|
| Narrow one chart only | **Visual-level filter** | Filters pane > Filters on this visual |
| Narrow every chart on a page | **Page-level filter** or a **slicer** | Filters pane > Filters on this page / on-canvas slicer |
| Narrow every chart in the report | **Report-level filter** | Filters pane > Filters on all pages |
| Let a reader pick values on the canvas | **Slicer** | Visualizations pane |
| Control how a selection spreads between visuals | **Visual interactions** (filter / highlight / none) | Format > Edit interactions |
| Jump to a detail page filtered to a selection | **Drillthrough** | Drillthrough filters well + right-click / button |
| Move between report pages | **Page navigation** button or **Page navigator** | Insert > Buttons |
| Save / restore a page state, build a slide show, spotlight | **Bookmark** | View > Bookmarks |
| Build a synced nav bar from pages or bookmarks | **Navigator** | Insert > Buttons > Navigator |
| Show rich detail on hover | **Report tooltip page** | Page Information > Tooltip = On |
| Add a few extra values on hover | **Quick (field) tooltip** | Tooltips bucket |
| Highlight values/outliers in a table or matrix | **Conditional formatting** | Field dropdown > Conditional formatting |
| Apply report-wide colors/fonts/styles | **Report theme** | View > Themes (Desktop only) |
| Make a page readable on a phone | **Mobile layout** | View > Mobile layout |

## Filter scope (visual / page / report / drillthrough)

The Filters pane has three drop buckets plus the navigation-driven drillthrough scope:

| Scope | Bucket label | Applies to | Notes |
|---|---|---|---|
| **Visual** | Filters on this visual | One selected visual | Fields in the visual are automatic filters; you can add fields not shown, incl. a numeric column with a condition (e.g. `is greater than 10,000`) applied to underlying rows |
| **Page** | Filters on this page | All visuals on the active page | Redraws the whole page |
| **Report** | Filters on all pages | Every visual on every page | Report-level |
| **Drillthrough** | Drillthrough well (Visualizations pane) | The target page when navigated to | Set by selecting a data point on a source page; see navigation recipes |

Filter selections **persist** between sessions in the service. Author-saved filters become the **default state**; readers use **Reset to default** (Reading view, top menu) to return to it.

## Basic vs Advanced filtering

| Mode | What it does |
|---|---|
| **Basic** | Lists all values in the field; select/clear; searchable (search can be turned off) |
| **Advanced** | Conditions like contains / doesn't contain / starts with; combine with AND/OR |

Advanced filters support up to **100 distinct conditions** (temporary **24**-condition limit in Power BI Desktop). For more, create a dedicated table in the model. Authors can disable Basic↔Advanced switching and disable search.

## Filter types capability matrix

How each filter *type* responds to author actions (Y = supported). From the filter-types doc:

| Filter type | Edit | Clear | Delete | Hide | Lock | Sort | Rename |
|---|---|---|---|---|---|---|---|
| Automatic (built from a visual's fields) | Y | Y | N | Y | Y | Y | Y |
| Manual (dragged into the pane) | Y | Y | Y | Y | Y | Y | Y |
| Include / Exclude | N | N | Y | Y | Y | Y | N |
| Drill-down | Y | Y | N | N | N | N | N |
| Cross-drill | N | N | N | N | N | N | N |
| Drillthrough (invokes drillthrough) | Y | Y | Y | Y | Y | N | N |
| Drillthrough (transient) | Y | Y | Y | N | N | N | N |
| URL (transient) | Y | Y | Y | N | N | N | N |
| Pass-through (from Q&A) | N | N | Y | Y | N | Y | N |

Automatic and Manual are the everyday, fully-editable filters. The rest are created by other features (include/exclude, drilling, drillthrough, URL parameters, Q&A) and are intentionally restricted.

## Visual interactions: filter vs highlight vs none

Default behavior: a selection in one visual cross-filters **and** cross-highlights the others. Override per source→target pair after **Edit interactions** is on.

| Setting | Effect on the target | Best for |
|---|---|---|
| **Filter** | Removes unrelated data (acts like a filter) | line charts, scatter charts, maps (these **only** cross-filter) |
| **Highlight** | Dims unrelated data but keeps it visible | column / bar charts |
| **None** | No effect | independent visuals that shouldn't react |

Drilling, by default, does **not** affect other visuals; enable **Apply drill down filters to > Entire page** (Desktop) or **Drilling filters other visuals > On** (service) to change that. Interaction settings are saved with the report, so authors and readers see the same behavior.

## Bookmarks: what is captured and which property to toggle

A **report bookmark** captures: current page, filters, slicers (type + state), visual selection / cross-highlight state, sort order, drill location, object visibility (Selection pane), and spotlight/focus.

Per-bookmark properties (More options ... > toggle):

| Property | Controls | Turn off when… |
|---|---|---|
| **Data** | filters and slicers | the bookmark only switches views — so it won't reset reader filters |
| **Display** | visibility and spotlight | you only want a data state, not a visibility change |
| **Current page** | shows the page that was active | the bookmark should stay on the current page |
| **All visuals / Selected visuals** | scope of the bookmark | you want it to affect only selected visuals |

Notes: slicers keep filtering even when hidden (Selection pane), which enables layered views. Bookmarks save **state, not position** — moving a visual later isn't restored. Adding a slicer after creating bookmarks clears it; **Update** affected bookmarks. Group bookmarks to scope a bookmark navigator and the **View** slide show.

## Buttons: action types

Set **Action = On** in the Format button pane, then pick **Type**:

| Action type | Behavior |
|---|---|
| **Back** | Return to the previous page (auto-added on drillthrough pages) |
| **Bookmark** | Go to a bookmark's saved state |
| **Drill through** | Go to a drillthrough page filtered to the selection (no bookmark) |
| **Page navigation** | Go to another page (no bookmark); also works on shapes/images |
| **Bookmark navigation** | Go to a different state, same or other page, via a bookmark |
| **Q&A** | Open the Q&A Explorer for natural-language questions |
| **Web URL** | Open a web page |
| **Apply all slicers** / **Clear all slicers** | Apply or clear all slicers on the page |

Button states you can format independently: **Default**, **On hover**, **On press**, **Disabled**, **Loading**. An action field can't be a numeric measure. Test in edit mode with **Ctrl+select**.

## Navigators: page vs bookmark

| Navigator | Auto-syncs | Extra options |
|---|---|---|
| **Page navigator** | Labels = page names, order = page order, current page selected; updates as pages are added/renamed | Show/hide hidden pages and tooltip pages |
| **Bookmark navigator** | Labels = bookmark names, order = bookmark order; updates as bookmarks change | Scope to a bookmark group; **Allow deselection** + **Launch on deselection** + **Hide deselection bookmark** for toggle/empty states |

Both support Grid layout (Horizontal / Vertical / Grid + Padding) and a formattable **Selected state**. Create bookmarks (and groups) **before** the bookmark navigator. Limitation: the bookmark navigator selected state isn't reflected in report exports.

## Tooltips: quick vs report-page

| Type | Build | Use when |
|---|---|---|
| **Quick (field) tooltip** | Drag fields into the visual's **Tooltips** bucket; pick aggregation or a quick measure | A few extra values on hover |
| **Report tooltip page** | New page > Canvas Type = Tooltip > Page Information Tooltip = On > add fields to **Tooltip fields** | Rich, visual, styled hover (cards, charts, help text) |

Report tooltip rules: associate via the trigger field; override per visual through its **Tooltips** card **Page** dropdown (**Auto** reverts to default). Always **hide** tooltip pages; keep them small (built-in Tooltip size is 320×240). Page tooltips are **not interactive** — for clicks, use a drillthrough page. For line charts, one tooltip covers all lines (legend isn't passed as a filter); when cross-highlighting, the tooltip shows highlighted data.

## Conditional formatting options (table/matrix)

Applies to **values** in **Table** and **Matrix** visuals only (not row/column headers; for matrices it targets the lowest visible level). Open via the field's dropdown > **Conditional formatting**:

| Option | Format styles | Notes |
|---|---|---|
| **Background color** / **Font color** | Gradient (color scale), Rules, Field value | Combine multiple options in one column; overrides manual cell colors |
| **Data bars** | min/max, colors, direction, axis; optional **Show bar only** | In-cell value comparison |
| **Icons** | Rules or Field value; Directional / Shapes / Indicators / Ratings sets | Icon left/right/only |
| **Web URL** | based on a URL field | Renders cells as live links |

Pitfalls: for percentage fields enter rule values as **decimals** and select **Number** (not **Percent**); NaN values (e.g. divide-by-zero) break auto min/max and percentage rules — use the `DIVIDE` DAX function. To color text states, build a color measure (`SWITCH(...)`) and format by **Field value**.
