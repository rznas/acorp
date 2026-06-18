# Layout Composition Rules

How to arrange visuals on a Power BI report page (and service dashboard) so the screen reads clearly, fits one view, and performs well.

## Contents

- Audience-driven density
- One-screen storytelling
- Visual hierarchy and placement (Z-pattern)
- Grid, alignment, and spacing
- Choosing the right visual for the slot
- Number and label formatting
- Performance rules (fewer visuals, restrictive filters, tile cache)
- Layering and tab order
- Layout validation checklist

## Audience-driven density

| Archetype | Visual budget | Scroll? | Detail strategy |
|-----------|---------------|---------|-----------------|
| Executive | 4-8 | No | Drill into underlying reports for detail |
| Operational | 8-14 | Minimal | Status visuals + alert table; drillthrough for detail |
| Analytical | up to ~20 | OK | Drill-down, drillthrough, AI visuals, multiple pages |

A dashboard is an overview. Don't put a detail on it unless the audience needs to monitor that detail. Where it's displayed matters: a wall monitor can hold more; a phone needs far fewer tiles.

## One-screen storytelling

- Tell one story on one screen. Avoid scroll bars on executive/operational dashboards.
- If the page feels cluttered, remove everything non-essential. Two visuals showing the same thing is one too many.
- Use full-screen / focus mode when presenting.

## Visual hierarchy and placement (Z-pattern)

- Most people read top-to-bottom, left-to-right (Western audiences). Put the **highest level of data top-left** and add detail as the eye moves right and down.
- Reserve the **top row** for headline numbers using **Card**, **Multi-row card**, or **KPI** visuals so the key figure stands out.
- Provide context for every number (a bare card with no label/comparison is ambiguous).
- Group related visuals together so the eye doesn't jump around.

## Grid, alignment, and spacing

- Turn on **Gridlines** and **Snap to grid** (View menu) and align visual edges to it.
- Keep consistent gutters between visuals; ragged spacing reads as noise.
- Keep slicers in the **same position on every page** of a report for predictability.
- Use the **Selection** pane to group a background **Shape** with its visuals and set **Maintain layer order** so backgrounds don't jump to the front when clicked.

## Choosing the right visual for the slot

Avoid variety for its own sake; a visual should be easy to read.

- **Bar** / **Column** beat **Pie**, **Donut**, **Gauge** for comparing values (side-by-side comparison is easier for the eye).
- **Pie** / **Donut**: only for part-to-whole and only with fewer than 8 categories.
- **Gauge**: current value vs a goal.
- **Card** / **KPI**: a single prominent number.
- Avoid 3D charts and other hard-to-read forms.
- Keep chart scales, axis ordering, and category colors consistent across the page.
- Don't mix large and small measures on one axis (e.g., millions vs thousands); if you must, use a **Combo** chart with a secondary axis.
- Sort intentionally: by measure to surface highest/lowest; by axis to help users find a specific category.

## Number and label formatting

- Don't exceed three or four numerals; show 1-2 digits left of the decimal and scale (display "3.4 million", not "3,400,000").
- Keep time frames distinct and consistent across visuals; don't place last-month next to a single filtered month on the same page.
- Remove unnecessary data labels — users read bar values fine without them. If your eye is drawn more to labels than to the data, turn labels off (squint test).

## Performance rules

The single most important page-performance lever is **less data and fewer visuals**.

1. **Limit visuals per page.** Each visual issues at least one query; more visuals = slower page and more load on capacity. Microsoft guidance is to keep "the number of visuals on a particular report page to only what is necessary" and to offload depth to drillthrough pages and tooltips — it does not name a number. Aim for fewer than ~20 visuals as a practical ceiling (the master-plan deliverable / a community rule of thumb), not a documented Microsoft limit.
2. **Apply the most restrictive filters.** A **Table** left unfiltered can load 100M+ rows into memory each refresh. Apply a **Top N** visual-level filter (set a generous cap such as 10,000) — the user experience is unchanged but memory and query time drop sharply. Ask of every visual: is all this data needed?
3. **Move detail off the canvas.** Use **drillthrough pages** and **report-page tooltips** to provide depth without adding visuals to the main page.
4. **Profile custom visuals.** A poorly optimized custom visual can degrade the whole report; test each one.
5. **Aggregate at the right grain.** Summarize in the model; don't push atomic rows into summary visuals.

### Service-dashboard tile cache (service dashboards only)

Power BI caches dashboard tiles (except live-report tiles and streaming tiles), so a service dashboard makes a fast, consistent "first landing page"; users click through to reports for detail. With DirectQuery/Live connection, the cache refreshes periodically (hourly by default) and each refresh queries the source — once per security context when **row-level security (RLS)** is enabled, so RLS multiplies cache-refresh queries. Pin only often-used visuals.

## Layering and tab order

- Set **Tab order** (Selection pane) to match the visual reading order so keyboard users traverse the page logically.
- Hide decorative shapes/images from tab order so screen readers skip them.

## Layout validation checklist

```
- [ ] Page visual count kept to only what's necessary (aim for fewer than ~20 as a heuristic ceiling)
- [ ] Executive/operational page fits one screen (no scroll)
- [ ] Headline KPIs (Card/KPI/Multi-row card) are in the top row
- [ ] Highest-level data is top-left; detail flows right/down
- [ ] Visuals aligned to grid; consistent gutters
- [ ] Slicers in the same position as on other pages
- [ ] Comparison uses Bar/Column (not Pie/Donut/Gauge)
- [ ] Pie/Donut used only for part-to-whole with < 8 categories
- [ ] Numbers scaled (e.g., "3.4M"); time frames consistent
- [ ] Large tables filtered with Top N
- [ ] Detail pushed to drillthrough pages / tooltips
- [ ] Tab order set; decorative items hidden from tab order
```
