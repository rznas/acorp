---
name: building-paginated-reports
description: Builds pixel-perfect, printable, multi-page operational reports in Power BI Report Builder (RDL) — tablix tables and matrices, grouping, pagination (page breaks, headers/footers, repeating headings), report parameters including cascading parameters, embedded data sources and datasets, row-level security, and publishing to the Power BI service. Use when the user needs print-ready or PDF output, invoices, statements, or operational reports that must show every row across many pages, or asks about paginated reports, RDL files, Report Builder, tablix, or printing/exporting tables that would otherwise be truncated. Not for interactive analytic Power BI reports (use building-interactive-reports).
---

# Building Paginated Reports

## Overview

Power BI paginated reports (RDL) are optimized for **printing and PDF generation** and produce highly formatted, print-ready layouts. They print **every** row of a table across as many pages as needed — unlike an interactive Power BI report table, which shows a scroll bar and prints only the visible rows. Use them for operational documents such as invoices, statements, and catalogs.

Paginated reports use a **separate tool track**: you author them in **Power BI Report Builder** (a standalone, free tool), not in Power BI Desktop. The output is a **report definition** (`.rdl`) that specifies where to get the data, which data to get, and how to display it — it does not contain the data itself. A single paginated report has **no underlying data model**; you create **embedded** data sources and datasets inside the report. Shared data sources/datasets (`.rds`/`.rsd`) are **not** supported.

## When to Use

Choose a paginated report when ANY of these is true:
- The report **must be printed or output as PDF**.
- A table/matrix would **overflow** — an interactive Power BI table can't dynamically resize to print all rows; a paginated report paginates every row.
- You need **precision layout** (margins, page size in inches/centimeters, page headers/footers), **multiple render formats** (Excel, Word, PowerPoint, PDF, CSV, XML, MHTML, TIFF), **mail-merge**, **subreports**, **per-user layouts**, or **cascading/data-driven parameters**.

Choose an **interactive** Power BI report instead (skill `building-interactive-reports`) for exploration, slicers, cross-filtering, and modern visuals. See [paginated-vs-interactive-decision-guide.md](paginated-vs-interactive-decision-guide.md) for the full decision matrix and scenario list.

## Build workflow

Copy this checklist and check off items as you complete them:

```
Paginated report build:
- [ ] Step 1: Decide paginated vs interactive (decision guide)
- [ ] Step 2: Install Power BI Report Builder
- [ ] Step 3: Create embedded data source + dataset (wizard or manual)
- [ ] Step 4: Build the tablix (Table/Matrix wizard) with row/column groups
- [ ] Step 5: Format for print (page size, margins, title, page numbers, currency)
- [ ] Step 6: Configure pagination (repeat headings, page breaks, page names)
- [ ] Step 7: Add parameters (incl. cascading) to filter data
- [ ] Step 8: Add RLS via the UserID built-in field (if data must be per-user)
- [ ] Step 9: Preview (Run), then Publish to the Power BI service
- [ ] Step 10: Set data-source credentials / gateway in the service
```

**Step 1 — Decide.** Confirm a paginated report is the right tool. See [paginated-vs-interactive-decision-guide.md](paginated-vs-interactive-decision-guide.md).

**Step 2 — Install.** Install **Power BI Report Builder** from the Microsoft Download Center (`https://aka.ms/pbireportbuilder`). Licensing matches `.pbix`: a free license publishes only to **My Workspace**; Pro or PPU publishes to other workspaces (needs at least a **Contributor** role there).

**Step 3 — Data source + dataset.** A paginated report uses **embedded** data sources and datasets. Natively supported sources include Azure SQL Database, Azure Synapse Analytics, Azure SQL Managed Instance, Azure Analysis Services, **Power BI semantic models**, Direct Lake models, Dataverse, and **Enter data** (static). SQL Server, SQL Server Analysis Services, Oracle, Teradata, and ODBC connect **via a gateway**. See [report-builder-build-steps.md](report-builder-build-steps.md) for the connection walkthrough and the full source table. Relates to skill `connecting-data-sources`.

**Step 4 — Tablix.** Start with a Report Builder **wizard**: **Table or Matrix Wizard** for column/summarized data, **Chart Wizard** for graphics, **Map Wizard** for spatial data, or a free-form **list** for invoices. Drag fields to **Row groups** / **Column groups** / **Values** (numeric values default to sums). The resulting data region is a **tablix** (the table, matrix, and list are all tablix behind the scenes), named `Tablix1` by default. Groups appear in the **Grouping** pane.

**Step 5 — Format for print.** A typical page is 8.5 × 11 inches with 1-inch margins; narrow margins (e.g. 0.75 in) so content fits. Add a title, drag **Page Number** and **Overall Total Pages** built-in fields into the footer, and format currency cells via the **$** button + **Sample Values**. Full step-by-step: [report-builder-build-steps.md](report-builder-build-steps.md).

**Step 6 — Pagination.** Make column headings repeat on every printed page and control page breaks/names. See [parameters-and-pagination-recipes.md](parameters-and-pagination-recipes.md).

**Step 7 — Parameters.** Add report parameters to filter data; use **cascading** parameters so each parameter's choices depend on the previous one. See [parameters-and-pagination-recipes.md](parameters-and-pagination-recipes.md).

**Step 8 — RLS.** For per-user data, create a parameter assigned the built-in **UserID** field and use it in a dataset **filter** or **query**. See [parameters-and-pagination-recipes.md](parameters-and-pagination-recipes.md). Relates to skill `securing-row-level-access`.

**Step 9 — Preview & publish.** Select **Run** to preview; use **Print Layout** to verify pagination. Then **File > Publish**, sign in, pick a workspace, and **Save** — or **Upload** the `.rdl` from the service. Relates to skill `publishing-and-sharing-content`.

**Step 10 — Credentials/gateway.** In the service, set credentials via the report's **... > Manage > Edit credentials**. On-premises sources require an on-premises data gateway (**Manage connections and gateways**).

## Key concepts

| Concept | What it means in a paginated report |
|---------|--------------------------------------|
| **Report definition (.rdl)** | Specifies *where/which/how* to get & display data; contains no data. Max file size 20 MB. |
| **Embedded data source/dataset** | Defined inside the report; shared `.rds`/`.rsd` not supported. |
| **Tablix** | The single underlying data region for table, matrix, and list; supports nested/adjacent groups. |
| **Row/Column groups** | Grouping of data in the Grouping pane; drive subtotals and pagination behavior. |
| **Report body vs physical page** | The body grows to fit content and can span pages; the physical page size (paper) drives where hard page breaks fall. |
| **Render-extension pagination** | Pagination differs by renderer: HTML/Excel/Word/MHTML add soft page breaks; PDF/Image use hard page breaks. Design for the renderer you'll deliver. |
| **Report parameter** | Filters data, controls appearance, or supports what-if; can be cascading or data-driven. |
| **Cascading parameters** | Ordered parameters where one parameter's available values depend on an earlier selection. |
| **UserID built-in field** | The authenticated user; the basis for RLS in a paginated report. |

## Common pitfalls

- **Using a paginated report for exploration.** If users need slicers, cross-filtering, and modern visuals, build an interactive report instead. Paginated is for print/PDF/operational output.
- **Report renders across too many pages.** Body width + margins exceeds physical page width. Reduce the container by dragging its corner left; keep body + margins ≤ page width.
- **Headings vanish after page 1.** Set the static row-group member's **RepeatOnNewPage = True** (and **KeepWithGroup = After**) via Advanced Mode in the Grouping pane.
- **Designing for the wrong renderer.** HTML/Excel and PDF paginate differently; newsletter-style columns apply only to PDF/Image; **document maps render only on export, not in the service**. Verify in the target format.
- **Expecting "View As role" to test RLS.** The **Test/View as role** feature **does not work** for paginated reports — test by signing in as the actual user or by verifying the UserID filter/query logic.
- **Assuming stored procedures everywhere.** Stored procedures are preferred for performance but are **not supported for gateway relational sources** (SQL Server, Oracle, Teradata). Use parameterized text queries there.
- **Forgetting credentials/gateway after upload.** A freshly uploaded report errors until you set data-source credentials; on-prem sources need a gateway binding.
- **Naive period-end aggregation in dataset SQL.** Semi-additive balances (inventory on hand, account balance) must not be naively summed across dates in your query — sum across non-time dimensions only and take a period-end/average across time (cross-ref `understanding-fact-additivity`).

## Success criteria

- The correct tool was chosen (paginated for print/PDF/overflow; interactive otherwise).
- Data source(s) and dataset(s) are embedded and connect successfully (with a gateway for on-prem).
- The tablix prints every row, with column headings repeating on each page.
- Page size/margins/headers/footers produce clean printed pages with page numbers in the target render format.
- Parameters (and cascading parameters) filter correctly and are ordered so dependencies resolve.
- RLS, where required, filters by the UserID built-in field and was verified by signing in as a real user.
- The report is published to the intended workspace with working credentials.

---
*Provenance: built per SKILL 22 "Building Paginated Reports" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `guidance/report-paginated-or-power-bi.md`, `paginated-reports/paginated-reports-report-builder-power-bi.md`, `paginated-reports-quickstart-aw.md`, `paginated-reports-data-sources.md`, `paginated-reports-pagination.md`, `report-builder-add-document-map.md`, `paginated-reports-rls.md`, `paginated-reports-save-to-power-bi-service.md`, `parameters/*`, and `guidance/paginated-report-cascading-parameter.md`.*
