# Endorsement Checklist (Promoted / Certified)

Sources: `collaborate-share/service-endorsement-overview.md`, `service-endorse-content.md`. Endorsement makes quality content easier to find: endorsed items are labeled with badges, given priority in search, and sortable in lists - in Power BI and in places like Excel.

## Promotion vs Certification

| | **Promoted** | **Certified** |
|---|---|---|
| Meaning | "I think this is valuable and ready for others to use." | "Meets org quality standards; authoritative and reliable org-wide." |
| Who can apply it | Any content owner or anyone with **write permission** on the workspace | Only **authorized reviewers** defined by the Power BI admin |
| Availability | Always available | Only if a Power BI admin has **enabled and configured** certification |
| If you can't apply it | n/a | **Request certification** per org guidelines |
| Shown in report/app header | Promotion: in header **drop-down** only | Certification: in header **and** drop-down, including who certified |

**Endorsable content types:** semantic models, dataflows, reports, apps.

## What to endorse

- Sharing data with a **broad audience** is best done via an **app** - so **endorse the app** so people can find it.
- If you also share **reports** directly, endorse the report.
- If the underlying **semantic models** (or **dataflows**) are clean and reusable, endorse them too.

## Pre-endorsement quality gate

Confirm before promoting, and especially before certifying:

```
Pre-endorsement check:
- [ ] You have write permission on the workspace holding the content.
- [ ] Data is correct, refresh succeeds, and the last refresh is recent (verify in lineage view).
- [ ] (Semantic model/dataflow) It has an informative description - it's what users see in the
       semantic models hub tooltip and details page.
- [ ] Naming is clear and the content is ready for reuse, not a work-in-progress.
- [ ] For certification: content meets the organization's documented certification standards.
```

## Promote content

1. Get **write permission** on the workspace where the content lives.
2. Open the content's **Settings** (see "How to reach content settings" below).
3. Expand the **endorsement** section and select **Promoted**.
4. (Semantic models) If a **Make discoverable** checkbox appears, optionally enable it so users without access can find and request the model. Ensure the model has a good description.
5. Select **Apply**.

## Certify content (authorized reviewers only)

1. Get **write permission** on the workspace where the content lives.
2. Carefully review the content against your organization's certification standards.
3. Open the content's **Settings** and expand the **endorsement** section.
4. Select **Certified** (this option is greyed out for non-authorized users).
5. (Semantic models) Optionally set **Make discoverable**; ensure a good description.
6. Select **Apply**.

## Request certification (if you're not an authorized reviewer)

1. Open the content's **Settings** and expand the **endorsement** section.
2. The **Certified** button is greyed out; click the link explaining how to get content certified and follow your org's process.
   - If the link redirects back with no info, your Power BI admin hasn't published guidance - contact the admin directly.

## How to reach content settings

- **Semantic models / Dataflows / Reports**: in list view, hover the item > **More options (...)** > **Settings**. (Open report: **File > Settings**.)
- **Apps**: go to the app workspace > **More options (...)** on the menu bar > **Endorse this app**.

## Notes and limits

- Endorsement badges/icons appear in lists, cards, report/app headers, lineage view (semantic models/dataflows show certified/promoted state), and external surfaces like Excel.
- **Discoverability** (the **Make discoverable** option) applies to **semantic models** and only when semantic model discoverability is enabled in your organization.
- Endorsement is a discovery/trust signal - it does not change access permissions. Combine with apps/sharing/roles for actual distribution.
