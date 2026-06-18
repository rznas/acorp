# Monitoring, searching, and reading issues

Table of contents
- [Two monitoring workflows](#two-monitoring-workflows)
- [Sorting issues](#sorting-issues)
- [Filtering and status](#filtering-and-status)
- [Search bar: two modes](#search-bar-two-modes)
- [Chained exceptions in search](#chained-exceptions-in-search)
- [Issue detail page](#issue-detail-page)
- [Filtering exceptions within an issue](#filtering-exceptions-within-an-issue)
- [Search performance tips](#search-performance-tips)
- [Discovering errors from logs](#discovering-errors-from-logs)

Lives at `app.posthog.com/error_tracking`.

## Two monitoring workflows

- **Exploring** for impactful/problematic areas -> use **sorting**.
- **Working your queue** -> filter by **Assigned to** (you or your team), especially when paired with auto-assignment + alerts.

## Sorting issues

| Property | Meaning | PM use |
| --- | --- | --- |
| Last seen | Most recent exception | Triage freshest problems |
| First seen | Oldest exception | Find newly introduced issues |
| Occurrences | Count of exceptions | General volume signal |
| Users | Unique users affected | True impact — prioritize |
| Sessions | Unique sessions affected | Sessions disrupted |

Last seen + occurrences give a general sense; users + sessions (combined with filters) surface the most impactful issues.

## Filtering and status

Filter the list by **status** (Active / Resolved / Suppressed) and by **Assigned to**. Standard workflow: show **Active** issues assigned to you/your team, then fix or suppress each.

## Search bar: two modes

Results match against properties of the exceptions in each issue; you may see "unrelated" issues because a token matched somewhere in the group's stack traces (e.g. searching `schema` surfaces `RefreshError` because `get_schema` is in the trace).

1. **Exact property filtering** — like property filters elsewhere in PostHog: `where 'library' equals 'web'`, `where 'http_referer' is set`. Click the property name to add.
2. **Freeform text search** — tokenizes your text; an exception matches if *each* token appears in one of: exception type, message, stack-trace function names, or file paths. Quote a phrase to match it exactly: `"Cannot read property 'name' of undefined" myfile.js`. Ungrouped tokens match if each appears *somewhere*, which can be surprising.

Property filters and freeform search combine with **AND**.

## Chained exceptions in search

For chained exceptions, all types/messages/functions/file paths go into one list; a token matches if it appears in any one of them.

## Issue detail page

Shows, for the **currently selected exception**: stack trace, properties, and related sessions. Plus issue-level name, description, status, assignee, and external tracking links, and a filterable list of every exception in the issue. Selecting a different exception updates the top panels.

Tabs: **Stacktrace** (where the error occurred, line + call sequence), **Properties** (event metadata), **Session** (replay), and **Fix with AI** (copy a debugging prompt).

## Filtering exceptions within an issue

Use the same search interface inside an issue to filter its exception list — e.g. show only exceptions where `http_referer` is set — useful when some occurrences carry debug info others lack. If you reuse a filter set often, turn it into an alert.

## Search performance tips

- Limit the time range (7 days is usually enough for trend sense).
- Prefer freeform search over property filters (less data processed, faster).
- Report queries that time out or exceed ~30s in-app.

## Discovering errors from logs

When logs are linked to sessions, any log entry's **Related errors** tab surfaces error-tracking issues from the same session — useful for backend issues captured alongside logs.
