---
name: managing-feature-flags
description: Guides product managers using PostHog feature flags to ship features safely without redeploying code. Covers creating boolean release toggles, multivariate flags, and remote config; percentage rollouts; user/person, group/organization, and device targeting; release conditions and cohorts; canary and phased (gradual) rollouts; kill switches; beta and early access management; scheduled and recurring flag changes; flag dependencies; and cleaning up stale flags. Use when a PM asks how to gradually roll out a feature, set up a kill switch, give beta or early access to specific users or accounts, do a targeted release, ramp a feature to a percentage of users, schedule a flag change, or interpret and decide on a rollout's progress.
---

# Managing feature flags in PostHog

Feature flags let a PM toggle a feature on/off and control *who* sees it without a code deploy. This skill maps PM questions to the right flag setup, how to read the result, and what decision it supports.

> Self-hosted note: Core feature flags (create, target, rollout, schedule, dependencies, remote config, cleanup) are fully self-serve and available on self-hosted PostHog. The natural-language **PostHog AI / Max** assistant and the **MCP server** for managing flags are managed/cloud-oriented features; availability and setup may differ on self-hosted — see the using-posthog-ai-assistant skill and confirm in your instance.

## PM question → feature map

| PM question | Use this | Where it lives |
|---|---|---|
| "Turn a feature on/off without a deploy" | Boolean **release toggle** flag | Feature flags tab → New |
| "Test 2+ versions / button colors" | **Multivariate** flag (variants + rollout %) | New → Multivariate |
| "Ship config/values without deploying" | **Remote config** flag (payload) | New → Remote config |
| "Roll out gradually to X% of users" | Rollout percentage on a release condition | Flag → Release conditions |
| "Give beta access to specific users/accounts" | Person-property / cohort / group targeting | Release conditions → Match by |
| "Turn it on for a whole company/org" | **Group** targeting | Match by → group type |
| "Run a self-serve beta opt-in / waitlist" | **Early access management** | Early Access Management tab |
| "Kill switch if something breaks" | Disable the flag (one toggle) | Flag overview → toggle off |
| "Phased rollout: 5% → 25% → 100%" | Edit rollout % over time, or schedule it | Release conditions / Schedule tab |
| "Change the flag at midnight / off-hours" | **Scheduled flag changes** | Flag → Schedule tab |
| "Enable feature only after another is live" | **Flag dependencies** | Release conditions → Feature flags |
| "Measure if the rollout helped a metric" | Filter insights/funnels by the flag | Product analytics / Session replay |
| "Clean up old flags we don't need" | Stale flag cleanup workflow | Flags list → filter Stale |

## The three flag types

| Type | Returns | PM use |
|---|---|---|
| **Release toggle (boolean)** | `true`, `false`, or `null`/`undefined` if not evaluated | On/off feature gating, kill switches, canary releases |
| **Multivariate (A/B/n)** | a variant key (e.g. `control`, `test`) | Compare 2+ experiences; assign % per variant |
| **Remote config** | always the same payload (JSON) to everyone | Ship config values (limits, copy, URLs) without a deploy; default 100% rollout |

**Payloads**: any flag can carry JSON delivered when matched. Boolean flags get one payload; multivariate flags get a payload *per variant*; remote config *is* a payload. This lets a PM change values (pricing, copy, limits) without engineering redeploying.

## Creating a flag (PM walkthrough)

1. **Feature Flags tab → New**. Pick boolean, multivariate, remote config, or experiment.
2. **Key** — the unique string code checks, e.g. `new-checkout`. Cannot be safely renamed later (it would break SDK calls), so agree on it with engineering up front. Add a **description** documenting intent and rollout plan.
3. **Served value** — choose the type; for multivariate set each variant key + its rollout %.
4. **Release conditions** — who gets it (see targeting below).
5. Optional: usage dashboard (on by default — tracks call volume and variant split), persistence across login, evaluation runtime (client/server/both), tags.
6. Leave **Enable feature flag** on to go live, or off to stage it.

You can **inline-edit** release conditions, variants, payload, and tags directly from the flag overview page (one section at a time). The **key** must be changed from the full edit form with confirmation. Experiment-linked flags lock variants/conditions (the experiment manages them).

> Tip: Set project-wide **default release conditions** (Settings → Feature Flags) so new flags pre-populate with your team's standard rollout pattern. Enable **flag change confirmation** with a custom warning so edits that affect live users require a deliberate click.

## Targeting and release conditions (overview)

A **release condition** = property filters + rollout %. By default all condition sets are evaluated and *any* pass = match. The **Match by** dropdown picks the entity that gets bucketed: **User** (distinct ID, default), **Device** (anonymous/pre-login consistency), **Group** (whole org/team/company — needs group analytics), or **User & Group** (mixed, beta).

Quick reference:
- **Rollout %** supports decimals down to 0.01%. Hashing is deterministic — the same user stays in/out as you ramp, so users don't churn in and out when you raise the percentage.
- **Person properties / cohorts** target individuals (e.g. `email contains @acme.com`, or a `Beta testers` cohort). Behavioral/lifecycle cohorts can't be used directly — duplicate as a **static cohort** first.
- **Group properties** (e.g. `plan = enterprise`) turn a feature on for every user in matching organizations.
- **Semver** operators target app versions (e.g. `$app_version >= 2.0.0`) — useful for mobile staged rollouts.
- **Stop evaluation at first matching condition set** makes evaluation deterministic top-to-bottom (first match wins) instead of "any pass."

→ Full detail, mixed-targeting evaluation order, group setup, non-user (page/server/device) targeting, and worked examples: see **[reference/targeting-and-conditions.md](reference/targeting-and-conditions.md)**.

## Rollout patterns

### Kill switch
A boolean flag wrapping a feature *is* a kill switch. If something breaks, **disable the flag** (one toggle on the overview page) — the feature turns off for everyone instantly, no deploy. Keep intentional kill-switch flags tagged so cleanup workflows don't remove them.

### Canary release (validate on a few before everyone)
Recommended staged sequence (edit the release condition at each step):
1. **Just yourself** — `email equals you@company.com`, 100%. Verify the feature and that toggling off works.
2. **Internal team** — `email contains @yourcompany.com`. Testing in production.
3. **Beta users / orgs** — early access management, a company domain, or a group name.
4. **Expanded beta** — a rollout % or a matching property; watch overall metrics.
5. **Full release** — 100%, recheck metrics, then clean up the flag.

**Read it**: filter insights/funnels and session replays by the flag to compare exposed vs. not. **Decide**: proceed to the next stage only if metrics hold and feedback is clean; otherwise disable.

### Phased / gradual rollout
Start at 5–10%, monitor, increase. Either edit the rollout % over time, or pre-define each phase as a **cohort** and swap cohorts in the condition. To measure impact rigorously, attach an experiment. To automate the ramp, use scheduled changes (below).

### Scheduled & recurring changes
Flag → **Schedule** tab. Change types: **Change status** (enable/disable at a time), **Add condition** (append a rollout condition later), **Update variants**. Schedules use the **project timezone**.
- **Repeats**: daily/weekly/monthly/yearly or **custom cron** (5-field). Good for "enable at midnight for a ticket sale" or recurring maintenance windows.
- **Paired schedules** (for Change status): create complementary enable+disable schedules at once — presets **Business hours** (9am–5pm Mon–Fri) and **Weekdays only**, or a custom pair.
- Recurring is supported for Change status and Update variants, **not** Add condition (would duplicate).

### Dependencies (sequenced rollouts)
Make a **dependent flag** require a **base flag**'s value (`base_flag equals true` or a specific variant). PostHog evaluates the chain automatically. Use for "enable `new_dashboard` only for the 10% in `beta_program`," conditional experiments, or safety gating. Keep chains shallow; avoid circular deps. Note: dependencies are limited when flags have different evaluation runtimes.

## Early access management (self-serve beta program)

Let users opt in/out of features themselves — beta programs, waitlists, "coming soon" interest. Lives in the **Early Access Management** tab; each feature links to a flag.

> Self-hosted note: Early access management works only with the **JavaScript Web SDK** and does **not** support groups. The opt-in widget needs `opt_in_site_apps: true` in the SDK config.

**Lifecycle stages** (control whether opted-in users get the flag enabled):

| Stage | Flag on for opt-ins? | PM purpose |
|---|---|---|
| Draft | No | Setting up; invisible to users |
| Concept | No | Gauge demand / waitlist before building |
| Alpha | Yes | Small early-test group |
| Beta | Yes | Wider testing (default shown in opt-in UI) |
| General availability | Yes | Stable; becomes read-only after promotion |

Opt-in is an **overriding condition**: it beats the flag's normal release conditions. Promoting to **GA** offers "Roll out to all users — include users who previously opted out" — check it to clear opt-out conditions and set 100%; leave unchecked to respect prior opt-outs. Stage changes fire a `$feature_enrollment_update` event you can use to notify users. You can also opt users in/out manually from the feature detail screen.

## Testing a flag before rollout

1. **Assign yourself a value** — add a condition `email = you@...` (or `distinct_id equals`) at 100%; for multivariate set the **optional override** to the variant you want. For logged-out users, target a `utm_source` and append `?utm_source=...` to the URL.
2. **In-code override** — `posthog.featureFlags.overrideFeatureFlags()` (JS web / React Native only).
3. **PostHog toolbar** — toggle flags live on your own site (JS web only).

## Measuring & deciding

- **Usage dashboard** (auto-created) shows call volume and variant distribution — confirms the flag is being evaluated and how users split.
- **Breakdown by flag**: in any insight/funnel, break down or filter by the feature flag to compare exposed vs. control (e.g. signup conversion). Click a funnel step to jump to **session replays** for that group.
- Pair a rollout with an **in-app survey** for qualitative feedback.
- **Decision**: ramp up if the target metric holds/improves and no error spike; disable (kill switch) if metrics drop or errors rise.

## Cleaning up stale flags

A flag at **100% with no targeting for 30+ days**, or **not evaluated in 30+ days**, is stale — it still incurs billable `/flags` evaluations and clutters code. Filter the flags list to **Stale** to find them.

**Safe order** (do not skip): 1) Identify → 2) remove the flag check from code (keep the winning path) → 3) deploy → 4) **disable** in PostHog (now a no-op). Disable is reversible; delete is a soft-delete (restorable from the flag page, key freed). Flags in use by an early access feature, a *running* experiment, a survey, session replay conditions, or another active flag are blocked from deletion until you remove the reference. Bulk delete supports multi-page and select-all-matching.

→ Cost reasoning, code-search patterns per SDK, VS Code extension, and CI automation: see **[reference/targeting-and-conditions.md](reference/targeting-and-conditions.md)** (cleanup section).

## Manage flags with AI / MCP

PostHog AI (Max) creates and manages flags from plain language — "Create a dark-mode flag rolled out to 20%," "Roll out `dark-mode` to 50%," "Which flags haven't been evaluated in 90 days?", "Disable `new-checkout`." It can also generate code-cleanup prompts. The **MCP server** gives the same control plus bulk tag/delete and scheduled-change tools to an AI coding agent in the editor.

> Self-hosted note: these AI/MCP helpers are cloud-oriented; on self-hosted, confirm availability and setup. See the **using-posthog-ai-assistant** skill for the PM value and how to drive it.

## Related skills

- **running-experiments** — flags power A/B tests, holdouts, and statistical analysis.
- **using-posthog-ai-assistant** — natural-language flag creation/management and MCP.
- **analyzing-with-cohorts** / **building-insights-and-dashboards** — segment rollouts and measure flag impact.
- **capturing-and-managing-events** — identify users and set person/group properties so targeting evaluates correctly.
