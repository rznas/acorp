# Distribution-Method Decision Guide

## Contents
- Decision tree
- Method comparison table
- Direct sharing
- Apps (broad distribution)
- Email subscriptions
- Automated exports (Power Automate)
- License and capacity rules
- Choosing across methods (worked scenarios)

Sources: `collaborate-share/service-create-distribute-apps.md`, `end-user-subscribe.md`, `service-automate-power-bi-report-export.md`, `service-roles-new-workspaces.md`, `service-endorsement-overview.md`.

## Decision tree

```
Who needs the content?
- A broad audience / whole org, curated and navigable?      -> APP  (best practice for broad sharing)
- A few specific people who need live access?               -> DIRECT SHARE
- People who want it pushed to their inbox on a schedule?   -> SUBSCRIPTION
- A system/process needing files generated automatically?   -> EXPORT via Power Automate

Refinements:
- Need different content for different groups in one place?  -> APP with multiple audiences (up to 25)
- Recipients should find it later by searching?            -> APP + endorse it (see endorsement-checklist.md)
- Need a file (PDF/PPTX) rather than live access?           -> SUBSCRIPTION (attachment) or EXPORT
- Need recipients to keep working with the numbers?         -> Excel: Analyze in Excel / live connection (see SKILL.md Integrations)
```

## Method comparison table

| Dimension | Direct share | App | Subscription | Export (Power Automate) |
|---|---|---|---|---|
| Best for | A few named users | Broad / org-wide | Scheduled inbox delivery | Automated file generation |
| Recipient experience | Live item access | Read-only curated collection w/ navigation | Emailed snapshot or attachment | Emailed file (PDF/PPTX/PNG/etc.) |
| Per-group targeting | Manual per item | Up to 25 audience groups, show/hide per group | Per subscription (up to 24/report) | Per flow |
| Who can set it up | Role with share permission | Admin/Member (Contributor if delegated) | Self anytime; others need paid license + Contributor+ | Flow author w/ capacity |
| Freshness | Live | Live (republish to update) | Snapshot at schedule (or after refresh) | Snapshot at schedule |
| RLS respected | Yes (live) | Yes (live) | Live view yes; preview image = owner's data | Yes, if identities passed |
| Capacity needs | None (license rules apply) | Pro/PPU to publish | Premium for attachment / subscribing externally | Reserved/Premium capacity |

## Direct sharing

Use for a small, known set of recipients who need live access. Resharing and sharing the underlying semantic model depend on workspace role and on app/item settings ("Allow others to reshare items"). Embedding a report in Teams or pasting a link does **not** grant access - you must still share it. Sharing to a colleague outside your domain does **not** require Premium (but **subscribing** them does).

## Apps (broad distribution)

The recommended way to reach a broad audience. One app per workspace; built-in navigation; users can't modify content.

- **Audiences**: up to **25** audience groups per app; show/hide items per group; grant each group access to specific users/groups or the **Entire organization**.
- **Limits**: 10,000 users/user-groups total per app; 1,000 per audience group; 200 dashboards max. Each workspace user/group counts per audience group (so groups x users can multiply toward the 10,000 cap - use security groups).
- **Update model**: the workspace is the staging area; edits go live only on **Update app**. On update, newly added items are **hidden by default** - unhide per audience.
- **Build/copy**: per-audience **Advanced** options allow sharing and building on the app's semantic models; "Allow users to make a copy of the reports" lets build-permitted users save their own copy.
- **Entire organization** is disabled if you chose auto-install, if you're a guest, or if the tenant setting forbids it.
- **Distribution**: users find apps in **Apps marketplace** / **AppSource**, via a direct link, or via admin-enabled **auto-install** (can't auto-install to the entire org).

## Email subscriptions

Self-service push of a snapshot/attachment on a schedule. Open the report/dashboard and select **Subscribe**.

- Up to **24** subscriptions per report/dashboard; each with its own recipients, time, and frequency.
- **Repeat**: Hourly, Daily, Weekly, Monthly, or **After data refresh** (sends once daily after the first scheduled refresh). Monthly supports specific days or **Last day of month**.
- **Attachment**: full report as **PDF** or **PowerPoint** only when the workspace is on **Premium/PPU** (max 20 pages, < 25 MB). Otherwise the email carries a snapshot + link.
- **Subscribing others** needs a **paid (Pro/PPU)** license and **Contributor/Member/Admin** role; recipients also need a paid license unless content is in Premium. **B2B guests** can subscribe only themselves.
- **Include my changes** lets the subscriber capture their own filters/slicers/bookmarks/etc. (not for dashboards or paginated reports).
- **Run now** sends a test immediately and does **not** refresh data.
- **Paginated** subscriptions differ: attachment of the entire output in PDF/PPTX/XLSX/DOCX/CSV/XML, unlimited subscriptions, parameter values per subscription, no "After data refresh", 60-minute execution limit.

> [!CAUTION]
> Sensitivity labels apply to subscription **attachments** but not to email text or preview images. For RLS content, the preview image reflects the **subscription owner's** row-level view - turn off **Preview image** to avoid leaking restricted data to recipients.

## Automated exports (Power Automate)

For programmatic, scheduled file delivery beyond what subscriptions offer.

- Build a **Scheduled cloud flow**, add the **Export To File for Power BI Reports** action (select workspace, report, export format; optionally specific pages or a report bookmark), then **Send an email (V2)** with the **File Content** as the attachment.
- Requires a workspace backed by a **reserved/Premium capacity** (A1/EM1 - A6/P3 SKUs) and standard Power Automate connectors.
- **RLS**: pass **Identities Username**, **Identities Semantic model** (the semantic model ID from its settings URL), and **Identities Roles** to apply RLS rules.
- Exports reflect the **saved default slicer values**; to change them, re-save the report or set values via API.

## License and capacity rules (summary)

- Creating/updating an **app** needs Pro/PPU. App **users** need Pro/PPU **unless** the workspace is in **Premium / F64+** capacity (then free users can view but can't copy reports or build on the models).
- Subscribing **others** or sending an **attachment** generally requires **Premium/PPU**; PPU workspaces don't count as Premium capacity for subscribing **group aliases** or **external** users.
- **External B2B** users can be subscribed only if content is in **Premium capacity**; they can also subscribe themselves (with the tenant setting enabled). **Internal guests** can't create subscriptions.
- Group aliases in subscriptions: **distribution groups**, **dynamic distribution groups**, and **mail-enabled security groups** are supported; **Microsoft 365 groups** and plain **security groups** are not.

## Choosing across methods (worked scenarios)

1. **"Roll out the Sales dashboard to all of Sales, with regional teams seeing only their region."** -> **App** with one audience per region (show/hide regional reports), or pair with RLS for a single report. Endorse the app so people can find it.
2. **"Send the exec team a weekly PDF of the company KPI report."** -> **Subscription** with **Attach full report (PDF)** on a **Weekly** schedule (needs Premium/PPU for the attachment); or a Power Automate **export** flow if the workspace is on capacity and you need finer control.
3. **"Give two analysts live access to drill into the model."** -> **Direct share** (or add them as **Contributor** in the workspace if they will edit).
4. **"Finance wants to keep working with the numbers in Excel."** -> **Analyze in Excel** / **Export with live connection** (see SKILL.md Integrations), not a subscription.
5. **"A downstream system needs the report as a file every morning."** -> **Export via Power Automate** scheduled flow.
