# ALM, source control, and report/model separation

Detailed reference for application lifecycle management of Power BI / Fabric content: separating reports from semantic models, choosing content formats for source control, version control vs source control, branching, and the TMDL view. Grounded in the Microsoft report-separate-from-model, content-lifecycle develop-manage, and TMDL-view docs.

## Contents
- Separate the report from the semantic model
- Preserve the model interface (safe vs breaking changes)
- Content formats for source control
- Version control vs source control
- Remote Git repository: branches, commits, pull requests
- TMDL view workflow (script → preview → apply → back up)
- Quick decision tables

## Separate the report from the semantic model

"Get data" in Power BI Desktop produces one of two outcomes: a **live connection** to an already-published model, or development of a **new model** (Import, DirectQuery, or Composite). The separation decision concerns the new-model case.

**Single-file solution** (model + report in one .pbix) works when there is only ever a single report on the model, usually built by one person (Personal BI, ad hoc, or role-scoped reports).

**Separate files** (one model file + multiple live-connected report files) when:
- Data modelers and report authors are different people.
- One model will be the source for multiple reports, now or in future.

Workflow for separation: a modeler can still use Desktop's report canvas to test/validate the model, but **just after publishing the model file, remove the accompanying report from the workspace** — and remember to remove it again every time the model is republished/overwritten.

Why it matters for ALM: a reusable model is a shared asset feeding reports, paginated reports, scorecards, Analyze in Excel, and composite models; separating it lets you version, deploy, and secure the model independently of reports.

## Preserve the model interface (safe vs breaking changes)

Model changes can break dependent report visuals and dashboard tiles, which surface as errors and erode trust. Manage changes carefully.

| Change | Safe? | Notes |
|--------|-------|-------|
| Add a table, column, hierarchy, hierarchy level, or measure | Safe | One exception: a new measure name could collide with a report-scoped measure name |
| Rename a table, column, hierarchy, hierarchy level, or measure | **Breaking** | Avoid; breaks references |
| Change a column data type | **Breaking** | Avoid |
| Change a measure expression so it returns a different data type | **Breaking** | Avoid |
| Move a measure to a different home table | **Breaking** | Breaks report-scoped measures that fully qualify the measure with its home table; do not write fully-qualified measure references |

To reduce collisions, report authors should adopt a naming convention for report-scoped measures (e.g., prefix with `_`).

Before any unavoidable breaking change:
- Use **Data lineage** view in the Power BI service (preferred — shows each related item's contact, as a mailto hyperlink), or **view related content** for the model.
- Contact the owner of each related report/dashboard so they can fix and republish, minimizing downtime.

## Content formats for source control

| Format | What it is | Use when |
|--------|------------|----------|
| **.pbix** | Binary Power BI Desktop file (model and/or report) | Simple; single shareable file; OneDrive refresh publishing; self-service users who find it simpler. Cannot diff or track in-file changes. |
| **.pbip** (Power BI Project) | Folder of human-readable metadata files for model and report | Source control; Git integration with Desktop; collaboration; programmatic/bulk edits; automation. Required to use Git integration when developing in Desktop. |
| **TMDL** | YAML-like model metadata format (vs .bim / TMSL) | Source control and Git; concise, structured, mergeable model metadata; developing with VS Code or Tabular Editor; reusing model objects (e.g., a date table) across models. |
| **PBIR** | Enhanced report metadata format inside .pbip | Source control and Git; reusing/copying visuals and pages; find-and-replace across visual config; fixing broken visuals; CI/CD annotations. Check its considerations/limitations first. |
| **.pbit** (template) | Reusable report layout / model structure | Sharing reusable templates — **not** for managing development. |

Note: **TMDL format** (the file format) is different from the **TMDL view** (the scripting interface in Desktop). You can use the TMDL view regardless of whether the file is .pbix or .pbip.

## Version control vs source control

- **Version control**: managing changes to whole *files* in a central location (e.g., SharePoint/OneDrive file versions). You can see that a file changed and roll back to a prior file version, but you cannot see *what* changed inside it or compare versions.
- **Source control**: managing more granular changes *within* content — its code and metadata (e.g., a remote Git repository). Enables diffs, branching, and pull requests.

| Use SharePoint/OneDrive when | Use a remote Git repository when |
|------------------------------|----------------------------------|
| Content is only models/reports saved as .pbix | Content includes other item types (dataflows, notebooks) or .pbip |
| Self-service users; collaborate via Microsoft Teams | Centralized/decentralized teams; collaborate via Azure DevOps |
| Creators are inexperienced with Git | Creators are familiar with Git / DataOps |
| Single item, limited complexity | Complex/critical content expected to scale |
| .pbix files have an encrypting sensitivity label | (Note: Fabric Git integration does **not** support sensitivity labels) |

SharePoint/OneDrive practices: check files in/out to avoid conflicts; write descriptive check-in comments; limit and prune version history; back up to a separate location.

## Remote Git repository: branches, commits, pull requests

Effective use of branching, commits, pull requests, and merges is essential to benefit from source control.

- **Branches**: each creator works in isolation on their own branch scoped to a work item (feature/fix). Decide which branch creators clone (e.g., trunk-based development off main, or release branches) and which branches connect to which workspace under Git integration.
- **Commits**: batch related changes and commit with clear messages. Place .pbip files in top-level folders when a repo holds multiple models/reports. Use a `.gitignore` / `.gitattributes` file to ignore innocuous metadata churn so every committed change is meaningful.
- **Pull requests**: submit for peer review; resolve merge conflicts before merge. Decide review standards (e.g., Best Practice Analyzer rules for models), how to review report metadata, and how to resolve conflicts. A technical owner reviews and merges; a release manager approves releases.

Prerequisites for Azure Repos source control: install **Git**; use a CLI or GUI client (Visual Studio / VS Code); Basic access level, organization+project membership, and repository permissions; **Fabric Git integration** to sync the repo with a workspace. Fabric Git integration has supported-item and scenario limitations and does not support sensitivity labels — validate fit first.

Recommended local-dev pattern: author content in **Power BI Desktop**, then stage/commit/push with **Visual Studio Code**.

## TMDL view workflow (script → preview → apply → back up)

The TMDL view is a code editor in Power BI Desktop that scripts, modifies, and applies changes to semantic-model metadata. It gives full visibility/control over objects and properties — including ones with no Desktop UI (e.g., perspectives, translations, `IsAvailableInMDX`, `DetailRowsDefinition`).

Workflow:
1. **Script**: drag an object (or a whole section — measures/tables/columns) from the Data pane onto the editor, or right-click → **Script TMDL** to a new tab or clipboard. Produces a `createOrReplace` script. Multi-select with Ctrl.
2. **Edit**: use semantic highlighting, autocomplete (Ctrl+Space), tooltips, code actions, formatting (Shift+Alt+F), and error diagnostics (Problems pane).
3. **Preview**: the **Preview** button shows a full before/after model TMDL diff (red = removed/changed, green = new); use **Update Preview** after edits. The diff is a comprehensive model comparison, so property/object ordering may differ from the tab.
4. **Apply**: the **Apply** button executes the script against the model. Success/failure shows in a banner; failure details are in the Output pane.

Key constraints and uses:
- TMDL view changes **metadata only** — it does not refresh data. Altering a Power Query or calculated-column expression requires a manual refresh. Renaming a field can break report visuals.
- **Object renaming** requires scripting the parent (rename a column → script the table; rename a table → script the whole model). Bulk rename via find-and-replace (regex supported), e.g., remove `dim_`/`fact_` prefixes across 100+ tables, or lowercase all columns.
- `column name` differs from `sourceColumn`; sync between the model and Power Query relies on `sourceColumn`.
- **Reuse**: script a table (with columns, Power Query expression, sort-by) and apply it in another model. Share reusable scripts (calendar tables, time-intelligence calculation groups) via a central SharePoint site.
- **Back up / roll back**: script the model (or a part), make changes elsewhere, and re-run the saved script to restore.
- **PBIP**: each TMDL-view tab is saved as a `.tmdl` file in a `\TMDLScripts` folder; tabs are saved in the report file. Setting up Git integration *from the workspace* won't include TMDL-view scripts saved in a published model.
- **Compatibility level**: TMDL view lets you add objects/properties above the current compatibility level and prompts to upgrade when applying.

Valid TMDL example (perspective — used for personalize visuals / composite models, no Desktop UI):

```tmdl
createOrReplace
      perspective SalesView
        perspectiveTable Sales
            perspectiveMeasure 'Sales Amount'
            perspectiveMeasure 'Sales Qty'
            perspectiveColumn Quantity
            perspectiveColumn 'Amount'
```

## Quick decision tables

**Which file format?**

| Goal | Format |
|------|--------|
| Source control + Git integration with Desktop | .pbip |
| Diffable, mergeable model metadata | TMDL |
| Diffable, reusable report metadata | PBIR |
| Simplest single shareable file / OneDrive refresh | .pbix |
| Reusable layout/structure template | .pbit |

**Which version-control tool?**

| Scenario | Tool |
|----------|------|
| .pbix only, self-service, simple | SharePoint / OneDrive |
| Complex/critical, collaborative, multiple item types | Remote Git repository (Azure Repos / GitHub) |

**Before changing a published model**

| Step | Action |
|------|--------|
| 1 | Prefer add-only changes |
| 2 | If breaking, open Data lineage / view related content |
| 3 | Notify each related report/dashboard owner |
| 4 | (Optional) Back up the model via a TMDL-view script before applying |
