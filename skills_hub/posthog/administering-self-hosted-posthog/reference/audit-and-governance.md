# Audit logs, approvals & projects (detailed)

## Table of contents
- [Activity (audit) logs](#activity-audit-logs)
- [Approvals](#approvals)
- [Projects & environments](#projects--environments)

## Activity (audit) logs
Scale/Enterprise. Records who changed what, when — for investigating changes, tracking flag rollouts, and compliance audits.

**What's logged** (org / project / resource):
- Org: member invited/joined/removed/role-changed, role created/updated/deleted, SSO configured, domain verified, SCIM config changed, org settings changed, 2FA enforcement changed.
- Project: project created/deleted, settings changed, access control changed, API key created/revoked.
- Resource: feature flags, experiments, dashboards, insights, surveys, actions, cohorts, notebooks, annotations, data pipelines, batch exports, data-warehouse sources/views, alerts, early-access features, session replays (shared/exported), personal API keys, error-tracking issues, persons (updated/merged/split/deleted), groups, event/property definitions, tags, comments — each with relevant actions.

**Entry fields**: timestamp (UTC), actor (name/email; **MCP badge** if via MCP), action, scope, target (name+ID), before/after changes. API entries include `client` (e.g. `"mcp"`) and `is_system`.

**Where to view**:
- **Activity side panel** — context-aware feed; on a resource shows that item's changes, on a list page shows all items of that type. Only appears on pages with activity scope.
- **Project log** — Settings > Project > Activity logs. Toggle "Include organization-level activity" to add org events (also gates org-event notifications).
- **Org-wide** (admins/owners only) — Organization toggle, or `?view=organization`; adds a Project column and Project filter. Export not available in org view.
- **Resource history** — the History tab on a flag/dashboard/experiment.

**Filters**: date range, user, scope, project (org mode), activity type; More filters: was-impersonated, is-system-activity, item IDs, detail filters (equals/contains/is-one-of on fields and before/after values), client (e.g. MCP).

**Access, integrity, retention**:
- Controlled by access control; default visible to all members, restrict via roles.
- Append-only — cannot be modified or deleted by users.
- Retention is plan-based (Enterprise longest + custom windows). **Deleted logs are unrecoverable — export regularly.**
- Not everything is logged; gaps exist.

**Programmatic access**:
- Export: project view only, CSV/Excel, respects filters (for SIEM/compliance).
- API (`activity_log:read`): `GET /api/projects/@current/activity_log/`, `.../advanced_activity_logs/`, `GET /api/organizations/<id>/advanced_activity_logs/` (admin/owner, `team_ids[]`).
- HogQL: `SELECT ... FROM system.activity_logs` (team-scoped only; org-level events not in this table).
- Notifications: powered by data pipelines via internal `$activity_log_entry_created` event (not in your event stream) → Slack/webhook/etc., filtered by resource/action/specific item.
- PostHog AI can query logs in natural language (requires audit-logs feature).

**Use cases**:
- *Investigate a flag change*: flag History tab or Project Activity logs > Scope: Feature flag > Item IDs filter > inspect before/after.
- *SOC 2 audit*: Include org-level activity > set date range > Scope: Organization/membership/invite > export.
- *Monitor API keys*: Scope: Personal API key > Activity: created/deleted > export regularly.

## Approvals
Scale/Enterprise. Adds a review/quorum step before high-impact changes. Today gates **feature flags**: enable, disable, change rollout percentage.

**Flow**: gated action → change request (captures intended change, notifies approvers) → reviewers approve/reject → on quorum PostHog auto-applies; on reject it's discarded and requester notified.

**Policy** (Settings > Project > Approvals > New policy): action, optional conditions, approvers (users and/or roles — anyone in either can approve), quorum, allow self-approve (off by default; self-approver must still be in the approver set).

**Conditions** (operators `> < >= <= == !=`):
- Any change — any modification to a field.
- Before/after — new value crosses a threshold (e.g. rollout > 50%).
- Change amount — magnitude exceeds a threshold (e.g. increase > 10%). Only compares values present in both before and after; newly added groups won't trigger it (use any_change/before_after).

**Change request states**: Pending, Approved, Applied, Rejected, Expired, Failed.

**Acting**: approve/reject (reject requires a reason) from the resource banner or Settings > Project > Approvals. Only the requester can cancel a pending request; blocked if it has approvals unless stale.

**Staleness / version tracking**: flags carry a version number recorded with the request. If the flag changes (bypass, API, another approved request) the version increments and the request becomes inapplicable — the flow can still complete but the change won't apply; create a new request. Prevents race conditions; coordinate before editing resources with pending requests.

**Use cases**: require approval for all production flag changes (no conditions, quorum 1+); gate only large rollout increases (change_amount > 20); dual approval for compliance (quorum 2, self-approval off).

## Projects & environments
A project is a data silo with its own write-only token (regenerable; old token instantly revoked). Each org starts with a "Default Project."

- **Recommended**: separate Local Dev / Staging / Production projects to isolate test data; but keep app + marketing site in **one** production project to track the full journey (filter by `host` or super properties to split). Use separate projects for genuinely separate products, internal/admin apps.
- **Inheritance**: new projects inherit org defaults — IP data capture policy and access-control defaults.
- **Moving a project between orgs** (Settings > Project > Move project): need owner/admin of source + membership of target; source org must have 2+ projects (free orgs limited to one, so may need temporary upgrade). Same-region = self-serve and free; cross-region (e.g. US↔EU) needs **Scale/Enterprise** and a PostHog engineer (physical data transfer). Members of the source org lose access (incl. API keys) unless also in the target org.
