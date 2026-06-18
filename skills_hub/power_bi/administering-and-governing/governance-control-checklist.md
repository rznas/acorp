# Governance Control Checklist by Layer

Power BI / Fabric governance controls organized by **tenant**, **workspace**, and **content** layers. Every control area follows the same repeatable loop from the implementation-planning docs: **Review → Decide → Update → Document → Manage → Audit**. Decisions are made *with* the COE, BI strategy working team, security/compliance, and (for strategic items) the executive sponsor — never by an admin alone.

## Contents
- Layer 0: Tenant foundation (Entra, region, admins)
- Layer 1: Tenant controls (settings, domains, embed codes, organizational visuals, Azure connections, capacities, gateways)
- Layer 2: Workspace controls
- Layer 3: Content controls
- Cross-layer audit activities (activity-log operation names)
- The Review → Audit loop (reusable template)

---

## Layer 0: Tenant foundation

- [ ] Confirm the Microsoft Entra tenant is **managed** (has a global administrator); take over any unmanaged/shadow tenant.
- [ ] Identify the **home region** (default data region); use **Multi-Geo** (capacity in a specific region; not available on shared capacity or PPU) for data-residency needs, or move the tenant via a support request if the region is unsuitable.
- [ ] For external users, record the **CTID** in the tenant URL and set up **custom branding** so users know which tenant they're in.
- [ ] Identify and involve **Microsoft Entra administrators** for security groups, group ownership, service principals (for unattended admin-API scripts and embedding), external/guest users, and real-time control policies.
- [ ] Appoint **2 to 4 Fabric administrators**; use **PIM** for occasional just-in-time rights; assign backups; cross-train; audit role membership regularly (include Power Platform admin and Global admin in the review).

## Layer 1: Tenant controls

### Tenant settings
- [ ] Review every setting; record *date reviewed, setting, current value, security groups allowed, groups not allowed, delegated to other admins*.
- [ ] Decide each setting **with** the COE/stakeholders; record decision, decision-makers, approver, affected setting, pending action; note whether temporary, per-business-unit, or requires approval.
- [ ] Update via one or two designated admins; test before widespread change; follow change management; communicate to users.
- [ ] Document; automate **weekly snapshots** with the **Get Tenant Settings** REST API; provide admins/COE/auditors access; publish a **user-facing summary** (setting + which security group to request).
- [ ] Manage: watch for new settings (admin-portal messages, snapshot diffs, Power BI/Fabric blogs); handle access requests (form or help desk).
- [ ] Audit: `UpdatedAdminFeatureSwitch` activity flags a change; diff snapshots for *what*; Microsoft Graph group-membership data for *who*; optionally alert via Microsoft 365 / Defender for Cloud Apps.

### Domains
- [ ] Review existing domains, their admins/contributors, assigned workspaces, and any delegated (overridden) tenant settings.
- [ ] Decide groupings (subject area, team, business unit, project — one domain per workspace); who administers; who may assign workspaces (domain contributors must also be workspace admins).
- [ ] Update, document (purpose, approver, owner/steward, compliance), manage requests, review quarterly/biannually.
- [ ] Audit with: `InsertDataDomainAsAdmin` (new), `UpdateDataDomainAsAdmin` (changed), `UpdateDataDomainFoldersRelationsAsAdmin` (workspace assigned), `UpdateDataDomainAccessAsAdmin` (admins/contributors changed).

### Embed codes (Publish to web)
- [ ] Review each embed code's status and the *Publish to Web* tenant setting.
- [ ] Restrict the setting to a very limited set of users; never use Publish to web for internal/sensitive content (it is public and search-discoverable) — use API embedding or secure no-code embed instead.
- [ ] Verify no sensitive data is exposed; relocate to a clearly public workspace; confirm sensitivity label indicates a public audience.
- [ ] Audit with `PublishToWebReport` (new code) and `UpdatedAdminFeatureSwitch` (item `PublishToWeb`).

### Organizational visuals
- [ ] Review the org-store repository and the *Power BI visuals* tenant settings.
- [ ] Decide whether custom visuals are permitted (data-leakage risk: visuals can query semantic-model data and call web services), how they're validated/approved, who may use them, whether certified-only, and whether centrally managed.
- [ ] Register approved visuals (AppSource or `.pbiviz`) for consistent versions; remember custom-visual tenant settings apply only to **published** reports — manage Desktop visuals via **Group Policy**.
- [ ] Audit with `InsertOrganizationalGalleryItem`, `UpdateOrganizationalGalleryItem`, and `UpdatedAdminFeatureSwitch` (items `CustomVisualsTenant`, `CertifiedCustomVisualsTenant`).

### Azure connections
- [ ] Review tenant-level storage, workspace-level storage permissions, and the *Azure Log Analytics connections for workspace administrators* setting.
- [ ] Decide BYOL (dataflows Gen1 storage), semantic-model backup/restore, and Log Analytics integration needs; centralized vs decentralized storage; OneLake vs Azure connections going forward.
- [ ] Audit with `AddLinkToExternalResource` / `DeleteLinkToExternalResource` (ResourceType = storage or Log Analytics), `AddExternalResource`, and `UpdatedAdminFeatureSwitch` (items `storageAccountAttachForWorkspaceAdminsEnabled`, `LogAnalyticsAttachForWorkspaceAdmins`).

### Capacities and gateways (tenant-level)
- [ ] Right-size capacities (one tenant, possibly multiple capacities per workload/business unit); monitor Autoscale cost-effectiveness; consider chargebacks via activity-log operations mapped to business units.
- [ ] Plan gateways (standard / personal / VNet); restrict installers; use clusters for prod; separate clusters per environment/region; store recovery keys securely; test updates on dev/test first; manage gateway/connection roles with security groups and least privilege. (Full detail in SKILL.md "Capacities and data gateways".)

## Layer 2: Workspace controls

- [ ] Review all workspaces, their role assignments and settings, and the *Create workspaces* tenant setting.
- [ ] Build a workspace inventory: admin-portal export, **scanner (metadata scanning) APIs** (incremental — large tenants), or **Get Groups As Admin** / **Get-PowerBIWorkspace** (full pull — small tenants).
- [ ] Decide who may create workspaces, the creation process, **naming conventions**, organization (by subject/scope), appropriate use of personal workspaces, and who gets access (limit to creators/managers — see `securing-row-level-access` for consumer access).
- [ ] Assign roles per least privilege: **Admin / Member / Contributor / Viewer** (covered in `publishing-and-sharing-content`).
- [ ] Use admin powers when needed: manage roles, recover deleted workspaces (within retention), reassign license mode (Pro/PPU/capacity), 24-hour temporary personal-workspace access, rename to enforce conventions. The role does not auto-grant data access, but admins can grant themselves workspace roles.
- [ ] Document supplementary info (purpose, approver, owner, compliance, sensitivity, lifecycle).
- [ ] Audit with: `CreateFolder` (new), `UpdateFolder` (changed), `UpdateWorkspaceAccess` / `UpdateFolderAccess` (access changed), `MigrateWorkspaceIntoCapacity` (reassigned), `AddAdminPersonalWorkspaceAccess` (admin accessed a personal workspace), `UpdatedAdminFeatureSwitch` item `CreateAppWorkspaces` (create-workspaces setting changed).

## Layer 3: Content controls

- [ ] **Endorsement**: define who may **Certify** (and requirements); encourage **Promoted/Certified** based on usage; track % endorsed and high-view non-certified reports.
- [ ] **Sensitivity labels**: apply Microsoft Purview labels (optionally mandatory); they inherit from sources, persist on export, and can encrypt. Verify coverage via metadata scanning APIs.
- [ ] **DLP for Power BI**: detect sensitive info types in published semantic models (Premium/Fabric); act on alerts and policy tips.
- [ ] **RLS / object-level security / per-item sharing**: enforce data security (see `securing-row-level-access`); verify RLS on certified models with Get Datasets As Admin.
- [ ] **Lineage & data reuse**: use the tenant inventory + lineage to find duplicate semantic models, oversharing, unused content, and highly used data sources/gateways.
- [ ] **Data policies**: maintain short policies (ownership, certification/endorsement, classification & protection) in the centralized portal, each with owner, approver, scope, audit/enforcement method, exception process, effective/revision dates.

## Cross-layer audit activities (quick reference)

| Looking for… | Activity-log operation name |
|---|---|
| Tenant setting changed | `UpdatedAdminFeatureSwitch` (check the item name) |
| New / changed domain; workspace→domain assign; domain access | `InsertDataDomainAsAdmin` / `UpdateDataDomainAsAdmin` / `UpdateDataDomainFoldersRelationsAsAdmin` / `UpdateDataDomainAccessAsAdmin` |
| New / changed workspace | `CreateFolder` / `UpdateFolder` |
| Workspace access changed | `UpdateWorkspaceAccess` / `UpdateFolderAccess` |
| Workspace reassigned to capacity | `MigrateWorkspaceIntoCapacity` |
| Admin accessed a personal workspace (auto-revoked after 24h, not logged) | `AddAdminPersonalWorkspaceAccess` |
| Report shared (link or direct) | `ShareReport` (check `SharingAction`) |
| App created / updated / installed | `CreateApp` / `UpdateApp` / `InstallApp` |
| Report viewed | `ViewReport` |
| Publish to web code created | `PublishToWebReport` |
| Organizational visual added / updated | `InsertOrganizationalGalleryItem` / `UpdateOrganizationalGalleryItem` |
| Azure connection linked / unlinked | `AddLinkToExternalResource` / `DeleteLinkToExternalResource` |

Operation names evolve as the product evolves — confirm against the current Fabric operation list and prefer extracting raw activity data rather than hard-coding names.

## The Review → Audit loop (reusable template)

Apply to any control area:

```
Governance loop for <control area>:
- [ ] Review:   capture current state (admin portal and/or API snapshot)
- [ ] Decide:   workshop with COE/stakeholders; document decision + approver
- [ ] Update:   one or two designated admins change it; test first
- [ ] Document: record values + rationale; publish a user-facing summary
- [ ] Manage:   handle new options, user requests, security-group needs
- [ ] Audit:    detect changes via the activity log + snapshot diffs; alert on high-risk events
```
