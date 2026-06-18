# SSO, SAML & SCIM (detailed)

## Table of contents
- [Capability by plan](#capability-by-plan)
- [Authentication domains](#authentication-domains)
- [JIT provisioning](#jit-provisioning)
- [SSO enforcement](#sso-enforcement)
- [Third-party providers (self-host env vars)](#third-party-providers-self-host-env-vars)
- [SAML](#saml)
- [SCIM](#scim)
- [Security warnings](#security-warnings)

## Capability by plan
| Feature | Free/PAYG | Boost | Scale | Enterprise |
|---|---|---|---|---|
| JIT provisioning | no | yes | yes | yes |
| SSO enforcement | no | yes | yes | yes |
| SAML | no | no | yes | yes |
| SCIM | no | no | no | yes |

## Authentication domains
SSO is domain-based. Add/verify in Settings > Organization > Authentication domains. On Cloud, verify via DNS: add a `TXT` record at `_posthog-challenge.yourdomain.com` with the supplied value (TTL 3600). The record must stay in place — PostHog re-verifies periodically. A verified domain is required for JIT, SSO enforcement, SAML, and SCIM. One SAML provider per domain (multi-tenant SAML across domains is supported).

## JIT provisioning
Creates an account on first SSO login when the email matches your verified domain. Cloud supports GitHub, GitLab, Google, and SAML. Enable via the "Enable automatic provisioning" toggle on the domain.

## SSO enforcement
Forces one provider for a domain. When on, users cannot use a password, request resets, or use any other provider — **including admins and owners**. Applies to existing users and even users not in your org if their email is on the domain. Verify the provider works before enabling. Passwords are preserved, so you can turn enforcement off to recover.

## Third-party providers (self-host env vars)
Configured per-instance (not per-domain). After setting vars, restart the server. The **first user of an instance cannot sign in via SSO** — create a password user first.

- **GitHub**: register an OAuth app; callback `<instance>/complete/github/`; set `SOCIAL_AUTH_GITHUB_KEY`, `SOCIAL_AUTH_GITHUB_SECRET`.
- **GitLab**: redirect `<instance>/complete/gitlab/`, scope `read_user`; set `SOCIAL_AUTH_GITLAB_KEY`, `SOCIAL_AUTH_GITLAB_SECRET`, plus `SOCIAL_AUTH_GITLAB_API_URL` if self-hosting GitLab.
- **Google**: **not available on open-source deployments.** On supported plans: create OAuth client ID, redirect `https://{hostname}/complete/google-oauth2/`; set `SOCIAL_AUTH_GOOGLE_OAUTH2_KEY`, `SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET`. Prefer internal-only consent.

## SAML
Scale+. Domain-based (env-var-based SAML is fully deprecated; self-host needs version 1.35.0+). Requires `SITE_URL` set and the instance running over TLS.

IdP must send these assertion attributes (PostHog default names): `name_id` (permanent ID, required), `email` (required), `first_name` (required), `last_name` (optional).

Service Provider values: ACS/SSO URL `<yourdomain>/complete/saml/`; Entity ID = your `SITE_URL` verbatim; RelayState from the SAML config modal (self-host: empty/default). Metadata available at `<yourdomain>/api/saml/metadata/`.

Configure in PostHog (per domain): SAML Entity ID (IdP issuer), SAML ACS URL (IdP sign-on endpoint), SAML X.509 certificate (keep spaces/newlines). Docs include full Okta and OneLogin (quick + advanced) walkthroughs. Debug self-host via app logs, or temporarily `DEBUG=1` (remove after).

## SCIM
Enterprise. Auto-syncs users and roles from the IdP. Prerequisites: verified domain, working SAML, SCIM 2.0 IdP (Okta, Entra ID, OneLogin).
- Assigning a user in the IdP creates them in PostHog with the mapped role; removing deactivates them; matching emails are updated.
- Roles matched by name (case-sensitive).
- SCIM changes (enable/disable/token rotation) are recorded in the activity log.
Setup: domain ⋯ menu > Configure SCIM; copy SCIM Base URL + Token into the IdP; map roles; assign users/groups. Okta and OneLogin examples provided in docs.

## Security warnings
- Only use SAML with IdPs that validate the user's email (first login associates by email; spoofable email = impersonation risk).
- Enabling/enforcing SAML does **not** disable personal API keys — users can still create and use them (API only).
- SAML handles auth + provisioning only; removing a user from the IdP does not delete their PostHog account (they may just be unable to log in).
- Existing passwords are preserved when enabling/enforcing SAML, enabling rollback.
