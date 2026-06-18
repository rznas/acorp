# Paginated vs Interactive Power BI Report — Decision Guide

## Contents
- Quick decision rule
- Side-by-side comparison
- When to choose paginated (scenario list)
- When to choose interactive
- Legacy SSRS / RDL reports
- Considerations and limitations of paginated reports

## Quick decision rule

> **Will the output be printed or exported to PDF, and must it show every row no matter how long?**
> YES → **paginated report** (Power BI Report Builder, `.rdl`).
> NO, users explore interactively → **interactive Power BI report** (Power BI Desktop, `.pbix`; skill `building-interactive-reports`).

Paginated reports are optimized for **printing / PDF generation** and **highly formatted, print-ready layouts** — ideal for operational reports such as **sales invoices**. Interactive Power BI reports are optimized for **exploration and interactivity** with a comprehensive range of modern visuals — ideal for analytic reports.

The deciding factor for tables: a table or matrix in an interactive Power BI report **can't dynamically resize** to display all data — it gives scroll bars, and printing/PDF captures only the rows currently on the page. The **same table in a paginated report** produces as many pages as needed to print **every** row.

## Side-by-side comparison

| Dimension | Paginated report (RDL) | Interactive Power BI report (.pbix) |
|-----------|------------------------|--------------------------------------|
| Authoring tool | **Power BI Report Builder** (standalone) | Power BI Desktop |
| Optimized for | Printing, PDF, exact layout | Exploration, interactivity |
| Data model | **None** — embedded data sources/datasets per report | Shared semantic model |
| Prints all rows of a long table | **Yes** — paginates to multiple pages | No — only visible rows print |
| Precision layout (margins, page size) | **Yes** — to fractions of an inch/cm | Limited |
| Render/export formats | Excel, Word, PowerPoint, PDF, CSV, XML, MHTML, TIFF, HTML | Mainly PDF / PowerPoint / image of the page |
| Parameters | Report parameters incl. **cascading / data-driven**; defaults; what-if | Slicers and filters |
| Modern interactive visuals & cross-filtering | Limited (toggle visibility, sort, links, tooltips) | **Full** |
| Per-user layout / mail-merge / subreports | **Yes** | No |
| Native queries / stored procedures | **Yes** (no model required) | Via Power Query / model |

## When to choose paginated (scenario list)

Favor a paginated report for these capabilities (most are not supported by interactive Power BI reports):

- **Print-ready** output with controlled overflow to multiple pages; margins, page headers/footers.
- **Multiple render formats** so users export in the format they need (Excel, Word, PowerPoint, PDF, CSV, XML, MHTML).
- **Precision layout** — exact size/location in inches or centimeters.
- **Dynamic / render-specific layout** via VB.NET expressions (e.g. disable drill toggles when rendered to PDF).
- **Native queries / stored procedures** against any supported data source, with parameterization, and **no need to publish a semantic model first**.
- **Static datasets** (Enter data) embedded in the report — handy for demos/POCs.
- **Highly customized parameterization** — data-driven and **cascading** parameters, parameter defaults, what-if, dynamic filtering/styling.
- **Flexible data grids** — fine-grained tablix layouts with nested/adjacent groups, headings repeating across pages, embedded subreports, data bars, sparklines, indicators.
- **Spatial data** (map data region with GEOGRAPHY/GEOMETRY or ESRI shapefiles), **gauges**, **HTML rendering**, **image data** stored as binary.
- **Mail merge**, **subreports**, **subscriptions** (scheduled emails with attachments), **per-user layouts**, and **interactivity** (toggle visibility, links, interactive sorting, tooltips, drillthrough to Power BI or other paginated reports).

## When to choose interactive

Choose an interactive Power BI report when users need to **explore data and discover relationships and patterns**: slicers, cross-highlighting, drillthrough across visuals, bookmarks, and the full library of modern visuals. Build these with the `building-interactive-reports` skill.

## Legacy SSRS / RDL reports

If you already have **SQL Server Reporting Services (SSRS)** RDL reports you can either **migrate** them to Power BI as paginated reports, or **redevelop** them as interactive Power BI reports. Reports intended to deliver **analytic** experiences are often better **redeveloped** as interactive reports; operational/print reports migrate well as paginated. Once published, paginated and interactive reports live **side by side** in a workspace and can be distributed via Power BI apps. Paginated reports authored for SSRS 2016+ or Power BI Report Server on-premises are compatible with the Power BI service (not all features are available).

## Considerations and limitations of paginated reports

- **Memory limits** vary by license type and apply to processing and to PDF/Excel export; large datasets or complex formatting can hit them.
- **Not supported:** pinning report pages/visuals to dashboards; shared data sources & shared datasets; data-driven subscriptions in the service.
- **Document maps** (table-of-contents side pane) **don't render in the service** — they do render on **export** (PDF Bookmarks pane, Word table of contents, Excel worksheet of links; ignored by Atom/TIFF/XML/CSV).
- **`.rdl` file size** max 20 MB — for large files prefer external images over embedded.
- **Test/View as role does not work** for paginated-report RLS.
