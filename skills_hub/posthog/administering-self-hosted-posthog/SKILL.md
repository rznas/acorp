---
name: administering-self-hosted-posthog
description: Administers and governs a self-hosted (or Cloud) PostHog instance for a product manager with admin needs - managing member roles and access control (organization/project/resource/property levels, RBAC roles, SSO/SAML/SCIM), controlling cost with billing limits, alerts and spike detection, staying compliant (GDPR, HIPAA, CCPA, data retention, right-to-be-forgotten, IP capture), and configuring projects, environments, organizations, approvals and audit/activity logs. Use when asked how to manage access or permissions, add or remove members, restrict who sees what data, control or cap PostHog costs, set up SSO, delete person or event data, meet GDPR/privacy requirements, configure the instance, or which admin features are unavailable on self-hosted open source.
---

# Administering self-hosted PostHog

Governance and admin reference for a PM running PostHog on their own infrastructure. Focus: who can do what, what it costs, staying compliant, and configuring the instance — not devops.

## Self-hosted reality check (read first)

The open-source hobby Docker Compose deploy is **MIT-licensed, single-machine, unsupported**, and unlikely to scale past a few hundred thousand events. Critically for a PM:

> Self-hosted note: **All paid-plan features are Cloud-only.** On open-source self-host you LOSE: access controls / RBAC / roles, SSO enforcement & SAML, audit (activity) logs, approvals, custom data retention, group analytics, data pipelines, multiple projects/organizations, and more. See [reference/self-host-feature-gaps.md](reference/self-host-feature-gaps.md). If you need governance features (RBAC, SSO, audit logs, approvals), the docs steer you to **PostHog Cloud (US or EU region)** — self-hosting cannot deliver them.

> Self-hosted note: Multiple organizations and multiple projects are **not available on open-source hobby deployments** — you get one organization, one project.

Deploy/upgrade/ops (install commands, containers, troubleshooting, instance settings, staff users, securing the instance) is in [reference/deploy-and-ops.md](reference/deploy-and-ops.md). The rest of this skill is the PM-relevant governance layer (assumes a plan that includes these features).

## PM question -> feature map

| PM question | Feature | Where it lives |
|---|---|---|
| Who can view/edit what? | Access control (org/project/resource/property) | Settings > Organization/Project > Access control |
| Add / remove / re-role a teammate | Members + invites | Settings > Organization > Members |
| Group permissions at scale | RBAC roles (Enterprise) | Settings > Organization > Roles |
| Force login via our IdP | SSO enforcement / SAML / SCIM | Settings > Organization > Authentication domains |
| Stop a surprise bill | Billing limits + alerts | Organization > Billing |
| Catch a usage spike early | Spike detection (automatic) | Email (no setup) |
| Lock budget / get a discount | Pre-paid plans | Sales |
| Who changed this flag / audit for SOC 2 | Activity (audit) logs | Settings > Project > Activity logs |
| Require review before a flag change | Approvals | Settings > Project > Approvals |
| Delete a user's data (GDPR) | Person/event deletion, right to be forgotten | Persons tab / API |
| Stop collecting IPs / PII | IP data capture, masking, transformations | Org/Project settings, CDP transformations |
| Separate dev/staging/prod data | Projects | Project switcher (top bar) |

---

## Access control & member management

Three (really four) nested levels. A user's effective access is the **highest** granted from any source. Org admins/owners always have full access and can't be restricted. Full detail, precedence rules, and copy-paste setup recipes: [reference/access-control.md](reference/access-control.md).

**Organization roles** (set in Settings > Organization > Members):
- **Member** (base) – view/query all project data, invite at own level or below.
- **Admin** – + billing, projects, access controls, auth settings, members, RBAC roles.
- **Owner** – + transfer ownership, delete org. At least one required.

**Project level**: default access of `No access` / `Member` / `Admin` applies to all org members; override per member or role. Use `No access` default to build a private project (e.g. executives-only).

**Resource level** (Boost/Scale/Enterprise): per-object or per-type access — `No access` / `Viewer` / `Editor` / `Manager` — on dashboards, insights, feature flags, experiments, surveys, notebooks, recordings, etc.

**Property level**: hide a property (e.g. `email`) from a role with `No access` / `Read` / `Read & write`. The PM move for sharing data with a vendor without exposing PII.

> Critical limitation a PM must know: resource access controls limit the **UI and API**, but do **not yet** restrict the underlying data. Any project member can still query events/warehouse via SQL/HogQL and **via PostHog AI**, view session replays, and open person/group profiles. Do not treat resource ACLs as a data firewall.

**RBAC roles** (Enterprise): roles can be created on any plan but only *enforce* access on Enterprise. Use them for "country teams," "vendors," reviewers, etc.

**Members & invites**: invites are valid 3 days, email-specific. Admins can disable "Members can invite." Org members are auto-emailed when someone new joins (security signal) — toggle in Notification Preferences.

> Self-hosted note: invite emails only send if the instance is email-enabled; otherwise share the invite link manually. See deploy-and-ops reference.

---

## Authentication & SSO

SSO config is domain-based and lives in **Settings > Organization > Authentication domains** (domain must be DNS-verified on Cloud). Capabilities by plan: JIT provisioning & SSO enforcement (Boost+), SAML (Scale+), SCIM auto-provisioning/deprovisioning (Enterprise). Full setup incl. Okta/OneLogin SAML & SCIM, GitHub/GitLab/Google self-host env vars, and security warnings: [reference/sso-and-auth.md](reference/sso-and-auth.md).

- **SSO enforcement** blocks passwords/resets and forces one provider — applies to admins/owners too, so verify it works first.
- **Personal API keys still work** even with SAML enforced (auth-only). Org admins/owners can audit all keys in Settings > Organization > Security.
- **Account security**: per-user 2FA, password rules, connected OAuth apps — see [reference/access-control.md](reference/access-control.md).

> Self-hosted note: Google SSO is **not available** on open-source deployments. GitHub/GitLab/Google are configured per-instance via env vars; the **first user cannot sign in via SSO** (create a password user first).

---

## Controlling cost

PostHog is usage-based (events, recordings, flag requests, pipeline rows). For a PM the levers are:

1. **Billing limits** — Organization > Billing > set a $ limit per product. Hitting the limit **drops data permanently** (events/replays lost, not deferred). Trade-off: protect budget vs. lose data.
2. **Billing alerts** — automatic emails to the org owner at **80% and 100%** of both the limit and the free allotment.
3. **Spike detection** — runs daily, emails about unusual up/down usage swings (implementation bugs, traffic surges, outages). No setup; sent only to customers without an assigned account owner; max one consolidated email per 7 days.
4. **Estimate before committing** — sign up free and read projected volume on the billing page after ~1 week, or use the [pricing calculator](/pricing). Event/MAU heuristics and per-product reduction tips: [reference/billing-and-cost.md](reference/billing-and-cost.md).
5. **Pre-paid plans** — discounted credit usable across products, aligns to budget cycle; roll over up to half unused credit on renewal. Via Sales.

Billing emails go to **all owners + admins** and can't be re-pointed — invite a finance alias as an admin if needed. Invoices/cards: Billing > Manage card details (Stripe).

> Self-hosted note: billing limits, the billing dashboard, and Stripe invoicing apply to PostHog Cloud / paid plans. Open-source hobby has no PostHog bill (you pay your own infra).

---

## Staying compliant (GDPR / HIPAA / CCPA / data retention)

PostHog gives controls; **you decide what to collect and own compliance**. On self-host, *you are both data processor and controller*. Detailed regulation walkthroughs, masking, transformations, and deletion APIs: [reference/privacy-and-compliance.md](reference/privacy-and-compliance.md).

PM-relevant controls by lifecycle stage:

- **Before data leaves the client**: top-level opt-out (`opt_out_capturing`, blocks everything), cookieless mode (`always` / `on_reject`), autocapture controls, session-replay masking (mask all inputs/text by default, selectively unmask), `before_send` hook to strip fields.
- **IP capture**: set org default (Settings > Organization > General > IP data capture default) and per-project override (Settings > Project > General). **EU orgs default to IP capture disabled.** GeoIP/bot detection can still use IP before it's discarded.
- **Before storage**: CDP realtime transformations to hash PII, anonymize IP, drop events, disable GeoIP.
- **Right to be forgotten / deletion**: delete a **person** (and optionally their events via `delete_events=true`) in the Persons tab or via the persons API; delete projects/orgs/accounts in their settings. Event deletion is **asynchronous** (runs off-peak / weekends on Cloud); check status via the deletion-status endpoint.
- **Data location**: for robust GDPR, docs recommend **PostHog Cloud EU** (Frankfurt). If self-hosting outside the EU with EU users, **anonymize** their personal data via transformations.
- **HIPAA**: requires a BAA — only offered for **PostHog Cloud**, not self-host.
- **Audit trail for compliance**: activity logs (below) are append-only; export for SOC 2 / SIEM.

> Self-hosted note: these are not legal advice. SSRF egress protection (Smokescreen) is Cloud infrastructure — on self-host **you** must add egress filtering. HIPAA BAAs and PostHog Cloud EU residency are not available to self-hosted instances.

---

## Audit, change governance & projects

**Activity (audit) logs** (Scale/Enterprise) — Settings > Project > Activity logs. Records actor (with MCP badge if via MCP), action, scope, target, and before/after values at org/project/resource level. Org admins/owners can view org-wide (`?view=organization`). Filter by user/scope/action/date; export CSV/Excel (project view only); query via `system.activity_logs` HogQL or the activity-log API; subscribe to Slack/webhook notifications. Append-only and integrity-protected; retention is plan-based — **export regularly, deleted logs are unrecoverable**. Use cases (investigate a flag change, prep a SOC 2 audit, monitor API-key creation): [reference/audit-and-governance.md](reference/audit-and-governance.md).

**Approvals** (Scale/Enterprise) — Settings > Project > Approvals. Gate high-impact changes behind a review quorum. Today covers **feature flag** enable/disable/rollout. Define policy: action, optional conditions (any change / before-after threshold / change-amount), approvers (users or roles), quorum, self-approve on/off. PM use: require dual approval for production flag changes, or only gate rollout jumps > 20%. Stale-request/version-tracking rules: [reference/audit-and-governance.md](reference/audit-and-governance.md).

**Projects & environments** — A project is a data silo with its own write token. Recommended setup: separate Local Dev / Staging / Production projects, but keep app + marketing site in **one** production project to track full journeys. New projects inherit org defaults (IP capture, access defaults). Moving a project between orgs is self-serve within a region (needs 2+ projects in source org); cross-region moves need Scale/Enterprise + a PostHog engineer.

---

## AI & MCP helpers for admins

PostHog AI (Max) can answer governance questions in natural language over your **activity logs** — e.g. "Who changed the checkout-flow feature flag?", "What experiments were modified last week?" — using the same filters as the UI. Requires the audit-logs feature (Scale/Enterprise). Actions taken through **MCP** are tagged in activity logs (MCP badge / `client: mcp`), so you can audit AI/agent-driven changes.

> Self-hosted note: PostHog AI / Max and hosted LLM features are Cloud-oriented; availability may differ or require special configuration on self-hosted instances. For how a PM uses Max, see the **using-posthog-ai-assistant** skill.

---

## Related skills

- **using-posthog-ai-assistant** — querying activity logs and product data with Max/MCP.
- **managing-feature-flags-and-experiments** — the resources most often gated by approvals and access controls.
- **configuring-data-pipelines-and-cdp** — transformations used for PII hashing/anonymization and activity-log notifications.
- **analyzing-product-analytics** — the events, persons, and groups that access/property controls protect.
