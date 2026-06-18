# Mobile-Layout Checklist

Checklist for `building-interactive-reports`, Step 7. Grounded in the Power BI mobile-optimized report docs. A mobile (portrait) layout is a **separate, per-page** layout; the regular page is unchanged.

## When a mobile layout matters

- The mobile-optimized view shows in the **Power BI mobile apps for iOS and Android** when the phone is held in **portrait**. Tipping the phone landscape shows the regular page.
- A **web browser always shows the standard (non-optimized) view** — mobile layouts never appear there.
- Reports with a mobile layout get a special **optimized icon** in the mobile apps.

## Build checklist

```
Mobile layout build:
- [ ] 1. Open mobile layout view (View ribbon > Mobile layout, or the layout switcher)
- [ ] 2. (Optional) Auto-create mobile layout as a starting point
- [ ] 3. Drag the visuals that matter onto the portrait grid canvas
- [ ] 4. Reformat visuals for small screens (Visualizations pane)
- [ ] 5. Order layers (Selection pane); lock visuals; use snap-to-grid
- [ ] 6. Verify buttons, slicers, and visuals on the interactive canvas
- [ ] 7. Publish the main report (or save in the service)
- [ ] 8. Test in the mobile app in portrait
```

### 1. Open mobile layout view
- **Desktop:** **View** ribbon > **Mobile layout**, or the layout switcher at the bottom of the page.
- **Service:** **Edit**, then **Mobile layout** or the bottom layout switcher (under **More options (...)** if Edit isn't visible).

### 2. Auto-create (optional starting point)
- **Auto-create mobile layout** generates a layout from the desktop page. It reads **left-to-right from the top**, considers visual position/size/type/order, and places both **visible and hidden** visuals (so visibility bookmarks keep working).
- It works best on **simple** reports. Re-running it replaces the existing mobile layout.

### 3–5. Lay out and format
- Drag visuals from the **Page visuals** pane onto the gridded **mobile layout canvas**; the grid scales across phone sizes.
- Use the **Visualizations** pane to reformat each visual for small screens; use the **Selection** pane to set layering order.
- Canvas controls: toggle the grid, enable/disable **snap-to-grid**, and **lock** visuals so they aren't moved accidentally.
- Pick only the visuals that make sense on a phone — a focused, readable subset beats the full page.

### 6. Verify on the canvas
- The canvas is interactive: you can see how **buttons, slicers, and visuals** behave before publishing.

### 7. Publish
- Built in **Desktop:** publish the main report — the mobile layout publishes with it.
- Built in the **service:** just **save** the report.

## Design factors that improve auto-create quality
- Favor a **horizontal, top-down** desktop layout; strong vertical elements (e.g. a left-hand nav bar) render worse.
- The engine **can't handle background images**.
- Minimize **overlaying** visuals.
- Too many **images** may not all fit; remember hidden images are also placed, so the canvas can be fuller than it looks (leaving "unplaced" visuals in the Page visuals pane).

## Considerations and limitations
- **Tooltips are disabled on the mobile layout canvas**, but work when viewing in the mobile app.
- **Metric visuals aren't interactive** on the mobile layout canvas.
- **Slicer selections** on the mobile canvas don't carry to web layout; on publish, the **web-layout** slicer selections apply regardless of which view is shown.
