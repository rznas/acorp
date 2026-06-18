# Workspace Role Permission Matrix

## Contents
- The four roles at a glance
- Full capability matrix (Admin / Member / Contributor / Viewer)
- Role-selection guidance (least privilege)
- Licensing rules
- Key role behaviors and gotchas

Source: `collaborate-share/service-roles-new-workspaces.md`, `service-new-workspaces.md`, `service-give-access-new-workspaces.md`. Roles can be assigned to individuals, security groups, Microsoft 365 groups, and distribution lists. If a user is in several groups, they get the **highest** role granted.

## The four roles at a glance

| Role | One-line intent |
|---|---|
| **Admin** | Owns the workspace: manage access/roles, delete the workspace, manage everything Members can, plus subscriptions created by others. |
| **Member** | Full content + app publishing + share, and can add lower-permission users (but can't change existing roles). |
| **Contributor** | Create/edit/delete content; can update the app only if the Admin delegated it. No app publish, no access management. |
| **Viewer** | Read-only: view and interact with content; receive subscriptions. Use for consumers and to enforce RLS. |

## Full capability matrix

`Y` = supported. Blank = not granted by that role. "If allowed" = only when an Admin delegates it.

| Capability | Admin | Member | Contributor | Viewer |
|---|:---:|:---:|:---:|:---:|
| Update and delete the workspace | Y | | | |
| Add or remove any user in any workspace role | Y | | | |
| Allow Contributors to update the app for the workspace | Y | | | |
| Add members or others with lower permissions | Y | Y | | |
| Publish, unpublish, and change permissions for an app | Y | Y | | |
| Update an app | Y | Y | If allowed | |
| Share items in apps, including semantic models | Y | Y | | |
| Allow others to reshare items | Y | Y | | |
| Feature apps on colleagues' home | Y | Y | | |
| Manage semantic model permissions | Y | Y | | |
| Feature dashboards and reports on colleagues' home | Y | Y | Y | |
| Create, edit, and delete content in the workspace | Y | Y | Y | |
| Create a report in another workspace from a semantic model here | Y | Y | Y | |
| Copy a report | Y | Y | Y | |
| Create goals based on a semantic model in the workspace | Y | Y | Y | |
| Schedule data refreshes via the on-premises gateway | Y | Y | Y | |
| Modify gateway connection settings | Y | Y | Y | |
| View and interact with an item | Y | Y | Y | Y |
| Read data stored in workspace dataflows | Y | Y | Y | Y* |
| Create subscriptions to reports | Y | Y | Y | Y |
| Subscribe others to reports | Y | Y | Y | |
| Analyze in Excel | Y | Y | Y | ** |
| Download a PBIX file | Y | Y | Y | |
| Manage subscriptions created by others | Y | | | |
| Receive subscriptions created by others | | Y | Y | Y |

\* Reading dataflow data is granted at Viewer, **but** consuming a **dataflow Gen2** via the dataflow connector needs Admin/Member/Contributor - Viewer is not sufficient.

\*\* A Viewer can **Analyze in Excel** / export underlying data only if granted **Build** permission on the relevant semantic model.

Capabilities that require **Build permission** on the semantic model plus at least the **Contributor** role on source and destination workspaces: manage semantic model permissions, create a report in another workspace, copy a report, create goals. With at least Contributor you automatically get Build through the role. Gateway capabilities additionally require permissions on the gateway itself (managed separately).

## Role-selection guidance (least privilege)

- **Report/dashboard consumers** -> **Viewer**. Read-only, can self-subscribe, and is the role used to enforce RLS for people browsing the workspace.
- **Report authors who edit content** -> **Contributor**. Can create/edit/delete content but cannot publish the app or manage access.
- **Authors who also own the app and distribution** -> **Member**. Can publish/update the app and share content.
- **Workspace owners / lifecycle managers** -> **Admin**. Can manage roles and delete the workspace.
- Always prefer assigning **groups** (security, Microsoft 365, distribution lists) over individuals for maintainability.
- To let a Contributor maintain the app without making them a Member, enable **Allow contributors to update the app for this workspace** (workspace settings). Delegated contributors can update app metadata, add/remove items, and change item visibility - but cannot publish the app for the first time, manage app users/permissions, toggle auto-install, or change semantic-model share/build settings.

## Licensing rules

- Collaborating in a workspace or sharing content requires a **Power BI Pro or Premium Per User (PPU)** license (except pure viewing).
- In a **shared capacity**, everyone added to the workspace needs Pro/PPU.
- In a **Premium capacity**, a user with **only** the **Viewer** role can access without Pro/PPU. Assigning a higher role (Admin/Member/Contributor) prompts them to start a Pro trial when they try to create content. To keep a free Viewer free, make sure they have no higher role via any group.
- You **can** assign users to a role even if they lack the license to use it.
- Receiving subscriptions requires the recipient to have a paid license unless the content is in Premium capacity.

## Key role behaviors and gotchas

- **Members can't change existing users' roles.** They can add users at lower permission but can't remove anyone. To upgrade a Viewer to Contributor, an **Admin** must remove the user, then a Member (or Admin) re-adds them at the new role.
- Workspace creators are automatically **Admins**.
- Only **Admins** see/use **Manage access**. If the button is missing, the user isn't an Admin (or must reach it via **Workspace settings**).
- Permission changes take effect the next time the user signs in.
- If a user is deleted from Microsoft Entra ID, their workspace permissions are removed 30 days later.
- A Pro user (or service principal) can be a member of up to **1,000 workspaces**.
- B2B guest users assigned a role get that role's capabilities, but can't subscribe others (only themselves).
