# Retail Dashboard Templates

Audience-specific Power BI report layouts. Each visual named below is a confirmed standard Power BI visual. Measures referenced are defined in dax-measure-library.md. Place primary KPIs top-left (Western reading order); put Slicers in a left sidebar.

## Contents
- Executive retail dashboard
- Store manager operational dashboard
- Merchandiser dashboard
- E-commerce dashboard
- Layout and interactivity guidance

---

## Executive retail dashboard

Audience: leadership. Goal: company-wide health at a glance, minimal detail.

| Zone | Visual | Fields / measures |
|---|---|---|
| Top row | **Card** x4 | Total Revenue, Net Profit, Total Units, Transaction Count |
| Mid-left | **Line chart** | Total Revenue by Date (week), target reference line |
| Mid-right | **Filled map (choropleth)** | Total Revenue by Store geography |
| Bottom-left | **Bar chart** | Top 10 products by Total Revenue (Top N filter) |
| Bottom-right | **Donut chart** or **Treemap** | Total Revenue by Category |
| Sidebar | **Slicers** | Date range, Region, Category |
| Optional | **Smart narrative** | Auto summary of revenue/profit trend |

Drill-through: from the Bar chart to a product transaction detail page.

## Store manager operational dashboard

Audience: single-store manager. Goal: today's operations and alerts.

| Zone | Visual | Fields / measures |
|---|---|---|
| Top | **Gauge chart** | Today's Total Revenue vs daily target |
| Top | **KPI** | Same-Store Sales % vs prior period |
| Mid | **Column chart** | Total Revenue by hour-of-day (or Day of Week) |
| Mid | **Matrix** | Product rows; columns Total Units, Gross Margin %, Ending On Hand Qty; conditional formatting |
| Lower | **Funnel chart** | Promotion participation -> purchase conversion |
| Lower | **Table** | Low-stock alerts (Ending On Hand Qty < Reorder Point), red conditional formatting |
| Sidebar | **Slicers** | Date, Department |

Notes: store is fixed via report-level filter; inventory uses semi-additive Ending On Hand Qty.

## Merchandiser dashboard

Audience: category/merchandising manager. Goal: assortment, promotion, and inventory efficiency.

| Zone | Visual | Fields / measures |
|---|---|---|
| Top row | **Card** x3 | Gross Margin %, Sell-Through %, GMROI |
| Mid-left | **Treemap** | Total Revenue by Department -> Category -> Brand |
| Mid-right | **Scatter chart** | Discount % (X) vs Promotion Lift % (Y), bubble = Total Units |
| Lower-left | **Combo chart** | Total Gross Profit (column) + Gross Margin % (line) by Category |
| Lower-right | **Scatter chart** | Days of Supply (X) vs Sales Velocity (Y) - stockout/excess quadrants |
| Bottom | **Ribbon chart** | Brand rank by Total Revenue across quarters |
| Sidebar | **Slicers** | Category, Brand, Promotion Name |

Drill-through: Decomposition tree page on Net Profit (Channel -> Category -> Product) for root cause.

## E-commerce dashboard

Audience: online/digital team. Goal: traffic, conversion, channel ROI.

| Zone | Visual | Fields / measures |
|---|---|---|
| Top row | **Card** x4 | Total Sessions, Conversion Rate %, Order Dollars, Revenue per Session |
| Mid-left | **Funnel chart** | Visit -> Add to Cart -> Checkout -> Purchase (distinct sessions) |
| Mid-right | **Combo chart** | Abandoned Sessions (column) + Abandonment Rate % (line) by week |
| Lower-left | **Bar chart** | Revenue per Session by Referral Medium/Source (campaign attribution) |
| Lower-right | **Clustered column chart** | Conversion Rate % by Device Type |
| Optional | **Q&A visual** | Ad-hoc natural-language queries for the digital team |
| Sidebar | **Slicers** | Date, Channel, Campaign |

Drill-across: place a clickstream visual and a POS visual on one page; cross-filter by conformed Customer to study web-to-store behavior.

---

## Layout and interactivity guidance

- Keep visuals per page modest (target under ~20) for performance.
- Use one consistent Slicer set per report; sync slicers across pages when appropriate.
- Reserve AI visuals (Key influencers chart, Decomposition tree, Smart narrative, Q&A visual) for exploratory pages, not dense operational pages.
- Configure drill-through detail pages filtered by Product / Store / Promotion, showing a Table at POS atomic grain (POS Transaction Number degenerate dimension).
- For semi-additive inventory tiles, use Ending On Hand Qty or Average On Hand Qty - never a raw SUM across dates.
