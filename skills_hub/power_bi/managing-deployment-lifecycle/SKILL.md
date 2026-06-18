---
name: managing-deployment-lifecycle
description: Manages application lifecycle management (ALM) for Power BI / Microsoft Fabric content — promoting semantic models and reports through development, test, and production workspaces with Fabric deployment pipelines (and deployment rules), Git integration, OneDrive refresh, or Azure Pipelines; running the six content-lifecycle stages (plan, develop, validate, deploy, support, retire); separating reports from semantic models and using source-control-friendly formats (.pbip, TMDL, PBIR); and planning a high-level migration to Power BI. Use when promoting content dev→test→prod, setting up deployment pipelines or deployment rules, choosing a publish/deploy method, putting Power BI content under source control, or planning a migration from a legacy BI platform (such as SSRS).
---

# Managing the Power BI Deployment Lifecycle

## Overview

Promoting Power BI / Fabric content from development to production reliably is the **Operate** stage of ALM. The core principle from the docs: **use separate environments (workspaces) and never publish directly from a local machine to production.** Content moves through six lifecycle stages — plan → develop → validate → deploy → support → retire — and you promote it across dev/test/prod workspaces with a chosen deployment mechanism. Source-control-friendly formats (.pbip with TMDL and PBIR) make the develop/validate stages auditable and reversible.

This skill grounds every recommendation in the Microsoft content-lifecycle-management, migration, and TMDL-view docs. It depends on `publishing-and-sharing-content` (workspaces, apps, endorsement), `optimizing-model-performance` (separating report from model), and `administering-and-governing` (tenant settings, capacity, gateways that gate which features are available). It cross-references the Kimball `program-management` skill for the deployment/testing/phased-rollout framing.

## When to Use

- Promoting content from a **development** workspace to **test**, and from test to **production**.
- Setting up or choosing a deployment mechanism: **Fabric deployment pipelines** (with deployment rules), **Fabric Git integration**, **Azure Pipelines**, **OneDrive refresh**, **Power BI Desktop publish**, or **third-party / XMLA** publish.
- Deciding how many workspaces (stages) you need and how report and model workspaces link across them.
- Putting models/reports under **source control** and choosing a format: .pbix vs **.pbip**, .bim (TMSL) vs **TMDL**, default report vs **PBIR**.
- Using the **TMDL view** in Power BI Desktop to script, preview, apply, and back up model metadata changes.
- Planning a **high-level migration** to Power BI from a legacy BI platform (for example, SSRS).

## The two-part mental model

Lifecycle work has two layers — keep them distinct:

1. **Stages** (what state the content is in): plan → develop → validate → deploy → support → retire. These are *activities*. Use [content-lifecycle-stage-checklist.md](content-lifecycle-stage-checklist.md) to run each stage.
2. **Environments** (where the content lives): local → development workspace → test workspace → production workspace. You *deploy* content between environments.

A single deployment (for example, dev→test) executes only after the validate stage passes for those changes. You iterate stages 2–3 (develop ↔ validate) repeatedly before each promotion.

## Choose how many workspaces (environments)

Match environment count to how critical the content is (docs recommend **separate workspaces**, never local→prod):

| Approach | Workspaces | Use when |
|----------|-----------|----------|
| **Test + production** | 2 | Simpler, non-critical content; limited collaboration; local development only |
| **Development + test + production** | 3+ | Business-critical content; collaboration; a dev workspace holds the latest integrated version |
| **Private workspace + Git** | per-developer + dev/test/prod | Each developer gets an isolated workspace synced to their own branch; merge to main, then dev→test→prod |

You can use **up to ten** environments with deployment pipelines (for example, a pre-production stage for performance testing). Consider **separate gateways/capacities per environment**, and **separate apps** for test and production — apps cannot be copied or deployed between workspaces; any app change is made manually per workspace.

## Deployment-pipeline workflow (dev → test → prod)

Fabric **deployment pipelines** are the straightforward, built-in promotion tool. Use them when content is on **PPU, Premium, or Fabric capacity** and the item types are supported.

```
Deployment pipeline run:
- [ ] 1. Assign one workspace to each stage (a pipeline admin maps dev / test / prod).
- [ ] 2. Publish content into the FIRST stage only (dev) — via Desktop, Git, or APIs.
- [ ] 3. Promote dev → test: deploy all content, or use SELECTIVE deployment for ready items.
- [ ] 4. Configure DEPLOYMENT RULES so each stage points at its own config
       (e.g., test model → test database; prod model → prod database).
- [ ] 5. Validate in test (and update the test app — app changes are manual).
- [ ] 6. Promote test → prod after release approval. Deploy in ONE direction only.
- [ ] 7. Review deployment history; check item lineage for broken auto-binding.
```

Key facts (do not invent beyond these):
- **Deployment rules** set different configurations per stage — most commonly repointing a semantic model's data source (dev database vs prod database) so promotion doesn't carry dev connections into prod.
- **Selective deployment** promotes only chosen items; **backwards deployment** copies from a later stage to an earlier one when changes exist downstream but not upstream.
- **Deploy in a single direction** (dev→test→prod). Avoid changing content in a later stage before it has been validated upstream.
- When models and reports live in **separate workspaces**, use **separate deployment pipelines** linked by **auto-binding** (keeps a report bound to the model in the same stage). You can disable auto-binding if your pattern differs.
- Triggers can be **manual** or **programmatic** via the **Power BI REST APIs** (`pipelines/selective-deploy`); REST triggering lets Azure Pipelines orchestrate deployment pipelines.
- **Compare stages** in the change-review window to see differences before deploying — especially useful when you don't use a Git repo.
- Restrict pipeline access to **release managers / technical owners**; review deployment history to catch unapproved or failed deployments.

## Choose a publish / deploy mechanism

Two distinct actions: **publish** (local → dev workspace) and **promote** (workspace → workspace). Pick per workflow:

| Mechanism | What it does | Choose when |
|-----------|--------------|-------------|
| **Power BI Desktop publish** | Manual publish of model/report to a workspace | Simplest; manual control; not automatable; content is only models/reports |
| **Third-party / XMLA** (e.g., Tabular Editor) | Publish a model via the workspace XMLA read/write endpoint | Model metadata workflows (TMDL/.bim); PPU/Premium/Fabric capacity; semantic models only |
| **OneDrive refresh** | Auto-publishes .pbix saved to OneDrive/SharePoint hourly | Self-service; version control via OneDrive/SharePoint; models/reports as .pbix; **hard to reverse** |
| **Fabric Git integration** | Syncs a remote-repo branch to a workspace | Using Git/Azure DevOps; **.pbip** files; Fabric capacity; supported items; **no sensitivity labels** |
| **Azure Pipelines** | Programmatic CI/CD via Fabric/Power BI REST APIs or XMLA | Centralized DevOps orchestration; most control; most setup effort |

Promotion options between workspaces: **Fabric deployment pipelines**, **Azure Pipelines** (orchestrating deployment pipelines, or deploying directly via REST/XMLA), or **Fabric Git integration** (sync different branches to different workspaces — main↔production; promote by merging dev→test, test→main via pull requests). Approaches can be combined (e.g., Azure Pipeline to deploy to dev with automated testing, then a deployment pipeline for dev→test→prod).

## ALM best practices (apply these every time)

1. **Separate the report from the semantic model.** Develop them as separate Power BI Desktop files (model file + live-connected report files) when modelers and report authors differ, or when one model will feed multiple reports. After publishing a model file, remove its accompanying report from the workspace. See [alm-and-source-control.md](alm-and-source-control.md).
2. **Preserve the model interface.** Adding tables/columns/measures is safe; **avoid breaking changes** — renaming tables/columns/hierarchies/measures, changing a column data type, changing a measure's return type, or moving a measure to a different home table — because they break dependent report visuals and dashboard tiles. Before a breaking change, use **Data lineage** view or *view related content* to find and notify affected report owners.
3. **Use source-control-friendly formats.** Prefer **.pbip** over .pbix (text, human-readable, diffable, required for Git integration with Desktop). Prefer **TMDL** over .bim for model metadata; use **PBIR** for reports. Reserve .pbit for reusable templates, not for managing development.
4. **Pick the right version-control tool.** SharePoint/OneDrive for simple, small, .pbix-only scenarios; a **remote Git repository** (Azure Repos or GitHub) for complex/critical/collaborative content (branches, commits, pull requests, rollback).
5. **Validate before you deploy** — functionality, data accuracy, performance, security, effectiveness, accessibility. Content creators validate in dev; users validate in test (UAT). Never deploy to prod before validation.
6. **Plan post-deployment activities** — set data-source credentials, configure/trigger scheduled refresh, update the Power BI app. Automate via the REST APIs where possible.

Detailed source-control, branching, TMDL-view, and report/model-separation guidance: [alm-and-source-control.md](alm-and-source-control.md).

## TMDL view for model ALM (Power BI Desktop)

The **TMDL view** is a code editor in Power BI Desktop for scripting model metadata — distinct from the TMDL *format* (the file format). It supports ALM directly:
- **Script** an object by dragging it from the Data pane, or right-click → **Script TMDL** (produces a `createOrReplace` script).
- **Preview** shows a before/after TMDL diff (red = removed/changed, green = new) so you can assess a pasted script before running it; **Apply** executes it against the model.
- **Reuse** scripts across models (e.g., share a calendar table or time-intelligence calculation group via a central SharePoint site) — supports the report/model-reuse goal.
- **Back up / roll back**: script the model (or a part) before a significant change, then re-run the saved script to restore.
- TMDL view changes **metadata only** — it does not refresh data; renaming a field can break report visuals. Saving as **PBIP** stores each tab as a `.tmdl` file in a `\TMDLScripts` folder for source control.

Valid TMDL example (a perspective, which has no Desktop UI — needed for personalize visuals / composite models):

```tmdl
createOrReplace
      perspective SalesView
        perspectiveTable Sales
            perspectiveMeasure 'Sales Amount'
            perspectiveMeasure 'Sales Qty'
            perspectiveColumn Quantity
            perspectiveColumn 'Amount'
```

## Migrating to Power BI (high level)

A formal migration runs as **Phase 3** of a four-phase rollout (set up & evaluate → build quick wins → migrate legacy assets → adopt/govern/monitor) and proceeds in five stages: pre-migration → (1) gather requirements & prioritize → (2) plan deployment → (3) proof of concept → (4) create & validate content → (5) deploy, support & monitor. Apply these doc-grounded principles:
- Secure an **executive sponsor**; prioritize **quick wins** with tangible value.
- **Modernize, don't photocopy** — focus on the business question a legacy report answers; refactor, consolidate, and eliminate unused items rather than reproducing layouts exactly.
- In Stage 2, decide the **product** (Power BI service vs Power BI Report Server), **workspace management** (separate dev/test/prod; separate model and report workspaces), **data acquisition** (storage mode, shared models, gateways), and **where original .pbix/.pbip files are stored** (protected, version-controlled location).
- Use the Power BI service (not file shares) for security, distribution, collaboration, and auditing.

This migration deployment thinking aligns with the Kimball `program-management` deployment stage: honest readiness assessment, pre-deployment testing, tiered support, and **phased rollout** (pilot → controlled expansion → general availability).

## Common pitfalls

| Pitfall | Why it hurts | Correction |
|---------|--------------|------------|
| Publishing local → production directly | Preventable disruptions and errors in prod | Publish only to dev (or a private workspace); promote via pipeline/Git/Azure Pipelines |
| Editing content in a later stage (e.g., prod) | Bypasses validation; later stage diverges from source | Deploy in one direction dev→test→prod; fix upstream and re-promote |
| Promoting without deployment rules | Dev data-source connections leak into prod | Set deployment rules so each stage points at its own database/config |
| Tracking .pbix in source control for diffing | .pbix is binary — changes aren't visible | Use .pbip with TMDL (model) and PBIR (report) |
| Breaking the model interface (rename / retype / move measure) | Breaks dependent report visuals and dashboard tiles; erodes trust | Add-only when possible; check Data lineage and notify report owners before breaking changes |
| Forgetting to update the app after deploying to test/prod | Report/dashboard changes aren't visible to users; apps don't auto-deploy | Manually update the app per workspace as a post-deployment step |
| Relying on Git integration for content with sensitivity labels | Git integration doesn't support sensitivity labels | Use OneDrive/SharePoint version control, or plan label handling separately |
| Reproducing legacy reports verbatim during migration | Carries over manual prep and single-report business rules | Modernize: answer the business question, refactor and consolidate |

## Success criteria

- Separate **dev/test/prod** (or at least test/prod) workspaces exist; nothing is published local→prod.
- Content is promoted in **one direction** through a chosen mechanism (deployment pipeline / Git integration / Azure Pipelines), with **deployment rules** repointing data sources per stage.
- Reports are **separated from semantic models**; the model interface is preserved (no unannounced breaking changes; lineage checked first).
- Models/reports are stored as **.pbip** with **TMDL/PBIR** under appropriate version control (Git for critical content); TMDL view is used to script, preview, and back up model changes.
- Every promotion is preceded by **validation** (creator + user/UAT) and followed by **post-deployment activities** (credentials, refresh, app update).
- Releases are gated by **release approval**; deployment history is reviewed; access is limited to release managers/technical owners.
- Migrations follow the five-stage process with an executive sponsor, quick wins, modernization, and a phased rollout.

---
*Provenance: built per SKILL 21 "Managing Deployment Lifecycle" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI implementation-planning content-lifecycle-management docs (overview, plan-design, develop-manage, validate, deploy), the report-separate-from-model guidance, the migration docs (overview, planning), and the TMDL-view doc; cross-referenced with Kimball `program-management` and the sibling skills `publishing-and-sharing-content`, `optimizing-model-performance`, and `administering-and-governing`.*
