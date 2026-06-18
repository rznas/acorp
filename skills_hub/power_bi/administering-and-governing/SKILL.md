---
name: administering-and-governing
description: Operates a Power BI (Microsoft Fabric) tenant at organizational scale — managing tenant settings, workspaces, domains, capacities and data gateways; auditing and monitoring usage via the activity log, tenant inventory, and usage metrics; applying information protection and data loss prevention; and establishing a Center of Excellence and adoption practices. Use when administering a Fabric/Power BI tenant, governing workspaces or content, appointing or auditing Fabric administrators, setting up activity-log extraction or monitoring, planning gateways, classifying content with sensitivity labels, or assessing governance and adoption maturity.
---

# Administering and Governing Power BI

## Overview

Operating Power BI at org scale is the **Operate** stage of the lifecycle. A Fabric administrator (the renamed Power BI administrator role) balances three competing goals: **governance**, **user empowerment**, and **adoption**. Governance is enacted day-to-day through *system oversight* (administration): managing what users can do, auditing what they did, protecting data, and supporting the community via a Center of Excellence (COE).

This skill grounds every recommendation in the Microsoft Power BI implementation-planning and Fabric adoption-roadmap docs. It depends on `publishing-and-sharing-content` (workspace roles, apps, endorsement) and cross-references the Kimball `program-management` skill for the program/governance-board framing of a BI platform.

## When to Use

- Setting up or reviewing the Microsoft Entra tenant, home region / Multi-Geo, and CTID for external users.
- Appointing Fabric administrators and governing **tenant settings**, **domains**, **workspaces**, **embed codes**, **organizational visuals**, and **Azure connections**.
- Planning **data gateways** (standard / personal / VNet) and gateway/connection roles.
- Building an **auditing/monitoring** solution: activity log, tenant inventory, usage metrics, monitoring at tenant/report/data level.
- Applying **information protection** (sensitivity labels) and **DLP** at the org level.
- Establishing a **COE** and assessing **adoption maturity** (organizational, user, solution).

## Foundational principles (do not skip)

1. **A Fabric administrator must not decide alone.** Every tenant-setting or governance change is a governance decision — discuss, plan, and document it with the COE, BI strategy working team, and (for strategic items) the executive sponsor. Acting alone is "a clear indicator that you need to improve and refine your governance processes."
2. **It is a high-privilege role — appoint 2 to 4 administrators.** Too many increases risk of unapproved change; too few under-supports the system. Use **Privileged Identity Management (PIM)** for occasional just-in-time admin rights, assign backups, cross-train, and audit role membership regularly.
3. **Empowerment over prohibition.** Before disabling a feature because you see overuse (for example, many data exports in the activity log), understand *why* users rely on it. Prohibiting features drives workarounds. Prefer redesign or training; "the answer isn't 'no' by default, but 'yes and' with clear, transparent rules."
4. **Extract and store activity-log data now**, even before you can analyze it — it is available for **only 28 days**.
5. **Use the most lightweight governance model that meets objectives**, applied iteratively (method 3: iterative governance with staged rollouts is the best choice for most organizations).

## Governance control layers

Govern at three layers; see [governance-control-checklist.md](governance-control-checklist.md) for the full per-layer checklist.

| Layer | What you control | Primary surfaces |
|-------|------------------|------------------|
| **Tenant** | Tenant settings (feature delegation by security group), domains, embed codes, organizational visuals, Azure connections, capacities, who is admin | Fabric admin portal; Get Tenant Settings REST API |
| **Workspace** | Roles (Admin/Member/Contributor/Viewer), creation permissions, naming conventions, organization, license mode, domain assignment | Admin portal workspaces page; scanner/Groups APIs |
| **Content** | Endorsement (Promoted/Certified), sensitivity labels, RLS, per-item sharing, lineage | Endorsement; sensitivity labels; metadata scanning APIs |

The six-step pattern in the docs — **Review → Decide → Update → Document → Manage → Audit** — applies uniformly to tenant settings, domains, workspaces, embed codes, organizational visuals, and Azure connections. Use it as your repeatable governance loop.

### Tenant settings essentials

- Record current state with columns: *date reviewed, setting, current value, security groups allowed, groups not allowed, delegated to other admins*. Automate snapshots (weekly) via the **Get Tenant Settings** REST API and diff snapshots to detect changes.
- Each setting should map to a documented governance decision (decision, decision-makers, approver, affected setting, pending action). Re-review decisions on a schedule (twice a year is a good default).
- **Publish a user-facing summary** of settings and which security group to join to request a capability — this cuts confusion and support requests.
- Audit changes with the `UpdatedAdminFeatureSwitch` activity in the activity log; snapshot diffs reveal *what* changed; group-membership data (Microsoft Graph) reveals *who*.

### Domains and workspaces

- A **domain** groups workspaces with similar characteristics into a single management boundary (for example, all sales workspaces). A workspace belongs to exactly one domain. Domain admins/contributors can override delegated tenant settings.
- Use the admin portal workspaces page or **metadata scanning (scanner) APIs** (incremental, best for large tenants) / **Get Groups As Admin** / **Get-PowerBIWorkspace** (full pull, best for small tenants) to build a workspace inventory.
- Fabric admins can manage workspace roles, recover deleted workspaces, reassign license mode, and get 24-hour temporary access to a personal workspace. The role does **not** automatically grant access to all data — but admins can grant themselves workspace roles.

### Embed codes and organizational visuals

- **Publish to web** creates a public embed code discoverable by search engines — never use it for internal/sensitive content; for internal embedding use API embedding or the no-code secure embed. Restrict the *Publish to Web* tenant setting to a very limited set of users and review active codes regularly.
- **Organizational visuals**: register tested/approved custom visuals (from AppSource or `.pbiviz`) in the org store for consistent versions; pair with the *block uncertified visuals* / *certified visuals only* tenant settings. Note: custom-visual tenant settings apply only to published reports — manage Desktop visuals via **Group Policy**.

## Capacities and data gateways

- Administering Fabric/Premium **capacity** is distinct from managing the Power BI service. An org has one tenant but can provision multiple capacities (per workload or business unit). Capacity license type affects features such as **Multi-Geo** (any capacity license except shared; not PPU) and **VNet gateways** (Fabric/Premium capacities only).
- **Gateways** bridge cloud Power BI to on-premises or private-network data. Choosing the type and managing roles is a core admin task — see [gateways are needed when the source is on-premises, in a private network (including cloud sources behind a firewall), requires host connector software, or needs security isolation].

| Gateway type | Use it for | Sharing |
|--------------|-----------|---------|
| **On-premises (standard mode)** | Most scenarios: shared use, DirectQuery, SSAS live connections, custom connectors, multiple Power Platform services | Shared, centrally managed on an always-on VM |
| **Personal mode** | Limited personal BI only; Python/R in Power Query | Cannot be shared — avoid for team/dept/enterprise |
| **VNet (managed service)** | Private-network sources / private endpoints | Shared; Fabric/Premium capacity only; single fixed hardware config |

- Use **gateway clusters** (multiple nodes) for high availability and load balancing on business-critical workloads; keep **separate clusters per environment** (dev/test/prod) and per region (install gateways close to data sources). Restrict who can install gateways; document **recovery keys** in a shared secure vault; test monthly updates on dev/test before prod (no rollback exists).
- Manage access with **gateway roles** (Admin / Connection creator with sharing / Connection creator) and **connection roles** (Owner / User with sharing / User), using security groups and least privilege. A pure data user needs only the *User* connection role, no gateway role.

Cross-reference: gateway choice and storage-mode/refresh tradeoffs are detailed in `connecting-data-sources` and `transforming-data-power-query`.

## Auditing and monitoring

**Auditing** = understanding the system and user activity (manual + automated). **Monitoring** = ongoing alerting/automation about what's happening now. Auditing data drives adoption analysis, usage understanding, risk mitigation, compliance, license/cost management, and performance input.

Sources span three levels (tenant / report / data) plus the activity log. Full table with each source, what it answers, and how to access it: [auditing-monitoring-sources.md](auditing-monitoring-sources.md).

Key facts to apply:
- The **Power BI activity log** is retrieved with `Get-PowerBIActivityEvent` (Power BI Management PowerShell module) or the **Get Activity Events** admin REST API. The cmdlet returns one day per call; the raw API returns one hour per call. Caller must be a Fabric administrator (or use a service principal for production). The API is throttled to **200 requests/hour** — extract raw data once per day (ELT: extract, then transform separately), do not filter during extraction.
- A **tenant inventory** is a point-in-time metadata snapshot (workspaces, reports, semantic models, lineage, gateways, capacities). Combine it with activity-log data to find unused content, oversharing, and the semantic-model-to-report ratio. Snapshot weekly or monthly.
- Working DAX/PowerShell-adjacent example — extracting yesterday's full log avoids partial days:

```powershell
# Extract all activity for the previous full UTC day (avoids partial-day data)
Connect-PowerBIServiceAccount   # interactive; use a service principal for scheduled runs
$DateToExtract = ([datetime]::Today.ToUniversalTime().Date).AddDays(-1).ToString('yyyy-MM-dd')
Get-PowerBIActivityEvent `
    -StartDateTime ($DateToExtract + 'T00:00:00.000') `
    -EndDateTime   ($DateToExtract + 'T23:59:59.999') |
    Out-File "C:/Power-BI-Raw-Data/Activity-Log/PBIActivityEvents-$($DateToExtract -replace '-','').json"
```

## Information protection and data loss prevention (DLP)

- Information protection and DLP is an **organization-wide** initiative (Microsoft Purview), far larger than Power BI; the Fabric admin usually collaborates rather than owns it. Involve the information security team.
- **Sensitivity labels** (Microsoft Purview Information Protection) classify content; they display in plain text (educating users), can be **inherited from data sources**, are **retained on export** to supported file types, and can apply **encryption** that follows the file across devices/users.
- **DLP for Power BI** detects sensitive information types in published semantic models (Premium/Fabric workspaces) and raises alerts and policy tips. **Defender for Cloud Apps** adds real-time controls (for example, block download of content with a given label).
- Security ≠ information protection: RLS, workspace roles, app permissions, and per-item sharing are *security* (covered by `securing-row-level-access`); labels/DLP are *information protection*.

## Center of Excellence and adoption

- A **COE** is a team that establishes/maintains the BI platform and a single source of truth with consistent company-wide metrics — not a help desk. Large orgs use a **core COE** plus **satellite COEs** at department level (satellites "pay to play" and extend core conformed dimensions/facts with local logic).
- Recommended COE community activities: regular **office hours**, a Teams channel for Q&A, user groups, and formal training (for example, Dashboard in a Day).
- Assess **adoption maturity** on three inter-related axes — organizational (governance effectiveness), user (consumer/creator proficiency), solution (business value of a deployed solution). Aim for at least level 300–400. See the quick guide: [adoption-maturity-guide.md](adoption-maturity-guide.md).

Cross-reference: the COE governance board, steering committee, roles/RACI, and program portfolio prioritization mirror the Kimball `program-management` governance and sponsorship framework.

## Common pitfalls

| Pitfall | Why it hurts | Correction |
|---------|--------------|------------|
| Admin disables a feature unilaterally after seeing activity-log spikes | Drives user workarounds; signals broken governance process | Investigate *why*; redesign or train; decide with the COE |
| Too many (or too few) Fabric administrators | High-privilege role; risk of unapproved change or poor support | Appoint 2–4; use PIM for occasional needs; audit membership |
| Not extracting the activity log | Data is gone after 28 days; no historical analysis | Automate daily extraction to storage immediately |
| Filtering/looping the activity log during extraction | Hits the 200 req/hour throttle; brittle | ELT — extract raw once per day, transform later |
| Using *Publish to web* for internal/sensitive reports | Public, search-discoverable data leakage | Restrict the tenant setting; use API or secure embed |
| Personal gateways for team/enterprise solutions | Can't be shared; tied to a user machine; hard to govern | Use standard-mode gateway on an always-on VM; cluster for prod |
| Treating sensitivity labels as access control | Labels classify/protect; they don't grant or deny access | Use RLS/roles/sharing for security; labels for protection |
| Documentation without engagement | False sense of control; complacency | Pair docs with COE engagement, auditing, and monitoring |

## Success criteria

- 2–4 documented Fabric admins with backups, cross-training, PIM for occasional access, and periodic role audits.
- Every tenant setting maps to a documented, approved governance decision; snapshots are automated and diffed; user-facing summary published.
- Workspace, domain, embed-code, organizational-visual, and Azure-connection governance each run the Review→Decide→Update→Document→Manage→Audit loop on a schedule.
- Gateways are inventoried and named consistently, prod uses clusters, recovery keys are stored securely, updates are tested on dev/test first.
- Activity log is extracted daily and retained; a tenant inventory snapshot runs weekly/monthly; monitoring/alerts exist for high-risk events (for example, `UpdatedAdminFeatureSwitch`, `PublishToWebReport`).
- Sensitivity labels and (where applicable) DLP/Defender for Cloud Apps are in place with security-team collaboration.
- A COE exists with clear scope, an executive sponsor, and a community of practice; adoption maturity is assessed per goal and trending toward 300–400+.

---
*Provenance: built per SKILL 20 "Administering and Governing" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI implementation-planning docs (tenant-setup, tenant-administration, workspaces, auditing-monitoring, info-protection-and-DLP, data-gateways), the admin activity-log doc, and the Fabric adoption-roadmap (governance, maturity-levels) and center-of-excellence-establish docs; cross-referenced with Kimball `program-management`.*
