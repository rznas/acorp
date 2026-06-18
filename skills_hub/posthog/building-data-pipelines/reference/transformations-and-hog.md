# Transformations & hog functions details

Reshaping events before they're stored, and customizing any pipeline block with Hog. See SKILL.md for the overview.

## Table of contents
- [What transformations can and can't do](#what-transformations-can-and-cant-do)
- [Drop events transformation](#drop-events-transformation)
- [Setup, testing, monitoring](#setup-testing-monitoring)
- [Custom Hog transformations](#custom-hog-transformations)
- [Hog functions API](#hog-functions-api)
- [Writing hog functions with PostHog AI](#writing-hog-functions-with-posthog-ai)

## What transformations can and can't do
Transformations run **during ingestion, before the event is fully stored**.
- **Can:** anonymize sensitive data, enrich with geo, drop/modify properties, filter bot/test traffic, enforce a schema, label events, reduce billing.
- **Cannot:** access `person` or `group` properties (not resolved yet — use a realtime destination instead), make external HTTP/`fetch` calls.
- **Globals available:** only `event` (uuid, event, distinct_id, properties, timestamp) and `project` (id, name, url).
- **Note:** transformations don't apply to error-tracking exceptions.
- **Free**, no event limit. A transformation that errors / is too slow / uses too much memory for a prolonged period is auto-quarantined and disabled; fix and re-enable.

## Drop events transformation
Permanently discard matching events before ingestion. **Dropped events do not count toward usage/billing** — a fast lever to cut volume without a code deploy, and useful for excluding specific bundled autocapture events. Dropped data is unrecoverable and never appears in analytics, cohorts, or exports.

Filter logic:
- **Match events** filters → **OR** (event matches any).
- **Property filters** → **AND** (all conditions must hold), applied after event matching.
- You can attach property filters directly to an event-match filter (incl. "All events").

Monitor via the Metrics tab (processed vs dropped counts).

## Setup, testing, monitoring
Data pipeline → **Transformations** → search a template (e.g. Drop events) → **+ Create** → configure filters → **Create & Enable**. Every transformation has:
- **Testing**: run with the example event, a real event from your instance, or custom JSON — before enabling.
- **Metrics** + **Logs** tabs.
- **History** audit trail (who/what/when) on platform-package projects.

## Custom Hog transformations
Start from the "Custom transformation" template or **edit source code** of an existing one. A transformation receives `event` and returns a modified copy (or `null`/`undefined` to **drop the event permanently** — use with extreme care).

Pattern (the global `event` is immutable, so copy it):
```hog
let returnEvent := event
returnEvent.properties := returnEvent.properties ?? {}
returnEvent.properties.my_custom_property := "new value"
return returnEvent
```

Anonymize-IP example (mask last octet, keep geo):
```hog
if (empty(event.properties?.$ip)) { return event }
let parts := splitByString('.', event.properties.$ip)
if (length(parts) != 4) { return event }
let returnEvent := event
returnEvent.properties.$ip := concat(parts[1], '.', parts[2], '.', parts[3], '.0')
return returnEvent
```

Guidance: start from a close template; push logic into built-in **filters**/**inputs** (cleaner + faster); use `inputs` for config (mark secrets → encrypted at rest); keep it short (tight time/memory limits); debug with `print()` (Logs tab); no external calls. The same `show/edit source code` + inputs/secrets model applies to **destinations** (which *can* `fetch`, max 5 calls).

## Hog functions API
Create/manage transformations and destinations programmatically (good for templating new projects). Enable the `Hog functions` scope on a personal API key, then POST to `/api/environments/:project_id/hog_functions` with a `type` (`transformation`/`destination`) and a `template_id`. Find template IDs via `/api/projects/:project_id/hog_function_templates?types=transformation`.

## Writing hog functions with PostHog AI
[PostHog AI](/docs/posthog-ai) generates Hog functions from natural language for transformations, filters, and destinations — choosing inputs, event-property access, and API calls. Examples a PM can paste: "Create a destination that sends high-value events to Slack", "Filter out internal traffic by IP", "Parse timestamps into day/week/month", "Add a `user_tier` property based on plan type". Iterate with follow-ups ("also send the user's email"). Tips: describe input→output, name real events/properties, name the destination service, and cover edge cases (nulls).

> Self-hosted note: PostHog AI / Max is a hosted LLM feature; it may be unavailable or require extra configuration on self-hosted instances. The visual templates, custom Hog editor, and Hog functions API remain available. See the **using-posthog-ai-assistant** skill.
