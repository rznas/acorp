# Identity resolution, persons, and merges

## Contents
- [Persons and distinct IDs](#persons-and-distinct-ids)
- [When a person profile is created](#when-a-person-profile-is-created)
- [What `identify` does](#what-identify-does)
- [How merges work (person processing)](#how-merges-work-person-processing)
- [Duplicate person profiles (the #1 data-quality bug)](#duplicate-person-profiles-the-1-data-quality-bug)
- [`reset()` and reverting to anonymous](#reset-and-reverting-to-anonymous)
- [The person profile page (what a PM can see)](#the-person-profile-page-what-a-pm-can-see)
- [Initial vs latest person properties](#initial-vs-latest-person-properties)

## Persons and distinct IDs

A **person** represents the human behind the events. Every event has a **distinct ID** that groups events together, but **not every distinct ID has a person profile**:

- **Anonymous events** use an SDK-generated UUID distinct ID and create **no** profile.
- **Identified events** (after `identify`) create a profile and link it to both the identified ID and the prior anonymous IDs from that session.

If the UI shows a UUID instead of a name/email, the user is anonymous (not yet identified). Anonymous events are still stored; calling `identify` later creates the profile, merges the prior anonymous events, and links the IDs.

## When a person profile is created

A profile is created when you:

- Call `identify()` with a distinct ID,
- Set person properties (`setPersonProperties()`), or
- Capture an event with `$set` / `$set_once`.

To control cost, configure which events need profiles (`personProfiles` / `person_profiles` config: `identified_only` vs `always`). **Anonymous events never store person data** — switching config to `always` later does **not** retroactively recover person properties from past anonymous events.

## What `identify` does

`identify` ties an anonymous visitor to a stable, known user ID (commonly your internal user ID or email). Best practices from the docs:

1. **Call `identify` as soon as you can** (right at login) — on **every device and browser**.
2. **Use unique, stable strings** for distinct IDs (never reuse one across different real users).
3. Set up person properties at the same time (`$set` / `$set_once`).

## How merges work (person processing)

During ingestion, PostHog decides which person an event belongs to using `distinct_id` (and, for client `$identify`, `$anon_distinct_id`). For an `$identify` event:

| Scenario | Result |
|---|---|
| Neither ID seen before | Create a new person, map the distinct ID to it |
| Only one of the two IDs seen | Map both IDs to the existing person |
| **Both IDs already belong to different persons** | **Merge**: the anonymous person is merged into the identified person; future anonymous events go to the identified person |

On property conflicts during a merge, the **non-anonymous person's values win** (it's the one with meaningful history). `$create_alias` / `$merge_dangerously` do the same but let you merge two arbitrary distinct IDs.

Person properties are applied in **ingestion order, not by event timestamp**: `$set` overwrites (last *ingested* wins), `$set_once` only sets if the key is absent. A late-arriving/offline/retried/imported event can therefore overwrite a value set by a more recent event ingested earlier — important when events arrive out of order.

## Duplicate person profiles (the #1 data-quality bug)

One real user can split into multiple profiles when `identify` isn't called consistently — e.g. the user is on phone + laptop but `identify` only fires on one, so each device keeps its own anonymous profile. Another cause: using different distinct IDs for the same user across systems without `alias`.

When PostHog **can't** merge two already-identified profiles, it blocks the merge and logs the **"Refused to merge an already identified user"** ingestion warning.

**PM impact:** duplicates inflate cohort sizes and feature-flag targeting counts. If those numbers look too high, check **Data management → Ingestion warnings** and confirm `identify` fires at login on every device/browser. Fix existing duplicates via the merge-users flow.

## `reset()` and reverting to anonymous

Calling `reset()` unlinks the current person and starts a **new anonymous ID** (and a new session/replay) — use it on logout. Note: re-using a previously *identified* distinct ID still produces identified events; you need a fresh distinct ID to capture as anonymous again. Identified events can still be "anonymous" in the privacy sense — a profile exists but need not contain personal data.

## The person profile page (what a PM can see)

Open **People → click a person** to see all properties plus tabs:

- **Events** — everything they triggered (searchable/filterable)
- **Recordings** — their session replays (subject to retention)
- **Logs** — logs carrying their distinct ID (cross-service debugging)
- **Cohorts** — cohorts they belong to
- **Related groups** — groups (orgs/projects) they belong to
- **Feature flags** — flags enabled for them
- **History** — manual profile changes

The **People** list shows Person, ID, Created at, **Last seen** (opt-in, updates hourly), and a delete option. Deleting a person removes all their associated data.

## Initial vs latest person properties

Many attribution properties have two forms on the profile: **initial** (first-touch, e.g. `$initial_utm_source`, `$initial_referrer`) and **latest** (most recent). Initial values for a user identified later are derived from the SDK persistence store, not from the past anonymous events (which hold no person data). Cross-domain or web↔mobile flows need `identify()` or `createPersonProfile()` (with the anonymous ID bootstrapped) to carry attribution across.
