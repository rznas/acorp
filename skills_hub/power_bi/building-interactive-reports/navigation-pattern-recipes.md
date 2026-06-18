# Navigation Pattern Recipes (Bookmarks + Buttons)

Step-by-step recipes for `building-interactive-reports`. Each is grounded in the Power BI docs for bookmarks, buttons, navigators, and drillthrough. All require Edit permission; test buttons with **Ctrl+select** in edit mode.

## Contents
- Recipe 1 — Page navigation bar (Page navigator)
- Recipe 2 — Bookmark navigation bar (Bookmark navigator)
- Recipe 3 — Toggle / deselect state (Allow deselection)
- Recipe 4 — Visual table of contents (bookmark-linked buttons)
- Recipe 5 — Swap views without resetting filters (Selection pane + bookmarks)
- Recipe 6 — Spotlight / slide-show walkthrough
- Recipe 7 — Drillthrough page (right-click + button)
- Recipe 8 — Conditional single-button navigation (slicer-driven destination)

## Recipe 1 — Page navigation bar (Page navigator)

Use when readers need to move between report pages and you don't want to manage bookmarks.

1. **Insert** ribbon > **Buttons** > **Navigator** > **Page navigator** (service: Edit > menu bar > **Buttons** > **Navigator** > **Page navigator**).
2. Power BI auto-creates a bar synced to your pages — labels = page display names, order = page order, current page selected; it auto-updates as you add/rename pages.
3. To hide pages or include tooltip pages, open **Format navigator** > **Pages**.
4. Test: **Ctrl+select** a page button.

## Recipe 2 — Bookmark navigation bar (Bookmark navigator)

Use to switch between saved states (e.g. different views of the same data).

1. First **create the bookmarks** you want in the bar (View > Bookmarks > Add). Put them in a **bookmark group** if you'll have more than one navigator.
2. **Insert** > **Buttons** > **Navigator** > **Bookmark navigator**.
3. The bar syncs to your bookmarks (labels, order, last-selected). To limit it, open **Format navigator** > **Bookmarks** and select a specific group.
4. Format the **Selected state** (Fill/Text) so the active button stands out; set **Grid layout** (Horizontal/Vertical/Grid + Padding).
5. Accessibility: give the navigator descriptive **Alt text** (e.g. "2025 sales data modifier"), keep it next to the visual it changes, and name those visuals to match the button labels.

## Recipe 3 — Toggle / deselect state (Allow deselection)

Use for an on/off toggle or an empty default (e.g. a "no filter" landing state).

1. Build a bookmark capturing the **deselected** state (e.g. *No filter*).
2. Create a **Bookmark navigator** (Recipe 2).
3. In **Format navigator** > **Bookmarks**, turn on **Allow deselection**.
4. Under **Launch on deselection**, select the deselected bookmark (e.g. *No filter*).
5. To keep that bookmark out of the visible bar, turn on **Hide deselection bookmark**.

## Recipe 4 — Visual table of contents (bookmark-linked buttons)

Use to build a custom landing/index page where each item jumps to a curated view.

1. Create one **bookmark** per destination view (set each up exactly as you want it).
2. For each entry, insert a **button** (Insert > Buttons > Blank) — or a shape/image.
3. In **Format button**, set **Action = On**, **Type = Bookmark**, then pick the bookmark.
4. Optionally style each button's states (Default / On hover / On press) and add fill images.
5. For view-only bookmarks (no data change), turn **off** the bookmark's **Data** property so reader filters aren't reset.

## Recipe 5 — Swap views without resetting filters (Selection pane + bookmarks)

Use to show/hide groups of visuals (e.g. "chart view" vs "table view") in place.

1. Place all visuals for both views on the page.
2. Open **View > Selection** pane. Toggle the eye icons to set the visibility for **view A**; **Add** a bookmark.
3. Toggle visibility for **view B**; **Add** a second bookmark.
4. For each bookmark, turn **off Data** (keep filters) and keep **Display** on (visibility).
5. Wire buttons (Recipe 4) or a navigator (Recipe 2) to the two bookmarks.
6. After any later visibility change, right-click the bookmark > **Update** (changing visibility otherwise reverts to default).

## Recipe 6 — Spotlight / slide-show walkthrough

Use to present a sequence of insights.

1. Configure each insight page state (filters, spotlight on a visual) and **Add** a bookmark; arrange the order by drag-and-drop in the Bookmarks pane.
2. (Optional) Group the bookmarks; **View** from inside a group plays only that group, otherwise all bookmarks play.
3. In the Bookmarks pane select **View** to start the slide show (works in Desktop and the service in Edit or Reading mode).

## Recipe 7 — Drillthrough page (right-click + button)

Use to send users from a summary point to a detail page filtered to that entity.

1. Create a new page; in the Visualizations pane drag the entity field (e.g. `Customer Name`) into the **Drillthrough filters** well.
2. Add visuals that respond to that filter; match the report's theme/style so users feel they're in the same report.
3. (Author tip) Temporarily set the drillthrough filter to preview a realistic result; **remove** that preview filter before publishing.
4. A **Back** button is auto-added when you assign the drillthrough filter — keep it.
5. Users trigger drillthrough by **right-clicking** a data point and choosing the page. To also offer a button: insert a button, **Action Type = Drill through**, select the target page; use **conditional formatting** to show/hide it by filter context.
6. **Hide** the drillthrough page. If you keep it visible, add a button assigned to a bookmark that **clears all filters**.
7. Cross-report drillthrough (service only): publish source and target to the same workspace, enable **Cross-report drillthrough** in the target's settings, use matching field names and data types, and refresh both.

Design patterns (guidance): **Additional depth** (summary → transaction detail) and **Broader perspective** (a point → holistic context). Avoid visuals that go BLANK or error when filtered.

Troubleshooting: if the page isn't in the right-click menu, confirm the field is in the **Drillthrough filters** well, matches the source field, and isn't aggregated; use **Performance analyzer** to inspect.

## Recipe 8 — Conditional single-button navigation (slicer-driven destination)

Use to save canvas space: one "Go" button navigates to whichever page the user picks.

1. **Home** > **Enter data** to create a one-column table whose values exactly match your target page names; add it as a **single-select slicer**.
2. Insert a button (Insert > Buttons > Blank); set **Action = On**, **Type = Page navigation**.
3. Under **Destination**, select the **Conditional formatting (fx)** icon.
4. In **Destination – Action**, under "What field should we base this on?", choose the table column from Step 1.
5. Pick a page in the slicer, then select the button — it navigates to that page. (Power BI matches the destination by exact string, so page names and slicer values must match.)
