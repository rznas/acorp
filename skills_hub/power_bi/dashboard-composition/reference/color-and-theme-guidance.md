# Color and Theme Guidance

How to apply color, themes, and conditional formatting to a Power BI dashboard for accessibility and corporate branding.

## Contents

- Apply one theme for consistency
- Built-in and accessible themes
- Custom (branded) themes via JSON
- Accent color for emphasis
- Conditional formatting (status color)
- Semantic status palette
- Accessibility rules (contrast, color independence, unsafe pairs)
- Color validation checklist

## Apply one theme for consistency

Apply a **report theme** (View > Themes) so every visual inherits one coordinated palette and default formatting. Changing the theme restyles all visuals at once. Themes are applied in **Power BI Desktop**; you can't apply a theme to an existing report in the Power BI service. Undo a theme change with Ctrl+Z.

Keep color, font, and font size consistent across the report: same title font size on every visual, same color for the same dimension value across charts.

## Built-in and accessible themes

Power BI ships built-in themes including Default, Executive, Innovate, Solar, Storm, and others. For accessibility-first dashboards use:

- **Color-blind safe** theme
- **Accessible default** / **Accessible city park** themes
- **High contrast** theme

These are good starting palettes but still verify contrast and color independence with the rules below.

## Custom (branded) themes via JSON

For corporate branding, create a custom theme:

- Adjust a current theme in the **Customize theme** dialog and save it, or
- Author a theme **JSON file** defining the corporate color palette, fonts, and default visual formatting, then **Browse for themes** to import it.

A tenant admin can publish **organizational themes** so every author in the company starts from the brand palette. Use the custom theme to encode the brand's primary, secondary, and accent colors as the data-color sequence so charts are on-brand by default.

## Accent color for emphasis

Reserve a single accent color to call out one data point. In a chart, set all categories to one base color, then set the one data point (e.g., the launch segment) to the accent color so it stands out. Set the **Default color** so newly added series keep the scheme.

## Conditional formatting (status color)

Drive color from a value so status is visible at a glance. On a chart's color (fx) or in **Table**/**Matrix** cells, choose a format style:

- **Rules** — discrete bands (e.g., red < 85%, yellow 85-95%, green > 95%).
- **Gradient** — min/max (optionally mid) color scale tied to a measure.
- **Field value** — color comes from a measure/column that returns a color string.

You can base the color on a measure not shown in the visual (e.g., color bars by `Sum of Profit`). Use this for KPI status on **Matrix** (over/under budget), stockout risk, claim aging, etc. — but always pair color with text or an icon (see color independence below).

Example status measure returning a color string for **Field value** formatting:

```dax
Status Color =
SWITCH (
    TRUE (),
    [On Time Delivery %] >= 0.95, "#107C10",   -- green
    [On Time Delivery %] >= 0.85, "#F2C811",   -- amber
    "#D13438"                                   -- red
)
```

## Semantic status palette

Use a consistent, color-independent meaning across the whole dashboard:

| State | Color (example) | Always pair with |
|-------|-----------------|------------------|
| Good / on target | green `#107C10` | up arrow / check / "On target" |
| Warning | amber `#F2C811` | dash / "Watch" |
| Bad / off target | red `#D13438` | down arrow / X / "Off target" |
| Neutral / no target | grey `#605E5C` | "n/a" |

## Accessibility rules

1. **Contrast >= 4.5:1.** WCAG 2.1 (1.4.3) requires title, axis-label, and data-label text to have at least a 4.5:1 contrast ratio against the background. Verify with a contrast checker.
2. **Never use color as the only signal.** Add markers, icons, or text. For **Line**, **Area**, **Combo**, **Scatter**, and **Bubble** visuals, turn on **Markers** and use a different marker shape per series.
3. **Use fewer colors / a monochrome palette** where possible — it is more accessible and less cluttering.
4. **Test for color vision deficiency** (simulator) and **low vision** (lower screen brightness / squint test).

### Color pairs to avoid together in one chart or page

- green & red
- green & brown
- blue & purple
- green & blue
- light green & yellow
- blue & grey
- green & grey
- green & black

## Color validation checklist

```
- [ ] One report theme applied (brand JSON for corporate look)
- [ ] Accessible / Color-blind safe theme used, or contrast verified
- [ ] Text vs background contrast >= 4.5:1
- [ ] No information conveyed by color alone (icons/text/markers added)
- [ ] Status colors consistent (green/amber/red mean the same everywhere)
- [ ] No unsafe color pairs combined in one chart/page
- [ ] Same dimension value uses the same color across all visuals
- [ ] One accent color reserved for emphasis only
```
