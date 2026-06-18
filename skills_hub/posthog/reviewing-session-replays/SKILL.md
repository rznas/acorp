---
name: reviewing-session-replays
description: Guides a product manager through PostHog Session Replay to qualitatively understand user behavior, validate funnel drop-offs, and diagnose friction. Covers watching recordings, the activity/inspector panel, finding and filtering replays, saved filters and collections, controlling which sessions record (sampling, URL/event/exception triggers, feature flags, trigger groups, minimum duration), console and network capture, canvas, mobile (web vs Android/iOS/React Native/Flutter), privacy masking, retention, sharing/embedding, and linking replays to funnels, experiments, feature flags, and error tracking. Use for questions like "why are users dropping off", "what does friction look like", "watch recordings of users who didn't convert", "show me rage clicks", "how do I record only checkout", or "is my data masked".
---

# Reviewing session replays

Session replay is PostHog's qualitative layer: it answers *why* the numbers in your funnels, experiments, and dashboards look the way they do. As a PM you rarely configure SDKs — you watch recordings, filter to the right ones, and turn what you see into product decisions. This skill maps PM questions to the replay features that answer them.

> Self-hosted note: core replay (watching, filtering, collections, triggers, privacy, retention, sharing) is fully available self-hosted. The natural-language **PostHog AI / Max** helpers (find replays, summarize sessions) are hosted LLM features — availability may differ on self-hosted; confirm AI is configured for your instance. See `using-posthog-ai-assistant` and `reference/ai-and-mcp.md`.

## PM question -> feature map

| You want to know... | Use | Where it lives |
| --- | --- | --- |
| Why are users dropping off at a funnel step? | Click the drop-off data point in a funnel -> watch those users' recordings | Insight -> persons modal -> **Watch recording** / **Recordings** tab |
| What does friction look like? | Filter by rage clicks, dead clicks, errors; or the **Frustration signals** built-in collection | Replay page -> filters / collections |
| Which sessions had a specific error? | Filter by exception event, or jump from Error tracking to the replay | Error tracking issue -> linked replay |
| How did A vs B behave (experiment/flag)? | Filter replays by feature flag variant | Replay filters -> feature flag |
| What did this user actually do, fast? | Activity/inspector panel + AI session summary | Player -> **Activity** panel; AI summary sidebar |
| Did a slow/failed API call hurt UX? | Network waterfall in the inspector | Player -> Activity -> Network tab |
| Save a recurring view for the team | Saved filters (dynamic) or collections (curated) | Replay page |

## Watching a recording (what the player gives you)

Open the [Replay page](https://app.posthog.com/replay) or click any insight data point to land on the persons behind that number, then **Watch recording**. This is the primary workflow for **validating funnel drop-offs**: drill into a funnel step, watch converters vs droppers.

Inside the player a PM should know to:
- **Open the Activity panel** — the synced event timeline. Jump to any event; see console logs/warnings/errors at the moment they fired; inspect the **network waterfall**, page performance score, and asset breakdown. This is how you connect "user got stuck" to a concrete cause.
- **AI summary segment markers** appear on the timeline — green = success, red = failure. Click to jump to that moment.
- **Skip inactivity** and change playback speed (in the `...` menu) to scan long sessions.
- **Capture and share moments**: screenshot (`S`), 5/10/15s GIF or MP4 clip (`X`), comment (`C`, or **Add as task** to track a fix), emoji reaction (`e`). Use clips/screenshots as visual evidence in Jira/Linear bug reports.
- **Inspect DOM** and **View heatmap** for deeper UI inspection.

Full keyboard shortcuts and player controls: see `reference/watching-and-finding.md`.

## Finding the right replays

You usually can't watch everything — narrow first.

- **Show filters** (manual): filter by date, active duration, events, person/group properties, console logs, feature flags, rage clicks, errors, and more.
- **Saved filters** (dynamic): save a filter; new matching replays auto-flow into it and it's shared with the whole project via the **Shared filters** tab. Editable name + events without recreating.
- **Collections** (curated, permanent): `+ Add to collection` on a replay to hand-pick recordings for a product review or to share. Managed on the [collections page](https://app.posthog.com/replay/playlists).
- **Built-in collections** (auto, dynamic): Watch history, Recordings with comments, Shared recordings, Exported recordings, Expiring soon, and **Frustration signals** (multiple rage clicks or exceptions in last 7 days) — a fast entry point for "show me friction".

**Decision support**: a saved filter for "checkout drop-off + error" becomes a standing triage queue; a collection becomes the evidence pack for a roadmap discussion.

Sorting (by errors, activity, timestamp), autoplay modes, hide-watched, console-log search syntax, and full filter details: `reference/watching-and-finding.md`.

## Controlling which sessions record

PMs should spec recording scope to balance coverage vs cost/noise. Configured at the [replay ingestion settings page](https://app.posthog.com/replay/settings#selectedSetting=replay-triggers) (creating/editing trigger groups needs project admin). Default = record everything.

| Goal | Mechanism |
| --- | --- |
| Record a slice of all traffic | **Sampling** (e.g. 20% — deterministic by session ID; you can't pick which sessions) |
| Only record a key flow | **URL trigger** (e.g. /checkout) — buffers up to ~1 min before the trigger so you see how they arrived |
| Record around a behavior | **Event trigger** (e.g. a custom event) |
| Record sessions that error | **Event trigger on the exception event** (needs Error tracking) |
| Record only a flag's audience | **Feature flag** gate on recording |
| Drop junk bounces | **Minimum duration** (e.g. ignore <2s sessions) |
| Combine several rules cleanly | **Trigger groups** — each group has its own URL/event/flag/sample/min-duration + ANY/ALL match; a session records if it matches any group |

Watch the trap: **100% sampling with "any" matching records every session**, defeating your other conditions — lower the rate or use "all". Recommendation: start at 100% sampling, then dial down. Full trigger-group, match-type, minimum-duration (legacy vs strict), and billing-limit detail: `reference/recording-controls-and-privacy.md`.

## Console, network, and canvas capture

- **Console logs** (logs/info/warn/error): debugging context for what the user's browser/app was doing. View in the inspector's **Console** tab with rich search tokens (fuzzy, exact `=`, include `'`, exclude `!`, prefix `^`, suffix `$`).
- **Network performance**: request URLs + performance timing are captured when enabled; optionally method, status, headers, and request/response bodies. Lets you tie "the page felt slow" or "the button did nothing" to a real slow/failed call. Enabled in project settings.
- **Canvas** (2D/WebGL): off by default and **cannot be masked**, so only enable once you've confirmed no PII. Captured at ~4 fps.

Setup specifics live in `reference/recording-controls-and-privacy.md`.

## Privacy and masking (what you must confirm before sharing)

Masking runs in the browser/app, so masked data never reaches PostHog. PM-relevant defaults and levers:
- **All input elements are masked by default** (emails, passwords). General non-input text is **not** masked by default.
- Engineers can mask/unmask via `maskAllInputs`, `maskTextSelector`, the `ph-no-capture` CSS class (element becomes a blank block on playback), and selector/function callbacks — including selective "only reveal what's marked safe".
- **Network bodies/headers**: a deny-list (authorization, cookie, x-api-key, etc.) is never captured; credit-card and SSN-looking bodies are auto-redacted; a callback can scrub more. URLs/query strings can be redacted too.
- **Sharing carries no PII guarantee** — only share recordings you're sure are clean, or embed them where only authorized users can see.

Masking config patterns and the network deny-list: `reference/recording-controls-and-privacy.md`.

## Mobile session replay

GA on Android, iOS, React Native, and Flutter (screen recordings, network, logs, touches). Default config is restrictive with automatic masking.
- **Wireframe mode** (default, native Android/iOS): renders the view hierarchy as a wireframe — close enough to read behavior, not pixel-perfect.
- **Screenshot mode** (optional Android/iOS; **always** used by React Native and Flutter): real screenshots — higher fidelity but **you must mask sensitive views**.
- Sampling and minimum duration are supported per-SDK with version minimums (see reference).

## Retention, sharing, and exporting

- **Retention** is set in replay settings and only applies to *new* recordings; max depends on plan (Free up to 30 days, Pay-as-you-go 90 days, Boost/Scale 1 year, Enterprise 5 years). Expired recordings cannot be restored — useful for data-protection compliance.
- **Export to JSON** preserves an individual recording past expiry and can be re-imported for playback.
- **Sharing**: every replay has a **private link** (teammates only) and a **public link** (anyone; embeddable via `<iframe src="…/embedded/…">`). Collections share a curated set. Auto-share to Intercom/Crisp or via the API to attach replays to support tickets. Note: events don't render inside a shared iframe.

## Tying replays to the rest of PostHog

- **Funnels / paths / retention**: click a data point -> playlist of those exact users' replays. No ID matching — same data layer. This is the core *qualitative validation of quantitative drop-offs* loop.
- **Experiments**: metrics say which variant won; replays (esp. AI summaries per variant) say *why* — different navigation, confusion points, feature usage.
- **Feature flags**: filter replays by variant to see A vs B behavior before a full rollout.
- **Error tracking**: from an exception, jump straight to the replay to see what the user did before/after — no reproduction needed.

## PM workflow: validate a funnel drop-off with replays

1. Open the funnel; click the drop-off count at the failing step.
2. In the persons modal, watch a handful of **dropped** users' recordings; then a few **converters** as a baseline.
3. In the Activity panel, look for rage/dead clicks, console errors, and slow/failed network calls at the drop point.
4. Filter the Replay page (or use the Frustration signals collection) to confirm the pattern at scale; save it as a saved filter.
5. Capture clips/screenshots; add comments or **Add as task** for the fix.
6. Optionally ask PostHog AI to summarize the matched sessions to quantify the pattern (see `reference/ai-and-mcp.md`).
7. Hand the collection + clips to engineering as the evidence pack.

## Related skills

- `using-posthog-ai-assistant` — natural-language find/summarize replays (Max) and the MCP server.
- `analyzing-product-funnels` — building the funnels you drill into.
- `running-experiments` — pairing experiment metrics with replay summaries.
- `managing-feature-flags` — variant gating for both rollout and replay filtering.
- `tracking-and-triaging-errors` — error tracking that links to replays.
