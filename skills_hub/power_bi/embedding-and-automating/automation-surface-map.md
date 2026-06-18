# Automation Surface Map (REST API / SDK / PowerShell)

The programmatic surfaces for automating Power BI, plus the multitenancy provisioning pattern with service principal profiles. Grounded in `embed-tokens.md`, `embed-customer-app.md`, `embed-organization-app.md`, `generate-embed-token.md`, the usage-scenario docs, and `develop-scalable-multitenancy-apps-with-powerbi-embedding.md`. Keep code high-level; full samples live in the official docs and the PowerBI-Developer-Samples repo.

## Contents
- Automation surfaces at a glance
- Authentication prerequisites
- Common REST API operation groups
- Tenant provisioning sequence (app owns data)
- Generating an embed token
- Service principal profiles
- Client-side (JavaScript) embedding config
- PowerShell / package commands

## Automation surfaces at a glance

| Surface | Language | Use it for |
|---------|----------|------------|
| **Power BI REST API** | HTTP (any client) | Workspaces (`groups`), imports, datasets, refresh, takeover, credentials, embed tokens, profiles, user permissions. Needs a Microsoft Entra token. |
| **Power BI .NET SDK** (`Microsoft.PowerBI.Api`) | C#/.NET | Typed wrapper over REST (e.g. `Reports.GenerateTokenAsync`, `Groups.CreateGroup`, `Profiles.CreateProfile`). |
| **Power BI client APIs** (`powerbi-client`) | JavaScript / TypeScript | Browser-side embedding, filters/slicers, bookmarks, navigation, layout, events. |
| **PowerShell / terminal** | PowerShell, shell | Drive SDK/package setup and script REST calls. |

## Authentication prerequisites

- All REST API operations require a **Microsoft Entra token** (expires after 1 hour).
- App-owns-data also requires generating an **embed token** from that Entra token.
- Service principal path: register a Microsoft Entra app, enable *Service principals can use Fabric APIs* tenant setting (scope with a security group), and add the service principal as **Member/Admin** of each workspace.
- The default Power BI API scope used by the .NET sample for app-owns-data is `https://analysis.windows.net/powerbi/api/.default`; the org sample uses delegated scopes such as `https://analysis.windows.net/powerbi/api/Report.Read.All`.

## Common REST API operation groups

Operations referenced across the source docs (use the linked REST reference for parameters):

| Need | Operation (group) |
|------|-------------------|
| List workspaces | `Get Groups` (groups) |
| Create a workspace | `Create Group` / `CreateGroup` (groups) |
| List reports in a workspace | `Get Reports In Group` (reports) |
| Import a .pbix to create a semantic model | `Post Import In Group` (imports) — also lets a service principal publish without Pro/PPU |
| Take over a semantic model | `TakeOver` (datasets) |
| Set data source credentials | datasets / gateways credential operations |
| Generate an embed token (V2, reports + semantic models) | `Generate token` (embed-token) |
| Generate a token for dashboards/tiles (V1) | `Dashboards GenerateTokenInGroup`, `Tiles GenerateTokenInGroup` |
| Refresh a user's permissions immediately | `Refresh User Permissions` (users) |
| Manage service principal profiles | `Create/Delete/Get/Update Profile` (profiles) |

## Tenant provisioning sequence (app owns data)

Recommended pattern from the multitenancy guidance — run as the service principal (or, for multitenancy, the service principal profile):

```
1. Create a new workspace.
2. Associate the workspace with a dedicated capacity.
3. Import a Power BI Desktop (.pbix) file to create a semantic model.
4. Set the semantic model parameters.
5. Set data source credentials (these are scoped to the identity, not the workspace).
6. Set up scheduled data refresh.
```

After these calls the identity is workspace admin, semantic-model owner, and owner of the data source credentials. Note: data source credentials are scoped to the service principal/profile (or user account) across the whole Microsoft Entra tenant, not per-workspace — a subtle isolation gotcha.

## Generating an embed token

`Generate token` (V2) creates a token for one or many reports / semantic models. In the request you set:
- **Access level** — `allowEdit` for view/edit; add the workspace ID to allow `SaveAs`/`CreateNew`.
- **Effective identity** — for RLS, so the embedded content is filtered per end user (see `securing-row-level-access`). Service principals must always supply an identity for an RLS semantic model. Master users cannot generate an embed token for RDL (paginated) reports.

High-level .NET shape (from `embed-customer-app.md`):

```csharp
// app owns data: generate a read-only embed token for a report
var tokenRequest = new GenerateTokenRequest(TokenAccessLevel.View, report.DatasetId);
var embedTokenResponse = await pbiClient.Reports.GenerateTokenAsync(workspaceId, reportId, tokenRequest);
string embedToken = embedTokenResponse.Token;
```

## Service principal profiles

For large multitenancy (many workspaces, >1,000 app users). The embedding identity cannot be granted access to more than 1,000 workspaces (supported limit), and one workspace per identity is optimal for performance.

Rules:
- A single **parent service principal** creates lightweight per-tenant **profiles** via the `Profiles` REST API. One Microsoft Entra app registration covers the whole solution.
- Profiles are **local to Power BI** (Microsoft Entra does not know them), so the app needs no special Entra permissions to create them. Consequence: a profile cannot be added to a Microsoft Entra group, and external data sources cannot recognize a profile as the connecting identity.
- Profiles are **first-class security principals** in Power BI: they hold workspace roles and own semantic models and data source credentials → true tenant isolation.

Calling-context rule (from the guidance):
- Use the **parent service principal** to **create/manage profiles**.
- Use the **profile** for everything else (create/set up workspaces, import files, set parameters/credentials, retrieve metadata, generate embed tokens).

Mechanics:
- Pass the parent service principal's access token in the **Authorization** header.
- Add header **`X-PowerBI-profile-id`** with the profile's ID. The .NET SDK accepts the profile ID in the `PowerBIClient` constructor.

```csharp
// create a profile as the parent service principal
var profile = pbiClient.Profiles.CreateProfile(new CreateOrUpdateProfileRequest("Contoso"));

// later: act as that profile
var pbiClientForProfile = new PowerBIClient(serviceRootUri, tokenCredentials, profile.Id);
var workspaces = pbiClientForProfile.Groups.GetGroups();
```

## Client-side (JavaScript) embedding config

Both scenarios embed with the `powerbi-client` library into a `div`. The only scenario difference below is `tokenType`:

```javascript
var models = window['powerbi-client'].models;
var config = {
  type: 'report',
  id: reportId,
  embedUrl: embedUrl,
  accessToken: token,
  permissions: models.Permissions.All,
  tokenType: models.TokenType.Embed, // app owns data; use models.TokenType.Aad for embed for your organization
  viewMode: models.ViewMode.View
};
powerbi.embed(reportContainer, config);
```

## PowerShell / package commands

The docs use PowerShell (or the IDE terminal) mainly for setup; example commands seen in the tutorials:

```powershell
# add the .NET SDK + auth packages
dotnet add package Microsoft.Identity.Web
dotnet add package Microsoft.PowerBI.Api

# node sample
npm install
npm start

# python sample
pip3 install -r requirements.txt
```

Acquire a Microsoft Entra token ad hoc with an external client (e.g. Bruno) against `https://login.windows.net/{tenantId}/oauth2/token`.
