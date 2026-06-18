# Content lifecycle stage checklist

A per-stage checklist for managing the lifecycle of Power BI / Fabric content. Run the relevant stage(s); stages 2–3 iterate. Grounded in the Microsoft content-lifecycle-management series (overview, plan-design, develop-manage, validate, deploy) and the support/retire stages described in the overview.

## Contents
- The six stages (at a glance)
- Stage 1 — Plan and design
- Stage 2 — Develop and manage changes
- Stage 3 — Validate
- Stage 4 — Deploy
- Stage 5 — Support and monitor
- Stage 6 — Retire and archive
- Simple vs advanced approach (component map)

## The six stages (at a glance)

| # | Stage | Goal | Typical environment |
|---|-------|------|---------------------|
| 1 | Plan and design | Gather requirements; make lifecycle decisions | (none / design docs) |
| 2 | Develop and manage changes | Build content; version/source control | Local + development workspace |
| 3 | Validate | Test quality before deployment | Development (creator) → test (user/UAT) |
| 4 | Deploy | Publish and promote across environments | dev → test → production |
| 5 | Support and monitor | Address issues; monitor reliability | Production |
| 6 | Retire and archive | Remove and archive unused content | Production → archive |

Stages 2 and 3 iterate in successive develop ↔ validate cycles before each promotion. Supporting content (stage 5) often feeds back into planning new content (stage 1).

## Stage 1 — Plan and design

```
- [ ] Conduct solution planning: gather business and technical requirements.
- [ ] Identify the content format: item types and counts (semantic models, reports,
      dataflows, dashboards, scorecards, paginated reports) and expected complexity/growth.
- [ ] Identify who will create AND who will support the content; involve both in planning.
- [ ] Determine importance/criticality → drives how robust the lifecycle approach must be.
- [ ] Determine how users will consume the content (view reports, build on the model
      with build permissions, Analyze in Excel, Copilot/AI skills, paginated, composite).
- [ ] Decide whether multiple creators must collaborate → choose a collaboration hub
      (Microsoft Teams for simple; Azure DevOps or GitHub for advanced).
- [ ] Decide content file format(s): .pbix vs .pbip; .bim (TMSL) vs TMDL; default vs PBIR.
- [ ] Decide where to store files: content files, data source files, supporting files,
      templates/documentation (SharePoint/OneDrive, OneLake, or a remote Git repo).
- [ ] Plan for development and deployment (establish tools, test data-source connections).
```

Decision drivers from the docs: who creates content, whether creation is collaborative, the type/scope of content, the **licensing** in place (deployment pipelines need PPU/Premium/Fabric capacity; Fabric Git integration needs Fabric capacity), the content-delivery scope, and implementation maturity.

## Stage 2 — Develop and manage changes

```
- [ ] Choose development tools (Power BI Desktop, web authoring, Tabular Editor,
      Visual Studio Code, Power BI Report Builder).
- [ ] Choose format: prefer .pbip for source control; .pbix is simpler for self-service.
      Use TMDL for model metadata; PBIR for reports; .pbit only for reusable templates.
- [ ] Separate semantic model and report development (good practice for reuse and ALM).
- [ ] Decide how many workspaces/stages you need (test+prod, or dev+test+prod, or
      private-workspace+Git) and how report/model workspaces link across stages.
- [ ] Decide supporting resources per environment (separate gateways/capacities;
      separate apps for test and prod — apps can't be copied between workspaces).
- [ ] Decide the version-control approach: SharePoint/OneDrive (simple, .pbix only) or a
      remote Git repository (Azure Repos/GitHub) for complex/critical content.
- [ ] If using source control: set up the repo, .gitignore/.gitattributes, alerts,
      access (mirror deployment-pipeline/workspace permissions), and documentation.
- [ ] If using Git: define branching and merge strategies; batch changes into meaningful
      commits with clear messages; review/merge via pull requests.
```

## Stage 3 — Validate

Assess six aspects: **functionality, data accuracy, performance, security, effectiveness, accessibility.**

```
- [ ] Design and document test conditions and success criteria up front.
- [ ] Creator manual testing in the development workspace:
      - Models: missing/duplicate/incorrect values; DAX results and query times;
        scheduled refresh succeeds; (Blank) from referential-integrity violations;
        RLS/OLS prevents unauthorized access; objects in display folders.
      - Reports: meets requirements; right visual types; clear pages; behaves when
        filtered narrowly; export behavior; cross-report drillthrough/personalization.
      - Tools: Power BI Desktop (visual canvas, DAX query view, Performance analyzer,
        query diagnostics), Fabric (View As for RLS, Monitoring hub), DAX Studio,
        Tabular Editor, Best Practice Analyzer (BPA).
- [ ] Peer review (functional, technical, or pull-request review) before user testing.
- [ ] If justified, automate testing (BPA via Tabular Editor CLI in Azure DevOps;
      Fabric notebooks + semantic link / Great Expectations; Power Automate query checks).
      Content that fails automated tests must not be deployed.
- [ ] User validation in the TEST workspace: set up a test app (identical to prod app),
      provision test-user access and security roles, set up scheduled refresh.
      Use observatory testing, focus groups, or user acceptance testing (UAT).
- [ ] Collect feedback (e.g., Microsoft Forms) and document all results and changes.
```

## Stage 4 — Deploy

```
- [ ] Identify available options (licensing/content gate which mechanisms you can use:
      deployment pipelines, Azure DevOps, Git integration, Fabric/Power BI REST APIs,
      XMLA read/write endpoints).
- [ ] Decide how to PUBLISH (local → dev): Power BI Desktop, third-party/XMLA,
      OneDrive refresh, Fabric Git integration, or Azure Pipelines.
- [ ] Decide how to PROMOTE between workspaces: Fabric deployment pipelines,
      Azure Pipelines, or Fabric Git integration (branch-per-workspace).
- [ ] Configure deployment rules (e.g., model points at dev DB in dev, prod DB in prod).
- [ ] Plan the release strategy: a release manager approves the final review before
      deploying to test and to production (release approval gates).
- [ ] Promote in ONE direction (dev→test→prod); use selective deployment for partial
      readiness; compare stages in the change-review window.
- [ ] Plan and run post-deployment activities: set data-source credentials, configure
      and trigger scheduled refresh, update the Power BI app (manual, per workspace).
- [ ] Deploy content to production.
```

## Stage 5 — Support and monitor

```
- [ ] Address user issues and requests after publication; route to the support model.
- [ ] Monitor reliability of deployed content (default Power BI monitoring reports / the
      admin monitoring workspace, or a customized monitoring report with Azure Log
      Analytics integration).
- [ ] Review deployment history regularly to spot unapproved deployments or failures;
      check item lineage for broken auto-binding across pipelines.
- [ ] Feed support findings back into planning new or improved content (return to Stage 1).
```

(Detailed auditing/monitoring sources are owned by the `administering-and-governing` skill.)

## Stage 6 — Retire and archive

```
- [ ] Identify content consumers no longer need or use.
- [ ] Remove the content from production.
- [ ] Archive it for later reference (OneDrive/SharePoint for files, or Azure Repos for
      metadata) so it can be restored if needed.
```

## Simple vs advanced approach (component map)

The docs map example components to each stage from simple (self-service) to advanced (enterprise):

| Stage | Simple approach | Advanced approach |
|-------|-----------------|-------------------|
| Plan & design | Microsoft Teams | Azure DevOps (Boards, Wiki) |
| Develop & manage | OneDrive/SharePoint version control of .pbix | Azure Repos source control of .pbip / TMDL |
| Validate | Power BI manual testing | Azure Test Plans + automated testing |
| Deploy | Power BI deployment pipelines | Azure Pipelines (CI/CD) |
| Support & monitor | Default monitoring reports / admin monitoring workspace | Custom report + Azure Log Analytics |
| Retire & archive | OneDrive/SharePoint archive | Azure Repos metadata archive |

Choose the most lightweight approach that meets the content's criticality and collaboration needs.
