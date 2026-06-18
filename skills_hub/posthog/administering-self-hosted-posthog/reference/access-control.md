# Access control & member management (detailed)

## Table of contents
- [Four levels overview](#four-levels-overview)
- [Organization roles](#organization-roles)
- [Project level](#project-level)
- [Resource level](#resource-level)
- [Property level](#property-level)
- [Precedence](#precedence)
- [RBAC roles by plan](#rbac-roles-by-plan)
- [Members, invites, notifications](#members-invites-notifications)
- [Account security](#account-security)
- [Setup recipes](#setup-recipes)
- [Access control API](#access-control-api)

## Four levels overview
PostHog manages permissions at organization, project, resource, and property levels. A user's **effective access is the highest** granted from any source. Organization owners/admins automatically get project admin access and full resource access; you cannot restrict them.

> Limitation: resource access controls limit the UI and API but **not the underlying data**. A member with "No access" to a resource can still query events / data-warehouse tables via SQL/HogQL, ask PostHog AI to query them, view session replays, and open person/group profiles. Data-level controls are planned, not shipped.

## Organization roles
Set in Settings > Organization > Members. Levels: Member (base), Admin, Owner. At least one Owner required.

| Permission | Member | Admin | Owner |
|---|---|---|---|
| View & query project data | yes | yes | yes |
| Invite members (own level or below)* | yes | yes | yes |
| Leave organization | yes | yes | no |
| Billing management | no | yes | yes |
| Manage reverse proxies | no | yes | yes |
| Create/delete projects | no | yes | yes |
| Manage project access controls | no | yes | yes |
| Auth settings (SAML, SSO, 2FA enforcement) | no | yes | yes |
| Org settings (name, logo) | no | yes | yes |
| Manage RBAC roles | no | yes | yes |
| Manage members (roles, remove) | no | yes | yes |
| Transfer ownership | no | no | yes |
| Delete organization | no | no | yes |

*"Members can invite" is toggleable off by admins/owners.

## Project level
Each project has a default access level applied to all org members: **No access**, **Member**, or **Admin**. Override per member or per role.

| Permission | Member | Admin |
|---|---|---|
| View/edit own or permitted resources | yes | yes |
| Manage project access controls | no | yes |
| Edit project settings | no | yes |
| Delete project | no | yes |

## Resource level
Available on Boost/Scale (members, not roles) and Enterprise (members + roles). Access via the **Access control** sidebar button on a supported resource, or project access settings.

Levels: **No access**, **Viewer**, **Editor**, **Manager** (Manager can also change the resource's access controls). New resources default to Editor. Creators, org admins, and Managers always retain full access.

Supported resources: actions, activity logs, customer analytics, dashboards, endpoints, error tracking, experiments, feature flags, insights, LLM analytics, logs, managed data-warehouse sources, notebooks, session recordings, surveys, traces, web analytics.

Two scopes:
- **Object access controls** — one specific object (e.g. Dashboard X). Limit 1,000 objects per resource type per project.
- **Resource access controls** — all objects of a type in the project (past and future), set at project level. Object-level overrides resource-level.

## Property level
Managed via **Property definitions** (event, person, and group properties supported). Levels: **Read & write**, **Read**, **No access**. "No access" fully hides the property — cannot be viewed, queried, or filtered. Primary tool for hiding PII (e.g. `email`) from a vendor role.

## Precedence
Highest priority first: 1) specific object permission, 2) resource-type permission, 3) default object permission. Resource creators and org admins always have full access regardless.

## RBAC roles by plan
- **Free / Pay-as-you-go**: no access controls. All projects/resources open to all members at Editor.
- **Boost / Scale**: default access levels for projects and resources; specific levels for individual members (not roles).
- **Enterprise**: full RBAC — create roles, assign permissions to roles at project and resource levels. Roles can be created on any plan but only enforce access on Enterprise.

## Members, invites, notifications
- Invites valid **3 days**, only for the specified email. Resend by removing and re-adding.
- Members see only invites at or below their level.
- Email-enabled instances auto-send invites and notify all existing members when someone joins (security signal; disable in Notification Preferences).
- New joiners start as Member.
- Org admins/owners can view all personal API keys (org-scoped, project-scoped, unscoped) in Settings > Organization > Security > Personal API key access: owner, masked value, scopes, access scope, last used, created. Secret value never exposed.

## Account security
Per-user, in Account settings:
- **2FA** — set up, save backup codes.
- **Password** — 8–72 chars, zxcvbn score ≥3, no expiry, lockout 10 min after 30 failed attempts.
- **Connected applications** — third-party OAuth apps (e.g. Claude Code, Cursor); revoke per app to kill its tokens.
- Email can't be changed under an SSO-enforced domain; verifying a new email removes linked social auth — set a password first.

## Setup recipes
- **Contractor, one dashboard only**: set all resource types to No access for them; grant Viewer on the one dashboard.
- **Country teams**: role per country; Viewer on all resource types; Editor on that country's dashboards/insights.
- **Executives-only project**: project default No access; create Executives role; grant it Admin on the project.
- **Analyst (create insights, view dashboards)**: No access default; Editor on all Insights; Viewer on all Dashboards.
- **Hide PII from vendor**: create vendor role; on the property definition's Roles tab, add override No access for that role.

## Access control API
Requires personal API key with `access_control:read` / `access_control:write`.
- `GET/PUT /{resource}/{id}/access_controls` — object-level (e.g. `/api/projects/@current/dashboards/{id}/access_controls`).
- `GET/PUT /{resource}/{id}/resource_access_controls` — resource-type level. For a project-wide default pass `null` for both `organization_member` and `role`.
- `GET /{resource}/{id}/users_with_access` — who has access and via what source (`creator`, `organization_admin`, `explicit_member`, `explicit_role`, `project_admin`, `default`).
Specify either `organization_member` OR `role`, never both. Set `access_level` to `null` to remove.
