---
name: publishing-and-sharing-content
description: Distributes Power BI content through the Power BI service - workspaces and roles (Admin/Member/Contributor/Viewer), apps with audiences, direct sharing, email subscriptions and Power Automate exports, endorsement (Promoted/Certified), and integration with Microsoft Teams, Excel, and PowerPoint, plus data lineage. Use when publishing a report from Power BI Desktop to the service, creating or assigning workspace roles, choosing between an app and direct sharing, setting up subscriptions or scheduled exports, certifying or promoting content, embedding reports in Teams, building refreshable Excel workbooks, or tracing data lineage and impact.
---

# Publishing and Sharing Content in the Power BI Service

## Purpose

After a report is built, it must reach its audience through the Power BI service (app.powerbi.com). This skill covers the distribution layer: where content lives (**workspaces**), who can do what (**roles**), how to broadcast it (**apps**, **direct sharing**, **subscriptions**, **exports**, **endorsement**), how to surface it inside Microsoft 365 (**Teams**, **Excel**, **PowerPoint**), and how to trace dependencies (**data lineage**).

**Prerequisites (from the full-lifecycle plan):** This is the Phase 6 "Report & distribute" skill (SKILL 17). It assumes a finished report from `building-interactive-reports`. It feeds `securing-row-level-access` (SKILL 18 - use the Viewer role to enforce RLS), `administering-and-governing` (SKILL 20), and `managing-deployment-lifecycle` (SKILL 21 - apps and workspaces are what deployment pipelines promote).

## When to use

Trigger this skill when the task involves any of:
- Publishing from Power BI Desktop to a service workspace, or creating/configuring a workspace.
- Assigning workspace roles or deciding which role someone needs.
- Choosing a distribution method (direct share vs app vs subscription).
- Publishing or updating an **app**, including multiple **audiences**.
- Setting up email **subscriptions** or automated **exports** (Power Automate).
- **Promoting** or **certifying** content.
- Putting reports in **Teams**, building refreshable **Excel** workbooks (Analyze in Excel), or live **PowerPoint** slides.
- Reading **data lineage** / impact analysis.

## Core concepts

**Workspace** = the collaboration container holding semantic models, reports, dashboards, paginated reports, and dataflows. Created in the service (browser), not in Desktop - Desktop only *publishes* to an existing workspace. A workspace can have up to 1,000 semantic models and reports, and one app.

**Role** = what a person can do in a workspace: **Admin**, **Member**, **Contributor**, **Viewer** (assignable to individuals, security groups, Microsoft 365 groups, or distribution lists). Highest role wins if a user is in multiple groups. Full grid: see [workspace-role-permission-matrix.md](workspace-role-permission-matrix.md).

**App** = packaged, read-only content published from a workspace to a broad audience, with built-in navigation and up to 25 **audience groups** (show/hide content per group). One app per workspace.

**Endorsement** = trust signal on content: **Promoted** (any writer says "good to use") or **Certified** (authorized reviewers say "org-standard, authoritative"). Endorsable types: semantic models, dataflows, reports, apps.

## The four distribution methods

| Method | Audience | What recipient gets | Key gate |
|---|---|---|---|
| **Direct share** | A few named people/groups | Live access to the item | Sharing permission + recipient license/Premium |
| **App** | Broad / whole org | Curated, navigable, read-only collection | Pro/PPU to publish; per-audience access |
| **Subscription** | Self or others on a schedule | Emailed snapshot or full PDF/PPTX attachment | Paid license to subscribe others |
| **Export (Power Automate)** | Automated/programmatic delivery | Scheduled file (PDF, PPTX, PNG, etc.) by email | Reserved/Premium capacity workspace |

Decision logic, license rules, and edge cases: see [distribution-method-decision-guide.md](distribution-method-decision-guide.md).

## Publishing workflow

Copy this checklist and work top-down:

```
Publish & distribute:
- [ ] Step 1 - Workspace ready? Create one in the service if needed.
- [ ] Step 2 - Assign roles (least privilege; Viewer for consumers).
- [ ] Step 3 - Publish report from Desktop to the workspace.
- [ ] Step 4 - Pick distribution method (share / app / subscription / export).
- [ ] Step 5 - Endorse the content (Promote, or request Certification).
- [ ] Step 6 - Surface it where users work (Teams / Excel / PowerPoint) if needed.
- [ ] Step 7 - Verify via lineage that dependencies and refresh are intact.
```

**Step 1 - Create the workspace.** In the service: **Workspaces** > **New workspace** (labels can vary: **Create** > **Workspaces**, or **+ New** > **Workspace**). Name it uniquely, set optional image, **Contact list**, **Workspace OneDrive**, **Allow contributors to update the app**, and **Premium** capacity. Free licenses can't create workspaces beyond My workspace.

**Step 2 - Assign roles.** Only a workspace **Admin** can manage access. **Manage access** > **+ Add people or groups** > assign Admin/Member/Contributor/Viewer. Prefer groups over individuals. Use the **Viewer** role for read-only consumers (and to enforce RLS - see `securing-row-level-access`). Note: **Members can add lower-permission users but cannot change existing users' roles** - an Admin must remove then re-add to change a role.

**Step 3 - Publish.** From Power BI Desktop, publish to the target workspace. Publishing enforces licensing: without Pro/PPU you get "Only users with Power BI Pro licenses can publish to this workspace."

**Step 4 - Distribute.** Use the decision guide. For broad audiences, the best practice is an **app**:
1. Workspace list view > **Create app**.
2. **Setup** tab: name, description, theme, support site, contact; optionally **Allow users to make a copy of the reports**, **Access to hidden content**.
3. **Content** tab: **Add content** from the workspace (and optional links). Newly added content on *update* is hidden by default - unhide per audience.
4. **Audience** tab: **New Audience**, show/hide items per audience, grant access (specific users/groups or **Entire organization**), and per-audience **Advanced** options for share/build on semantic models.
5. **Publish app** - copy the shareable link. To change later, edit the workspace (staging) then **Update app**.

**Step 5 - Endorse.** Promote content you're confident in; request certification per org guidelines. Steps and a quality gate: see [endorsement-checklist.md](endorsement-checklist.md).

**Step 6 - Surface in Microsoft 365.** See the Integrations section below.

**Step 7 - Verify lineage.** Open **lineage view** (workspace list > **View** arrow > **Lineage**) to confirm data sources, gateways, refresh times, and downstream reports. Requires Admin/Member/Contributor (Viewer can't open lineage) and a Pro license.

## Sharing, subscriptions, and exports

**Direct share** gives named recipients live access. Reshare and semantic-model sharing depend on role and on app settings ("Allow users to share the semantic models in this app").

**Email subscriptions** (`Subscribe` on the report/dashboard menu): emailed snapshot + link, or a full report attachment (**PDF** or **PowerPoint**) when the workspace is on **Premium/PPU** (max 20 pages, < 25 MB). Up to 24 subscriptions per report/dashboard. **Repeat**: Hourly, Daily, Weekly, Monthly, or **After data refresh** (once daily). **Subscribing others requires a paid (Pro/PPU) license** and a Contributor/Member/Admin role; B2B guests can only subscribe themselves. Use **Run now** to test (it does not refresh data).

> [!CAUTION]
> Sensitivity labels are applied to subscription **attachments** but NOT to the email text or preview image. For RLS reports, the static preview image reflects the **subscription owner's** data, not the recipient's - turn off **Preview image** to avoid leaking restricted data.

**Automated exports** use **Power Automate** ("**Export To File for Power BI Reports**" action) on a scheduled cloud flow, then "Send an email (V2)". Requires a workspace on a reserved/Premium capacity. RLS can be applied by passing **Identities Username**, **Semantic model** (ID), and **Roles** to the action.

## Integrations (Microsoft 365)

- **Teams**: Add the Power BI app to Teams, embed interactive reports in a **Power BI tab**, paste link previews, chat a filtered view, get activity-feed notifications. Embedding does NOT grant access - you must still share the report. Dashboards can't be embedded in the Power BI tab (paginated reports/.rdl, scorecards, and org apps can be).
- **Excel - Analyze in Excel**: **Export** > **Analyze in Excel** (or item **More options** > **Analyze in Excel**) builds a workbook connected to the whole semantic model for PivotTables. **Export with live connection** exports a refreshable table from one visual (500,000-row max). The Power BI **Excel add-in** inserts connected PivotTables/tables from within Excel. Requires **Build** permission or **Contributor** role; RLS and OLS are enforced.
- **PowerPoint add-in**: Adds **live, interactive** report pages or individual visuals to slides (not dashboards or paginated reports). Data stays in Power BI and respects permissions and RLS; no data is exported.

## Common pitfalls

| Pitfall | Symptom | Correction |
|---|---|---|
| Embedding/linking a report instead of granting access | Recipients see "Content isn't available" | Embedding/links don't grant access; share the report (or use a group) |
| Member tries to change a user's role | Role change appears impossible | Members can't edit roles; an Admin must remove then re-add the user |
| Updating an app and expecting new items to appear | New content invisible to audiences | On update, new content is hidden by default; unhide per audience group |
| Subscription preview leaks RLS data | Recipients see another user's filtered data | Turn off **Preview image**; rely on the attachment (which respects RLS) |
| Subscribing others on a non-Premium workspace | Recipients can't receive it | Subscribing others requires recipients to have Pro/PPU unless content is in Premium capacity |
| Giving consumers Contributor "to be safe" | Over-permissioned users can edit/delete content | Use **Viewer** for read-only consumers (least privilege; also needed for RLS) |
| Expecting certification to be self-service | "Certified" option greyed out | Only admin-authorized reviewers can certify; others **request** certification |
| Free user can't create a workspace | No **Create**/**New workspace** option | Workspace creation needs Pro/PPU or Premium capacity, and tenant permission |

## Success criteria

- Content is in a named service workspace (not My workspace) with roles assigned by least privilege, preferably via groups.
- The distribution method matches the audience size and freshness need (app for broad, share for few, subscription/export for scheduled push).
- Apps expose only the intended items per audience; new items are explicitly unhidden after updates.
- Endorsement reflects real trust (Promoted = vetted; Certified = org-authoritative via authorized reviewer).
- RLS-protected content is never leaked through subscription preview images.
- Lineage view confirms sources, gateways, refresh, and downstream dependencies before announcing availability.

## Reference files

- **Workspace-role permission matrix** (Admin/Member/Contributor/Viewer, capability by capability, plus licensing): [workspace-role-permission-matrix.md](workspace-role-permission-matrix.md)
- **Distribution-method decision guide** (share vs app vs subscription vs export, with license/capacity rules): [distribution-method-decision-guide.md](distribution-method-decision-guide.md)
- **Endorsement checklist** (Promoted/Certified steps, who can do it, pre-endorsement quality gate): [endorsement-checklist.md](endorsement-checklist.md)

---
*Provenance: built per SKILL 17 "Publishing and Sharing Content" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; sourced from the Power BI `collaborate-share` docs (service-new-workspaces, service-roles-new-workspaces, service-give-access-new-workspaces, service-create-the-new-workspaces, service-create-distribute-apps, end-user-subscribe, service-automate-power-bi-report-export, service-endorsement-overview, service-endorse-content, service-collaborate-microsoft-teams, service-analyze-in-excel, service-power-bi-powerpoint-add-in-about, service-data-lineage).*
