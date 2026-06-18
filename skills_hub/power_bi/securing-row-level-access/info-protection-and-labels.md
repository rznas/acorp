# Information Protection & Sensitivity Labels

Microsoft Purview **sensitivity labels** classify Power BI content and can apply **encryption** to files. This is an organization-wide effort far larger than Power BI — labels and label policies are created in the **Microsoft Purview compliance portal**, and Power BI behavior is then turned on via **Fabric tenant settings**.

## Contents
- What a label does (and does NOT do)
- Label structure (taxonomy)
- Encryption protection
- Inheritance (upstream and downstream)
- Label policies (default / mandatory)
- Fabric tenant settings
- Licensing
- Planning checklist

## What a label does (and does NOT do)

A label is a single clear-text tag applied to an item (e.g. a semantic model in the service) or a file (e.g. a .pbix). One label per item; when data spans classifications, apply the **most restrictive** label.

Purposes: **classification**, **user education/awareness**, and a basis for **policies/DLP**.

**Critical:** assigning a label **does not change who can access content in the Power BI service.** Standard Power BI permissions (workspace roles, app permissions, sharing) still control access. The exception is **encryption** on **exported/Desktop files**. So:

- For *access control to data*, use **RLS / OLS / permissions** — not labels.
- For *protecting content that leaves the service* (exports), use **labels with encryption**.

## Label structure (taxonomy)

- Create **3–7 labels**, forming a hierarchy from least to most sensitive (e.g. *General Internal Use* → *Restricted* → *Highly Restricted*). Labels should rarely change.
- **Sub-labels** add variation/scope within a label; use sparingly.
- Use intuitive, unambiguous, generic names that localize cleanly; avoid jargon/acronyms (use *Highly Restricted*, not *PII*).

## Encryption protection

- **Encryption** applies to files/emails (a .pbix can be encrypted); **markings** (headers/footers/watermarks) apply to Office files and are **not** shown on Power BI content.
- Encryption persists with the file even if sent externally or renamed.
- A label with encryption follows **exports** to Excel, PowerPoint, PDF, and Power BI Desktop files from the **service**; only authorized RMS users can open them.
- **Limitation:** an offline user cannot open an encrypted file — Azure RMS must verify authorization online.
- **Caveat:** exporting from **Power BI Desktop to PDF does not retain protection**; export from the **service** for maximum protection.
- Authorized RMS users are defined **within the label** (Azure RMS permissions), which is **separate** from the label policy's authorized users — keep them consistent to avoid "can't open file" support tickets.

## Inheritance (upstream and downstream)

- **Upstream:** a semantic model can inherit a label from a supported data source (e.g. an Azure SQL Database column labeled *Highly Restricted*). Requires the **Apply sensitivity labels from data sources to their data in Power BI** tenant setting.
- **Downstream:** downstream items (reports) can inherit the semantic model's label. Requires the **Automatically apply sensitivity labels to downstream content** tenant setting.
- Both promote consistency and reduce manual effort.

## Label policies (default / mandatory)

A label is unusable until included in a published **label policy** (Microsoft Purview compliance portal). Policies define which users may use which labels, plus:

- **Default label** — auto-applies to new/edited Power BI content (does not retroactively label existing content; use the information protection APIs for bulk).
- **Mandatory label** — forces users to choose a label on create/edit; prevents empty/removed labels. If you require mandatory labels, also set a default.
- Assign **groups**, not individuals, to simplify management.

## Fabric tenant settings (information protection)

Set these only **after** labels/policies are published in Purview:

- **Allow users to apply sensitivity labels for content** — who can label (typically all; workspace roles then govern who can edit/label).
- **Apply sensitivity labels from data sources to their data in Power BI** — upstream inheritance.
- **Automatically apply sensitivity labels to downstream content** — downstream inheritance.
- **Allow workspace administrators to override automatically applied sensitivity labels** — lets admins change auto-applied/inherited protected labels.
- **Restrict content with protected labels from being shared via link with everyone in your organization** — recommended on; reduces leakage of encrypted content. (Distinct from *Allow shareable links to grant access to everyone in your organization*, which governs org-wide link creation regardless of label.)
- Export-format settings — optionally disable formats that can't carry protection (.csv, .xml, .mhtml, .png) in regulated scenarios.

## Licensing

- **Microsoft Purview Information Protection** license for admins and for users who apply labels or open encrypted files (often included in Microsoft 365 E5 / E5 Compliance).
- **Power BI Pro or PPU** for users who apply/manage labels in the service or Desktop.

## Planning checklist

```
Information protection rollout:
- [ ] Define 3-7 labels (hierarchical, generic, localizable); decide sub-labels
- [ ] Decide which labels are encrypted; map RMS users/groups per encrypted label
- [ ] Write a data classification & protection policy (what each label allows)
- [ ] Create labels + publish label policy in Microsoft Purview compliance portal
- [ ] Decide default and/or mandatory label for Power BI content
- [ ] Decide upstream (data source) and downstream inheritance
- [ ] Set Fabric tenant settings (only after Purview setup): who can label, inheritance,
      admin override, restrict sharing of protected content, export formats
- [ ] Verify licensing (Purview Information Protection + Power BI Pro/PPU)
- [ ] Test on a small group: service items, .pbix files, exports, mobile/browser
- [ ] Monitor via Power BI activity log (label assign/change events) + Purview reports
```
