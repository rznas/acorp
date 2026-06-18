# Recording controls, capture, and privacy — reference

Deep detail for what gets recorded and how it's protected. See SKILL.md for the overview. Configured mainly at the [replay ingestion settings page](https://app.posthog.com/replay/settings#selectedSetting=replay-triggers) and project settings.

## Table of contents
- [Sampling](#sampling)
- [URL and event triggers](#url-and-event-triggers)
- [Trigger buffer](#trigger-buffer)
- [Feature flag gating](#feature-flag-gating)
- [Combining controls and the 100% trap](#combining-controls-and-the-100-trap)
- [Trigger groups](#trigger-groups)
- [Minimum duration](#minimum-duration)
- [Billing limits](#billing-limits)
- [Console log capture](#console-log-capture)
- [Network capture](#network-capture)
- [Canvas capture](#canvas-capture)
- [Privacy and masking (web)](#privacy-and-masking-web)
- [Mobile capture modes](#mobile-capture-modes)
- [Retention](#retention)

## Sampling

Record a percentage of all sessions. Deterministic by session ID (hashed to 0–1, recorded if below the rate), so the decision is stable across page refreshes. You cannot choose *which* sessions — only roughly what fraction. Recommendation: start at 100%, decrease as needed. SDK minimums: web 1.85.0+, Android 3.34.0+, iOS 3.42.0+, React Native 4.37.0+, Flutter 5.26.0+.

## URL and event triggers

- **URL trigger**: start recording once the user hits a matching page; recording continues after they leave.
- **Event trigger** (posthog-js 1.186.0+): start once a chosen event fires; continues afterward.
- **Exception trigger**: with Error tracking, select the exception event as the event trigger to record sessions that error.

## Trigger buffer

While waiting for a trigger ("trigger pending") the client keeps an in-memory buffer:
- A full snapshot is taken **once per minute**.
- Only the most recent snapshot up to the trigger is kept — so you get **up to ~1 minute** of pre-trigger context (actual amount depends on timing).
- Full page navigations (refreshes) clear the buffer and start over.

## Feature flag gating

Record only users for whom a flag is enabled:
1. Create a boolean or multivariate flag.
2. Replay ingestion settings -> **Enable recordings using feature flag** -> link the flag.

## Combining controls and the 100% trap

Since web SDK 1.238.0 you choose how multiple triggers combine:
- **Any matching**: record if any condition matches (e.g. 20% sampled OR exception OR on /checkout).
- **All matching**: record only if all match (e.g. 20% of /checkout sessions that also had an exception).

Caution: **100% sampling + "any" matching records every session**, making your other conditions ineffective. Fix by lowering the sample rate or switching to "all".

## Trigger groups

A more flexible model. Each group has its own URL triggers, event triggers, feature flags, sample rate, minimum duration, and ANY/ALL match type. Multiple groups can be active; a session records if it matches **any** group, with within-group conditions combined by that group's match type. Example groups:
- All /checkout sessions at 100%.
- 10% of all sessions site-wide.
- Any session with an exception.

Creating/editing/deleting groups needs **project admin**; non-admins can view only. Deleting the last group auto-creates a default "Record all sessions" 100% group. Legacy settings can be auto-converted via **Create from legacy triggers**.

## Minimum duration

Skip sessions shorter than a threshold (e.g. drop <2s bounces). SDK minimums: web 1.291.0+, iOS 3.53.0+, Android 3.44.0+, Flutter 5.24.3+.

Two web modes via `strictMinimumDuration`:
- **Legacy (default, `false`)**: measured against total session age. After a page refresh the buffer may clear, so you can miss the start of the session.
- **Strict (`true`, 1.291.0+)**: measured against actual continuous buffered data on a single page; more accurate at excluding short sessions but may drop more data from users who bounce across pages. Will become the default later.

iOS/Android/Flutter enforce by buffer length, similar to web strict mode.

## Billing limits

Set a [billing limit](/docs/billing/limits-alerts); PostHog stops ingesting recordings at the limit — the safety valve against surprise replay costs.

## Console log capture

Captures console logs, info, warnings, and errors from the browser/app for debugging context. View in the inspector's Console tab; search syntax is in watching-and-finding.md.

## Network capture

Enabled in project settings. Always captured when on: request URL and performance timing. Optionally: request method, response code, request/response headers, request/response bodies.

Sensitive-data handling:
- **Header deny-list** never captured (even with a mask function): authorization, x-forwarded-for, cookie, set-cookie, x-api-key, x-real-ip, remote-addr, forwarded, proxy-authorization, x-csrf-token, x-csrftoken, x-xsrf-token.
- Bodies that look like credit-card numbers or SSNs are auto-redacted.
- Provide `maskCapturedNetworkRequestFn` to scrub more; the same callback also redacts the page URL/query string captured in the snapshot. Runs in the browser, so redacted values never leave the device.

Capturing network data can affect performance on high-volume sites — sample or limit if needed.

## Canvas capture

For 2D/WebGL canvas elements. **Off by default and cannot be masked** — only enable after confirming no PII (replay settings, global). Requires JS SDK v1.105.7+. Captured at ~4 fps by default; `captureCanvas.canvasFps` (0–12) and `canvasQuality` ("0"–"1") tune frequency/quality vs performance.

## Privacy and masking (web)

Masking runs in the browser; masked data never reaches PostHog.
- **Inputs masked by default** (`maskAllInputs`); keep at least `password: true` if you disable global input masking.
- **General text not masked by default**; `maskTextSelector: "*"` masks all non-input text, or use CSS selectors (e.g. `.email, #sensitive`). Masking applies to children, so `:not` selectors don't work.
- `maskInputFn` / `maskTextFn` callbacks for fine control (e.g. only mask email-looking text, or unmask search boxes).
- **`ph-no-capture` CSS class**: element is replaced by a same-size blank block on playback (also blocks autocapture on it). Tell teammates so they don't think the product is broken.
- **Selective privacy**: mask everything, then unmask only elements explicitly marked safe (e.g. `data-record="true"`).
- Sensitive third-party screens (e.g. payment iframes) have additional handling.

Common configs: maximum privacy (`maskAllInputs: true, maskTextSelector: "*"`); email/password-only masking; selective reveal.

## Mobile capture modes

- **Wireframe mode** (default; native Android/iOS): view hierarchy rendered as a wireframe — behavior-accurate, not pixel-perfect; restrictive automatic masking.
- **Screenshot mode** (`screenshot` Android / `screenshotMode` iOS; **always** for React Native and Flutter): real screenshots — higher fidelity, so you must mask sensitive views yourself.
- Mobile captures screen recordings, network, logs, and touches.

## Retention

Set in the Data retention section of replay settings; applies only to **new** recordings (existing ones keep their original period). Max by plan: Free up to 30 days, Pay-as-you-go 90 days, Boost 1 year, Scale 1 year, Enterprise 5 years. Deletion isn't instant; once deleted, recordings can't be restored. **Export to JSON** (more-options menu) preserves a recording past expiry and can be re-imported.
