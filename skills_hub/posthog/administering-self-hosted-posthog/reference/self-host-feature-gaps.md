# What you lose on open-source self-hosted PostHog

The open-source hobby deploy only includes free-plan features. **All paid-plan features are Cloud-only.** Use this list when a PM asks "can I do X on our self-hosted instance?"

## Governance / admin features NOT in open-source self-host
- Access controls and RBAC (roles) — projects and resources are open to all members at Editor.
- SSO enforcement (Teams/Boost+); SAML (Scale+); SCIM (Enterprise).
- Google SSO (open-source has GitHub/GitLab only).
- Audit / activity logs (Scale/Enterprise).
- Approvals / change requests (Scale/Enterprise).
- Custom data retention.
- Multiple projects and multiple organizations (hobby = one org, one project).
- Priority/commercial support, account management, training, custom MSA.
- Billing limits / billing dashboard / spike-detection emails (no PostHog bill on hobby).
- HIPAA BAA and PostHog Cloud EU data residency.
- Infra-level SSRF egress protection (Smokescreen).

## Product features lost (per the disclaimer)
- **Product analytics**: insight & dashboard subscriptions, advanced paths, correlation analysis, lifecycle insights, one-year event retention, more than two alerts, group analytics, data pipelines.
- **Web analytics**: path cleaning.
- **Session replay**: download recordings.
- **Feature flags & experiments**: multi-environment support, group experiments.
- **Surveys**: multiple questions, custom colors/positioning, custom HTML, recurring surveys, event-based surveys.

## Platform packages (Cloud) for reference
- **Teams**: priority support, unlimited projects, white labeling, SSO enforcement.
- **Enterprise**: SAML, highest support, RBAC, custom MSA, account management, training/onboarding, audit logs, custom data retention, unlimited projects.

## PM takeaway
If the requirement is governance (RBAC, SSO/SAML/SCIM, audit logs, approvals, custom retention, multi-project), open-source self-host cannot deliver it — the docs' decision flow routes such needs to **PostHog Cloud** (US or EU). Self-host suits hobbyists or strict "data never leaves our infra" cases willing to accept no support and feature loss.
