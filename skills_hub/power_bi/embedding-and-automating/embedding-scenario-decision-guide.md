# Embedding Scenario Decision Guide

How to choose between the two Power BI embedding scenarios, the embedding identity, capacity/SKU, and no-code alternatives. Grounded in `embedded-analytics-power-bi.md`, `embed-tokens.md`, the two usage-scenario guidance docs, and `embedded-capacity.md`.

## Contents
- Decision tree
- Scenario comparison (org vs customer)
- Embedding identity: service principal vs master user
- Capacity and SKU matrix
- Embeddable content
- RLS in embedded scenarios
- No-code embedding alternatives
- Secure embed

## Decision tree

```
Who are the users?
├─ Internal users with organizational accounts (sign in to Microsoft Entra ID)
│     -> EMBED FOR YOUR ORGANIZATION (user owns data, SaaS)
│        Each user needs a Power BI license (or F64+/P/EM capacity).
│        Token: Microsoft Entra token used directly.  tokenType: Aad.
│
└─ External users / users with no Power BI account or license (typical ISV app)
      -> EMBED FOR YOUR CUSTOMERS (app owns data, PaaS)
         Your app authenticates with any method; users need no license.
         An embedding identity (service principal or master user) holds the license.
         Token: Microsoft Entra token -> generate an EMBED token.  tokenType: Embed.
         Requires an A, EM, P, or F capacity for production.
```

If you do not need a programmatic solution at all, consider **no-code embedding** (see below) before building either scenario.

## Scenario comparison (org vs customer)

| Dimension | Embed for your organization | Embed for your customers |
|-----------|-----------------------------|--------------------------|
| Alias | User owns data / SaaS | App owns data / PaaS |
| Typical author | Enterprise internal app, BI portal, intranet | ISV / third-party application |
| App-user authentication | Interactive (Microsoft Entra ID, user's own credentials) | Non-interactive / silent (any method) |
| App user needs Power BI license | Yes | No |
| Embedding identity | The signed-in user | Service principal (recommended) or master user |
| Token to embed | Microsoft Entra token (direct) | Embed token (generated from the Entra token) |
| `tokenType` in client config | `models.TokenType.Aad` | `models.TokenType.Embed` |
| R & Python visuals | Supported (region limits) | Not supported |
| A SKU | Not supported | Supported |
| Content location | Any workspace, including the user's personal workspace | Any workspace except a personal workspace |

## Embedding identity: service principal vs master user

Applies to **embed for your customers**. From `embed-sample-for-customers.md` and the usage-scenario doc:

| Consideration | Service principal (recommended) | Master user |
|---------------|---------------------------------|-------------|
| Mechanism | Microsoft Entra app's service principal object authenticates the app | A real Power BI user's username/password authenticates the app |
| Security | Microsoft Entra's recommended method; secret or certificate | Less secure; guard credentials, rotate password, never expose in the app |
| Delegated permissions (scopes) | Not required | Master user or admin must consent (e.g. `Report.ReadWrite.All`) |
| Power BI service (portal) access | Cannot sign in to the service | Can sign in with the credentials |
| License | No Pro license required; must be Member/Admin of the workspace | Requires Pro or PPU |
| Tenant setting | Must enable *Service principals can use Fabric APIs* (limit via a security group) | Not applicable |
| Paginated (RDL) reports | Required (master user cannot embed paginated reports) | Cannot embed paginated reports |
| Multitenancy at scale | Use **service principal profiles** | Not suitable for large multitenancy |

Setup either way: register a Microsoft Entra app, create a workspace, publish a report, gather parameter values (Client ID, Workspace ID, Report ID, plus Tenant ID + Client secret for SP, or username/password for master user), and add the identity to the workspace as **Member** or **Admin**.

## Capacity and SKU matrix

From `embedded-capacity.md`. A custom web app = an app built with the JavaScript/.NET SDKs or REST APIs (full UX control).

| Scenario | Fabric (F SKU) | Azure (A SKU) | Office (P / EM SKU) |
|----------|:---:|:---:|:---:|
| Embed for your customers (app owns data) | yes | yes | yes |
| Embed for your organization (user owns data) | yes | no | yes |
| Microsoft 365 apps (Teams / SharePoint / PowerPoint) | yes | no | yes |
| Secure URL embedding (from the service) | yes | no | yes |

Notes:
- **Embed for your organization** production needs one of: all users on Pro, all on PPU, or an F64+/capacity (which lets all users hold free licenses).
- **Free embed trial tokens** are for development testing only (Pro license). A capacity is mandatory for production, otherwise the *Free trial version* banner persists.
- A publishing account needs Pro or PPU; alternatively a service principal can publish via the `Post Import In Group` REST API without a Pro/PPU license.

## Embeddable content

Both scenarios can embed: Power BI reports, specific report visuals, paginated reports, the Q&A experience, dashboards, and specific dashboard tiles.

## RLS in embedded scenarios

For **embed for your customers**, set the embed token's **effective identity** to enforce row-level security, because users do not authenticate to Power BI directly. RLS authentication-method nuances (from `generate-embed-token.md`): Cloud RLS, AS live connection, and SSO each have different rules for whether the effective user ID may be omitted, and service principals must supply an identity for any RLS semantic model. Master users cannot generate an embed token for RDL (paginated) reports. For full RLS modeling and DAX, cross-reference the `securing-row-level-access` skill.

## No-code embedding alternatives

Before building code, consider these (confirmed in the usage-scenario docs):
- **Embed for your organization** no-code: Power BI report web part in **SharePoint Online**; **secure embed** code/HTML in internal portals; **Power Pages**; **Microsoft Teams** channel/chat. All require the consumer to be in the org, authenticated, and permitted.
- **Embed for your customers** no-code: embed reports/dashboards in **Power Pages**.
- For a branded internal BI portal, **custom branding** of the Power BI service may be enough without any embedding.

## Secure embed

**Secure embed** is the simplest no-code way to embed a report into any portal that accepts a URL or iFrame. The viewer must have a proper Power BI license, can interact with the report but cannot edit/save it. It is available in the Power BI service and is distinct from the developer embedding solutions above.
