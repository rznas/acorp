# Releases, source maps, and integrations

Table of contents
- [Why a PM cares about releases](#why-a-pm-cares-about-releases)
- [Creating releases](#creating-releases)
- [Release context on exceptions](#release-context-on-exceptions)
- [Source linking](#source-linking)
- [Source maps and stack traces](#source-maps-and-stack-traces)
- [Integrations](#integrations)
- [Creating external issues](#creating-external-issues)

## Why a PM cares about releases

A **release** describes a single deployment — its uploaded artefacts, version, commit, and build metadata. Error Tracking overlays release info on stack traces so you can trace an issue down to the commit and version that produced it. This is the key tool for answering "did this deploy cause the regression?": filter issues by release and check **First seen**.

## Creating releases

Release info is created automatically when engineering uploads source maps via `posthog-cli sourcemap upload`, or via PostHog framework packages (Next.js, Nuxt). The CLI finds/creates the release for the detected project + version and records it in each source map. It can take `--release-name`, `--release-version`, and `--build` (e.g. iOS `CFBundleVersion`, Android `versionCode`); otherwise it infers values from Git (repo name, commit SHA). Artefacts are locked to their release once written. A PM specs "create a release per deploy"; engineering wires the CLI/CI.

## Release context on exceptions

Captured exceptions inherit the release info recorded during source-map injection. PostHog attaches release, project, and Git metadata to each exception in the ingestion pipeline, so you can see release details on issues and filter issues by release.

## Source linking

With a GitHub or GitLab integration plus a release that includes repository + commit info, PostHog adds **View commit** buttons to each stack frame, linking to the exact file and line in your repo. Setup:

1. Connect a GitHub or GitLab integration.
2. Create a release with repository and commit information.
3. PostHog auto-adds View commit links to stack traces.

This shortens "error -> exact source line" investigation.

## Source maps and stack traces

Source maps turn minified production traces into readable code. Beyond readability, resolved stack traces drive **accurate fingerprinting and grouping** — without them, issue grouping is inconsistent. A PM should treat "upload source maps for every deploy" as a baseline requirement for trustworthy error data.

## Integrations

| Integration | Capabilities |
| --- | --- |
| GitHub | Pinpoint commits causing issues, open files from stack traces, create GitHub issues from errors |
| GitLab | Same for GitLab (connect with a project access token, `api` scope) |
| Linear | Create Linear issues from errors, track fixes in Linear |
| Jira | Create Jira issues from errors, track fixes in Jira (OAuth 2.0) |

GitHub connects via a GitHub App (read metadata; read/write code, issues, PRs). After connecting, create a release to enable source linking.

> Self-hosted note: integrations rely on OAuth/app connections to third-party services; confirm outbound connectivity and any required configuration on your self-hosted instance.

## Creating external issues

After setting up an integration, open an issue -> **External references** -> **Create issue**. The external ticket gets a partial stack trace and a link back to the PostHog issue, so engineering tracks the fix in their existing workflow while the PM keeps the PostHog issue as the source of truth for impact.
