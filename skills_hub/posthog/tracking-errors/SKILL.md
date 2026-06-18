---
name: tracking-errors
description: Guides product managers on PostHog Error Tracking to answer what is breaking, how big an error's impact and volume is, who is affected, and whether a release caused a regression. Use for questions about exceptions and issues, error grouping and fingerprints, assigning and triaging issues, resolving/suppressing, alerts and spike detection, monitoring and searching issues, releases and source maps, rate limiting and error costs, and linking errors to session replay. Triggers include "what's our most common crash", "how many users hit this error", "did the last deploy cause new errors", "alert me when errors spike", "who should own this bug", and "stop noisy errors from blowing our bill".
---

# Tracking errors with PostHog Error Tracking

PostHog Error Tracking captures application errors as `$exception` events, groups them into **issues**, and lets a PM see what is breaking, how badly, and whether a release caused it. Because exceptions are normal PostHog events, they connect to session replay, product analytics, feature flags, and surveys.

> Self-hosted note: the docs do not call out cloud-only restrictions for core Error Tracking (issues, grouping, assignment, alerts, spike detection, rate limiting, releases). AI/MCP helpers (Fix with AI, MCP debugging) and managed real-time CDP destinations may behave differently or need extra config on self-hosted; verify in your instance. When in doubt, availability may differ on self-hosted.

## Core model: exceptions -> fingerprints -> issues

- **Exception** = one `$exception` event. Has properties like `$exception_types`, `$exception_values` (message), `$exception_sources`, `$exception_functions`, `$exception_level`, `$exception_handled`, plus standard event props (`$browser`, `$device_type`, `$current_url`, `$feature_flag`).
- **Fingerprint** = a hash PostHog builds from exception type + stack trace (or message). One fingerprint -> exactly one issue.
- **Issue** = a group of similar exceptions sharing a fingerprint. This is the unit a PM triages: it has a status, assignee, volume, affected users/sessions, and external links.

Grouping precedence (first match wins): client-side `$exception_fingerprint` -> custom grouping rules in PostHog -> automatic fingerprinting. Details in [reference/grouping-and-fingerprints.md](reference/grouping-and-fingerprints.md).

Where it lives: the **Error tracking** product in the left nav (`app.posthog.com/error_tracking`), with a settings/configuration page for grouping, assignment, alerting, spike detection, and rate limits.

## PM question -> feature map

| PM question | Feature / where | What it tells you |
| --- | --- | --- |
| What is breaking right now? | Issues list, sort by **Last seen** or **Occurrences** | Most recent / highest-volume problems |
| Which errors hurt the most users? | Issues list, sort by **Users** or **Sessions** | Impact-ranked issues (not just noise) |
| How big is one error? | Issue detail: occurrences, unique users, sessions, first/last seen | Volume + reach + age of a single problem |
| Did the last deploy cause new errors? | **Releases** + filter issues by release; **First seen** date | Regressions tied to a version/commit |
| Alert me when an error spikes | **Spike detection** + alerts | Catch incidents/regressions in real time |
| Notify the team of new/reopened issues | **Alerting** (Slack/Discord/Teams/webhook) | Route new issues to the right channel |
| Who should fix this? | Manual assign + **auto-assignment rules** (to user or role) | Ownership and team routing |
| Is this fixed / should we ignore it? | Issue **status**: Active / Resolved / Suppressed | Triage state; resolved auto-reopens on recurrence |
| Reduce noise / control error costs | **Rate limiting**, suppression, billing limits | Cap volume and spend |
| What did the user do before crashing? | **Session replay** tab / View recording on an exception | Reproduce and root-cause |
| Spot error patterns over time | `$exception` in trends/funnels/retention | Prioritize with product analytics |
| Roll back for affected users | Feature flag targeting users who hit an exception | Mitigate while fixing |

## Reading impact and volume (the PM's daily job)

In the issues list, sort to find what matters:

| Sort by | Use when |
| --- | --- |
| Last seen | Triage the freshest problems |
| First seen | Find newly introduced issues (regressions) |
| Occurrences | Raw volume — good general signal |
| Users | True customer impact — prioritize by reach |
| Sessions | Sessions disrupted |

Decision rule: high **occurrences** but low **users** = often one noisy client or loop (candidate for rate limiting/suppression). High **users** = real customer pain, prioritize the fix. Open an issue to see the stack trace, properties, affected sessions, and a filterable list of every exception in the group.

Search/filter the list with freeform text (matches exception type, message, stack function names, file paths) or exact property filters (e.g. `library equals web`, `http_referer is set`). Combine with `AND`. Full search behavior in [reference/monitoring-and-search.md](reference/monitoring-and-search.md).

## Triage workflow (checklist)

1. Filter issues to **Active** + **Assigned to** your team (or unassigned).
2. Sort by **Users** to surface highest-impact problems first.
3. Open an issue: read the stack trace + properties; watch a **session replay** to confirm/reproduce.
4. Decide status:
   - **Resolved** — fixed; auto-reopens if it recurs (great regression signal).
   - **Suppressed** — won't action (noisy/unhelpful). Suppressing **drops** associated exceptions going forward; pair with client-side suppression to also stop capture.
   - **Active** — needs work.
5. **Assign** to a teammate or a role (team). Optionally **Create issue** in GitHub/GitLab/Linear/Jira to track the fix.
6. Use **bulk actions** (checkboxes) to mark-as / assign / merge / hide many issues at once.

## Assigning and routing ownership

Assign manually from the list or issue page, or set **auto-assignment rules** (settings -> auto-assignment) that match exception properties (`$exception_types`, `$exception_functions`, `$exception_sources`, `$browser`, `$current_url`, `$feature_flag`, custom props) and route to a **user** or **role**. Rules run during ingestion against the unminified stack trace; first matching rule wins, so order specific-before-general. A common pattern: auto-assign by area, then alert that assignee on new/reopened issues. See [reference/assigning-and-alerting.md](reference/assigning-and-alerting.md).

## Alerts and spike detection (catching regressions)

Two distinct alert types — pick by the question:

- **Issue created or reopened** — fires when a *new* problem appears or a resolved one comes back. Best for "tell us about new bugs / regressions after deploy." Destinations: Slack, Discord, Teams, HTTP webhook. Filter by issue properties/assignee so teams only get their own.
- **Spike detection** — fires when an *existing* issue's rate jumps above its rolling baseline (e.g. a bad deploy makes a known error surge). Configure **minimum threshold**, **multiplier**, and **snooze**. Best for "alert us when something blows up."

Also available: `$exception` real-time CDP destinations, and **trend alerts** on `$exception` volume increase/decrease (emails). Setup details and tuning in [reference/assigning-and-alerting.md](reference/assigning-and-alerting.md).

## Releases and regressions

Create a **release** per deploy (automatic when you upload source maps via `posthog-cli sourcemap upload`, or via the Next.js/Nuxt packages). Releases attach version, commit/build metadata, and source maps to exceptions. PM value:

- Filter issues by release and read **First seen** to confirm a deploy introduced an error.
- With a GitHub/GitLab integration + release, stack traces get **View commit** links — jump from error to the exact line/commit.
- Source maps make stack traces (and therefore fingerprints/grouping) accurate.

A PM specs these; engineering wires the CLI. Don't worry about the upload snippets. Integration setup is in [reference/releases-and-integrations.md](reference/releases-and-integrations.md).

## Controlling noise and cost

Error Tracking is usage-priced (first 100K exceptions/month free, then per-exception). Levers a PM can ask for:

- **Rate limiting** (settings -> rate limits): server-side token-bucket caps that drop excess exceptions at ingestion *before billing*. Two independent kinds: **project-wide** (total safety cap) and **per-issue** (each issue gets its own bucket so one noisy issue can't drown others). Configure max exceptions per 15/30/60 min.
- **Suppress** noisy issues (drops their exceptions) and add client-side suppression/`before_send`/burst protection to stop capture at the source.
- Set **billing limits** to avoid surprise charges.

More in [reference/rate-limiting-and-cost.md](reference/rate-limiting-and-cost.md).

## Linking to session replay and other products

- **Session replay**: from an exception, click the **Session** tab or **View recording** to watch what the user did before/after the crash — the fastest path to reproduce and root-cause.
- **Product analytics**: build trends/funnels/retention on `$exception` to spot patterns and prioritize; see exceptions in a person's event history.
- **Feature flags**: target/roll back users who hit a specific exception.
- **Logs**: backend issues surface in the Logs viewer's **Related errors** tab when logs are linked to sessions.

## AI and MCP helpers (PM value)

- **Fix with AI** — every issue has a "Fix with AI" button that copies a ready prompt (full stack trace + instructions) to paste into a coding assistant for diagnosis and a suggested fix.
- **MCP debugging** — the PostHog MCP server lets AI agents triage issues, fetch stack traces, change issue status, and create assignment/grouping rules from the editor (Cursor, Claude Code, etc.). PM value: faster diagnosis and "show me the most common / highest-impact errors" in natural language.

> Self-hosted note: these AI/MCP features depend on external AI clients and the MCP server; confirm setup and availability on your self-hosted instance.

For broader AI assistant usage, see the **using-posthog-ai-assistant** skill.

## Related skills

Related PostHog skills: **using-session-replay** (watch what users did before a crash), **building-insights-and-dashboards** / **analyzing-product-analytics** (trend/funnel patterns on `$exception`), **managing-feature-flags** (roll back affected users), **using-posthog-ai-assistant** (Fix with AI, MCP), and **managing-data-pipelines** (real-time `$exception` destinations).
