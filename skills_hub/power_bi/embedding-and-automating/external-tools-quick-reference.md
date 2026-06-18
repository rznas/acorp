# External Tools Quick Reference

How third-party tools extend Power BI Desktop, plus custom-visual basics. Grounded in `transform-model/desktop-external-tools.md` and `developer/visuals/develop-power-bi-visuals.md` (+ `guidelines-powerbi-visuals.md`). Microsoft supports only the underlying public APIs, not the third-party tools themselves.

## How the External Tools ribbon works

The **External Tools** ribbon shows locally installed, *registered* tools. When you launch a tool from the ribbon, Power BI Desktop passes the running model's **Analysis Services server name, port number, and model name** as command-line parameters, so the tool connects to the live model automatically. (Power BI Desktop runs an Analysis Services process per `.pbix` session with a random port and a GUID model name, which is why this hand-off is needed.)

Tools connect via Analysis Services client libraries and the **Tabular Object Model (TOM)**:
- All TOM metadata is readable.
- Write operations are supported (e.g. override a measure's format string, edit KPIs/detail rows, create object-/row-level-security roles, add translations) but can cause unexpected behavior or model inconsistencies — use caution and validate in Desktop afterward.

Limitations: external tools are not supported in Power BI Desktop for Power BI Report Server; editing a `.pbit` template is not supported; sending processing commands to a model loaded in Desktop is not supported.

## Tool categories

- **Semantic modeling** — DAX optimization, ALM, metadata translation (Tabular Editor, DAX Studio, ALM Toolkit, Metadata Translator).
- **Data analysis** — read-only connect to query/analyze (e.g. launch Python, Excel, Power BI Report Builder; model documentation tools).
- **Miscellaneous** — tools that do not connect to the model (tutorials, DAX Guide, installers).
- **Custom** — your own scripts, registered via a `*.pbitool.json` file.

## Featured open-source tools

| Tool | What it does | Source |
|------|--------------|--------|
| **PowerBI.tips – Business Ops** | One-stop installer/registrar for the external tools below | powerbi.tips |
| **Tabular Editor** | Build/maintain tabular models; hierarchical object view, multi-select property editing, DAX syntax highlighting | tabulareditor.com |
| **DAX Studio** | DAX authoring, object browsing, integrated tracing, query execution breakdowns/statistics, DAX formatting | daxstudio.org |
| **ALM Toolkit** | Schema compare for models; diff/merge metadata files/branches/repos, deploy across environments, retain incremental-refresh history, reuse definitions | alm-toolkit.com |
| **Metadata Translator** | Auto-translate captions, descriptions, display-folder names (tables/columns/measures/hierarchies) via Azure Cognitive Services; export/import CSV for bulk editing | GitHub (microsoft/Analysis-Services) |

## Registering a tool

A tool is registered when a `*.pbitool.json` file exists in:
`C:/Program Files (x86)/Common Files/Microsoft Shared/Power BI Desktop/External Tools`

Some tools (ALM Toolkit, DAX Studio) create this file on install; others (e.g. SQL Profiler) must be registered manually. A registered tool with an icon appears on the ribbon.

**Disabling the ribbon** (Group Policy or registry):
- Key: `Software/Policies/Microsoft/Power BI Desktop/`
- Value: `EnableExternalTools` — `1` = enabled (default), `0` = disabled.

## Custom visuals (basics)

When the core visuals and AppSource imports do not fit, build a custom visual:
- A custom visual is packaged as a single **`.pbiviz`** file containing the rendering code; it is imported into a report.
- Audience: just you, your organization, or the whole community via **Microsoft AppSource**.
- AppSource visuals can be free or **additional purchase may be required** (IAP); both can be **certified** (meet certification requirements). Paid licensing uses the Licensing API; customers manage licenses in the Microsoft 365 admin portal.
- To build one (web developer): set up the developer environment, learn the visual project structure, and follow the circle-card tutorial — see the `developer/visuals/` docs. Do not invent visuals-API calls; defer to those docs for code.
