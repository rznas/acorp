# Targeting, release conditions, and cleanup (reference)

Deep detail for the **managing-feature-flags** skill: how PostHog decides who gets a flag, every targeting mode, mixed targeting evaluation, non-user targeting, and the full stale-flag cleanup workflow.

## Contents
- [How evaluation works](#how-evaluation-works)
- [Match by: targeting modes](#match-by-targeting-modes)
- [Rollout percentage](#rollout-percentage)
- [Property and cohort filters](#property-and-cohort-filters)
- [Group / organization targeting](#group--organization-targeting)
- [Mixed user + group targeting](#mixed-user--group-targeting)
- [Stop evaluation at first matching condition set](#stop-evaluation-at-first-matching-condition-set)
- [Semver (app version) targeting](#semver-app-version-targeting)
- [Device targeting and persistence](#device-targeting-and-persistence)
- [Non-user targeting: pages, servers, machines](#non-user-targeting-pages-servers-machines)
- [Flag dependencies](#flag-dependencies)
- [Worked targeting examples](#worked-targeting-examples)
- [Stale flag cleanup workflow](#stale-flag-cleanup-workflow)

## How evaluation works

A flag is a deterministic (pure) function: it hashes the **flag key + the entity ID** (distinct ID by default) and returns the same result every time for the same inputs. On top of the hash, PostHog layers:

1. **Property targeting** — does the entity match the condition's filters? If not, the flag returns `false` (the hash never runs).
2. **Rollout percentage** — is the entity's hash position below the threshold?
3. **Variant assignment** (multivariate) — a second salted hash maps to a variant range.

Practical consequence for a PM: "the flag flipped for a user" is almost always an **input problem** — the distinct ID or a targeted property changed (e.g. `identify()` ran after evaluation). It is not random. Target on **stable** properties; avoid "Latest" auto-properties (like Latest Current URL) that change on every event.

## Match by: targeting modes

Each condition set has its own **Match by** setting (per condition, not per flag).

| Mode | Bucketed on | When to use |
|---|---|---|
| **User** (default) | `distinct_id` | In-app features for logged-in users |
| **Device** (beta) | `$device_id` | Anonymous / pre-login; consistent through signup |
| **Group** | a group key (e.g. `company_id`) | On/off for an entire org, team, or account |
| **User & Group** (beta) | per condition | One flag mixing user- and group-level conditions |

## Rollout percentage

- Available on every flag type.
- Supports decimals to 0.01% precision (e.g. `0.15%`, `33.33%`) — type decimals in the text input; the slider uses whole steps.
- Deterministic: raising 10% → 25% keeps the original 10% included and adds new users. Users don't drop out when you ramp up.

## Property and cohort filters

Available when you capture identified events:
- **Person properties** — e.g. `email`, `plan`, custom traits. `distinct_id equals` searches persons by name/email/ID.
- **Cohorts** — reuse a saved segment as a condition.
- **GeoIP** (if enabled) — target by location.

**Behavioral cohort caveat**: cohorts with event-based (behavioral/lifecycle) filters can't be used directly in a flag — evaluating them is too slow for flag speed. Workaround: build the behavioral cohort, then **duplicate it as a static cohort** and use that. (E.g. "users who viewed the pricing page 3+ times.")

## Group / organization targeting

Turns a feature on for an entire organization/team — everyone in the group sees the same value because PostHog hashes the **group key**, not the user.

Prerequisites (group analytics):
1. Define group types in PostHog (e.g. `company`, `team`).
2. Send `.group()` / `groupIdentify` from SDKs so PostHog knows each user's group.

Setup: create/edit flag → Release conditions → **Match by** = your group type → optionally filter on group properties (e.g. `plan equals enterprise`) → set rollout %. Example: a 50% org rollout hashes each org's key; all users in an included org get it.

Example `.group()` call (illustrative, engineering implements):
```js
posthog.group('company', 'company_abc', { name: 'Acme Inc', plan: 'enterprise' })
```

## Mixed user + group targeting

(Beta — if the **User & Group** option is missing in Match by, contact support.) Lets one flag combine user-level and group-level conditions. Each condition set picks its own targeting type.

**Evaluation is top-to-bottom, first match wins.** User-targeted and group-targeted conditions hash different values, so their rollout distributions are **independent** — a 50% group rollout and a 25% user rollout do not overlap predictably.

**Caution**: if a group-targeted condition needs a group type the SDK call didn't include, that condition is **silently skipped** and evaluation falls through to the next condition — a misconfigured SDK call can push users to a broader catch-all. Verify SDK calls send the right `groups` data. Order conditions most-specific-first.

Property-filter rules: user conditions can filter person properties/cohorts/other flags; group conditions can only filter that group type's properties (enforced by the UI).

## Stop evaluation at first matching condition set

Default: all condition sets evaluated, any pass = match — so a user excluded by one condition's rollout can still match a later one. Enable this option to evaluate in order; the **first matching condition set decides**, and if a user matches the filters but falls outside that condition's rollout, evaluation stops and returns `false` (no fall-through to a broader catch-all). Local-evaluation support for this varies by SDK.

## Semver (app version) targeting

Use semver operators on string properties like `$app_version` / `$lib_version` (labeled `(semver)` in the operator dropdown): add a string property → pick a semver operator → enter a version (e.g. `1.2.0`). Ideal for mobile staged rollouts — enable for everyone on `>= 2.0.0`, or hotfix only `< 1.5.0`. Mobile SDKs include `$app_version` automatically.

## Device targeting and persistence

- **Device bucketing** (`Match by` = Device) keeps a consistent experience for anonymous users across the login transition on the same device — without the cost of persistence.
- **Persisting flags across authentication** (optional, off by default) keeps the same value before/after login but is incompatible with local evaluation and bootstrapping, requires Person Profiles + `identify()`, and slows responses. PostHog recommends device bucketing or a stable identity flow instead for most cases.

## Non-user targeting: pages, servers, machines

A "user" is just a distinct-ID string, so you can target anything:
- **Pages** — use a flag **payload** mapping URLs to values (e.g. button colors per page), or a static cohort of repeat page-viewers.
- **Servers / services / regions** — send the entity ID as the distinct ID (e.g. `distinct_id="canada-cloud-1"`), filter on a property like `server_id equals canada-cloud-1`, and evaluate the flag with that ID. Enables canary releases to specific servers/regions.
- **One-time interactions** — set a person property after first use and target `is_first_interaction is not false` so the experience happens once.

## Flag dependencies

Make a **dependent flag** require a **base flag**'s state. Add condition → property type **Feature flags** → pick the base flag → set `equals true`, `equals false`, or a specific variant (`equals true` matches any variant). PostHog auto-evaluates the chain. Rollout % on the dependent flag still applies on top. Limited when flags have different evaluation runtimes; avoid circular chains.

Pattern: base `beta_program` at 10% → dependent `new_dashboard` with condition `beta_program = true` at 100% → only beta users see the dashboard.

## Worked targeting examples

**Tech companies + 25% of everyone else** (first match wins):
| # | Match by | Filter | Rollout |
|---|---|---|---|
| 1 | Group: company | `industry equals tech` | 100% |
| 2 | User | (none) | 25% |

**Guarantee one org + 10% of all users:**
| # | Match by | Filter | Rollout |
|---|---|---|---|
| 1 | Group: company | `name equals Acme Inc` | 100% |
| 2 | User | (none) | 10% |

**Enterprise orgs + individual beta opt-ins:**
| # | Match by | Filter | Rollout |
|---|---|---|---|
| 1 | Group: company | `plan equals enterprise` | 100% |
| 2 | User | Cohort `Beta testers` | 100% |

## Stale flag cleanup workflow

**Why it matters**: billing is per `/flags` request, and a request is billable when it evaluates at least one active billable flag — even if your code no longer checks it (the web SDK and `getAllFlags()` evaluate active flags). Stale flags = unnecessary cost + clutter.

**What counts as stale**: not evaluated in 30+ days, OR at 100% with no property filters for 30+ days (a hardcoded value in disguise). Disabled flags are not stale. Filter the flags list by status to find them.

**Correct order (don't skip):**
1. **Identify** stale flags in the app (or ask PostHog AI / MCP).
2. **Remove** the flag check from code, keeping the right path:
   - *Fully rolled out (100%)* → keep the enabled path, delete the `if`/`else`.
   - *Not rolled out (0%)* → keep the fallback path, delete the enabled body.
   - *Partial* → manual review of intent.
3. **Deploy** the code change.
4. **Disable** in PostHog (now a no-op). Re-enabling is instant.

**Disable vs delete**: disable (`active: false`) preserves config, reversible. Delete is a soft-delete — list stays clean, restore from the flag page (key is freed; reused keys get a numeric suffix like `my-flag-2`).

**Code search patterns** (search the flag key as a string everywhere — constants, configs, tests, comments):
| Language | Patterns |
|---|---|
| JS/TS | `isFeatureEnabled('key')`, `getFeatureFlag('key')`, `useFeatureFlag('key')` |
| Python | `flags.is_enabled('key')`, `flags.get_flag('key')`, `feature_enabled('key')` |
| Node | `flags.isEnabled('key')`, `flags.getFlag('key')`, `isFeatureEnabled('key')` |
| Ruby | `flags.enabled?('key')`, `flags.get_flag('key')` |
| Go | `flags.IsEnabled("key")`, `flags.GetFlag("key")` |
| Other | the key as a string literal |

**Tooling**:
- **VS Code extension** — AST-based scan/removal for JS/TS, batch across files (review diffs).
- **PostHog AI / MCP** — list stale flags (`active: STALE`), disable, and generate code-cleanup prompts.
- **CI automation** — three-step propose → human decide → execute (`PATCH active:false`). Paginate the API list; sort stably; default to "skip"; treat delete as more sensitive than disable.

**Hygiene tips**: tag intentional kill switches / dormant flags so cleanup skips them; annotate at creation (`safe-to-remove-at-100%`); wrap a flag used in many places in one function; name flags clearly (positive booleans, purpose suffixes); pick an evaluation runtime deliberately instead of leaving the "both" default.
