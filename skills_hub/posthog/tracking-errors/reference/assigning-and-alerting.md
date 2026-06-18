# Assigning issues, alerting, and spike detection

Table of contents
- [Manual assignment](#manual-assignment)
- [Assigning to a team via roles](#assigning-to-a-team-via-roles)
- [Auto-assignment rules](#auto-assignment-rules)
- [Alert type 1: issue created or reopened](#alert-type-1-issue-created-or-reopened)
- [Alert type 2: spike detection](#alert-type-2-spike-detection)
- [Other alerting options](#other-alerting-options)
- [Recommended PM pattern](#recommended-pm-pattern)

## Manual assignment

Assign from the issue list (the **unassigned** selector under each issue) or the issue detail page (**Assignee** selector). Assign to a user or a role; click again to remove. Assignment drives both filtering ("show issues assigned to me/my team") and team-specific alerting.

## Assigning to a team via roles

Create a **role** in organization settings (`settings/organization-roles`) to represent a team, then assign issues to that role instead of an individual.

## Auto-assignment rules

Configured in error tracking settings -> auto-assignment (also creatable via the PostHog MCP server). Rules are evaluated **during ingestion** against the exception that created the issue, so the stack trace is unminified (you can filter on function name / source file).

Match **any** or **all** of a set of filters, route to a user or role. When a property has multiple values, the filter is true if it matches **any** of them.

Common properties:

| Property | Event property |
| --- | --- |
| Exception type | `$exception_types` |
| Exception message | `$exception_values` |
| Exception function | `$exception_functions` |
| Exception source | `$exception_sources` |
| Exception was handled | `$exception_handled` |
| Device type | `$device_type` |
| Browser | `$browser` |
| Current URL | `$current_url` |
| Feature flag | `$feature_flag` |

Plus custom properties (e.g. `params_received`). Rules evaluate in order; first match wins — order **specific before general**. Rules can be **disabled** by ingestion errors (banner shows the message); edit + save to re-enable.

## Alert type 1: issue created or reopened

Settings -> Alerting -> **New notification**. Fires when an issue is first created or a resolved issue reopens. Destinations: Slack, Discord, Teams, HTTP webhook (may require setting up the Slack integration or pasting a webhook URL). Test with **Test function**, then **Create & enable**.

Filter alerts by issue properties / **assignee** so a team is only notified about its own auto-assigned issues. Best for: surfacing new bugs and regressions after a deploy.

## Alert type 2: spike detection

Different mechanism: instead of firing on a *new* issue, spike detection fires when an **existing** issue's rate surges past its historical baseline (e.g. a known error explodes after a bad deploy).

How it works: PostHog tracks each issue's exception rate using a rolling baseline, measuring 5-minute buckets each hour. When a bucket exceeds the baseline by the configured **multiplier**, a spike event fires. New issues without enough history fall back to an average baseline derived from your other issues (no warm-up needed).

Configure in settings -> spike detection:

| Option | Effect |
| --- | --- |
| Minimum threshold | Issues below this many occurrences per 5-min bucket never spike (kills low-volume noise) |
| Multiplier | How far above baseline triggers a spike (higher = only dramatic surges) |
| Snooze duration | Minutes to stay quiet after a spike fires (prevents alert fatigue in sustained incidents) |

Wire spike notifications to Slack/Discord/Teams/webhook. Best for: real-time incident/regression detection on errors you already know about.

## Other alerting options

Since errors are `$exception` events, event-triggered features apply:

- **Real-time CDP destinations** — data pipelines -> Destinations -> add an event matcher for `$exception`, filter properties, set triggers; pushes errors to external tools as ingested.
- **Trend alerts** — build a trend on `$exception`, then Alerts -> New alert for volume value/increase/decrease (sends email).

## Recommended PM pattern

1. Auto-assign issues by area (function/source/URL/flag) to the owning role.
2. Add an **issue created/reopened** alert filtered to each assignee -> their Slack channel.
3. Add **spike detection** alerts for real-time surges after deploys.
4. Use trend alerts for slow-burn volume creep on `$exception`.
