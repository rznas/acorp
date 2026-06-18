# Interactivity Patterns

How to wire a Power BI dashboard so users can filter, focus, switch views, and reach detail — without clutter or surprises. Drill-down/drillthrough *mechanics* are owned by the **drill-pattern-design** skill; this file covers placement and coordination on a composed page.

## Contents

- Filter layers (visual / page / report / drillthrough)
- Slicer placement and styles
- Sync slicers across pages
- Slicer selection controls
- Cross-filter and cross-highlight (Edit interactions)
- Bookmarks for view switching
- Buttons and bookmark navigators
- Tooltips and drillthrough (coordination)
- Q&A for ad-hoc questions
- Interactivity validation checklist

## Filter layers

Four filter scopes, narrowest to broadest, set in the **Filters** pane:

- **Visual filter** — one visual only (e.g., Top N on a table).
- **Page filter** — all visuals on the page.
- **Report filter** — all pages.
- **Drillthrough filter** — passes the clicked entity to a focused destination page.

Report readers can switch **Basic** vs **Advanced** filtering; turn switching off (and lock filter type) when you need a fixed filter. Apply the **most restrictive** filters for performance.

## Slicer placement and styles

Place a **Slicer** where users expect it and keep it there on every page:

- **Left rail (vertical list)** — the default; good for a small stack of filters.
- **Top bar (Tile style + Responsive layout)** — horizontal buttons that resize to the screen; ideal for one or two high-traffic filters and for mobile.

Slicers show the current filter state on the canvas (no need to open the Filters pane), can filter on hidden columns, and focus a report when placed next to key visuals. They do **not** support input fields or drill-down.

Format options: **Vertical list** (default) vs **Tile**; add **Background**, **Border**, **Shadow** under General > Effects; **Lock aspect** to preserve proportions on resize.

## Sync slicers across pages

Use **View > Sync slicers** to share a selection across pages:

- In the **Sync** column choose which pages share the selection; in the **Visible** column choose where the slicer is shown.
- A slicer synced but not visible on a page still filters that page.
- You can move/resize/format a synced slicer independently per page.
- Sync two *separate* slicers by giving them the same **group name** under Advanced options (useful for composite models); choose to sync filter changes, field changes, or both.

## Slicer selection controls

- **Show "Select all"** (off by default) — adds a Select all item; deselecting one item while all are selected creates an *is-not* filter.
- **Single select** (off by default) — restricts to one value; vertical-list checkboxes become radio buttons.
- Default sort is ascending; switch via the slicer ellipsis > Sort axis.

## Cross-filter and cross-highlight (Edit interactions)

By default, selecting a data point **filters or highlights** the other visuals on the page; highlighting dims unrelated data rather than removing it. Control this explicitly:

1. Select the source visual (or slicer).
2. Format > **Edit interactions** (Visual interactions in the service).
3. On each target visual choose **Filter**, **Highlight**, or **None**.

Pattern: set context visuals (e.g., a "company total" or comparison chart) to **None** so a slicer/selection doesn't filter them, keeping the baseline stable while the rest of the page reacts.

## Bookmarks for view switching

A **bookmark** captures page, filters, slicer state and type, cross-highlight selection, sort order, drill location, spotlight, and object visibility (from the Selection pane).

View-switching pattern (e.g., toggle a visual between Revenue and Units, or swap a chart for a table):

1. Arrange state A; add a bookmark. Arrange state B; add a bookmark.
2. For switching bookmarks, **turn off the bookmark's Data property** so selecting it doesn't reset the user's slicers — keep only Display/Current page.
3. Use the **Selection** pane to show/hide the relevant visuals per bookmark; after changing visibility, **Update** the bookmark.

Group bookmarks (Ctrl-select > Group) to organize a set; use **View** to play a group as a guided slide show.

Caveats: adding a slicer to a page after bookmarks were created clears that slicer; bookmarks restore visual *state*, not visual *position*.

## Buttons and bookmark navigators

Drive bookmarks from UI controls:

- Assign a bookmark to a **Button** or **Shape** (Action > Type = Bookmark) to build a table of contents or a view toggle.
- A **bookmark navigator** auto-creates a button set linked to a bookmark group — the cleanest way to offer "Revenue | Units | Margin" style toggles.
- In editing mode Ctrl-click the control to follow it; in reading mode a single click follows it.

## Tooltips and drillthrough (coordination)

Keep detail off the main canvas to protect both clarity and performance:

- **Report-page tooltips** show a mini-page of context on hover. Add them as ancillary info only — not for essential information, since keyboard/no-mouse users can't reach them easily.
- **Drillthrough pages** focus on the entity the user right-clicked (e.g., one supplier). Define the layout and back-button experience in **drill-pattern-design**; on the composed page just expose the entry points (right-click data point) and keep destination pages tightly filtered for speed.

## Q&A for ad-hoc questions

Add the **Q&A** visual on analytical pages to let users type natural-language questions ("top 10 products by revenue this year"); it builds an appropriate visual. It needs a well-named, well-related semantic model — coordinate naming/synonyms with **ai-powered-analysis-selection**.

## Interactivity validation checklist

```
- [ ] Slicers in a consistent position across all pages
- [ ] High-traffic filters synced with Sync slicers
- [ ] Edit interactions reviewed; context/baseline visuals set to None
- [ ] View-switch bookmarks have Data property turned off
- [ ] Bookmark visibility updated after Selection-pane changes
- [ ] View toggles delivered via Buttons / bookmark navigator
- [ ] Detail pushed to drillthrough pages and tooltips, not the main canvas
- [ ] No essential info living only in a tooltip (accessibility)
- [ ] Q&A page backed by a well-named semantic model
```
