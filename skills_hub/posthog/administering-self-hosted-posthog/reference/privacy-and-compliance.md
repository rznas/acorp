# Privacy & compliance (detailed)

## Table of contents
- [Who is responsible](#who-is-responsible)
- [Controlling collection (before data leaves client)](#controlling-collection-before-data-leaves-client)
- [IP data capture](#ip-data-capture)
- [Processing before storage](#processing-before-storage)
- [Data deletion / right to be forgotten](#data-deletion--right-to-be-forgotten)
- [Data location & residency](#data-location--residency)
- [GDPR](#gdpr)
- [HIPAA](#hipaa)
- [CCPA](#ccpa)

## Who is responsible
You decide what data you collect, whether it complies, and how you inform users. PostHog provides the tools and stores data securely. Hosting type sets your GDPR roles:
- **PostHog Cloud**: PostHog is the data processor; you are the data controller.
- **Self-hosted**: you are **both** processor and controller.
None of the docs are legal advice.

## Controlling collection (before data leaves client)
- **Top-level opt-out**: `posthog.opt_out_capturing()` blocks all autocapture, manual capture, and replays. `posthog.opt_in_capturing()`, `has_opted_out_capturing()`. `opt_out_capturing_by_default: true` to start opted out. Persisted in local storage/cookies (web, configurable via `opt_out_capturing_persistence_type`), shared prefs (Android), files (iOS/RN).
- **Cookieless mode**: `cookieless_mode: 'always'` (no cookie banner needed, no `identify()`, privacy-preserving server-side hash count) or `'on_reject'` (no events until consent given/denied; still counts denied users via hash).
- **Autocapture controls**: Settings > Project > Autocapture & heatmaps, plus per-session SDK config and element masking.
- **Session-replay masking**: mask inputs, text, other elements, and redact network captures. Recommended default = mask all inputs/text, selectively unmask (`maskAllInputs`, `maskTextSelector`, `maskTextFn`). Mobile replay can't selectively unmask — be selective about screens.
- **`before_send` hook** (JS web only): final chance to modify/strip a property before send.

## IP data capture
IPs can be personal data under GDPR. Configure at:
- **Org**: Settings > Organization > General > IP data capture default — applies to all *new* projects. **EU orgs default to disabled.** Existing projects unaffected.
- **Project**: Settings > Project > General > IP data capture configuration — overrides org default. Also "Discard client IP data" toggle under IP data capture configuration.
When discarding, GeoIP enrichment and bot detection can still use the IP *before* it's dropped (except in cookieless server-hash mode, where IP is stripped first).

## Processing before storage
CDP realtime transformations run on all events before storage:
- Hash PII properties.
- IP anonymization.
- Disable default GeoIP enrichment.
- Drop events by filter.
- Custom transformations in the Hog language.

## Data deletion / right to be forgotten
| What | Where | Notes |
|---|---|---|
| Account | Account settings | Deleted immediately; stored data cleared within 30 days. Does not delete projects/orgs you belong to. |
| Project | Project settings | All project data incl. events removed. |
| Organization | Org settings | Async (a couple minutes, blocking screen); all projects' data removed. |
| Persons | Persons tab or API | Optionally delete the person's events. |

Right to be forgotten via API (needs personal API key): query persons (`GET /api/projects/:id/persons?email=...`), then `DELETE /api/projects/:id/persons/:uuid?delete_events=true`. Bulk delete returns counts (`persons_found`, `persons_deleted`, `events_queued_for_deletion`, etc.). **Event deletion is asynchronous**, run off-peak (weekends on Cloud) because ClickHouse deletes are expensive — check via `GET /api/projects/:id/persons/deletion_status?status=pending`. Manual UI deletion: Persons > find by distinct_id/email > view > Delete person.

## Data location & residency
For robust GDPR, docs recommend **PostHog Cloud EU** (Frankfurt) so EU data never leaves the EU. If self-hosting outside the EU (or using Cloud US) with EU users, **anonymize** their personal data via before-storage transformations.

## GDPR
Three principles: have a lawful basis to collect personal data, obtain unambiguous consent (freely given, specific, informed; withdrawable; documented; under-13 needs parental consent), and handle data securely. Setup steps: choose hosting (EU recommended), deploy, secure the instance (HTTPS, restrict access incl. shared dashboard links, careful with CDPs), configure consent (list PostHog as a tool on Cloud; self-host may use a generic "Product Analytics" label), control collection/storage, honor right-to-be-forgotten. PII-management features map: autocapture controls, masking, event sanitizing (collection); anonymizing transformations (before ingestion).

## HIPAA
PostHog can capture data under HIPAA, but a **Business Associate Agreement (BAA) is only offered for PostHog Cloud** — contact Sales. (Google Analytics is not HIPAA compliant.)

## CCPA
Applies to qualifying for-profit businesses collecting personal information on California residents. Use the same collection/storage controls; see the CCPA doc for specifics.
